# Other Semantic Elements: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

"Other semantic elements" is a collective term for the HTML elements that convey specialised meaning beyond basic document structure — bundling media with captions (`<figure>`/`<figcaption>`), creating native disclosure widgets (`<details>`/`<summary>`), defining dialogs (`<dialog>`), encoding machine-readable dates (`<time>`), supplying contact information (`<address>`), and defining search landmarks (`<search>`).

**Technical Definition**

These elements are defined by the WHATWG HTML Living Standard and are categorised across flow content, phrasing content, palpable content, and interactive content. Each carries specific content-model rules and maps to ARIA roles through the HTML Accessibility API Mappings (HTML-AAM): `<figure>` → `figure`, `<figcaption>` → no corresponding role, `<details>` → `group`, `<summary>` → `button` (or `disclosure triangle`), `<dialog>` → `dialog`, `<time>` → `time`, `<address>` → `group`, and `<search>` → `search`. The `<dialog>` element also exposes imperative APIs (`show()`, `showModal()`, `close()`) and the `::backdrop` pseudo-element. The `<details>` element provides native disclosure behaviour without JavaScript, and the `<time>` element supports machine-readable `datetime` values for microformats, search engines, and assistive technologies.

**Beginner-Friendly Explanation**

HTML has more than just headings, paragraphs, and divs. Some elements exist for very specific jobs: `<figure>` groups an image with its caption, `<details>` creates a collapsible section without JavaScript, `<dialog>` creates a real popup window, `<time>` tells computers exactly what date you mean, `<address>` holds contact details, and `<search>` marks a search box. Using these elements instead of generic `<div>`s makes your pages more accessible, more semantic, and often more functional with less code.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Native behaviour** | Several elements provide built-in interactive behaviour (no JavaScript required) |
| **Machine-readable** | `<time>` exposes dates and durations in standard formats |
| **Accessibility mapping** | Each element maps to a specific ARIA role and is announced by screen readers |
| **Content-model rules** | Each element has strict rules about what it may contain and where it may appear |
| **Semantic precision** | Choosing these elements improves SEO, accessibility, and maintainability |
| **Progressive enhancement** | `<details>` works without JavaScript; `<dialog>` provides a native API with JS enhancement |
| **Standards-based** | Defined by the WHATWG HTML Living Standard |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Awareness of the DOM and CSS selectors
- Basic knowledge of accessibility principles (helpful but not required)
- Familiarity with basic JavaScript (for `<dialog>` enhancement)

---

### Related Programming Areas

- **Web Accessibility (A11y)** – These elements provide accessible semantics and behaviour
- **Search Engine Optimization (SEO)** – `<time>`, `<figure>`, and `<search>` improve machine readability
- **Progressive Enhancement** – `<details>` and `<dialog>` work without JavaScript
- **Web Components** – These elements provide patterns that components often reinvent
- **Microformats and Structured Data** – `<time datetime>` and `<address>` feed structured data
- **Form Design** – `<search>` defines a landmark around search forms

---

## Core Concepts / Features

---

### 1. The `<figure>` and `<figcaption>` Elements

#### Definitions

**Core Definition**

The `<figure>` element bundles a self-contained unit of media — such as an illustration, diagram, photograph, code snippet, or quotation — with its optional caption, provided by the `<figcaption>` element.

**Technical Definition**

The `<figure>` element represents self-contained content, potentially with an optional caption, which is specified using the first or last `<figcaption>` child. The figure, its caption, and its contents are referenced as a single unit. It is categorised as flow content and palpable content. Its content model is: either one `<figcaption>` followed by flow content, or flow content followed by one `<figcaption>`, or just flow content. Its DOM interface is `HTMLElement`. The `<figcaption>` element provides the figure with an accessible name and maps to no corresponding ARIA role, though it permits the `group`, `none`, and `presentation` roles.

**Beginner-Friendly Explanation**

A `<figure>` is like a framed picture in a book — the picture and its caption travel together. If you move the figure somewhere else, it still makes sense. It's not just for images: you can use it for code listings, diagrams, video, audio, tables, or even quotations. The `<figcaption>` gives the figure a caption.

#### Purposes

- To group self-contained content into a single referenced unit
- To associate a visible caption with a figure
- To provide an accessible name for grouped media
- To allow content to be moved without breaking the document's meaning
- To display images, diagrams, code, and quotations with captions

#### Syntax Rules and Structure

**General Syntax**

```html
<figure>
    <img src="photo.jpg" alt="Description">
    <figcaption>Caption text</figcaption>
</figure>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<figure>` | Container for the self-contained content |
| `<figcaption>` | Optional caption; must be first or last child |
| `</figure>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags of `<figure>` are mandatory
- `<figcaption>` must be the first or last child of `<figure>`
- `<figure>` accepts only global attributes
- Content inside `<figure>` must be self-contained (removable without affecting the main flow)
- `<figcaption>` accepts only global attributes

**Constraints and Limitations**

- `<figcaption>` cannot appear outside a `<figure>`
- The caption text should not simply duplicate the `alt` attribute
- `<figure>` should not be nested inside another `<figure>`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Image with Caption**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Figure Demo</title>
</head>
<body>
    <figure>
        <img src="sunset.jpg" alt="A vibrant orange sunset over the Pacific Ocean" width="600" height="400">
        <figcaption>Sunset over the Pacific Ocean, photographed from Santa Monica, California.</figcaption>
    </figure>
</body>
</html>
```

**Expected Output**

The image displays with a caption below it. Screen readers announce the image, then the caption as the figure's accessible name.

**Why This Output Occurs**

The `<figure>` groups the image and caption as one unit. The `<figcaption>` provides a visible caption and an accessible name.

---

**Example 2: Code Listing with Caption**

```html
<figure>
    <pre><code>function greet(name) {
    return `Hello, ${name}!`;
}</code></pre>
    <figcaption>A JavaScript function that greets a user by name.</figcaption>
</figure>
```

**Expected Output**

The code appears in a monospaced block, and the caption appears below it.

**Why This Output Occurs**

`<figure>` supports any flow content — including `<pre>` and `<code>` — making it ideal for annotated code listings.

#### Real-World Cases

**Case 1: Documentation Sites**

Diagrams and code snippets are wrapped in `<figure>` for accessible captions.

**Case 2: News Articles**

Photographs use `<figure>` with credits in the `<figcaption>`.

**Case 3: Academic Papers**

Charts and illustrations use `<figure>` with numbered captions.

---

### 2. The `<details>` and `<summary>` Elements

#### Definitions

**Core Definition**

The `<details>` element creates a native disclosure widget that toggles content visibility, and the `<summary>` element provides the visible label that users click to expand or collapse the content.

**Technical Definition**

The `<details>` element represents a disclosure widget in which information is visible only when the widget is toggled into an "open" state. A `<summary>` element, if present, must be the first child and represents a summary, caption, or legend for the rest of the contents. When the `<details>` element is opened, the content is rendered. It is categorised as flow content, interactive content, and palpable content. The `<summary>` element is categorised as none. It maps to `group` for `<details>` and `button` (or `disclosure triangle`) for `<summary>`. The `<details>` element supports the `open` and `name` attributes.

**Beginner-Friendly Explanation**

`<details>` and `<summary>` create a collapsible section — like an accordion or a "read more" widget — without any JavaScript. The `<summary>` is what the user clicks; the rest of the content expands or collapses. The browser handles the toggle automatically.

#### Purposes

- To create collapsible sections without JavaScript
- To provide native disclosure behaviour with accessible semantics
- To build accordions, FAQs, and progressive disclosure UIs
- To allow users to control the level of detail visible
- To improve page scanability

#### Syntax Rules and Structure

**General Syntax**

```html
<details>
    <summary>Click to expand</summary>
    <p>Hidden content revealed when expanded.</p>
</details>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<details>` | Container for the disclosure widget |
| `open` | Optional boolean attribute; initially open |
| `name` | Optional; groups multiple details into an accordion |
| `<summary>` | First child; the visible toggle label |
| `</details>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- `<summary>` must be the first child of `<details>`
- If no `<summary>` is provided, the browser provides a default label ("Details")
- The `open` attribute sets the initial state
- The `name` attribute groups details elements into a mutually-exclusive accordion

**Constraints and Limitations**

- The default browser styling of `<summary>` uses a triangle marker; CSS can customise it
- Only one `<summary>` per `<details>`
- Content inside `<details>` is still in the DOM when closed; use CSS or JS to control

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic FAQ**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Details Demo</title>
</head>
<body>
    <details>
        <summary>What is semantic HTML?</summary>
        <p>Semantic HTML is the practice of using HTML elements for their intended meaning.</p>
    </details>

    <details>
        <summary>Why does it matter?</summary>
        <p>It improves accessibility, SEO, and maintainability.</p>
    </details>
</body>
</html>
```

**Expected Output**

Two collapsible sections. Clicking a summary expands the content; clicking again collapses it.

**Why This Output Occurs**

The browser provides native toggle behaviour for `<details>`. The `<summary>` is the clickable label.

---

**Example 2: Initially Open with `open` Attribute**

```html
<details open>
    <summary>Shipping Information</summary>
    <p>Free shipping on orders over $50.</p>
</details>
```

**Expected Output**

The section is initially expanded on page load.

**Why This Output Occurs**

The `open` boolean attribute sets the initial state to open.

---

**Example 3: Accordion with `name` Attribute**

```html
<details name="faq">
    <summary>Question 1</summary>
    <p>Answer 1.</p>
</details>

<details name="faq">
    <summary>Question 2</summary>
    <p>Answer 2.</p>
</details>

<details name="faq">
    <summary>Question 3</summary>
    <p>Answer 3.</p>
</details>
```

**Expected Output**

Opening one section closes the others (mutually exclusive accordion behaviour).

**Why This Output Occurs**

The `name` attribute groups the `<details>` elements, and only one may be open at a time.

#### Real-World Cases

**Case 1: FAQ Pages**

FAQs use `<details>`/`<summary>` for collapsible questions and answers.

**Case 2: Documentation**

Documentation uses `<details>` for expandable code samples or troubleshooting notes.

**Case 3: Settings Panels**

Settings use `<details>` for advanced options that are hidden by default.

---

### 3. The `<dialog>` Element

#### Definitions

**Core Definition**

The `<dialog>` element initialises native modal or non-modal dialog boxes, complete with built-in backdrop styling and focus-trapping capabilities.

**Technical Definition**

The `<dialog>` HTML element represents a dialog box or other interactive component, such as a dismissible alert, inspector, or subwindow. It is categorised as flow content and palpable content. Its content model is flow content. Its DOM interface is `HTMLDialogElement`. It exposes the `show()`, `showModal()`, and `close()` methods, the `open` and `returnValue` attributes, and the `::backdrop` pseudo-element for styling the backdrop. When shown modally, the dialog is rendered on top of everything else, the rest of the page is inert, and focus is trapped inside. It maps to the `dialog` ARIA role.

**Beginner-Friendly Explanation**

A `<dialog>` is a real popup window built into the browser. You can show it modally (blocking the rest of the page until dismissed) or non-modally (allowing interaction with the rest of the page). Modal dialogs automatically trap focus inside and disable the background. No custom JavaScript library needed.

#### Purposes

- To create native modal and non-modal dialogs
- To provide accessible focus trapping without custom code
- To display alerts, confirmations, and subwindows
- To support dismissible overlays with `::backdrop` styling
- To replace third-party modal libraries with a native solution

#### Syntax Rules and Structure

**General Syntax**

```html
<dialog id="myDialog">
    <h2>Dialog Title</h2>
    <p>Dialog content.</p>
    <button id="close">Close</button>
</dialog>

<script>
    const dialog = document.getElementById('myDialog');
    document.getElementById('open').addEventListener('click', () => dialog.showModal());
    document.getElementById('close').addEventListener('click', () => dialog.close());
</script>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<dialog>` | Container for the dialog |
| `open` | Boolean attribute; indicates the dialog is shown non-modally |
| `showModal()` | JS method; shows the dialog modally |
| `show()` | JS method; shows the dialog non-modally |
| `close()` | JS method; closes the dialog |
| `::backdrop` | CSS pseudo-element for the modal backdrop |

**Syntax Rules**

- Both start and end tags of `<dialog>` are mandatory
- Use `showModal()` for modal dialogs (focus trap + inert background)
- Use `show()` for non-modal dialogs
- The `open` attribute reflects the dialog's visibility but does not create modality
- The `::backdrop` pseudo-element styles the area outside a modal dialog
- The `close()` method accepts a `returnValue` string

**Constraints and Limitations**

- Modal dialogs trap focus; ensure a visible close button and Escape handling
- The `open` attribute alone does not create a modal; use `showModal()`
- Older browsers may not support `<dialog>`; check compatibility or use a polyfill

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Modal Dialog**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Dialog Demo</title>
    <style>
        dialog::backdrop { background: rgba(0, 0, 0, 0.5); }
        dialog { padding: 2rem; border-radius: 8px; border: 1px solid #ccc; }
    </style>
</head>
<body>
    <button id="open">Open Dialog</button>

    <dialog id="myDialog" aria-labelledby="dialog-title">
        <h2 id="dialog-title">Confirm Action</h2>
        <p>Are you sure you want to proceed?</p>
        <button id="confirm">Confirm</button>
        <button id="cancel">Cancel</button>
    </dialog>

    <script>
        const dialog = document.getElementById('myDialog');
        document.getElementById('open').addEventListener('click', () => dialog.showModal());
        document.getElementById('confirm').addEventListener('click', () => dialog.close('confirmed'));
        document.getElementById('cancel').addEventListener('click', () => dialog.close('cancelled'));

        dialog.addEventListener('close', () => {
            console.log('Dialog returned:', dialog.returnValue);
        });
    </script>
</body>
</html>
```

**Expected Output**

Clicking "Open Dialog" shows a modal dialog with a darkened backdrop. Focus is trapped inside. Escape or clicking Cancel closes it.

**Why This Output Occurs**

`showModal()` renders the dialog modally, disables the background, traps focus, and applies the `::backdrop` styles. `close()` closes the dialog and sets `returnValue`.

---

**Example 2: Non-Modal Dialog**

```html
<dialog id="toast" open>
    <p>Your changes have been saved.</p>
    <button onclick="document.getElementById('toast').close()">Dismiss</button>
</dialog>
```

**Expected Output**

A non-modal dialog appears and does not block interaction with the rest of the page.

**Why This Output Occurs**

The `open` attribute (or `show()`) displays the dialog without trapping focus or blocking the background.

#### Real-World Cases

**Case 1: Confirmation Dialogs**

"Are you sure you want to delete this?" uses `<dialog>` with `showModal()`.

**Case 2: Settings Panels**

Application settings open in a modal dialog for focused interaction.

**Case 3: Image Lightboxes**

Clicking a thumbnail opens the full image in a modal `<dialog>` with a backdrop.

---

### 4. The `<time>` Element

#### Definitions

**Core Definition**

The `<time>` element represents a specific point in time or a duration, leveraging the machine-readable `datetime` attribute format.

**Technical Definition**

The `<time>` HTML element represents a specific period in time. It may include the `datetime` attribute to translate dates into machine-readable format, allowing for better search engine results or custom features such as reminders. It is categorised as flow content and phrasing content. Its content model is phrasing content, but with no descendant `<time>` elements. Its DOM interface is `HTMLTimeElement`. It maps to the `time` ARIA role. The `datetime` attribute must be a valid date string, time string, local date and time string, global date and time string, week string, year string, duration string, or month string.

**Beginner-Friendly Explanation**

The `<time>` element tells computers exactly what date or time you mean, even if you write it in a human-friendly way. For example, `<time datetime="2026-09-27">September 27, 2026</time>` shows readable text but gives machines the exact ISO date. This helps search engines, calendars, and screen readers understand dates correctly.

#### Purposes

- To encode dates and times in a machine-readable format
- To improve search engine understanding of dates
- To enable calendar and reminder integrations
- To represent durations
- To provide accurate date announcements for assistive technology

#### Syntax Rules and Structure

**General Syntax**

```html
<time datetime="2026-09-27">September 27, 2026</time>
<time datetime="PT2H30M">2 hours 30 minutes</time>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<time>` | Opening tag; indicates a date/time |
| `datetime` | Optional; machine-readable date/time |
| `Content` | Human-readable text |
| `</time>` | Closing tag; required |

**Valid `datetime` Formats**

| Format | Example |
|---|---|
| Month string | `2011-11` |
| Date string | `1887-12-01` |
| Yearless date | `12-01` |
| Time string | `23:59`, `12:15:47` |
| Local date and time | `2013-12-25 11:12` |
| Global date and time | `2013-12-25 11:12+0200` |
| Week string | `2013-W46` |
| Year string | `2013` |
| Duration string | `PT7H12M13S` |

**Syntax Rules**

- If `datetime` is absent, the text content must be a valid date/time string
- If `datetime` is present, the text content may be human-readable
- `<time>` must not contain a descendant `<time>` element
- The `pubdate` attribute is obsolete and must not be used

**Constraints and Limitations**

- The `datetime` value must be valid; invalid values are ignored
- Imprecise dates (e.g., "the Jurassic period") should not use `<time>`
- `<time>` should not be used for dates before the Gregorian calendar

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Publication Date**

```html
<article>
    <h1>Understanding Semantic HTML</h1>
    <p>Published on <time datetime="2026-09-27">September 27, 2026</time></p>
    <p>Reading time: <time datetime="PT8M">8 minutes</time></p>
</article>
```

**Expected Output**

The dates display as readable text; machines extract `2026-09-27` and `PT8M`.

**Why This Output Occurs**

The `datetime` attribute provides machine-readable values while the text content remains human-readable.

---

**Example 2: Global Date and Time**

```html
<p>Event starts at <time datetime="2026-12-25T18:00-08:00">6:00 PM on December 25, 2026 (Pacific Time)</time></p>
```

**Expected Output**

Displays the human-readable date; machines extract the ISO global date and time.

**Why This Output Occurs**

The `datetime` value includes the timezone offset, making it unambiguous globally.

#### Real-World Cases

**Case 1: News Sites**

Article publication dates use `<time datetime>` for search engine indexing.

**Case 2: Event Listings**

Event times use `<time datetime>` for calendar integrations.

**Case 3: Blog Posts**

Post dates and reading times use `<time>`.

---

### 5. The `<address>` Element

#### Definitions

**Core Definition**

The `<address>` element supplies contact information for a person or organisation, typically bound within the global `<footer>`.

**Technical Definition**

The `<address>` HTML element indicates that the enclosed HTML provides contact information for a person or people, or for an organisation. It is categorised as flow content and palpable content. Its content model is flow content, but with no `<address>` element descendants, no heading content descendants, no sectioning content descendants, and no `<header>` or `<footer>` element descendants. Its permitted parents are any element that accepts flow content, but it should not be nested inside another `<address>` element. It maps to the ARIA `group` role. Contact information may take any appropriate form: physical address, URL, email address, phone number, social media handle, or geographic coordinates.

**Beginner-Friendly Explanation**

The `<address>` element holds contact information. It's most commonly used in the `<footer>` of a page or article. It can contain a physical address, an email link, a phone number, or a website URL. It's italic by default, but you can style it however you like.

#### Purposes

- To provide contact information for a person or organisation
- To scope contact information to the nearest `<article>` or `<body>`
- To enable styling of contact details
- To improve semantic structure of page footers
- To support machine-readable contact information

#### Syntax Rules and Structure

**General Syntax**

```html
<address>
    Contact Name<br>
    <a href="mailto:contact@example.com">contact@example.com</a><br>
    <a href="tel:+15551234567">+1 (555) 123-4567</a>
</address>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<address>` | Opening tag; indicates contact information |
| `Content` | Flow content (contact details) |
| `</address>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- The `<address>` element must not contain heading content, sectioning content, `<header>`, or `<footer>`
- It must not contain another `<address>` element
- It should be used for contact information only, not arbitrary addresses
- The `<address>` element accepts only global attributes

**Constraints and Limitations**

- Not for arbitrary postal addresses; only for contact information
- Publication dates should use `<time>`, not `<address>`
- Cannot contain headings

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Author Contact in an Article Footer**

```html
<article>
    <h1>Understanding Semantic HTML</h1>
    <p>Semantic HTML is the practice...</p>

    <footer>
        <address>
            Written by <a href="mailto:jane@example.com">Jane Developer</a><br>
            Visit us at <a href="https://example.com">example.com</a><br>
            123 Main Street, Anytown, ST 12345
        </address>
    </footer>
</article>
```

**Expected Output**

The contact information appears below the article, in italic text, with clickable email and website links.

**Why This Output Occurs**

The `<address>` element is scoped to the nearest `<article>` ancestor, providing contact information for that article's author.

---

**Example 2: Site-Wide Contact in the Page Footer**

```html
<footer>
    <address>
        Acme Corporation<br>
        <a href="mailto:info@acme.com">info@acme.com</a><br>
        <a href="tel:+15551234567">+1 (555) 123-4567</a>
    </address>
</footer>
```

**Expected Output**

The company contact details appear in the site footer.

**Why This Output Occurs**

When `<address>` has no `<article>` ancestor, it applies to the entire document.

#### Real-World Cases

**Case 1: Blog Posts**

Blog articles use `<address>` for author contact information.

**Case 2: Corporate Sites**

Company footers use `<address>` for the physical address and contact details.

**Case 3: Government Sites**

Government contact pages use `<address>` for department contact information.

---

### 6. The `<search>` Element

#### Definitions

**Core Definition**

The `<search>` element encloses the structural parts of a search or filtering feature — forms, inputs, and buttons — to formally define a search landmark.

**Technical Definition**

The `<search>` HTML element is a container representing the parts of the document or application with form controls or other content related to performing a search or filtering operation. The `<search>` element semantically identifies the content as having the role of a search landmark. It is categorised as flow content. Its content model is flow content. Its permitted parents are any element that accepts flow content. It maps to the ARIA `search` role. The `<search>` element was added to the HTML Living Standard to formally define a search landmark without requiring `role="search"` or `<form role="search">`.

**Beginner-Friendly Explanation**

The `<search>` element marks a search box as a search region. Screen readers can jump directly to it, the same way they can jump to `<nav>`, `<main>`, or `<aside>`. Before `<search>`, developers had to add `role="search"` to a form or div. Now you can use the native element.

#### Purposes

- To define a search landmark for assistive technology
- To semantically identify search and filtering features
- To replace `role="search"` on forms and divs
- To improve screen reader navigation of search functionality
- To provide a native HTML solution for a common pattern

#### Syntax Rules and Structure

**General Syntax**

```html
<search>
    <form action="/search" method="get">
        <label for="q">Search:</label>
        <input type="search" id="q" name="q">
        <button type="submit">Go</button>
    </form>
</search>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<search>` | Container for the search feature |
| `<form>` | The search form (optional but typical) |
| `<input type="search">` | The search field |
| `</search>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags of `<search>` are mandatory
- The `<search>` element accepts only global attributes
- It should contain the form, inputs, and buttons that make up the search feature
- Do not use `<search>` for non-search forms (login, signup, etc.)

**Constraints and Limitations**

- Newer element; browser support is good in modern browsers but verify for your audience
- Older browsers may not recognise the element but will still render its content
- Use `role="search"` as a fallback if supporting very old browsers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Site Search Landmark**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Search Element Demo</title>
</head>
<body>
    <header>
        <h1>My Site</h1>
        <search>
            <form action="/search" method="get">
                <label for="site-search">Search this site:</label>
                <input type="search" id="site-search" name="q" required>
                <button type="submit">Search</button>
            </form>
        </search>
    </header>

    <main>
        <h2>Welcome</h2>
        <p>Use the search box above to find content.</p>
    </main>
</body>
</html>
```

**Expected Output**

Screen readers announce a "search landmark" containing the search form.

**Why This Output Occurs**

The `<search>` element maps to the `search` ARIA role, exposing a search landmark to assistive technology.

---

**Example 2: Filtering with `<search>`**

```html
<search>
    <h2>Filter Products</h2>
    <form action="/products" method="get">
        <label for="category">Category:</label>
        <select id="category" name="category">
            <option value="">All</option>
            <option value="laptops">Laptops</option>
            <option value="phones">Phones</option>
        </select>

        <label for="price">Max price:</label>
        <input type="number" id="price" name="max-price" min="0" step="10">

        <button type="submit">Apply Filters</button>
    </form>
</search>
```

**Expected Output**

Screen readers identify this as a search/filter region.

**Why This Output Occurs**

The `<search>` element semantically defines the filtering form as a search landmark.

#### Real-World Cases

**Case 1: E-Commerce**

Site search boxes and product filters use `<search>`.

**Case 2: Documentation**

Docs sites use `<search>` for the search bar.

**Case 3: Web Applications**

Apps use `<search>` for command palettes and filter panels.

---

### 7. Choosing the Right Semantic Element

#### Definitions

**Core Definition**

Choosing the right semantic element means selecting the element that most accurately describes the purpose and behaviour of the content, based on the content type and its relationship to surrounding material.

**Technical Definition**

The choice depends on the content's role and behaviour. Self-contained media with captions uses `<figure>`. Collapsible disclosure widgets use `<details>`/`<summary>`. Modal or non-modal dialogs use `<dialog>`. Machine-readable dates use `<time>`. Contact information uses `<address>`. Search or filtering features use `<search>`. When no semantic element fits, use `<div>` or `<span>` with a class.

#### Decision Guide

| Content Purpose | Recommended Element |
|---|---|
| Image, diagram, or code with caption | `<figure>` + `<figcaption>` |
| Collapsible/expandable section | `<details>` + `<summary>` |
| Modal or non-modal popup | `<dialog>` |
| Machine-readable date or duration | `<time>` |
| Contact information | `<address>` |
| Search or filter feature | `<search>` |
| Arbitrary decorative container | `<div>` |
| Inline text with no semantics | `<span>` |

---

## References

- MDN Web Docs – `<figure>`: The Figure with Optional Caption element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/figure
- MDN Web Docs – `<figcaption>`: The Figure Caption element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/figcaption
- MDN Web Docs – `<details>`: The Details disclosure element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/details
- MDN Web Docs – `<summary>`: The Disclosure Summary element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/summary
- MDN Web Docs – `<dialog>`: The Dialog element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog
- MDN Web Docs – `<time>`: The (Date) Time element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/time
- MDN Web Docs – `<address>`: The Contact Address element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/address
- MDN Web Docs – `<search>`: The generic search element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/search
- WHATWG HTML Living Standard – Grouping content: The figure element – https://html.spec.whatwg.org/multipage/grouping-content.html#the-figure-element
- WHATWG HTML Living Standard – Interactive elements: The details element – https://html.spec.whatwg.org/multipage/interactive-elements.html#the-details-element
- WHATWG HTML Living Standard – Interactive elements: The dialog element – https://html.spec.whatwg.org/multipage/interactive-elements.html#the-dialog-element
- WHATWG HTML Living Standard – Text-level semantics: The time element – https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-time-element
- WHATWG HTML Living Standard – Sections: The address element – https://html.spec.whatwg.org/multipage/sections.html#the-address-element
- WHATWG HTML Living Standard – Grouping content: The search element – https://html.spec.whatwg.org/multipage/grouping-content.html#the-search-element
- W3C – HTML Accessibility API Mappings (HTML-AAM) – https://w3c.github.io/html-aam/
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – WCAG 2.1 Understanding Success Criterion 4.1.2: Name, Role, Value – https://www.w3.org/WAI/WCAG21/Understanding/name-role-value.html