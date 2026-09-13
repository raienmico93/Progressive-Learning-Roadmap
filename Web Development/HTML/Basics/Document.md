# Basic HTML Document Structure

Every HTML document follows a standard structure that browsers rely on to correctly parse and render content. This guide breaks down each component of a minimal HTML document, from the `<!DOCTYPE html>` declaration to the closing `</html>` tag.

---

## 1. The Minimal HTML Document

Here's the simplest valid HTML5 document:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Document Title</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <p>This is a paragraph.</p>
  </body>
</html>
```

Every element in this example has a specific purpose. Let's break it down piece by piece.

---

## 2. The `<!DOCTYPE html>` Declaration

The **DOCTYPE** (Document Type Declaration) is the very first line of an HTML document. It tells the browser which version of HTML the document uses.

### In HTML5

```html
<!DOCTYPE html>
```

This is all you need. It's short, case-insensitive (though lowercase is conventional), and triggers **standards mode** in all modern browsers.

### Why It Matters

Browsers have two rendering modes:

| Mode | Trigger | Behavior |
|---|---|---|
| **Standards Mode** | Valid `<!DOCTYPE html>` | Renders according to modern W3C/WHATWG specs |
| **Quirks Mode** | Missing or malformed DOCTYPE | Mimics legacy browser behavior for backward compatibility |

**Consequences of quirks mode:**
- Inconsistent box model calculations
- Different default styles
- Unpredictable rendering across browsers

### Historical Context

Older DOCTYPEs were long and complex:

```html
<!-- HTML 4.01 Strict -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
  "http://www.w3.org/TR/html4/strict.dtd">

<!-- XHTML 1.0 Strict -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```

HTML5 simplified this dramatically. **Always use `<!DOCTYPE html>`** — there's no reason not to in modern development.

### Common Mistake

```html
<!-- WRONG: This triggers quirks mode -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">

<!-- RIGHT -->
<!DOCTYPE html>
```

---

## 3. The Root Element: `<html>`

The `<html>` element is the **root element** of every HTML document. It contains all other elements (except the DOCTYPE).

### Basic Syntax

```html
<html lang="en">
  <!-- head and body go here -->
</html>
```

### The `lang` Attribute

The `lang` attribute declares the **primary language** of the document's content. It's not required for the page to render, but it's essential for:

- **Screen readers**: Correct pronunciation
- **Search engines**: Language-specific indexing
- **Translation tools**: Knowing the source language
- **Hyphenation and spell-check**: Language-appropriate rules

**Common values (BCP 47 language tags):**

| Code | Language |
|---|---|
| `en` | English |
| `en-US` | English (United States) |
| `en-GB` | English (United Kingdom) |
| `es` | Spanish |
| `fr` | French |
| `de` | German |
| `ja` | Japanese |
| `zh-CN` | Chinese (Simplified) |
| `fil` | Filipino |

**Best practice:** Use the most specific tag that applies. If your content is in US English, use `en-US`.

### Only One `<html>` Element

A valid HTML document has exactly **one** `<html>` element. It is the outermost container for all other content.

---

## 4. Document Structure Overview

Inside `<html>`, there are exactly **two** children:

```
<html>
├── <head>   → Metadata section (not displayed)
└── <body>   → Document content section (displayed)
```

This two-part division is fundamental to HTML:

- **`<head>`** contains information *about* the document.
- **`<body>`** contains the document's actual content that users see.

---

## 5. The Metadata Section: `<head>`

The `<head>` element contains **metadata** — information about the document that is not rendered in the browser viewport (with rare exceptions like `<title>`).

### Required Elements

#### `<meta charset="UTF-8">`

Declares the **character encoding** of the document. It should be the **first** element inside `<head>`.

```html
<meta charset="UTF-8">
```

**Why it matters:**
- Ensures correct display of special characters (accents, emojis, non-Latin scripts)
- Prevents garbled text (mojibake)
- Should be within the first 1024 bytes of the document

**Common encodings:**

| Encoding | Use Case |
|---|---|
| `UTF-8` | Universal — use this for all modern sites |
| `ISO-8859-1` | Legacy Western European |
| `Windows-1252` | Legacy Windows Western |

#### `<title>`

Defines the document's title, shown in:

- Browser tab
- Bookmarks
- Search engine results
- Browser history

```html
<title>My Awesome Website | Home</title>
```

**Best practices:**
- Keep under ~60 characters to avoid truncation in search results
- Use a consistent format (e.g., "Page Name | Site Name")
- Every page should have a unique title

### Commonly Used Elements

#### `<meta name="viewport">`

Essential for responsive design on mobile devices:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

Without this, mobile browsers render the page at desktop width and zoom out.

#### `<meta name="description">`

Provides a summary used by search engines:

```html
<meta name="description" content="Learn HTML structure, semantics, and best practices.">
```

#### `<meta name="author">`

```html
<meta name="author" content="Jane Developer">
```

#### Linking External Resources

```html
<!-- Stylesheet -->
<link rel="stylesheet" href="css/style.css">

<!-- Favicon -->
<link rel="icon" href="assets/favicon.ico" type="image/x-icon">

<!-- Preload critical resources -->
<link rel="preload" href="fonts/main.woff2" as="font" type="font/woff2" crossorigin>
```

#### Embedding or Linking Scripts

```html
<!-- External script (deferred) -->
<script src="js/main.js" defer></script>

<!-- Inline script -->
<script>
  console.log("Page loaded");
</script>
```

**Note:** Scripts can also go in `<body>` — often at the end for performance reasons — but `<head>` is valid, especially with `defer` or `async`.

#### `<style>` — Inline CSS

```html
<style>
  body { font-family: sans-serif; }
</style>
```

### Full `<head>` Example

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="A sample HTML document demonstrating proper structure.">
  <meta name="author" content="Jane Developer">
  <title>Sample Page | My Site</title>
  <link rel="stylesheet" href="css/style.css">
  <link rel="icon" href="assets/favicon.ico">
  <script src="js/main.js" defer></script>
</head>
```

### Key Rules for `<head>`

- Exactly **one** `<head>` element per document
- Must come **before** `<body>`
- Metadata elements are **not displayed** in the viewport
- Some elements (`<title>`, `<meta>`, `<link>`, `<style>`, `<script>`, `<base>`) are **void or metadata-only** elements

---

## 6. The Document Content Section: `<body>`

The `<body>` element contains all the **visible content** of the document — everything the user sees and interacts with.

### What Goes Inside `<body>`

- Headings (`<h1>`–`<h6>`)
- Paragraphs (`<p>`)
- Links (`<a>`)
- Images (`<img>`)
- Lists (`<ul>`, `<ol>`, `<li>`)
- Tables (`<table>`, `<tr>`, `<td>`)
- Forms (`<form>`, `<input>`, `<button>`)
- Sections (`<header>`, `<main>`, `<article>`, `<section>`, `<footer>`)
- Scripts and other interactive content

### Basic Example

```html
<body>
  <header>
    <h1>Welcome to My Site</h1>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
    </nav>
  </header>

  <main>
    <article>
      <h2>First Post</h2>
      <p>This is the content of my first post.</p>
    </article>
  </main>

  <footer>
    <p>&copy; 2026 My Site</p>
  </footer>
</body>
```

### Key Rules for `<body>`

- Exactly **one** `<body>` element per document
- Must come **after** `<head>`
- Contains all rendered content
- Can contain `<script>` elements (often at the end for performance)

---

## 7. Complete Document Structure Diagram

```
<!DOCTYPE html>              ← Declaration (not an element)
<html lang="en">             ← Root element
│
├── <head>                   ← Metadata section (not rendered)
│   ├── <meta charset="UTF-8">
│   ├── <meta name="viewport" ...>
│   ├── <title>...</title>
│   ├── <link rel="stylesheet" ...>
│   └── <script src="..." defer></script>
│
└── <body>                   ← Content section (rendered)
    ├── <header>...</header>
    ├── <main>...</main>
    └── <footer>...</footer>
</html>
```

---

## 8. Whitespace and Formatting

HTML is **whitespace-insensitive** in most cases:

```html
<p>Hello     World</p>
<p>Hello World</p>
```

These render identically. However:

- **Indentation** improves readability for developers
- **Consistent formatting** aids maintenance
- Whitespace inside `<pre>` or with CSS `white-space` is preserved

**Recommended formatting:**
- 2 spaces (or 4) per indentation level
- One element per line for block-level content
- Attributes on the same line unless very long

---

## 9. Common Mistakes to Avoid

| Mistake | Problem |
|---|---|
| Missing `<!DOCTYPE html>` | Triggers quirks mode |
| Missing `<html>` element | Invalid document structure |
| Missing `<head>` or `<body>` | Browsers auto-insert them, but implicit structure is error-prone |
| Content directly in `<html>` outside head/body | Browsers move it to `<body>`, but it's invalid |
| `<meta charset>` not first in `<head>` | May cause encoding issues |
| Multiple `<head>` or `<body>` elements | Invalid; only the first is used |
| Missing `lang` attribute | Hurts accessibility and SEO |
| Forgetting `<title>` | Tab shows URL; bad for UX and SEO |

---

## 10. Auto-Generated Boilerplate

Most editors generate this boilerplate for you:

**VS Code:** Type `!` and press Tab in an empty `.html` file.

**Result:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>

</body>
</html>
```

This is a solid starting point — just change the `lang`, `title`, and start adding content to `<body>`.

---

## Key Takeaways

1. **`<!DOCTYPE html>`** must be the first line — it triggers standards mode.
2. **`<html>`** is the root element; include the `lang` attribute for accessibility.
3. **`<head>`** holds metadata (charset, viewport, title, links, scripts) — not rendered.
4. **`<body>`** holds all visible content — everything the user sees.
5. A valid document has **exactly one** `<html>`, `<head>`, and `<body>`.
6. `<meta charset="UTF-8">` should be the **first** element in `<head>`.
7. **`<title>`** is required and affects tabs, bookmarks, and SEO.
8. Proper structure improves accessibility, SEO, and cross-browser consistency.

---

Would you like me to continue with **"3. HTML Elements and Tags"** next, or expand on any part of this section (e.g., a deeper dive into `<meta>` tags, or the `<head>` vs `<body>` rendering implications)?