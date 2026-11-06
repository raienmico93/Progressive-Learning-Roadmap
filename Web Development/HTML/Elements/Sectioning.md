# Sectioning Elements — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Sectioning elements are the HTML elements that define the major structural regions of a document — its body, articles, sections, navigation, complementary content, headings, headers, footers, main content, and contact information. They create the semantic architecture that browsers, assistive technologies, and search engines use to understand and navigate a page.

**Technical Definition**
In the WHATWG HTML Living Standard, the sectioning elements are grouped under “Sections” (with `<main>` defined under “Grouping content”). The sectioning content elements — `<article>`, `<aside>`, `<nav>`, and `<section>` — create sections in the document’s outline, defining the scope of headings and of `<header>` and `<footer>` elements. The `<body>` element represents the document’s contents, the heading elements `<h1>`–`<h6>` and `<hgroup>` provide heading content, `<header>` and `<footer>` provide introductory and concluding content for their nearest sectioning ancestor, `<main>` represents the dominant content of the document, and `<address>` provides contact information for its nearest `<article>` or `<body>` ancestor.

**Beginner-Friendly Explanation**
Sectioning elements are the “rooms and walls” of a web page. Instead of dumping everything into anonymous `<div>` containers, you use sectioning elements to say “this is the navigation”, “this is the main article”, “this is a sidebar”, and “this is the footer”. This helps screen readers, search engines, and other tools understand the page’s structure and lets users jump directly to the part they need.

---

### Key Characteristics

- **Semantic structure over presentation**: Sectioning elements describe the role of content, not its visual appearance.
- **Landmark generation**: Many sectioning elements automatically create ARIA landmark regions — `<aside>` creates `complementary`, `<footer>` (in the context of `<body>`) creates `contentinfo`, `<header>` (in the context of `<body>`) creates `banner`, `<main>` creates `main`, `<nav>` creates `navigation`, and `<section>` creates `region` when it has an accessible name.
- **Heading scoping**: Sectioning content elements define the scope of heading elements and of `<header>` and `<footer>` elements.
- **Outline participation**: `<article>`, `<aside>`, `<nav>`, and `<section>` are sectioning content and participate in the document outline; `<body>`, `<blockquote>`, `<details>`, `<dialog>`, `<fieldset>`, `<figure>`, and `<td>` are sectioning roots that establish independent outlines.
- **Accessibility-critical**: Misusing sectioning elements — for example, using `<section>` as a styling wrapper, or skipping heading levels — creates confusion for assistive-technology users.
- **Content-model governed**: Each element has a defined content model; for example, `<header>` and `<footer>` may not contain descendant `<header>` or `<footer>` elements.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<!DOCTYPE html>`, `<html>`, `<head>`, and `<body>`.
2. **Flow content and phrasing content**: understanding the difference between block-level and inline content, and how content models constrain nesting.
3. **Heading hierarchy concepts**: awareness that `<h1>`–`<h6>` create a logical outline.
4. **Browser developer tools**: ability to inspect the DOM and use accessibility-tree or landmark panels.
5. **A screen reader or accessibility inspection tool**: useful for observing how landmark roles are announced.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | Sectioning elements map to landmark roles; correct use satisfies WCAG 1.3.1 (Info and Relationships), 2.4.1 (Bypass Blocks), and 2.4.10 (Section Headings). |
| **SEO** | Search engines use headings and semantic regions to understand page structure and topic hierarchy. |
| **CSS layout** | Sectioning elements provide structural hooks for grid and flexbox layouts. |
| **JavaScript / DOM** | Sectioning elements are DOM nodes that scripts can query and manipulate; `<main>`, `<nav>`, and `<aside>` provide stable navigation targets. |
| **Web components** | Custom elements often use sectioning elements internally to preserve semantics. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<body>`** — the document body
2. **`<article>`** — self-contained composition
3. **`<section>`** — generic thematic section
4. **`<nav>`** — navigation links
5. **`<aside>`** — tangentially related content
6. **`<h1>` to `<h6>`** — headings
7. **`<hgroup>`** — heading with subheading or tagline
8. **`<header>`** — introductory content for a section
9. **`<footer>`** — concluding content for a section
10. **`<main>`** — dominant content of the document
11. **`<address>`** — contact information
12. **Sectioning element best practices**

---

## 1. `<body>` — The Document Body Element

### Definitions

**Core Definition**
The `<body>` element represents the contents of the document. In conforming documents, there is only one `<body>` element.

**Technical Definition**
`<body>` has no content category. It may be used as the second element in an `<html>` element, after `<head>`. Its content model is **flow content**. The element exposes the `HTMLBodyElement` DOM interface and supports global attributes plus a set of window-reflecting event handler attributes such as `onload`, `ononline`, `onoffline`, and `onhashchange`.

**Beginner-Friendly Explanation**
The `<body>` is the part of the HTML file that actually appears on the page. Everything a user sees — text, images, buttons, videos — goes inside the `<body>`.

---

### Purposes

- **To contain all visible document content**: text, images, forms, and media.
- **To serve as the default sectioning root**: headings and sectioning elements inside `<body>` establish the page’s primary outline.
- **To expose document-level event handlers**: `onload`, `onresize`, `onscroll`, and similar events can be attached to `<body>`.
- **To provide a styling hook**: the `body` selector is commonly used for page-wide CSS (background, font, margin).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document Title</title>
</head>
<body>
  <!-- flow content -->
</body>
</html>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<body>` | The start tag; must follow `<head>`. |
| Flow content | Headings, paragraphs, lists, media, sectioning elements, and other visible content. |
| `</body>` | The end tag; may be omitted if not immediately followed by a comment. |

#### Syntax Rules

1. **Exactly one `<body>` per document** in conforming documents.
2. **`<body>` must be the second element in `<html>`**, after `<head>`.
3. **Content model is flow content**; phrasing-only content should be wrapped in appropriate flow containers.
4. **Tag omission is permitted** under specific conditions, but explicit tags are recommended for clarity.

#### Constraints and Limitations

- `<body>` must not contain `<head>` or `<html>`.
- Using `<body>` as a generic wrapper for a single component is inappropriate; use `<div>` or a sectioning element instead.
- Event handler content attributes on `<body>` mirror window events, which can be surprising if not understood.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Minimal Body with Sectioning Content

**Setup guide**: Save as `body-minimal.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Body Example</title>
</head>
<body>
  <!-- The body contains the visible content -->
  <header>
    <h1>Site Title</h1>
  </header>

  <main>
    <p>This is the main content of the page.</p>
  </main>

  <footer>
    <p>© 2026 Example Company</p>
  </footer>
</body>
</html>
```

**Expected output**: A page with a heading “Site Title” at the top, a paragraph in the middle, and a copyright line at the bottom.

**Why this output occurs**: The `<body>` contains all visible content. The `<header>`, `<main>`, and `<footer>` are flow-content children that structure the page into a banner landmark, a main landmark, and a contentinfo landmark (when `<header>` and `<footer>` are direct children of `<body>`).

---

#### Example 2: Body Event Handlers for Online/Offline Status

**Setup guide**: Save as `body-events.html` and toggle the browser’s network connection.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Body Events</title>
  <script>
    function updateStatus(online) {
      document.getElementById('status').textContent = online ? 'Online' : 'Offline';
    }
  </script>
</head>
<body ononline="updateStatus(true)" onoffline="updateStatus(false)" onload="updateStatus(navigator.onLine)">
  <p>You are: <span id="status">(Unknown)</span></p>
</body>
</html>
```

**Expected output**: On page load, the status displays “Online” or “Offline” based on the current network state. When the connection changes, the status updates automatically.

**Why this output occurs**: The `<body>` element exposes window-reflecting event handler attributes. The `onload`, `ononline`, and `onoffline` attributes are triggered by the corresponding window events, and the JavaScript updates the text content of the status span.

---

### Real-World Cases

1. **Every HTML page**: `<body>` is the universal container for visible content.
2. **Offline-aware web apps**: `ononline` and `onoffline` on `<body>` are used to show connection status banners.
3. **Page-wide event delegation**: Some frameworks attach delegated event listeners to `<body>` to handle events across the entire document.

---

### References

- WHATWG — The `body` element — https://html.spec.whatwg.org/multipage/sections.html#the-body-element
- MDN Web Docs — `<body>`: The Document Body element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/body
- W3C — HTML 5.2: The `body` element — https://www.w3.org/TR/html52/sections.html#the-body-element

---

## 2. `<article>` — Self-Contained Composition Element

### Definitions

**Core Definition**
The `<article>` element represents a complete, or self-contained, composition in a document, page, application, or site that is, in principle, independently distributable or reusable.

**Technical Definition**
`<article>` belongs to the **flow content**, **sectioning content**, and **palpable content** categories. It may be used where sectioning content is expected. Its content model is flow content. When `<article>` elements are nested, the inner articles represent content that is in principle related to the outer article.

**Beginner-Friendly Explanation**
An `<article>` is a piece of content that could stand on its own — a blog post, a news story, a forum post, a user comment, or a product card. If you could syndicate it or show it in a feed without the rest of the page, it is probably an `<article>`.

---

### Purposes

- **To mark up self-contained content** that can be distributed or reused independently.
- **To support syndication**: blog posts, news articles, and feed items are natural `<article>` candidates.
- **To enable nested article relationships**: comments can be `<article>` elements nested inside the main blog-post `<article>`.
- **To provide a semantic container** for a complete composition with its own heading, metadata, and footer.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<article>
  <header>
    <h2>Article Title</h2>
    <p>By Author · <time datetime="2026-11-06">6 November 2026</time></p>
  </header>
  <p>Article body content…</p>
  <footer>
    <a href="/comments">Show comments…</a>
  </footer>
</article>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<article>` | The container for a self-contained composition. |
| `<header>` | Optional introductory content, often containing the heading and byline. |
| Flow content | The article’s body: paragraphs, sections, media, nested articles. |
| `<footer>` | Optional concluding content, such as comment links or author information. |

#### Syntax Rules

1. **Content model is flow content**; the article may contain headings, sections, nested articles, and media.
2. **Author information associated with an `<article>` does not apply to nested articles**; use `<address>` carefully.
3. **Nested `<article>` elements represent related articles**, such as comments on a blog post.
4. **When the entire main content of a page is a single composition**, marking it as `<article>` is technically redundant but not harmful.

#### Constraints and Limitations

- `<article>` should not be used for arbitrary content that is not self-contained; use `<section>` or `<div>` instead.
- An `<article>` should generally have a heading to identify its content.
- Using `<article>` merely for styling is discouraged; choose the element for meaning.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Blog Post with Nested Comment Articles

**Setup guide**: Save as `article-blog.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Blog Post with Comments</title>
</head>
<body>
  <main>
    <!-- Outer article: the blog post -->
    <article>
      <header>
        <h1>The Very First Rule of Life</h1>
        <p>Published <time datetime="2026-11-01">5 days ago</time></p>
      </header>
      <p>If there’s a microphone anywhere near you, assume it’s hot and sending whatever you’re saying to the world.</p>

      <section>
        <h2>Comments</h2>

        <!-- Inner articles: user comments -->
        <article>
          <footer>
            <p>Posted by George Washington · <time datetime="2026-11-02">4 days ago</time></p>
          </footer>
          <p>Yeah! Especially when talking about your lobbyist friends!</p>
        </article>

        <article>
          <footer>
            <p>Posted by George Hammond · <time datetime="2026-11-02">4 days ago</time></p>
          </footer>
          <p>Hey, you have the same first name as me.</p>
        </article>
      </section>
    </article>
  </main>
</body>
</html>
```

**Expected output**: A blog post with a title, publication date, body paragraph, and a “Comments” section containing two comment articles, each with its own author and timestamp.

**Why this output occurs**: The outer `<article>` is the self-contained blog post. The nested `<article>` elements represent comments that are independently meaningful but related to the outer post. This nesting is explicitly supported by the specification.

---

#### Example 2: Using `<article>` for a Widget

**Setup guide**: Save as `article-widget.html`. This example illustrates `<article>` as an interactive widget container.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Widget Article</title>
</head>
<body>
  <article>
    <h2>Stock Ticker</h2>
    <table>
      <thead>
        <tr><th>Stock</th><th>Value</th><th>Delta</th></tr>
      </thead>
      <tbody>
        <tr><td>EXAMPLE</td><td>$123.45</td><td>+1.2%</td></tr>
      </tbody>
    </table>
    <button type="button">Refresh</button>
  </article>
</body>
</html>
```

**Expected output**: A self-contained stock-ticker widget with a heading, a table of stock data, and a refresh button.

**Why this output occurs**: The WHATWG specification explicitly permits `<article>` to be used for “an interactive widget or gadget” — any independent item of content. The widget is self-contained and could be syndicated or embedded elsewhere.

---

### Real-World Cases

1. **Blog and news sites**: Each post or story is an `<article>`; nested comments may also be `<article>` elements.
2. **Forum software**: Each forum post is an `<article>`, allowing threaded replies to be nested.
3. **Product listing pages**: Each product card can be an `<article>` if it is independently distributable (e.g. in a product feed).

---

### References

- WHATWG — The `article` element — https://html.spec.whatwg.org/multipage/sections.html#the-article-element
- MDN Web Docs — `<article>`: The Article Contents element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/article
- W3C — HTML 5.2: The `article` element — https://www.w3.org/TR/html52/sections.html#the-article-element

---

## 3. `<section>` — Generic Section Element

### Definitions

**Core Definition**
The `<section>` element represents a generic section of a document or application — a thematic grouping of content, typically with a heading.

**Technical Definition**
`<section>` belongs to the **flow content**, **sectioning content**, and **palpable content** categories. It may be used where sectioning content is expected. Its content model is flow content. A `<section>` is not a generic container; it is appropriate only if the element’s contents would be listed explicitly in the document’s outline. When it has an accessible name, it creates a `region` landmark.

**Beginner-Friendly Explanation**
A `<section>` is a themed chunk of a page — like a chapter in a book or a tab panel. It usually has a heading. If you just need a box for styling, use a `<div>` instead. If the content could stand alone, use `<article>`.

---

### Purposes

- **To group thematically related content** under a common heading.
- **To divide articles into chapters or subsections**: for example, an article about apples might have sections for “Red Delicious” and “Granny Smith”.
- **To create a labelled landmark region** when an accessible name is provided.
- **To provide styling and scripting hooks** for distinct content areas without using generic `<div>` elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<section>
  <h2>Section Heading</h2>
  <p>Section content…</p>
</section>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<section>` | The sectioning container. |
| Heading | Ideally an `<h2>`–`<h6>` identifying the section; MDN recommends each section have a heading whenever possible. |
| Flow content | The section’s body content. |

#### Syntax Rules

1. **A `<section>` should typically have a heading** to identify its content.
2. **Use `<section>` only when a more specific element is not appropriate**: if the content is self-contained, use `<article>`; if it is tangential, use `<aside>`.
3. **Do not use `<section>` as a styling wrapper**; use `<div>` for purely presentational containers.
4. **When `<section>` has an accessible name** (via `aria-labelledby`, `aria-label`, or `title`), it is exposed as a `region` landmark.

#### Constraints and Limitations

- Using `<section>` without a heading can be ambiguous; screen-reader users may not understand its purpose.
- Overusing `<section>` for every small grouping creates a flat, meaningless outline.
- `<section>` is not a substitute for `<article>` when content is independently distributable.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: An Article Divided into Thematic Sections

**Setup guide**: Save as `section-article.html` and open it in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Article with Sections</title>
</head>
<body>
  <article>
    <h1>Apples</h1>
    <p>The apple is the pomaceous fruit of the apple tree.</p>

    <!-- Thematic section 1 -->
    <section>
      <h2>Red Delicious</h2>
      <p>These bright red apples are the most common found in many supermarkets.</p>
    </section>

    <!-- Thematic section 2 -->
    <section>
      <h2>Granny Smith</h2>
      <p>These juicy, green apples make a great filling for apple pies.</p>
    </section>
  </article>
</body>
</html>
```

**Expected output**: An article titled “Apples” followed by two subsections, “Red Delicious” and “Granny Smith”, each with its own paragraph.

**Why this output occurs**: The `<section>` elements divide the article into thematic chunks, each identified by an `<h2>` heading. This matches the WHATWG example of an article about apples with two short sections. The heading hierarchy remains logical: `h1` → `h2` → `h2`.

---

#### Example 2: `<section>` as a Labelled Landmark

**Setup guide**: Save as `section-landmark.html` and inspect the accessibility tree.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Section Landmark</title>
</head>
<body>
  <main>
    <h1>Dashboard</h1>

    <!-- A section with an accessible name becomes a region landmark -->
    <section aria-labelledby="sales-heading">
      <h2 id="sales-heading">Sales Overview</h2>
      <p>Total sales this quarter: $1.2M</p>
    </section>

    <section aria-labelledby="traffic-heading">
      <h2 id="traffic-heading">Website Traffic</h2>
      <p>Unique visitors this month: 45,000</p>
    </section>
  </main>
</body>
</html>
```

**Expected output**: Two labelled regions in the accessibility tree, named “Sales Overview” and “Website Traffic”, each announced as a `region` landmark.

**Why this output occurs**: When a `<section>` has an accessible name — in this case via `aria-labelledby` pointing to the heading — it is exposed as a `region` landmark. Screen-reader users can navigate directly to these named regions.

---

### Real-World Cases

1. **Long-form articles**: Chapters or major topics within an article are marked as `<section>` elements, each with a heading.
2. **Tabbed interfaces**: Each tab panel can be a `<section>` with an accessible name, creating navigable regions.
3. **Documentation pages**: API reference pages use `<section>` elements to divide methods, properties, and examples into distinct thematic chunks.

---

### References

- WHATWG — The `section` element — https://html.spec.whatwg.org/multipage/sections.html#the-section-element
- MDN Web Docs — `<section>`: The Generic Section element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/section
- W3C — HTML 5.2: The `section` element — https://www.w3.org/TR/html52/sections.html#the-section-element

---

## 4. `<nav>` — Navigation Section Element

### Definitions

**Core Definition**
The `<nav>` element represents a section of a page that links to other pages or to parts within the page — a section with navigation links.

**Technical Definition**
`<nav>` belongs to the **flow content**, **sectioning content**, and **palpable content** categories. It may be used where sectioning content is expected. Its content model is flow content. The element creates a `navigation` landmark, which assistive technologies can use to provide direct access to navigation regions.

**Beginner-Friendly Explanation**
A `<nav>` is a block of links that helps users move around — a site menu, a table of contents, or a set of pagination links. Not every group of links needs a `<nav>`; it is intended for major navigation blocks.

---

### Purposes

- **To identify major navigation blocks**: site-wide menus, in-page tables of contents, and breadcrumb trails.
- **To provide a `navigation` landmark** for screen-reader users, allowing them to jump directly to navigation.
- **To scope navigation links** so that user agents can skip or prioritise them.
- **To semantically distinguish navigation from other link groups** in footers or prose.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<nav>` | The navigation container. |
| List or prose | Navigation links, typically in a `<ul>` but may be in prose. |
| `<a>` elements | The actual navigation links. |

#### Syntax Rules

1. **Use `<nav>` for major navigation blocks**, not for every group of links.
2. **A `<nav>` may contain a list of links or links in prose**; a list is common but not required.
3. **Multiple `<nav>` elements are allowed** on a page, for example primary site navigation and secondary in-page navigation.
4. **A `<nav>` creates a `navigation` landmark** automatically.

#### Constraints and Limitations

- Footer links to terms of service and privacy pages do not necessarily require a `<nav>`; the `<footer>` alone is often sufficient.
- Overusing `<nav>` for minor link groups dilutes the landmark’s usefulness.
- `<nav>` should not contain the page’s main content; it is for navigation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Primary and Secondary Navigation

**Setup guide**: Save as `nav-primary-secondary.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Navigation Example</title>
</head>
<body>
  <header>
    <h1>The Wiki Center of Exampland</h1>

    <!-- Primary site navigation -->
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/events">Current Events</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <article>
      <header>
        <h2>Demos in Exampland</h2>
      </header>

      <!-- Secondary in-page navigation -->
      <nav>
        <ul>
          <li><a href="#public">Public demonstrations</a></li>
          <li><a href="#destroy">Demolitions</a></li>
        </ul>
      </nav>

      <section id="public">
        <h2>Public demonstrations</h2>
        <p>…</p>
      </section>

      <section id="destroy">
        <h2>Demolitions</h2>
        <p>…</p>
      </section>
    </article>
  </main>
</body>
</html>
```

**Expected output**: A header with primary site navigation, followed by an article containing a secondary navigation block that links to in-page sections. Both navigation blocks are exposed as `navigation` landmarks.

**Why this output occurs**: The first `<nav>` provides primary site-wide navigation; the second `<nav>` provides in-page navigation. Both are valid and useful, and multiple `<nav>` elements are explicitly permitted.

---

#### Example 2: Navigation Links in Prose

**Setup guide**: Save as `nav-prose.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Prose Navigation</title>
</head>
<body>
  <nav>
    <h1>Navigation</h1>
    <p>
      You are on my home page. To the north lies
      <a href="/blog">my blog</a>, from whence the sounds of battle can be heard.
      To the east you can see a large mountain, upon which many
      <a href="/school">school papers</a> are littered.
      To the south lies a dark and dank <a href="/about">contacts page</a>.
    </p>
  </nav>
</body>
</html>
```

**Expected output**: A navigation region containing prose with embedded links, rather than a traditional list.

**Why this output occurs**: The WHATWG specification explicitly notes that a `<nav>` element does not have to contain a list; it can contain other kinds of content as well, including prose. The navigation landmark is still created, and the links remain navigable.

---

### Real-World Cases

1. **Site-wide menu bars**: The primary `<nav>` in the header provides links to the site’s major sections.
2. **Table of contents**: Long articles use a `<nav>` containing in-page links to each section.
3. **Breadcrumb navigation**: A `<nav>` with `aria-label="Breadcrumb"` provides a navigable trail of links.

---

### References

- WHATWG — The `nav` element — https://html.spec.whatwg.org/multipage/sections.html#the-nav-element
- MDN Web Docs — `<nav>`: The Navigation Section element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/nav
- W3C — HTML 5.2: The `nav` element — https://www.w3.org/TR/html52/sections.html#the-nav-element

---

## 5. `<aside>` — Complementary Content Element

### Definitions

**Core Definition**
The `<aside>` element represents a section of a page that consists of content that is tangentially related to the content around it, and which could be considered separate from that content.

**Technical Definition**
`<aside>` belongs to the **flow content**, **sectioning content**, and **palpable content** categories. It may be used where sectioning content is expected. Its content model is flow content. The element creates a `complementary` landmark, which identifies supporting content that is meaningful on its own but not the main focus.

**Beginner-Friendly Explanation**
An `<aside>` is a sidebar or a call-out box — content that adds useful context but is not the main point. Examples include pull quotes, related links, author bios, advertising, and background information.

---

### Purposes

- **To mark up tangentially related content** such as sidebars, pull quotes, and related links.
- **To create a `complementary` landmark** for screen-reader navigation.
- **To separate supporting content from the main flow** while keeping it associated with the section it belongs to.
- **To provide a semantic container for advertising or supplementary widgets**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<aside>
  <h2>Related Information</h2>
  <p>Supporting content…</p>
</aside>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<aside>` | The complementary content container. |
| Heading | Optional but recommended for identifying the aside’s topic. |
| Flow content | Supporting content: paragraphs, links, pull quotes, widgets. |

#### Syntax Rules

1. **Use `<aside>` for content that is tangentially related** to the surrounding content, not for parenthetical remarks that are part of the main flow.
2. **The `<aside>` creates a `complementary` landmark** automatically.
3. **An `<aside>` inside an `<article>` relates to that article**; an `<aside>` outside articles relates to the page as a whole.
4. **Multiple `<aside>` elements are allowed**, for example a blogroll aside and a Twitter-feed aside.

#### Constraints and Limitations

- Do not use `<aside>` for parentheticals or content that is integral to the main flow.
- Avoid placing essential information exclusively in an `<aside>`, because it may be skipped by users navigating via landmarks.
- An `<aside>` inside an `<article>` should not contain page-level content such as the site’s global blogroll.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Pull Quote as an Aside

**Setup guide**: Save as `aside-pullquote.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Pull Quote Aside</title>
</head>
<body>
  <article>
    <h1>Interview with a Developer</h1>
    <p>He later joined a large company, continuing on the same work.</p>

    <p>
      <q>I love my job. People ask me what I do for fun when I’m not at work.
      But I’m paid to do my hobby, so I never know what to answer.</q>
    </p>

    <!-- Pull quote: tangential, visually emphasised -->
    <aside>
      <q>People ask me what I do for fun when I’m not at work.
      But I’m paid to do my hobby, so I never know what to answer.</q>
    </aside>

    <p>Of course his work — or should that be hobby? — isn’t his only passion.</p>
  </article>
</body>
</html>
```

**Expected output**: An article with a pull quote displayed in an `<aside>`, visually separated from the main text flow.

**Why this output occurs**: The WHATWG specification explicitly permits `<aside>` for “typographical effects like pull quotes”. The aside is tangentially related to the article and could be removed without breaking the main narrative.

---

#### Example 2: Page-Level Sidebar with Blogroll and Twitter Feed

**Setup guide**: Save as `aside-sidebar.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sidebar Asides</title>
</head>
<body>
  <header>
    <h1>My Wonderful Blog</h1>
    <p>My tagline</p>
  </header>

  <!-- Aside 1: Blogroll -->
  <aside>
    <nav>
      <h2>My blogroll</h2>
      <ul>
        <li><a href="https://blog.example.com/">Example Blog</a></li>
      </ul>
    </nav>
  </aside>

  <!-- Aside 2: Twitter feed -->
  <aside>
    <h1>Twitter Feed</h1>
    <blockquote cite="https://twitter.example.net/t31351234">
      I’m on vacation, writing my blog.
    </blockquote>
    <blockquote cite="https://twitter.example.net/t31219752">
      I’m going to go on vacation soon.
    </blockquote>
  </aside>

  <main>
    <article>
      <h2>My last post</h2>
      <p>This is my last post.</p>
    </article>
  </main>
</body>
</html>
```

**Expected output**: Two sidebars — one with a blogroll and one with a Twitter feed — alongside the main blog content.

**Why this output occurs**: The WHATWG example demonstrates `<aside>` for “blogrolls and other side content on a blog”. Both asides are tangentially related to the page but separate from the main article content. Each creates a `complementary` landmark.

---

### Real-World Cases

1. **News article sidebars**: Related articles, author bios, and advertisements are marked as `<aside>`.
2. **Documentation “Note” boxes**: Supplementary notes or warnings are often `<aside>` elements.
3. **Blog sidebars**: Blogrolls, archives, and social-media feeds are classic `<aside>` use cases.

---

### References

- WHATWG — The `aside` element — https://html.spec.whatwg.org/multipage/sections.html#the-aside-element
- MDN Web Docs — `<aside>`: The Aside element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/aside
- W3C — HTML 5.2: The `aside` element — https://www.w3.org/TR/html52/sections.html#the-aside-element

---

## 6. `<h1>` to `<h6>` — Heading Elements

### Definitions

**Core Definition**
The `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, and `<h6>` elements represent headings for their sections. The number in the element name indicates the heading level, with `<h1>` being the highest (most important) and `<h6>` the lowest.

**Technical Definition**
The heading elements belong to the **flow content**, **heading content**, and **palpable content** categories. They may be used as a child of an `<hgroup>` element or where heading content is expected. Their content model is **phrasing content**. The heading level corresponds to the levels of nested sections: `<h1>` for a top-level section, `<h2>` for a subsection, `<h3>` for a sub-subsection, and so on.

**Beginner-Friendly Explanation**
Headings are the titles and subtitles of a page. `<h1>` is the main title, `<h2>` is a major section title, `<h3>` is a subsection title, and so on. They create a table of contents that helps readers and screen readers navigate.

---

### Purposes

- **To define the title of a section**: each heading names the content that follows.
- **To create a hierarchical outline**: nested heading levels reflect nested content structure.
- **To support screen-reader navigation**: users can jump between headings to skim the page.
- **To satisfy WCAG requirements**: Technique H69 requires headings at the beginning of each section of content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<h1>Main Page Title</h1>
<h2>Major Section</h2>
<h3>Subsection</h3>
<h4>Sub-subsection</h4>
<h5>Deeper Level</h5>
<h6>Deepest Level</h6>
```

**Component breakdown**

| Element | Rank | Typical Use |
|---|---|---|
| `<h1>` | Highest | Page or article main title. |
| `<h2>` | Second | Major section heading. |
| `<h3>` | Third | Subsection heading. |
| `<h4>`–`<h6>` | Lower | Deeper subsection headings. |

#### Syntax Rules

1. **Do not skip heading levels**: use `<h1>`, then `<h2>`, then `<h3>`, and so on. Skipping levels (e.g. `<h1>` to `<h3>`) is discouraged because it suggests the document structure was not properly thought through.
2. **Use headings hierarchically**: `<h2>` is a subsection of `<h1>`, `<h3>` is a subsection of `<h2>`, and so on.
3. **Heading content model is phrasing content**; headings cannot contain block-level elements.
4. **The first heading element in a sectioning element is the heading for that section**.

#### Constraints and Limitations

- Using headings purely for visual effects (e.g. choosing `<h3>` because it is smaller) damages document structure and accessibility.
- Skipping heading levels can confuse screen-reader users and create a misleading outline.
- `<h1>` must not appear as a descendant of the `<address>` element.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Correctly Nested Heading Hierarchy

**Setup guide**: Save as `headings-correct.html` and inspect the heading outline in the browser’s accessibility tools.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Heading Hierarchy</title>
</head>
<body>
  <h1>Web Development Guide</h1>

  <h2>HTML Fundamentals</h2>
  <p>HTML provides the structure of web pages.</p>

  <h3>Elements</h3>
  <p>Elements are the building blocks of HTML.</p>

  <h3>Attributes</h3>
  <p>Attributes modify element behaviour.</p>

  <h2>CSS Fundamentals</h2>
  <p>CSS controls the presentation of web pages.</p>
</body>
</html>
```

**Expected output**: A page with “Web Development Guide” as the largest heading, “HTML Fundamentals” and “CSS Fundamentals” as second-level headings, and “Elements” and “Attributes” as third-level headings nested under “HTML Fundamentals”. The outline is logical and non-skipping.

**Why this output occurs**: The heading levels follow the recommended hierarchy: `h1` → `h2` → `h3` → `h3` → `h2`. Screen readers can present this as a structured outline, and the visual rendering follows the browser’s default heading styles. Technique H69 recommends this structure for conveying content hierarchy.

---

#### Example 2: Incorrect Heading Skipping (Anti-Pattern)

**Setup guide**: Save as `headings-skip.html` and compare with the previous example.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Heading Skipping — Don’t Do This</title>
</head>
<body>
  <h1>Main Title</h1>
  <!-- WRONG: jumps from h1 directly to h3 -->
  <h3>Skipped h2</h3>
  <p>This heading level skips h2, creating a confusing outline.</p>
</body>
</html>
```

**Expected output**: The browser renders “Skipped h2” smaller than “Main Title”, but a screen reader’s heading list will show a gap where an `h2` should be, potentially confusing users.

**Why this output occurs**: Skipping from `h1` to `h3` violates the recommended hierarchical nesting. Technique H69 explicitly warns that skipping levels “may create the impression that the structure of the document has not been properly thought through”.

---

### Real-World Cases

1. **Documentation sites**: `h1` for the page title, `h2` for major topics, and `h3` for sub-topics.
2. **Blog posts**: The post title is typically the page’s `h1`; section headings use `h2` and `h3`.
3. **E-commerce category pages**: `h1` names the category, `h2` names product groups, and `h3` names individual products or sub-groups.

---

### References

- WHATWG — The `h1`–`h6` elements — https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements
- MDN Web Docs — `<h1>`–`<h6>`: The HTML Section Heading elements — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements
- W3C — Technique H69: Providing heading elements at the beginning of each section of content — https://www.w3.org/WAI/WCAG21/Techniques/html/H69

---

## 7. `<hgroup>` — Heading Group Element

### Definitions

**Core Definition**
The `<hgroup>` element represents a heading and related content. It groups an `<h1>`–`<h6>` element with one or more `<p>` elements containing content representing a subheading, alternative title, or tagline.

**Technical Definition**
`<hgroup>` belongs to the **flow content**, **heading content**, and **palpable content** categories. Its content model is: zero or more `<p>` elements, followed by one `<h1>`–`<h6>` element, followed by zero or more `<p>` elements, optionally intermixed with script-supporting elements. The element is part of the WHATWG specification but was removed from the W3C HTML5 specification; it is still at least partially supported by most browsers.

**Beginner-Friendly Explanation**
`<hgroup>` is a way to group a main heading with a subtitle or tagline. For example, you might group “Dr. Strangelove” (the title) with “Or: How I Learned to Stop Worrying and Love the Bomb” (the subtitle) so that they are treated as a single heading unit.

---

### Purposes

- **To group a heading with its subheading or tagline**: providing a single semantic heading unit.
- **To prevent subheadings from creating separate outline entries**: without `<hgroup>`, a `<p>` after an `<h2>` might be interpreted as separate content.
- **To provide a structured container for alternative titles** or multi-part headings.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<hgroup>
  <h1>Main Title</h1>
  <p>Subtitle or tagline</p>
</hgroup>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<hgroup>` | The grouping container. |
| `<p>` before | Zero or more paragraphs before the heading (rare). |
| `<h1>`–`<h6>` | Exactly one heading element. |
| `<p>` after | Zero or more paragraphs after the heading (subtitle, tagline, alternative title). |

#### Syntax Rules

1. **Content model requires exactly one `<h1>`–`<h6>` element** and zero or more `<p>` elements, with the `<p>` elements before or after the heading as specified.
2. **The `<hgroup>` rank is the rank of its highest-ranked heading descendant**.
3. **The element is part of the WHATWG Living Standard** but was removed from W3C HTML5; browser support is partial.

#### Constraints and Limitations

- `<hgroup>` is less widely supported than other sectioning elements; do not rely on it for critical document structure.
- Overusing `<hgroup>` for simple headings with no subtitle adds unnecessary complexity.
- The WHATWG specification notes that the document outline algorithm is not reliably implemented, so `<hgroup>`’s effect on outlines should not be depended upon.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Heading with a Subtitle

**Setup guide**: Save as `hgroup-subtitle.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hgroup Example</title>
</head>
<body>
  <hgroup>
    <h1>Dr. Strangelove</h1>
    <p>Or: How I Learned to Stop Worrying and Love the Bomb</p>
  </hgroup>
</body>
</html>
```

**Expected output**: A main heading “Dr. Strangelove” followed by a subtitle in a smaller paragraph, grouped together as a single heading unit.

**Why this output occurs**: The `<hgroup>` groups the `<h1>` with the `<p>` subtitle. The specification provides this exact example. The grouping indicates that the paragraph is part of the heading, not separate body content.

---

#### Example 2: An Article Header with an Hgroup

**Setup guide**: Save as `hgroup-article.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Article with Hgroup</title>
</head>
<body>
  <article>
    <header>
      <hgroup>
        <h2>The Reality Dysfunction</h2>
        <p>Space is not the only void</p>
      </hgroup>
      <p>Published <time datetime="2026-11-06">6 November 2026</time></p>
    </header>
    <p>The article body begins here.</p>
  </article>
</body>
</html>
```

**Expected output**: An article whose header contains a grouped heading and tagline (“The Reality Dysfunction” with “Space is not the only void”), followed by the publication date.

**Why this output occurs**: The `<hgroup>` groups the `<h2>` heading with its tagline. The publication date is outside the `<hgroup>` because it is not part of the heading itself. This structure clarifies which text is the heading and which is supporting metadata.

---

### Real-World Cases

1. **Book and movie titles**: Grouping a title with its subtitle or tagline is the primary use case for `<hgroup>`.
2. **Blog post headers**: A post title with a descriptive tagline can be grouped in `<hgroup>`.
3. **Landing pages**: Hero-section headings with subheadings benefit from grouping.

---

### References

- WHATWG — The `hgroup` element — https://html.spec.whatwg.org/multipage/sections.html#the-hgroup-element
- MDN Web Docs — `<hgroup>`: The Heading Group element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/hgroup
- W3C — HTML 5.2: The `hgroup` element (removed) — https://www.w3.org/TR/html52/sections.html#the-hgroup-element

---

## 8. `<header>` — Introductory Content Element

### Definitions

**Core Definition**
The `<header>` element represents a group of introductory or navigational aids for its nearest ancestor sectioning content or sectioning root element. It typically contains the section’s heading, byline, and other introductory material.

**Technical Definition**
`<header>` belongs to the **flow content** and **palpable content** categories. It may be used where flow content is expected. Its content model is flow content, **but with no `<header>` or `<footer>` element descendants**. When `<header>` is a direct child of `<body>`, it creates a `banner` landmark; when it is a descendant of `<article>`, `<aside>`, `<main>`, `<nav>`, or `<section>`, it does not create a `banner` landmark.

**Beginner-Friendly Explanation**
A `<header>` is the introductory part of a section or page — usually the title, logo, byline, and maybe navigation. It is not necessarily at the top of the page; it is at the top of whatever section it belongs to.

---

### Purposes

- **To contain introductory content for a section or page**: headings, bylines, publication dates, and navigation.
- **To create a `banner` landmark** when used as a direct child of `<body>`.
- **To scope introductory content** within an `<article>` or `<section>`, keeping the heading and metadata grouped.
- **To separate a section’s introduction from its body content**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<header>
  <h1>Page Title</h1>
  <p>Tagline or byline</p>
  <nav>
    <!-- navigation links -->
  </nav>
</header>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<header>` | The introductory container. |
| Heading | Typically an `<h1>`–`<h6>` identifying the section. |
| Metadata | Byline, publication date, tagline, or logo. |
| Navigation | Optional `<nav>` for section-level navigation. |

#### Syntax Rules

1. **No `<header>` or `<footer>` descendants** are permitted inside a `<header>`.
2. **A `<header>` that is a direct child of `<body>` creates a `banner` landmark**; inside sectioning content it does not.
3. **Multiple `<header>` elements are allowed** on a page, one per section if needed.
4. **The header does not take part in the document outline algorithm**; headings inside it still create sections.

#### Constraints and Limitations

- A `<header>` inside an `<article>` is not a `banner` landmark; only a `<header>` directly inside `<body>` is.
- Do not put a `<header>` inside another `<header>`.
- Avoid using `<header>` purely for styling; use it when the content genuinely introduces a section.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Page Header with Banner Landmark

**Setup guide**: Save as `header-page.html` and inspect the accessibility tree.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Page Header</title>
</head>
<body>
  <!-- Direct child of body: creates a banner landmark -->
  <header>
    <h1>Little Green Guys With Guns</h1>
    <nav>
      <ul>
        <li><a href="/games">Games</a></li>
        <li><a href="/forum">Forum</a></li>
        <li><a href="/download">Download</a></li>
      </ul>
    </nav>
    <h2>Important News</h2>
    <p>To play today’s games you will need to update your client.</p>
  </header>

  <main>
    <p>Main content begins here.</p>
  </main>
</body>
</html>
```

**Expected output**: A page header with the site title, navigation links, and a news announcement. The header is exposed as a `banner` landmark.

**Why this output occurs**: The `<header>` is a direct child of `<body>`, so it creates a `banner` landmark. The heading and navigation inside it are introductory content. The `<h2>` inside the header starts a subsection, but the header itself does not create an outline entry.

---

#### Example 2: Article Header (Not a Banner Landmark)

**Setup guide**: Save as `header-article.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Article Header</title>
</head>
<body>
  <main>
    <article>
      <!-- Header inside an article: not a banner landmark -->
      <header>
        <h2>My Day at the Beach</h2>
        <p>Posted <time datetime="2026-11-06">6 November 2026</time></p>
      </header>
      <p>Today I went to the beach and had a lot of fun.</p>
    </article>
  </main>
</body>
</html>
```

**Expected output**: An article with a heading and publication date in its header. The header is not exposed as a `banner` landmark because it is inside an `<article>`.

**Why this output occurs**: The `<header>` is a descendant of `<article>`, so it does not create a `banner` landmark. It still groups the article’s introductory content, but its landmark role is suppressed.

---

### Real-World Cases

1. **Site-wide page header**: The top strip with the logo, site name, and primary navigation is a `<header>` direct child of `<body>`.
2. **Article headers**: Blog posts and news stories use `<header>` to group the title, byline, and publication date.
3. **Section headers**: Long documents use `<header>` inside `<section>` to introduce each major section.

---

### References

- WHATWG — The `header` element — https://html.spec.whatwg.org/multipage/sections.html#the-header-element
- MDN Web Docs — `<header>`: The Header element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/header
- W3C — HTML 5.2: The `header` element — https://www.w3.org/TR/html52/sections.html#the-header-element

---

## 9. `<footer>` — Concluding Content Element

### Definitions

**Core Definition**
The `<footer>` element represents a footer for its nearest ancestor sectioning content or sectioning root element. A footer typically contains information about its section such as who wrote it, links to related documents, copyright data, and the like.

**Technical Definition**
`<footer>` belongs to the **flow content** and **palpable content** categories. It may be used where flow content is expected. Its content model is flow content, **but with no `<header>` or `<footer>` element descendants**. When `<footer>` is a direct child of `<body>`, it creates a `contentinfo` landmark; when it is a descendant of `<article>`, `<aside>`, `<main>`, `<nav>`, or `<section>`, it does not create a `contentinfo` landmark.

**Beginner-Friendly Explanation**
A `<footer>` is the concluding part of a section or page — usually copyright information, author credits, or links to related documents. It does not have to be at the very bottom of the page; it is at the bottom of whatever section it belongs to.

---

### Purposes

- **To contain concluding information for a section**: copyright, author credits, related links.
- **To create a `contentinfo` landmark** when used as a direct child of `<body>`.
- **To scope footer content** within an `<article>` or `<section>`, keeping metadata with the content it describes.
- **To provide a semantic container for site-wide legal and contact information**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<footer>
  <p>© 2026 Example Company</p>
  <nav>
    <a href="/privacy">Privacy</a> |
    <a href="/terms">Terms</a>
  </nav>
</footer>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<footer>` | The concluding container. |
| Flow content | Copyright, credits, related links, contact information. |

#### Syntax Rules

1. **No `<header>` or `<footer>` descendants** are permitted inside a `<footer>`.
2. **A `<footer>` that is a direct child of `<body>` creates a `contentinfo` landmark**; inside sectioning content it does not.
3. **Footers do not necessarily have to appear at the end of a section**, though they usually do.
4. **A footer inside an `<article>` provides metadata for that article**, such as author and publication date.

#### Constraints and Limitations

- A `<footer>` inside an `<article>` is not a `contentinfo` landmark; only a `<footer>` directly inside `<body>` is.
- Do not put a `<footer>` inside another `<footer>`.
- Avoid duplicating the same footer content across multiple sections unnecessarily.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Page Footer with Contentinfo Landmark

**Setup guide**: Save as `footer-page.html` and inspect the accessibility tree.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Page Footer</title>
</head>
<body>
  <main>
    <p>Main content goes here.</p>
  </main>

  <!-- Direct child of body: creates a contentinfo landmark -->
  <footer>
    <p>© 2026 Example Company</p>
    <nav>
      <a href="/privacy">Privacy Policy</a> |
      <a href="/terms">Terms of Service</a> |
      <a href="/contact">Contact Us</a>
    </nav>
  </footer>
</body>
</html>
```

**Expected output**: A page footer containing a copyright notice and legal links, exposed as a `contentinfo` landmark.

**Why this output occurs**: The `<footer>` is a direct child of `<body>`, so it creates a `contentinfo` landmark. The links inside it provide site-wide legal navigation.

---

#### Example 2: Article Footer with Comment Metadata

**Setup guide**: Save as `footer-article.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Article Footer</title>
</head>
<body>
  <article>
    <h1>Blog Post Title</h1>
    <p>Post content…</p>

    <section>
      <h2>Comments</h2>

      <article>
        <!-- Footer inside an article: provides comment metadata -->
        <footer>
          <p>Posted by: George Washington</p>
          <p><time datetime="2026-11-06T10:00">6 November 2026, 10:00</time></p>
        </footer>
        <p>Yeah! Especially when talking about your lobbyist friends!</p>
      </article>
    </section>
  </article>
</body>
</html>
```

**Expected output**: A blog post with a comments section. Each comment article has a footer containing the author name and timestamp.

**Why this output occurs**: The `<footer>` inside the comment `<article>` provides metadata for that specific comment. The WHATWG example demonstrates this pattern: the footer gives information about who wrote the comment and when. It is not a `contentinfo` landmark because it is inside an `<article>`.

---

### Real-World Cases

1. **Site-wide page footer**: Copyright, privacy policy, and contact links at the bottom of every page.
2. **Article footers**: Author bios, publication dates, and related-article links at the end of a blog post.
3. **Comment footers**: Author names and timestamps attached to individual comments.

---

### References

- WHATWG — The `footer` element — https://html.spec.whatwg.org/multipage/sections.html#the-footer-element
- MDN Web Docs — `<footer>`: The Footer element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/footer
- W3C — HTML 5.2: The `footer` element — https://www.w3.org/TR/html52/sections.html#the-footer-element

---

## 10. `<main>` — Main Content Element

### Definitions

**Core Definition**
The `<main>` element represents the dominant contents of the document. A document must not have more than one `<main>` element that does not have the `hidden` attribute specified.

**Technical Definition**
`<main>` is defined in the “Grouping content” section of the WHATWG standard. It belongs to the **flow content** and **palpable content** categories. Its content model is flow content. The element creates a `main` landmark, which assistive technologies use to provide direct access to the page’s primary content. A “hierarchically correct” `<main>` element has ancestors limited to `<html>`, `<body>`, `<div>`, `<form>` without an accessible name, and autonomous custom elements.

**Beginner-Friendly Explanation**
The `<main>` element wraps the most important content on the page — the stuff that is unique to this page and not repeated across the site. There should be only one visible `<main>` per page. Headers, footers, navigation, and sidebars should be outside it.

---

### Purposes

- **To identify the dominant content** of the document, distinct from repeated site-wide chrome.
- **To create a `main` landmark** for screen-reader navigation, allowing users to skip directly to the primary content.
- **To improve “skip to content” functionality**: a `<main>` element provides a stable target for skip links.
- **To satisfy WCAG requirements** for bypassing blocks and identifying the main content region.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<body>
  <header>…</header>
  <nav>…</nav>

  <main>
    <!-- dominant content -->
  </main>

  <footer>…</footer>
</body>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<main>` | The dominant content container. |
| Flow content | The page’s unique primary content. |

#### Syntax Rules

1. **Only one visible `<main>` per document**; additional `<main>` elements must have the `hidden` attribute.
2. **`<main>` must not be a descendant of `<article>`, `<aside>`, `<footer>`, `<header>`, or `<nav>`**.
3. **The `<main>` element creates a `main` landmark** automatically.
4. **`<main>` should contain the page’s unique content**, excluding site-wide headers, footers, and navigation.

#### Constraints and Limitations

- Multiple visible `<main>` elements are non-conforming.
- `<main>` should not be used for repeated content such as site-wide navigation or copyright notices.
- Older browsers (notably Internet Explorer) do not support `<main>` by default, though a CSS `display: block` rule can mitigate rendering issues.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Typical Page Layout with `<main>`

**Setup guide**: Save as `main-layout.html` and inspect the landmarks.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Main Content Layout</title>
</head>
<body>
  <header>
    <h1>Site Title</h1>
    <nav>
      <a href="/">Home</a> |
      <a href="/about">About</a> |
      <a href="/contact">Contact</a>
    </nav>
  </header>

  <!-- The main landmark contains the page’s unique content -->
  <main>
    <h2>Welcome</h2>
    <p>This is the primary content of the page.</p>
  </main>

  <aside>
    <p>Related links or sidebar content.</p>
  </aside>

  <footer>
    <p>© 2026 Example Company</p>
  </footer>
</body>
</html>
```

**Expected output**: A page with a banner header, a main content region, a complementary sidebar, and a contentinfo footer. Screen-reader users can jump directly to the `main` landmark to skip the header and navigation.

**Why this output occurs**: The `<main>` element creates a `main` landmark. It contains the page’s unique content, while the header, aside, and footer are outside it. This structure satisfies the recommendation that the `<main>` element include all content directly related to the page’s main idea, with only one per page.

---

#### Example 2: `<main>` with a Skip Link

**Setup guide**: Save as `main-skip-link.html` and use keyboard navigation to activate the skip link.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Skip Link to Main</title>
</head>
<body>
  <!-- Skip link targets the main element -->
  <a href="#main-content" class="skip-link">Skip to main content</a>

  <header>
    <nav>
      <a href="/">Home</a> |
      <a href="/about">About</a>
    </nav>
  </header>

  <main id="main-content">
    <h1>Primary Content</h1>
    <p>This is the content the skip link jumps to.</p>
  </main>
</body>
</html>
```

**Expected output**: A “Skip to main content” link appears when the page loads (or on keyboard focus). Activating it moves focus to the `<main>` element, bypassing the navigation.

**Why this output occurs**: The skip link’s `href="#main-content"` targets the `<main>` element’s `id`. Because `<main>` creates a `main` landmark, screen-reader users can also navigate directly to it using landmark navigation. Technique H101 explains how semantic HTML elements like `<main>` provide programmatic access to page regions.

---

### Real-World Cases

1. **Every content-heavy page**: News articles, blog posts, and documentation pages use `<main>` to isolate the primary content.
2. **Skip-to-content links**: Accessibility best practice is to provide a skip link that targets `<main>`.
3. **Single-page applications**: The `<main>` element wraps the active route’s content, while the header and footer remain outside.

---

### References

- WHATWG — The `main` element — https://html.spec.whatwg.org/multipage/grouping-content.html#the-main-element
- MDN Web Docs — `<main>`: The Main element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/main
- W3C — Technique H101: Using semantic HTML elements to identify regions of a page — https://www.w3.org/WAI/WCAG21/Techniques/html/H101

---

## 11. `<address>` — Contact Information Element

### Definitions

**Core Definition**
The `<address>` element represents the contact information for its nearest `<article>` or `<body>` element ancestor. If that is the `<body>` element, the contact information applies to the document as a whole.

**Technical Definition**
`<address>` belongs to the **flow content** and **palpable content** categories. It may be used where flow content is expected. Its content model is flow content, **but with no `<address>` element descendants, no heading content descendants, and no sectioning content descendants**. The `<address>` element must not be used to represent arbitrary addresses (e.g. postal addresses) unless those addresses are contact information for the section.

**Beginner-Friendly Explanation**
The `<address>` element is for contact information — an email address, a phone number, a social-media handle, or a physical address that is specifically the contact point for the page or article. It is not for arbitrary postal addresses or for marking up every address on the page.

---

### Purposes

- **To provide contact information for a document or article**: email, phone, social media, or physical address.
- **To scope contact details to the correct section**: an `<address>` inside an `<article>` applies to that article; an `<address>` in `<body>` applies to the whole document.
- **To support microdata and machine-readable contact information**: `<address>` can be combined with schema.org vocabulary.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<address>
  Contact: <a href="mailto:author@example.com">author@example.com</a><br>
  Phone: <a href="tel:+15551234567">+1 (555) 123-4567</a>
</address>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<address>` | The contact-information container. |
| Flow content | Contact details: email, phone, social media, or physical address. |
| `<a>` elements | Links for email (`mailto:`) and phone (`tel:`). |

#### Syntax Rules

1. **No `<address>` descendants, no heading content descendants, and no sectioning content descendants** are permitted.
2. **The `<address>` element must not contain information other than contact information**.
3. **An `<address>` inside an `<article>` applies to that article**; an `<address>` in `<body>` applies to the document as a whole.
4. **Author information associated with an `<article>` does not apply to nested articles**.

#### Constraints and Limitations

- Do not use `<address>` for arbitrary postal addresses unless they are the contact information for the section.
- `<address>` should not contain headings, sections, or other `<address>` elements.
- The element is for contact information, not for general address markup.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Document-Level Contact Information

**Setup guide**: Save as `address-document.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document Contact Info</title>
</head>
<body>
  <main>
    <h1>Contact Us</h1>
    <p>For questions about this site, reach the author:</p>

    <!-- Address in body: contact info for the whole document -->
    <address>
      Email: <a href="mailto:author@example.com">author@example.com</a><br>
      Phone: <a href="tel:+15551234567">+1 (555) 123-4567</a><br>
      Twitter: <a href="https://twitter.com/example">@example</a>
    </address>
  </main>
</body>
</html>
```

**Expected output**: A “Contact Us” section with the author’s email, phone, and Twitter handle formatted as contact information.

**Why this output occurs**: The `<address>` is inside `<body>` (via `<main>`), so it represents contact information for the document as a whole. It contains only contact details, no headings or sections, satisfying the content model.

---

#### Example 2: Article-Scoped Contact Information

**Setup guide**: Save as `address-article.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Article Contact Info</title>
</head>
<body>
  <article>
    <h1>How to Build Accessible Forms</h1>
    <p>Article content…</p>

    <!-- Address inside article: contact info for this article’s author -->
    <footer>
      <address>
        Written by <a href="mailto:jane@example.com">Jane Doe</a>.
        Visit <a href="https://jane.example.com">jane.example.com</a>.
      </address>
    </footer>
  </article>
</body>
</html>
```

**Expected output**: An article with a footer containing the author’s contact information, scoped to that article.

**Why this output occurs**: The `<address>` is inside an `<article>`, so it applies to that article’s author. The `<footer>` groups the contact information as concluding metadata. The `<address>` contains only contact details and links, with no headings or sectioning content.

---

### Real-World Cases

1. **Author contact information**: Blog posts and articles use `<address>` to provide the author’s email or social-media handle.
2. **Site-wide contact pages**: A “Contact Us” page uses `<address>` to provide the site owner’s contact details.
3. **Microdata-enhanced contact info**: `<address>` combined with `itemprop` from schema.org can provide machine-readable contact data.

---

### References

- WHATWG — The `address` element — https://html.spec.whatwg.org/multipage/sections.html#the-address-element
- MDN Web Docs — `<address>`: The Contact Address element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/address
- W3C — HTML 5.2: The `address` element — https://www.w3.org/TR/html52/sections.html#the-address-element

---

## 12. Sectioning Element Best Practices

### Definitions

**Core Definition**
Sectioning element best practices are authoring guidelines for choosing, nesting, and labelling the structural elements of a document so that the resulting markup is semantic, accessible, maintainable, and conforming to the HTML standard.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, the WAI-ARIA Authoring Practices landmark guidance, and WCAG techniques. They encompass correct element selection, heading hierarchy, landmark labelling, content-model compliance, and the avoidance of generic containers when semantic alternatives exist.

**Beginner-Friendly Explanation**
Best practices are the “good habits” for structuring a page. They do not always change how the page looks, but they make it easier for screen readers, search engines, and other tools to understand and navigate.

---

### Purposes

- **To maximise accessibility** for screen-reader and keyboard-only users.
- **To ensure a logical document outline** that reflects the content hierarchy.
- **To avoid landmark confusion** by using the correct element for each region.
- **To improve maintainability** by replacing anonymous `<div>` elements with meaningful semantic containers.
- **To support SEO** through clear heading and section structure.

---

### Syntax Rules and Structure

Best practices are applied across elements. The following table summarises the core rules.

| Practice | Rule |
|---|---|
| **Use the right element for the job** | `<nav>` for navigation, `<article>` for self-contained content, `<section>` for thematic groupings, `<aside>` for tangential content, `<main>` for dominant content. |
| **Give every section a heading** | Each `<section>` should typically contain a heading (`<h1>`–`<h6>`). |
| **Do not skip heading levels** | Use `h1` → `h2` → `h3`; skipping levels confuses assistive technology. |
| **Use only one visible `<main>`** | Additional `<main>` elements must be `hidden`. |
| **Label repeated landmarks** | If a landmark role appears more than once, give each instance a unique accessible name. |
| **Do not use `<section>` as a styling wrapper** | Use `<div>` for purely presentational containers. |
| **Place `<header>` and `<footer>` correctly** | A `<header>` or `<footer>` directly inside `<body>` creates `banner` or `contentinfo`; inside sectioning content it does not. |
| **Use `<address>` only for contact information** | Do not use it for arbitrary postal addresses. |
| **Ensure all content is within a landmark** | Best practice is to include all perceivable content in a landmark region so screen-reader users do not lose track of it. |

#### Syntax Rules

1. **Semantic correctness**: Choose the element that describes the content’s role, not its appearance.
2. **Accessibility first**: Provide accessible names for landmarks and headings for sections.
3. **Valid nesting**: Respect content models — no `<header>` or `<footer>` inside `<header>` or `<footer>`, no heading content inside `<address>`, and so on.
4. **Heading discipline**: Maintain a non-skipping heading hierarchy throughout the document.
5. **Landmark completeness**: Wrap all perceivable content in appropriate landmark regions where possible.

#### Constraints and Limitations

- Best practices can conflict with legacy markup; incremental improvement is acceptable.
- Landmark roles are implicit in sectioning elements; adding redundant ARIA roles (e.g. `<main role="main">`) is discouraged.
- Some sectioning elements (`<hgroup>`) have incomplete browser support; do not rely on them for critical structure.
- The document outline algorithm is not reliably implemented; explicit heading levels remain the primary structural signal.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Well-Structured Accessible Page

**Setup guide**: Save as `best-practice-page.html` and inspect the landmarks and heading outline.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best-Practice Sectioning</title>
</head>
<body>
  <!-- Banner landmark: direct child of body -->
  <header>
    <h1>My Accessible Site</h1>
    <nav aria-label="Primary">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/articles">Articles</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  </header>

  <!-- Main landmark: dominant content -->
  <main>
    <article>
      <header>
        <h2>How to Structure a Page</h2>
        <p>By <a href="mailto:author@example.com">Jane Doe</a></p>
      </header>

      <section aria-labelledby="why-heading">
        <h3 id="why-heading">Why Structure Matters</h3>
        <p>Semantic structure helps assistive technologies.</p>
      </section>

      <section aria-labelledby="how-heading">
        <h3 id="how-heading">How to Use Sectioning Elements</h3>
        <p>Choose the element that best describes the content.</p>
      </section>

      <footer>
        <address>
          Contact <a href="mailto:jane@example.com">Jane Doe</a>.
        </address>
      </footer>
    </article>
  </main>

  <!-- Complementary landmark -->
  <aside aria-labelledby="related-heading">
    <h2 id="related-heading">Related Articles</h2>
    <ul>
      <li><a href="/article-1">Article One</a></li>
      <li><a href="/article-2">Article Two</a></li>
    </ul>
  </aside>

  <!-- Contentinfo landmark: direct child of body -->
  <footer>
    <p>© 2026 My Accessible Site</p>
  </footer>
</body>
</html>
```

**Expected output**: A page with a banner header, primary navigation, main article content divided into labelled sections, an article footer with contact information, a complementary sidebar, and a page footer. Screen-reader users can navigate by landmarks (banner, navigation, main, region, complementary, contentinfo) and by headings (`h1` → `h2` → `h3` → `h3`).

**Why this output occurs**: Every sectioning element is used according to its intended role. Landmarks are correctly generated: `<header>` (direct child of `<body>`) → banner, `<nav>` → navigation, `<main>` → main, `<section>` with accessible name → region, `<aside>` → complementary, `<footer>` (direct child of `<body>`) → contentinfo. The heading hierarchy is logical and non-skipping.

---

#### Example 2: Avoiding Common Sectioning Anti-Patterns

**Setup guide**: Save as `sectioning-anti-patterns.html` and compare with the best-practice example.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sectioning Anti-Patterns</title>
</head>
<body>
  <!-- ANTI-PATTERN 1: <section> as a styling wrapper without a heading -->
  <section class="hero-banner">
    <p>Welcome to our site!</p>
  </section>

  <!-- ANTI-PATTERN 2: Multiple visible <main> elements -->
  <main>
    <p>First main content.</p>
  </main>
  <main>
    <p>Second main content — this is invalid.</p>
  </main>

  <!-- ANTI-PATTERN 3: Skipping heading levels -->
  <h1>Page Title</h1>
  <h3>Skipped h2</h3>

  <!-- ANTI-PATTERN 4: <address> used for a non-contact postal address -->
  <address>
    123 Main Street, Anytown, USA
  </address>
</body>
</html>
```

**Expected output**: The page may render, but validators will report errors, and assistive technologies will produce a confused outline: an unnamed `region` landmark for the hero section, multiple `main` landmarks, a heading gap, and an `<address>` that misuses the element.

**Why this output occurs**: These are common anti-patterns. `<section>` without a heading creates an ambiguous landmark. Multiple visible `<main>` elements violate the one-per-document rule. Skipping heading levels creates a confusing outline. Using `<address>` for a non-contact postal address violates the element’s definition.

---

### Real-World Cases

1. **Government and public-sector websites**: Must meet WCAG, making correct landmark structure and heading hierarchy legally required.
2. **Design systems**: Component libraries codify sectioning best practices so that every page built from the system inherits accessible structure.
3. **Content management systems**: Templates enforce best practices by providing pre-structured `<header>`, `<main>`, `<article>`, `<section>`, and `<footer>` regions.

---

### References

- WHATWG — HTML Living Standard: Sections — https://html.spec.whatwg.org/multipage/sections.html
- W3C — ARIA Landmark Regions — https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/
- W3C — Technique H101: Using semantic HTML elements to identify regions of a page — https://www.w3.org/WAI/WCAG21/Techniques/html/H101
- W3C — Technique H69: Providing heading elements at the beginning of each section of content — https://www.w3.org/WAI/WCAG21/Techniques/html/H69
- MDN Web Docs — Structuring documents — https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Document_and_website_structure
- MDN Web Docs — `<section>`: The Generic Section element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/section

---

## Summary Table of Sectioning Elements

| Element | Primary Role | Landmark Role | Key Content Model |
|---|---|---|---|
| `<body>` | Document contents | None | Flow content |
| `<article>` | Self-contained composition | None (sectioning content) | Flow content |
| `<section>` | Generic thematic section | `region` (if named) | Flow content |
| `<nav>` | Navigation links | `navigation` | Flow content |
| `<aside>` | Tangential content | `complementary` | Flow content |
| `<h1>`–`<h6>` | Section headings | None (heading content) | Phrasing content |
| `<hgroup>` | Heading with subheading | None (heading content) | `<p>`* + one `<h1>`–`<h6>` + `<p>`* |
| `<header>` | Introductory content | `banner` (direct child of `<body>`) | Flow content, no `<header>`/`<footer>` descendants |
| `<footer>` | Concluding content | `contentinfo` (direct child of `<body>`) | Flow content, no `<header>`/`<footer>` descendants |
| `<main>` | Dominant content | `main` | Flow content |
| `<address>` | Contact information | None | Flow content, no `<address>`, heading, or sectioning descendants |

---

## Final Notes on Deprecated, Obsolete, and Implementation-Dependent Features

- **Document outline algorithm**: The theoretical algorithm that would have derived document structure from sectioning elements was never implemented consistently in browsers. Explicit heading levels remain the primary structural signal; do not rely on sectioning elements alone to create an outline.
- **`<hgroup>`**: Removed from the W3C HTML5 specification but retained in the WHATWG Living Standard with partial browser support.
- **Multiple `<h1>` elements**: Allowing multiple `<h1>` elements in nested sectioning elements was present in older HTML versions but was never a best practice and is now non-conforming.
- **`<main>` in Internet Explorer**: Older versions of IE do not recognise `<main>` by default; a CSS `display: block` rule is a common workaround, though modern browsers support the element natively.
- **Implicit landmark roles**: Modern browsers expose sectioning elements as landmarks without the need for explicit ARIA roles. Adding `role="main"` to `<main>` or `role="navigation"` to `<nav>` is redundant and discouraged.

---

This cheat sheet has covered the sectioning elements — `<body>`, `<article>`, `<section>`, `<nav>`, `<aside>`, `<h1>`–`<h6>`, `<hgroup>`, `<header>`, `<footer>`, `<main>`, and `<address>` — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate sectioning markup against the WHATWG HTML Living Standard and WCAG accessibility guidelines.