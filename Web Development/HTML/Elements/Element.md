# HTML Element Fundamentals — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
HTML elements are the fundamental building blocks of HyperText Markup Language documents. An element is a syntactic construct that describes the meaning and structure of a piece of web content through a start tag, optional content, and an end tag. Together, elements form a tree-structured document object model that browsers parse, style, and render.

**Technical Definition**
In the WHATWG HTML Living Standard, an element is a node in the DOM tree produced by the HTML parser. Each element is defined by a start tag (possibly with attributes) and an end tag (except for void elements), and is associated with a content model that normatively specifies which child nodes — elements, text, comments, or processing instructions — are permitted inside it. Every element also belongs to zero or more content categories that determine where it may appear and what it may contain.

**Beginner-Friendly Explanation**
Think of an HTML element as a labelled container on a web page. A paragraph is a container for text; an image is a container for a picture; a button is a container for something you can click. The label (the tag name) tells the browser what kind of content the container holds, and the browser uses that label to decide how to display the content and how it relates to neighbouring containers.

---

### Key Characteristics

- **Tag-delimited syntax**: Most elements begin with `<name>` and end with `</name>`, though void elements such as `<img>` and `<br>` have no end tag.
- **Tree-structured**: Elements nest inside one another to form a document tree; the `<html>` element is the root.
- **Categorised by content type**: Every element belongs to one or more content categories (metadata, flow, sectioning, heading, phrasing, embedded, interactive, palpable, script-supporting) that govern its allowed placement and children.
- **Rule-governed nesting**: An element may only contain children permitted by its content model; violations produce non-conforming documents.
- **Layout behaviour varies by type**: Block-level elements occupy the full available inline width and start on a new line; inline elements flow within text; inline-block elements sit in the text flow but accept width and height.
- **Living Standard governance**: The authoritative definition is maintained by WHATWG as the HTML Living Standard, not a frozen W3C recommendation.

---

### Prerequisites

To follow this cheat sheet productively, a learner should be comfortable with:

1. **Basic computer literacy** — file creation, editing, and opening files in a browser.
2. **Fundamental HTML document structure** — the roles of `<!DOCTYPE html>`, `<html>`, `<head>`, and `<body>`.
3. **How to create and save an `.html` file** and open it in a browser to inspect output.
4. **Introductory CSS concepts** — selectors, the box model, and the `display` property, because element types are most clearly observed when borders, padding, and dimensions are applied.
5. **Browser developer tools** — the Elements panel for inspecting the parsed DOM tree.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS layout** | Element type (block, inline, inline-block) determines default formatting behaviour. |
| **DOM scripting (JavaScript)** | Elements become DOM nodes that scripts create, query, and modify. |
| **Web accessibility** | Correct element choice and heading nesting determine screen-reader navigation and semantic meaning. |
| **Semantic Web / SEO** | Sectioning and heading content categories convey document structure to crawlers and assistive technologies. |
| **Web standards & validation** | Content models and nesting rules are enforced by validators and the HTML parser. |

---

### Core Concepts / Features

The remainder of this cheat sheet treats the following concepts, each with the same uniform structure:

1. **Elements**
2. **Element Types**
   - Block-level elements
   - Inline elements
   - Inline-block elements
3. **Element Categories**
   - Metadata content
   - Flow content
   - Sectioning content
   - Heading content
   - Phrasing content
   - Embedded content
   - Interactive content
   - Palpable content
   - Script-supporting content
4. **Element Nesting**
5. **Element Content Model**
6. **Element Best Practices**

---

## 1. Elements

### Definitions

**Core Definition**
An element is the basic unit of an HTML document, consisting of a start tag, content, and an end tag (or a self-closing syntax for void elements). It represents a discrete piece of meaning or functionality, such as a paragraph, a link, or an image.

**Technical Definition**
Per the WHATWG standard, an element is a DOM node whose node type is `ELEMENT_NODE`. Its interface is determined by its tag name (for example, `HTMLParagraphElement` for `<p>`), and its permitted content is defined by a normative content model. The HTML syntax defines six kinds of elements: void elements, raw text elements, escapable raw text elements, foreign elements, template elements, and ordinary elements.

**Beginner-Friendly Explanation**
An element is a single “thing” you place on a page. The tag name is the label that tells the browser what that thing is. If you write `<p>Hello</p>`, you have created a paragraph element whose content is the word “Hello”.

---

### Purposes

- **To convey meaning**: Element names describe the role of their content (paragraph, heading, navigation, emphasis).
- **To structure documents**: Elements organise content into a hierarchical tree that browsers and assistive technologies can interpret.
- **To enable styling**: Elements provide hooks for CSS selectors to apply visual presentation.
- **To enable scripting**: Elements expose DOM interfaces that JavaScript can manipulate.
- **To support accessibility**: Correct element choice gives screen readers and other tools the semantic information needed to present content meaningfully.

---

### Syntax Rules and Structure

#### Complete General Syntax

```
<element-name attribute="value" ...>
    content
</element-name>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<element-name>` | The **start tag**. Contains the tag name and optionally one or more attributes. |
| `attribute="value"` | An attribute modifying the element’s behaviour or providing metadata. Attributes appear only in the start tag. |
| `content` | The element’s children — text, other elements, comments, or nothing. |
| `</element-name>` | The **end tag**. Omitted for void elements and in certain optional-tag cases. |

For a **void element**:

```
<img src="photo.jpg" alt="A mountain landscape">
```

No end tag is permitted.

#### Syntax Rules

1. **Tag names are case-insensitive** in HTML (though lowercase is the universal convention).
2. **Attributes must be separated by whitespace** and must not be repeated on the same element.
3. **Elements must be properly nested**: `<p><em>text</em></p>` is valid; `<p><em>text</p></em>` is overlapping and invalid.
4. **Void elements must not have end tags**: `<br>` is valid; `<br></br>` is invalid.
5. **Raw-text elements** such as `<script>` and `<style>` treat their content as text and do not parse nested tags normally.
6. **The `template` element** has a special content model whose children live in a separate document fragment until instantiated.

#### Constraints and Limitations

- An element’s content model limits which children are allowed; a `<p>` cannot contain a `<div>` or another block-level element in conforming HTML.
- Interactive elements must not be nested inside other interactive elements (for example, an `<a>` must not contain a `<button>`).
- The `html`, `head`, and `body` elements have restricted placement and content rules.
- Custom elements (via `customElements.define()`) must contain a hyphen in their name to avoid collision with future standard elements.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Paragraph Element

**Setup guide**: Create a file named `paragraph.html` and paste the code below. Open it in any modern browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Element Example — Paragraph</title>
</head>
<body>
  <!-- Start tag <p> opens the paragraph element -->
  <p>
    <!-- Text content is the child of the element -->
    This is a paragraph element. It contains plain text.
  <!-- End tag </p> closes the paragraph -->
  </p>
</body>
</html>
```

**Expected output**: A single line of text rendered in the browser:

> This is a paragraph element. It contains plain text.

The paragraph occupies the full width of the body and sits on its own line because `<p>` is a block-level element by default.

**Why this output occurs**: The browser parses the start tag `<p>`, creates an `HTMLParagraphElement` node, inserts the text node as its child, and renders the paragraph as a block-level box. The closing `</p>` signals that subsequent content belongs outside the paragraph.

---

#### Example 2: A Void Element with Attributes

**Setup guide**: Save as `void.html` and open in a browser. Place a small image file named `photo.jpg` in the same directory, or replace the `src` value with any accessible image URL.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Element Example — Void Element</title>
</head>
<body>
  <!-- <img> is a void element: it has no end tag -->
  <img
    src="photo.jpg"          <!-- required: URL of the image resource -->
    alt="A mountain at sunrise"  <!-- required: alternative text for accessibility -->
    width="400"               <!-- optional: intrinsic display width in CSS pixels -->
    height="300"              <!-- optional: intrinsic display height in CSS pixels -->
  >
  <!-- The element ends here; no </img> is allowed -->
</body>
</html>
```

**Expected output**: The image `photo.jpg` is displayed at 400 × 300 CSS pixels. If the image cannot load, the alternative text “A mountain at sunrise” is shown instead.

**Why this output occurs**: `<img>` is a void element, so the parser creates an element node with the specified attributes and no child nodes. The browser fetches the `src` resource and renders it; the `alt` text provides a fallback and accessibility description.

---

#### Example 3: A Nested Element Tree

**Setup guide**: Save as `nested.html` and open in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Element Example — Nesting</title>
</head>
<body>
  <!-- Outer block-level element -->
  <div>
    <!-- Nested heading element -->
    <h2>Welcome</h2>
    <!-- Nested paragraph element -->
    <p>
      This paragraph contains
      <!-- Nested inline emphasis element -->
      <em>emphasised text</em>
      inside it.
    </p>
  </div>
</body>
</html>
```

**Expected output**:

> **Welcome**
>
> This paragraph contains *emphasised text* inside it.

**Why this output occurs**: The `<div>` creates a block-level container. Inside it, `<h2>` creates a heading block and `<p>` creates a paragraph block. The `<em>` element inside the paragraph is inline, so it flows within the paragraph’s text line and renders the enclosed words in italics by default. The nesting is valid because `<div>` accepts flow content (including headings and paragraphs), and `<p>` accepts phrasing content (including `<em>`).

---

### Real-World Cases

1. **Article body content**: News sites use `<p>` elements for paragraphs, `<h2>`–`<h3>` for subheadings, and `<blockquote>` for quotations. Each element conveys the role of its content, enabling reader modes and accessibility tools to present the article correctly.
2. **Navigation menus**: Although menus can be built from `<div>` elements, using `<nav>` with nested `<ul>` and `<li>` elements gives screen-reader users a navigable landmark and a list structure.
3. **Image galleries**: Each thumbnail is typically an `<img>` element wrapped in an `<a>` (when clickable) or a `<figure>` with a `<figcaption>` (when a caption is needed). Choosing the right elements avoids the need for excessive ARIA attributes.

---

### References

- HTML Standard — Elements — https://html.spec.whatwg.org/multipage/dom.html#elements
- MDN Web Docs — HTML elements reference — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements
- Wikipedia — HTML element — https://en.wikipedia.org/wiki/HTML_element

---

## 2. Element Types

Element types describe the **default layout behaviour** of an element in normal flow. Historically, HTML elements were described as block-level or inline-level; modern HTML uses content categories instead, but the block/inline distinction remains the primary mental model for CSS layout and is still widely used in documentation and teaching.

> **Important note on standardisation**: The HTML Living Standard no longer defines “block-level elements” and “inline elements” as normative categories. The distinction is a legacy presentational concept largely governed by CSS’s `display` property. Nevertheless, browsers’ default stylesheets implement these behaviours, and the terminology remains indispensable for understanding layout.

---

### 2.1 Block-Level Elements

#### Definitions

**Core Definition**
A block-level element is an element that, by default, starts on a new line and occupies the full width available in its containing block, stretching from the left edge to the right edge of its parent.

**Technical Definition**
In CSS terms, a block-level element generates a block-level box in normal flow. Its `display` value is `block` (or another block-level value such as `list-item` or `table`). Block-level boxes participate in a block formatting context: they stack vertically, and their width defaults to `auto`, which resolves to the containing block’s content width.

**Beginner-Friendly Explanation**
A block-level element is like a full-width paragraph in a document. It always begins on a fresh line and takes up the entire horizontal space available, pushing the next element onto a new line below it.

---

#### Purposes

- **To create major page structures**: headers, footers, sections, articles, and navigation regions.
- **To stack content vertically**: block-level elements naturally form a vertical document flow.
- **To control full-width backgrounds and borders**: because they span the container width by default.
- **To contain other block-level and inline content**: most block-level elements accept flow content, allowing nested structural hierarchies.

---

#### Syntax Rules and Structure

**General syntax**

```
<block-element>
    flow content
</block-element>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<block-element>` | A tag whose default CSS `display` is `block` (e.g. `<div>`, `<p>`, `<section>`, `<h1>`). |
| `flow content` | Child content permitted by the element’s content model; may include other block-level elements and inline content. |
| `</block-element>` | The end tag (omitted only for void block-level elements such as `<hr>`). |

**Common block-level elements**: `<address>`, `<article>`, `<aside>`, `<blockquote>`, `<div>`, `<dl>`, `<fieldset>`, `<figure>`, `<footer>`, `<form>`, `<h1>`–`<h6>`, `<header>`, `<hr>`, `<li>`, `<main>`, `<nav>`, `<ol>`, `<p>`, `<pre>`, `<section>`, `<table>`, `<ul>`.

#### Syntax Rules

1. Block-level elements may contain inline elements and, depending on their content model, other block-level elements.
2. A block-level element must not be placed inside a `<p>` element, because `<p>`’s content model is phrasing content only.
3. The `width` and `height` properties apply to block-level boxes.
4. Horizontal margins and padding affect the layout of block-level elements in ways that inline elements cannot replicate.

#### Constraints and Limitations

- Not every block-level element can contain every other block-level element; `<p>` cannot contain `<div>`, and `<li>` has specific parent requirements.
- Using block-level elements purely for visual styling (for example, `<div>` for a button) sacrifices semantics and accessibility.
- Nested block-level elements with no text or embedded content can violate the palpable-content guideline.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Block-Level Elements Stack Vertically

**Setup guide**: Save as `block.html` and open in a browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Block-Level Elements</title>
  <style>
    /* Visualise the box of each block-level element */
    .box {
      border: 2px solid #333;  /* draw a border around each element */
      padding: 8px;            /* add internal space */
      margin-bottom: 8px;      /* add space below each box */
      background-color: #f0f0f0; /* light grey background */
    }
  </style>
</head>
<body>
  <!-- Each <div> is block-level and will stack vertically -->
  <div class="box">First block-level element</div>
  <div class="box">Second block-level element</div>
  <div class="box">Third block-level element</div>
</body>
</html>
```

**Expected output**: Three grey rectangular boxes with borders, stacked one above the other. Each box spans the full width of the browser viewport (minus any default body margin).

**Why this output occurs**: Each `<div>` has `display: block` by default. Block-level boxes occupy the full available width and begin on a new line, so the three boxes cannot sit side by side in normal flow.

---

##### Example 2: A Block-Level Element Containing Nested Blocks

**Setup guide**: Save as `nested-block.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested Block-Level Elements</title>
  <style>
    section { border: 2px solid navy; padding: 10px; margin: 10px 0; }
    h2      { margin-top: 0; color: navy; }
    p       { margin-bottom: 0; }
  </style>
</head>
<body>
  <!-- <section> is block-level and accepts flow content -->
  <section>
    <!-- <h2> is a block-level heading -->
    <h2>Section Title</h2>
    <!-- <p> is a block-level paragraph -->
    <p>This paragraph is nested inside the section.</p>
  </section>
</body>
</html>
```

**Expected output**: A navy-bordered rectangular section containing a heading “Section Title” followed by a paragraph. Both the heading and the paragraph start on new lines within the section.

**Why this output occurs**: `<section>` accepts flow content, which includes headings and paragraphs. Each child is block-level, so they stack vertically inside the section’s content box.

---

#### Real-World Cases

1. **Page layout containers**: A `<div>` wrapping the entire page content is a classic block-level container that establishes the main content column.
2. **Article cards**: Each card in a grid is often a `<div>` or `<article>` with `display: block`, containing a heading, image, and paragraph.
3. **Form sections**: A `<fieldset>` groups related form controls; its block-level nature ensures the group occupies its own horizontal band.

---

#### References

- MDN Web Docs — Block-level elements — https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements
- MDN Web Docs — Normal Flow — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flow_layout/Block_and_inline_layout_in_normal_flow
- SitePoint — HTML Block and Inline Elements — https://www.sitepoint.com/block-and-inline-elements/

---

### 2.2 Inline Elements

#### Definitions

**Core Definition**
An inline element is an element that flows within a line of text, occupying only as much horizontal space as its content requires and not forcing a line break before or after itself.

**Technical Definition**
An inline-level element generates an inline-level box that participates in an inline formatting context. Its default `display` value is `inline`. Width and height properties do not apply to non-replaced inline boxes; vertical margins and padding do not affect line height, though horizontal margins and padding do affect spacing within the line.

**Beginner-Friendly Explanation**
An inline element is like a highlighted word in a sentence: it sits within the text flow, takes up only the space of the word itself, and does not push the rest of the sentence onto a new line.

---

#### Purposes

- **To mark up parts of text**: emphasis (`<em>`), importance (`<strong>`), code (`<code>`), abbreviations (`<abbr>`).
- **To create hyperlinks within prose**: the `<a>` element is inline by default.
- **To insert line breaks or inline media**: `<br>` and `<img>` (a replaced inline element) participate in text flow.
- **To preserve the natural reading flow**: inline elements do not disrupt the sentence-level layout of paragraphs.

---

#### Syntax Rules and Structure

**General syntax**

```
<inline-element>text or inline content</inline-element>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<inline-element>` | A tag whose default CSS `display` is `inline` (e.g. `<span>`, `<em>`, `<a>`, `<strong>`, `<code>`). |
| `text or inline content` | Phrasing content only; inline elements cannot contain block-level elements in conforming HTML. |
| `</inline-element>` | The end tag. |

**Common inline elements**: `<a>`, `<abbr>`, `<b>`, `<bdi>`, `<bdo>`, `<br>`, `<cite>`, `<code>`, `<data>`, `<dfn>`, `<em>`, `<i>`, `<kbd>`, `<mark>`, `<q>`, `<rp>`, `<rt>`, `<ruby>`, `<s>`, `<samp>`, `<small>`, `<span>`, `<strong>`, `<sub>`, `<sup>`, `<time>`, `<u>`, `<var>`, `<wbr>`.

#### Syntax Rules

1. Inline elements may contain only phrasing content; they must not contain block-level elements such as `<div>`, `<p>`, or `<h1>`.
2. `width` and `height` do not apply to non-replaced inline elements.
3. Vertical `margin` and `padding` on inline elements do not affect line box height (though they may visually overlap adjacent lines).
4. Inline elements may be nested inside one another (e.g. `<strong><em>text</em></strong>`), provided no interactive nesting rules are violated.

#### Constraints and Limitations

- Inline elements cannot be used to build full-width layout containers without changing their `display` value.
- An `<a>` element must not contain another `<a>` or any other interactive element.
- Inline elements with large vertical padding can overlap surrounding lines, creating accessibility and readability problems.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Inline Elements Within a Paragraph

**Setup guide**: Save as `inline.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inline Elements</title>
  <style>
    /* Borders reveal the inline boxes without changing their flow behaviour */
    a, strong, em { border: 1px solid crimson; padding: 2px; }
  </style>
</head>
<body>
  <p>
    This is a paragraph with
    <a href="#">a link</a>,
    <strong>important text</strong>, and
    <em>emphasised text</em>
    — all flowing within the same line.
  </p>
</body>
</html>
```

**Expected output**: A single paragraph in which “a link”, “important text”, and “emphasised text” are each outlined with a thin crimson border, but all remain on the same line as the surrounding words.

**Why this output occurs**: `<a>`, `<strong>`, and `<em>` generate inline-level boxes. They sit in the inline formatting context of the paragraph, so they do not force line breaks and occupy only the width of their content.

---

##### Example 2: Inline Element Cannot Contain a Block-Level Element (Invalid)

**Setup guide**: Save as `invalid-inline.html` and inspect it with an HTML validator or the browser’s DOM inspector.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Invalid Inline Nesting</title>
</head>
<body>
  <p>
    <!-- INVALID: <span> is inline and must not contain <div>, which is block-level -->
    <span>
      <div>This block-level div is illegally nested inside an inline span.</div>
    </span>
  </p>
</body>
</html>
```

**Expected output in the browser**: The browser will still render the text, but the DOM inspector will show that the parser has **foster-parented** or restructured the invalid nesting. A validator will report a conformance error.

**Why this output occurs**: `<span>` is an inline element whose content model permits only phrasing content. `<div>` is flow content, not phrasing content, so the HTML is non-conforming. Browsers apply error-recovery parsing, but the resulting DOM may not match the author’s intent.

---

#### Real-World Cases

1. **Article prose**: Inline elements such as `<a>`, `<em>`, `<strong>`, and `<code>` are used throughout body text to add meaning without breaking paragraph flow.
2. **Inline code snippets**: Technical documentation uses `<code>` inline to distinguish code tokens from surrounding prose.
3. **Abbreviations and definitions**: `<abbr title="HyperText Markup Language">HTML</abbr>` provides an expansion tooltip while remaining inline.

---

#### References

- MDN Web Docs — Inline elements — https://developer.mozilla.org/en-US/docs/Web/HTML/Inline_elements
- MDN Web Docs — Inline formatting context — https://developer.mozilla.org/en-US/docs/Web/CSS/Inline_formatting_context
- W3C — HTML 5.1: Phrasing content — https://www.w3.org/TR/html51/dom.html#phrasing-content

---

### 2.3 Inline-Block Elements

#### Definitions

**Core Definition**
An inline-block element is an element that participates in the inline flow — sitting alongside text on the same line — but that also accepts width, height, vertical margins, and vertical padding like a block-level element.

**Technical Definition**
An element with `display: inline-block` generates an inline-level box that establishes a block formatting context internally. Externally it behaves as an inline-level box (it sits in the line box), but internally its children are laid out as block-level boxes, and the element itself respects width, height, margin, and padding in all directions.

**Beginner-Friendly Explanation**
An inline-block element is a hybrid: it sits in a line of text like a word, but you can give it a fixed width and height and treat it like a small box. Think of a row of buttons or a set of cards that flow side by side but can each have their own dimensions.

---

#### Purposes

- **To create horizontally flowing boxes with controlled dimensions**: navigation buttons, tag chips, product cards.
- **To align inline content while retaining block-level sizing**: icons with labels, form controls with fixed widths.
- **To avoid floats or flexbox for simple horizontal arrangements**: inline-block is a lightweight layout tool for small groups of boxes.
- **To preserve text-flow participation**: inline-block elements remain sensitive to `text-align` and vertical alignment within a line.

---

#### Syntax Rules and Structure

**General syntax**

```css
selector {
    display: inline-block;
    width: <length>;      /* now applies */
    height: <length>;     /* now applies */
    margin: <length>;     /* all four sides apply */
    padding: <length>;    /* all four sides apply */
}
```

**Component breakdown**

| Component | Description |
|---|---|
| `display: inline-block` | The CSS declaration that changes the element’s outer display type to inline and inner display type to block flow. |
| `width` / `height` | Explicit dimensions that inline-block boxes respect, unlike pure inline boxes. |
| `margin` / `padding` | All four sides affect layout, unlike pure inline boxes where vertical margin/padding do not affect line height. |

Any HTML element can become inline-block via CSS; there is no dedicated HTML tag for inline-block. Common examples include styling `<li>` elements for horizontal navigation or `<a>` elements as buttons.

#### Syntax Rules

1. `display: inline-block` is a CSS declaration, not an HTML attribute or element type.
2. Inline-block boxes are affected by **whitespace between elements** in the HTML source, which can create small gaps; this is a well-known layout quirk.
3. Vertical alignment of inline-block boxes within a line is controlled by the `vertical-align` property.
4. Inline-block elements can contain block-level children because their inner display type is block flow.

#### Constraints and Limitations

- Whitespace collapsing between inline-block elements can introduce unwanted gaps; solutions include removing whitespace in the source, using HTML comments between elements, or setting the parent’s `font-size` to `0`.
- Inline-block is not ideal for complex two-dimensional layouts; flexbox and grid are more appropriate for modern layout.
- Very tall inline-block elements can affect line height and vertical rhythm in unexpected ways.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Horizontal Navigation Buttons with Inline-Block

**Setup guide**: Save as `inline-block.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inline-Block Navigation</title>
  <style>
    /* Remove default list styling and spacing */
    .nav {
      list-style: none;   /* remove bullet points */
      padding: 0;         /* remove default padding */
      margin: 0;          /* remove default margin */
      font-size: 0;       /* remove whitespace gaps between inline-block items */
    }
    .nav li {
      display: inline-block;  /* make each list item sit in the line */
      font-size: 16px;        /* restore font size for content */
    }
    .nav a {
      display: inline-block;  /* make the link a box */
      padding: 10px 20px;     /* comfortable clickable area */
      background-color: #0077cc;
      color: white;
      text-decoration: none;
      border-radius: 4px;
      margin-right: 4px;      /* spacing between buttons */
    }
  </style>
</head>
<body>
  <ul class="nav">
    <li><a href="#">Home</a></li>
    <li><a href="#">About</a></li>
    <li><a href="#">Services</a></li>
    <li><a href="#">Contact</a></li>
  </ul>
</body>
</html>
```

**Expected output**: Four blue rounded buttons arranged horizontally in a row, each with padding and white text, separated by a small gap.

**Why this output occurs**: Each `<li>` is set to `display: inline-block`, so the list items sit side by side in the inline flow. Each `<a>` is also inline-block, giving it a clickable box with padding and dimensions. The `font-size: 0` on the parent collapses the whitespace text nodes between inline-block items, eliminating unwanted gaps.

---

##### Example 2: Inline-Block vs Pure Inline with Width

**Setup guide**: Save as `inline-block-width.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inline-Block Width Demonstration</title>
  <style>
    .pure-inline {
      display: inline;        /* width will be ignored */
      width: 200px;           /* has no effect on non-replaced inline boxes */
      background-color: #ffcccc;
    }
    .inline-block {
      display: inline-block;  /* width will be respected */
      width: 200px;
      background-color: #ccffcc;
    }
  </style>
</head>
<body>
  <p>
    <span class="pure-inline">Pure inline with width:200px</span>
    <span class="inline-block">Inline-block with width:200px</span>
  </p>
</body>
</html>
```

**Expected output**: The pure-inline span’s background extends only as far as its text content, because `width` is ignored. The inline-block span’s background extends to exactly 200 pixels, regardless of its text content.

**Why this output occurs**: Non-replaced inline boxes ignore `width` and `height`. The inline-block box, by contrast, establishes a block formatting context internally and therefore honours explicit dimensions.

---

#### Real-World Cases

1. **Tag clouds and chips**: Each tag is an inline-block element with padding and a background, flowing naturally in a line and wrapping at the container edge.
2. **Product grids with fixed-size cards**: Before CSS Grid, inline-block was commonly used to place product cards side by side while giving each card fixed dimensions.
3. **Form layouts**: A label and its input can be placed on the same line using inline-block, with the label given a fixed width for alignment.

---

#### References

- MDN Web Docs — `display` — https://developer.mozilla.org/en-US/docs/Web/CSS/display
- SitePoint — HTML Block and Inline Elements — https://www.sitepoint.com/block-and-inline-elements/
- LabEx — What is `inline-block` display? — https://labex.io/tutorials/css-what-is-inline-block-display-297478

---

## 3. Element Categories

Content categories are **groupings of elements that share common characteristics**. They are used to define each element’s content model — in other words, which descendants an element may contain and where it may itself appear. An element may belong to zero, one, or multiple categories.

The categories below follow the MDN and WHATWG taxonomy. The seven main categories are metadata, flow, sectioning, heading, phrasing, embedded, and interactive. Palpable content and script-supporting content are additional categories with special roles.

---

### 3.1 Metadata Content

#### Definitions

**Core Definition**
Metadata content consists of elements that modify the presentation or behaviour of the rest of the document, establish relationships to other documents, or convey out-of-band information about the document.

**Technical Definition**
Metadata content elements are those that belong to the metadata content category as defined by the HTML specification. They typically appear in the document’s `<head>`, though some (such as `<script>` and `<style>`) may also appear in the body under certain conditions. Metadata elements do not render as visible page content in the normal sense.

**Beginner-Friendly Explanation**
Metadata content is “information about the page” rather than the page’s visible content. It tells the browser what the page title is, which character encoding to use, which stylesheet to load, and which scripts to run.

---

#### Purposes

- **To define the document title** displayed in browser tabs and search results (`<title>`).
- **To specify character encoding and viewport behaviour** (`<meta>`).
- **To link external resources** such as stylesheets and icons (`<link>`).
- **To embed or reference scripts and styles** (`<script>`, `<style>`).
- **To set the base URL for relative links** (`<base>`).

---

#### Syntax Rules and Structure

**General syntax (head section)**

```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document Title</title>
    <link rel="stylesheet" href="styles.css">
    <script src="app.js" defer></script>
    <style>
        /* internal CSS rules */
    </style>
    <base href="https://example.com/">
</head>
```

**Component breakdown**

| Element | Role |
|---|---|
| `<meta>` | Provides metadata such as character encoding (`charset`), viewport settings, description, author, and keywords. |
| `<title>` | Sets the document title; required in every conforming HTML document. |
| `<link>` | Links external resources; most commonly stylesheets. |
| `<script>` | Embeds or references JavaScript. |
| `<style>` | Embeds CSS rules directly in the document. |
| `<base>` | Specifies the base URL and/or default target for relative URLs. |

**Metadata elements**: `<base>`, `<link>`, `<meta>`, `<noscript>`, `<script>`, `<style>`, `<template>`, `<title>`.

#### Syntax Rules

1. `<title>` must appear exactly once per document and must be inside `<head>`.
2. `<meta charset>` should appear within the first 1024 bytes of the document for reliable encoding detection.
3. `<base>` must appear before any element that uses a relative URL.
4. `<script>` and `<style>` may appear in the body, but their metadata-category membership is context-dependent.

#### Constraints and Limitations

- `<title>` cannot contain other elements; its content is plain text.
- Multiple `<base>` elements are allowed, but only the first one with an `href` or `target` attribute is used.
- `<meta>` must not be used to replicate functionality already provided by `<title>`, `<link>`, or `<base>`.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: A Complete Metadata Head

**Setup guide**: Save as `metadata.html` and open in a browser. Observe the browser tab title and the document’s behaviour.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Character encoding: ensures correct rendering of text -->
  <meta charset="UTF-8">

  <!-- Viewport: makes the page responsive on mobile devices -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Description: used by search engines and social media -->
  <meta name="description" content="A comprehensive guide to HTML elements.">

  <!-- Document title: appears in the browser tab and search results -->
  <title>HTML Element Fundamentals</title>

  <!-- External stylesheet -->
  <link rel="stylesheet" href="styles.css">

  <!-- External script, deferred until HTML parsing is complete -->
  <script src="app.js" defer></script>

  <!-- Base URL for relative links -->
  <base href="https://example.com/docs/">
</head>
<body>
  <p>Metadata is not visible here, but it shapes how this page behaves.</p>
</body>
</html>
```

**Expected output**: The browser tab displays “HTML Element Fundamentals”. The page applies styles from `styles.css`. The script `app.js` executes after parsing. Relative links on the page resolve against `https://example.com/docs/`.

**Why this output occurs**: Each metadata element performs a specific role: `<title>` sets the tab label, `<link>` loads CSS, `<script>` loads JavaScript, `<meta charset>` ensures correct character interpretation, and `<base>` changes URL resolution.

---

##### Example 2: Metadata Elements Also Used in the Body

**Setup guide**: Save as `body-metadata.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Metadata in Body</title>
</head>
<body>
  <p>This paragraph is styled by an internal style element placed later.</p>

  <!-- <style> is metadata content but may appear in the body -->
  <style>
    p { color: darkgreen; font-weight: bold; }
  </style>

  <!-- <script> is metadata, flow, phrasing, and script-supporting content -->
  <script>
    console.log("Script executed inside the body.");
  </script>
</body>
</html>
```

**Expected output**: The paragraph appears in bold dark green. The browser console logs “Script executed inside the body.”

**Why this output occurs**: `<style>` and `<script>` are members of multiple content categories. When used in the body, `<style>` still applies CSS rules, and `<script>` still executes, even though both are metadata content by primary classification.

---

#### Real-World Cases

1. **SEO metadata**: `<title>` and `<meta name="description">` directly influence how search engines display a page in results.
2. **Responsive design**: The viewport meta tag is essential for mobile rendering; without it, mobile browsers assume a desktop-width viewport.
3. **Favicon and app icons**: `<link rel="icon">` and `<link rel="apple-touch-icon">` provide icons for browser tabs and mobile home screens.

---

#### References

- MDN Web Docs — Metadata content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#metadata_content
- WHATWG — Metadata content — https://html.spec.whatwg.org/multipage/dom.html#metadata-content-2
- W3C — HTML 5.2: Metadata content — https://www.w3.org/TR/html52/document-metadata.html#metadata-content

---

### 3.2 Flow Content

#### Definitions

**Core Definition**
Flow content is the broad category of elements that can appear inside the `<body>` element and participate in the normal document flow. It includes headings, sections, paragraphs, lists, media, forms, and most other visible content.

**Technical Definition**
Flow content is a superset category defined by the HTML specification. Most elements that render visible content are flow content. The category includes text nodes (except those consisting only of whitespace) and a wide range of elements. Some elements belong to flow content only under specific conditions.

**Beginner-Friendly Explanation**
Flow content is “anything that can normally go on a page”. If an element is part of the visible page body — a paragraph, a heading, an image, a form — it is almost certainly flow content.

---

#### Purposes

- **To provide a permissive container model**: `<div>`, `<body>`, and `<section>` accept flow content, enabling flexible page composition.
- **To define the default placement context** for most elements.
- **To include text and embedded content** within the document body.
- **To support mixed content structures** such as paragraphs, lists, and media blocks.

---

#### Syntax Rules and Structure

**General syntax**

```
<flow-container>
    <!-- flow content: headings, paragraphs, lists, media, forms, etc. -->
</flow-container>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<flow-container>` | An element whose content model allows flow content (e.g. `<body>`, `<div>`, `<section>`, `<article>`). |
| Flow content | Any element or text node belonging to the flow content category. |
| `</flow-container>` | The end tag. |

**Flow content elements (representative list)**: `<a>`, `<abbr>`, `<address>`, `<article>`, `<aside>`, `<audio>`, `<b>`, `<blockquote>`, `<br>`, `<button>`, `<canvas>`, `<code>`, `<details>`, `<div>`, `<dl>`, `<em>`, `<fieldset>`, `<figure>`, `<footer>`, `<form>`, `<h1>`–`<h6>`, `<header>`, `<hr>`, `<img>`, `<input>`, `<li>`, `<main>`, `<nav>`, `<ol>`, `<p>`, `<pre>`, `<section>`, `<table>`, `<ul>`, `<video>`.

#### Syntax Rules

1. Flow content may include both block-level and inline elements, as well as text.
2. Not all flow content is palpable; empty `<div>` elements are flow content but not palpable.
3. Conditional flow-content members include `<area>`, `<link>`, and `<meta>` when specific attributes are present.

#### Constraints and Limitations

- Flow content cannot appear directly inside `<head>` or inside phrasing-only containers such as `<p>`.
- Some flow-content elements impose their own content-model restrictions on their children.
- Overusing generic flow containers (`<div>`) without semantic elements reduces accessibility and maintainability.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: A `<div>` Accepting Diverse Flow Content

**Setup guide**: Save as `flow.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Flow Content</title>
</head>
<body>
  <!-- <div> accepts flow content: a heading, a paragraph, and a list -->
  <div>
    <h1>Flow Content Example</h1>
    <p>This paragraph is flow content.</p>
    <ul>
      <li>List item one (flow content)</li>
      <li>List item two (flow content)</li>
    </ul>
    <img src="diagram.png" alt="A diagram" width="300">
  </div>
</body>
</html>
```

**Expected output**: A heading, a paragraph, a bulleted list, and an image, all stacked vertically inside the `div`.

**Why this output occurs**: `<div>` has a content model of flow content. Headings, paragraphs, lists, and images are all flow content, so they are valid children and render according to their own default styles.

---

##### Example 2: Conditional Flow Content — `<link>` with `itemprop`

**Setup guide**: Save as `conditional-flow.html`. This example illustrates a specification detail; it may not produce a visible difference in all browsers.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Conditional Flow Content</title>
</head>
<body>
  <div itemscope itemtype="https://schema.org/Book">
    <!-- <link> is flow content only when the itemprop attribute is present -->
    <link itemprop="url" href="https://example.com/book">
    <span itemprop="name">Example Book Title</span>
  </div>
</body>
</html>
```

**Expected output**: The book title is displayed. The `<link>` element is not rendered visually but provides machine-readable metadata.

**Why this output occurs**: `<link>` belongs to the flow content category **conditionally**, specifically when the `itemprop` attribute is present. This allows microdata to be expressed within the body without violating content-model rules.

---

#### Real-World Cases

1. **Article bodies**: The `<article>` element accepts flow content, allowing authors to combine headings, paragraphs, images, lists, and quotations within a single semantic container.
2. **Landing page sections**: Each `<section>` is a flow-content container that groups related headings and prose.
3. **Form layouts**: `<form>` accepts flow content, so labels, inputs, fieldsets, and buttons can be arranged within it.

---

#### References

- MDN Web Docs — Flow content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#flow_content
- WHATWG — Flow content — https://html.spec.whatwg.org/multipage/dom.html#flow-content-2
- W3C — HTML 5.2: Flow content — https://www.w3.org/TR/html52/dom.html#flow-content

---

### 3.3 Sectioning Content

#### Definitions

**Core Definition**
Sectioning content is a subset of flow content that creates a new section in the document outline, defining the scope of headings and of `<header>` and `<footer>` elements within that section.

**Technical Definition**
Sectioning content elements are those whose use creates a section in the current outline. They include `<article>`, `<aside>`, `<nav>`, and `<section>`. The HTML outline algorithm (which was never implemented consistently in browsers) theoretically uses these elements to generate a document outline; in practice, heading levels remain the primary structural signal.

**Beginner-Friendly Explanation**
Sectioning elements divide a page into meaningful regions: a navigation area, a sidebar, an article, or a generic section. They help both humans and machines understand how the page is organised.

---

#### Purposes

- **To divide a document into semantic regions**: navigation, main content, complementary content, and articles.
- **To scope headings and footers**: a `<header>` inside a `<section>` belongs to that section.
- **To improve accessibility and machine readability**: landmarks and document structure become explicit.
- **To support styling and scripting hooks** for distinct page areas.

---

#### Syntax Rules and Structure

**General syntax**

```
<sectioning-element>
    heading content (usually)
    flow content
</sectioning-element>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<sectioning-element>` | One of `<article>`, `<aside>`, `<nav>`, or `<section>`. |
| Heading content | Sectioning elements typically begin with a heading (`<h1>`–`<h6>`) identifying the section. |
| Flow content | The body of the section. |
| `</sectioning-element>` | The end tag. |

**Sectioning elements**: `<article>`, `<aside>`, `<nav>`, `<section>`.

#### Syntax Rules

1. Each sectioning element should generally have an accessible name, often provided by a heading child.
2. `<nav>` should be used for major navigation blocks, not every group of links.
3. `<article>` should be self-contained and independently distributable (e.g. a blog post or news story).
4. `<section>` is a generic sectioning element; it should be used when a more specific element (`<article>`, `<aside>`, `<nav>`) is not appropriate.

#### Constraints and Limitations

- The theoretical outline algorithm is not reliably implemented; do not rely on sectioning elements alone to create a document outline. Use properly nested headings.
- `<section>` without a heading can be ambiguous to assistive technologies.
- `<aside>` content should be tangentially related to the surrounding content, not essential to it.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: A Page with Navigation, Article, and Aside

**Setup guide**: Save as `sectioning.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sectioning Content</title>
  <style>
    nav   { background: #eee; padding: 10px; }
    article { border-left: 4px solid #0077cc; padding-left: 16px; margin: 16px 0; }
    aside { background: #f9f9f9; padding: 10px; font-style: italic; }
  </style>
</head>
<body>
  <!-- Navigation region -->
  <nav>
    <a href="#home">Home</a> |
    <a href="#articles">Articles</a> |
    <a href="#about">About</a>
  </nav>

  <!-- Main article: self-contained content -->
  <article>
    <h1>Understanding Sectioning Content</h1>
    <p>Sectioning elements divide a document into meaningful regions.</p>
  </article>

  <!-- Complementary sidebar content -->
  <aside>
    <p>Related tip: use headings to label each section.</p>
  </aside>
</body>
</html>
```

**Expected output**: A navigation bar with three links, followed by an article with a heading and paragraph, and then a styled aside note.

**Why this output occurs**: `<nav>`, `<article>`, and `<aside>` are sectioning elements. Each creates a distinct region with its own semantic role, and the CSS styles differentiate them visually. The `<article>` is self-contained, and the `<aside>` is tangentially related.

---

##### Example 2: Nested Sections with Headings

**Setup guide**: Save as `nested-sections.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested Sections</title>
</head>
<body>
  <article>
    <h1>Guide to HTML Sections</h1>

    <section>
      <h2>What Is a Section?</h2>
      <p>A section groups related content under a common heading.</p>
    </section>

    <section>
      <h2>Why Use Sections?</h2>
      <p>Sections improve document structure and accessibility.</p>
    </section>
  </article>
</body>
</html>
```

**Expected output**: A top-level article heading “Guide to HTML Sections”, followed by two subsections, each with an `h2` heading and a paragraph.

**Why this output occurs**: The `<article>` is a sectioning element containing an `<h1>`. Each `<section>` creates a subsection with an `<h2>` heading. The heading levels reflect the hierarchy: `h1` for the article, `h2` for its sections.

---

#### Real-World Cases

1. **Blog post pages**: `<article>` wraps the post, `<nav>` provides site navigation, and `<aside>` holds related links or author information.
2. **Documentation sites**: `<section>` elements divide long reference pages into logical chunks, each with its own heading.
3. **News websites**: Multiple `<article>` elements appear on a single page, each representing a separate news story.

---

#### References

- MDN Web Docs — Sectioning content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#sectioning_content
- WHATWG — Sectioning content — https://html.spec.whatwg.org/multipage/dom.html#sectioning-content-2
- W3C WAI — Page Structure Tutorial — https://www.w3.org/WAI/tutorials/page-structure/

---

### 3.4 Heading Content

#### Definitions

**Core Definition**
Heading content defines the title of a section, whether that section is marked by an explicit sectioning element or implicitly defined by the heading itself. The heading elements are `<h1>` through `<h6>`.

**Technical Definition**
Heading content is a subset of flow content consisting of the six HTML heading elements. `<h1>` has the highest rank and `<h6>` the lowest. Heading levels create a hierarchical outline of the document. The `<hgroup>` element, which groups a heading with subheadings, is also related but is not itself heading content in the narrow sense.

**Beginner-Friendly Explanation**
Heading elements are the titles and subtitles of a page. `<h1>` is the main title, `<h2>` is a major section title, `<h3>` is a sub-section title, and so on down to `<h6>`. They create a table of contents that helps readers and screen readers navigate the page.

---

#### Purposes

- **To define the title of a section**: each heading names the content that follows.
- **To create a hierarchical document outline**: nested heading levels reflect nested content structure.
- **To support screen-reader navigation**: many assistive technologies allow users to jump between headings.
- **To improve SEO**: search engines use headings to understand page structure and topic hierarchy.

---

#### Syntax Rules and Structure

**General syntax**

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
| `<h4>` | Fourth | Sub-subsection heading. |
| `<h5>` | Fifth | Deep subsection heading. |
| `<h6>` | Lowest | Deepest practical heading level. |

**Heading elements**: `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`.

#### Syntax Rules

1. **Do not skip heading levels**: start with `<h1>`, then use `<h2>`, then `<h3>`, and so on. Skipping levels (e.g. `<h1>` directly to `<h3>`) confuses screen-reader users.
2. **Prefer one `<h1>` per page**. Multiple `<h1>` elements in nested sectioning elements were allowed in older standards but are now non-conforming and never represented best practice.
3. Headings must contain phrasing content; they cannot contain block-level elements.
4. Do not choose a heading level for its default font size; use CSS for visual styling.

#### Constraints and Limitations

- Heading elements cannot be nested inside other headings.
- A heading without content is invalid and unhelpful.
- Using headings purely for visual effects (e.g. `<h3>` for small text) damages document structure and accessibility.
- The `<hgroup>` element can group a heading with a secondary heading, but it does not change the heading hierarchy.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: A Correctly Nested Heading Hierarchy

**Setup guide**: Save as `headings.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Heading Hierarchy</title>
</head>
<body>
  <!-- Main page title -->
  <h1>Web Development Guide</h1>

  <!-- Major section -->
  <h2>HTML Fundamentals</h2>
  <p>HTML provides the structure of web pages.</p>

  <!-- Subsection of HTML Fundamentals -->
  <h3>Elements</h3>
  <p>Elements are the building blocks of HTML.</p>

  <h3>Attributes</h3>
  <p>Attributes modify element behaviour.</p>

  <!-- Another major section -->
  <h2>CSS Fundamentals</h2>
  <p>CSS controls the presentation of web pages.</p>
</body>
</html>
```

**Expected output**: A page with “Web Development Guide” as the largest heading, “HTML Fundamentals” and “CSS Fundamentals” as second-level headings, and “Elements” and “Attributes” as third-level headings nested under “HTML Fundamentals”.

**Why this output occurs**: The heading levels form a logical hierarchy: `h1` → `h2` → `h3` → `h3` → `h2`. No levels are skipped. Screen readers can present this hierarchy as an outline, and the visual rendering follows the browser’s default heading styles.

---

##### Example 2: Incorrect Heading Skipping (Anti-Pattern)

**Setup guide**: Save as `heading-skip.html` and compare with the previous example.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Heading Skipping — Don't Do This</title>
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

**Why this output occurs**: The heading hierarchy is broken. Assistive technologies that generate a heading outline expect consecutive levels. Skipping from `h1` to `h3` violates the recommended nesting practice and accessibility guidance.

---

#### Real-World Cases

1. **Documentation sites**: Long technical documents use `h1` for the page title, `h2` for major topics, and `h3` for sub-topics, creating a navigable table of contents.
2. **Blog posts**: The post title is typically the page’s `h1`; section headings within the post use `h2` and `h3`.
3. **E-commerce category pages**: `h1` names the category, `h2` names product groups, and `h3` names individual products or sub-groups.

---

#### References

- MDN Web Docs — Heading content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#heading_content
- MDN Web Docs — `<h1>`–`<h6>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements
- W3C WAI — Headings Tutorial — https://www.w3.org/WAI/tutorials/page-structure/headings/

---

### 3.5 Phrasing Content

#### Definitions

**Core Definition**
Phrasing content is a subset of flow content that refers to the text and the markup within a document. Sequences of phrasing content make up paragraphs.

**Technical Definition**
Phrasing content is the category of elements and text nodes that can appear inside phrasing-level containers such as `<p>`, `<span>`, and `<em>`. It includes inline elements and text nodes. Some elements belong to the phrasing content category only under specific conditions.

**Beginner-Friendly Explanation**
Phrasing content is the “text-level” content of a page: words, links, emphasis, code snippets, and other small pieces that make up sentences and paragraphs.

---

#### Purposes

- **To mark up text semantically**: emphasis, importance, definitions, abbreviations, citations.
- **To create inline structures** within paragraphs and headings.
- **To define the allowed children of phrasing-only containers** such as `<p>` and `<span>`.
- **To preserve text flow** while adding meaning and interactivity.

---

#### Syntax Rules and Structure

**General syntax**

```
<phrasing-container>
    text and phrasing elements
</phrasing-container>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<phrasing-container>` | An element whose content model is phrasing content (e.g. `<p>`, `<span>`, `<em>`, `<strong>`). |
| Phrasing content | Text nodes and inline elements belonging to the phrasing content category. |
| `</phrasing-container>` | The end tag. |

**Phrasing elements (representative list)**: `<a>` (if it contains only phrasing content), `<abbr>`, `<audio>`, `<b>`, `<bdi>`, `<bdo>`, `<br>`, `<button>`, `<canvas>`, `<cite>`, `<code>`, `<data>`, `<datalist>`, `<del>` (if phrasing only), `<dfn>`, `<em>`, `<i>`, `<iframe>`, `<img>`, `<input>`, `<ins>` (if phrasing only), `<kbd>`, `<label>`, `<map>` (if phrasing only), `<mark>`, `<meter>`, `<noscript>`, `<object>`, `<output>`, `<picture>`, `<progress>`, `<q>`, `<ruby>`, `<s>`, `<samp>`, `<script>`, `<select>`, `<small>`, `<span>`, `<strong>`, `<sub>`, `<sup>`, `<textarea>`, `<time>`, `<u>`, `<var>`, `<video>`, `<wbr>`.

#### Syntax Rules

1. Phrasing content may contain only other phrasing content and text.
2. Block-level elements must not appear inside phrasing-only containers.
3. Some elements (e.g. `<a>`, `<del>`, `<ins>`) are phrasing content only when their own content is exclusively phrasing content.
4. `<script>` and `<template>` are script-supporting elements that can appear wherever phrasing content is expected.

#### Constraints and Limitations

- `<p>` is a phrasing-content container; placing a `<div>` inside a `<p>` is invalid.
- Interactive phrasing content (e.g. `<button>`, `<a href>`) must not be nested inside other interactive content.
- Phrasing content alone cannot create block-level layout structures.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Phrasing Content Inside a Paragraph

**Setup guide**: Save as `phrasing.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Phrasing Content</title>
</head>
<body>
  <p>
    <!-- All of the following are phrasing content -->
    The <code>console.log()</code> function outputs
    <strong>important</strong> messages.
    Use <kbd>Ctrl</kbd> + <kbd>C</kbd> to copy.
    Visit <a href="https://example.com">the documentation</a> for more.
  </p>
</body>
</html>
```

**Expected output**: A single paragraph in which “console.log()” is rendered in monospace, “important” is bold, “Ctrl” and “C” are styled as keyboard input, and “the documentation” is a hyperlink — all flowing on the same line(s) without block-level breaks.

**Why this output occurs**: `<code>`, `<strong>`, `<kbd>`, and `<a>` are all phrasing content. They are valid children of `<p>`, which accepts phrasing content only.

---

##### Example 2: Invalid Block Content Inside a Phrasing Container

**Setup guide**: Save as `invalid-phrasing.html` and inspect the parsed DOM.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Invalid Phrasing Content</title>
</head>
<body>
  <p>
    This is a paragraph.
    <!-- INVALID: <div> is flow content, not phrasing content -->
    <div>This block-level div is inside a paragraph.</div>
  </p>
</body>
</html>
```

**Expected output**: Browsers will render the text, but the DOM inspector reveals that the parser has closed the `<p>` before the `<div>` and created a separate block, because a `<div>` is not allowed inside a `<p>`.

**Why this output occurs**: `<p>` has a phrasing-content content model. `<div>` is flow content, not phrasing content, so the markup is non-conforming. The browser’s error-recovery algorithm restructures the DOM to produce a renderable tree.

---

#### Real-World Cases

1. **Article text with inline links**: Every hyperlink inside body prose is phrasing content (`<a>`), allowing readers to move through the text without breaking paragraph structure.
2. **Technical documentation with inline code**: `<code>` elements mark variable names, function calls, and file paths within sentences.
3. **Legal and editorial content**: `<del>` and `<ins>` mark deleted and inserted text within paragraphs, preserving the flow of the sentence.

---

#### References

- MDN Web Docs — Phrasing content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#phrasing_content
- WHATWG — Phrasing content — https://html.spec.whatwg.org/multipage/dom.html#phrasing-content-2
- W3C — HTML 5.2: Phrasing content — https://www.w3.org/TR/html52/dom.html#phrasing-content

---

### 3.6 Embedded Content

#### Definitions

**Core Definition**
Embedded content is content that imports another resource or inserts content from another markup language or namespace into the document.

**Technical Definition**
Embedded content elements are those that bring external resources into the HTML document — images, video, audio, embedded web pages, and similar media. They are a subset of flow content and often also belong to the phrasing content and/or interactive content categories.

**Beginner-Friendly Explanation**
Embedded content is content that comes from outside the HTML file itself: a picture, a video, a map, or another web page shown inside a frame. The element acts as a window through which that external resource is displayed.

---

#### Purposes

- **To display images** (`<img>`).
- **To play audio and video** (`<audio>`, `<video>`).
- **To embed another web page** within the current page (`<iframe>`).
- **To integrate external applications or content** (`<embed>`, `<object>`).
- **To provide fallback content and alternatives** for embedded resources.

---

#### Syntax Rules and Structure

**General syntax**

```
<embedded-element src="resource-url" ...>
    fallback content (optional)
</embedded-element>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<embedded-element>` | An element such as `<img>`, `<video>`, `<audio>`, `<iframe>`, `<embed>`, `<object>`, `<picture>`, `<canvas>`. |
| `src` attribute | Points to the external resource (not used by `<canvas>` or `<picture>` in the same way). |
| Fallback content | Text or elements displayed if the resource cannot be loaded (supported by `<video>`, `<audio>`, `<object>`, `<iframe>`). |
| `</embedded-element>` | The end tag (omitted for void elements such as `<img>` and `<embed>`). |

**Embedded content elements**: `<audio>`, `<canvas>`, `<embed>`, `<iframe>`, `<img>`, `<math>`, `<object>`, `<picture>`, `<svg>`, `<video>`.

#### Syntax Rules

1. `<img>` must have a `src` and an `alt` attribute for accessibility.
2. `<iframe>` should include a `title` attribute to describe its content to assistive technologies.
3. `<video>` and `<audio>` should provide fallback text for browsers that do not support them.
4. Embedded content may be interactive (e.g. `<video controls>`, `<iframe>`) and therefore must not be nested inside other interactive elements.

#### Constraints and Limitations

- Embedding external resources can introduce security and privacy risks; use `sandbox` on `<iframe>` and avoid untrusted sources.
- Embedded content may not be palpable if it is hidden or fails to load.
- `<embed>` is a void element and cannot contain fallback content; `<object>` is preferred when fallback content is needed.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Image and Video with Fallback

**Setup guide**: Save as `embedded.html`. Provide a video file named `sample.mp4` in the same directory.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Embedded Content</title>
</head>
<body>
  <!-- Image: void element with required alt text -->
  <img src="landscape.jpg" alt="A mountain landscape at sunset" width="600">

  <!-- Video: embedded content with fallback text -->
  <video controls width="600" poster="poster.jpg">
    <source src="sample.mp4" type="video/mp4">
    <!-- Fallback content for unsupported browsers -->
    Your browser does not support the video element.
    <a href="sample.mp4">Download the video</a> instead.
  </video>
</body>
</html>
```

**Expected output**: A landscape image followed by a video player with controls. If the browser cannot play the video, the fallback text and download link are displayed.

**Why this output occurs**: `<img>` is a void embedded element that renders the image resource. `<video>` is an embedded element that creates a media player; the `<source>` child specifies the video file, and the text content provides fallback for unsupported environments.

---

##### Example 2: Embedded Web Page with `<iframe>`

**Setup guide**: Save as `iframe.html`. Replace the `src` URL with a page that allows embedding.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Embedded Iframe</title>
</head>
<body>
  <!-- <iframe> embeds another HTML page inside this document -->
  <iframe
    src="https://example.com/embedded-page"
    title="Embedded example page"
    width="800"
    height="400"
    sandbox="allow-scripts allow-same-origin"
    loading="lazy">
    <!-- Fallback content for browsers that do not support iframes -->
    <p>Your browser does not support iframes. <a href="https://example.com/embedded-page">Open the page directly</a>.</p>
  </iframe>
</body>
</html>
```

**Expected output**: A rectangular frame displaying the content of `https://example.com/embedded-page`. If the browser does not support iframes or the page cannot be embedded, the fallback paragraph is shown.

**Why this output occurs**: `<iframe>` creates a nested browsing context. The `sandbox` attribute restricts the embedded page’s capabilities for security. The `title` attribute provides an accessible name, and the fallback content is displayed only when the iframe cannot be rendered.

---

#### Real-World Cases

1. **Video platforms**: YouTube and Vimeo provide `<iframe>` embed codes that allow their players to appear inside other websites.
2. **Maps**: Google Maps and OpenStreetMap provide `<iframe>` embed snippets for displaying interactive maps.
3. **Podcasts and music**: `<audio>` elements embed playable audio files directly into blog posts and documentation pages.

---

#### References

- MDN Web Docs — Embedded content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#embedded_content
- MDN Web Docs — From object to iframe — other embedding technologies — https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Other_embedding_technologies
- WHATWG — Embedded content — https://html.spec.whatwg.org/multipage/dom.html#embedded-content-2

---

### 3.7 Interactive Content

#### Definitions

**Core Definition**
Interactive content is content specifically designed for user interaction — elements that users can click, focus, activate, or otherwise manipulate.

**Technical Definition**
Interactive content elements are those that belong to the interactive content category as defined by the HTML specification. Some elements belong to this category only when certain attributes are present (e.g. `<a>` with `href`, `<audio>` with `controls`, `<input>` with a non-hidden type).

**Beginner-Friendly Explanation**
Interactive content is anything on a page that responds to what you do: links you can click, buttons you can press, form fields you can type into, and videos with controls you can use.

---

#### Purposes

- **To enable user actions**: navigation links, form submission, media playback controls.
- **To collect user input**: text fields, checkboxes, radio buttons, dropdowns.
- **To provide interactive widgets**: disclosure panels, embedded applications.
- **To support keyboard and assistive-technology interaction**: interactive elements are focusable and expose accessible roles.

---

#### Syntax Rules and Structure

**General syntax (button example)**

```html
<button type="button" onclick="handleClick()">
    Click me
</button>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<button>` | An interactive element that users can activate. |
| `type` attribute | Specifies the button’s behaviour (`button`, `submit`, or `reset`). |
| Content | The button’s label (phrasing content). |
| Event handling | JavaScript can respond to click, focus, and other events. |

**Interactive content elements (representative list)**: `<a>` (with `href`), `<audio>` (with `controls`), `<button>`, `<details>`, `<embed>`, `<iframe>`, `<img>` (with `usemap`), `<input>` (non-hidden), `<label>`, `<select>`, `<textarea>`, `<video>` (with `controls`).

#### Syntax Rules

1. Interactive content must not be nested inside other interactive content (an `<a>` must not contain a `<button>`).
2. Interactive elements should be keyboard-accessible and have a visible focus indicator.
3. Form controls should be associated with `<label>` elements for accessibility.
4. The `disabled` attribute can make an interactive element non-interactive.

#### Constraints and Limitations

- Nesting interactive elements creates ambiguous activation behaviour and is non-conforming.
- Custom interactive widgets built from non-interactive elements (e.g. `<div>` with `onclick`) lack built-in keyboard support and ARIA semantics unless carefully implemented.
- Embedded interactive content (e.g. `<iframe>`, `<video controls>`) can capture keyboard events, which may conflict with page-level interactions.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: A Button and a Link — Both Interactive

**Setup guide**: Save as `interactive.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Interactive Content</title>
</head>
<body>
  <!-- <a> with href is interactive content -->
  <p>
    Visit the
    <a href="https://developer.mozilla.org/">MDN Web Docs</a>
    for reference material.
  </p>

  <!-- <button> is interactive content -->
  <button type="button" onclick="alert('Button clicked!')">
    Click Me
  </button>

  <!-- <details> and <summary> create a disclosure widget -->
  <details>
    <summary>Show more information</summary>
    <p>This content is revealed when the summary is activated.</p>
  </details>
</body>
</html>
```

**Expected output**: A paragraph containing a clickable link, a button that shows an alert when clicked, and a disclosure widget that expands to reveal a paragraph when the summary is clicked.

**Why this output occurs**: `<a href>`, `<button>`, and `<details>` are interactive elements. They are focusable, respond to user input, and expose interactive semantics to assistive technologies.

---

##### Example 2: Invalid Nesting of Interactive Elements

**Setup guide**: Save as `invalid-interactive.html` and inspect the DOM.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Invalid Interactive Nesting</title>
</head>
<body>
  <!-- INVALID: <a> must not contain another interactive element such as <button> -->
  <a href="https://example.com">
    <button type="button">Nested Button</button>
  </a>
</body>
</html>
```

**Expected output**: The browser may still render the link and button, but the markup is non-conforming and may produce unpredictable activation behaviour. Validators will report an error.

**Why this output occurs**: The `<a>` element with `href` is interactive content. Its content model forbids interactive descendants. Nesting a `<button>` inside it creates conflicting activation targets and violates the specification.

---

#### Real-World Cases

1. **Search forms**: A text `<input>` with a `<button type="submit">` creates a fully keyboard-accessible search interface.
2. **FAQ accordions**: Native `<details>`/`<summary>` elements provide disclosure widgets without JavaScript.
3. **Media players**: `<video controls>` and `<audio controls>` provide interactive playback controls that are built into the browser.

---

#### References

- MDN Web Docs — Interactive content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#interactive_content
- WHATWG — Interactive elements — https://html.spec.whatwg.org/multipage/interactive-elements.html
- W3C — HTML 5.2: Interactive content — https://www.w3.org/TR/html52/dom.html#interactive-content

---

### 3.8 Palpable Content

#### Definitions

**Core Definition**
Palpable content is content that is neither empty nor hidden; it is rendered and substantive.

**Technical Definition**
Palpable content is not used to define content models directly. Instead, it defines a general conformance rule: elements whose content model allows any flow content or phrasing content **should** have at least one descendant node that is palpable content and that does not have the `hidden` attribute specified.

**Beginner-Friendly Explanation**
Palpable content is content you can actually perceive — text you can read, an image you can see, a sound you can hear, or a control you can interact with. A page section that is completely empty or invisible is not palpable.

---

#### Purposes

- **To prevent empty or meaningless elements** that serve no user-facing purpose.
- **To ensure that containers have substantive content** — a `<section>` with no text or media is not palpable.
- **To guide authors toward content that benefits users** rather than purely structural or script-only containers.
- **To support accessibility** by discouraging hidden or empty landmarks.

---

#### Syntax Rules and Structure

There is no dedicated “palpable element” syntax. Palpability is a property of an element’s content. The rule applies as follows:

```
<flow-or-phrasing-container>
    <!-- at least one palpable descendant should be present -->
    visible text, image, audio with controls, video, canvas, or interactive control
</flow-or-phrasing-container>
```

**Component breakdown**

| Component | Description |
|---|---|
| Flow/phrasing container | An element such as `<div>`, `<section>`, `<p>`, `<li>` whose content model allows flow or phrasing content. |
| Palpable descendant | Non-empty text, an `<img>` with `alt`, an `<audio>` with `controls`, a `<video>`, a `<canvas>`, an interactive form control, etc. |
| `hidden` attribute | An element with `hidden` is not palpable even if it contains text. |

**Palpable elements (representative list)**: `<a>`, `<abbr>`, `<address>`, `<article>`, `<aside>`, `<audio>` (with `controls`), `<b>`, `<blockquote>`, `<button>`, `<canvas>`, `<code>`, `<details>`, `<div>` (if it has palpable content), `<dl>` (with name-value groups), `<em>`, `<figure>`, `<footer>`, `<form>`, `<h1>`–`<h6>`, `<header>`, `<img>`, `<input>` (non-hidden), `<li>`, `<ol>` (with `<li>` children), `<p>`, `<pre>`, `<section>`, `<select>`, `<strong>`, `<table>`, `<textarea>`, `<ul>` (with `<li>` children), `<video>`.

#### Syntax Rules

1. The palpable-content rule uses **“should”**, not “must”; it is a best-practice conformance guideline rather than a hard parsing rule.
2. Elements with the `hidden` attribute are not palpable even if their DOM subtree contains text.
3. An element containing only whitespace is not palpable.
4. An `<img>` with an empty `alt=""` is still palpable if it is rendered; the rule concerns visibility and substance, not the length of the alternative text.

#### Constraints and Limitations

- Purely structural `<div>` wrappers with no content are non-palpable and should be avoided.
- Script-only elements such as `<script>` and `<template>` are not palpable; they are script-supporting content.
- The palpable-content guideline does not apply to metadata content or elements whose content model does not allow flow or phrasing content.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Palpable vs Non-Palpable Containers

**Setup guide**: Save as `palpable.html`. Use the browser inspector to compare the two sections.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Palpable Content</title>
  <style>
    section { border: 1px dashed #999; padding: 8px; margin: 8px 0; min-height: 20px; }
  </style>
</head>
<body>
  <!-- PALPABLE: contains visible text -->
  <section>
    <p>This section contains palpable text content.</p>
  </section>

  <!-- NON-PALPABLE: empty section, no visible content -->
  <section></section>

  <!-- NON-PALPABLE: content is hidden with the hidden attribute -->
  <section hidden>
    <p>This text is hidden and therefore not palpable.</p>
  </section>
</body>
</html>
```

**Expected output**: The first section displays a dashed border containing visible text. The second section displays only an empty dashed border. The third section is completely hidden.

**Why this output occurs**: The first section contains a text node, which is palpable. The second section has no content, so it fails the palpable-content guideline. The third section contains text, but the `hidden` attribute removes it from rendering, making it non-palpable.

---

##### Example 2: An Image Provides Palpability

**Setup guide**: Save as `palpable-image.html`. Provide an image file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Palpable Image</title>
</head>
<body>
  <div>
    <!-- The <img> element provides palpable content even without text -->
    <img src="chart.png" alt="Sales chart for Q1 2025" width="400">
  </div>
</body>
</html>
```

**Expected output**: A visible chart image inside the `div`. The `div` is palpable because it contains a rendered image.

**Why this output occurs**: A rendered `<img>` is palpable content. Even though the `div` contains no text, the image itself is substantive and visible, satisfying the palpable-content guideline for flow-content containers.

---

#### Real-World Cases

1. **Landmark regions**: A `<nav>` with no links or a `<section>` with no heading or text is non-palpable and unhelpful; authors should ensure such regions contain substantive content.
2. **Card components**: A product card must contain an image, title, price, or button — at least one palpable descendant — to be meaningful.
3. **Hidden accessibility text**: Text hidden with `hidden` is not palpable; if visually hidden text is needed for screen readers, CSS clipping techniques (not the `hidden` attribute) are used instead.

---

#### References

- MDN Web Docs — Palpable content — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#palpable_content
- WHATWG — Palpable content — https://html.spec.whatwg.org/multipage/dom.html#palpable-content-2
- W3C — HTML 5.2: Palpable content — https://www.w3.org/TR/html52/dom.html#palpable-content

---

### 3.9 Script-Supporting Content

#### Definitions

**Core Definition**
Script-supporting elements are elements that do not directly contribute to a document’s rendered output. Instead, they either contain script code or specify data that will be used by scripts.

**Technical Definition**
There are exactly two script-supporting elements in HTML: `<script>` and `<template>`. They can appear in contexts where metadata content, phrasing content, or script-supporting elements are expected. `<script>` contains executable code or references an external script; `<template>` contains markup that is not rendered until instantiated by script.

**Beginner-Friendly Explanation**
Script-supporting elements are “behind-the-scenes” elements. They do not show up on the page themselves, but they provide the JavaScript or reusable markup that makes the page interactive.

---

#### Purposes

- **To embed or reference executable scripts** (`<script>`).
- **To define reusable HTML fragments** that scripts can clone and insert (`<template>`).
- **To keep markup and behaviour modular**: templates allow HTML to be defined once and instantiated multiple times.
- **To support dynamic content generation** without string concatenation in JavaScript.

---

#### Syntax Rules and Structure

**General syntax**

```html
<!-- Script element with external source -->
<script src="app.js" defer></script>

<!-- Script element with inline code -->
<script>
    console.log("Hello from inline script");
</script>

<!-- Template element with inert content -->
<template id="card-template">
    <div class="card">
        <h2 class="card-title"></h2>
        <p class="card-body"></p>
    </div>
</template>
```

**Component breakdown**

| Element | Description |
|---|---|
| `<script>` | Contains or references JavaScript. May have `src`, `defer`, `async`, `type`, and other attributes. |
| `<template>` | Contains markup that is parsed into a separate `DocumentFragment` and is not rendered until cloned and inserted by script. |
| `id` attribute | Commonly used on `<template>` to allow scripts to select it. |
| `defer` / `async` | Control when external scripts execute relative to HTML parsing. |

**Script-supporting elements**: `<script>`, `<template>`.

#### Syntax Rules

1. `<script>` and `<template>` may appear almost anywhere — in `<head>`, in `<body>`, inside lists, tables, and other containers — because nearly all elements can contain script-supporting content.
2. `<template>` content is inert: scripts inside a template do not execute, and images do not load, until the template content is cloned into the document.
3. `<script>` content is treated as raw text; the parser does not interpret `<` and `>` inside it as markup (except for the closing `</script>` sequence).
4. External scripts should use `defer` or `async` to avoid blocking HTML parsing.

#### Constraints and Limitations

- `<template>` content is not rendered and is not part of the main document tree until instantiated.
- Inline `<script>` content must not contain the literal string `</script>` without escaping.
- Script-supporting elements do not contribute to palpable content and therefore do not satisfy the palpable-content guideline.
- `async` scripts may execute in any order, which can cause race conditions if scripts depend on one another.

---

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: `<script>` with `defer` and `<template>` Instantiation

**Setup guide**: Save as `script-supporting.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Script-Supporting Content</title>
</head>
<body>
  <!-- Container where template content will be inserted -->
  <div id="card-container"></div>

  <!-- Template: inert until cloned by script -->
  <template id="card-template">
    <div class="card">
      <h2 class="card-title"></h2>
      <p class="card-body"></p>
    </div>
  </template>

  <!-- Script: deferred until parsing is complete -->
  <script defer>
    // Select the template and the container
    const template = document.getElementById('card-template');
    const container = document.getElementById('card-container');

    // Clone the template content
    const clone = template.content.cloneNode(true);

    // Populate the clone with data
    clone.querySelector('.card-title').textContent = 'Dynamic Card';
    clone.querySelector('.card-body').textContent = 'This card was created from a template.';

    // Insert the populated clone into the document
    container.appendChild(clone);
  </script>
</body>
</html>
```

**Expected output**: A card with the heading “Dynamic Card” and the body text “This card was created from a template.” appears inside the `card-container` div after the page loads.

**Why this output occurs**: The `<template>` element holds inert markup that is not rendered initially. The deferred `<script>` waits until HTML parsing is complete, then clones the template’s `content` fragment, populates it, and appends it to the visible document. The `<script>` itself does not render anything, confirming its script-supporting role.

---

##### Example 2: `<script>` in the `<head>` with `defer`

**Setup guide**: Save as `defer-script.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Deferred Script</title>
  <!-- External script deferred until DOM is parsed -->
  <script src="analytics.js" defer></script>
</head>
<body>
  <h1>Page Title</h1>
  <p>The deferred script will run after this HTML is parsed.</p>
</body>
</html>
```

**Expected output**: The page renders immediately. The script `analytics.js` executes after the HTML has been fully parsed, so it can safely access elements in the body.

**Why this output occurs**: The `defer` attribute tells the browser to download the script in parallel with HTML parsing but execute it only after parsing is complete. This makes the script’s execution dependent on the DOM being available, which is ideal for scripts that manipulate page elements.

---

#### Real-World Cases

1. **Web components**: `<template>` is widely used to define the shadow DOM markup for custom elements.
2. **Client-side rendering**: Frameworks and vanilla JavaScript use `<template>` to define reusable UI fragments that are cloned for each data item.
3. **Analytics and tracking**: `<script>` elements in the `<head>` or `<body>` load analytics libraries without producing visible page content.

---

#### References

- MDN Web Docs — Script-supporting element — https://developer.mozilla.org/en-US/docs/Glossary/Script-supporting_element
- MDN Web Docs — Script-supporting elements (content category) — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories#script-supporting_elements
- WHATWG — The script element — https://html.spec.whatwg.org/multipage/scripting.html#the-script-element
- WHATWG — The template element — https://html.spec.whatwg.org/multipage/scripting.html#the-template-element

---

## 4. Element Nesting

### Definitions

**Core Definition**
Element nesting is the placement of one element inside another so that the inner element is a child of the outer element in the document tree.

**Technical Definition**
Nesting is constrained by each element’s content model. An element may contain another element only if the outer element’s content model permits the inner element’s content category. Nesting must be strictly hierarchical: elements must open and close in the correct order, with no overlapping boundaries. The HTML parser applies error-recovery rules when nesting is invalid.

**Beginner-Friendly Explanation**
Nesting means putting one container inside another. Just as a box can contain smaller boxes, an HTML element can contain other elements — but only if the rules allow it. For example, a paragraph can contain a link, but a paragraph cannot contain a whole new section.

---

### Purposes

- **To build document structure**: nesting creates the hierarchical tree that represents page content.
- **To express relationships**: parent-child nesting conveys containment and association.
- **To apply styles and scripts hierarchically**: CSS inheritance and DOM traversal depend on the nesting tree.
- **To enforce semantic correctness**: content-model rules prevent nonsensical structures such as a `<div>` inside a `<p>`.

---

### Syntax Rules and Structure

**General syntax**

```
<parent-element>
    <child-element>
        content
    </child-element>
</parent-element>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<parent-element>` | The outer element; its content model determines which children are allowed. |
| `<child-element>` | The inner element; must belong to a content category permitted by the parent’s content model. |
| `content` | The child’s own content. |
| Closing order | The child must close before the parent: `</child-element>` then `</parent-element>`. |

**Nesting rules at a glance**

| Rule | Explanation |
|---|---|
| **Proper nesting** | Elements must close in reverse order of opening: `<div><p>text</p></div>` is valid; `<div><p>text</div></p>` is invalid. |
| **Content-model compliance** | A `<p>` may contain only phrasing content; a `<div>` may contain flow content. |
| **No interactive nesting** | Interactive elements must not contain other interactive elements. |
| **No block inside phrasing** | Block-level elements must not be placed inside phrasing-only containers. |
| **Void elements have no children** | `<img>`, `<br>`, `<hr>`, `<input>`, etc. cannot contain anything. |

#### Syntax Rules

1. **Strict hierarchy**: Every start tag must have a matching end tag in the correct order (except void elements and optional-tag cases).
2. **Content-model compliance**: The child element must be permitted by the parent’s content model.
3. **No overlapping**: `<em><strong>text</em></strong>` is invalid because the tags overlap.
4. **Context-sensitive exceptions**: Some elements have special nesting rules — for example, `<li>` must be a child of `<ul>`, `<ol>`, or `<menu>`; `<td>` must be inside `<tr>` inside `<table>`.

#### Constraints and Limitations

- Browsers apply error recovery to invalid nesting, but the resulting DOM may differ from the author’s intent.
- Deeply nested structures can be difficult to maintain and may affect rendering performance.
- Some elements (e.g. `<p>`) have optional end tags, which can make nesting boundaries ambiguous in source code even though the parser resolves them.
- Custom elements must follow the same content-model principles if they are to be interoperable and accessible.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Valid Nested Structure

**Setup guide**: Save as `valid-nesting.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Valid Nesting</title>
</head>
<body>
  <!-- <div> accepts flow content -->
  <div>
    <!-- <h2> is flow content, allowed inside <div> -->
    <h2>Section Heading</h2>

    <!-- <p> is flow content, allowed inside <div> -->
    <p>
      This paragraph contains
      <!-- <em> is phrasing content, allowed inside <p> -->
      <em>emphasised text</em>
      and a
      <!-- <a> with href is phrasing and interactive content, allowed inside <p> -->
      <a href="#">link</a>.
    </p>
  </div>
</body>
</html>
```

**Expected output**: A section heading followed by a paragraph containing italicised text and a hyperlink, all inside the `div`.

**Why this output occurs**: Every nesting decision respects the content models: `<div>` accepts flow content (`<h2>`, `<p>`), and `<p>` accepts phrasing content (`<em>`, `<a>`). The tags close in the correct order, producing a valid tree.

---

#### Example 2: Invalid Overlapping Nesting

**Setup guide**: Save as `invalid-nesting.html` and inspect the DOM.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Invalid Overlapping Nesting</title>
</head>
<body>
  <!-- INVALID: <em> and <strong> overlap incorrectly -->
  <p>
    <em>Emphasised <strong>and strong</em> text</strong>
  </p>
</body>
</html>
```

**Expected output**: The browser will still render the text, but the DOM will show that the parser has restructured the elements. The intended nesting is destroyed.

**Why this output occurs**: The `<em>` start tag is followed by a `<strong>` start tag, but the `</em>` end tag appears before `</strong>`. This is overlapping nesting, which violates the strict hierarchy rule. The parser applies error recovery, producing a DOM that does not match the author’s intended structure.

---

### Real-World Cases

1. **Navigation lists**: `<nav><ul><li><a href="…">Home</a></li></ul></nav>` demonstrates correct nesting: `<li>` inside `<ul>`, `<a>` inside `<li>`, and `<ul>` inside `<nav>`.
2. **Data tables**: `<table><thead><tr><th>Name</th></tr></thead><tbody><tr><td>Alice</td></tr></tbody></table>` follows the required table nesting hierarchy.
3. **Article structure**: `<article><header><h1>Title</h1></header><p>Body text…</p><footer>Author</footer></article>` nests sectioning and heading content correctly.

---

### References

- WHATWG — Content models — https://html.spec.whatwg.org/multipage/dom.html#content-models
- MDN Web Docs — Content categories — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories
- W3C — HTML 5.2: Content models — https://www.w3.org/TR/html52/dom.html#content-models
- W3C — Nesting rules (HTML 4.01) — https://www.w3.org/TR/html401/struct/global.html#h-7.5.1

---

## 5. Element Content Model

### Definitions

**Core Definition**
An element’s content model is the normative description of what content is allowed inside that element — which child elements, text nodes, and other nodes it may contain.

**Technical Definition**
The HTML specification defines a content model for every element. The content model specifies the element’s structure: what child nodes are permitted, in what order, and under what conditions. Content models are expressed in terms of content categories (e.g. “phrasing content”, “flow content”) and sometimes with additional restrictions. Authors must only place elements inside an element if that element’s content model allows them.

**Beginner-Friendly Explanation**
A content model is an element’s “guest list”. It says what kinds of things are allowed inside that element. For example, a paragraph’s guest list is “text and inline elements only” — a whole `<div>` is not on the list and is therefore not allowed inside.

---

### Purposes

- **To define valid document structure**: content models specify exactly what may appear inside each element.
- **To enable validation**: conformance checkers use content models to detect invalid markup.
- **To guide authoring decisions**: knowing an element’s content model helps authors choose appropriate children.
- **To support parsing and rendering**: browsers rely on content models (and error-recovery rules) to build the DOM.
- **To preserve semantics and accessibility**: correct content models prevent nonsensical structures that confuse assistive technologies.

---

### Syntax Rules and Structure

**General concept**

```
<element>
    <!-- children must conform to the element's content model -->
</element>
```

**Common content-model patterns**

| Content model | Meaning | Example elements |
|---|---|---|
| **Phrasing content** | Text and inline elements only | `<p>`, `<span>`, `<em>`, `<strong>` |
| **Flow content** | Block-level and inline content | `<div>`, `<section>`, `<article>`, `<body>` |
| **Sectioning content** | Sectioning elements | `<article>`, `<aside>`, `<nav>`, `<section>` |
| **Heading content** | `<h1>`–`<h6>` | Heading containers |
| **Embedded content** | Media and embedded resources | `<video>`, `<audio>`, `<iframe>` |
| **Interactive content** | User-interactive elements | `<button>`, `<input>`, `<details>` |
| **Transparent** | Content model derived from the parent | `<a>`, `<ins>`, `<del>`, `<map>` |
| **Nothing** | No children permitted | `<br>`, `<hr>`, `<img>`, `<input>` |

**Transparent content model**: Some elements, such as `<a>`, `<ins>`, `<del>`, and `<map>`, have a transparent content model. This means their allowed content is derived from their parent: if the transparent element were removed but its children remained, the document would still be conforming. In practice, a transparent element must not contain content that would be invalid in its parent’s context.

#### Syntax Rules

1. Every element has a defined content model; authors must respect it.
2. Content models are expressed in terms of content categories, often with additional conditions.
3. Some elements have **conditional** content models: for example, `<a>` is phrasing content only if it contains only phrasing content.
4. **Void elements** have a content model of “nothing”; they cannot have children or end tags.
5. **Transparent elements** inherit constraints from their parent context.

#### Constraints and Limitations

- Content models are **normative**; violating them produces non-conforming HTML even if browsers render it.
- The HTML parser’s error-recovery behaviour is not a substitute for conforming markup.
- Transparent content models can be subtle: an `<a>` inside a `<div>` may contain flow content, but an `<a>` inside a `<p>` may contain only phrasing content.
- Content models do not always enforce **semantic** suitability; an element may be technically allowed but semantically inappropriate.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Phrasing Content Model in `<p>`

**Setup guide**: Save as `content-model-p.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Content Model — Paragraph</title>
</head>
<body>
  <!-- <p> content model: phrasing content -->
  <p>
    This paragraph contains
    <strong>strong text</strong>,
    <em>emphasised text</em>, and
    <a href="#">a link</a>
    — all valid phrasing content.
  </p>
</body>
</html>
```

**Expected output**: A paragraph with bold, italic, and linked text — all on the same line flow.

**Why this output occurs**: `<p>` has a phrasing-content content model. `<strong>`, `<em>`, and `<a>` (containing phrasing content) are all phrasing content, so they are valid children.

---

#### Example 2: Transparent Content Model

**Setup guide**: Save as `transparent.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Transparent Content Model</title>
</head>
<body>
  <!-- <a> inside <div> may contain flow content because <div> allows flow content -->
  <div>
    <a href="#">
      <h2>Heading inside a link</h2>
      <p>Paragraph inside a link.</p>
    </a>
  </div>

  <!-- <a> inside <p> may contain only phrasing content because <p> allows only phrasing content -->
  <p>
    <a href="#">This link contains only text, which is phrasing content.</a>
  </p>
</body>
</html>
```

**Expected output**: The first link wraps a heading and a paragraph (rendered as a large clickable block). The second link wraps only text.

**Why this output occurs**: `<a>` has a transparent content model. In the first case, its parent is `<div>` (flow content allowed), so the `<a>` may contain flow content such as `<h2>` and `<p>`. In the second case, its parent is `<p>` (phrasing content only), so the `<a>` may contain only phrasing content. This demonstrates how a transparent content model derives constraints from the parent context.

---

### Real-World Cases

1. **Custom link cards**: When an `<a>` wraps a card containing a heading and paragraph, the transparent content model allows this only because the `<a>` is placed in a flow-content container such as `<div>` or `<article>`.
2. **Editing markup**: `<ins>` and `<del>` have transparent content models, allowing them to wrap either phrasing content inside a paragraph or flow content inside a `<div>`, depending on context.
3. **Form controls**: `<button>` has a phrasing-content content model in most contexts, preventing block-level layout inside buttons.

---

### References

- WHATWG — Content models — https://html.spec.whatwg.org/multipage/dom.html#content-models
- MDN Web Docs — Content categories — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories
- W3C — HTML 5.2: Content models — https://www.w3.org/TR/html52/dom.html#content-models
- WHATWG — Transparent content models — https://html.spec.whatwg.org/multipage/dom.html#transparent-content-models

---

## 6. Element Best Practices

### Definitions

**Core Definition**
Element best practices are authoring guidelines that go beyond mere conformance to produce HTML that is semantic, accessible, maintainable, and robust across browsers and assistive technologies.

**Technical Definition**
Best practices encompass correct element selection, valid nesting, content-model compliance, heading hierarchy, accessibility attributes, separation of concerns, and progressive enhancement. They are informed by specifications, accessibility guidelines (WCAG), and authoritative documentation such as MDN and the WHATWG standard.

**Beginner-Friendly Explanation**
Best practices are the “good habits” of writing HTML. They are not always strictly required by the browser, but following them makes your pages easier to maintain, more accessible, and more likely to work correctly everywhere.

---

### Purposes

- **To maximise accessibility** for users of screen readers, keyboard navigation, and other assistive technologies.
- **To improve maintainability** by using clear, semantic, and consistent markup.
- **To ensure cross-browser reliability** by avoiding non-conforming or implementation-dependent markup.
- **To support SEO** through proper heading hierarchy and semantic structure.
- **To future-proof documents** against changes in browsers and standards.

---

### Syntax Rules and Structure

Best practices do not have a single syntax; they are a set of principles applied across markup. The following table summarises the core rules.

| Practice | Rule |
|---|---|
| **Use semantic elements** | Prefer `<button>` over `<div onclick>`, `<nav>` over `<div class="nav">`. |
| **Do not skip heading levels** | Start with `<h1>`, then `<h2>`, `<h3>`, and so on; do not jump from `<h1>` to `<h3>`. |
| **Use one `<h1>` per page** | Multiple `<h1>` elements in nested sectioning elements are non-conforming and were never best practice. |
| **Provide alternative text** | Every `<img>` needs a meaningful `alt` attribute; decorative images use `alt=""`. |
| **Label form controls** | Associate every `<input>` with a `<label>`. |
| **Do not nest interactive elements** | An `<a>` must not contain a `<button>` or another `<a>`. |
| **Respect content models** | Do not place block-level elements inside phrasing-only containers. |
| **Use `<button>` for actions and `<a>` for navigation** | The element should match the behaviour. |
| **Prefer native HTML over ARIA** | Native elements have built-in semantics and keyboard behaviour. |
| **Separate structure, presentation, and behaviour** | Use HTML for structure, CSS for presentation, JavaScript for behaviour. |

#### Syntax Rules

1. **Semantic correctness**: Choose the element that best describes the content’s meaning, not its default appearance.
2. **Accessibility first**: Provide `alt` text, labels, captions, and accessible names for interactive elements.
3. **Valid nesting**: Ensure every element respects its content model and the document forms a proper tree.
4. **Heading discipline**: Maintain a logical, non-skipping heading hierarchy.
5. **Progressive enhancement**: Ensure core content and functionality work without CSS or JavaScript where possible.

#### Constraints and Limitations

- Best practices can conflict with legacy codebases or third-party widgets; refactoring may be required gradually.
- Native HTML elements may not cover every advanced UI pattern; ARIA is appropriate only when native semantics are insufficient.
- Semantic markup sometimes requires more elements than a purely presentational approach, trading brevity for clarity and accessibility.
- Browser support for newer semantic elements (e.g. `<dialog>`, `<details>`) varies, though support is now broad.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Semantic Button vs Non-Semantic Clickable Div

**Setup guide**: Save as `semantic-button.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Semantic Button</title>
  <style>
    button { padding: 8px 16px; font-size: 16px; cursor: pointer; }
  </style>
</head>
<body>
  <!-- GOOD: native button is focusable, keyboard-activatable, and exposes button semantics -->
  <button type="button" onclick="alert('Saved!')">Save</button>

  <!-- BAD (commented for comparison): a div with onclick lacks keyboard support and semantics -->
  <!--
  <div onclick="alert('Saved!')" style="padding: 8px 16px; background: #eee; cursor: pointer;">
    Save
  </div>
  -->
</body>
</html>
```

**Expected output**: A native button labelled “Save” that can be activated by mouse click or by keyboard (Tab to focus, Enter or Space to activate).

**Why this output occurs**: The `<button>` element has built-in keyboard accessibility, focus management, and an accessible role. A `<div>` with `onclick` would require additional `tabindex`, `role`, and keyboard event handling to approximate the same behaviour, and is therefore a poorer practice.

---

#### Example 2: Correct Heading Hierarchy with Accessible Images

**Setup guide**: Save as `best-practice-structure.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Structure</title>
</head>
<body>
  <!-- One h1 per page for the main title -->
  <h1>Company Annual Report</h1>

  <!-- h2 for major sections -->
  <h2>Financial Highlights</h2>
  <p>Revenue increased by 12% this year.</p>

  <!-- h3 for subsections under Financial Highlights -->
  <h3>Revenue Breakdown</h3>
  <img src="revenue-chart.png" alt="Bar chart showing revenue by quarter, with Q4 highest" width="500">

  <h3>Operating Costs</h3>
  <p>Operating costs remained stable.</p>

  <!-- Another major section -->
  <h2>Future Outlook</h2>
  <p>The company plans to expand into new markets.</p>
</body>
</html>
```

**Expected output**: A page with a clear hierarchy: `h1` → `h2` → `h3` → `h3` → `h2`. The image has descriptive alternative text that conveys the chart’s key insight.

**Why this output occurs**: The heading levels form a logical, non-skipping outline. The `alt` text on the image provides a textual equivalent of the chart’s meaning, benefiting screen-reader users and search engines. This structure is both conforming and accessible.

---

#### Example 3: Accessible Form with Labels and Semantic Controls

**Setup guide**: Save as `accessible-form.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Form</title>
</head>
<body>
  <form action="/subscribe" method="post">
    <!-- <label> explicitly associated with the input via for/id -->
    <label for="email">Email address</label>
    <input type="email" id="email" name="email" required>

    <!-- Checkbox wrapped in its label for a larger click target -->
    <label>
      <input type="checkbox" name="newsletter" value="yes">
      Subscribe to the newsletter
    </label>

    <!-- Native submit button -->
    <button type="submit">Sign Up</button>
  </form>
</body>
</html>
```

**Expected output**: A form with a labelled email field, a checkbox with an associated label, and a submit button. The email field is required and will be validated by the browser.

**Why this output occurs**: The `<label for="email">` explicitly associates the label with the input, so clicking the label focuses the input and screen readers announce the label. The checkbox’s label wraps the input, providing an implicit association and a larger click target. The native `<button type="submit">` triggers form submission with built-in keyboard support.

---

### Real-World Cases

1. **Government and public-sector websites**: These are often legally required to meet WCAG accessibility standards, making semantic HTML and correct heading hierarchy mandatory.
2. **E-commerce product pages**: Semantic markup (product name as `h1`, price and description as structured content, images with informative `alt` text) improves both accessibility and search-engine visibility.
3. **Design systems and component libraries**: Best practices are codified into reusable components so that every page built from the system inherits accessible, semantic markup.

---

### References

- MDN Web Docs — HTML: HyperText Markup Language — https://developer.mozilla.org/en-US/docs/Web/HTML
- WHATWG — HTML Living Standard — https://html.spec.whatwg.org/
- W3C — Web Content Accessibility Guidelines (WCAG) — https://www.w3.org/WAI/standards-guidelines/wcag/
- W3C WAI — HTML Semantics and Accessibility — https://www.w3.org/WAI/tutorials/
- MDN Web Docs — Heading elements — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements
- MDN Web Docs — Content categories — https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories

---

## Summary Table of Content Categories

| Category | Role | Key Elements |
|---|---|---|
| **Metadata** | Information about the document | `<base>`, `<link>`, `<meta>`, `<title>`, `<style>`, `<script>` |
| **Flow** | Content that can appear in `<body>` | `<div>`, `<p>`, `<h1>`–`<h6>`, `<ul>`, `<section>`, `<img>` |
| **Sectioning** | Creates document sections | `<article>`, `<aside>`, `<nav>`, `<section>` |
| **Heading** | Defines section titles | `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>` |
| **Phrasing** | Text and inline markup | `<a>`, `<em>`, `<strong>`, `<code>`, `<span>`, `<img>` |
| **Embedded** | Imports external resources | `<img>`, `<video>`, `<audio>`, `<iframe>`, `<canvas>` |
| **Interactive** | Designed for user interaction | `<button>`, `<input>`, `<select>`, `<a href>`, `<details>` |
| **Palpable** | Rendered, substantive content | Text, images, media, form controls |
| **Script-supporting** | Supports scripting | `<script>`, `<template>` |

---

## Final Notes on Deprecated and Version-Specific Features

- **Block-level and inline terminology**: These terms are no longer normative categories in the HTML Living Standard; they are retained here because they remain the standard mental model for CSS layout and are widely used in teaching and documentation.
- **The document outline algorithm**: The theoretical algorithm that would have derived document structure from sectioning elements was never implemented consistently in browsers. Modern practice relies on explicit heading levels (`<h1>`–`<h6>`) for document structure.
- **Multiple `<h1>` elements**: Allowing multiple `<h1>` elements in nested sectioning elements was present in older HTML versions but was never a best practice and is now non-conforming.
- **`<embed>` vs `<object>`**: `<embed>` is a void element without fallback content; `<object>` is preferred when fallback content is required.
- **Custom elements**: Names must contain a hyphen (e.g. `<my-widget>`) to avoid conflicts with future standard elements.

---

This cheat sheet has covered the fundamental concepts of HTML elements, their types, categories, nesting rules, content models, and best practices, with annotated examples and authoritative references. Use it as a reference and a teaching aid, and always validate your markup against the WHATWG HTML Living Standard and accessibility guidelines.