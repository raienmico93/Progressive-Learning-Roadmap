# Embedded Content Elements — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Embedded content elements are the HTML elements that import external resources or insert content from other vocabularies into the document — images, responsive image sets, nested browsing contexts, plugin-hosted resources, video, audio, timed text tracks, image maps, bitmap drawing surfaces, and inline XML vocabularies such as SVG and MathML.

**Technical Definition**
In the WHATWG HTML Living Standard, embedded content is defined as “content that imports another resource into the document, or content from another vocabulary that is inserted into the document”. The embedded content category includes `<img>`, `<picture>`, `<source>`, `<iframe>`, `<embed>`, `<object>`, `<param>`, `<video>`, `<audio>`, `<track>`, `<map>`, `<area>`, and `<canvas>`. Additionally, elements from foreign namespaces — notably `<svg>` and `<math>` — are treated as embedded content when they appear in HTML documents. Most embedded content elements belong to the **flow content**, **phrasing content**, and **palpable content** categories, with some (such as `<iframe>`, `<video controls>`, and `<audio controls>`) also belonging to the **interactive content** category.

**Beginner-Friendly Explanation**
Embedded content elements are the “windows” in an HTML page that show something from outside the HTML file itself. An image is a window to a picture file; an iframe is a window to another web page; a video element is a window to a movie. Some embedded elements — like `<canvas>`, `<svg>`, and `<math>` — are windows into entirely different languages (drawing commands, vector graphics, and mathematical notation) that the browser knows how to render.

---

### Key Characteristics

- **External resource integration**: Embedded elements bring resources from outside the document into the page — files, URLs, or foreign-namespace markup.
- **Content-model constraints**: Each element has a defined content model; for example, `<picture>` must contain one `<img>` and may contain `<source>` elements, while `<iframe>` has a content model of “nothing” in the current standard.
- **Performance impact**: Every embedded resource adds a network request and processing cost; iframes in particular introduce a separate browsing context and document.
- **Accessibility-critical**: Embedded content must provide alternative text, captions, titles, or fallback content to be perceivable by assistive technologies.
- **Security-sensitive**: `<iframe>`, `<embed>`, and `<object>` can load third-party code; attributes such as `sandbox` and `referrerpolicy` control their capabilities.
- **Foreign vocabulary support**: `<svg>` and `<math>` allow XML-based vocabularies to be embedded directly in HTML, with their own element sets and rendering rules.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<html>`, `<head>`, and `<body>`.
2. **Phrasing and flow content**: understanding which elements can appear inside paragraphs and other containers.
3. **Introductory CSS**: enough to understand how embedded elements are sized and positioned.
4. **Browser developer tools**: the Network panel for observing resource loading and the Elements panel for inspecting embedded content.
5. **Basic accessibility awareness**: understanding why alternative text and captions matter.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Web performance** | Images, video, and iframes are often the largest contributors to page weight; `loading`, `preload`, and `srcset` directly affect performance. |
| **Accessibility (WCAG)** | `alt` text, captions, iframe `title`, and fallback content are required for WCAG conformance. |
| **Security** | `sandbox`, `referrerpolicy`, and `allow` control what embedded third-party content can do. |
| **Internationalisation** | `<track>` captions and subtitles support multilingual media; `<math>` supports mathematical notation across languages. |
| **JavaScript / Canvas API** | `<canvas>` is a bitmap surface whose contents are drawn and manipulated entirely by script. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. `<img>` — embedded image
2. `<picture>` — responsive image container
3. `<source>` — media resource candidate
4. `<iframe>` — nested browsing context
5. `<embed>` — external plugin content
6. `<object>` — generic external resource
7. `<param>` — object parameter (deprecated)
8. `<video>` — video playback
9. `<audio>` — audio playback
10. `<track>` — timed text track
11. `<map>` — image map container
12. `<area>` — image map clickable area
13. `<canvas>` — scriptable bitmap surface
14. `<svg>` — inline Scalable Vector Graphics
15. `<math>` — inline MathML
16. **Embedded content best practices**

---

## 1. `<img>` — The Embedded Image Element

### Definitions

**Core Definition**
The `<img>` element embeds an image in the document. It is a void element, and its `src` attribute specifies the URL of the image resource.

**Technical Definition**
`<img>` is a void element belonging to the **flow content**, **phrasing content**, **embedded content**, and **palpable content** categories (when it has a non-empty `alt` or is otherwise not decorative). The `src` attribute is required and must be a valid non-empty URL. The `alt` attribute is required except in specific cases such as when the image is purely decorative and accompanied by other content that already conveys the same information. The `srcset` and `sizes` attributes allow the browser to select among multiple image resources based on device pixel density and viewport size.

**Beginner-Friendly Explanation**
The `<img>` element puts a picture on the page. You tell it where the picture is with `src`, and you describe the picture with `alt` so that people who cannot see it still know what it shows.

---

### Purposes

- **To display a single image resource** on a page.
- **To provide accessible alternative text** via `alt` so that screen-reader users and search engines understand the image’s content.
- **To support responsive images** via `srcset` and `sizes`, allowing the browser to choose an appropriately sized image for the device.
- **To enable lazy loading** via `loading="lazy"` so off-screen images are not fetched until needed.
- **To support image maps** by linking to a `<map>` via `usemap`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img
  src="image.jpg"
  alt="Descriptive alternative text"
  width="400"
  height="300"
  srcset="image-small.jpg 480w, image-large.jpg 1024w"
  sizes="(max-width: 600px) 480px, 1024px"
  loading="lazy">
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | **Required.** URL of the image resource. |
| `alt` | **Required** (except for decorative images). A textual alternative for the image. |
| `width` / `height` | Intrinsic dimensions; help the browser reserve space and prevent layout shift. |
| `srcset` | Candidate images with width (`w`) or pixel-density (`x`) descriptors. |
| `sizes` | Media conditions paired with source sizes; used with `srcset` width descriptors. |
| `loading` | `lazy` or `eager`; controls when the image is fetched. |
| `usemap` | References a `<map>` element for an image map. |

#### Syntax Rules

1. **`<img>` is a void element**; it has no end tag and no children.
2. **`src` must be present and non-empty** for a meaningful image.
3. **`alt` must be present**; use `alt=""` for purely decorative images, but do not omit the attribute.
4. **`width` and `height` should be provided** to reserve space and prevent layout shifts.
5. **`srcset` and `sizes` work together** for responsive image selection.

#### Constraints and Limitations

- `<img>` must not be used as a layout tool; use CSS for positioning and sizing.
- An `<img>` with `usemap` must not be a descendant of an `<a>` element.
- Omitting `alt` entirely is non-conforming; decorative images should use `alt=""`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Image with Alternative Text

**Setup guide**: Save as `img-basic.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Basic Image</title>
</head>
<body>
  <h1>Marie Lloyd</h1>

  <!-- The img element embeds the image and provides alternative text -->
  <img
    src="/uploads/100-marie-lloyd.jpg"
    alt="Portrait of Marie Lloyd, an English music hall singer"
    width="100"
    height="150">
</body>
</html>
```

**Expected output**: A portrait image of Marie Lloyd rendered at 100 × 150 CSS pixels. If the image fails to load, the alternative text is shown instead.

**Why this output occurs**: The `src` attribute loads the image resource, and the browser renders it at the specified dimensions. The `alt` attribute provides a textual alternative for screen readers and when the image is unavailable. This matches the WHATWG example of embedding a single image resource.

---

#### Example 2: Responsive Image with `srcset` and `sizes`

**Setup guide**: Save as `img-responsive.html`. Provide image files at different widths (e.g. `photo-480.jpg`, `photo-1024.jpg`).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Responsive Image</title>
</head>
<body>
  <img
    src="photo-1024.jpg"
    alt="A mountain landscape at sunrise"
    srcset="photo-480.jpg 480w, photo-1024.jpg 1024w"
    sizes="(max-width: 600px) 480px, 1024px"
    width="1024"
    height="683">
</body>
</html>
```

**Expected output**: On a narrow screen (≤600px), the browser may load `photo-480.jpg`; on a wider screen, it may load `photo-1024.jpg`. The browser chooses the most appropriate candidate based on the `sizes` media conditions and the device’s capabilities.

**Why this output occurs**: The `srcset` attribute provides candidate images with width descriptors (`480w`, `1024w`), and `sizes` tells the browser how much viewport width the image will occupy under different media conditions. The browser selects the smallest candidate that is likely to be displayed without pixelation.

---

### Real-World Cases

1. **Article images**: News and blog posts use `<img>` with descriptive `alt` text to embed photographs and illustrations.
2. **Responsive product photos**: E-commerce sites use `srcset` and `sizes` to serve appropriately sized product images to phones and desktops.
3. **Lazy-loaded galleries**: Image galleries use `loading="lazy"` to defer off-screen images, improving initial page load performance.

---

### References

- WHATWG — Images — https://html.spec.whatwg.org/dev/images.html
- MDN Web Docs — `<img>`: The Image Embed element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img
- W3C — HTML 5.2: The `img` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-img-element

---

## 2. `<picture>` — The Responsive Image Container

### Definitions

**Core Definition**
The `<picture>` element contains zero or more `<source>` elements and exactly one `<img>` element to offer alternative versions of an image for different display or device scenarios.

**Technical Definition**
`<picture>` belongs to the **flow content**, **phrasing content**, **embedded content**, and **palpable content** categories. Its content model is zero or more `<source>` elements followed by exactly one `<img>` element. The browser evaluates each `<source>` element’s `media`, `type`, and `srcset` attributes and selects the first matching candidate. The `<img>` element serves two purposes: it describes the image’s dimensions and attributes, and it provides a fallback if no `<source>` matches.

**Beginner-Friendly Explanation**
The `<picture>` element lets you give the browser several versions of an image and let it choose the best one. You might provide a wide version for desktop, a cropped version for mobile, and a modern-format version for browsers that support it. The browser picks the first version that fits.

---

### Purposes

- **To support art direction**: providing cropped or modified images for different media conditions (e.g. a simpler image on small screens).
- **To offer alternative image formats**: serving AVIF or WebP to supporting browsers while falling back to JPEG or PNG.
- **To save bandwidth**: loading the most appropriate image for the viewer’s display, rather than a one-size-fits-all resource.
- **To provide a standards-based fallback**: the `<img>` element ensures that an image is still displayed if `<picture>` is unsupported or no `<source>` matches.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<picture>
  <source
    media="(max-width: 600px)"
    srcset="image-square.jpg">
  <source
    type="image/webp"
    srcset="image.webp">
  <img
    src="image.jpg"
    alt="Description of the image"
    width="800"
    height="600">
</picture>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<picture>` | The container. |
| `<source>` | Zero or more candidate resources, each with `media`, `type`, or `srcset`. |
| `<img>` | **Required.** The fallback image and the element that receives the rendered dimensions. |

#### Syntax Rules

1. **`<picture>` must contain exactly one `<img>` element**, typically as the last child.
2. **`<source>` elements must precede the `<img>`**.
3. **The browser selects the first `<source>` that matches**; order matters.
4. **The `<img>` element must have `src` and `alt`**; it is the fallback and the rendered element.

#### Constraints and Limitations

- `<picture>` does not itself render anything; all rendering is done by the selected image, ultimately the `<img>`.
- If no `<source>` matches and the `<img>` has a `src`, the `<img>` resource is used.
- `<picture>` should not be used for simple resolution switching where `srcset` on `<img>` alone suffices; use `srcset` directly on `<img>` for that case.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Art Direction for Different Screen Sizes

**Setup guide**: Save as `picture-art.html`. Provide `image-wide.jpg` and `image-square.jpg`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Art Direction with Picture</title>
</head>
<body>
  <picture>
    <!-- On screens wider than 600px, use the wide crop -->
    <source media="(min-width: 601px)" srcset="image-wide.jpg">

    <!-- On narrower screens, use the square crop -->
    <source media="(max-width: 600px)" srcset="image-square.jpg">

    <!-- Fallback image -->
    <img src="image-wide.jpg" alt="A landscape scene" width="800" height="400">
  </picture>
</body>
</html>
```

**Expected output**: On a wide screen, the wide landscape image is displayed. On a narrow screen, the square crop is displayed. If `<picture>` is unsupported, the fallback `<img>` is shown.

**Why this output occurs**: The browser evaluates the `media` conditions on each `<source>`. The first matching source’s `srcset` is used. This is the art-direction use case described by MDN: cropping or modifying images for different media conditions.

---

#### Example 2: Alternative Image Formats

**Setup guide**: Save as `picture-format.html`. Provide `photo.avif` and `photo.jpg`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Alternative Formats</title>
</head>
<body>
  <picture>
    <!-- Browsers that support AVIF will use this -->
    <source type="image/avif" srcset="photo.avif">

    <!-- Browsers that do not support AVIF fall back to the img -->
    <img src="photo.jpg" alt="A city skyline at night" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: In a browser that supports AVIF, `photo.avif` is loaded. In a browser that does not support AVIF, the `<img>` fallback `photo.jpg` is loaded.

**Why this output occurs**: The `type` attribute on `<source>` tells the browser the MIME type of the candidate resource. If the browser supports that type, it uses the source; otherwise, it skips to the next source or the `<img>` fallback.

---

### Real-World Cases

1. **News sites**: Serving wide hero images to desktop and square crops to mobile.
2. **E-commerce**: Offering WebP or AVIF versions of product photos to modern browsers while retaining JPEG fallbacks.
3. **Performance-focused sites**: Reducing bandwidth by serving the smallest adequate image for each viewport.

---

### References

- MDN Web Docs — `<picture>`: The Picture element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/picture
- WHATWG — The `picture` element — https://html.spec.whatwg.org/multipage/embedded-content.html#the-picture-element
- W3C — HTML 5.2: The `picture` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-picture-element

---

## 3. `<source>` — The Media Resource Candidate

### Definitions

**Core Definition**
The `<source>` element specifies multiple media resources for `<picture>`, `<video>`, and `<audio>` elements. It is a void element.

**Technical Definition**
`<source>` belongs to the **flow content**, **phrasing content**, and **embedded content** categories when used inside `<picture>`, `<video>`, or `<audio>`. Its content model is “nothing” because it is void. Key attributes include `src`, `type`, `srcset`, `sizes`, and `media`. For media elements (`<video>` and `<audio>`), `src` and `type` are used; for `<picture>`, `srcset`, `sizes`, and `media` are used.

**Beginner-Friendly Explanation**
The `<source>` element is a way to give the browser multiple options. For a video, you might provide an MP4 and a WebM file. For an image, you might provide different crops or formats. The browser picks the first option it can play or display.

---

### Purposes

- **To provide alternative media resources** for `<video>`, `<audio>`, and `<picture>`.
- **To specify MIME types** so the browser can skip unsupported formats.
- **To define responsive image candidates** via `srcset` and `sizes` inside `<picture>`.
- **To apply media conditions** via `media` inside `<picture>`.

---

### Syntax Rules and Structure

#### Complete General Syntax (inside `<picture>`)

```html
<source srcset="image.webp" type="image/webp" media="(min-width: 800px)">
```

#### Complete General Syntax (inside `<video>` or `<audio>`)

```html
<source src="video.webm" type="video/webm">
<source src="video.mp4" type="video/mp4">
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the media resource (for `<video>` and `<audio>`). |
| `type` | MIME type of the resource. |
| `srcset` | Candidate image URLs with descriptors (inside `<picture>`). |
| `sizes` | Viewport-relative image sizes (inside `<picture>`). |
| `media` | Media query that determines when the source applies (inside `<picture>`). |

#### Syntax Rules

1. **`<source>` is a void element**; no end tag, no children.
2. **Inside `<video>` and `<audio>`**, use `src` and `type`; the browser selects the first playable source.
3. **Inside `<picture>`**, use `srcset`, `sizes`, and/or `media`; the browser selects the first matching source.
4. **`<source>` must be a child of `<picture>`, `<video>`, or `<audio>`**.

#### Constraints and Limitations

- `<source>` cannot be used as a standalone element outside a media or picture container.
- The order of `<source>` elements matters: the browser picks the first compatible one.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Video Sources

**Setup guide**: Save as `source-video.html`. Provide `movie.webm` and `movie.mp4`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Video Sources</title>
</head>
<body>
  <video controls width="640">
    <source src="movie.webm" type="video/webm">
    <source src="movie.mp4" type="video/mp4">
    Your browser does not support the video element.
  </video>
</body>
</html>
```

**Expected output**: The browser tries `movie.webm` first; if it cannot play WebM, it tries `movie.mp4`. If neither is supported, the fallback text is shown.

**Why this output occurs**: The `<source>` elements provide candidate resources in order. The browser selects the first source whose `type` it supports and whose `src` it can load.

---

#### Example 2: Responsive Image Sources

**Setup guide**: Save as `source-picture.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Picture Sources</title>
</head>
<body>
  <picture>
    <source
      media="(min-width: 800px)"
      srcset="hero-wide.jpg 1200w, hero-wide-small.jpg 800w"
      sizes="800px">
    <source
      media="(max-width: 799px)"
      srcset="hero-mobile.jpg 480w"
      sizes="100vw">
    <img src="hero-wide.jpg" alt="Hero image" width="1200" height="600">
  </picture>
</body>
</html>
```

**Expected output**: On wide screens, one of the wide candidates is selected based on viewport width; on narrow screens, the mobile candidate is selected.

**Why this output occurs**: Each `<source>` combines `media`, `srcset`, and `sizes` to describe a set of candidates for a specific media condition. The browser evaluates the media queries, then selects the best candidate from the matching source’s `srcset`.

---

### Real-World Cases

1. **Cross-format video**: Providing WebM and MP4 to maximise browser compatibility.
2. **Responsive hero images**: Delivering different image crops for mobile and desktop.
3. **Format negotiation**: Offering AVIF, WebP, and JPEG fallbacks.

---

### References

- MDN Web Docs — `<source>`: The Media or Image Source element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/source
- W3C — HTML 5.1: The `source` element — https://www.w3.org/TR/html51/semantics-embedded-content.html#the-source-element
- WHATWG — The `source` element — https://html.spec.whatwg.org/multipage/embedded-content.html#the-source-element

---

## 4. `<iframe>` — The Nested Browsing Context

### Definitions

**Core Definition**
The `<iframe>` element represents a nested browsing context, embedding another HTML page into the current document.

**Technical Definition**
`<iframe>` belongs to the **flow content**, **phrasing content**, **embedded content**, **interactive content**, and **palpable content** categories. Its content model is **nothing** — it is an empty element in the current standard. Key attributes include `src`, `srcdoc`, `sandbox`, `allow`, `allowfullscreen`, `width`, `height`, `referrerpolicy`, and `loading`. The `sandbox` attribute applies restrictions to the embedded content, and `referrerpolicy` controls the referrer sent when fetching the frame’s resource.

**Beginner-Friendly Explanation**
An `<iframe>` puts another web page inside your page. It is like a window that shows a different website. You can use it to embed a map, a video player, or another page.

---

### Purposes

- **To embed another HTML page** within the current document.
- **To isolate third-party content** in a separate browsing context.
- **To apply security restrictions** via `sandbox`, limiting what the embedded page can do.
- **To control referrer information** via `referrerpolicy`.
- **To defer loading** via `loading="lazy"` for off-screen frames.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<iframe
  src="https://example.com/embed"
  title="Description of embedded content"
  width="800"
  height="400"
  sandbox="allow-scripts allow-same-origin"
  referrerpolicy="no-referrer"
  loading="lazy">
</iframe>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the page to embed. |
| `srcdoc` | Inline HTML document to render instead of `src`. |
| `sandbox` | Security restrictions; empty value applies all restrictions. |
| `allow` | Permissions policy for the frame. |
| `referrerpolicy` | Referrer policy for the fetch. |
| `loading` | `lazy` or `eager`. |
| `title` | Accessible name for the frame. |

#### Syntax Rules

1. **`<iframe>` content model is nothing** in the current standard; fallback content is not permitted.
2. **`title` should always be provided** for accessibility.
3. **`sandbox` should be used for untrusted content**; an empty `sandbox` applies all restrictions.
4. **`loading="lazy"` defers loading** until the frame is near the viewport.

#### Constraints and Limitations

- Iframes create a separate browsing context, which can confuse screen-reader navigation.
- Each iframe loads a full document, increasing page weight and load time.
- The `srcdoc` attribute must contain a complete HTML document; it cannot contain fragment markup.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Sandboxed Embedded Page

**Setup guide**: Save as `iframe-sandbox.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sandboxed Iframe</title>
</head>
<body>
  <iframe
    src="https://example.com/widget"
    title="Example widget"
    width="600"
    height="300"
    sandbox="allow-scripts"
    referrerpolicy="no-referrer"
    loading="lazy">
  </iframe>
</body>
</html>
```

**Expected output**: The embedded page loads in a sandboxed context. Scripts are allowed (`allow-scripts`), but same-origin access and form submission are blocked by default. The referrer is omitted (`no-referrer`), and the frame is lazy-loaded.

**Why this output occurs**: The `sandbox` attribute applies restrictions. With only `allow-scripts`, the frame can run scripts but cannot access the parent’s origin, submit forms, or open popups. The `referrerpolicy` controls the `Referer` header.

---

#### Example 2: Inline HTML with `srcdoc`

**Setup guide**: Save as `iframe-srcdoc.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Srcdoc Iframe</title>
</head>
<body>
  <iframe
    srcdoc="<p>This content is defined inline.</p>"
    title="Inline content frame"
    width="400"
    height="100">
  </iframe>
</body>
</html>
```

**Expected output**: A frame displaying the inline paragraph “This content is defined inline.”

**Why this output occurs**: The `srcdoc` attribute contains a complete HTML document (simplified here), which the browser renders in the nested browsing context instead of fetching an external URL.

---

### Real-World Cases

1. **Embedded maps**: Google Maps and OpenStreetMap provide iframe embed codes.
2. **Video embeds**: YouTube and Vimeo players are commonly embedded with iframes.
3. **Third-party widgets**: Payment forms, chat widgets, and social media feeds are isolated in sandboxed iframes.

---

### References

- WHATWG — The `iframe` element — https://html.spec.whatwg.org/dev/iframe-embed-object.html
- MDN Web Docs — `<iframe>`: The Inline Frame element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/iframe
- W3C — HTML 5.2: The `iframe` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-iframe-element

---

## 5. `<embed>` — The External Plugin Content Element

### Definitions

**Core Definition**
The `<embed>` element embeds external content at the specified point in the document. It is a void element and is typically used for content that requires a plugin or external application.

**Technical Definition**
`<embed>` belongs to the **flow content**, **phrasing content**, **embedded content**, **interactive content**, and **palpable content** categories. It has a content model of “nothing” because it is void. Key attributes include `src`, `type`, `width`, and `height`. The element is intended for content that the browser cannot render natively, such as certain plugin-based formats.

**Beginner-Friendly Explanation**
The `<embed>` element puts external content into the page — content that the browser might need a plugin or special handler to display. It is less commonly used today, as native elements like `<video>` and `<iframe>` cover most needs.

---

### Purposes

- **To embed external, plugin-dependent content** such as certain document formats.
- **To provide a lightweight container** for content that does not fit other elements.
- **To pass dimensions** via `width` and `height`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<embed src="document.pdf" type="application/pdf" width="600" height="400">
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the external resource. |
| `type` | MIME type of the resource. |
| `width` / `height` | Dimensions of the embedded content. |

#### Syntax Rules

1. **`<embed>` is a void element**; no end tag, no children.
2. **`src` should point to a valid resource**.
3. **`type` helps the browser select the appropriate handler**.
4. **Fallback content is not supported**; use `<object>` if fallback is needed.

#### Constraints and Limitations

- `<embed>` is void, so it cannot contain fallback content.
- It is not suitable for content that has a native HTML element (e.g. video, audio).
- Plugin support has declined; many formats previously handled by plugins are now unsupported.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Embedding a PDF

**Setup guide**: Save as `embed-pdf.html`. Provide a PDF file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Embed PDF</title>
</head>
<body>
  <embed
    src="document.pdf"
    type="application/pdf"
    width="800"
    height="600">
</body>
</html>
```

**Expected output**: The PDF is displayed inside the page, if the browser supports inline PDF viewing.

**Why this output occurs**: The `src` points to the PDF, and `type` tells the browser the MIME type. The browser uses its built-in PDF viewer or a configured handler.

---

#### Example 2: Embedding with a Custom Type

**Setup guide**: Save as `embed-type.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Embed Custom Type</title>
</head>
<body>
  <embed
    src="interactive.swf"
    type="application/x-shockwave-flash"
    width="400"
    height="300">
</body>
</html>
```

**Expected output**: In modern browsers, this will likely fail to render because Flash support has been removed. In older environments, it would render the Flash content.

**Why this output occurs**: `<embed>` relies on browser support for the MIME type. When support is absent, nothing is displayed and no fallback is available.

---

### Real-World Cases

1. **Legacy document viewing**: Some systems embed PDFs or other documents with `<embed>`.
2. **Rarely used in modern development**: Most use cases have been replaced by `<iframe>`, `<video>`, `<audio>`, or `<object>`.

---

### References

- MDN Web Docs — `<embed>`: The External Embed element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/embed
- W3C — HTML 5.2: The `embed` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-embed-element
- WHATWG — The `embed` element — https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-embed-element

---

## 6. `<object>` — The External Resource Element

### Definitions

**Core Definition**
The `<object>` element represents an external resource, which can be treated as an image, a nested browsing context, or a resource to be handled by a plugin.

**Technical Definition**
`<object>` belongs to the **flow content**, **phrasing content**, **embedded content**, **interactive content**, and **palpable content** categories. Its content model is **transparent**, meaning it can contain fallback content that is rendered if the external resource cannot be displayed. Key attributes include `data` (the resource URL), `type` (MIME type), `width`, and `height`.

**Beginner-Friendly Explanation**
The `<object>` element is a flexible way to embed external content. Unlike `<embed>`, it can contain fallback content — HTML that is shown if the external resource cannot be loaded. It is useful for embedding PDFs, Flash content (historically), or other resources.

---

### Purposes

- **To embed an external resource** with fallback content.
- **To provide a nested browsing context** or plugin-hosted content.
- **To support graceful degradation** when the resource is unavailable.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<object
  data="resource.pdf"
  type="application/pdf"
  width="600"
  height="400">
  <p>Fallback content shown if the object cannot be displayed.</p>
</object>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `data` | URL of the external resource. |
| `type` | MIME type of the resource. |
| `width` / `height` | Dimensions. |
| Content | Fallback content, rendered if the resource fails. |

#### Syntax Rules

1. **`data` specifies the resource URL**.
2. **Content model is transparent**; fallback content is permitted.
3. **`type` helps the browser choose a handler**.
4. **Fallback content should be meaningful** and accessible.

#### Constraints and Limitations

- `<object>` is more complex than `<embed>` but offers fallback support.
- Support for various MIME types varies by browser.
- Deprecated `<param>` elements were historically used to pass parameters to plugins.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: PDF with Fallback

**Setup guide**: Save as `object-pdf.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Object PDF</title>
</head>
<body>
  <object
    data="document.pdf"
    type="application/pdf"
    width="800"
    height="600">
    <p>
      Your browser cannot display the PDF.
      <a href="document.pdf">Download the PDF</a> instead.
    </p>
  </object>
</body>
</html>
```

**Expected output**: The PDF is displayed if the browser supports inline PDF viewing; otherwise, the fallback paragraph and download link are shown.

**Why this output occurs**: The `data` attribute specifies the PDF, and the content inside `<object>` is the fallback. If the resource cannot be rendered, the fallback is displayed.

---

#### Example 2: Object with Plugin Parameters (Historical)

**Setup guide**: Save as `object-param.html`. This illustrates deprecated behaviour.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Object with Param</title>
</head>
<body>
  <object
    data="movie.swf"
    type="application/x-shockwave-flash"
    width="400"
    height="300">
    <param name="quality" value="high">
    <param name="bgcolor" value="#ffffff">
    <p>Flash content is no longer supported.</p>
  </object>
</body>
</html>
```

**Expected output**: In modern browsers, the Flash content is not rendered; the fallback paragraph is shown. In historical browsers with Flash support, the `<param>` elements would configure the plugin.

**Why this output occurs**: `<param>` elements pass parameters to plugins. With plugin support removed, the parameters have no effect, and the fallback content is used.

---

### Real-World Cases

1. **PDF embedding**: Displaying PDFs with fallback download links.
2. **Legacy plugin content**: Historical Flash or Java applet embedding.
3. **Interactive content**: Some specialised interactive documents.

---

### References

- MDN Web Docs — `<object>`: The External Object element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/object
- W3C — HTML 5.2: The `object` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-object-element
- WHATWG — The `object` element — https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-object-element

---

## 7. `<param>` — The Object Parameter Element (Deprecated)

### Definitions

**Core Definition**
The `<param>` element defines parameters for an `<object>` element. It is a void element and is **deprecated** in modern HTML.

**Technical Definition**
`<param>` historically belonged to the **metadata content** category when used inside `<object>`. It has a content model of “nothing” because it is void. Its two attributes — `name` and `value` — define a name-value pair passed to the plugin or external application invoked by `<object>`. MDN explicitly marks `<param>` as deprecated and advises against its use in new projects.

**Beginner-Friendly Explanation**
The `<param>` element used to pass settings to plugins. It is now obsolete because plugins are no longer supported in most browsers. Modern embedded content does not use `<param>`.

---

### Purposes

- **Historically: to pass parameters to a plugin** invoked by `<object>`.
- **Historically: to configure Flash, Java applets, or similar technologies.**
- **Modern guidance: do not use**; use native HTML elements and attributes instead.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<object data="applet.class" type="application/x-java-applet">
  <param name="code" value="MyApplet">
  <param name="archive" value="applet.jar">
</object>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `name` | Name of the parameter. |
| `value` | Value of the parameter. |

#### Syntax Rules

1. **`<param>` is a void element**.
2. **Both `name` and `value` must be present**.
3. **It must be a child of `<object>`**.

#### Constraints and Limitations

- **Deprecated**: Not recommended for new content.
- Plugin support has been removed from modern browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Historical Java Applet Configuration

**Setup guide**: Save as `param-historical.html`. This will not run in modern browsers.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Param Example</title>
</head>
<body>
  <object data="MyApplet.class" type="application/x-java-applet" width="300" height="200">
    <param name="code" value="MyApplet">
    <param name="archive" value="applet.jar">
    <p>Java applets are no longer supported.</p>
  </object>
</body>
</html>
```

**Expected output**: In modern browsers, the applet is not loaded, and the fallback paragraph is displayed. In historical browsers with Java plugin support, the applet would run with the specified parameters.

**Why this output occurs**: The `<param>` elements configure the Java plugin. Without plugin support, they are ignored and the fallback content is rendered.

---

### Real-World Cases

- **Legacy content only**: `<param>` appears in old HTML documents. It has no role in modern web development.

---

### References

- MDN Web Docs — `<param>`: The Object Parameter element (Deprecated) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/param
- W3C — HTML 5.2: The `param` element (obsolete) — https://www.w3.org/TR/html52/obsolete.html#the-param-element
- WHATWG — The `param` element (obsolete) — https://html.spec.whatwg.org/multipage/obsolete.html#the-param-element

---

## 8. `<video>` — The Video Playback Element

### Definitions

**Core Definition**
The `<video>` element embeds a media player that supports video playback. It can contain `<source>` elements and `<track>` elements for captions and subtitles.

**Technical Definition**
`<video>` belongs to the **flow content**, **phrasing content**, **embedded content**, **interactive content** (when `controls` is present), and **palpable content** categories. Its content model is zero or more `<source>` elements followed by zero or more `<track>` elements, followed by transparent fallback content. Key attributes include `src`, `poster`, `controls`, `autoplay`, `muted`, `loop`, `preload`, `playsinline`, and `crossorigin`.

**Beginner-Friendly Explanation**
The `<video>` element puts a video player on the page. You can give it one or more video files, and the browser picks one it can play. With `controls`, the browser shows play/pause and volume buttons. With `<track>`, you can add captions.

---

### Purposes

- **To play video content** natively in the browser.
- **To provide multiple video sources** via `<source>` for format compatibility.
- **To display captions and subtitles** via `<track>`.
- **To show a poster image** before playback via `poster`.
- **To support autoplay** (often muted) and looping.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<video controls width="640" poster="poster.jpg" preload="metadata">
  <source src="video.webm" type="video/webm">
  <source src="video.mp4" type="video/mp4">
  <track kind="captions" src="captions.vtt" srclang="en" label="English">
  Your browser does not support the video element.
</video>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the video resource (alternative to `<source>`). |
| `controls` | Shows browser playback controls. |
| `poster` | Image shown before playback. |
| `preload` | `none`, `metadata`, or `auto`. |
| `autoplay` / `muted` / `loop` | Playback behaviour. |
| `playsinline` | Prevents fullscreen playback on iOS. |

#### Syntax Rules

1. **Content model is `<source>` elements, then `<track>` elements, then fallback content**.
2. **`controls` should be present** unless custom controls are provided.
3. **`muted` is often required for `autoplay`** to work in modern browsers.
4. **`<track>` must come after `<source>` elements**.

#### Constraints and Limitations

- Autoplay with sound is often blocked; use `muted` if autoplay is essential.
- Video files can be large; use `preload="metadata"` and appropriate compression.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Video with Captions

**Setup guide**: Save as `video-captions.html`. Provide `movie.mp4` and `captions.vtt`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Video with Captions</title>
</head>
<body>
  <video controls width="800" poster="poster.jpg">
    <source src="movie.mp4" type="video/mp4">
    <track kind="captions" src="captions.vtt" srclang="en" label="English">
    Your browser does not support the video element.
  </video>
</body>
</html>
```

**Expected output**: A video player with controls and a poster image. Captions can be toggled on, and the WebVTT file supplies the text.

**Why this output occurs**: The `<video>` element renders the player. The `<source>` provides the video file, and the `<track>` supplies the timed text track for captions.

---

#### Example 2: Autoplaying Muted Background Video

**Setup guide**: Save as `video-autoplay.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Autoplay Video</title>
</head>
<body>
  <video
    autoplay
    muted
    loop
    playsinline
    width="100%"
    poster="background.jpg">
    <source src="background.webm" type="video/webm">
    <source src="background.mp4" type="video/mp4">
  </video>
</body>
</html>
```

**Expected output**: The video begins playing automatically, muted, and loops continuously. On iOS, `playsinline` prevents automatic fullscreen.

**Why this output occurs**: `autoplay` starts playback, `muted` satisfies browser autoplay policies, `loop` repeats the video, and `playsinline` keeps it inline on iOS.

---

### Real-World Cases

1. **News and media sites**: Embedding video reports with captions.
2. **Marketing pages**: Background hero videos with `autoplay muted loop`.
3. **Educational platforms**: Lecture recordings with subtitle tracks.

---

### References

- WHATWG — The `video` element — https://html.spec.whatwg.org/multipage/media.html#the-video-element
- MDN Web Docs — `<video>`: The Video Embed element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/video
- W3C — HTML 5.2: The `video` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-video-element

---

## 9. `<audio>` — The Audio Playback Element

### Definitions

**Core Definition**
The `<audio>` element embeds sound content in the document. It can contain `<source>` elements and `<track>` elements.

**Technical Definition**
`<audio>` belongs to the **flow content**, **phrasing content**, **embedded content**, **interactive content** (when `controls` is present), and **palpable content** categories. Its content model is zero or more `<source>` elements followed by zero or more `<track>` elements, followed by transparent fallback content. Key attributes are the same media attributes as `<video>`: `src`, `controls`, `autoplay`, `muted`, `loop`, and `preload`.

**Beginner-Friendly Explanation**
The `<audio>` element puts an audio player on the page — for music, podcasts, or sound effects. You can provide multiple audio files, and the browser picks one it can play.

---

### Purposes

- **To play audio content** natively in the browser.
- **To provide multiple audio sources** for format compatibility.
- **To offer captions or transcript tracks** via `<track>`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<audio controls preload="metadata">
  <source src="audio.ogg" type="audio/ogg">
  <source src="audio.mp3" type="audio/mpeg">
  Your browser does not support the audio element.
</audio>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the audio resource. |
| `controls` | Shows playback controls. |
| `preload` | `none`, `metadata`, or `auto`. |
| `autoplay` / `muted` / `loop` | Playback behaviour. |

#### Syntax Rules

1. **Content model is `<source>` then `<track>` then fallback content**.
2. **`controls` should be present** unless custom controls are used.
3. **`<source>` provides alternative formats**; the browser selects the first playable one.

#### Constraints and Limitations

- Audio cannot be visually perceived; transcripts are essential for accessibility.
- Autoplay may be blocked; `muted` may be required.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Audio with Multiple Formats

**Setup guide**: Save as `audio-formats.html`. Provide `sound.ogg` and `sound.mp3`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Audio Formats</title>
</head>
<body>
  <audio controls>
    <source src="sound.ogg" type="audio/ogg">
    <source src="sound.mp3" type="audio/mpeg">
    Your browser does not support the audio element.
  </audio>
</body>
</html>
```

**Expected output**: An audio player with controls. The browser plays `sound.ogg` if supported, otherwise `sound.mp3`.

**Why this output occurs**: The `<source>` elements provide candidate audio files. The browser selects the first one whose `type` it supports and whose `src` it can load.

---

#### Example 2: Podcast Episode with Transcript Track

**Setup guide**: Save as `audio-transcript.html`. Provide an audio file and a WebVTT transcript.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Audio with Transcript</title>
</head>
<body>
  <audio controls>
    <source src="episode.mp3" type="audio/mpeg">
    <track kind="subtitles" src="transcript.vtt" srclang="en" label="English transcript">
    Your browser does not support the audio element.
  </audio>
</body>
</html>
```

**Expected output**: An audio player. If the browser supports text tracks for audio, the transcript can be displayed; otherwise, the audio plays without the track.

**Why this output occurs**: The `<track>` element provides a timed text track for the audio. Support for text tracks on audio varies, but the track is valid and can be used by capable browsers.

---

### Real-World Cases

1. **Podcasts**: Embedding episodes with playback controls.
2. **Music previews**: Short audio samples on music sites.
3. **Accessibility**: Providing audio versions of text content with transcripts.

---

### References

- MDN Web Docs — `<audio>`: The Embed Audio element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/audio
- W3C — HTML 5.2: The `audio` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-audio-element
- WHATWG — The `audio` element — https://html.spec.whatwg.org/multipage/media.html#the-audio-element

---

## 10. `<track>` — The Timed Text Track Element

### Definitions

**Core Definition**
The `<track>` element is used as a child of `<audio>` and `<video>` elements to specify timed text tracks — such as subtitles, captions, descriptions, chapters, or metadata.

**Technical Definition**
`<track>` is a void element that belongs to the **phrasing content** category when used inside media elements. Its content model is “nothing”. Key attributes include `kind` (`subtitles`, `captions`, `descriptions`, `chapters`, `metadata`), `src` (URL of a WebVTT file), `srclang` (language of the track), `label` (user-visible title), and `default` (whether the track is enabled by default).

**Beginner-Friendly Explanation**
The `<track>` element adds text to a video or audio player — most commonly captions for deaf and hard-of-hearing users, or subtitles in another language. The text is provided in a separate file (WebVTT) with timestamps.

---

### Purposes

- **To provide closed captions** for accessibility.
- **To provide subtitles** for translation.
- **To provide audio descriptions** for visually impaired users.
- **To provide chapter markers** or metadata.
- **To support multiple languages** via different `<track>` elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<video controls>
  <source src="video.mp4" type="video/mp4">
  <track kind="captions" src="captions-en.vtt" srclang="en" label="English" default>
  <track kind="subtitles" src="subs-fr.vtt" srclang="fr" label="Français">
</video>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `kind` | Type of track: `subtitles`, `captions`, `descriptions`, `chapters`, `metadata`. |
| `src` | URL of the WebVTT file. |
| `srclang` | Language of the track (required for `subtitles`). |
| `label` | User-visible name of the track. |
| `default` | Enables the track by default. |

#### Syntax Rules

1. **`<track>` is a void element**.
2. **`kind` defaults to `subtitles`** if omitted.
3. **`srclang` is required when `kind="subtitles"`**.
4. **`default` should be used on at most one track** per media element.

#### Constraints and Limitations

- WebVTT is the required format for `<track>` files.
- Browser support for text tracks varies, but is generally good for video and captions.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Captions and Subtitles

**Setup guide**: Save as `track-captions.html`. Provide `captions-en.vtt` and `subs-fr.vtt`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Captions and Subtitles</title>
</head>
<body>
  <video controls width="640">
    <source src="video.mp4" type="video/mp4">

    <!-- Captions for accessibility, enabled by default -->
    <track kind="captions" src="captions-en.vtt" srclang="en" label="English captions" default>

    <!-- French subtitles, optional -->
    <track kind="subtitles" src="subs-fr.vtt" srclang="fr" label="Français">
  </video>
</body>
</html>
```

**Expected output**: A video player with English captions shown by default. Users can switch to French subtitles via the player’s menu.

**Why this output occurs**: The `<track>` elements provide timed text tracks. `kind="captions"` is intended for accessibility, and `default` enables it automatically. The second track provides an alternative language.

---

#### Example 2: Audio Descriptions

**Setup guide**: Save as `track-descriptions.html`. Provide a descriptions WebVTT file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Audio Descriptions</title>
</head>
<body>
  <video controls width="640">
    <source src="video.mp4" type="video/mp4">
    <track kind="descriptions" src="descriptions.vtt" srclang="en" label="Audio descriptions">
  </video>
</body>
</html>
```

**Expected output**: The video player offers an “Audio descriptions” track that narrates visual information for blind and low-vision users.

**Why this output occurs**: `kind="descriptions"` indicates that the track provides descriptions of visual content, which assistive technologies can use to convey the video’s visual information.

---

### Real-World Cases

1. **Accessible video players**: Captions for deaf and hard-of-hearing users.
2. **International content**: Subtitles in multiple languages.
3. **Educational media**: Transcripts and audio descriptions.

---

### References

- MDN Web Docs — `<track>`: The Embed Text Track element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/track
- W3C — HTML 5.2: The `track` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-track-element
- WHATWG — The `track` element — https://html.spec.whatwg.org/multipage/media.html#the-track-element

---

## 11. `<map>` — The Image Map Container

### Definitions

**Core Definition**
The `<map>` element, used with `<area>` elements, defines an image map — a set of clickable regions on an image that link to different destinations.

**Technical Definition**
`<map>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **transparent**, meaning its content model is derived from its parent. The `name` attribute is required and provides the name by which an `<img>` element’s `usemap` attribute references the map.

**Beginner-Friendly Explanation**
An image map makes different parts of a single image clickable. For example, a map of a country might have each province linking to a different page. The `<map>` element holds the clickable areas defined by `<area>` elements.

---

### Purposes

- **To define clickable regions on an image**.
- **To associate geometric areas with hyperlinks**.
- **To provide navigation without requiring separate images**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="map.png" alt="Interactive map" usemap="#regions">

<map name="regions">
  <area shape="rect" coords="0,0,100,50" href="/north" alt="North region">
  <area shape="rect" coords="100,0,200,50" href="/south" alt="South region">
</map>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<map>` | The container for image map areas. |
| `name` | **Required.** The name referenced by the image’s `usemap` attribute. |
| `<area>` | Defines a clickable region. |

#### Syntax Rules

1. **`name` is required and must be non-empty** with no whitespace.
2. **The value must match the `usemap` attribute** on the associated `<img>` (prefixed with `#`).
3. **Content model is transparent**; `<area>` elements are the typical children.

#### Constraints and Limitations

- Image maps are not always accessible; each `<area>` needs an `alt` attribute.
- Responsive image maps can be complex to implement; CSS positioning may be needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Image Map

**Setup guide**: Save as `map-basic.html`. Provide an image `map.png`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image Map</title>
</head>
<body>
  <img src="map.png" alt="Regional navigation map" usemap="#regions" width="400" height="100">

  <map name="regions">
    <area shape="rect" coords="0,0,100,100" href="/north" alt="North region">
    <area shape="rect" coords="100,0,200,100" href="/south" alt="South region">
    <area shape="rect" coords="200,0,300,100" href="/east" alt="East region">
    <area shape="rect" coords="300,0,400,100" href="/west" alt="West region">
  </map>
</body>
</html>
```

**Expected output**: The image is divided into four clickable rectangular regions, each linking to a different page.

**Why this output occurs**: The `<img>` `usemap` attribute references the `<map>` by name. Each `<area>` defines a shape and coordinates and links to a URL. The `alt` attributes provide accessible names for each region.

---

### Real-World Cases

1. **Geographic maps**: Clickable regions for countries or provinces.
2. **Product diagrams**: Clickable parts of a machine or anatomical diagram.
3. **Navigation menus**: Single-image menus with clickable zones.

---

### References

- WHATWG — The `map` element — https://html.spec.whatwg.org/multipage/image-maps.html#the-map-element
- MDN Web Docs — `<map>`: The Image Map element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/map
- W3C — HTML 5.2: The `map` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-map-element

---

## 12. `<area>` — The Image Map Area Element

### Definitions

**Core Definition**
The `<area>` element defines a clickable area inside an image map. It is a void element and must be a descendant of a `<map>` element.

**Technical Definition**
`<area>` belongs to the **flow content** and **phrasing content** categories when used inside `<map>`. Its content model is “nothing” because it is void. Key attributes include `shape` (`rect`, `circle`, `poly`, `default`), `coords` (coordinates for the shape), `href` (link target), `alt` (alternative text), and `target`.

**Beginner-Friendly Explanation**
The `<area>` element defines one clickable region on an image map. You specify its shape (rectangle, circle, or polygon) and coordinates, and where it should link to.

---

### Purposes

- **To define a clickable region** on an image map.
- **To provide a hyperlink** for a geometric area of an image.
- **To supply alternative text** for each clickable region.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<area shape="rect" coords="x1,y1,x2,y2" href="/target" alt="Description">
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `shape` | `rect`, `circle`, `poly`, or `default`. |
| `coords` | Coordinates for the shape. |
| `href` | Link target. |
| `alt` | Alternative text describing the area. |

#### Syntax Rules

1. **`<area>` is a void element**.
2. **`alt` is required** when `href` is present.
3. **`shape` and `coords` define the clickable geometry**.
4. **`<area>` must be a child of `<map>`**.

#### Constraints and Limitations

- Image maps are less common on responsive designs; alternative techniques may be needed.
- Coordinates are in CSS pixels and may need adjustment for responsive images.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Rectangular Area

**Setup guide**: Save as `area-rect.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Area Example</title>
</head>
<body>
  <img src="diagram.png" alt="Diagram of a system" usemap="#diagram-map" width="600" height="400">

  <map name="diagram-map">
    <area shape="rect" coords="0,0,200,200" href="/component-a" alt="Component A">
    <area shape="rect" coords="200,0,400,200" href="/component-b" alt="Component B">
  </map>
</body>
</html>
```

**Expected output**: The image has two clickable rectangular regions, each linking to a different component page.

**Why this output occurs**: Each `<area>` defines a rectangle via `coords` and links via `href`. The `alt` attributes provide accessible names for the regions.

---

#### Example 2: Circular Area

**Setup guide**: Save as `area-circle.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Circular Area</title>
</head>
<body>
  <img src="map.png" alt="Map with a city marker" usemap="#city-map" width="400" height="300">

  <map name="city-map">
    <area shape="circle" coords="200,150,50" href="/city" alt="The city centre">
  </map>
</body>
</html>
```

**Expected output**: A circular clickable region centred at (200,150) with a radius of 50 pixels.

**Why this output occurs**: `shape="circle"` and `coords="x,y,r"` define the circular area. The `href` and `alt` provide the link and description.

---

### Real-World Cases

1. **Interactive diagrams**: Clickable components in technical illustrations.
2. **Geographic maps**: Clickable regions for cities or countries.
3. **Navigation images**: Single-image navigation with multiple hot spots.

---

### References

- WHATWG — The `area` element — https://html.spec.whatwg.org/multipage/image-maps.html#the-area-element
- MDN Web Docs — `<area>`: The Image Map Area element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/area
- W3C — HTML 5.2: The `area` element — https://www.w3.org/TR/html52/semantics-embedded-content.html#the-area-element

---

## 13. `<canvas>` — The Scriptable Bitmap Surface

### Definitions

**Core Definition**
The `<canvas>` element provides a scriptable bitmap surface on which graphics can be drawn using the Canvas API or WebGL API.

**Technical Definition**
`<canvas>` belongs to the **flow content**, **phrasing content**, **embedded content**, and **palpable content** categories. Its content model is **transparent fallback content** that is displayed if the browser does not support `<canvas>`. The element itself has only two attributes — `width` and `height` — which default to 300 × 150 pixels. All drawing is performed by JavaScript via the `getContext()` method.

**Beginner-Friendly Explanation**
The `<canvas>` element is a blank drawing surface. You use JavaScript to draw shapes, images, and animations on it. It is like a digital whiteboard controlled by code.

---

### Purposes

- **To draw 2D graphics** with the Canvas API.
- **To render 3D scenes** with WebGL.
- **To create animations and interactive visualisations**.
- **To generate dynamic images** that cannot be expressed as static files.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<canvas id="myCanvas" width="400" height="300">
  Fallback content for browsers without canvas support.
</canvas>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<canvas>` | The drawing surface. |
| `width` / `height` | Dimensions in CSS pixels; default 300 × 150. |
| Fallback content | Displayed if the browser does not support canvas. |

#### Syntax Rules

1. **Set `width` and `height` attributes** to define the drawing surface size.
2. **Use JavaScript `getContext('2d')`** to obtain a drawing context.
3. **Fallback content is permitted** and should be meaningful.

#### Constraints and Limitations

- Canvas content is not accessible to screen readers unless alternative text or ARIA is provided.
- Canvas graphics are raster-based and do not scale as cleanly as SVG.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Drawing a Rectangle

**Setup guide**: Save as `canvas-rect.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Canvas Rectangle</title>
</head>
<body>
  <canvas id="myCanvas" width="300" height="200">
    Your browser does not support the canvas element.
  </canvas>

  <script>
    // Get the canvas element
    const canvas = document.getElementById('myCanvas');
    // Get the 2D drawing context
    const ctx = canvas.getContext('2d');
    // Set fill colour
    ctx.fillStyle = '#0077cc';
    // Draw a filled rectangle at (50, 50) with width 200 and height 100
    ctx.fillRect(50, 50, 200, 100);
  </script>
</body>
</html>
```

**Expected output**: A blue rectangle drawn on the canvas.

**Why this output occurs**: The JavaScript obtains the 2D context, sets the fill colour, and calls `fillRect()` to draw a filled rectangle at the specified coordinates.

---

#### Example 2: Drawing Text

**Setup guide**: Save as `canvas-text.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Canvas Text</title>
</head>
<body>
  <canvas id="textCanvas" width="400" height="100">
    Canvas not supported.
  </canvas>

  <script>
    const canvas = document.getElementById('textCanvas');
    const ctx = canvas.getContext('2d');
    ctx.font = '24px sans-serif';
    ctx.fillStyle = '#333';
    ctx.fillText('Hello, Canvas!', 50, 60);
  </script>
</body>
</html>
```

**Expected output**: The text “Hello, Canvas!” rendered on the canvas in 24px sans-serif.

**Why this output occurs**: The context’s `font` and `fillStyle` are set, and `fillText()` draws the text at the specified coordinates.

---

### Real-World Cases

1. **Data visualisation**: Charts and graphs drawn with libraries like Chart.js.
2. **Games**: 2D and 3D browser games.
3. **Image editing**: Client-side photo filters and cropping tools.

---

### References

- MDN Web Docs — `<canvas>`: The Graphics Canvas element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/canvas
- MDN Web Docs — Canvas API — https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- WHATWG — The `canvas` element — https://html.spec.whatwg.org/multipage/canvas.html#the-canvas-element

---

## 14. `<svg>` — Inline Scalable Vector Graphics

### Definitions

**Core Definition**
The `<svg>` element is the root container for Scalable Vector Graphics (SVG) content embedded directly in an HTML document. SVG is an XML-based language for describing two-dimensional vector graphics.

**Technical Definition**
When `<svg>` appears in an HTML document, it is a foreign-namespace element belonging to the SVG namespace `http://www.w3.org/2000/svg`. It is treated as **embedded content**, **phrasing content**, **flow content**, and **palpable content** for the purposes of HTML content models. SVG defines its own element set — `<circle>`, `<rect>`, `<path>`, `<polygon>`, and many others — and its own attributes for coordinates, styling, and transformation.

**Beginner-Friendly Explanation**
The `<svg>` element lets you draw vector graphics directly in HTML. Unlike images, vector graphics can scale to any size without losing quality. You use SVG elements like `<circle>` and `<rect>` to describe shapes with code.

---

### Purposes

- **To embed vector graphics** that scale without pixelation.
- **To create icons and logos** that remain sharp at any size.
- **To draw complex illustrations** with paths and shapes.
- **To animate graphics** with CSS or SMIL.
- **To provide accessible graphics** with `<title>` and `<desc>`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<svg width="200" height="200" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
  <circle cx="100" cy="100" r="80" fill="#0077cc" />
  <text x="100" y="110" text-anchor="middle" fill="white">SVG</text>
</svg>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<svg>` | The SVG root container. |
| `width` / `height` | Display dimensions. |
| `viewBox` | Coordinate system definition. |
| SVG child elements | Shapes, paths, text, and other SVG content. |

#### Syntax Rules

1. **`xmlns` is optional in HTML** but required in XML documents.
2. **SVG elements use their own namespace** and follow SVG syntax rules.
3. **`viewBox` defines the internal coordinate system**.
4. **SVG content can be styled with CSS** and scripted with JavaScript.

#### Constraints and Limitations

- SVG is XML-based, so syntax must be well-formed.
- Complex SVG can increase DOM size and rendering cost.
- Accessibility requires `<title>` and `<desc>` elements or ARIA attributes.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple SVG Circle

**Setup guide**: Save as `svg-circle.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SVG Circle</title>
</head>
<body>
  <svg width="200" height="200" viewBox="0 0 200 200">
    <circle cx="100" cy="100" r="80" fill="#e63946" />
  </svg>
</body>
</html>
```

**Expected output**: A red circle with radius 80 centred in a 200 × 200 area.

**Why this output occurs**: The `<circle>` element uses `cx`, `cy`, and `r` to define its geometry and `fill` to set its colour. The `viewBox` ensures the coordinate system maps to the display size.

---

#### Example 2: SVG with Accessible Title

**Setup guide**: Save as `svg-accessible.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible SVG</title>
</head>
<body>
  <svg width="100" height="100" viewBox="0 0 100 100" role="img" aria-labelledby="svg-title">
    <title id="svg-title">A blue square</title>
    <rect x="10" y="10" width="80" height="80" fill="#0077cc" />
  </svg>
</body>
</html>
```

**Expected output**: A blue square. Screen readers announce “A blue square” because of the `<title>` element and `aria-labelledby`.

**Why this output occurs**: The `<title>` element provides an accessible name, and the `role="img"` and `aria-labelledby` attributes ensure that assistive technologies treat the SVG as a labelled image.

---

### Real-World Cases

1. **Icons**: SVG icons that scale with text size.
2. **Logos**: Crisp logos at any resolution.
3. **Data visualisations**: Charts and diagrams that remain sharp when zoomed.

---

### References

- MDN Web Docs — SVG in HTML — https://developer.mozilla.org/en-US/docs/Web/SVG/Guides/SVG_in_HTML
- MDN Web Docs — `<svg>` — https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Element/svg
- W3C — SVG Specification — https://www.w3.org/TR/SVG2/

---

## 15. `<math>` — Inline MathML

### Definitions

**Core Definition**
The `<math>` element is the top-level MathML element, used to write a single mathematical formula. MathML is an XML-based language for describing mathematical notation.

**Technical Definition**
When `<math>` appears in an HTML document, it is a foreign-namespace element belonging to the MathML namespace `http://www.w3.org/1998/Math/MathML`. It is treated as **embedded content**, **phrasing content**, **flow content**, and **palpable content** for HTML content-model purposes. MathML token elements such as `<mi>`, `<mo>`, `<mn>`, `<ms>`, and `<mtext>` can contain HTML content when they are descendants of HTML elements.

**Beginner-Friendly Explanation**
The `<math>` element lets you write mathematical formulas directly in HTML. Instead of images of equations, you write the formula with MathML tags, and the browser renders it natively.

---

### Purposes

- **To embed mathematical formulas** in web pages.
- **To provide accessible, scalable math** that screen readers can interpret.
- **To avoid images of equations**, which are not accessible or scalable.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<math>
  <mi>x</mi>
  <mo>=</mo>
  <mfrac>
    <mrow><mo>−</mo><mi>b</mi><mo>±</mo><msqrt><msup><mi>b</mi><mn>2</mn></msup><mo>−</mo><mn>4</mn><mi>a</mi><mi>c</mi></msqrt></mrow>
    <mrow><mn>2</mn><mi>a</mi></mrow>
  </mfrac>
</math>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<math>` | The MathML root. |
| `<mi>` | Identifier (variable). |
| `<mo>` | Operator. |
| `<mn>` | Number. |
| `<mfrac>` | Fraction. |
| `<msqrt>` | Square root. |
| `<msup>` | Superscript. |

#### Syntax Rules

1. **`xmlns` is optional in HTML** but required in XML documents.
2. **MathML elements follow MathML syntax rules**.
3. **The `<math>` element must not be nested** inside another `<math>`.
4. **MathML content can be styled with CSS** in supporting browsers.

#### Constraints and Limitations

- Browser support for MathML has historically varied, though it is improving.
- Complex formulas may require careful structuring for accessibility.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Equation

**Setup guide**: Save as `math-simple.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>MathML Equation</title>
</head>
<body>
  <p>The quadratic formula is:</p>
  <math>
    <mi>x</mi>
    <mo>=</mo>
    <mfrac>
      <mrow>
        <mo>−</mo><mi>b</mi><mo>±</mo>
        <msqrt>
          <msup><mi>b</mi><mn>2</mn></msup>
          <mo>−</mo><mn>4</mn><mi>a</mi><mi>c</mi>
        </msqrt>
      </mrow>
      <mrow><mn>2</mn><mi>a</mi></mrow>
    </mfrac>
  </math>
</body>
</html>
```

**Expected output**: The quadratic formula rendered as a mathematical expression with a fraction and square root.

**Why this output occurs**: The MathML elements describe the structure of the formula. The browser renders it according to MathML layout rules.

---

#### Example 2: Simple Inline Math

**Setup guide**: Save as `math-inline.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inline Math</title>
</head>
<body>
  <p>The equation <math><mi>E</mi><mo>=</mo><mi>m</mi><msup><mi>c</mi><mn>2</mn></msup></math> is famous.</p>
</body>
</html>
```

**Expected output**: The equation “E = mc²” is rendered inline within the paragraph.

**Why this output occurs**: The `<math>` element is embedded inline, and the MathML elements describe the equation. The browser renders it in the text flow.

---

### Real-World Cases

1. **Educational content**: Textbooks and online courses with formulas.
2. **Scientific publishing**: Research papers with complex equations.
3. **Technical documentation**: Mathematical notation in API docs.

---

### References

- MDN Web Docs — `<math>` — https://developer.mozilla.org/en-US/docs/Web/MathML/Reference/Element/math
- MDN Web Docs — MathML — https://developer.mozilla.org/en-US/docs/Web/MathML
- W3C — MathML Core — https://www.w3.org/TR/mathml-core/

---

## 16. Embedded Content Best Practices

### Definitions

**Core Definition**
Embedded content best practices are authoring guidelines for selecting, embedding, and configuring external resources so that they are accessible, performant, secure, and maintainable.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG accessibility guidelines, and performance and security guidance. They encompass providing alternative text and captions, controlling iframe capabilities with `sandbox`, using lazy loading, optimising media resources, and preferring native HTML elements over plugin-based embedding.

**Beginner-Friendly Explanation**
Best practices are the “good habits” for embedding content. They ensure that images are described, videos have captions, iframes are safe, and the page still loads quickly.

---

### Purposes

- **To maximise accessibility** through `alt` text, captions, iframe `title`, and fallback content.
- **To minimise performance impact** by lazy-loading embedded resources and avoiding unnecessary embeds.
- **To improve security** by sandboxing untrusted iframes and controlling referrer information.
- **To ensure graceful degradation** with fallback content and progressive enhancement.
- **To comply with legal requirements** such as WCAG and accessibility legislation.

---

### Syntax Rules and Structure

Best practices are applied across elements. The following table summarises the core rules.

| Practice | Rule |
|---|---|
| **Always provide `alt` for images** | Use descriptive `alt` text; use `alt=""` for decorative images. |
| **Provide captions and transcripts for media** | Use `<track kind="captions">` for video and provide transcripts for audio. |
| **Give iframes a `title`** | The `title` attribute provides an accessible name for the frame. |
| **Use `sandbox` for untrusted iframes** | Apply the least privilege necessary. |
| **Lazy-load off-screen embeds** | Use `loading="lazy"` on `<img>` and `<iframe>`. |
| **Prefer native elements over plugins** | Use `<video>`, `<audio>`, and `<canvas>` instead of `<embed>` or plugin-based `<object>`. |
| **Use `<picture>` for responsive images** | Provide alternative formats and art-directed crops. |
| **Label SVG and canvas** | Provide `<title>` or `aria-labelledby` for informative graphics. |
| **Avoid embedding entire third-party pages** | Use embeds sparingly and avoid heavy third-party content. |
| **Provide fallback content** | Use `<object>` or fallback children where supported. |

#### Syntax Rules

1. **Accessibility first**: Every non-decorative embedded resource must have a text alternative.
2. **Performance awareness**: Embed only what is needed, and defer off-screen content.
3. **Security by default**: Sandbox iframes and control referrer policies.
4. **Graceful degradation**: Provide fallback content for unsupported features.

#### Constraints and Limitations

- Some embedded content (e.g. third-party widgets) may not be fully controllable; choose providers carefully.
- Performance budgets may require limiting the number or size of embeds.
- Accessibility requirements vary by jurisdiction; WCAG is a widely accepted baseline.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible and Performant Media Embed

**Setup guide**: Save as `best-practice-media.html`. Provide video, captions, and poster files.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Media</title>
</head>
<body>
  <figure>
    <video controls width="640" poster="poster.jpg" preload="metadata">
      <source src="video.mp4" type="video/mp4">
      <track kind="captions" src="captions-en.vtt" srclang="en" label="English" default>
      Your browser does not support the video element.
    </video>
    <figcaption>Video with English captions and a poster image.</figcaption>
  </figure>
</body>
</html>
```

**Expected output**: A video player with controls, a poster image before playback, and captions enabled by default. The fallback text is available if the video element is unsupported.

**Why this output occurs**: The `<video>` element provides native playback, `<track>` provides captions for accessibility, `poster` improves perceived performance, and `preload="metadata"` avoids loading the entire video until needed.

---

#### Example 2: Sandboxed and Lazy-Loaded Iframe

**Setup guide**: Save as `best-practice-iframe.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Iframe</title>
</head>
<body>
  <iframe
    src="https://example.com/widget"
    title="Example interactive widget"
    width="600"
    height="300"
    sandbox="allow-scripts"
    referrerpolicy="no-referrer"
    loading="lazy">
  </iframe>
</body>
</html>
```

**Expected output**: The widget loads only when it is near the viewport (lazy loading), runs with restricted capabilities (sandbox), and sends no referrer.

**Why this output occurs**: `loading="lazy"` defers loading, `sandbox="allow-scripts"` permits scripts but restricts other capabilities, and `referrerpolicy="no-referrer"` protects privacy.

---

### Real-World Cases

1. **Government websites**: Mandatory WCAG compliance drives alt text, captions, and iframe titles.
2. **Performance-critical pages**: Lazy loading and responsive images reduce page weight.
3. **Security-sensitive applications**: Sandboxed third-party embeds prevent privilege escalation.

---

### References

- W3C — WCAG 2.2: Info and Relationships — https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships
- MDN Web Docs — Embedded content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#embedded_content
- web.dev — Lazy loading — https://web.dev/articles/lazy-loading
- OWASP — Iframe security — https://cheatsheetseries.owasp.org/cheatsheets/Iframe_Embedding_Protection_Cheat_Sheet.html
- W3C — Media Accessibility — https://www.w3.org/WAI/media/av/

---

## Summary Table of Embedded Content Elements

| Element | Primary Role | Content Model | Key Attributes |
|---|---|---|---|
| `<img>` | Embedded image | Nothing (void) | `src`, `alt`, `srcset`, `sizes` |
| `<picture>` | Responsive image container | `<source>`* + `<img>` | global |
| `<source>` | Media resource candidate | Nothing (void) | `src`, `type`, `srcset`, `media` |
| `<iframe>` | Nested browsing context | Nothing | `src`, `sandbox`, `title`, `loading` |
| `<embed>` | External plugin content | Nothing (void) | `src`, `type` |
| `<object>` | Generic external resource | Transparent | `data`, `type` |
| `<param>` | Object parameter (deprecated) | Nothing (void) | `name`, `value` |
| `<video>` | Video playback | `<source>`* + `<track>`* + fallback | `controls`, `poster`, `autoplay` |
| `<audio>` | Audio playback | `<source>`* + `<track>`* + fallback | `controls`, `autoplay` |
| `<track>` | Timed text track | Nothing (void) | `kind`, `src`, `srclang` |
| `<map>` | Image map container | Transparent | `name` |
| `<area>` | Image map clickable area | Nothing (void) | `shape`, `coords`, `href`, `alt` |
| `<canvas>` | Scriptable bitmap surface | Transparent fallback | `width`, `height` |
| `<svg>` | Inline vector graphics | SVG content | `width`, `height`, `viewBox` |
| `<math>` | Inline MathML | MathML content | global |

---

## Final Notes on Deprecated, Obsolete, and Implementation-Dependent Features

- **`<param>`**: Deprecated; plugin support has been removed from modern browsers.
- **`<embed>` for plugin content**: Largely obsolete; use native elements or `<object>` with fallback.
- **Flash and Java applets**: No longer supported; content using these technologies will not run.
- **`<iframe>` fallback content**: The current standard defines `<iframe>` content model as “nothing”; older practices of including fallback content inside iframes are non-conforming.
- **Autoplay with sound**: Blocked by most browsers; use `muted` if autoplay is required.
- **SVG and MathML namespace declarations**: Optional in HTML but required in XML documents.

---

This cheat sheet has covered the embedded content elements — `<img>`, `<picture>`, `<source>`, `<iframe>`, `<embed>`, `<object>`, `<param>`, `<video>`, `<audio>`, `<track>`, `<map>`, `<area>`, `<canvas>`, `<svg>`, and `<math>` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate embedded content against the WHATWG HTML Living Standard, WCAG accessibility guidelines, and performance and security best practices.