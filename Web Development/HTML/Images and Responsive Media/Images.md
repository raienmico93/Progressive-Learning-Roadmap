# HTML Images: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML images are embedded visual content inserted into a web document using the `<img>` element, which references an external image file through the `src` attribute and provides a textual alternative through the `alt` attribute.

**Technical Definition**

The `<img>` element embeds an image into the document. It is a replaced element, meaning its content is replaced by an external resource. The element is categorised as flow content, phrasing content, embedded content, and palpable content. Its content model is nothing (it is a void element, with no closing tag). It supports global attributes plus `src`, `alt`, `width`, `height`, `srcset`, `sizes`, `loading`, `decoding`, `fetchpriority`, `crossorigin`, `referrerpolicy`, `ismap`, and `usemap`. Its DOM interface is `HTMLImageElement`.

**Beginner-Friendly Explanation**

An image on a webpage is like a picture in a book. You use the `<img>` tag to tell the browser where to find the picture (using the `src` attribute) and what the picture shows (using the `alt` attribute). The browser then loads and displays the picture. You can also control the size of the image with the `width` and `height` attributes, and you can choose from several different image file formats depending on what kind of picture you have.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Void element** | `<img>` has no closing tag and no content model |
| **Replaced element** | The element‘s content is replaced by the external image file |
| **`src` is essential** | Without `src` (or `srcset`), the image cannot be loaded |
| **`alt` is required** | The `alt` attribute must be present and appropriate for accessibility |
| **Dimension attributes** | `width` and `height` should be specified to prevent layout shifts |
| **Multiple formats** | JPEG, PNG, GIF, SVG, WebP, and AVIF are all supported by modern browsers |
| **Responsive support** | `srcset` and `sizes` enable responsive image loading |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Awareness of URLs and how they identify resources
- Basic knowledge of file formats and image types

---

### Related Programming Areas

- **Web Accessibility (A11y)** – The `alt` attribute is critical for screen reader users
- **Responsive Web Design** – `srcset` and `<picture>` enable responsive images
- **Web Performance** – Image format selection and lazy loading affect page speed
- **CSS** – Images can be styled and sized with CSS
- **SEO** – Descriptive `alt` text and file names improve image search ranking

---

## Core Concepts / Features

---

### 1. The `<img>` Element

#### Definitions

**Core Definition**

The `<img>` element embeds an image into an HTML document, replacing its content with the external image file referenced by the `src` attribute.

**Technical Definition**

The `<img>` HTML element embeds an image into the document. It is a replaced element whose content is replaced by the external resource specified in the `src` attribute. The element is categorised as flow content, phrasing content, embedded content, and palpable content. Its content model is nothing. It supports global attributes plus several image-specific attributes. Its DOM interface is `HTMLImageElement`.

**Beginner-Friendly Explanation**

The `<img>` tag is how you put a picture on a webpage. It doesn‘t have a closing tag because there’s no content inside it — the browser loads the picture from the URL you provide. You always need at least a `src` attribute to tell the browser where the picture is, and an `alt` attribute to describe it.

#### Purposes

- To embed a visual image into an HTML document
- To display photographs, illustrations, logos, and icons
- To provide visual content that complements text
- To serve as a link target when wrapped in an `<a>` element
- To support responsive and adaptive image loading

#### Syntax Rules and Structure

**General Syntax**

```html
<img src="URL" alt="description">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<img>` | Void element; no closing tag |
| `src` | Required; URL of the image to embed |
| `alt` | Required; textual alternative for the image |
| `width` | Optional; intrinsic width in pixels |
| `height` | Optional; intrinsic height in pixels |

**Syntax Rules**

- The `<img>` element must have a `src` attribute (or `srcset` with `src` as fallback)
- The `alt` attribute must be specified and its value must not be empty unless the image is decorative
- The element has no closing tag
- Both `width` and `height` should be specified to prevent content layout shifts
- The element may be wrapped in an `<a>` element to create a linked image

**Constraints and Limitations**

- Without `src`, the image cannot be loaded
- The `alt` attribute value must be an appropriate replacement for the image, not a description of the file
- Using `<img>` for decorative purposes requires an empty `alt` attribute (`alt=""`)
- The element does not support fallback content like `<picture>` does

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Image Embed**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Basic Image Demo</title>
</head>
<body>
    <h1>My Photo</h1>

    <!-- Basic img element with required attributes -->
    <img
        src="https://github.com/mdn/learning-area/blob/main/html/multimedia-and-embedding/tasks/images/images/blueberries.jpg?raw=true"
        alt="A pile of small blue berries"
        width="400"
        height="272"
    >
</body>
</html>
```

**Expected Output**

A photograph of blueberries displayed at 400 pixels wide, maintaining the correct aspect ratio.

**Why This Output Occurs**

The `src` attribute provides the image URL. The `alt` attribute provides a textual description. The `width` and `height` attributes set the intrinsic size, allowing the browser to reserve space before the image loads, preventing content layout shifts. The intrinsic size of the image is 615 × 419 pixels, so the width of 400 pixels is scaled proportionally.

---

**Example 2: Linked Image**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Linked Image</title>
</head>
<body>
    <!-- Image wrapped in a link -->
    <a href="https://example.com/full-size-image.html">
        <img
            src="thumbnail.jpg"
            alt="View the full-size photograph of a mountain landscape"
            width="200"
            height="150"
        >
    </a>
</body>
</html>
```

**Expected Output**

A thumbnail image that, when clicked, navigates to a page showing the full-size image.

**Why This Output Occurs**

The `<img>` element is wrapped in an `<a>` element, making the image a clickable link. The `alt` text describes both the image and the link’s purpose.

#### Real-World Cases

**Case 1: E-Commerce Product Images**

Online stores use `<img>` to display product photos, often with `srcset` to serve different resolutions.

**Case 2: News Article Images**

News websites use `<img>` for article hero images, with `alt` text describing the scene.

**Case 3: User Avatars**

Social media and forum platforms use `<img>` for user profile pictures.

---

### 2. The `src` Attribute

#### Definitions

**Core Definition**

The `src` attribute specifies the URL of the image file to embed in the document.

**Technical Definition**

The `src` attribute contains the path to the image you want to embed. It can be an absolute URL or a relative URL, similar to the `href` attribute of the `<a>` element. The `src` attribute is not mandatory if the `srcset` attribute is available, but at least one of `src` or `srcset` must be provided. If `src` is omitted and `srcset` is present, `src` is used as a fallback for browsers that do not support `srcset`.

**Beginner-Friendly Explanation**

The `src` attribute is the address of the image. It tells the browser where to find the picture file. You can use a full web address (like `https://example.com/photo.jpg`) or a relative path (like `images/photo.jpg`). Without `src`, the browser doesn’t know where to look.

#### Purposes

- To specify the location of the image file
- To enable the browser to load and display the image
- To provide a fallback URL when `srcset` is not supported
- To allow the image to be cached and reused across pages

#### Syntax Rules and Structure

**General Syntax**

```html
<img src="URL" alt="description">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `src` | Attribute name |
| `"URL"` | The URL of the image file (absolute or relative) |

**Syntax Rules**

- The `src` attribute must contain a valid URL
- The URL can be absolute or relative
- At least one of `src` or `srcset` must be present
- If `srcset` is present and `src` is omitted, `src` should be provided as a fallback

**Constraints and Limitations**

- The image must be in a supported format (JPEG, PNG, GIF, SVG, WebP, AVIF)
- The URL must be accessible to the browser
- Cross-origin images may require CORS headers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Absolute URL**

```html
<img
    src="https://developer.mozilla.org/shared-assets/images/examples/grapefruit-slice.jpg"
    alt="Grapefruit slice atop a pile of other slices"
    width="250"
>
```

**Expected Output**

A photograph of a grapefruit slice displayed at 250 pixels wide.

**Why This Output Occurs**

The absolute URL points directly to the image file on the MDN server. The browser loads and displays the image.

---

**Example 2: Relative URL**

```html
<!-- Image is in the same directory as the HTML file -->
<img src="logo.png" alt="Company logo" width="150" height="50">

<!-- Image is in a subdirectory -->
<img src="images/hero.jpg" alt="Hero image" width="800" height="400">
```

**Expected Output**

The logo and hero image are loaded from the local file system or the same website.

**Why This Output Occurs**

Relative URLs are resolved relative to the current document‘s location. The browser looks for the file in the specified directory.

#### Real-World Cases

**Case 1: CDN-Hosted Images**

Large websites host images on content delivery networks (CDNs) and reference them with absolute URLs.

**Case 2: Local Development**

Developers use relative URLs when building sites locally before deploying.

**Case 3: User-Generated Content**

Platforms store user-uploaded images and reference them with absolute URLs.

---

### 3. The `alt` Attribute

#### Definitions

**Core Definition**

The `alt` attribute provides a textual alternative for an image, displayed when the image cannot be loaded and read by screen readers.

**Technical Definition**

The `alt` attribute defines text that can replace the image in the page. Browsers do not always display images — for example, in non-visual browsers, when the user chooses not to display images, or when the image is invalid. In these cases, the browser may replace the image with the text in the `alt` attribute. Setting the `alt` attribute to an empty string (`alt=""`) indicates that the image is not a key part of the content (it is decoration or a tracking pixel), and non-visual browsers may omit it from rendering. The `alt` attribute is also used when copying and pasting the image to text, or saving a linked image to a bookmark.

**Beginner-Friendly Explanation**

The `alt` attribute is a description of the image. It‘s used when the image can’t be seen — for example, by people using screen readers, or when the image fails to load. Every image should have an `alt` attribute, but if the image is purely decorative, you can use an empty `alt=""`.

#### Purposes

- To provide a textual alternative for screen reader users
- To display text when the image fails to load
- To improve accessibility for users with visual impairments
- To provide context for search engines
- To describe the image for users who have images disabled

#### Syntax Rules and Structure

**General Syntax**

```html
<img src="URL" alt="description">
<img src="URL" alt="">  <!-- Decorative image -->
```

**Component Breakdown**

| Component | Description |
|---|---|
| `alt` | Attribute name |
| `"description"` | A brief, descriptive text for the image |
| `""` | Empty string; indicates a decorative image |

**Syntax Rules**

- The `alt` attribute must be specified on every `<img>` element
- The value must not be empty unless the image is decorative
- The text should be brief and concise, providing all relevant information conveyed by the image
- For decorative images, use `alt=""` to tell screen readers to skip the image

**Constraints and Limitations**

- The `alt` attribute value must not describe the image file (e.g., “photo.jpg”)
- The `alt` text must not be redundant with surrounding text
- Omitting the `alt` attribute entirely indicates the image is a key part of the content but no textual equivalent is available

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Informative Image**

```html
<img
    src="dinosaur.jpg"
    alt="A two-legged dinosaur standing upright like a human, with small arms, and a large head with lots of sharp teeth"
>
```

**Expected Output**

When the image is displayed, the dinosaur is visible. When it is not displayed, the `alt` text is shown.

**Why This Output Occurs**

The `alt` attribute provides a textual equivalent of the image’s content. The description is brief but includes all the information conveyed by the image that is not duplicated in the surrounding text.

---

**Example 2: Decorative Image**

```html
<img src="background-pattern.png" alt="" width="100" height="100">
```

**Expected Output**

The decorative background pattern is displayed visually. Screen readers skip it entirely.

**Why This Output Occurs**

The empty `alt` attribute (`alt=""`) tells assistive technology that the image is purely decorative and can be ignored.

#### Real-World Cases

**Case 1: Screen Reader Accessibility**

Screen readers announce the `alt` text when encountering images, providing context for users who cannot see them.

**Case 2: Image Search**

Search engines use `alt` text to understand and index image content.

**Case 3: Broken Image Fallback**

When an image fails to load, browsers display the `alt` text in its place.

---

### 4. The `width` and `height` Attributes

#### Definitions

**Core Definition**

The `width` and `height` attributes specify the intrinsic dimensions of an image in CSS pixels, allowing the browser to reserve space before the image loads.

**Technical Definition**

The `width` and `height` attributes define the intrinsic size of the image in CSS pixels. These attributes are useful for setting the space the image will occupy, ensuring layout stability before the image loads. The `height` attribute specifies the intrinsic height in pixels, and the `width` attribute specifies the intrinsic width. The DOM properties `image.width` and `image.height` return the actual rendered dimensions of the image, or 0 if the dimensions are not known.

**Beginner-Friendly Explanation**

The `width` and `height` attributes tell the browser how big the image should be before it actually loads. This prevents the page from jumping around when the image appears. If you don‘t specify these, the browser has to wait for the image to load before it knows how much space to reserve, which can cause content layout shifts.

#### Purposes

- To reserve space for the image before it loads
- To prevent content layout shifts and improve perceived performance
- To set the intrinsic size of the image
- To allow the browser to calculate the correct aspect ratio

#### Syntax Rules and Structure

**General Syntax**

```html
<img src="URL" alt="description" width="400" height="272">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `width` | Intrinsic width in CSS pixels |
| `height` | Intrinsic height in CSS pixels |

**Syntax Rules**

- Both `width` and `height` should be specified together to maintain aspect ratio
- Values are unitless integers (pixels)
- CSS can override these attributes for responsive layouts
- The intrinsic size should reflect the image‘s natural dimensions or a proportionally scaled version

**Constraints and Limitations**

- Specifying only one dimension may cause the image to be scaled disproportionately
- These attributes set the intrinsic size, but CSS `width` and `height` properties take precedence
- Omitting both attributes can cause layout shifts

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Preventing Layout Shift**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Layout Stability Demo</title>
    <style>
        .container {
            border: 1px solid #ccc;
            padding: 1em;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Article Title</h1>

        <!-- width and height prevent layout shift -->
        <img
            src="hero-image.jpg"
            alt="Mountain landscape with a lake in the foreground"
            width="800"
            height="450"
            style="max-width: 100%; height: auto;"
        >

        <p>Article content begins here. The space for the image is reserved
        before the image loads, so this text does not jump down when the
        image appears.</p>
    </div>
</body>
</html>
```

**Expected Output**

The browser reserves an 800 × 450 pixel space for the image. The text below remains in place even before the image loads.

**Why This Output Occurs**

The `width` and `height` attributes tell the browser the intrinsic size of the image. The CSS `max-width: 100%` and `height: auto` make the image responsive while maintaining its aspect ratio.

---

**Example 2: Responsive Image with CSS**

```html
<img
    src="photo.jpg"
    alt="A sunset over the ocean"
    width="1200"
    height="800"
    style="width: 100%; height: auto; max-width: 600px;"
>
```

**Expected Output**

The image displays at up to 600 pixels wide on large screens and scales down on smaller screens, always maintaining its 3:2 aspect ratio.

**Why This Output Occurs**

The `width` and `height` attributes define the intrinsic size (1200 × 800). The CSS `width: 100%` makes the image fill its container up to `max-width: 600px`, and `height: auto` maintains the aspect ratio.

#### Real-World Cases

**Case 1: News Websites**

News sites specify `width` and `height` on article images to prevent content jumping as readers scroll.

**Case 2: E-Commerce Product Grids**

Product images use `width` and `height` to reserve consistent space in grid layouts.

**Case 3: Core Web Vitals**

Google’s Core Web Vitals metrics measure Cumulative Layout Shift (CLS), which is reduced by specifying image dimensions.

---

### 5. Image Formats

#### Definitions

**Core Definition**

Image formats are the file types used to store and display digital images on the web, each with different characteristics for compression, transparency, animation, and quality.

**Technical Definition**

Web browsers support several image file formats, each with distinct characteristics. Raster formats (JPEG, PNG, GIF, WebP, AVIF) encode individual pixel values within a rectangular grid. Vector formats (SVG) use lines, points, and polygons to represent images mathematically. The choice of format depends on the image content (photographic vs. graphical), the need for transparency or animation, and the desired balance between file size and visual quality.

**Beginner-Friendly Explanation**

Different types of images are saved in different file formats. Photos are usually JPEG or WebP. Images with transparency are PNG or WebP. Simple animations are GIF or WebP. Logos and icons are often SVG because they stay sharp at any size. Newer formats like WebP and AVIF give you smaller files with better quality, but older formats like JPEG and PNG work everywhere.

#### Purposes

- To provide an appropriate format for different types of visual content
- To optimise file size for faster page loading
- To support transparency, animation, or vector graphics as needed
- To balance quality and performance across different browsers

#### Format Comparison

| Format | Type | Transparency | Animation | Best For | Browser Support |
|---|---|---|---|---|---|
| **JPEG** | Raster | No | No | Photographs, complex images | Universal |
| **PNG** | Raster | Yes | APNG | Graphics, images with transparency | Universal |
| **GIF** | Raster | Yes (binary) | Yes | Simple animations | Universal |
| **SVG** | Vector | Yes | Yes (SMIL/CSS) | Icons, logos, diagrams | Universal |
| **WebP** | Raster | Yes | Yes | General-purpose, both photos and graphics | Modern browsers |
| **AVIF** | Raster | Yes | Yes | High-quality images, future-proofing | Modern browsers |

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Using Different Formats**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Image Formats Demo</title>
</head>
<body>
    <!-- JPEG: Photograph -->
    <img src="photo.jpg" alt="Mountain landscape" width="800" height="450">

    <!-- PNG: Image with transparency -->
    <img src="logo.png" alt="Company logo with transparent background" width="200" height="60">

    <!-- GIF: Simple animation -->
    <img src="loading.gif" alt="Loading spinner animation" width="50" height="50">

    <!-- SVG: Vector icon -->
    <img src="icon.svg" alt="Settings gear icon" width="24" height="24">

    <!-- WebP with JPEG fallback using picture -->
    <picture>
        <source srcset="photo.webp" type="image/webp">
        <img src="photo.jpg" alt="Mountain landscape" width="800" height="450">
    </picture>

    <!-- AVIF with WebP and JPEG fallback -->
    <picture>
        <source srcset="photo.avif" type="image/avif">
        <source srcset="photo.webp" type="image/webp">
        <img src="photo.jpg" alt="Mountain landscape" width="800" height="450">
    </picture>
</body>
</html>
```

**Expected Output**

Each image is displayed using the appropriate format. Modern browsers load WebP and AVIF; older browsers fall back to JPEG.

**Why This Output Occurs**

The `<picture>` element allows multiple sources to be specified. The browser selects the first supported format. The `<img>` element serves as the final fallback.

#### Real-World Cases

**Case 1: Photography Websites**

Photography sites use JPEG for maximum compatibility, with WebP or AVIF for modern browsers.

**Case 2: E-Commerce Product Images**

Product images use PNG for transparent backgrounds or JPEG for photographic quality.

**Case 3: Icon Libraries**

Icon libraries use SVG for scalable, resolution-independent icons.

---

#### 5.1 JPEG (Joint Photographic Expert Group)

##### Definitions

**Core Definition**

JPEG is a lossy raster image format optimised for photographic images with smooth colour transitions.

**Technical Definition**

JPEG (Joint Photographic Expert Group image) is a lossy compression format that achieves small file sizes by discarding some visual information. It is the most popular format for photographic images on the web. JPEG does not support transparency or animation. The MIME type is `image/jpeg`, and the file extensions are `.jpg`, `.jpeg`, `.jfif`, `.pjpeg`, and `.pjp`. Support: Universal across all browsers.

**Beginner-Friendly Explanation**

JPEG is the go-to format for photographs. It makes files smaller by throwing away some details that your eye won‘t notice. JPEG doesn’t support transparent backgrounds, so it‘s not good for logos or icons.

##### Purposes

- To display photographic images with small file sizes
- To balance visual quality and compression
- To ensure compatibility across all browsers

##### Syntax Rules and Structure

**General Syntax**

```html
<img src="photo.jpg" alt="description" width="800" height="450">
```

**Constraints and Limitations**

- No transparency support
- Lossy compression means repeated saving degrades quality
- Not suitable for images with text or sharp edges

##### Annotated Code Example

```html
<img src="sunset.jpg" alt="A vibrant orange sunset over the ocean" width="1200" height="800">
```

**Expected Output**

A high-quality photograph displayed at 1200 × 800 pixels.

**Why This Output Occurs**

JPEG’s lossy compression is ideal for the smooth gradients and colour transitions in a sunset photograph.

---

#### 5.2 PNG (Portable Network Graphics)

##### Definitions

**Core Definition**

PNG is a lossless raster image format that supports transparency and is ideal for graphics, logos, and images requiring precise reproduction.

**Technical Definition**

PNG (Portable Network Graphics) is a lossless compression format that supports full alpha channel transparency. It is preferred over JPEG for more precise reproduction of source images, or when transparency is needed. The MIME type is `image/png`, and the file extension is `.png`. Support: Universal across all browsers.

**Beginner-Friendly Explanation**

PNG is the best choice for images that need transparent backgrounds, like logos. It also gives you sharper, more precise images than JPEG, but the files are usually bigger.

##### Purposes

- To display images with transparent backgrounds
- To reproduce graphics and text with sharp edges
- To preserve image quality without compression artefacts

##### Syntax Rules and Structure

**General Syntax**

```html
<img src="logo.png" alt="description" width="200" height="60">
```

**Constraints and Limitations**

- Larger file sizes than JPEG for photographs
- Not ideal for photographic images

##### Annotated Code Example

```html
<img src="logo.png" alt="Company logo with transparent background" width="200" height="60">
```

**Expected Output**

A logo displayed with a transparent background, allowing the page background to show through.

**Why This Output Occurs**

PNG’s alpha channel supports per-pixel transparency, making it ideal for logos and graphics that need to blend with different backgrounds.

---

#### 5.3 GIF (Graphics Interchange Format)

##### Definitions

**Core Definition**

GIF is a raster image format that supports simple animations and binary transparency, best for basic animated images.

**Technical Definition**

GIF (Graphics Interchange Format) is a raster format that supports animation and binary transparency (each pixel is either fully transparent or fully opaque). It is limited to 256 colours per frame. The MIME type is `image/gif`, and the file extension is `.gif`. Support: Universal across all browsers. GIF is a good choice for basic images and animations, but PNG is preferred for lossless still images, and WebP or AVIF are preferred for animation sequences.

**Beginner-Friendly Explanation**

GIF is the classic format for simple animations — like the looping images you see on social media. It only supports 256 colours, so it‘s not great for photos. For animations, WebP or AVIF are better choices today.

##### Purposes

- To display simple, looping animations
- To provide a universally supported animation format
- To show images with binary transparency

##### Syntax Rules and Structure

**General Syntax**

```html
<img src="animation.gif" alt="description" width="300" height="200">
```

**Constraints and Limitations**

- Limited to 256 colours per frame
- Binary transparency only (no partial transparency)
- Larger file sizes than modern animated formats

##### Annotated Code Example

```html
<img src="loading.gif" alt="Loading spinner animation" width="50" height="50">
```

**Expected Output**

A small, looping loading spinner animation.

**Why This Output Occurs**

GIF’s animation support makes it suitable for simple UI animations. However, the 256-colour limit is acceptable for simple graphics.

---

#### 5.4 SVG (Scalable Vector Graphics)

##### Definitions

**Core Definition**

SVG is an XML-based vector image format that describes images using mathematical shapes, allowing them to scale to any size without loss of quality.

**Technical Definition**

SVG (Scalable Vector Graphics) is a vector image format defined in XML. It is ideal for user interface elements, icons, diagrams, and illustrations that must be drawn accurately at different sizes. SVG supports animation and transparency. The MIME type is `image/svg+xml`, and the file extension is `.svg`. Support: Universal across all browsers.

**Beginner-Friendly Explanation**

SVG images are made of lines and shapes instead of pixels. This means they stay perfectly sharp no matter how big or small you make them. They‘re perfect for logos, icons, and diagrams because they look great at any size and the files are usually very small.

##### Purposes

- To display resolution-independent graphics
- To create icons and logos that scale perfectly
- To provide small file sizes for simple graphics
- To enable animation and interactivity with CSS and JavaScript

##### Syntax Rules and Structure

**General Syntax**

```html
<img src="icon.svg" alt="description" width="24" height="24">
```

**Constraints and Limitations**

- Not suitable for complex photographic images
- Can become large for very complex illustrations
- Requires XML syntax

##### Annotated Code Example

```html
<img src="settings-icon.svg" alt="Settings gear icon" width="24" height="24">
```

**Expected Output**

A sharp settings gear icon displayed at 24 × 24 pixels.

**Why This Output Occurs**

SVG’s vector nature means the icon remains crisp at any size. The file is small because it describes the shape mathematically rather than storing pixel data.

---

#### 5.5 WebP (Web Picture Format)

##### Definitions

**Core Definition**

WebP is a modern raster image format developed by Google that provides superior compression for both photographic and graphic images, with support for transparency and animation.

**Technical Definition**

WebP (Web Picture format) is a modern image format that offers much better compression than PNG or JPEG with support for higher colour depths, animated frames, and transparency. It is an excellent choice for both images and animated images. The MIME type is `image/webp`, and the file extension is `.webp`. Support: Chrome, Edge, Firefox, Opera, Safari.

**Beginner-Friendly Explanation**

WebP is a newer format that gives you smaller files than JPEG or PNG, with the same or better quality. It supports transparency and animation too. Most modern browsers support it, but you should still provide a fallback for older browsers.

##### Purposes

- To reduce file size while maintaining image quality
- To support both photographic and graphic content with one format
- To provide transparency and animation in a single format
- To improve page load performance

##### Syntax Rules and Structure

**General Syntax**

```html
<picture>
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="description" width="800" height="450">
</picture>
```

**Constraints and Limitations**

- Not supported in Internet Explorer
- Slightly less compression than AVIF
- Requires `<picture>` for fallback in older browsers

##### Annotated Code Example

```html
<picture>
    <source srcset="hero.webp" type="image/webp">
    <img src="hero.jpg" alt="Hero image" width="1200" height="600">
</picture>
```

**Expected Output**

Modern browsers load the WebP version, while older browsers fall back to JPEG.

**Why This Output Occurs**

The `<picture>` element provides multiple sources. The browser selects the first supported format, ensuring compatibility while delivering the best possible experience.

---

#### 5.6 AVIF (AV1 Image File Format)

##### Definitions

**Core Definition**

AVIF is a next-generation raster image format based on the AV1 video codec, offering superior compression and quality compared to older formats.

**Technical Definition**

AVIF (AV1 Image File Format) is a modern image format that offers much better compression than PNG or JPEG with support for higher colour depths, animated frames, and transparency. It is a royalty-free format developed by the Alliance for Open Media. The MIME type is `image/avif`, and the file extension is `.avif`. Support: Chrome, Edge, Firefox, Opera, Safari. When using AVIF, you should include fallbacks to formats with better browser support using the `<picture>` element.

**Beginner-Friendly Explanation**

AVIF is the newest image format. It gives you even smaller files than WebP, with great quality. It supports transparency and animation. Not all browsers support it yet, so you should always provide a fallback like JPEG or WebP.

##### Purposes

- To achieve the smallest possible file size with high quality
- To future-proof image delivery for modern browsers
- To support high colour depths and advanced features
- To improve Core Web Vitals and page performance

##### Syntax Rules and Structure

**General Syntax**

```html
<picture>
    <source srcset="image.avif" type="image/avif">
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="description" width="800" height="450">
</picture>
```

**Constraints and Limitations**

- Limited browser support compared to JPEG and PNG
- Requires fallbacks for older browsers
- Encoding can be slower than other formats

##### Annotated Code Example

```html
<picture>
    <source srcset="photo.avif" type="image/avif">
    <source srcset="photo.webp" type="image/webp">
    <img src="photo.jpg" alt="Mountain landscape" width="1200" height="800">
</picture>
```

**Expected Output**

Modern browsers load the AVIF version (smallest file), slightly older browsers load WebP, and the oldest browsers fall back to JPEG.

**Why This Output Occurs**

The `<picture>` element allows the browser to choose the best format it supports. This progressive enhancement approach ensures broad compatibility while delivering optimal performance to modern browsers.

---

### 6. Choosing the Right Image Format

#### Definitions

**Core Definition**

Choosing the right image format is the process of selecting the most appropriate file type for a given image based on its content, the need for transparency or animation, and the desired balance between file size and quality.

**Technical Definition**

Format selection involves evaluating the image content (photographic vs. graphical), the required features (transparency, animation, vector scaling), and the target browsers‘ format support. Photographic images are best served by lossy formats (JPEG, WebP, AVIF), while graphics, logos, and icons benefit from lossless formats (PNG, SVG) or modern formats with lossless modes (WebP, AVIF). The `<picture>` element enables progressive enhancement by providing multiple format sources.

**Beginner-Friendly Explanation**

Think about what your image is: a photo, a logo, or an animation. Photos should be JPEG or WebP. Logos and icons should be SVG or PNG. Animations should be WebP or AVIF if possible, with GIF as a fallback. Always provide fallbacks for newer formats so older browsers still work.

#### Purposes

- To optimise page load performance by serving appropriately sized files
- To maintain visual quality for the image‘s intended use
- To ensure compatibility across different browsers and devices
- To support features like transparency and animation when needed

#### Decision Guide

| Image Type | Recommended Format | Fallback |
|---|---|---|
| Photographs | AVIF, WebP | JPEG |
| Logos, icons | SVG | PNG |
| Images with transparency | AVIF, WebP | PNG |
| Simple animations | AVIF, WebP | GIF |
| Complex illustrations | SVG | PNG |

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Progressive Enhancement with `<picture>`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Progressive Image Enhancement</title>
</head>
<body>
    <!-- Best format first, fallbacks after -->
    <picture>
        <source srcset="photo.avif" type="image/avif">
        <source srcset="photo.webp" type="image/webp">
        <img
            src="photo.jpg"
            alt="A mountain landscape at sunset"
            width="1200"
            height="800"
            loading="lazy"
        >
    </picture>
</body>
</html>
```

**Expected Output**

Modern browsers load AVIF (smallest file), browsers that support WebP but not AVIF load WebP, and older browsers load JPEG.

**Why This Output Occurs**

The `<picture>` element evaluates each `<source>` in order and selects the first format the browser supports. The `<img>` element provides the final fallback.

---

**Example 2: SVG vs. PNG for Icons**

```html
<!-- SVG: Scalable, small file size -->
<img src="icon.svg" alt="Settings icon" width="24" height="24">

<!-- PNG: Raster fallback for older browsers -->
<img src="icon.png" alt="Settings icon" width="24" height="24">
```

**Expected Output**

The SVG icon is sharp at any size. The PNG icon is sharp at 24 × 24 pixels but becomes pixelated if scaled up.

**Why This Output Occurs**

SVG uses vector math to describe the icon, so it scales infinitely. PNG stores pixel data, so it has a fixed resolution.

#### Real-World Cases

**Case 1: Google PageSpeed Insights**

Google’s PageSpeed Insights recommends using modern formats like WebP and AVIF to improve page load performance.

**Case 2: Responsive Images**

The `srcset` attribute allows different image sizes to be served based on the viewport width.

**Case 3: Art Direction**

The `<picture>` element enables art direction — serving different crops or compositions for different screen sizes.

---

## References

- MDN Web Docs – `<img>`: The Image Embed element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img
- MDN Web Docs – Image file type and format guide – https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Formats/Image_types
- MDN Web Docs – HTML images – https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Images_in_HTML
- MDN Web Docs – Responsive images – https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Responsive_images
- WHATWG HTML Living Standard – The img element – https://html.spec.whatwg.org/multipage/embedded-content.html#the-img-element
- W3C – WCAG 2.1 Understanding Success Criterion 1.1.1: Non-text Content – https://www.w3.org/WAI/WCAG21/Understanding/non-text-content.html
- web.dev – Choose the right image format – https://web.dev/articles/choose-the-right-image-format
- web.dev – Serve responsive images – https://web.dev/articles/serve-responsive-images
- web.dev – Use WebP images – https://web.dev/articles/use-webp-images
- web.dev – Use AVIF images – https://web.dev/articles/use-avif-images