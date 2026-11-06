# Document Metadata Elements — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Document metadata elements are the HTML elements that describe the document itself — its title, character encoding, relationships to external resources, embedded styles, scripts, and fallback content — rather than its visible body content. They reside primarily inside the `<head>` element and, together with the root `<html>` element, form the document's machine-readable envelope.

**Technical Definition**
In the WHATWG HTML Living Standard, document metadata is represented by elements belonging to the **metadata content category**. These elements modify the presentation or behaviour of the rest of the document, set up links to other documents, or convey out-of-band information. The metadata elements are `<base>`, `<link>`, `<meta>`, `<noscript>`, `<script>`, `<style>`, and `<title>`. The `<head>` element collects this metadata, and the `<html>` element is the root of the entire document. Some metadata elements, notably `<script>` and `<noscript>`, also belong to the flow, phrasing, and script-supporting content categories and may therefore appear in the body under defined conditions.

**Beginner-Friendly Explanation**
Metadata elements are the “information about the page” rather than the page’s visible content. They tell the browser what the page is called, which language it uses, which stylesheet to load, which scripts to run, and how to behave if scripting is turned off. Most of them live in the `<head>` — the part of the HTML file you do not see on the page itself.

---

### Key Characteristics

- **Non-visible by default**: Metadata elements do not render as ordinary page content in the viewport (with conditional exceptions such as `<noscript>` fallback content).
- **Primarily head-scoped**: `<head>`, `<title>`, `<base>`, `<link>`, `<meta>`, and `<style>` are designed for the document head.
- **Multi-category members**: `<script>`, `<noscript>`, and `<style>` can appear in multiple content categories and contexts.
- **Relationship-oriented**: `<link>` and `<base>` define relationships between the document and external resources or URL resolution contexts.
- **Order-sensitive in cases**: `<base>` must precede elements that use relative URLs; `<meta charset>` should appear early for reliable encoding detection.
- **Governed by the HTML Living Standard**: The authoritative content models, attributes, and conformance rules are maintained by WHATWG.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<!DOCTYPE html>`, `<html>`, `<head>`, and `<body>`.
2. **How to create and open an `.html` file** in a browser.
3. **Browser developer tools**: the Elements panel for inspecting the `<head>` and the Network panel for observing resource loading.
4. **Introductory CSS and JavaScript awareness**: enough to understand what stylesheets and scripts do, even if you are not yet writing them.
5. **Knowledge of character encoding concepts**: a basic understanding that text is stored as bytes and must be decoded correctly.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS** | `<style>` and `<link rel="stylesheet">` deliver styles; `<style media>` controls conditional application. |
| **JavaScript** | `<script>` loads and executes code; `<noscript>` provides fallback behaviour. |
| **Web performance** | `<link rel="preload">`, `rel="preconnect"`, and `<script defer/async>` directly affect loading priorities. |
| **SEO** | `<title>` and `<meta name="description">` influence search-result presentation and ranking signals. |
| **Accessibility** | `<title>`, `lang`, and `<noscript>` fallbacks determine how assistive technologies announce and degrade pages. |
| **Security** | `<meta http-equiv="Content-Security-Policy">`, `integrity` on scripts and links, and `referrerpolicy` are security-relevant. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<html>`** — the document root
2. **`<head>`** — the metadata container
3. **`<title>`** — the document title
4. **`<base>`** — the base URL and default browsing context
5. **`<link>`** — external resource relationships
6. **`<meta>`** — generic metadata
7. **`<style>`** — embedded style information
8. **`<script>`** — embedded or referenced scripts
9. **`<noscript>`** — script-disabled fallback content
10. **Metadata element best practices**

---

## 1. `<html>` — The Document Root Element

### Definitions

**Core Definition**
The `<html>` element represents the root (top-level element) of an HTML document. All other elements must be descendants of this element, and there can be only one `<html>` element per document.

**Technical Definition**
In the WHATWG standard, `<html>` has no content category and no permitted parent; its content model is **one `<head>` element followed by one `<body>` element**. It exposes the `HTMLHtmlElement` DOM interface and accepts global attributes. Authors are strongly encouraged to specify a `lang` attribute containing a valid BCP 47 language tag, which aids speech synthesis, translation tools, and screen readers.

**Beginner-Friendly Explanation**
The `<html>` element is the outermost container of the entire page. Everything else — the head and the body — goes inside it. It is like the cover of a book: it holds all the pages together.

---

### Purposes

- **To declare the root of the document**: every other element must be a descendant of `<html>`.
- **To specify the document’s language**: the `lang` attribute tells browsers, screen readers, and translation tools which language the content is primarily in.
- **To provide a single container for `<head>` and `<body>`**: the two major document divisions.
- **To support XML/XHTML compatibility**: the `xmlns` attribute can be used when the document is parsed with XML parsers.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- metadata -->
  </head>
  <body>
    <!-- visible content -->
  </body>
</html>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<!DOCTYPE html>` | The document type declaration, which triggers standards mode. |
| `<html lang="en">` | The root element. `lang` is a global attribute containing a BCP 47 language tag. |
| `<head>…</head>` | The metadata container; must be the first child of `<html>`. |
| `<body>…</body>` | The visible content container; must follow `<head>`. |
| `</html>` | The end tag; may be omitted if not immediately followed by a comment. |

#### Syntax Rules

1. **Exactly one `<html>` element** per document.
2. **`<head>` followed by `<body>`** is the required child order.
3. **`lang` should be present** with a valid BCP 47 language tag; this is a strong accessibility recommendation.
4. **Tag omission is permitted** in text/html under specific conditions, but explicit start and end tags are recommended for clarity and for attaching `lang`.
5. **`xmlns` is required only for XML parsing**; it is optional in `text/html` documents.

#### Constraints and Limitations

- `<html>` must not be used as a styling hook for ordinary layout; use `<body>` or inner containers instead.
- The `version` attribute is obsolete and unnecessary because version information is carried by the DOCTYPE.
- Omitting the `lang` attribute is legal but reduces accessibility and can cause screen readers to use the operating system’s default language, leading to mispronunciation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Minimal Document Root with Language Declaration

**Setup guide**: Save as `root.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Root Element Example</title>
</head>
<body>
  <p>This document is declared as English.</p>
</body>
</html>
```

**Expected output**: The page displays the sentence “This document is declared as English.” A screen reader will use English pronunciation rules because `lang="en"` is present on `<html>`.

**Why this output occurs**: The `<html>` element wraps the entire document and carries the `lang` attribute. The browser propagates the language to descendant text nodes unless an inner element overrides it. The `lang` attribute does not produce visible text but influences speech synthesis and translation.

---

#### Example 2: A Document Root with Explicit XML Namespace

**Setup guide**: Save as `root-xhtml.html` and serve it with an XHTML MIME type if possible; otherwise, inspect the source.

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="en">
<head>
  <title>XHTML-Compatible Root</title>
</head>
<body>
  <p>This document declares the XHTML namespace on the root element.</p>
</body>
</html>
```

**Expected output**: In a browser, the page renders normally. In an XML parser, the `xmlns` attribute ensures that the elements are interpreted in the XHTML namespace.

**Why this output occurs**: The `xmlns` attribute specifies the XML namespace `http://www.w3.org/1999/xhtml`. This is required when the document is parsed as XML (e.g. application/xhtml+xml) and optional in ordinary text/html parsing.

---

### Real-World Cases

1. **Multilingual sites**: The `lang` attribute on `<html>` is the foundation of language declaration. A site serving English and French pages sets `lang="en"` or `lang="fr"` on the root, and may override it on individual elements containing quotations in another language.
2. **Accessibility compliance**: WCAG requires that the default human language of each page be programmatically determinable; the `lang` attribute on `<html>` satisfies this requirement.
3. **XHTML/XML workflows**: When HTML is processed as XML (e.g. in some publishing pipelines), the `xmlns` attribute on `<html>` is necessary for namespace-aware processing.

---

### References

- WHATWG — The `html` element — https://html.spec.whatwg.org/multipage/semantics.html#the-html-element
- MDN Web Docs — `<html>`: The HTML Document / Root element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/html
- W3C — Language tags in HTML and XML — https://www.w3.org/International/articles/language-tags/

---

## 2. `<head>` — The Document Metadata Container

### Definitions

**Core Definition**
The `<head>` element contains machine-readable information (metadata) about the document, such as its title, scripts, and style sheets. There can be only one `<head>` element per HTML document.

**Technical Definition**
In the WHATWG standard, `<head>` has no content category and must be the first child of `<html>`. Its content model is **one or more metadata content elements, of which exactly one must be a `<title>`** (except in special cases such as `iframe srcdoc` documents or HTML email where title information comes from a higher-level protocol). The `<head>` element exposes the `HTMLHeadElement` DOM interface and accepts global attributes.

**Beginner-Friendly Explanation**
The `<head>` is the “control room” of the page. Nothing inside it is directly displayed in the browser viewport, but it tells the browser what the page is called, how to display it, and what resources to load.

---

### Purposes

- **To contain the document title**: `<title>` is required inside `<head>` in conforming documents.
- **To link external resources**: stylesheets, icons, and preloaded assets via `<link>`.
- **To declare character encoding and viewport behaviour**: via `<meta>`.
- **To embed or reference styles and scripts**: via `<style>` and `<script>`.
- **To provide fallback content for script-disabled environments**: via `<noscript>`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document Title</title>
  <base href="https://example.com/">
  <link rel="stylesheet" href="styles.css">
  <style>
    /* internal CSS */
  </style>
  <script src="app.js" defer></script>
  <noscript>
    <link rel="stylesheet" href="no-script.css">
  </noscript>
</head>
```

**Component breakdown**

| Child element | Role in `<head>` |
|---|---|
| `<meta charset>` | Declares character encoding. |
| `<meta name="viewport">` | Configures viewport for mobile devices. |
| `<title>` | Sets the document title; exactly one is required. |
| `<base>` | Sets the base URL and/or default browsing context. |
| `<link>` | Links external stylesheets, icons, and other resources. |
| `<style>` | Embeds CSS rules. |
| `<script>` | Embeds or references JavaScript. |
| `<noscript>` | Provides fallback `<link>`, `<style>`, or `<meta>` elements when scripting is disabled. |

#### Syntax Rules

1. **Exactly one `<title>`** must be present in a conforming `<head>` (with narrow exceptions for `srcdoc` and HTML email).
2. **`<head>` must be the first child of `<html>`**.
3. **Metadata content is permitted**; flow content and body-level elements are not permitted inside `<head>`.
4. **`<base>` must come before URL-bearing elements** if it has an `href` attribute.
5. **`<noscript>` inside `<head>`** may contain only `<link>`, `<style>`, and `<meta>` elements when scripting is disabled.

#### Constraints and Limitations

- `<head>` does not render visible content; do not place text or flow content directly inside it.
- The `profile` attribute on `<head>` is obsolete and should not be used.
- Omitting `<head>` start and end tags is permitted in limited cases, but explicit tags are recommended for clarity.
- `<noscript>` in `<head>` has a restricted content model; it cannot contain arbitrary flow content.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Typical `<head>` for a Modern Web Page

**Setup guide**: Save as `typical-head.html` and open it in a browser. Use the developer tools to inspect the `<head>`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Character encoding: must appear early -->
  <meta charset="UTF-8">

  <!-- Viewport: enables responsive layout on mobile -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Document title: appears in the browser tab -->
  <title>Typical Head Example</title>

  <!-- External stylesheet -->
  <link rel="stylesheet" href="styles.css">

  <!-- Favicon -->
  <link rel="icon" href="favicon.ico" sizes="any">

  <!-- Deferred script: executes after HTML parsing -->
  <script src="app.js" defer></script>
</head>
<body>
  <h1>The head is not visible, but it shapes this page.</h1>
</body>
</html>
```

**Expected output**: The browser tab shows “Typical Head Example”. The page applies styles from `styles.css` and displays the heading. The script `app.js` executes after parsing, and the favicon appears in the tab.

**Why this output occurs**: Each metadata element performs its designated role. `<meta charset>` ensures correct decoding, `<meta name="viewport">` configures mobile layout, `<title>` sets the tab label, `<link>` loads CSS and the icon, and `<script defer>` loads JavaScript without blocking parsing.

---

#### Example 2: `<noscript>` Inside `<head>` for Script-Disabled Styling

**Setup guide**: Save as `noscript-head.html`. Disable JavaScript in the browser settings and reload to observe the fallback.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Noscript in Head</title>

  <!-- Styles applied when scripting is enabled -->
  <style>
    body { background: #ffffff; color: #222; }
  </style>

  <!-- Fallback styles applied only when scripting is disabled -->
  <noscript>
    <style>
      body { background: #fffbe6; color: #663300; }
    </style>
    <link rel="stylesheet" href="no-script-fallback.css">
  </noscript>
</head>
<body>
  <p>If JavaScript is disabled, a warm background colour and an extra stylesheet are applied.</p>
</body>
</html>
```

**Expected output**: With JavaScript enabled, the page has a white background. With JavaScript disabled, the `<noscript>` content is activated: the warm background colour and the external fallback stylesheet are applied.

**Why this output occurs**: When scripting is disabled and `<noscript>` is a descendant of `<head>`, its children are parsed as actual `<style>` and `<link>` elements, and their effects take hold. When scripting is enabled, the `<noscript>` children are treated as text and are not applied.

---

### Real-World Cases

1. **Every web page**: `<head>` is the universal container for title, encoding, viewport, styles, and scripts.
2. **Progressive enhancement**: `<noscript>` in the head can load a fallback stylesheet or meta refresh when JavaScript is unavailable.
3. **Performance-critical pages**: `<link rel="preload">` and `<link rel="preconnect">` inside `<head>` allow the browser to start fetching critical resources before the body is parsed.

---

### References

- WHATWG — The `head` element — https://html.spec.whatwg.org/multipage/semantics.html#the-head-element
- MDN Web Docs — `<head>`: The Document Metadata (Header) element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/head
- MDN Web Docs — What’s in the head? Web page metadata — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Webpage_metadata

---

## 3. `<title>` — The Document Title Element

### Definitions

**Core Definition**
The `<title>` element defines the document’s title, which is shown in the browser’s title bar or page tab. It contains only text; any HTML tags inside it are treated as plain text.

**Technical Definition**
`<title>` belongs to the **metadata content** category and is required inside `<head>` in conforming documents. Its content model is **text only** (no child elements). The title is used by browsers for tab labels and bookmarks, by search engines as a ranking and display signal, and by assistive technologies as the primary label for the page.

**Beginner-Friendly Explanation**
The `<title>` is the name of the page as it appears on the browser tab. It is not the big heading on the page — that is usually an `<h1>`. The `<title>` is metadata that helps users and search engines identify the page before they even open it.

---

### Purposes

- **To label the browser tab or window**: the most visible function of `<title>`.
- **To provide the page name in bookmarks and history**: browsers use the title as the default bookmark name.
- **To influence search-engine result presentation**: the title is a key component of search snippets and ranking algorithms.
- **To support assistive-technology navigation**: screen-reader users often read the page title first to decide whether the page is relevant.
- **To signal page-state changes**: the title can be updated dynamically to reflect form errors or significant state changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<head>
  <title>Primary Purpose of the Page — Site Name</title>
</head>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<title>` | The start tag; must appear inside `<head>`. |
| Text content | A descriptive phrase; HTML tags are treated as literal text. |
| `</title>` | The end tag. |

#### Syntax Rules

1. **Exactly one `<title>` per document** (with narrow exceptions for `srcdoc` and HTML email).
2. **Text only**: element children are not parsed as elements; they are treated as text.
3. **Place inside `<head>`**: the title must be a child of `<head>`.
4. **No keyword stuffing**: search engines may penalise titles that are merely lists of keywords.
5. **Keep titles unique within a site**: duplicate or near-duplicate titles can produce inaccurate search results.

#### Constraints and Limitations

- Titles longer than roughly **55–60 characters** may be truncated in search results; important information should appear early.
- Titles should not be one or two words; descriptive phrases are preferred.
- The `<title>` element is not a substitute for the page’s visible `<h1>`; they serve different purposes.
- Dynamically updating the title does not automatically announce the change to screen readers; ARIA live regions may be needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Descriptive, Uniquely Scoped Title

**Setup guide**: Save as `title-good.html` and open it in a browser. Observe the tab label.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- Title: primary purpose first, followed by site name -->
  <title>HTML Element Fundamentals — Web Development Guide</title>
</head>
<body>
  <h1>HTML Element Fundamentals</h1>
  <p>The h1 is the visible heading; the title is the tab label.</p>
</body>
</html>
```

**Expected output**: The browser tab displays “HTML Element Fundamentals — Web Development Guide”. The page body displays the `<h1>` heading “HTML Element Fundamentals”.

**Why this output occurs**: The `<title>` provides the browser-tab label and search-result title, while the `<h1>` provides the visible page heading. Placing the primary purpose first ensures that it survives search-engine truncation.

---

#### Example 2: Updating the Title to Reflect a Form Error

**Setup guide**: Save as `title-error.html`. Submit the form with an empty required field to trigger the JavaScript.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Contact Form</title>
</head>
<body>
  <form id="contact-form">
    <label for="name">Name</label>
    <input type="text" id="name" required>
    <button type="submit">Send</button>
  </form>

  <script>
    const form = document.getElementById('contact-form');
    form.addEventListener('submit', function (event) {
      const nameInput = document.getElementById('name');
      if (!nameInput.value.trim()) {
        event.preventDefault(); // stop submission
        // Update the title to reflect the error state
        document.title = 'Error: Name is required — Contact Form';
      }
    });
  </script>
</body>
</html>
```

**Expected output**: If the user submits the form with an empty name field, the browser tab title changes to “Error: Name is required — Contact Form” and submission is prevented.

**Why this output occurs**: `document.title` reflects the `<title>` element’s text. Updating it changes the tab label, providing a persistent visual cue about the page’s state. Because screen readers may not announce the change automatically, additional ARIA live-region techniques may be needed in production.

---

### Real-World Cases

1. **E-commerce product pages**: Titles combine the product name with the category and brand (e.g. “Wireless Headphones — Electronics — ExampleStore”), improving search-result relevance.
2. **Documentation pages**: Each reference page uses a unique title beginning with the API or concept name, followed by the documentation site name.
3. **Single-page applications**: The title is updated on route changes so that bookmarks and browser history entries remain meaningful.

---

### References

- WHATWG — The `title` element — https://html.spec.whatwg.org/multipage/semantics.html#the-title-element
- MDN Web Docs — `<title>`: The Document Title element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/title
- Google Search Central — Title links — https://developers.google.com/search/docs/appearance/title-link

---

## 4. `<base>` — The Document Base URL Element

### Definitions

**Core Definition**
The `<base>` element specifies the base URL to use for all relative URLs in a document. A document may contain at most one `<base>` element that has an `href` or `target` attribute.

**Technical Definition**
`<base>` belongs to the **metadata content** category. It must have an `href` attribute, a `target` attribute, or both. If at least one of these attributes is present, the element must appear **before** other elements with attribute values that are URLs, such as `<link href>`. The `href` attribute sets the document base URL used to resolve relative URLs; the `target` attribute sets the default browsing context for links and form submissions that do not specify a target.

**Beginner-Friendly Explanation**
The `<base>` element is like setting a “home address” for all relative links on the page. If your page is at `https://example.com/docs/page.html` and you use `<base href="https://example.com/docs/">`, then a link written as `images/photo.jpg` resolves to `https://example.com/docs/images/photo.jpg` instead of the browser’s default resolution.

---

### Purposes

- **To set a document-wide base URL** for resolving relative URLs.
- **To set a default browsing context** for links and forms that omit a `target` attribute.
- **To simplify markup in sites where relative URLs should resolve against a known root** rather than the current page’s directory.
- **To control the default target** for navigation (e.g. opening all unspecified links in a new tab).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<head>
  <base href="https://example.com/docs/" target="_blank">
  <!-- other URL-bearing elements come after <base> -->
  <link rel="stylesheet" href="styles.css">
</head>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `href` | The base URL for all relative URLs in the document. Absolute and relative URLs are allowed; `data:` and `javascript:` URLs are not allowed. |
| `target` | The default browsing context for navigation from `<a>`, `<area>`, and `<form>` elements without explicit targets. Keywords: `_self`, `_blank`, `_parent`, `_top`. |

#### Syntax Rules

1. **At most one `<base>` element with `href` or `target`** per document; if multiple are present, only the first `href` and first `target` are honoured.
2. **`href` or `target` (or both) is required** on `<base>`.
3. **`<base>` must precede URL-bearing elements** such as `<link>` with `href`.
4. **`target` values must not contain ASCII newline, tab, or `<` characters**; if they do, the value is reset to `_blank` to prevent dangling-markup injection attacks.
5. **In-page fragment links are resolved against the base URL**, potentially causing an HTTP request to the base URL with the fragment attached.

#### Constraints and Limitations

- `<base>` cannot use `data:` or `javascript:` URLs in `href`.
- Open Graph metadata and other protocols that require absolute URLs do not acknowledge `<base>` and must use full absolute URLs.
- Using `<base target="_blank">` changes the default for all links and forms, which can be surprising and may harm accessibility if users are not warned.
- The element is not a substitute for writing correct absolute URLs where required.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting a Base URL for Relative Resources

**Setup guide**: Save as `base-href.html`. Serve the file from a local web server so that relative URL resolution can be observed.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- All relative URLs resolve against this base -->
  <base href="https://example.com/assets/">
  <title>Base URL Example</title>

  <!-- Because of <base>, this resolves to https://example.com/assets/styles.css -->
  <link rel="stylesheet" href="styles.css">

  <!-- This resolves to https://example.com/assets/logo.png -->
  <link rel="icon" href="logo.png">
</head>
<body>
  <p>Relative URLs in this document resolve against the base URL.</p>
</body>
</html>
```

**Expected output**: The browser requests `https://example.com/assets/styles.css` and `https://example.com/assets/logo.png`, regardless of the actual location of `base-href.html`.

**Why this output occurs**: The `<base href>` sets the document base URL. All relative URLs in URL-bearing attributes are resolved against this base rather than the document’s own URL. This is useful when the HTML is served from a different path than the assets.

---

#### Example 2: Setting a Default Target for Links

**Setup guide**: Save as `base-target.html` and click the link.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- All links without an explicit target open in a new tab -->
  <base target="_blank">
  <title>Base Target Example</title>
</head>
<body>
  <!-- This link has no target attribute; <base target="_blank"> applies -->
  <a href="https://example.com">Visit Example (opens in a new tab)</a>
</body>
</html>
```

**Expected output**: Clicking the link opens `https://example.com` in a new, unnamed browsing context (a new tab or window), because the `<base target="_blank">` supplies the default target.

**Why this output occurs**: The `<base target>` attribute defines the default browsing context for navigation from `<a>`, `<area>`, and `<form>` elements that do not specify their own `target`. Individual elements can override this by specifying their own `target`.

---

### Real-World Cases

1. **Static site generators**: When HTML pages are generated into nested directories but assets live at a common root, a `<base href="/">` can simplify relative asset URLs.
2. **Single-page applications**: Some SPAs use `<base href="/app/">` to ensure that relative API calls and asset URLs resolve correctly regardless of the current route.
3. **Controlling link behaviour globally**: A site may use `<base target="_blank">` to open external links in new tabs, though this should be applied with accessibility caution.

---

### References

- WHATWG — The `base` element — https://html.spec.whatwg.org/multipage/semantics.html#the-base-element
- MDN Web Docs — `<base>`: The Document Base URL element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/base
- W3C — HTML 5.1: The `base` element — https://www.w3.org/TR/html51/document-metadata.html#the-base-element

---

## 5. `<link>` — External Resource Link Element

### Definitions

**Core Definition**
The `<link>` element specifies relationships between the current document and an external resource. It is most commonly used to link stylesheets, but it also establishes site icons, preloads critical resources, and declares other relationships.

**Technical Definition**
`<link>` is a **void element** belonging to the **metadata content** category (and conditionally to flow and phrasing content when `itemprop` is present). Its behaviour is governed by the `rel` attribute, which defines the link type (relationship). Common link types include `stylesheet`, `icon`, `preload`, `preconnect`, `dns-prefetch`, `canonical`, `alternate`, and `manifest`. The `href` attribute specifies the resource URL; `as`, `crossorigin`, `media`, `sizes`, `type`, and `integrity` provide additional control.

**Beginner-Friendly Explanation**
The `<link>` element is how HTML connects to the outside world. The most familiar use is loading a CSS file, but `<link>` can also set the favicon, tell the browser to start downloading a font early, or declare which page is the “official” version of the current page.

---

### Purposes

- **To load external stylesheets**: `<link rel="stylesheet" href="styles.css">`.
- **To define site icons**: favicons, Apple touch icons, and mask icons via `rel="icon"` and related values.
- **To preload or prefetch resources**: `rel="preload"` and `rel="prefetch"` improve loading performance.
- **To establish early connections**: `rel="preconnect"` and `rel="dns-prefetch"` reduce latency for cross-origin requests.
- **To declare canonical or alternate versions**: `rel="canonical"` and `rel="alternate"` convey document relationships to search engines and feed readers.
- **To link a web app manifest**: `rel="manifest"` provides metadata for progressive web apps.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<link rel="stylesheet" href="styles.css" media="screen" type="text/css">
<link rel="icon" href="favicon.ico" sizes="any">
<link rel="preload" href="hero-font.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preconnect" href="https://api.example.com">
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `rel` | **Required.** Defines the relationship type (e.g. `stylesheet`, `icon`, `preload`). |
| `href` | The URL of the linked resource. |
| `as` | For `preload`: the content type being fetched (e.g. `font`, `script`, `style`). |
| `crossorigin` | How the resource handles CORS; required for some preloads. |
| `media` | A media query that determines when the resource applies (mainly for stylesheets). |
| `sizes` | Icon sizes (e.g. `any`, `32x32`). |
| `type` | MIME type hint for the linked resource. |
| `integrity` | Subresource Integrity metadata for verification. |

#### Syntax Rules

1. **`rel` is mandatory** and its value determines the link type.
2. **`<link>` is a void element**: no end tag, no children.
3. **`as` is required when `rel="preload"`** and is used to specify the resource class.
4. **`<link>` may appear in `<head>` or, for body-ok link types such as `stylesheet`, in the body**; however, placing it in `<head>` is the recommended practice.
5. **`<base>` must precede `<link href>`** if `<base href>` is present.

#### Constraints and Limitations

- The `icon` link type is valid only in HTML, not in HTTP `Link` headers; `preconnect` and `preload` are the most reliable link types when used in HTTP headers.
- `preload` must be used carefully: preloading resources that are not used by the page wastes bandwidth and can delay other critical resources.
- The `disabled` attribute on `<link rel="stylesheet">` can disable a stylesheet, but support and behaviour vary.
- Event handler attributes on `<link>` are defined by the specification but have unclear practical use.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Stylesheet, Favicon, and Preconnect

**Setup guide**: Save as `link-resources.html` and inspect the Network panel.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Link Relationships</title>

  <!-- External stylesheet -->
  <link rel="stylesheet" href="styles.css" media="screen">

  <!-- Favicon -->
  <link rel="icon" href="favicon.ico" sizes="any">

  <!-- Preconnect to an API origin used later by scripts -->
  <link rel="preconnect" href="https://api.example.com">

  <!-- DNS prefetch as a lighter-weight fallback -->
  <link rel="dns-prefetch" href="https://cdn.example.com">
</head>
<body>
  <p>Styles, icon, and connection hints are declared in the head.</p>
</body>
</html>
```

**Expected output**: The browser loads `styles.css`, displays the favicon in the tab, and begins opening connections to `https://api.example.com` before the script that uses the API runs. DNS prefetching for `cdn.example.com` resolves the domain early.

**Why this output occurs**: `rel="stylesheet"` triggers CSS loading. `rel="icon"` identifies the favicon. `rel="preconnect"` and `rel="dns-prefetch"` are resource hints that reduce connection latency for later requests. None of these elements renders visible content.

---

#### Example 2: Preloading a Critical Font

**Setup guide**: Save as `link-preload.html`. Use a real font file path.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Preload a Font</title>

  <!-- Preload the font used in the hero section -->
  <link
    rel="preload"
    href="fonts/hero-font.woff2"
    as="font"
    type="font/woff2"
    crossorigin>
</head>
<body>
  <h1 style="font-family: 'HeroFont', sans-serif;">Fast-rendering heading</h1>
</body>
</html>
```

**Expected output**: The browser starts downloading `hero-font.woff2` with high priority during HTML parsing, so the font is more likely to be ready when the heading is rendered, reducing the flash of unstyled text (FOUT).

**Why this output occurs**: `rel="preload"` with `as="font"` tells the browser that the font is a critical resource needed soon. The `crossorigin` attribute is required for font preloads because fonts are subject to CORS. Without preload, the browser would discover the font only after parsing the CSS, delaying its fetch.

---

### Real-World Cases

1. **Every styled website**: `<link rel="stylesheet">` is the standard mechanism for loading external CSS.
2. **Performance optimisation**: `rel="preload"` for hero images, fonts, and critical CSS, combined with `rel="preconnect"` for third-party origins, is a core Web Vitals strategy.
3. **Favicons and PWA icons**: `rel="icon"`, `rel="apple-touch-icon"`, and `rel="mask-icon"` provide icons for browser tabs, home screens, and pinned tabs.
4. **Canonical URLs**: `rel="canonical"` tells search engines which URL is the preferred version when content is accessible through multiple URLs.

---

### References

- WHATWG — The `link` element — https://html.spec.whatwg.org/multipage/semantics.html#the-link-element
- MDN Web Docs — `<link>`: The External Resource Link element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/link
- MDN Web Docs — Link types — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/rel
- web.dev — Preload critical assets to improve loading speed — https://web.dev/articles/preload-critical-assets

---

## 6. `<meta>` — Generic Metadata Element

### Definitions

**Core Definition**
The `<meta>` element represents metadata that cannot be expressed by other meta-related elements such as `<base>`, `<link>`, `<script>`, `<style>`, or `<title>`.

**Technical Definition**
`<meta>` is a **void element** belonging to the **metadata content** category (and conditionally to flow and phrasing content when `itemprop` is present). It carries metadata through attributes: `name`/`content` pairs, `http-equiv`/`content` pairs, `charset`, and `itemprop`. Standard metadata names include `description`, `viewport`, `robots`, `theme-color`, and `author`. The `charset` attribute declares the document’s character encoding and should appear within the first 1024 bytes of the document.

**Beginner-Friendly Explanation**
The `<meta>` element is the “miscellaneous metadata” element. When information about the page does not fit into `<title>`, `<link>`, `<base>`, `<style>`, or `<script>`, `<meta>` carries it — things like the page description for search engines, the viewport settings for mobile, or instructions to search-engine crawlers.

---

### Purposes

- **To declare character encoding**: `<meta charset="UTF-8">` ensures correct text decoding.
- **To configure the viewport**: `<meta name="viewport" content="width=device-width, initial-scale=1">` enables responsive layout on mobile devices.
- **To provide a page description**: `<meta name="description" content="…">` supplies the summary used by search engines.
- **To control search-engine crawling**: `<meta name="robots" content="noindex, nofollow">`.
- **To suggest a theme colour**: `<meta name="theme-color" content="#ff00ff">` customises browser UI.
- **To declare author or application metadata**: `<meta name="author">`, `<meta name="application-name">`.
- **To simulate HTTP headers**: `http-equiv` can set pragma directives such as `content-type` or `default-style` (though many are obsolete).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="description" content="A concise summary of the page.">
<meta name="robots" content="index, follow">
<meta name="theme-color" content="#0077cc">
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```

**Component breakdown**

| Attribute form | Description |
|---|---|
| `charset="UTF-8"` | Declares the character encoding; must appear early in the document. |
| `name` + `content` | A named metadata pair (e.g. `description`, `viewport`, `robots`). |
| `http-equiv` + `content` | A pragma directive that simulates an HTTP header. |
| `itemprop` | Microdata property name (conditionally makes `<meta>` flow/phrasing content). |

**Standard metadata names** (representative): `application-name`, `author`, `description`, `generator`, `keywords` (largely ignored by search engines), `referrer`, `robots`, `theme-color`, `viewport`.

#### Syntax Rules

1. **`<meta>` is a void element**: no end tag, no children.
2. **Exactly one of `name`, `http-equiv`, `charset`, or `itemprop`** should be present in a given `<meta>` element.
3. **`charset` should appear within the first 1024 bytes** of the document for reliable encoding detection.
4. **`name="viewport"` should include `width=device-width`** and optionally `initial-scale=1` for responsive design.
5. **Do not use `user-scalable=no` or very low `maximum-scale` values**; these block pinch-zoom and constitute an accessibility failure.

#### Constraints and Limitations

- The `keywords` metadata name is largely ignored by modern search engines and should not be relied upon.
- `http-equiv="content-type"` is obsolete; use `<meta charset>` instead.
- `http-equiv="set-cookie"` is obsolete and should not be used.
- `meta name="robots"` values are advisory; well-behaved crawlers follow them, but they are not a security mechanism.
- `theme-color` is a hint and may be ignored by browsers that do not support it.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Essential Meta Tags for a Responsive, SEO-Friendly Page

**Setup guide**: Save as `meta-essential.html` and open it in a browser. Inspect the source and use a search-engine preview tool if available.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Character encoding: must be early -->
  <meta charset="UTF-8">

  <!-- Viewport: enables responsive layout; do not block zoom -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Description: used by search engines for snippets -->
  <meta name="description" content="A beginner-friendly guide to HTML metadata elements, including head, title, meta, link, style, and script.">

  <!-- Robots: allow indexing and following -->
  <meta name="robots" content="index, follow">

  <!-- Theme colour: suggests browser UI tint -->
  <meta name="theme-color" content="#0077cc">

  <title>HTML Metadata Elements Guide</title>
</head>
<body>
  <h1>HTML Metadata Elements</h1>
  <p>The head contains machine-readable information about this page.</p>
</body>
</html>
```

**Expected output**: The browser tab shows “HTML Metadata Elements Guide”. On mobile, the page is rendered at the device width. The browser may tint its UI with `#0077cc`. Search engines can use the description as a snippet and are permitted to index and follow links.

**Why this output occurs**: `<meta charset>` ensures correct decoding. The viewport meta enables responsive layout. The description provides a search snippet. The robots meta permits indexing. The theme-colour meta suggests a UI tint. None of these elements renders visible body content.

---

#### Example 2: Using `http-equiv` for a Content Security Policy

**Setup guide**: Save as `meta-csp.html`. Serve the page and check the browser console for CSP violations.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Meta CSP Example</title>

  <!-- Simulates a Content-Security-Policy header -->
  <meta http-equiv="Content-Security-Policy"
        content="default-src 'self'; img-src 'self' https://images.example.com; style-src 'self' 'unsafe-inline';">
</head>
<body>
  <p>This page declares a CSP via a meta element.</p>
  <img src="https://images.example.com/photo.jpg" alt="Allowed image">
</body>
</html>
```

**Expected output**: The page renders, and the image from `https://images.example.com` is allowed. Inline styles are allowed via `'unsafe-inline'`, but scripts from other origins would be blocked.

**Why this output occurs**: `http-equiv="Content-Security-Policy"` instructs the browser to treat the `content` value as a CSP directive set, equivalent to the HTTP header. This can provide defence-in-depth when HTTP headers cannot be configured, though header-delivered CSP is preferred.

---

### Real-World Cases

1. **Mobile-responsive websites**: The viewport meta tag is essential for pages to render at the device width rather than at a desktop width.
2. **SEO and social sharing**: `meta name="description"` and Open Graph `<meta property="og:…">` tags (which use `property` instead of `name`) control how pages appear in search results and social-media previews.
3. **Progressive web apps**: `meta name="theme-color"` customises the browser UI to match the app’s branding.
4. **Security hardening**: `http-equiv="Content-Security-Policy"` and `name="referrer"` provide additional security controls when HTTP headers are unavailable.

---

### References

- WHATWG — The `meta` element — https://html.spec.whatwg.org/multipage/semantics.html#the-meta-element
- MDN Web Docs — `<meta>`: The metadata element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta
- MDN Web Docs — Standard metadata names — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta/name
- web.dev — Metadata — https://web.dev/learn/html/metadata

---

## 7. `<style>` — Embedded Style Information Element

### Definitions

**Core Definition**
The `<style>` element contains style information (CSS) for a document or a portion of a document. The styles apply to the document’s content according to CSS cascade and specificity rules.

**Technical Definition**
`<style>` belongs to the **metadata content** category. Its content model is **CSS text** that must conform to the CSS syntax. It accepts the `media` attribute (a media query controlling when the styles apply) and the `type` attribute (which should be omitted or set to `text/css`). The `scoped` attribute was historically proposed to limit styles to the parent subtree but was removed from the specification and is not supported in modern browsers. The `title` attribute defines an alternative style-sheet set.

**Beginner-Friendly Explanation**
The `<style>` element lets you write CSS directly inside your HTML file instead of putting it in a separate `.css` file. It is useful for small amounts of page-specific styling, but large projects usually prefer external stylesheets so that styles can be cached and reused.

---

### Purposes

- **To embed CSS directly in the document** without an external file.
- **To apply page-specific or component-specific styles** that do not need to be shared.
- **To conditionally apply styles** using the `media` attribute (e.g. print styles).
- **To define alternative style-sheet sets** using the `title` attribute (historical, with limited browser support).
- **To prototype styles quickly** during development.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<style media="screen and (min-width: 600px)">
  /* CSS rules */
  body {
    font-family: system-ui, sans-serif;
  }
</style>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<style>` | The start tag; belongs in `<head>` (or in the body with the `scoped` attribute, which is obsolete). |
| CSS text | Rules conforming to CSS syntax; must not contain HTML tags. |
| `media` | A media query that determines when the styles apply. |
| `type` | Optional; should be omitted or set to `text/css`. |
| `title` | Defines an alternative style-sheet set; limited practical support. |
| `</style>` | The end tag. |

#### Syntax Rules

1. **CSS syntax must be valid**: malformed CSS is ignored according to CSS error-handling rules.
2. **The `media` attribute** accepts a media query; styles apply only when the query matches.
3. **The `type` attribute** is unnecessary in HTML5; omitting it is preferred.
4. **The `scoped` attribute is obsolete** and should not be used; it was removed from the specification and is not supported in current browsers.
5. **`<style>` may appear in the body** in some contexts, but placing it in `<head>` is the conventional and recommended practice.

#### Constraints and Limitations

- Inline `<style>` blocks are not cached across pages, unlike external stylesheets.
- Large `<style>` blocks increase HTML file size and can delay rendering if placed after content.
- The `title` attribute’s alternative style-sheet behaviour is inconsistently supported and should not be relied upon for critical functionality.
- The `scoped` attribute is obsolete; do not use it in new code.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Embedded Styles with a Media Query

**Setup guide**: Save as `style-media.html` and open it in a browser. Resize the window to observe the media query.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Style with Media Query</title>

  <!-- Base styles for all screens -->
  <style>
    body { font-family: sans-serif; margin: 20px; }
    .box { padding: 20px; background: #eef; }
  </style>

  <!-- Styles applied only when the viewport is at least 600px wide -->
  <style media="screen and (min-width: 600px)">
    .box { background: #efe; border: 2px solid green; }
  </style>
</head>
<body>
  <div class="box">Resize the window to see the style change.</div>
</body>
</html>
```

**Expected output**: On narrow screens (less than 600px), the box has a light blue background and no border. On wider screens (600px or more), the box has a light green background and a green border.

**Why this output occurs**: The second `<style>` element has a `media` attribute containing `screen and (min-width: 600px)`. The browser applies its rules only when the media query matches, demonstrating conditional styling without JavaScript.

---

#### Example 2: Using `<style>` in the Body (Not Recommended but Valid in Some Contexts)

**Setup guide**: Save as `style-body.html` and inspect the DOM.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Style in Body</title>
</head>
<body>
  <p>This paragraph is styled by a style element placed later in the body.</p>

  <!-- <style> is metadata content but can appear in the body -->
  <style>
    p { color: darkred; font-weight: bold; }
  </style>
</body>
</html>
```

**Expected output**: The paragraph appears in bold dark red.

**Why this output occurs**: `<style>` is metadata content, but the HTML parser does not restrict it exclusively to `<head>`. When encountered in the body, the CSS rules still apply to the whole document. However, separating styles into `<head>` or external files is better practice for maintainability and performance.

---

### Real-World Cases

1. **Critical CSS**: Small amounts of above-the-fold CSS are sometimes inlined in `<style>` to avoid a render-blocking external request.
2. **Print styles**: `<style media="print">` provides print-specific rules without affecting screen rendering.
3. **Component-level styling in single-file components**: Some frameworks generate `<style>` blocks scoped to a component (though native `scoped` is obsolete; frameworks simulate scoping with attribute selectors).
4. **Email templates**: HTML email often uses `<style>` blocks because external stylesheets are inconsistently supported by email clients.

---

### References

- WHATWG — The `style` element — https://html.spec.whatwg.org/multipage/semantics.html#the-style-element
- MDN Web Docs — `<style>`: The Style Information element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/style
- MDN Web Docs — `<style>`: `media` attribute — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/style#media

---

## 8. `<script>` — Script Element

### Definitions

**Core Definition**
The `<script>` element is used to embed or reference executable code, typically JavaScript, within an HTML document.

**Technical Definition**
`<script>` belongs to the **metadata content**, **flow content**, **phrasing content**, and **script-supporting element** categories. Its content model depends on the presence of `src` and the `type` attribute: inline scripts must match the script content restrictions, while external scripts must be empty or contain only script documentation. Key attributes include `src`, `type` (`text/javascript` or `module`), `async`, `defer`, `nomodule`, `crossorigin`, `integrity`, `referrerpolicy`, `fetchpriority`, and `blocking`.

**Beginner-Friendly Explanation**
The `<script>` element is how you add JavaScript to a page — either by writing the code directly between the tags or by pointing to an external `.js` file with `src`. Attributes like `defer` and `async` control when the script runs relative to the rest of the page.

---

### Purposes

- **To embed inline JavaScript**: code written directly inside `<script>…</script>`.
- **To reference external scripts**: `<script src="app.js"></script>`.
- **To control execution timing**: `defer` waits until parsing is complete; `async` executes as soon as the script is available.
- **To load ES modules**: `type="module"` enables module semantics and deferred execution by default.
- **To support older browsers**: `nomodule` prevents a script from executing in browsers that support modules.
- **To verify integrity**: `integrity` enables Subresource Integrity checks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- External classic script with defer -->
<script src="app.js" defer></script>

<!-- External module script -->
<script type="module" src="app.mjs"></script>

<!-- Inline classic script -->
<script>
  console.log("Hello from inline script");
</script>
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of an external script. If present, the element should be empty or contain only documentation. |
| `type` | `text/javascript` (default) or `module` for ES modules. |
| `async` | Fetches in parallel and executes as soon as available; does not guarantee order. |
| `defer` | Fetches in parallel and executes after parsing, in document order. |
| `nomodule` | Prevents execution in browsers that support module scripts; used for legacy fallbacks. |
| `integrity` | SRI hash for verifying the fetched resource. |
| `crossorigin` | Controls CORS behaviour for cross-origin fetches. |
| `referrerpolicy` | Referrer policy for the fetch. |

#### Syntax Rules

1. **Classic scripts** block HTML parsing unless `async` or `defer` is present.
2. **`defer` has no effect on inline scripts** (scripts without `src`) and is ignored.
3. **Module scripts defer by default**: there is no way to make a module script block the parser while it fetches.
4. **`async` on a module script** allows the module and its dependencies to be fetched in parallel and evaluated as soon as possible.
5. **`nomodule` should be used for legacy classic-script fallbacks** in module-supporting browsers.
6. **`integrity` must not be specified without `src`**.

#### Constraints and Limitations

- Inline scripts must not contain the literal string `</script>` without escaping; the parser will terminate the element early.
- `async` scripts execute out of order; do not use `async` when scripts depend on one another.
- `defer` preserves document order, making it suitable for scripts that depend on the DOM or on earlier scripts.
- Module scripts are always deferred; the `defer` attribute is ignored on them.
- `blocking="render"` can make a script render-blocking; use with caution.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `defer` vs `async` — Execution Order

**Setup guide**: Save as `script-defer-async.html` and observe the console.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Defer vs Async</title>

  <!-- Deferred scripts execute in order after parsing -->
  <script src="defer-1.js" defer></script>
  <script src="defer-2.js" defer></script>

  <!-- Async scripts execute as soon as available; order is not guaranteed -->
  <script src="async-1.js" async></script>
  <script src="async-2.js" async></script>
</head>
<body>
  <p>Open the console to see execution order.</p>
</body>
</html>
```

**Expected output**: The `defer` scripts execute after HTML parsing, in the order `defer-1.js` then `defer-2.js`. The `async` scripts execute as soon as each finishes downloading, which may be in any order depending on network timing.

**Why this output occurs**: `defer` queues scripts for execution after parsing, preserving document order. `async` executes immediately upon download completion, without waiting for parsing or for other scripts. This makes `defer` suitable for dependent scripts and `async` suitable for independent analytics or ads.

---

#### Example 2: Module Script with a `nomodule` Fallback

**Setup guide**: Save as `script-module.html`. Use a browser that supports ES modules to see the modern path; older browsers will run the `nomodule` fallback.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Module and Nomodule</title>

  <!-- Modern browsers: load the module -->
  <script type="module" src="modern-app.mjs"></script>

  <!-- Older browsers that do not support modules: load the classic fallback -->
  <script nomodule src="legacy-app.js"></script>
</head>
<body>
  <p>Module-capable browsers run modern-app.mjs; older browsers run legacy-app.js.</p>
</body>
</html>
```

**Expected output**: In a modern browser, `modern-app.mjs` executes; `legacy-app.js` is ignored because `nomodule` prevents its execution in module-supporting browsers. In an older browser, `modern-app.mjs` is ignored (unknown type), and `legacy-app.js` executes.

**Why this output occurs**: `type="module"` marks the script as an ES module. `nomodule` is a boolean attribute that tells module-supporting browsers not to execute the script, providing a clean legacy fallback.

---

### Real-World Cases

1. **Analytics and tracking**: Often loaded with `async` so they do not block page rendering.
2. **Application bundles**: Loaded with `defer` so they execute after the DOM is parsed and can safely query elements.
3. **Modern JavaScript applications**: Use `type="module"` for native ES module loading, with `nomodule` fallbacks for legacy browsers.
4. **Third-party widgets**: Loaded with `async` because their execution order relative to the page’s own scripts is not important.

---

### References

- WHATWG — The `script` element — https://html.spec.whatwg.org/multipage/scripting.html#the-script-element
- MDN Web Docs — `<script>`: The Script element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script
- web.dev — Efficiently load JavaScript with defer and async — https://web.dev/articles/efficiently-load-third-party-javascript

---

## 9. `<noscript>` — Script-Disabled Fallback Element

### Definitions

**Core Definition**
The `<noscript>` element defines a section of HTML to be inserted if a script type on the page is unsupported or if scripting is currently turned off in the browser.

**Technical Definition**
`<noscript>` belongs to the **metadata content**, **flow content**, and **phrasing content** categories. Its permitted content depends on its location and on whether scripting is enabled or disabled:

- **In `<head>`, scripting disabled**: zero or more `<link>`, `<style>`, and `<meta>` elements, in any order.
- **Outside `<head>`, scripting disabled**: transparent content, but no descendant `<noscript>` element.
- **Scripting enabled**: the element’s children are treated as text and do not render as markup.

**Beginner-Friendly Explanation**
The `<noscript>` element is a fallback for people who have JavaScript turned off or whose browser does not support it. You put alternative content or alternative resources inside `<noscript>`, and the browser shows or applies them only when scripting is unavailable.

---

### Purposes

- **To provide alternative content when scripting is disabled**: text, images, or links that explain what the page would do with JavaScript.
- **To load fallback styles or metadata in `<head>`**: alternative stylesheets or meta instructions.
- **To support progressive enhancement**: ensure core content and functionality remain accessible without scripts.
- **To warn users about missing functionality**: a clear message that the interactive features require JavaScript.

---

### Syntax Rules and Structure

#### Complete General Syntax (in `<head>`)

```html
<noscript>
  <link rel="stylesheet" href="no-script.css">
  <style>
    /* fallback styles */
  </style>
  <meta http-equiv="refresh" content="0; url=/no-script-fallback">
</noscript>
```

#### Complete General Syntax (in `<body>`)

```html
<noscript>
  <p>This feature requires JavaScript. Please enable JavaScript or use the <a href="/alternative">alternative version</a>.</p>
</noscript>
```

**Component breakdown**

| Context | Permitted content |
|---|---|
| In `<head>`, scripting disabled | `<link>`, `<style>`, `<meta>` only. |
| Outside `<head>`, scripting disabled | Transparent content (flow or phrasing, depending on parent); no nested `<noscript>`. |
| Scripting enabled | Children treated as text; markup is not parsed or rendered. |

#### Syntax Rules

1. **In `<head>`, only `<link>`, `<style>`, and `<meta>` are allowed** when scripting is disabled.
2. **Outside `<head>`, the content model is transparent**, meaning it must match the parent’s content model; nested `<noscript>` is forbidden.
3. **Both start and end tags are mandatory**; there is no tag omission for `<noscript>`.
4. **When scripting is enabled, the content is inert text**; it is not rendered as HTML.

#### Constraints and Limitations

- `<noscript>` in `<head>` cannot contain arbitrary flow content; only metadata elements are permitted.
- Nested `<noscript>` elements are invalid.
- The fallback may not be useful if the page’s core content depends entirely on JavaScript; progressive enhancement is preferred.
- `<noscript>` content is invisible to users with JavaScript enabled, so do not put critical information there exclusively.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Body Fallback Message

**Setup guide**: Save as `noscript-body.html`. Disable JavaScript in the browser and reload to see the fallback.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Noscript Body Fallback</title>
</head>
<body>
  <div id="app">
    <!-- JavaScript will replace this content if enabled -->
    <p>Loading interactive content…</p>
  </div>

  <noscript>
    <!-- This content appears only when scripting is disabled -->
    <p><strong>JavaScript is disabled.</strong> The interactive features of this page require JavaScript. Please enable it or use the <a href="/basic">basic version</a>.</p>
  </noscript>

  <script>
    document.getElementById('app').innerHTML = '<p>Interactive content loaded.</p>';
  </script>
</body>
</html>
```

**Expected output**: With JavaScript enabled, the page shows “Interactive content loaded.” and the `<noscript>` message is not displayed. With JavaScript disabled, the page shows “Loading interactive content…” followed by the fallback warning and link.

**Why this output occurs**: When scripting is enabled, the `<noscript>` children are treated as text and are not rendered as HTML. When scripting is disabled, the `<noscript>` content is parsed as HTML and displayed.

---

#### Example 2: Head Fallback Stylesheet

**Setup guide**: Save as `noscript-head-style.html`. Disable JavaScript and reload.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Noscript Head Styles</title>

  <!-- Normal styles -->
  <style>
    body { background: #ffffff; color: #000000; }
  </style>

  <!-- Fallback styles applied only when scripting is disabled -->
  <noscript>
    <style>
      body { background: #fff3cd; color: #856404; }
    </style>
    <link rel="stylesheet" href="no-script-extra.css">
  </noscript>
</head>
<body>
  <p>If JavaScript is disabled, a warning-coloured theme is applied.</p>
</body>
</html>
```

**Expected output**: With JavaScript enabled, the page has a white background and black text. With JavaScript disabled, the page has a warm yellow background, brown text, and any additional styles from `no-script-extra.css` are applied.

**Why this output occurs**: In `<head>`, when scripting is disabled, `<noscript>` may contain `<link>`, `<style>`, and `<meta>` elements, and those elements take effect. When scripting is enabled, the `<noscript>` content is treated as text and has no effect.

---

### Real-World Cases

1. **Progressive enhancement**: A form that works with server-side submission even when JavaScript is disabled can use `<noscript>` to warn users that client-side validation will not run.
2. **Analytics fallback**: A `<noscript>` image pixel can provide basic analytics when JavaScript-based tracking is unavailable.
3. **Critical CSS fallback**: Some performance-focused sites use `<noscript>` to load a full stylesheet when JavaScript is disabled, while JavaScript-enabled browsers load critical CSS asynchronously.

---

### References

- WHATWG — The `noscript` element — https://html.spec.whatwg.org/multipage/scripting.html#the-noscript-element
- MDN Web Docs — `<noscript>`: The Noscript element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/noscript
- W3C — HTML5: The `noscript` element — https://www.w3.org/TR/html5/scripting-1.html#the-noscript-element

---

## 10. Metadata Element Best Practices

### Definitions

**Core Definition**
Metadata element best practices are authoring guidelines for using `<html>`, `<head>`, `<title>`, `<base>`, `<link>`, `<meta>`, `<style>`, `<script>`, and `<noscript>` in ways that are valid, accessible, performant, secure, and maintainable.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG accessibility guidelines, search-engine documentation, and performance guidance. They encompass correct content models, required attributes, attribute-value constraints, ordering rules, and the separation of concerns between structure (HTML), presentation (CSS), and behaviour (JavaScript).

**Beginner-Friendly Explanation**
Best practices are the “good habits” for the `<head>`. They do not always produce visible differences, but following them makes your pages more accessible, easier to maintain, faster to load, and less likely to break in different browsers.

---

### Purposes

- **To maximise accessibility**: correct titles, language declarations, viewport settings, and fallbacks ensure that assistive technologies can present pages correctly.
- **To improve performance**: proper use of `defer`, `async`, `preload`, and `preconnect` reduces blocking and latency.
- **To support SEO**: unique, descriptive titles and meta descriptions improve search-result presentation.
- **To ensure validity and cross-browser reliability**: respecting content models and attribute constraints prevents parser error recovery.
- **To improve security**: `integrity`, `crossorigin`, and CSP meta tags reduce the risk of resource tampering and injection attacks.

---

### Syntax Rules and Structure

Best practices are applied across elements; the following table summarises the core rules.

| Practice | Rule |
|---|---|
| **Declare character encoding early** | `<meta charset="UTF-8">` should appear within the first 1024 bytes. |
| **Always include a viewport meta** | `<meta name="viewport" content="width=device-width, initial-scale=1.0">`; do not disable zoom. |
| **Use a unique, descriptive title** | Primary purpose first, site name after; roughly 55–60 characters. |
| **Provide a meta description** | Unique per page, roughly 150–160 characters, compelling and accurate. |
| **Declare the document language** | `lang="…"` on `<html>` with a valid BCP 47 tag. |
| **Place `<base>` before URL-bearing elements** | If `<base href>` is used, it must precede `<link href>` and similar elements. |
| **Use `defer` for DOM-dependent scripts** | Classic scripts that query the DOM should be deferred. |
| **Use `async` only for independent scripts** | Analytics and ads that do not depend on other scripts. |
| **Use `type="module"` with `nomodule` fallback** | For modern JavaScript with a legacy path. |
| **Use `<noscript>` for meaningful fallbacks** | Warn users or load alternative resources; do not rely on it for critical content exclusively. |
| **Prefer external stylesheets for large CSS** | Use `<style>` for small, page-specific, or critical CSS. |
| **Use `preload` sparingly** | Only for resources needed immediately; wasted preloads delay other resources. |
| **Do not use obsolete attributes** | Avoid `profile` on `<head>`, `version` on `<html>`, `scoped` on `<style>`, `http-equiv="content-type"`. |

#### Syntax Rules

1. **Exactly one `<title>`** per document.
2. **Exactly one `<head>`** as the first child of `<html>`.
3. **`<html>` must contain `<head>` followed by `<body>`.**
4. **`<meta charset>` must be early.**
5. **`<base>` must precede URL-bearing elements.**
6. **`defer` scripts execute in document order; `async` scripts do not.**
7. **`<noscript>` in `<head>` may contain only `<link>`, `<style>`, and `<meta>`.**

#### Constraints and Limitations

- Best practices can conflict with legacy systems; incremental improvement is acceptable.
- Performance hints (`preload`, `preconnect`) must be measured; overuse can harm performance.
- Search-engine guidelines evolve; what is best practice today may change.
- Some attributes (e.g. `blocking="render"`) are newer and may have varying browser support.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Well-Formed, Accessible, and Performant `<head>`

**Setup guide**: Save as `best-practice-head.html` and inspect it with browser developer tools and an HTML validator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- 1. Character encoding early -->
  <meta charset="UTF-8">

  <!-- 2. Viewport for responsive layout; do not disable zoom -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- 3. Unique, descriptive title (primary purpose first) -->
  <title>Metadata Best Practices — Web Development Guide</title>

  <!-- 4. Unique meta description -->
  <meta name="description" content="A practical guide to document metadata elements, including head, title, meta, link, style, script, and noscript best practices.">

  <!-- 5. Robots directive -->
  <meta name="robots" content="index, follow">

  <!-- 6. Theme colour -->
  <meta name="theme-color" content="#0a7ea4">

  <!-- 7. Preconnect to a critical third-party origin -->
  <link rel="preconnect" href="https://fonts.example.com">

  <!-- 8. External stylesheet -->
  <link rel="stylesheet" href="styles.css">

  <!-- 9. Deferred application script -->
  <script src="app.js" defer></script>

  <!-- 10. Fallback styles when scripting is disabled -->
  <noscript>
    <style>
      .js-only { display: none; }
    </style>
  </noscript>
</head>
<body>
  <h1>Metadata Best Practices</h1>
  <p class="js-only">This paragraph is hidden when JavaScript is disabled.</p>
</body>
</html>
```

**Expected output**: The page has a unique tab title, responsive viewport behaviour, a search-engine description, a theme colour, preconnected font origin, external styles, a deferred script, and a fallback style that hides JavaScript-only content when scripting is disabled.

**Why this output occurs**: Each element follows the corresponding best practice: encoding early, viewport without zoom-blocking, unique title and description, language declaration, preconnect hint, deferred script, and `<noscript>` fallback. The result is accessible, performant, and conforming.

---

#### Example 2: Avoiding the Zoom-Blocking Viewport Anti-Pattern

**Setup guide**: Save as `viewport-bad.html` and compare with a properly configured viewport on a mobile device or emulator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- BAD: user-scalable=no and maximum-scale=1 block pinch-zoom -->
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
  <title>Zoom-Blocking Viewport (Do Not Use)</title>
</head>
<body>
  <p>This viewport configuration prevents users from pinch-zooming, which is an accessibility failure.</p>
</body>
</html>
```

**Expected output**: On a mobile device, the page cannot be pinch-zoomed, making small text difficult to read for users with low vision.

**Why this output occurs**: `user-scalable=no` and low `maximum-scale` values disable zoom. MDN explicitly warns against these values because they block pinch-zoom and constitute an accessibility failure. The correct practice is to allow zoom by omitting `user-scalable=no` and not setting a restrictive `maximum-scale`.

---

#### Example 3: Correct Ordering of `<base>` and URL-Bearing Elements

**Setup guide**: Save as `base-order.html` and observe the Network panel.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Base Ordering</title>

  <!-- CORRECT: <base> comes before <link href> -->
  <base href="https://cdn.example.com/assets/">
  <link rel="stylesheet" href="styles.css">
  <link rel="icon" href="favicon.ico">

  <!-- If <base> came after these links, they would resolve against the document URL instead -->
</head>
<body>
  <p>Relative URLs resolve against the CDN base URL.</p>
</body>
</html>
```

**Expected output**: The browser requests `https://cdn.example.com/assets/styles.css` and `https://cdn.example.com/assets/favicon.ico`.

**Why this output occurs**: `<base href>` must precede other elements with URL-bearing attributes for those attributes to resolve against the new base. Placing `<base>` after `<link href>` would cause the links to resolve against the document’s original URL, defeating the purpose of the base URL.

---

### Real-World Cases

1. **Government and public-sector sites**: Must meet WCAG, so viewport zoom-blocking, missing `lang`, and non-descriptive titles are compliance failures.
2. **E-commerce platforms**: Unique titles and meta descriptions per product page are essential for search visibility and conversion.
3. **Performance-focused sites**: Preconnect and preload hints, combined with `defer` scripts, are standard practices for Core Web Vitals optimisation.
4. **Security-sensitive applications**: Subresource Integrity on scripts and stylesheets, plus CSP via HTTP headers or `<meta http-equiv>`, protect against tampered resources.

---

### References

- WHATWG — HTML Living Standard — https://html.spec.whatwg.org/
- MDN Web Docs — What’s in the head? Web page metadata — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Webpage_metadata
- MDN Web Docs — Metadata — https://developer.mozilla.org/en-US/docs/Glossary/Metadata
- W3C — Web Content Accessibility Guidelines (WCAG) — https://www.w3.org/WAI/standards-guidelines/wcag/
- web.dev — Metadata — https://web.dev/learn/html/metadata
- Google Search Central — Title links — https://developers.google.com/search/docs/appearance/title-link
- web.dev — Preload critical assets to improve loading speed — https://web.dev/articles/preload-critical-assets

---

## Summary Table of Document Metadata Elements

| Element | Primary Role | Key Attributes | Placement |
|---|---|---|---|
| `<html>` | Document root; language declaration | `lang`, `xmlns` | Outermost element |
| `<head>` | Metadata container | global attributes | First child of `<html>` |
| `<title>` | Document title | none (text only) | Inside `<head>` |
| `<base>` | Base URL and default target | `href`, `target` | Early in `<head>` |
| `<link>` | External resource relationships | `rel`, `href`, `as`, `media`, `integrity` | `<head>` (or body for body-ok types) |
| `<meta>` | Generic metadata | `charset`, `name`+`content`, `http-equiv`+`content` | `<head>` (or body for microdata) |
| `<style>` | Embedded CSS | `media`, `type` (omit), `title` | `<head>` (conventionally) |
| `<script>` | Embedded or external script | `src`, `type`, `async`, `defer`, `nomodule`, `integrity` | `<head>` or `<body>` |
| `<noscript>` | Script-disabled fallback | global attributes | `<head>` (metadata children) or `<body>` (transparent) |

---

## Final Notes on Deprecated, Obsolete, and Version-Specific Features

- **`profile` on `<head>`**: Obsolete; do not use.
- **`version` on `<html>`**: Obsolete; version information is carried by the DOCTYPE.
- **`scoped` on `<style>`**: Removed from the specification; not supported in modern browsers.
- **`http-equiv="content-type"`**: Obsolete; use `<meta charset>` instead.
- **`http-equiv="set-cookie"`**: Obsolete and should not be used.
- **`user-scalable=no` and low `maximum-scale` in viewport**: Accessibility failures; do not use.
- **`keywords` meta name**: Largely ignored by modern search engines; not harmful but not effective.
- **Multiple `<base>` elements**: Only the first `href` and first `target` are honoured; additional elements are ignored.

---

This cheat sheet has covered the document metadata elements — `<html>`, `<head>`, `<title>`, `<base>`, `<link>`, `<meta>`, `<style>`, `<script>`, and `<noscript>` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate metadata markup against the WHATWG HTML Living Standard and accessibility guidelines.