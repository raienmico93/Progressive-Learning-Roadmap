# Applying CSS

CSS can be applied to an HTML document in four ways: **inline**, **internal**, **external**, and via **`@import`**. Each method has distinct trade-offs in maintainability, performance, and specificity. Choosing the right approach is a foundational decision in any web project.

---

## 1. Inline CSS

### 1.1 The `style` Attribute

**Inline CSS** is applied directly to a single HTML element using the `style` attribute.

```html
<p style="color: navy; font-size: 1.25rem;">Hello, world!</p>
```

The attribute's value is a **declaration block** — the same `property: value;` pairs used inside a CSS rule set, but without the selector or braces.

### 1.2 Syntax

```html
<element style="property1: value1; property2: value2;">
```

**Examples:**

```html
<div style="background: #f4f4f4; padding: 1rem; border-radius: 8px;">
  Content
</div>

<button style="background: #0066cc; color: white; border: none; padding: 0.5rem 1rem;">
  Click me
</button>

<img src="photo.jpg" style="width: 100%; height: auto; display: block;">
```

### 1.3 When Inline CSS Is Applied

- Only to the **specific element** it is written on
- Does **not** require a selector
- Cannot be reused across elements
- Cannot use pseudo-classes (`:hover`, `:focus`) or pseudo-elements (`::before`)
- Cannot use media queries or other at-rules

### 1.4 Specificity

Inline styles have **very high specificity** — higher than any selector-based rule:

| Source | Specificity |
|---|---|
| Inline style | (1, 0, 0, 0) — highest of all normal author styles |
| ID selector | (0, 1, 0, 0) |
| Class / attribute / pseudo-class | (0, 0, 1, 0) |
| Type / pseudo-element | (0, 0, 0, 1) |
| Universal | (0, 0, 0, 0) |

Only `!important` declarations from other stylesheets can override inline styles.

```html
<p style="color: red;">Red text</p>
```

```css
p { color: blue !important; }   /* overrides inline */
```

### 1.5 Use Cases

| Use Case | Reason |
|---|---|
| **Email HTML** | Email clients often strip `<style>` and `<link>` |
| **Quick prototypes** | Fast iteration without a stylesheet |
| **Dynamic styles via JavaScript** | Setting `element.style.property` |
| **Third-party widget overrides** | Overriding styles in embedded widgets |
| **One-off styling** | A single element with unique styling |

### 1.6 Advantages

| Advantage | Explanation |
|---|---|
| **Immediate** | Style is applied exactly where the element is defined |
| **No selector needed** | Directly targets the element |
| **Highest normal specificity** | Overrides most other styles |
| **Works in email clients** | Many email clients strip `<style>` |
| **Simple for one-offs** | No need to define a class or ID |

### 1.7 Limitations

| Limitation | Explanation |
|---|---|
| **No reuse** | Each element must repeat the same styles |
| **No pseudo-classes** | Cannot use `:hover`, `:focus`, `::before` |
| **No media queries** | Cannot respond to viewport or device |
| **Bloats HTML** | Mixes structure with presentation |
| **Hard to maintain** | Changes require editing every element |
| **Overrides everything** | Hard to override without `!important` |
| **CSP restrictions** | Some Content Security Policies block inline styles |
| **No caching** | Inline styles cannot be cached separately |

### 1.8 JavaScript Manipulation

The `style` property on DOM elements maps to inline styles:

```javascript
const el = document.querySelector('.box');

el.style.color = 'red';
el.style.fontSize = '2rem';
el.style.setProperty('--brand', '#0066cc');
```

This is equivalent to writing the styles in the `style` attribute.

**Difference between `.style` and `setAttribute('style', ...)`:**

| Method | Behavior |
|---|---|
| `el.style.color = 'red'` | Sets a single property |
| `el.style.cssText = 'color: red;'` | Replaces all inline styles |
| `el.setAttribute('style', 'color: red;')` | Same as `cssText` |

---

## 2. Internal CSS

### 2.1 The `<style>` Element

**Internal CSS** (also called **embedded CSS**) is written inside a `<style>` element in the document's `<head>`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Internal CSS Example</title>
  <style>
    h1 {
      color: navy;
      font-family: Georgia, serif;
    }

    p {
      line-height: 1.6;
      color: #333;
    }

    .button {
      background: #0066cc;
      color: white;
      padding: 0.5rem 1rem;
      border: none;
      border-radius: 4px;
    }
  </style>
</head>
<body>
  <h1>Hello</h1>
  <p>This is a paragraph.</p>
  <button class="button">Click me</button>
</body>
</html>
```

### 2.2 Syntax

```html
<style>
  selector {
    property: value;
  }
</style>
```

The `<style>` element:

- Belongs in `<head>` (though browsers tolerate it in `<body>`)
- Contains one or more **rule sets**
- Can contain **at-rules** (`@media`, `@supports`, `@font-face`, `@keyframes`)
- Can contain **comments** (`/* ... */`)
- Does **not** support HTML comments (`<!-- -->`) for hiding — this is legacy and unnecessary

### 2.3 Multiple `<style>` Elements

A document can contain multiple `<style>` elements. They are all parsed, and rules cascade in document order:

```html
<style>
  p { color: red; }
</style>

<style>
  p { color: blue; }   /* wins — later in document order */
</style>
```

### 2.4 Specificity

Internal CSS has **the same specificity as external CSS**. The location (internal vs. external) does **not** affect specificity.

**Cascade order when specificity ties:**

| Order | Source |
|---|---|
| 1 (earliest) | User-agent styles |
| 2 | User styles |
| 3 | Author styles — external `<link>` |
| 4 | Author styles — internal `<style>` |
| 5 | Author styles — inline `style` attribute |
| 6 (latest) | Author `!important` declarations |

When **specificity is equal**, the **later source in document order wins** — so internal `<style>` placed after a `<link>` overrides the linked stylesheet's rules (for equal specificity).

### 2.5 Use Cases

| Use Case | Reason |
|---|---|
| **Single-page sites** | One page, one place for styles |
| **Critical CSS** | Inline the above-the-fold styles for faster first paint |
| **Email HTML (some clients)** | Some clients support `<style>` |
| **Prototyping** | Quick iteration without separate files |
| **Component-level demos** | Self-contained examples |
| **Third-party embeds** | Isolated widgets with their own styles |

### 2.6 Advantages

| Advantage | Explanation |
|---|---|
| **Centralized for the page** | All page styles in one place |
| **No extra HTTP request** | Styles arrive with the HTML |
| **Supports all selectors** | Classes, IDs, pseudo-classes, media queries |
| **Supports at-rules** | `@media`, `@supports`, `@keyframes`, `@font-face` |
| **No FOUC for that page** | Styles are available when HTML parses |
| **Good for single-page sites** | No need for a separate file |

### 2.7 Limitations

| Limitation | Explanation |
|---|---|
| **Not reusable across pages** | Every page needs its own `<style>` |
| **Duplication** | Same styles copied on each page |
| **No caching** | Cannot be cached across page loads |
| **Bloats HTML** | Increases HTML size |
| **No parallel loading** | Styles block HTML parsing until parsed |
| **Mixes concerns** | Presentation inside the HTML document |
| **Harder to maintain at scale** | Multiple pages drift out of sync |

### 2.8 Critical CSS

A common modern technique is to **inline critical CSS** in `<style>` and load the rest asynchronously:

```html
<head>
  <!-- Critical CSS: above-the-fold styles -->
  <style>
    /* Minimal styles for first paint */
    body { font-family: system-ui, sans-serif; margin: 0; }
    header { background: navy; color: white; padding: 1rem; }
  </style>

  <!-- Non-critical CSS: loaded asynchronously -->
  <link rel="preload" href="styles.css" as="style" onload="this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="styles.css"></noscript>
</head>
```

This improves **First Contentful Paint (FCP)** and **Largest Contentful Paint (LCP)**.

---

## 3. External CSS

### 3.1 The `<link>` Element

**External CSS** is stored in a separate `.css` file and linked to the HTML document via a `<link>` element in `<head>`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>External CSS Example</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h1>Hello</h1>
</body>
</html>
```

```css
/* styles.css */
h1 {
  color: navy;
  font-family: Georgia, serif;
}
```

### 3.2 Syntax

```html
<link rel="stylesheet" href="path/to/styles.css">
```

**Common attributes:**

| Attribute | Purpose |
|---|---|
| `rel="stylesheet"` | Declares the link type (required) |
| `href` | Path to the CSS file (required) |
| `media` | Media query for conditional loading |
| `disabled` | Disables the stylesheet |
| `crossorigin` | CORS mode for cross-origin stylesheets |
| `integrity` | Subresource Integrity hash |
| `referrerpolicy` | Referrer policy for the request |

### 3.3 Examples

**Basic:**

```html
<link rel="stylesheet" href="styles.css">
```

**Absolute URL:**

```html
<link rel="stylesheet" href="https://cdn.example.com/styles.css">
```

**Media-specific:**

```html
<link rel="stylesheet" href="print.css" media="print">
<link rel="stylesheet" href="mobile.css" media="(max-width: 600px)">
```

**With Subresource Integrity:**

```html
<link rel="stylesheet"
      href="https://cdn.example.com/bootstrap.css"
      integrity="sha384-...">
```

**Preloading for performance:**

```html
<link rel="preload" href="styles.css" as="style">
<link rel="stylesheet" href="styles.css">
```

### 3.4 Multiple Stylesheets

Multiple `<link>` elements are allowed. They cascade in document order:

```html
<link rel="stylesheet" href="reset.css">
<link rel="stylesheet" href="base.css">
<link rel="stylesheet" href="components.css">
<link rel="stylesheet" href="utilities.css">
```

When specificity is equal, later files override earlier ones.

### 3.5 Specificity

External CSS has **the same specificity as internal CSS**. Source location does not change specificity.

**Cascade order (lowest to highest priority, equal specificity):**

```
External <link>  →  Internal <style>  →  Inline style
```

### 3.6 Use Cases

| Use Case | Reason |
|---|---|
| **Multi-page sites** | Reuse across all pages |
| **Any production website** | Best practice for maintainability |
| **Team development** | Separation of concerns for collaboration |
| **Cacheable styles** | Browser caches across page loads |
| **Large projects** | Modular CSS files |

### 3.7 Advantages

| Advantage | Explanation |
|---|---|
| **Reusable** | One file serves many pages |
| **Cacheable** | Browser caches, reducing repeat downloads |
| **Maintainable** | Changes in one place affect all pages |
| **Separates concerns** | HTML for structure, CSS for presentation |
| **Parallel loading** | Can be preloaded; browser fetches concurrently |
| **Modular** | Can split into multiple files |
| **Tooling-friendly** | Works with bundlers, preprocessors, linters |
| **Team-friendly** | Multiple developers can work on separate files |

### 3.8 Limitations

| Limitation | Explanation |
|---|---|
| **Extra HTTP request** | One more round trip on first load |
| **Render-blocking** | Browser waits for CSS before rendering |
| **FOUC risk** | Flash of unstyled content if loading is slow |
| **Path dependency** | Broken `href` breaks the entire stylesheet |
| **CORS** | Cross-origin stylesheets need proper headers |

### 3.9 Performance Optimization

**1. Preload critical stylesheets:**

```html
<link rel="preload" href="styles.css" as="style">
<link rel="stylesheet" href="styles.css">
```

**2. Load non-critical CSS asynchronously:**

```html
<link rel="stylesheet" href="styles.css"
      media="print" onload="this.media='all'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>
```

**3. Use HTTP/2 or HTTP/3** — multiplexing reduces the cost of multiple stylesheets.

**4. Minify and compress** — remove whitespace and enable gzip/brotli.

**5. Concatenate** — combine multiple CSS files into one for HTTP/1.1.

**6. Content-hash filenames** — enable long-term caching:

```html
<link rel="stylesheet" href="styles.a1b2c3d4.css">
```

---

## 4. CSS `@import`

### 4.1 The `@import` Rule

`@import` allows one CSS file to import another. It is written **inside a CSS file or `<style>` element** — not in HTML.

```css
/* main.css */
@import url("reset.css");
@import url("typography.css");
@import url("layout.css");

body {
  font-family: system-ui, sans-serif;
}
```

### 4.2 Syntax

```css
@import url("path/to/styles.css");
@import "path/to/styles.css";           /* url() optional */
@import url("print.css") print;
@import url("mobile.css") (max-width: 600px);
@import url("styles.css") supports(display: grid);
```

**Optional media/feature conditions:**

| Syntax | Meaning |
|---|---|
| `@import "styles.css";` | Always applied |
| `@import "styles.css" print;` | Applied for print |
| `@import "styles.css" (max-width: 600px);` | Applied when the media query matches |
| `@import "styles.css" supports(display: grid);` | Applied if feature is supported |

### 4.3 Placement Rules

`@import` **must come before all other rules** (except `@charset` and `@layer` statements). Otherwise, it is ignored.

```css
/* ✓ Valid */
@charset "UTF-8";
@import url("reset.css");
@import url("base.css");

body { margin: 0; }
```

```css
/* ✗ Invalid — @import after a rule is ignored */
body { margin: 0; }
@import url("reset.css");   /* ignored */
```

### 4.4 Use Cases

| Use Case | Reason |
|---|---|
| **CSS module organization** | Split styles into logical files |
| **Conditional loading** | Load styles based on media query |
| **Feature detection** | Load styles based on `@supports` |
| **Vendor CSS** | Import third-party stylesheets |

### 4.5 Advantages

| Advantage | Explanation |
|---|---|
| **Modularity** | Break CSS into manageable files |
| **Single `<link>`** | Only one HTML link needed |
| **Conditional loading** | Load stylesheets based on media/features |
| **Organization** | Logical separation of concerns within CSS |

### 4.6 Limitations

| Limitation | Explanation |
|---|---|
| **Serial requests** | Imports load sequentially, not in parallel |
| **Performance cost** | Each import is a separate HTTP request |
| **Render-blocking** | Blocks rendering until all imports resolve |
| **No preload** | Browsers can't preload `@import` dependencies |
| **Hard to debug** | DevTools shows imported files separately |
| **Deprecated in practice** | Modern tooling prefers bundlers |

### 4.7 `@import` vs. `<link>`

| Aspect | `<link>` | `@import` |
|---|---|---|
| **Location** | HTML | CSS |
| **Parallel loading** | Yes | No (serial) |
| **Performance** | Better | Worse |
| **Preload support** | Yes | No |
| **Conditional loading** | `media` attribute | Media query syntax |
| **Cascade position** | Wherever `<link>` appears | Must be at top of CSS |
| **Modern recommendation** | **Preferred** | Avoid in production |

**Key performance issue:** When the browser encounters `<link rel="stylesheet" href="main.css">`, it starts fetching `main.css`. Only after parsing it does it discover `@import url("reset.css")` and start fetching `reset.css`. This **serialization** delays rendering.

**Modern alternative:** Use multiple `<link>` elements, which the browser fetches in parallel.

```html
<!-- Better: parallel loading -->
<link rel="stylesheet" href="reset.css">
<link rel="stylesheet" href="typography.css">
<link rel="stylesheet" href="layout.css">
```

### 4.8 `@import` in `<style>`

`@import` can also appear inside a `<style>` element:

```html
<style>
  @import url("reset.css");
  @import url("base.css");

  body { margin: 0; }
</style>
```

This has the same serialization cost as `@import` in an external file.

### 4.9 Modern Alternatives to `@import`

| Technique | Purpose |
|---|---|
| **Multiple `<link>` elements** | Parallel loading |
| **CSS bundlers** (Vite, Webpack, esbuild) | Combine at build time |
| **CSS preprocessors** (Sass, Less) | `@use`, `@import` resolved at build time |
| **HTTP/2 Server Push** | Deprecated; preload is preferred |
| **`<link rel="preload">`** | Hint the browser to fetch early |

**Recommendation:** Avoid `@import` in production. Use multiple `<link>` elements or a bundler.

---

## 5. Advantages and Limitations of Each Approach

### 5.1 Comparison Matrix

| Aspect | Inline | Internal | External | `@import` |
|---|---|---|---|---|
| **Location** | `style` attribute | `<style>` in `<head>` | Separate `.css` file via `<link>` | Inside CSS or `<style>` |
| **Reusable** | ✗ | ✗ (per page) | ✓ | ✓ |
| **Cacheable** | ✗ | ✗ | ✓ | ✓ |
| **Parallel loading** | N/A | N/A | ✓ | ✗ |
| **Supports selectors** | ✗ | ✓ | ✓ | ✓ |
| **Supports pseudo-classes** | ✗ | ✓ | ✓ | ✓ |
| **Supports media queries** | ✗ | ✓ | ✓ | ✓ |
| **Specificity** | Highest | Same as external | Same as internal | Same as external |
| **Separation of concerns** | ✗ | Partial | ✓ | ✓ |
| **Maintainability** | Low | Medium | High | High |
| **Performance** | No extra request | No extra request | One extra request | Multiple serial requests |
| **CSP-friendly** | Often blocked | Often blocked | ✓ | ✓ |
| **Recommended for production** | Rarely | Sometimes (critical CSS) | ✓ | ✗ |

### 5.2 Performance Comparison

| Approach | First Load | Repeat Load | FOUC Risk |
|---|---|---|---|
| **Inline** | Fast (no request) | Slow (no cache) | None |
| **Internal** | Fast (no request) | Slow (no cache) | None |
| **External** | One request, render-blocking | Fast (cached) | Possible |
| **`@import`** | Multiple serial requests | Depends | High |

### 5.3 Cascade Order

When multiple approaches are used, the cascade (for equal specificity) is:

```
1. External <link> (earliest in document)
2. Internal <style> (later in document)
3. Inline style attribute
4. !important declarations (highest priority, regardless of source)
```

### 5.4 Choosing the Right Approach

| Scenario | Recommended Approach |
|---|---|
| **Production multi-page site** | External `<link>` |
| **Single-page site** | External `<link>` + critical CSS inline |
| **Email HTML** | Inline |
| **Prototype** | Internal `<style>` |
| **Component demo** | Internal `<style>` or scoped styles |
| **Third-party widget** | Inline or Shadow DOM |
| **Performance-critical first paint** | Inline critical CSS + async external |
| **Team project** | External `<link>` with bundled files |
| **Modern SPA** | Bundler + external `<link>` |

---

## 6. Separation of Concerns

### 6.1 The Principle

**Separation of concerns** is a design principle that holds that each part of a system should focus on a single responsibility.

In web development, this means:

| Technology | Responsibility |
|---|---|
| **HTML** | Structure and content |
| **CSS** | Presentation and layout |
| **JavaScript** | Behavior and interactivity |

### 6.2 Why It Matters

| Benefit | Explanation |
|---|---|
| **Maintainability** | Change styles without touching HTML |
| **Reusability** | One stylesheet styles many pages |
| **Collaboration** | Designers edit CSS; developers edit HTML/JS |
| **Caching** | CSS and JS cached separately from HTML |
| **Accessibility** | Semantic HTML isn't obscured by presentation |
| **Testing** | Each layer can be tested independently |

### 6.3 Anti-Patterns

**1. Inline styles for repeated elements:**

```html
<!-- ✗ Repetitive, hard to maintain -->
<p style="color: navy; font-size: 1rem;">Paragraph 1</p>
<p style="color: navy; font-size: 1rem;">Paragraph 2</p>
<p style="color: navy; font-size: 1rem;">Paragraph 3</p>
```

**2. Styling hooks in HTML:**

```html
<!-- ✗ Presentation classes in HTML -->
<div class="red-text bold-text large-text">Content</div>
```

**3. `<br>` for spacing:**

```html
<!-- ✗ Using markup for layout -->
<p>Paragraph 1</p>
<br><br><br>
<p>Paragraph 2</p>
```

**4. Presentational HTML tags:**

```html
<!-- ✗ Deprecated presentational tags -->
<center><font color="red">Heading</font></center>
```

### 6.4 Best Practices

**1. Keep HTML semantic:**

```html
<!-- ✓ Structure only -->
<article class="post">
  <header>
    <h1>Article Title</h1>
  </header>
  <p>Content...</p>
</article>
```

**2. Use external stylesheets:**

```html
<link rel="stylesheet" href="styles.css">
```

**3. Use class names for styling, not IDs:**

```html
<!-- ✓ Classes are reusable -->
<button class="btn btn-primary">Save</button>
```

**4. Prefer semantic class names:**

```html
<!-- ✗ Presentational -->
<div class="red-box big-text">Content</div>

<!-- ✓ Semantic -->
<div class="alert alert-error">Content</div>
```

**5. Avoid `!important` and inline styles:**

```css
/* ✗ Avoid unless absolutely necessary */
.error { color: red !important; }
```

**6. Use CSS custom properties for theming:**

```css
:root {
  --brand: #0066cc;
  --error: #cc0000;
}
```

### 6.5 Pragmatic Exceptions

Separation of concerns is a **principle, not a religion**. Legitimate exceptions exist:

| Exception | Reason |
|---|---|
| **Email HTML** | Clients strip `<style>` and `<link>` |
| **Critical CSS** | Inline above-the-fold styles for performance |
| **Dynamic styles** | JavaScript sets inline styles for animations or state |
| **Third-party widget isolation** | Shadow DOM or scoped styles |
| **CSS-in-JS** | Component-scoped styles (React, Vue) |

### 6.6 Modern Perspectives

The strict separation of HTML/CSS/JS has evolved with modern frameworks:

**Traditional separation:**

```
index.html  → structure
styles.css  → presentation
app.js      → behavior
```

**Component-based separation (React, Vue, Svelte):**

```
Button.jsx  → structure + behavior + scoped styles
Card.vue    → template + script + scoped styles
```

In modern frameworks, **colocation** replaces strict separation — each component encapsulates its own structure, behavior, and styles. This is a **different kind of separation** — separating by **feature/component** rather than by **technology**.

Both approaches have merits:

| Approach | Strength |
|---|---|
| **Technology separation** | Clear responsibility boundaries; reusable stylesheets |
| **Component colocation** | Encapsulation; fewer cross-file lookups |

The right choice depends on project scale, team structure, and framework.

---

## Full Example — All Four Approaches Together

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Applying CSS — All Approaches</title>

  <!-- 3. External CSS (preferred for production) -->
  <link rel="stylesheet" href="styles.css">

  <!-- 4. @import inside a <style> element (avoid in production) -->
  <style>
    /* @import must come first */
    @import url("legacy.css");

    /* 2. Internal CSS */
    h1 {
      font-family: Georgia, serif;
    }

    .highlight {
      background: yellow;
    }
  </style>
</head>
<body>
  <!-- 1. Inline CSS -->
  <h1 style="color: navy;">Title</h1>

  <p class="highlight">Highlighted paragraph.</p>

  <p style="color: red; font-weight: bold;">Inline-styled warning.</p>
</body>
</html>
```

```css
/* styles.css — External CSS */
body {
  font-family: system-ui, sans-serif;
  line-height: 1.6;
  margin: 0;
  padding: 2rem;
}

h1 {
  font-size: 2rem;
  margin-top: 0;
}

p {
  margin-bottom: 1rem;
}
```

---

## Summary Table

| Approach | Where | Reusable | Cacheable | Best For |
|---|---|---|---|---|
| **Inline** | `style` attribute | ✗ | ✗ | Emails, one-offs, dynamic JS |
| **Internal** | `<style>` in `<head>` | Per page | ✗ | Single-page sites, critical CSS |
| **External** | `<link>` to `.css` | ✓ | ✓ | Production sites (recommended) |
| **`@import`** | Inside CSS | ✓ | ✓ | Avoid; use `<link>` instead |

---

## Key Takeaways

1. **Inline CSS** uses the `style` attribute — highest specificity, no reuse, no pseudo-classes, no media queries. Best for emails and dynamic JS.
2. **Internal CSS** uses a `<style>` element in `<head>` — supports all selectors and at-rules, but is not reusable across pages.
3. **External CSS** uses `<link rel="stylesheet">` — the **recommended approach** for production: reusable, cacheable, and maintainable.
4. **`@import`** loads CSS from within CSS — but causes **serial requests** and blocks rendering. Prefer multiple `<link>` elements or a bundler.
5. **Specificity is the same for internal and external CSS** — location does not affect specificity.
6. **Cascade order** (equal specificity): external `<link>` → internal `<style>` → inline `style` → `!important`.
7. **Separation of concerns** — HTML for structure, CSS for presentation, JavaScript for behavior.
8. **Avoid inline styles** for anything reusable — they bloat HTML and break separation of concerns.
9. **Critical CSS** — inline the above-the-fold styles for faster first paint; load the rest asynchronously.
10. **Modern frameworks** colocate structure, styles, and behavior per component — a different kind of separation that prioritizes encapsulation over technology boundaries.

---

Would you like me to continue with the next topic — **CSS Selectors in Depth**, **CSS Specificity and the Cascade**, or **CSS Inheritance**? I can format the next section in the same style.