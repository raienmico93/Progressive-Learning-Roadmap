# Image Formats — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Image formats are the file specifications and encoding methods used to store, compress, and deliver visual information on the web. Each format balances trade-offs between file size, visual quality, transparency support, animation capability, colour depth, and browser compatibility.

**Technical Definition**
An image format defines the bitstream structure, compression algorithm, colour model, metadata, and container for raster or vector graphics. Raster formats (JPEG, PNG, GIF, WebP, AVIF) encode a grid of pixels; vector formats (SVG) encode geometric primitives and drawing instructions. The choice of format determines the achievable compression ratio, whether transparency (alpha channel) is supported, whether animation is possible, the maximum colour depth, and the degree of browser support. Modern formats such as WebP and AVIF offer superior compression compared to legacy JPEG and PNG, but require fallbacks for older browsers.

**Beginner-Friendly Explanation**
An image format is the type of file your picture is saved as. Just as a document can be saved as a PDF or a Word file, a picture can be saved as a JPEG, PNG, GIF, WebP, or another format. Each format has its own strengths: some make files very small, some keep backgrounds transparent, some support animation, and some can be scaled without blurring. Choosing the right format makes your website faster and your images look better.

---

### Key Characteristics

- **Compression trade-off**: Lossy formats (JPEG, WebP, AVIF) discard visual data to achieve smaller files; lossless formats (PNG, GIF) preserve all pixel data but produce larger files.
- **Transparency support**: PNG, WebP, AVIF, and SVG support full alpha transparency; GIF supports only binary transparency; JPEG supports none.
- **Animation support**: GIF and APNG support animation; WebP and AVIF support animated sequences with far better compression than GIF.
- **Colour depth**: JPEG and PNG top out at 24-bit (16.7 million colours); GIF is limited to 256 colours; AVIF supports up to 12-bit and HDR.
- **Vector vs raster**: SVG is resolution-independent and ideal for icons, logos, and diagrams; raster formats blur when scaled beyond their intrinsic dimensions.
- **Browser support gradient**: JPEG and PNG have universal support; WebP has ~96% support; AVIF has ~93% support; HEIC is supported only in Safari.

---

### Prerequisites

1. **Basic HTML image syntax**: familiarity with `<img src alt>`.
2. **The concept of compression**: understanding the difference between lossy and lossless compression.
3. **Browser compatibility awareness**: knowing that not all formats work in all browsers.
4. **Awareness of responsive images**: understanding that different devices require different image sizes.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Web performance (Core Web Vitals)** | Modern formats reduce image payload, improving LCP and reducing bandwidth. |
| **Responsive images** | `srcset`, `sizes`, and `<picture>` enable format negotiation and resolution switching. |
| **Accessibility (WCAG)** | `alt` text remains mandatory regardless of format. |
| **CDN image transformation** | Services like Cloudinary and Imgix automate format selection and conversion. |
| **Print and archival** | TIFF and high-bit-depth formats preserve fidelity for non-web use. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **JPEG** — the photography workhorse
2. **PNG** — the lossless, transparent graphics format
3. **GIF** — the legacy animation format
4. **WebP** — the modern universal format
5. **AVIF** — the next-generation compression format
6. **SVG** — the vector graphics format
7. **HEIC** — the Apple ecosystem format
8. **Format selection** — choosing the right format
9. **Format best practices** — consolidated guidelines

---

## 1. JPEG — The Photography Workhorse

### Definitions

**Core Definition**
JPEG (Joint Photographic Experts Group) is a lossy raster image format designed for compressing photographic images. It has been the web's default image format since the early 1990s and remains universally supported.

**Technical Definition**
JPEG uses a discrete cosine transform (DCT) and quantisation to discard high-frequency visual information that the human eye is less sensitive to. It supports 24-bit colour (8 bits per channel) but no alpha transparency and no animation. The MIME type is `image/jpeg`; common extensions include `.jpg`, `.jpeg`, `.jfif`, `.pjpeg`, and `.pjp`. HTTP Archive data shows JPEG still accounts for about a third of all images online.

**Beginner-Friendly Explanation**
JPEG is the format you use for photographs. It makes photo files much smaller by throwing away tiny details that you probably will not notice. It does not support transparent backgrounds, and it cannot animate.

---

### Purposes

- **To compress photographs** to small file sizes with acceptable visual quality.
- **To provide a universal fallback** that works in every browser and email client.
- **To serve as the legacy baseline** when modern formats are not supported.
- **To deliver product shots, portraits, landscapes, and blog images**.

---

### Syntax Rules and Structure

#### Complete General Syntax (HTML Embedding)

```html
<img src="photo.jpg" alt="A mountain landscape at sunrise" width="1200" height="800">
```

**Component breakdown**

| Component | Description |
|---|---|
| `src="photo.jpg"` | The JPEG file. |
| `alt="…"` | Textual alternative (required for accessibility). |
| `width` / `height` | Intrinsic dimensions; prevent layout shift. |

#### MIME Type and Extensions

| Property | Value |
|---|---|
| MIME type | `image/jpeg` |
| Extensions | `.jpg`, `.jpeg`, `.jfif`, `.pjpeg`, `.pjp` |

#### Syntax Rules

1. **Use `image/jpeg` as the MIME type** in HTTP headers and `<source type>`.
2. **Quality settings of 75–85** offer the best size-quality balance; below 60 artifacts become visible.
3. **Do not re-save JPEGs repeatedly**; each save degrades quality further.

#### Constraints and Limitations

- **No transparency**: JPEG cannot store alpha channel data; backgrounds are always opaque.
- **No animation**: JPEG is a still-image format only.
- **Sharp edges suffer**: Text and graphics with hard edges become blurred and artifacted.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Standard JPEG with Dimensions

**Setup Guide**: Save as `jpeg-basic.html`. Provide a JPEG file named `photo.jpg`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>JPEG Example</title>
</head>
<body>
  <!-- JPEG for photographic content; dimensions prevent CLS -->
  <img src="photo.jpg" alt="A mountain landscape at sunrise" width="1200" height="800">
</body>
</html>
```

**Expected output**: The JPEG image is displayed at 1200 × 800 CSS pixels, and the browser reserves space before the image loads.

**Why this output occurs**: The `src` attribute loads the JPEG, and the `width`/`height` attributes provide the intrinsic aspect ratio so the layout does not shift.

---

#### Example 2: JPEG as a `<picture>` Fallback

**Setup Guide**: Save as `jpeg-fallback.html`. Provide AVIF, WebP, and JPEG versions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>JPEG Fallback</title>
</head>
<body>
  <picture>
    <source type="image/avif" srcset="photo.avif">
    <source type="image/webp" srcset="photo.webp">
    <!-- JPEG fallback for browsers that support neither AVIF nor WebP -->
    <img src="photo.jpg" alt="A city skyline at dusk" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: Modern browsers load the AVIF or WebP version; older browsers fall back to the JPEG.

**Why this output occurs**: The browser evaluates each `<source>` in order and selects the first format it supports; the `<img>` provides the universal fallback.

---

### Real-World Cases

1. **News and editorial photography**: JPEG remains the standard for photographic content.
2. **Email newsletters**: JPEG works in every email client.
3. **User-uploaded photos**: Platforms accept JPEG universally.

---

### References

- MDN Web Docs — Image file type and format guide: JPEG — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#jpeg_joint_photographic_experts_group_image
- ShortPixel — Beginner's Guide to Image File Formats — https://shortpixel.com/blog/beginners-guide-to-image-file-formats-jpeg-png-gif-svg-webp-avif/
- Cloudinary — Finding the Best Image Format — https://cloudinary.com/guides/image-formats/best-image-format

---

## 2. PNG — The Lossless, Transparent Graphics Format

### Definitions

**Core Definition**
PNG (Portable Network Graphics) is a lossless raster image format designed for sharp graphics, screenshots, and images requiring transparency. It uses DEFLATE compression and supports full alpha channel transparency.

**Technical Definition**
PNG supports indexed colour (1–8 bits), grayscale (1–16 bits), truecolour (8–16 bits per channel), and an optional alpha channel for full or partial transparency. It uses lossless DEFLATE compression, meaning the decoded image is pixel-identical to the source. The MIME type is `image/png`; the extension is `.png`. PNG is universally supported in all browsers.

**Beginner-Friendly Explanation**
PNG is the format you use when you need sharp edges or a transparent background. Screenshots, logos, and icons are often PNG. The downside is that PNG files are usually much larger than JPEG files.

---

### Purposes

- **To preserve exact pixel data** for screenshots, UI graphics, and text-heavy images.
- **To support full alpha transparency** for logos and overlays.
- **To serve as a lossless alternative to GIF** for static graphics.
- **To provide a fallback** for browsers that do not support WebP or AVIF.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="logo.png" alt="Example Company logo" width="240" height="80">
```

**Component breakdown**

| Component | Description |
|---|---|
| `src="logo.png"` | The PNG file. |
| `alt="…"` | Textual alternative. |
| `width` / `height` | Intrinsic dimensions. |

#### MIME Type and Extensions

| Property | Value |
|---|---|
| MIME type | `image/png` |
| Extension | `.png` |

#### Syntax Rules

1. **Use `image/png` as the MIME type**.
2. **Use PNG when sharp edges and transparency are required**; for photographs, JPEG or WebP is smaller.
3. **Use WebP or AVIF instead of PNG** when possible for better compression.

#### Constraints and Limitations

- **Large file sizes**: Lossless compression produces much larger files than JPEG for photographic content.
- **No animation** (unless using the APNG extension, which has broader support than GIF but is less common).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Transparent PNG Logo

**Setup Guide**: Save as `png-logo.html`. Provide a PNG with a transparent background.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>PNG Logo</title>
</head>
<body>
  <!-- PNG supports alpha transparency for logos -->
  <img src="logo.png" alt="Example Company logo" width="240" height="80">
</body>
</html>
```

**Expected output**: The logo displays with a transparent background, blending cleanly with any page background.

**Why this output occurs**: PNG's alpha channel preserves the transparency, and the `width`/`height` attributes reserve layout space.

---

#### Example 2: PNG as a Screenshot Format

**Setup Guide**: Save as `png-screenshot.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>PNG Screenshot</title>
</head>
<body>
  <figure>
    <img src="screenshot.png" alt="Screenshot of the settings panel" width="800" height="600">
    <figcaption>Figure 1: The settings panel.</figcaption>
  </figure>
</body>
</html>
```

**Expected output**: The screenshot displays with sharp text and UI elements, thanks to PNG's lossless compression.

**Why this output occurs**: PNG preserves every pixel, so text and hard edges remain crisp, unlike JPEG.

---

### Real-World Cases

1. **UI screenshots**: Documentation and tutorials use PNG for crisp screenshots.
2. **Logos and icons**: Transparent PNGs overlay cleanly on any background.
3. **Charts and diagrams**: PNG preserves sharp lines and text.

---

### References

- MDN Web Docs — Image file type and format guide: PNG — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#png_portable_network_graphics
- W3C — PNG Specification — https://www.w3.org/TR/PNG

---

## 3. GIF — The Legacy Animation Format

### Definitions

**Core Definition**
GIF (Graphics Interchange Format) is a lossless raster format that supports simple animations and binary transparency, but is limited to a 256-colour palette.

**Technical Definition**
GIF uses LZW lossless compression and an 8-bit indexed colour model, meaning each image can display at most 256 colours. It supports binary transparency (a pixel is either fully transparent or fully opaque) and simple frame-based animation. The MIME type is `image/gif`; the extension is `.gif`. GIF is universally supported but is increasingly replaced by video formats for animation and by WebP/AVIF for static images.

**Beginner-Friendly Explanation**
GIF is the old format for simple animations and memes. It is limited to 256 colours, so photos look bad, and the files are often very large. For animations, video is almost always a better choice.

---

### Purposes

- **To provide simple animations** where video is not feasible.
- **To serve as a universal fallback** for animated content.
- **To display simple graphics with binary transparency**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="animation.gif" alt="Loading animation" width="200" height="200">
```

**Component breakdown**

| Component | Description |
|---|---|
| `src="animation.gif"` | The GIF file. |
| `alt="…"` | Textual alternative. |

#### MIME Type and Extensions

| Property | Value |
|---|---|
| MIME type | `image/gif` |
| Extension | `.gif` |

#### Syntax Rules

1. **Use `image/gif` as the MIME type**.
2. **Prefer video formats (WebM/MP4) for animation**; GIF is inefficient.
3. **Use WebP or AVIF for animated sequences** where supported; they offer far better compression.

#### Constraints and Limitations

- **256-colour limit**: Photographs and gradients look terrible as GIFs.
- **Binary transparency only**: No partial transparency.
- **Large file sizes for animation**: GIF animations are often much larger than equivalent videos.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple GIF Animation

**Setup Guide**: Save as `gif-animation.html`. Provide a GIF file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GIF Animation</title>
</head>
<body>
  <img src="loading.gif" alt="Loading spinner" width="64" height="64">
</body>
</html>
```

**Expected output**: The GIF animation plays automatically.

**Why this output occurs**: The browser plays the animated GIF frames in sequence.

---

#### Example 2: Replacing GIF with Video

**Setup Guide**: Save as `gif-to-video.html`. Provide WebM and MP4 versions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GIF to Video</title>
</head>
<body>
  <!-- Video is far more efficient than GIF for animation -->
  <video autoplay loop muted playsinline width="200" height="200">
    <source src="loading.webm" type="video/webm">
    <source src="loading.mp4" type="video/mp4">
    Your browser does not support the video element.
  </video>
</body>
</html>
```

**Expected output**: The animation plays as a video, with dramatically smaller file size than the GIF equivalent.

**Why this output occurs**: Video codecs compress animation far more efficiently than GIF's frame-by-frame LZW encoding.

---

### Real-World Cases

1. **Legacy memes and reactions**: GIF remains culturally ubiquitous.
2. **Loading spinners**: Simple animations where video is overkill.
3. **Email animations**: Some email clients block video but allow GIF.

---

### References

- MDN Web Docs — Image file type and format guide: GIF — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#gif_graphics_interchange_format
- Chrome for Developers — Use video formats for animated content — https://developer.chrome.com/docs/performance/insights/image-delivery

---

## 4. WebP — The Modern Universal Format

### Definitions

**Core Definition**
WebP is a modern raster image format developed by Google that provides both lossy and lossless compression, alpha transparency, and animation, at file sizes significantly smaller than JPEG and PNG.

**Technical Definition**
WebP uses VP8-based predictive coding for lossy compression and a separate lossless mode for pixel-perfect images. It supports an alpha channel, animated sequences, and 8-bit Y'CbCr 4:2:0 colour for lossy images. Lossy WebP is typically 25–35% smaller than JPEG at equivalent visual quality, and lossless WebP is about 26% smaller than PNG. The MIME type is `image/webp`; the extension is `.webp`. Browser support is approximately 96% across all modern browsers.

**Beginner-Friendly Explanation**
WebP is a newer format that does everything JPEG and PNG can do, but with smaller files. It works in almost every modern browser. If you want one format that handles photos, transparency, and animation efficiently, WebP is a safe default.

---

### Purposes

- **To reduce file size** for both photographic and graphic content.
- **To provide a single format** that supports transparency and animation.
- **To serve as a safe modern default** with broad browser support.
- **To act as an intermediate fallback** between AVIF and JPEG.

---

### Syntax Rules and Structure

#### Complete General Syntax (Direct `<img>`)

```html
<img src="photo.webp" alt="A mountain landscape" width="1200" height="800">
```

#### Complete General Syntax (with Fallback)

```html
<picture>
  <source type="image/webp" srcset="photo.webp">
  <img src="photo.jpg" alt="A mountain landscape" width="1200" height="800">
</picture>
```

**Component breakdown**

| Component | Description |
|---|---|
| `type="image/webp"` | Signals WebP support in `<source>`. |
| `src="photo.webp"` | Direct WebP image. |

#### MIME Type and Extensions

| Property | Value |
|---|---|
| MIME type | `image/webp` |
| Extension | `.webp` |

#### Syntax Rules

1. **Use `image/webp` as the MIME type**.
2. **Provide a JPEG or PNG fallback** via `<picture>` for older browsers.
3. **WebP is a safe default** for photos, screenshots, transparency, and animation.

#### Constraints and Limitations

- **No progressive rendering**: WebP does not support progressive decoding like JPEG.
- **Browser support is not 100%**: ~96% of browsers support WebP; fallbacks are still recommended for legacy environments.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: WebP with JPEG Fallback

**Setup Guide**: Save as `webp-fallback.html`. Provide WebP and JPEG versions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WebP Fallback</title>
</head>
<body>
  <picture>
    <!-- WebP for supported browsers -->
    <source type="image/webp" srcset="photo.webp">
    <!-- JPEG fallback for older browsers -->
    <img src="photo.jpg" alt="A city skyline at dusk" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: Browsers that support WebP load the smaller WebP file; older browsers load the JPEG.

**Why this output occurs**: The browser selects the first `<source>` whose `type` it supports, falling back to the `<img>` if none match.

---

#### Example 2: Animated WebP

**Setup Guide**: Save as `webp-animated.html`. Provide an animated WebP file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Animated WebP</title>
</head>
<body>
  <img src="animation.webp" alt="Animated loading indicator" width="200" height="200">
</body>
</html>
```

**Expected output**: The animated WebP plays, offering much smaller file size than an equivalent GIF.

**Why this output occurs**: WebP's animation support uses VP8 bitstream frames, which compress far more efficiently than GIF frames.

---

### Real-World Cases

1. **E-commerce product images**: WebP reduces payload while maintaining quality.
2. **Content images**: Blog and news images benefit from 25–35% savings over JPEG.
3. **Animated UI elements**: WebP animation replaces GIF with smaller files.

---

### References

- MDN Web Docs — Image file type and format guide: WebP — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#webp_image
- web.dev — Use WebP images — https://web.dev/articles/serve-images-webp
- caniuse — WebP support — https://caniuse.com/webp

---

## 5. AVIF — The Next-Generation Compression Format

### Definitions

**Core Definition**
AVIF (AV1 Image File Format) is a modern, royalty-free raster image format based on the AV1 video codec. It offers the best compression ratio among web image formats, along with support for transparency, animation, HDR, and wide colour gamut.

**Technical Definition**
AVIF uses the AV1 bitstream for both still images and image sequences. It supports 8-, 10-, and 12-bit colour depths, alpha transparency, HDR, wide colour gamut, and animation. AVIF typically produces files 40–55% smaller than JPEG at equivalent quality. The MIME type is `image/avif`; the extension is `.avif`. Browser support is approximately 93% — Chrome 85+, Firefox 93+, Safari 16.4+, and Edge 121+. AVIF does not support progressive rendering.

**Beginner-Friendly Explanation**
AVIF is the newest format and it makes files even smaller than WebP. It is great for photos and supports transparency and animation. The catch is that very old browsers do not support it, so you still need a fallback.

---

### Purposes

- **To achieve the smallest possible file size** for photographic and illustrative content.
- **To support HDR and wide colour gamut** for high-fidelity imagery.
- **To provide next-generation compression** while remaining royalty-free.
- **To replace JPEG and WebP** as the primary format for performance-critical images.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<picture>
  <source type="image/avif" srcset="photo.avif">
  <source type="image/webp" srcset="photo.webp">
  <img src="photo.jpg" alt="A mountain landscape" width="1200" height="800">
</picture>
```

**Component breakdown**

| Component | Description |
|---|---|
| `type="image/avif"` | Signals AVIF support in `<source>`. |
| `srcset="photo.avif"` | The AVIF file. |

#### MIME Type and Extensions

| Property | Value |
|---|---|
| MIME type | `image/avif` |
| Extension | `.avif` |

#### Syntax Rules

1. **Use `image/avif` as the MIME type**.
2. **Always provide a fallback** (WebP or JPEG) because AVIF support is not universal.
3. **AVIF is the first choice** for photographic content in a `<picture>` chain.

#### Constraints and Limitations

- **No progressive rendering**: The file must be fully downloaded before display.
- **Browser support is ~93%**: Safari added support in 16.4, and older browsers require fallbacks.
- **Encoding is slower than JPEG/WebP**: Build pipelines may take longer.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: AVIF with WebP and JPEG Fallbacks

**Setup Guide**: Save as `avif-fallback.html`. Provide AVIF, WebP, and JPEG versions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>AVIF Fallback</title>
</head>
<body>
  <picture>
    <!-- AVIF: best compression for modern browsers -->
    <source type="image/avif" srcset="photo.avif">
    <!-- WebP: intermediate fallback -->
    <source type="image/webp" srcset="photo.webp">
    <!-- JPEG: universal fallback -->
    <img src="photo.jpg" alt="A mountain landscape at sunrise" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: Modern browsers load the AVIF file; browsers that support WebP but not AVIF load the WebP file; legacy browsers load the JPEG.

**Why this output occurs**: The browser evaluates each `<source>` in order, selecting the first supported type, and falls back to the `<img>` if none match.

---

#### Example 2: AVIF for a Performance-Critical Hero Image

**Setup Guide**: Save as `avif-hero.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>AVIF Hero</title>
</head>
<body>
  <picture>
    <source type="image/avif" srcset="hero-1200.avif 1200w, hero-800.avif 800w" sizes="100vw">
    <source type="image/webp" srcset="hero-1200.webp 1200w, hero-800.webp 800w" sizes="100vw">
    <img src="hero-1200.jpg" alt="A mountain landscape" width="1200" height="800" fetchpriority="high">
  </picture>
</body>
</html>
```

**Expected output**: The LCP hero image is delivered in AVIF at the smallest adequate size, with WebP and JPEG fallbacks.

**Why this output occurs**: The `<source>` elements provide format negotiation and `srcset` resolution switching; `fetchpriority="high"` prioritises the hero for LCP.

---

### Real-World Cases

1. **Performance-focused news sites**: AVIF reduces image payload substantially.
2. **E-commerce**: Smaller product images improve load times.
3. **HDR photography**: AVIF preserves HDR and wide colour gamut.

---

### References

- MDN Web Docs — Image file type and format guide: AVIF — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#avif_image
- caniuse — AVIF support — https://caniuse.com/avif
- Chrome for Developers — Improve image delivery: modern formats — https://developer.chrome.com/docs/performance/insights/image-delivery

---

## 6. SVG — The Vector Graphics Format

### Definitions

**Core Definition**
SVG (Scalable Vector Graphics) is an XML-based vector image format that describes images as geometric shapes, paths, and text rather than as a grid of pixels. It scales infinitely without loss of quality.

**Technical Definition**
SVG is a text-based format that can be embedded inline in HTML or referenced as an external file via `<img>` or CSS. It supports shapes, paths, text, gradients, filters, clipping, masking, and animation (via SMIL or CSS). The MIME type is `image/svg+xml`; the extension is `.svg`. SVG is ideal for logos, icons, diagrams, and charts where geometric precision and resolution independence are required. Browser support is universal.

**Beginner-Friendly Explanation**
SVG is for drawings made of shapes, not photos. A logo or icon made as SVG stays perfectly sharp whether it is displayed tiny or huge. It is not good for photographs, because describing every pixel as a shape would be impossibly large.

---

### Purposes

- **To provide resolution-independent graphics** that remain crisp at any size.
- **To deliver sharp icons and logos** that adapt to different screen densities.
- **To enable accessible, searchable graphics** because SVG text is real text.
- **To support interactive and animated vector graphics**.

---

### Syntax Rules and Structure

#### Complete General Syntax (External SVG)

```html
<img src="logo.svg" alt="Example Company logo" width="240" height="80">
```

#### Complete General Syntax (Inline SVG)

```html
<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="A diagonal line">
  <line x1="0" y1="80" x2="100" y2="20" stroke="black" />
</svg>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<img src="logo.svg">` | External SVG referenced as an image. |
| Inline `<svg>` | SVG markup embedded directly in HTML. |
| `viewBox` | Defines the coordinate system. |

#### MIME Type and Extensions

| Property | Value |
|---|---|
| MIME type | `image/svg+xml` |
| Extension | `.svg` |

#### Syntax Rules

1. **Use `image/svg+xml` as the MIME type**.
2. **Use SVG for logos, icons, charts, and diagrams** — not for photographs.
3. **Inline SVG allows CSS styling and JavaScript interaction**; external SVG is simpler for static assets.

#### Constraints and Limitations

- **Not suitable for photographs**: Describing a photograph with SVG shapes would produce an enormous file and unrealistic output.
- **Complexity**: Complex SVGs can have large source code and rendering costs.
- **Security**: Inline SVG can contain scripts; sanitise user-supplied SVG.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: External SVG Logo

**Setup Guide**: Save as `svg-logo.html`. Provide a logo SVG file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SVG Logo</title>
</head>
<body>
  <img src="logo.svg" alt="Example Company logo" width="240" height="80">
</body>
</html>
```

**Expected output**: The logo renders crisply at 240 × 80 and remains sharp if scaled.

**Why this output occurs**: SVG is vector-based, so it scales without pixelation.

---

#### Example 2: Inline SVG Icon with Accessibility

**Setup Guide**: Save as `svg-inline.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inline SVG Icon</title>
</head>
<body>
  <svg width="24" height="24" viewBox="0 0 24 24" role="img" aria-label="Settings">
    <path d="M12 8a4 4 0 100 8 4 4 0 000-8zm0 6a2 2 0 110-4 2 2 0 010 4z"/>
    <path d="M19.4 13a7.9 7.9 0 000-2l2.1-1.6-2-3.4-2.5 1a7.9 7.9 0 00-1.7-1L15 3h-4l-.3 2.9a7.9 7.9 0 00-1.7 1l-2.5-1-2 3.4L6.6 11a7.9 7.9 0 000 2l-2.1 1.6 2 3.4 2.5-1a7.9 7.9 0 001.7 1L11 21h4l.3-2.9a7.9 7.9 0 001.7-1l2.5 1 2-3.4L19.4 13z"/>
  </svg>
</body>
</html>
```

**Expected output**: A settings icon renders sharply, and screen readers announce “Settings” because of `role="img"` and `aria-label`.

**Why this output occurs**: Inline SVG is fully scalable, and the ARIA attributes provide an accessible name.

---

### Real-World Cases

1. **Icons and logos**: SVG is the standard for resolution-independent branding.
2. **Data visualisation**: Charts and diagrams remain crisp at any zoom.
3. **UI components**: Inline SVG enables styling and animation with CSS.

---

### References

- MDN Web Docs — Image file type and format guide: SVG — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#svg_scalable_vector_graphics
- web.dev — Choose the right image format: vector graphics — https://web.dev/articles/choose-the-right-image-format

---

## 7. HEIC — The Apple Ecosystem Format

### Definitions

**Core Definition**
HEIC (High Efficiency Image Container) is a modern image format based on the HEVC (H.265) video codec, developed by MPEG and used by Apple as the default photo format on iOS devices.

**Technical Definition**
HEIC uses HEVC intra-frame coding and generally offers better compression than WebP, JPEG, PNG, and GIF. However, browser support is severely limited: only Safari on Apple platforms (macOS and iOS) supports HEIC natively; Chrome, Firefox, Edge, and Opera do not. The MIME type is `image/heic`; the extension is `.heic`. The complexity and patent licensing costs of HEVC make broad browser adoption unlikely; AVIF and JPEG XL are designed to supersede HEIC with royalty-free alternatives.

**Beginner-Friendly Explanation**
HEIC is the format iPhones use for photos. It makes very small files, but almost no web browser can display it. You should convert HEIC images to JPEG or WebP before putting them on a website.

---

### Purposes

- **To store photos efficiently on Apple devices**.
- **To serve as the native iOS camera format**.
- **Historically: to offer better compression than JPEG** — but this is now matched by AVIF with far better browser support.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- HEIC is not reliably displayable in browsers; convert first -->
<img src="photo.heic" alt="A photo">
```

**Component breakdown**

| Component | Description |
|---|---|
| `src="photo.heic"` | HEIC file (Safari only). |

#### MIME Type and Extensions

| Property | Value |
|---|---|
| MIME type | `image/heic` (and `image/heif`) |
| Extension | `.heic`, `.heif` |

#### Syntax Rules

1. **Do not use HEIC directly on the web**; convert to JPEG, WebP, or AVIF first.
2. **Use the MIME type `image/heic` only in upload accept lists**, not for display.

#### Constraints and Limitations

- **Browser support is essentially Safari-only**: Chrome, Firefox, Edge, and Opera cannot display HEIC natively.
- **Patent licensing costs** make broad browser adoption unlikely.
- **AVIF and JPEG XL** are the royalty-free successors designed to replace HEIC.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Converting HEIC to Web-Friendly Formats

**Setup Guide**: Save as `heic-conversion.html`. Provide WebP and JPEG versions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>HEIC Conversion</title>
</head>
<body>
  <picture>
    <!-- Convert HEIC to WebP and JPEG for web delivery -->
    <source type="image/webp" srcset="photo.webp">
    <img src="photo.jpg" alt="A photo originally captured as HEIC" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: The photo displays correctly in all browsers using WebP or JPEG; the original HEIC is never served directly.

**Why this output occurs**: HEIC cannot be displayed outside Safari, so conversion is mandatory for web delivery.

---

### Real-World Cases

1. **iPhone photo uploads**: Websites must accept HEIC uploads but convert them for display.
2. **Apple ecosystem**: HEIC is the default camera format on iOS.
3. **Cloud photo storage**: Apple iCloud stores photos as HEIC.

---

### References

- caniuse — HEIF/HEIC image format — https://caniuse.com/heif
- YOOtheme — HEIC browser support discussion — https://www.yootheme.com/support/question/168021
- xConvert — Why Won't My WebP, HEIC, or AVIF Upload? — https://www.xconvert.com

---

## 8. Format Selection — Choosing the Right Format

### Definitions

**Core Definition**
Format selection is the process of choosing the most appropriate image format for a given image based on its content, required features (transparency, animation), quality needs, and the browser support requirements of the target audience.

**Technical Definition**
Format selection involves evaluating four dimensions: **content type** (photographic vs graphic), **feature requirements** (transparency, animation, HDR), **compression efficiency** (lossy vs lossless), and **compatibility** (browser support). The decision tree recommended by performance guidance is: photos and complex graphics → AVIF (primary) + WebP (fallback) + JPEG (legacy); logos, icons, and simple illustrations → SVG; screenshots and transparency → PNG (with WebP fallback).

**Beginner-Friendly Explanation**
Format selection means asking: what kind of image is this, and what do I need it to do? A photo should be AVIF or WebP. A logo should be SVG. A screenshot should be PNG or WebP. An animation should be video, not GIF.

---

### Purposes

- **To minimise file size** while maintaining acceptable quality.
- **To ensure every browser can display the image** through appropriate fallbacks.
- **To match the format to the content** (photographs vs graphics vs vector).
- **To avoid unnecessary conversions** and wasted bandwidth.

---

### Syntax Rules and Structure

#### Decision Tree

```
Is it a photo or complex graphic?
├── Yes → AVIF (primary) + WebP (fallback) + JPEG (legacy)
└── No → Is it a logo, icon, or simple illustration?
    ├── Yes → SVG (inline or external)
    └── No (raster required) → PNG with WebP fallback
```

#### Format Comparison Table

| Format | Best For | Compression | Transparency | Animation | Browser Support |
|---|---|---|---|---|---|
| **AVIF** | Photos, illustrations | ~50% smaller than JPEG | Yes | Yes | ~93% (Chrome 85+, Firefox 93+, Safari 16.4+) |
| **WebP** | Photos, screenshots | ~30% smaller than JPEG | Yes | Yes | ~96% (all modern) |
| **JPEG** | Photos | Baseline | No | No | Universal (100%) |
| **PNG** | Screenshots, transparency | Lossless | Yes | No (APNG possible) | Universal (100%) |
| **SVG** | Icons, logos, diagrams | Vector (tiny for simple shapes) | Yes | Yes (SMIL/CSS) | Universal |
| **GIF** | Simple animations | Poor (LZW) | Binary only | Yes | Universal |
| **HEIC** | Apple photos | Better than WebP | Yes | Yes | Safari only |

#### Syntax Rules

1. **Photos → AVIF → WebP → JPEG** fallback chain.
2. **Logos and icons → SVG**; use raster PNG/WebP only when SVG is not feasible.
3. **Screenshots and transparency → PNG or WebP**; prefer WebP when possible.
4. **Animation → Video (WebM/MP4)** instead of GIF.
5. **Never serve HEIC on the web** without conversion.

#### Constraints and Limitations

- **AVIF does not support progressive rendering**; for very large images, consider WebP or JPEG.
- **SVG is unsuitable for photographs**; the file size would be enormous.
- **Browser support changes over time**; always verify with caniuse.com.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Photo Format Chain with Fallbacks

**Setup Guide**: Save as `format-chain.html`. Provide AVIF, WebP, and JPEG versions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Format Chain</title>
</head>
<body>
  <picture>
    <source type="image/avif" srcset="photo.avif">
    <source type="image/webp" srcset="photo.webp">
    <img src="photo.jpg" alt="A mountain landscape" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: The browser selects the smallest supported format: AVIF first, then WebP, then JPEG.

**Why this output occurs**: The `<picture>` element evaluates `<source>` elements in order and selects the first supported type.

---

#### Example 2: SVG for an Icon

**Setup Guide**: Save as `format-svg.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SVG Icon</title>
</head>
<body>
  <svg width="24" height="24" viewBox="0 0 24 24" aria-label="Search" role="img">
    <circle cx="10" cy="10" r="7" fill="none" stroke="currentColor" stroke-width="2"/>
    <line x1="15" y1="15" x2="21" y2="21" stroke="currentColor" stroke-width="2"/>
  </svg>
</body>
</html>
```

**Expected output**: A crisp search icon renders at any size, with an accessible name.

**Why this output occurs**: SVG is the correct format for simple geometric graphics and scales without loss.

---

### Real-World Cases

1. **E-commerce**: AVIF/WebP for product photos, SVG for icons.
2. **News sites**: AVIF/WebP for article images, JPEG fallback for legacy.
3. **Design systems**: SVG for all icons and logos.

---

### References

- web.dev — Choose the right image format — https://web.dev/articles/choose-the-right-image-format
- Chrome for Developers — Improve image delivery — https://developer.chrome.com/docs/performance/insights/image-delivery
- GitHub — Image optimization decision tree — https://github.com/aiocean/claude-plugins

---

## 9. Format Best Practices

### Definitions

**Core Definition**
Format best practices are the consolidated guidelines for selecting, compressing, and delivering images in the most efficient and compatible way — combining modern format adoption, fallback chains, responsive delivery, and accessibility discipline.

**Technical Definition**
Best practices are informed by Chrome for Developers’ image delivery guidance, web.dev’s format selection documentation, and MDN’s format reference. They encompass: using AVIF and WebP as primary formats with JPEG/PNG fallbacks; replacing GIF with video for animation; using SVG for vector graphics; compressing images appropriately; setting explicit dimensions to prevent CLS; and always providing `alt` text.

**Beginner-Friendly Explanation**
Best practices help you use image formats correctly: pick the modern format first, provide fallbacks, do not use GIF for animation, use SVG for logos, compress your images, set dimensions, and always describe them.

---

### Purposes

- **To minimise image payload** and improve LCP.
- **To ensure cross-browser compatibility** through fallbacks.
- **To maintain visual quality** at the smallest possible file size.
- **To prevent layout shift** with explicit dimensions.
- **To satisfy accessibility requirements** with `alt` text.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use AVIF as primary for photos** | ~50% smaller than JPEG; fallback to WebP/JPEG. |
| **Use WebP as the safe default** | ~96% support; handles photos, transparency, and animation. |
| **Use SVG for icons and logos** | Resolution-independent and tiny for simple shapes. |
| **Replace GIF with video** | WebM/MP4 are far more efficient for animation. |
| **Never serve HEIC** | Convert to JPEG/WebP/AVIF first. |
| **Always set `width` and `height`** | Prevents cumulative layout shift (CLS). |
| **Always provide `alt` text** | Required for accessibility and fallback. |
| **Use `<picture>` for format negotiation** | Enables modern formats with fallbacks. |
| **Compress at quality 75–85 for JPEG** | Balances size and quality. |

#### Syntax Rules

1. **Modern-first**: Use AVIF or WebP as the primary format; fall back to JPEG/PNG.
2. **Content-appropriate**: Photos → AVIF/WebP; graphics → SVG or PNG; animation → video.
3. **Dimension discipline**: Always set `width` and `height`.
4. **Accessibility discipline**: Always provide `alt`.
5. **Compatibility discipline**: Always provide fallbacks for formats with <100% support.

#### Constraints and Limitations

- **AVIF encoding is slower**; build pipelines need to accommodate it.
- **SVG is not for photos**; use raster formats for photographic content.
- **HEIC is not a web format**; conversion is mandatory.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Image Delivery Chain

**Setup Guide**: Save as `best-practice-formats.html`. Provide AVIF, WebP, and JPEG at multiple widths.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Formats</title>
</head>
<body>
  <picture>
    <!-- AVIF with responsive candidates -->
    <source
      type="image/avif"
      srcset="hero-400.avif 400w, hero-800.avif 800w, hero-1200.avif 1200w"
      sizes="(max-width: 600px) 100vw, 50vw">
    <!-- WebP fallback -->
    <source
      type="image/webp"
      srcset="hero-400.webp 400w, hero-800.webp 800w, hero-1200.webp 1200w"
      sizes="(max-width: 600px) 100vw, 50vw">
    <!-- JPEG universal fallback with dimensions and priority -->
    <img
      src="hero-800.jpg"
      srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1200.jpg 1200w"
      sizes="(max-width: 600px) 100vw, 50vw"
      alt="A mountain landscape with a lake"
      width="1200"
      height="800"
      fetchpriority="high">
  </picture>
</body>
</html>
```

**Expected output**: The browser selects the best combination of format and size, with AVIF preferred, WebP as fallback, and JPEG as the universal fallback. Layout space is reserved, and the image is prioritised for LCP.

**Why this output occurs**: The `<picture>` chain implements format negotiation; `srcset`/`sizes` implement resolution switching; `width`/`height` prevent CLS; and `fetchpriority="high"` prioritises the hero image.

---

#### Example 2: Replacing an Animated GIF with Video

**Setup Guide**: Save as `best-practice-animation.html`.

```html
<!-- BAD: GIF animation at 2MB -->
<!-- <img src="loading.gif" alt="Loading animation"> -->

<!-- GOOD: Video animation at ~200KB -->
<video autoplay loop muted playsinline width="200" height="200" aria-label="Loading animation">
  <source src="loading.webm" type="video/webm">
  <source src="loading.mp4" type="video/mp4">
</video>
```

**Expected output**: The animation plays with a fraction of the bandwidth required by the GIF.

**Why this output occurs**: Video codecs compress temporal redundancy far more efficiently than GIF’s frame-by-frame encoding.

---

### Real-World Cases

1. **Core Web Vitals optimisation**: AVIF/WebP delivery directly improves LCP.
2. **Design systems**: SVG icons and logos ensure crisp rendering at all densities.
3. **Performance audits**: Replacing GIFs with video is a common quick win.

---

### References

- Chrome for Developers — Improve image delivery — https://developer.chrome.com/docs/performance/insights/image-delivery
- web.dev — Choose the right image format — https://web.dev/articles/choose-the-right-image-format
- MDN Web Docs — Image file type and format guide — https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types

---

## Summary Table of Image Formats

| Format | Type | Compression | Transparency | Animation | Browser Support | Best For |
|---|---|---|---|---|---|---|
| **JPEG** | Raster | Lossy | No | No | Universal | Photographs, fallback |
| **PNG** | Raster | Lossless | Yes (alpha) | No (APNG) | Universal | Screenshots, transparency |
| **GIF** | Raster | Lossless (LZW) | Binary only | Yes | Universal | Legacy animations |
| **WebP** | Raster | Lossy + Lossless | Yes | Yes | ~96% | Safe modern default |
| **AVIF** | Raster | Lossy + Lossless | Yes | Yes | ~93% | Best compression for photos |
| **SVG** | Vector | N/A (text-based) | Yes | Yes (SMIL/CSS) | Universal | Icons, logos, diagrams |
| **HEIC** | Raster | Lossy (HEVC) | Yes | Yes | Safari only | Apple ecosystem (not web) |

---

## Final Notes

- **Modern formats are not optional**: AVIF and WebP deliver 25–50% smaller files than JPEG and PNG, directly improving Core Web Vitals.
- **Fallbacks are mandatory**: No modern format has 100% browser support; always provide a JPEG or PNG fallback via `<picture>`.
- **SVG is for graphics, not photos**: Use vector formats for logos and icons, raster formats for photographs.
- **GIF is obsolete for animation**: Video (WebM/MP4) is far more efficient and should replace GIF wherever possible.
- **HEIC is not a web format**: Convert HEIC to JPEG, WebP, or AVIF before serving.
- **Dimensions and `alt` are non-negotiable**: Always set `width`/`height` and provide `alt` text regardless of format.

---

This cheat sheet has covered image formats — JPEG, PNG, GIF, WebP, AVIF, SVG, and HEIC — plus format selection and best practices, with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate format choices against MDN’s format guide, Chrome for Developers’ image delivery guidance, and caniuse.com browser support data.