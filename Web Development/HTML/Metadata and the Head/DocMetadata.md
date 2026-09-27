# HTML Document Metadata: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML document metadata is the collection of elements placed inside the `<head>` element that describe the document, define its relationships to external resources, and provide information to browsers, search engines, and social media platforms — without being displayed in the page body.

**Technical Definition**

Document metadata encompasses the elements permitted as children of the `<head>` element in the WHATWG HTML Living Standard: `<title>`, `<meta>`, `<link>`, `<style>`, `<script>`, `<base>`, `<noscript>`, and `<template>`. These elements are categorised as metadata content. They do not render visible content (with the exception of `<title>`, which is displayed in the browser tab or window title bar) but instead provide machine-readable information about the document. The `<head>` element itself is a container for metadata content, and its content model permits one or more metadata content elements, of which exactly one `<title>` is required (unless the document is an `<iframe srcdoc>` document or the title is provided by another protocol).

**Beginner-Friendly Explanation**

Every web page has a "backstage" area called the `<head>` that the visitor never sees directly. This is where you put information *about* the page — its title (which appears in the browser tab), a description for search engines, links to CSS files, scripts, and instructions for how the page should appear when shared on social media. This information is called "metadata" — data about data. It doesn't show up on the page, but it's essential for browsers, search engines, and social platforms to understand and display your page correctly.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Placed in `<head>`** | All metadata elements belong inside the `<head>` element |
| **Not visible on the page** | Metadata does not render in the page body (except `<title>` in the browser chrome) |
| **Machine-readable** | Metadata is consumed by browsers, search engines, and social platforms |
| **Required vs. optional** | `<title>` is required; most other metadata is optional but recommended |
| **Order matters (sometimes)** | `<meta charset>` must appear within the first 1024 bytes of the document |
| **Multiple elements allowed** | You can have multiple `<meta>`, `<link>`, `<style>`, and `<script>` elements |
| **SEO and social impact** | Metadata directly affects search engine results and social media previews |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Awareness of CSS and JavaScript basics (for `<style>` and `<script>`)
- Basic knowledge of URLs (for `<link>` and `<base>`)

---

### Related Programming Areas

- **Search Engine Optimization (SEO)** – Metadata like `<title>`, `<meta name="description">`, and canonical tags directly influence search rankings
- **Social Media Marketing** – Open Graph and Twitter Cards control link previews
- **Web Performance** – `<link rel="preload">`, `<link rel="preconnect">`, and script loading strategies affect page speed
- **Web Accessibility (A11y)** – `<title>` is the first thing screen readers announce; `<html lang>` aids pronunciation
- **Security** – Content Security Policy (CSP) is delivered via `<meta http-equiv>`
- **CSS and JavaScript** – `<style>`, `<link rel="stylesheet">`, and `<script>` control presentation and behaviour

---

## Core Concepts / Features

---

### 1. The `<title>` Element

#### Definitions

**Core Definition**

The `<title>` element defines the document's title, displayed in the browser's title bar or tab and used as the clickable headline in search engine results.

**Technical Definition**

The `<title>` HTML element defines the document's title that is shown in a browser's title bar or a page's tab. It only contains text; tags within the element are ignored. It is categorised as metadata content. Its permitted content is text that is not inter-element whitespace. Both start and end tags are mandatory. Its permitted parent is the `<head>` element (or an SVG `<svg>` element). Its DOM interface is `HTMLTitleElement`. Every HTML document must have exactly one `<title>` element.

**Beginner-Friendly Explanation**

The `<title>` tag is what appears in the browser tab — like "Gmail" or "YouTube." It's also the blue clickable headline you see in Google search results. It tells users and search engines what the page is about in a few words.

#### Purposes

- To provide a human-readable title for the document
- To identify the page in browser tabs, bookmarks, and history
- To serve as the headline in search engine results
- To provide the initial announcement for screen reader users

#### Syntax Rules and Structure

**General Syntax**

```html
<head>
    <title>Page Title - Site Name</title>
</head>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<title>` | Opening tag; indicates the document title |
| `Content` | Plain text; the title (no HTML tags) |
| `</title>` | Closing tag; required |

**Syntax Rules**

- Only one `<title>` element per document
- Must be placed inside `<head>`
- Plain text only; HTML tags are ignored
- Should be concise (typically 50–60 characters for SEO)
- Should be unique per page

**Constraints and Limitations**

- Very long titles are truncated in search results and browser tabs
- The `<title>` cannot contain HTML elements
- The `<title>` element is not displayed in the page body

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Document Title**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>HTML Tables - Complete Guide</title>
</head>
<body>
    <h1>HTML Tables</h1>
    <p>Learn how to build accessible data tables.</p>
</body>
</html>
```

**Expected Output**

The browser tab displays "HTML Tables - Complete Guide."

**Why This Output Occurs**

The `<title>` element provides the document title, which browsers render in the tab or title bar.

---

**Example 2: Title with Site Name**

```html
<head>
    <title>Contact Us | Acme Corporation</title>
</head>
```

**Expected Output**

The browser tab displays "Contact Us | Acme Corporation."

**Why This Output Occurs**

The pipe character (`|`) is a common convention for separating the page title from the site name in the `<title>` element.

#### Real-World Cases

**Case 1: E-Commerce Product Pages**

`<title>Red Leather Jacket - Size M | FashionStore</title>`

**Case 2: News Articles**

`<title>Breaking: New Climate Agreement Reached | NewsDaily</title>`

**Case 3: Documentation**

`<title>Getting Started - API Reference | Developer Docs</title>`

---

### 2. The `<meta>` Element

#### Definitions

**Core Definition**

The `<meta>` element represents metadata that cannot be expressed by other HTML elements, such as the document's character encoding, description, author, and viewport settings.

**Technical Definition**

The `<meta>` HTML element represents metadata that cannot be expressed by other HTML meta-related elements (`<base>`, `<link>`, `<script>`, `<style>`, or `<title>`). It is categorised as metadata content. Its content model is nothing (it is a void element). It supports the `name`, `http-equiv`, `charset`, `content`, and `media` attributes. Its DOM interface is `HTMLMetaElement`. The `name` and `http-equiv` attributes define the type of metadata; the `content` attribute provides the value. A `<meta charset>` element must be within the first 1024 bytes of the document.

**Beginner-Friendly Explanation**

The `<meta>` tag provides extra information about your page that doesn't fit in other tags. You use it for things like "this page uses UTF-8 character encoding," "here's a short description for search engines," "here's the author's name," and "here's how this page should look on mobile."

#### Purposes

- To declare the document's character encoding
- To provide a description for search engines
- To control the viewport for responsive design
- To specify the author, keywords, and other document information
- To set HTTP-equivalent headers like `refresh` and `Content-Security-Policy`

#### Syntax Rules and Structure

**General Syntax**

```html
<meta charset="UTF-8">
<meta name="description" content="Page description">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="author" content="Author Name">
<meta http-equiv="refresh" content="30">
```

**Component Breakdown**

| Attribute | Description |
|---|---|
| `charset` | Declares the character encoding (e.g., `UTF-8`) |
| `name` | Names the metadata type (e.g., `description`, `author`) |
| `http-equiv` | HTTP-equivalent header name (e.g., `refresh`) |
| `content` | The value of the metadata |

**Common `name` Values**

| Name | Purpose |
|---|---|
| `description` | Short page description for search engines |
| `viewport` | Mobile viewport settings |
| `author` | Document author |
| `keywords` | Relevant keywords (largely ignored by modern search engines) |
| `robots` | Controls search engine indexing (`index`, `noindex`, `follow`, `nofollow`) |
| `theme-color` | Browser UI colour on mobile |

**Syntax Rules**

- `<meta>` is a void element (no closing tag)
- The `<meta charset>` element must appear within the first 1024 bytes of the document
- Only one `<meta charset>` is permitted
- The `name` and `http-equiv` attributes are mutually exclusive in practice

**Constraints and Limitations**

- The `keywords` meta tag is ignored by Google and most modern search engines
- The `description` meta tag does not directly affect rankings but influences click-through rates
- `http-equiv` has limited effect for headers that are better set server-side

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Essential Meta Tags**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <!-- Character encoding: must be first -->
    <meta charset="UTF-8">

    <!-- Viewport: essential for responsive design -->
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- Description: shown in search results -->
    <meta name="description" content="Learn HTML document metadata: title, meta tags, link elements, and more.">

    <!-- Author -->
    <meta name="author" content="Jane Developer">

    <!-- Robots: allow indexing -->
    <meta name="robots" content="index, follow">

    <!-- Theme colour for mobile browser UI -->
    <meta name="theme-color" content="#4a6cf7">

    <title>HTML Metadata Guide</title>
</head>
<body>
    <h1>HTML Metadata</h1>
</body>
</html>
```

**Expected Output**

The page renders with proper character encoding, responsive viewport behaviour, a description for search engines, and a blue theme colour in mobile browser UI.

**Why This Output Occurs**

Each `<meta>` element provides a specific piece of metadata. The `charset` ensures proper character rendering. The `viewport` enables responsive design. The `description` is used by search engines. The `theme-color` colours the mobile browser's address bar.

---

**Example 2: HTTP-Equivalent Refresh**

```html
<head>
    <meta charset="UTF-8">
    <meta http-equiv="refresh" content="5; url=https://example.com/new-page">
    <title>Redirecting...</title>
</head>
```

**Expected Output**

After 5 seconds, the browser navigates to `https://example.com/new-page`.

**Why This Output Occurs**

The `http-equiv="refresh"` attribute with a `content` value of `"5; url=..."` instructs the browser to refresh and redirect after 5 seconds.

#### Real-World Cases

**Case 1: E-Commerce Product Pages**

```html
<meta name="description" content="Buy the Red Leather Jacket. Free shipping over $50.">
```

**Case 2: News Articles**

```html
<meta property="og:description" content="Breaking news: Climate agreement reached in Paris.">
```

**Case 3: Single-Page Applications**

```html
<meta name="theme-color" content="#1a1a1a">
```

---

### 3. The `<link>` Element

#### Definitions

**Core Definition**

The `<link>` element establishes a relationship between the current document and an external resource, most commonly used to link stylesheets and icons.

**Technical Definition**

The `<link>` HTML element specifies relationships between the current document and an external resource. It is categorised as metadata content, and if it is allowed in the body, flow content and phrasing content. Its content model is nothing (void element). It supports `href`, `rel`, `media`, `type`, `as`, `crossorigin`, `disabled`, `hreflang`, `integrity`, `referrerpolicy`, and `sizes`. Its DOM interface is `HTMLLinkElement`. The `rel` attribute defines the relationship type.

**Beginner-Friendly Explanation**

The `<link>` tag connects your HTML page to external files — like a stylesheet (CSS), a favicon (the little icon in the browser tab), or a font. It's like a reference list for your page.

#### Purposes

- To link external stylesheets to the document
- To define the document's favicon and touch icons
- To establish canonical URLs for SEO
- To preload, prefetch, or preconnect to external resources
- To provide alternate versions of the document (e.g., RSS feeds, translations)

#### Syntax Rules and Structure

**General Syntax**

```html
<link rel="stylesheet" href="styles.css">
<link rel="icon" href="/favicon.ico" type="image/x-icon">
<link rel="canonical" href="https://example.com/page">
```

**Common `rel` Values**

| Value | Purpose |
|---|---|
| `stylesheet` | Links a CSS file |
| `icon` | Defines the favicon |
| `canonical` | Preferred URL for the page (SEO) |
| `preload` | Preloads a resource for the current page |
| `prefetch` | Prefetches a resource for a future page |
| `preconnect` | Preconnects to an origin |
| `alternate` | Alternate version (e.g., RSS, translation) |
| `manifest` | Web app manifest |

**Syntax Rules**

- `<link>` is a void element (no closing tag)
- The `href` attribute is required for most `rel` values
- The `rel` attribute is required
- Multiple `<link>` elements are allowed

**Constraints and Limitations**

- The `<link>` element does not display content
- Invalid `rel` values may be ignored
- Some `rel` values are only valid in `<head>`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Stylesheet and Favicon**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- External stylesheet -->
    <link rel="stylesheet" href="/css/styles.css">

    <!-- Favicon -->
    <link rel="icon" href="/favicon.ico" type="image/x-icon">

    <!-- Apple touch icon -->
    <link rel="apple-touch-icon" href="/apple-touch-icon.png">

    <title>My Website</title>
</head>
<body>
    <h1>Welcome</h1>
</body>
</html>
```

**Expected Output**

The page loads the external CSS file, displays the favicon in the browser tab, and uses the Apple touch icon when added to an iOS home screen.

**Why This Output Occurs**

Each `<link>` element with its specific `rel` value tells the browser what external resource to load and how to use it.

---

**Example 2: Preload and Preconnect**

```html
<head>
    <!-- Preconnect to a font server -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

    <!-- Preload a critical font -->
    <link rel="preload" href="/fonts/body.woff2" as="font" type="font/woff2" crossorigin>

    <!-- Preload the hero image -->
    <link rel="preload" href="/images/hero.webp" as="image">

    <link rel="stylesheet" href="/css/styles.css">
</head>
```

**Expected Output**

The browser establishes connections to the font servers earlier, and the font and hero image are loaded with higher priority.

**Why This Output Occurs**

`rel="preconnect"` initiates early connections; `rel="preload"` tells the browser to fetch critical resources as soon as possible.

#### Real-World Cases

**Case 1: E-Commerce**

```html
<link rel="canonical" href="https://shop.example.com/products/red-jacket">
```

**Case 2: Blogs**

```html
<link rel="alternate" type="application/rss+xml" title="RSS Feed" href="/feed.xml">
```

**Case 3: Progressive Web Apps**

```html
<link rel="manifest" href="/manifest.webmanifest">
```

---

### 4. The `<style>` Element

#### Definitions

**Core Definition**

The `<style>` element contains internal CSS rules that apply to the document.

**Technical Definition**

The `<style>` HTML element contains style information for a document, or part of a document. It contains CSS, which is applied to the contents of the document containing the `<style>` element. It is categorised as metadata content (when in `<head>`) or flow content/phrasing content (when in `<body>`). Its permitted content is text that matches the CSS grammar. It supports `media`, `blocking`, `nonce`, and `title` attributes. Its DOM interface is `HTMLStyleElement`.

**Beginner-Friendly Explanation**

The `<style>` tag is where you put CSS directly inside your HTML file. Instead of linking to an external stylesheet, you write the CSS rules right there in the `<head>`.

#### Purposes

- To apply internal CSS without an external file
- To define page-specific styles
- To scope styles to a specific media query
- To reduce HTTP requests for critical styles

#### Syntax Rules and Structure

```html
<style>
    body { font-family: sans-serif; }
    h1 { color: #333; }
</style>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<style>` | Opening tag; contains CSS |
| `media` | Optional media query |
| `Content` | CSS rules |
| `</style>` | Closing tag; required |

**Syntax Rules**

- The `<style>` element must have a closing tag
- Multiple `<style>` elements are allowed
- The `media` attribute applies the styles only when the media query matches
- The `<style>` element is preferred over inline styles for maintainability

**Constraints and Limitations**

- External stylesheets are generally preferred for caching across pages
- Very large `<style>` blocks increase HTML file size

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Internal Stylesheet**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Internal Styles Demo</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 2em;
        }
        h1 {
            color: #1a1a1a;
            border-bottom: 2px solid #4a6cf7;
        }
    </style>
</head>
<body>
    <h1>Welcome</h1>
    <p>This page uses internal CSS.</p>
</body>
</html>
```

**Expected Output**

The page renders with Arial font, padding, and a blue-underlined heading.

**Why This Output Occurs**

The CSS rules inside `<style>` apply to the document.

---

**Example 2: Media-Specific Styles**

```html
<style media="print">
    body { font-size: 12pt; }
    nav, footer { display: none; }
</style>
```

**Expected Output**

The styles apply only when printing.

**Why This Output Occurs**

The `media="print"` attribute restricts the styles to print media.

#### Real-World Cases

**Case 1: Critical CSS**

```html
<style>/* above-the-fold critical CSS */</style>
```

**Case 2: Email Templates**

Emails use `<style>` because external stylesheets are often blocked.

**Case 3: Single-Page Demos**

CodePen-style demos use internal styles for self-contained examples.

---

### 5. The `<script>` Element

#### Definitions

**Core Definition**

The `<script>` element embeds or references executable client-side scripts, typically JavaScript.

**Technical Definition**

The `<script>` HTML element is used to embed executable code or data; this is typically used to embed or refer to JavaScript code. It is categorised as metadata content, flow content, and phrasing content. Its content model is dynamic: if the `src` attribute is absent, the content is script text; if `src` is present, the content must be empty or whitespace. It supports `src`, `type`, `async`, `defer`, `crossorigin`, `integrity`, `nomodule`, `referrerpolicy`, `blocking`, and `fetchpriority`. Its DOM interface is `HTMLScriptElement`.

**Beginner-Friendly Explanation**

The `<script>` tag is how you add JavaScript to your page. You can write the JavaScript directly inside the tag, or link to an external `.js` file. JavaScript makes your page interactive.

#### Purposes

- To embed or reference JavaScript code
- To add interactivity to the page
- To load third-party libraries and analytics
- To execute code before or after the page renders

#### Syntax Rules and Structure

**Inline Script**

```html
<script>
    console.log('Hello, world!');
</script>
```

**External Script**

```html
<script src="/js/app.js"></script>
```

**Deferred Script**

```html
<script src="/js/app.js" defer></script>
```

**Async Script**

```html
<script src="https://example.com/analytics.js" async></script>
```

**Component Breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of external script |
| `async` | Download in parallel, execute as soon as ready |
| `defer` | Download in parallel, execute after parsing |
| `type` | MIME type (default: `text/javascript`) |
| `nomodule` | Fallback for browsers without module support |

**Syntax Rules**

- The `<script>` element requires a closing tag
- Inline scripts must not contain `</script>` in the code (escape it as `<\/script>`)
- `async` and `defer` only apply to external scripts
- Module scripts (`type="module"`) are deferred by default

**Constraints and Limitations**

- Synchronous scripts block HTML parsing
- `async` scripts execute in unpredictable order
- `defer` scripts execute in document order after parsing

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Deferred External Script**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Script Loading Demo</title>

    <!-- Deferred: downloaded in parallel, executed after parsing -->
    <script src="/js/app.js" defer></script>
</head>
<body>
    <h1>Hello</h1>
    <p>The script loads without blocking.</p>
</body>
</html>
```

**Expected Output**

The page renders immediately; the script executes after the HTML is parsed.

**Why This Output Occurs**

The `defer` attribute delays execution until after parsing, improving perceived performance.

---

**Example 2: Inline Script with DOMContentLoaded**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Inline Script Demo</title>
</head>
<body>
    <button id="btn">Click me</button>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            document.getElementById('btn').addEventListener('click', function() {
                alert('Button clicked!');
            });
        });
    </script>
</body>
</html>
```

**Expected Output**

Clicking the button shows an alert.

**Why This Output Occurs**

The `DOMContentLoaded` event ensures the script runs after the HTML is parsed.

#### Real-World Cases

**Case 1: Analytics**

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_ID"></script>
```

**Case 2: Interactive Widgets**

```html
<script src="/js/carousel.js" defer></script>
```

**Case 3: Module Scripts**

```html
<script type="module" src="/js/app.mjs"></script>
```

---

### 6. The `<base>` Element

#### Definitions

**Core Definition**

The `<base>` element specifies the base URL and/or default target for all relative URLs in a document.

**Technical Definition**

The `<base>` HTML element specifies the base URL to use for all relative URLs in a document. There can be only one `<base>` element in a document. It is categorised as metadata content. It is a void element. It supports `href` and `target` attributes. Its DOM interface is `HTMLBaseElement`. If the `href` attribute is present, it must be an absolute URL. If multiple `<base>` elements are present, only the first is used; the rest are ignored.

**Beginner-Friendly Explanation**

The `<base>` tag sets a "starting point" for all relative links in your page. If you set `<base href="https://example.com/blog/">`, then a link like `<a href="post-1.html">` goes to `https://example.com/blog/post-1.html`.

#### Purposes

- To define a base URL for all relative URLs
- To set a default target for all links and forms
- To simplify internal links in deeply nested pages

#### Syntax Rules and Structure

```html
<base href="https://example.com/" target="_blank">
```

**Component Breakdown**

| Attribute | Description |
|---|---|
| `href` | The base URL for all relative URLs |
| `target` | Default browsing context for links and forms |

**Syntax Rules**

- Only one `<base>` element per document
- Must be placed inside `<head>`
- The `href` must be an absolute URL
- The `<base>` element must appear before any element that uses a relative URL

**Constraints and Limitations**

- Multiple `<base>` elements: only the first is used
- The `<base>` element affects fragment links and form actions
- The `<base>` element can break relative links if set incorrectly

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Base URL**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <base href="https://example.com/blog/">
    <title>Base URL Demo</title>
</head>
<body>
    <!-- Resolves to https://example.com/blog/post-1.html -->
    <a href="post-1.html">Post 1</a>

    <!-- Resolves to https://example.com/blog/images/hero.jpg -->
    <img src="images/hero.jpg" alt="Hero">
</body>
</html>
```

**Expected Output**

All relative URLs resolve against `https://example.com/blog/`.

**Why This Output Occurs**

The `<base href>` sets the base for all relative URLs.

---

**Example 2: Base Target**

```html
<head>
    <base target="_blank">
</head>
<body>
    <!-- Opens in a new tab -->
    <a href="https://example.com">External Link</a>
</body>
```

**Expected Output**

Clicking the link opens it in a new tab.

**Why This Output Occurs**

The `target` attribute on `<base>` sets the default browsing context for all links.

#### Real-World Cases

**Case 1: Documentation Sites**

Documentation hosted in subdirectories uses `<base>` to simplify links.

**Case 2: Email Templates**

Email templates sometimes use `<base>` to control link behaviour.

**Case 3: Legacy Applications**

Older applications use `<base>` for consistent URL resolution.

---

### 7. Open Graph and Twitter Cards

#### Definitions

**Core Definition**

Open Graph and Twitter Cards are `<meta>` property systems that control how a URL appears when shared on social media platforms.

**Technical Definition**

Open Graph is a protocol introduced by Facebook that uses `<meta property="og:...">` elements to describe a page's title, description, image, URL, and type. Twitter Cards use `<meta name="twitter:...">` elements. Both are placed inside the `<head>` element. The `og:title`, `og:description`, `og:image`, and `og:url` properties are the most commonly used. Twitter Cards fall back to Open Graph if `twitter:*` tags are absent.

**Beginner-Friendly Explanation**

When you share a link on Facebook, Twitter, or LinkedIn, you see a preview with a title, description, and image. Open Graph and Twitter Cards are the tags that control what that preview looks like. Without them, the platform guesses — often badly.

#### Purposes

- To control the title, description, and image shown in social media previews
- To improve click-through rates from social media
- To ensure consistent branding across platforms
- To provide structured data for social platforms

#### Syntax Rules and Structure

**Open Graph**

```html
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Page description">
<meta property="og:image" content="https://example.com/image.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">
```

**Twitter Cards**

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Page description">
<meta name="twitter:image" content="https://example.com/image.jpg">
```

**Common Open Graph Properties**

| Property | Purpose |
|---|---|
| `og:title` | Title in the preview |
| `og:description` | Description in the preview |
| `og:image` | Image URL (recommended 1200×630) |
| `og:url` | Canonical URL |
| `og:type` | Content type (`website`, `article`, `product`) |
| `og:site_name` | Site name |

**Syntax Rules**

- Open Graph uses `property` attribute; Twitter Cards use `name`
- Both are placed inside `<head>`
- `og:image` should be an absolute URL
- Twitter Cards fall back to Open Graph if missing

**Constraints and Limitations**

- Platforms cache previews; changes may not appear immediately
- Image dimensions and aspect ratios vary by platform
- Some platforms have their own additional tags (e.g., LinkedIn)

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Full Open Graph and Twitter Cards**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Article Title - My Site</title>

    <!-- Open Graph -->
    <meta property="og:title" content="Article Title">
    <meta property="og:description" content="A short description of the article.">
    <meta property="og:image" content="https://example.com/images/article.jpg">
    <meta property="og:url" content="https://example.com/article">
    <meta property="og:type" content="article">
    <meta property="og:site_name" content="My Site">

    <!-- Twitter Cards -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:title" content="Article Title">
    <meta name="twitter:description" content="A short description of the article.">
    <meta name="twitter:image" content="https://example.com/images/article.jpg">
</head>
<body>
    <h1>Article Title</h1>
</body>
</html>
```

**Expected Output**

When shared on Facebook or Twitter, the link shows the title, description, and image specified.

**Why This Output Occurs**

The social platform reads the Open Graph and Twitter Card meta tags to construct the preview.

#### Real-World Cases

**Case 1: News Articles**

News sites use `og:type="article"` with publication dates.

**Case 2: E-Commerce**

Product pages use `og:type="product"` with price and availability.

**Case 3: Blogs**

Blogs use `og:type="article"` with author and section.

---

### 8. Canonical Tags

#### Definitions

**Core Definition**

A canonical tag is a `<link rel="canonical">` element that specifies the preferred URL for a page, preventing duplicate content issues.

**Technical Definition**

The `rel="canonical"` link type indicates the preferred URL for the current document. Google's documentation states: "A canonical URL is the URL of the page that Google thinks is most representative from a set of duplicate pages on your site." The `<link rel="canonical">` element must be placed in the `<head>` element and must contain an absolute URL. It helps search engines consolidate signals and avoid duplicate content penalties.

**Beginner-Friendly Explanation**

Sometimes the same content is available at multiple URLs — like `example.com/page`, `example.com/page?ref=twitter`, and `example.com/page/`. A canonical tag tells search engines "this is the real URL — please use this one and ignore the others."

#### Purposes

- To prevent duplicate content issues in search engines
- To consolidate link equity to a single URL
- To specify the preferred URL for syndicated or parameterised content
- To improve SEO by indicating the authoritative page

#### Syntax Rules and Structure

```html
<link rel="canonical" href="https://example.com/preferred-url">
```

**Component Breakdown**

| Component | Description |
|---|---|
| `rel="canonical"` | Indicates the canonical relationship |
| `href` | Absolute URL of the preferred page |

**Syntax Rules**

- Must be placed inside `<head>`
- The `href` must be an absolute URL
- Only one canonical URL per page
- The canonical URL should return HTTP 200 (not 404 or redirect)

**Constraints and Limitations**

- Canonical tags are hints, not directives
- Incorrect canonical tags can deindex pages
- Cross-domain canonicals are allowed but require care

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Canonical for a Product Page**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Red Leather Jacket - FashionStore</title>
    <link rel="canonical" href="https://shop.example.com/products/red-leather-jacket">
</head>
<body>
    <h1>Red Leather Jacket</h1>
</body>
</html>
```

**Expected Output**

Search engines treat `https://shop.example.com/products/red-leather-jacket` as the canonical URL, even if the page is accessed via `?ref=twitter` or a tracking parameter.

**Why This Output Occurs**

The canonical link tells search engines which URL is the preferred version.

---

**Example 2: Canonical for Syndicated Content**

```html
<head>
    <link rel="canonical" href="https://original-site.com/article">
</head>
```

**Expected Output**

Search engines credit the original site for the content.

**Why This Output Occurs**

The cross-domain canonical indicates the original source.

#### Real-World Cases

**Case 1: E-Commerce Product Variants**

`/products/red-jacket?size=m` canonicalises to `/products/red-jacket`.

**Case 2: Paginated Content**

Each page of a paginated series has its own canonical URL.

**Case 3: Syndicated News**

Syndicated articles canonicalise to the original publisher.

---

### 9. Choosing the Right Metadata Approach

#### Definitions

**Core Definition**

Choosing the right metadata approach means selecting the appropriate combination of `<title>`, `<meta>`, `<link>`, `<style>`, `<script>`, and `<base>` elements to describe the document, control its presentation, and optimise it for search and social media.

**Technical Definition**

Metadata selection depends on the document's purpose, audience, and platform requirements. Every document requires a `<title>` and a `<meta charset>`. Responsive pages require `<meta name="viewport">`. SEO-conscious pages require `<meta name="description">` and `<link rel="canonical">`. Social sharing requires Open Graph and Twitter Card tags. Performance-critical pages benefit from `<link rel="preload">` and `<link rel="preconnect">`.

**Beginner-Friendly Explanation**

Different pages need different metadata. A simple article needs a title, description, and maybe social sharing tags. An e-commerce product page needs those plus canonical tags and structured data. Think about what the page is for and add the metadata that helps it succeed.

#### Decision Guide

| Page Type | Essential Metadata |
|---|---|
| **Any page** | `<title>`, `<meta charset>`, `<meta viewport>` |
| **SEO-focused** | `<meta description>`, `<link canonical>`, `<meta robots>` |
| **Social sharing** | Open Graph, Twitter Cards |
| **Performance-critical** | `<link preload>`, `<link preconnect>`, `<script defer>` |
| **Progressive Web App** | `<link manifest>`, `<meta theme-color>` |
| **Article/news** | `og:type="article"`, publication dates |

---

## References

- MDN Web Docs – `<title>`: The Document Title element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/title
- MDN Web Docs – `<meta>`: The metadata element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta
- MDN Web Docs – `<link>`: The External Resource Link element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/link
- MDN Web Docs – `<style>`: The Style Information element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/style
- MDN Web Docs – `<script>`: The Script element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script
- MDN Web Docs – `<base>`: The Base URL element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/base
- MDN Web Docs – `<head>`: The Document Metadata (Header) element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/head
- MDN Web Docs – Standard metadata names – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta/name
- WHATWG HTML Living Standard – The head element – https://html.spec.whatwg.org/multipage/semantics.html#the-head-element
- WHATWG HTML Living Standard – The title element – https://html.spec.whatwg.org/multipage/semantics.html#the-title-element
- WHATWG HTML Living Standard – The meta element – https://html.spec.whatwg.org/multipage/semantics.html#the-meta-element
- WHATWG HTML Living Standard – The link element – https://html.spec.whatwg.org/multipage/semantics.html#the-link-element
- OG.me – Open Graph Protocol – https://ogp.me/
- X Developer Platform – Cards – https://developer.x.com/en/docs/x-for-websites/cards/overview/abouts-cards
- Google Search Central – Canonical URLs – https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls
- Google Search Central – Meta tags and attributes – https://developers.google.com/search/docs/crawling-indexing/special-tags
- Google Search Central – Title links – https://developers.google.com/search/docs/appearance/title-link
- Google Search Central – Snippets – https://developers.google.com/search/docs/appearance/snippet
- web.dev – Document metadata – https://web.dev/learn/html/metadata