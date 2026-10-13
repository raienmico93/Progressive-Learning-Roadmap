# Accessible Structure: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Accessible structure is the practice of organising web content using semantic HTML elements — headings, landmarks, controls, links, images, and hidden text — so that assistive technologies can accurately interpret the layout, purpose, and relationships of every part of a page.

**Technical Definition**

Accessible structure is the implementation of WCAG Success Criteria 1.3.1 (Info and Relationships), 1.3.2 (Meaningful Sequence), 2.4.1 (Bypass Blocks), 2.4.6 (Headings and Labels), 2.4.10 (Section Headings), and 4.1.2 (Name, Role, Value) through semantic HTML and ARIA. It relies on the HTML Accessibility API Mappings (HTML-AAM), which define how HTML elements map to platform accessibility APIs. The document outline is established by heading elements (`<h1>` through `<h6>`); landmarks are established by `<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>`, and `<section>`; controls are established by `<button>`, `<a>`, `<input>`, `<select>`, and `<textarea>`; text alternatives are established by `alt`, `aria-label`, and `aria-labelledby`; and hidden content techniques use the `visually-hidden` CSS pattern and `aria-hidden="true"` to control what assistive technology perceives.

**Beginner-Friendly Explanation**

Imagine reading a book with no chapter titles, no page numbers, and no table of contents. You‘d have a hard time finding anything. That’s what a webpage feels like to a screen reader user when it lacks accessible structure. Accessible structure means using the right HTML tags so screen readers can announce “heading level 1, Site Title,” “navigation landmark,” “button, Submit,” or “list, 5 items.” It‘s how you make the invisible architecture of your page visible to assistive technology.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Hierarchical headings** | Headings follow a logical sequence without skipping levels |
| **Landmark regions** | `<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>` create navigable regions |
| **Native controls** | `<button>` for actions, `<a>` for navigation |
| **Descriptive link text** | Link text describes the destination, not the action |
| **Text alternatives** | Every image has an `alt` attribute appropriate to its purpose |
| **Hidden content control** | Screen-reader-only text and `aria-hidden` manage what AT perceives |
| **WCAG compliance** | Satisfies multiple Success Criteria at Level A and AA |
| **Screen reader navigation** | Enables efficient navigation by headings, landmarks, lists, and links |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Awareness of how screen readers interpret web content
- Basic knowledge of CSS (for visually-hidden techniques)
- Basic knowledge of ARIA attributes (helpful but not required)

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Accessible structure is the foundation of accessible web design
- **WCAG Compliance** – Multiple Success Criteria depend on accessible structure
- **Screen Reader Navigation** – Headings, landmarks, and lists enable efficient navigation
- **Semantic HTML** – The technical foundation of accessible structure
- **ARIA (Accessible Rich Internet Applications)** – Supplements native semantics for custom widgets
- **Search Engine Optimization (SEO)** – Semantic structure improves content indexing

---

## Core Concepts / Features

---

### 1. Proper Heading Hierarchy

#### Definitions

**Core Definition**

Proper heading hierarchy is the practice of sequencing heading elements (`<h1>` through `<h6>`) in a logical, nested order without skipping levels, establishing an accurate document outline.

**Technical Definition**

The `<h1>`–`<h6>` elements represent six levels of section headings, with `<h1>` being the highest rank and `<h6>` the lowest. The WHATWG HTML Living Standard encourages authors to use only `<h1>` elements or elements of the appropriate rank for the section‘s nesting level. WCAG Success Criterion 1.3.1 requires that headings convey structure programmatically. WCAG Technique H42 states that “using headings and subheadings to structure content provides users with a way to navigate the document by heading”. WCAG Success Criterion 2.4.10 (Section Headings, Level AAA) requires that section headings be used to organise content. Screen reader users frequently navigate by headings (using the `H` key in NVDA/JAWS or the rotor in VoiceOver), so a logical heading hierarchy is essential for efficient navigation.

**Beginner-Friendly Explanation**

Headings are like the chapter titles and section headers in a book. `<h1>` is the book title. `<h2>` is a chapter. `<h3>` is a section within that chapter. You never go from `<h2>` directly to `<h4>` — that‘s like skipping a chapter and jumping into a section. Proper heading hierarchy means your page has a clear, logical outline that screen readers can follow.

#### Purposes

- To establish a logical document outline for all users
- To enable screen reader users to navigate by heading level
- To communicate content hierarchy to search engines
- To satisfy WCAG Success Criteria 1.3.1, 2.4.6, and 2.4.10
- To provide visual hierarchy when combined with CSS

#### Syntax Rules and Structure

**General Syntax**

```html
<h1>Page Title</h1>
<h2>Major Section</h2>
<h3>Subsection</h3>
<h3>Another Subsection</h3>
<h2>Another Major Section</h2>
<h3>Subsection</h3>
<h4>Sub-subsection</h4>
```

**Heading Hierarchy Rules**

| Rule | Description |
|---|---|
| One `<h1>` per page | Recommended; represents the page title |
| No skipped levels | `<h2>` followed by `<h4>` is invalid |
| Sequential nesting | Each heading level sits below the most recent higher level |
| Can return to higher levels | `<h3>` can be followed by `<h2>` |
| Multiple same-level headings allowed | Multiple `<h2>` elements are permitted |

**Syntax Rules**

- The `<h1>` element should appear once per page (best practice)
- Headings must not be skipped (e.g., `<h2>` to `<h4>`)
- Headings must contain content (not empty)
- Headings may contain phrasing content (`<em>`, `<strong>`, `<code>`, `<a>`)
- Use CSS to control visual size, not heading level

**Constraints and Limitations**

- The HTML specification permits multiple `<h1>` elements but discourages them for accessibility
- The former document outline algorithm (which would have handled nesting automatically) has been removed
- Skipped levels confuse screen reader users who navigate by heading level

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct Heading Hierarchy**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Heading Hierarchy Demo</title>
</head>
<body>
    <!-- h1: Page title -->
    <h1>Web Development Guide</h1>

    <!-- h2: Major section -->
    <h2>HTML Fundamentals</h2>
    <p>HTML is the structure of the web.</p>

    <!-- h3: Subsections -->
    <h3>Elements and Tags</h3>
    <p>Elements are the building blocks.</p>

    <h3>Attributes</h3>
    <p>Attributes add information to elements.</p>

    <!-- h2: Another major section -->
    <h2>CSS Styling</h2>
    <p>CSS controls presentation.</p>

    <h3>Selectors</h3>
    <p>Selectors target elements.</p>

    <!-- h4: Sub-subsection -->
    <h4>Class Selectors</h4>
    <p>Class selectors target elements by class.</p>
</body>
</html>
```

**Expected Output**

A screen reader can navigate by heading level: H1 → H2 → H3 → H3 → H2 → H3 → H4. The outline is logical and complete.

**Why This Output Occurs**

Each heading follows the sequential order. No levels are skipped. Screen readers announce the heading level (e.g., "Heading level 2, CSS Styling"), enabling navigation.

---

**Example 2: Incorrect Heading Hierarchy**

```html
<!-- INCORRECT: Skipped heading levels -->
<h1>Page Title</h1>
<h2>Section</h2>
<h4>Skipped h3 — WRONG</h4>
<h2>Another Section</h2>
<h5>Skipped h3 and h4 — WRONG</h5>
```

**Expected Output**

Screen readers announce "Heading level 4" after "Heading level 2", confusing the user because level 3 was skipped.

**Why This Output Occurs**

The heading hierarchy is broken. Screen readers rely on sequential levels for navigation. Skipping levels makes the document outline illogical.

#### Real-World Cases

**Case 1: Documentation Websites**

MDN Web Docs and Read the Docs use strict heading hierarchies to enable navigation by heading level.

**Case 2: News Articles**

News sites use `<h1>` for the article headline, `<h2>` for section headers, and `<h3>` for subsections.

**Case 3: Government Websites**

Government websites are legally required to have accessible heading hierarchies.

---

### 2. Landmark Elements

#### Definitions

**Core Definition**

Landmark elements are semantic HTML elements (`<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>`, `<section>`) that identify major page regions, enabling screen reader users to jump directly to the content they need.

**Technical Definition**

Landmark elements map to ARIA landmark roles through the HTML Accessibility API Mappings (HTML-AAM). The `<header>` element maps to `banner` (when not nested in a sectioning element), `<nav>` maps to `navigation`, `<main>` maps to `main`, `<footer>` maps to `contentinfo` (when not nested), `<aside>` maps to `complementary`, and `<section>` maps to `region` (when it has an accessible name). WCAG Success Criterion 2.4.1 (Bypass Blocks) requires a mechanism to bypass repeated blocks of content. WCAG Technique ARIA11 documents the use of landmarks to identify regions.

**Beginner-Friendly Explanation**

Landmarks are like the sections of a newspaper: the front page (header), the table of contents (nav), the articles (main), the opinion section (aside), and the masthead (footer). Screen readers can pull up a list of landmarks and jump directly to any one. This saves users from having to tab through the entire navigation menu on every page.

#### Purposes

- To identify major page regions for assistive technology
- To enable screen reader users to jump between regions
- To satisfy WCAG Success Criterion 2.4.1 (Bypass Blocks)
- To provide a consistent structure across pages
- To improve navigation for all users

#### Landmark Elements and Roles

| Element | ARIA Role | Description |
|---|---|---|
| `<header>` | `banner` | Site header (not nested in section/article) |
| `<nav>` | `navigation` | Navigation links |
| `<main>` | `main` | Main content (one per page) |
| `<footer>` | `contentinfo` | Site footer (not nested) |
| `<aside>` | `complementary` | Tangentially related content |
| `<section>` | `region` | Thematic grouping (with accessible name) |
| `<form>` | `form` | Form (with accessible name) |

**Syntax Rules**

- Use `<main>` only once per page
- Use `<nav>` for major navigation blocks; label multiple navs with `aria-label`
- Use `<header>` and `<footer>` at the top level for `banner` and `contentinfo` roles
- Use `<section>` with `aria-labelledby` for `region` role
- Do not use ARIA landmark roles on elements that already have them

**Constraints and Limitations**

- Multiple `<nav>` elements should be labelled to distinguish them
- `<header>` and `<footer>` nested inside `<article>` or `<section>` do not map to banner/contentinfo
- `<main>` should not be nested inside another landmark

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Complete Landmark Structure**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Landmark Demo</title>
</head>
<body>
    <!-- Banner landmark -->
    <header>
        <h1>Site Title</h1>
        <!-- Navigation landmark -->
        <nav aria-label="Main">
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/about">About</a></li>
            </ul>
        </nav>
    </header>

    <!-- Main landmark -->
    <main>
        <h2>Main Content</h2>
        <p>Primary content goes here.</p>

        <!-- Complementary landmark -->
        <aside aria-label="Related links">
            <h3>Related</h3>
            <ul>
                <li><a href="/related">Related Article</a></li>
            </ul>
        </aside>
    </main>

    <!-- Contentinfo landmark -->
    <footer>
        <p>© 2026 Acme Corp.</p>
    </footer>
</body>
</html>
```

**Expected Output**

A screen reader can pull up a list of landmarks: "banner, navigation, main, complementary, contentinfo" and jump to any one.

**Why This Output Occurs**

Each landmark element maps to an ARIA role. Screen readers expose these roles and allow navigation between them.

---

**Example 2: Multiple Navigation Landmarks with Labels**

```html
<header>
    <h1>Site Title</h1>
</header>

<nav aria-label="Main">
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
    </ul>
</nav>

<main>
    <h2>Products</h2>
    <!-- Secondary navigation with distinct label -->
    <nav aria-label="Product categories">
        <ul>
            <li><a href="/products/laptops">Laptops</a></li>
            <li><a href="/products/phones">Phones</a></li>
        </ul>
    </nav>
</main>

<footer>
    <!-- Footer navigation with distinct label -->
    <nav aria-label="Footer">
        <ul>
            <li><a href="/privacy">Privacy</a></li>
            <li><a href="/terms">Terms</a></li>
        </ul>
    </nav>
</footer>
```

**Expected Output**

Screen readers announce "Main navigation," "Product categories navigation," and "Footer navigation" as distinct regions.

**Why This Output Occurs**

Each `<nav>` has a unique `aria-label`, allowing screen reader users to distinguish between them.

#### Real-World Cases

**Case 1: Government Websites**

Government websites use landmarks to help screen reader users bypass navigation and jump to main content.

**Case 2: Documentation Sites**

Documentation sites use landmarks for sidebar navigation, main content, and table of contents.

**Case 3: E-Commerce**

E-commerce sites use landmarks for header, main product area, sidebar filters, and footer.

---

### 3. Semantic Controls

#### Definitions

**Core Definition**

Semantic controls are native HTML interactive elements — `<button>` for actions and `<a>` for navigation — used according to their intended meaning, rather than generic elements styled to look interactive.

**Technical Definition**

The WHATWG HTML Living Standard defines `<button>` as an interactive element activated by a user with a mouse, keyboard, finger, voice command, or other assistive technology. The `<a>` element with an `href` attribute creates a hyperlink. Native controls have built-in keyboard support, focus management, and accessibility semantics. WCAG Success Criterion 4.1.2 (Name, Role, Value) requires that all user interface components have a programmatically determinable name and role. Using `<div>` or `<span>` with click handlers requires `role`, `tabindex`, and keyboard event handlers to replicate native behaviour, and even then may not fully match native semantics.

**Beginner-Friendly Explanation**

A button should be a `<button>`. A link should be an `<a>`. Don‘t build a fake button out of a `<div>` with an `onclick` handler. Native elements come with keyboard support, focus management, and screen reader semantics for free. If you fake it, you have to rebuild all of that yourself — and you‘ll probably miss something.

#### Purposes

- To provide built-in keyboard support and focus management
- To ensure screen readers announce the correct role
- To satisfy WCAG Success Criteria 4.1.2 and 2.1.1
- To reduce code complexity and maintenance burden
- To ensure consistent behaviour across browsers and devices

#### When to Use `<button>` vs. `<a>`

| Use `<button>` for… | Use `<a>` for… |
|---|---|
| Submitting a form | Navigating to a URL |
| Toggling a menu | Jumping to a section (`#id`) |
| Opening a modal | Downloading a file |
| Closing a dialog | Linking to an email (`mailto:`) |
| Incrementing a counter | Calling a phone number (`tel:`) |
| Any in-page action | Any navigation |

**Syntax Rules**

- Use `<button>` for actions that do not navigate
- Use `<a href>` for navigation
- Always specify `type` on `<button>` inside forms (`type="button"` or `type="submit"`)
- Do not use `<div>` or `<span>` for interactive controls
- If you must use a non-native element, add `role`, `tabindex`, and keyboard handlers

**Constraints and Limitations**

- A `<button>` inside a form defaults to `type="submit"` unless specified
- An `<a>` without `href` is not focusable and not announced as a link
- Custom controls must replicate all native keyboard behaviour

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct Semantic Controls**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Semantic Controls</title>
</head>
<body>
    <!-- Link for navigation -->
    <a href="/products">View Products</a>

    <!-- Button for action -->
    <button type="button" onclick="toggleMenu()">Toggle Menu</button>

    <!-- Button for form submission -->
    <form action="/submit" method="post">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        <button type="submit">Submit</button>
    </form>
</body>
</html>
```

**Expected Output**

Screen readers announce "View Products, link" and "Toggle Menu, button." Both are keyboard-focusable and operable.

**Why This Output Occurs**

The `<a>` element with `href` is announced as a link. The `<button>` element is announced as a button. Both are natively focusable and keyboard-operable.

---

**Example 2: Incorrect vs. Correct Custom Control**

```html
<!-- INCORRECT: Div-based fake button -->
<div class="btn" onclick="submitForm()">Submit</div>

<!-- CORRECT: Native button -->
<button type="submit">Submit</button>

<!-- INCORRECT: Div-based fake link -->
<div class="link" onclick="location.href='/about'">About</div>

<!-- CORRECT: Native link -->
<a href="/about">About</a>
```

**Expected Output**

The native elements are keyboard-accessible, focusable, and announced correctly. The div-based versions are not.

**Why This Output Occurs**

Native elements have built-in semantics, keyboard support, and focus management. Div-based controls lack all of these unless manually added.

#### Real-World Cases

**Case 1: Navigation Menus**

Navigation menus use `<a>` for links and `<button>` for dropdown toggles.

**Case 2: Forms**

Forms use `<button type="submit">` for submission and `<button type="reset">` for resetting.

**Case 3: Interactive Widgets**

Tabs, accordions, and modals use `<button>` for triggers and controls.

---

### 4. Meaningful Link Text

#### Definitions

**Core Definition**

Meaningful link text is anchor text that describes the destination or purpose of a link, allowing users to understand where the link leads without needing surrounding context.

**Technical Definition**

WCAG Success Criterion 2.4.4 (Link Purpose in Context, Level A) requires that the purpose of each link can be determined from the link text alone or from the link text together with its programmatically determined context. WCAG Success Criterion 2.4.9 (Link Purpose Link Only, Level AAA) requires that the purpose can be determined from the link text alone. WCAG Technique G91 states that “the objective of this technique is to describe the purpose of a link in the text of the link.” Screen reader users frequently pull up a list of all links on a page, so link text must make sense out of context.

**Beginner-Friendly Explanation**

Link text is the words you click on. It should tell the user where the link goes. “Click here” tells them nothing. “Read the 2026 Annual Report” tells them exactly what they’ll get. Screen readers can list all links on a page, so link text must be meaningful on its own.

#### Purposes

- To enable screen reader users to understand link destinations out of context
- To help users with cognitive limitations determine whether to follow a link
- To satisfy WCAG Success Criteria 2.4.4 and 2.4.9
- To improve SEO by giving search engines meaningful context
- To improve the experience for all users

#### Good vs. Poor Link Text

| Poor Link Text | Better Link Text |
|---|---|
| Click here | Download the 2026 Annual Report (PDF) |
| Read more | Read more about our accessibility policy |
| Link | Visit the W3C HTML specification |
| Info | View pricing and plans |
| Learn more | Learn more about semantic HTML |
| Details | See detailed product specifications |

**Syntax Rules**

- Link text should describe the destination or purpose
- Avoid generic phrases: “click here,” “read more,” “link,” “info”
- Do not use the same link text for different URLs on the same page
- Do not use raw URLs as link text unless necessary
- Use `aria-label` to supplement link text when the visible text is insufficient

**Constraints and Limitations**

- Link text is read out of context by screen readers
- Vague link text is a common accessibility failure
- The `title` attribute is not a reliable substitute for meaningful link text

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Good vs. Poor Link Text**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Link Text Demo</title>
</head>
<body>
    <!-- POOR: Generic link text -->
    <p>To learn about web accessibility, <a href="/accessibility">click here</a>.</p>

    <!-- GOOD: Descriptive link text -->
    <p>Learn more in our <a href="/accessibility">complete guide to web accessibility</a>.</p>
</body>
</html>
```

**Expected Output**

Both links work, but the second tells users (and screen readers) exactly what to expect.

**Why This Output Occurs**

The first link text “click here” is meaningless out of context. The second describes the destination.

---

**Example 2: Multiple Links to the Same Destination**

```html
<!-- CONSISTENT: Same link text for same destination -->
<p><a href="/docs/intro">Introduction to HTML</a></p>
<p>Start with the <a href="/docs/intro">Introduction to HTML</a>.</p>

<!-- INCONSISTENT: Different link text for same destination -->
<p><a href="/docs/intro">Getting Started</a></p>
<p>Start with the <a href="/docs/intro">Introduction</a>.</p>
```

**Expected Output**

Both sets work, but only the first follows the consistency guideline.

**Why This Output Occurs**

The W3C recommends that if multiple links share the same destination, they should use consistent link text.

#### Real-World Cases

**Case 1: Government Websites**

Government sites follow strict accessibility guidelines requiring descriptive link text.

**Case 2: News Articles**

News sites use descriptive link text for source citations and related content.

**Case 3: E-Commerce**

Product pages use the product name as link text instead of “View product.”

---

### 5. Alternative Text for Imagery

#### Definitions

**Core Definition**

Alternative text (alt text) is a textual description of an image provided via the `alt` attribute on `<img>` elements, serving as a replacement when the image cannot be seen.

**Technical Definition**

The `alt` attribute is required on every `<img>` element. WCAG Success Criterion 1.1.1 (Non-text Content, Level A) requires that all non-text content has a text alternative that serves the equivalent purpose. Informative images require descriptive alt text conveying the essential information. Decorative images require an empty `alt=""` so assistive technology skips them. Functional images (used as links or buttons) require alt text describing the function. Complex images (charts, diagrams) require a two-part alternative: a short `alt` plus a longer textual equivalent.

**Beginner-Friendly Explanation**

Every image needs an `alt` attribute. If the image conveys information, the alt text should describe it. If the image is purely decorative, use an empty `alt=""` so screen readers skip it. If the image is a button, the alt text should describe what the button does, not what the image looks like.

#### Purposes

- To provide a textual equivalent for screen reader users
- To display descriptive text when images fail to load
- To satisfy WCAG Success Criterion 1.1.1
- To improve SEO by giving search engines context
- To support users with images disabled

#### Image Types and Alt Text

| Image Type | Alt Text | Example |
|---|---|---|
| **Informative** | Describes the information | `alt="Bar chart showing Q1 sales of $1.2M"` |
| **Decorative** | Empty string | `alt=""` |
| **Functional** | Describes the function | `alt="Search"` |
| **Complex** | Short alt + long description | `alt="Sales chart"` + `aria-describedby` |
| **Logo** | Company name | `alt="Acme Corporation"` |
| **Image of text** | The text content | `alt="Welcome to Acme"` |

**Syntax Rules**

- Every `<img>` must have an `alt` attribute
- Informative images: describe the content
- Decorative images: `alt=""`
- Functional images: describe the function
- Complex images: short alt + long description
- Do not include “image of” or “picture of”
- Use `aria-hidden="true"` on decorative SVGs

**Constraints and Limitations**

- Alt text cannot convey complex information fully; use long descriptions for charts
- The `title` attribute is not a substitute for `alt`
- Overly long alt text burdens screen reader users

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Informative vs. Decorative Images**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Alt Text Demo</title>
</head>
<body>
    <!-- INFORMATIVE: describes the content -->
    <img src="sunset.jpg"
         alt="A vibrant orange sunset over the Pacific Ocean"
         width="600" height="400">

    <!-- DECORATIVE: empty alt so screen readers skip it -->
    <img src="divider.png" alt="" width="600" height="20">
</body>
</html>
```

**Expected Output**

The sunset image is announced as “A vibrant orange sunset over the Pacific Ocean, image.” The divider is skipped entirely.

**Why This Output Occurs**

The `alt` attribute provides a text alternative for the informative image. The empty `alt=""` tells assistive technology the divider is decorative.

---

**Example 2: Functional Image**

```html
<a href="/search">
    <img src="search-icon.svg" alt="Search" width="24" height="24">
</a>

<button type="button" onclick="window.print()">
    <img src="print-icon.svg" alt="Print this page" width="20" height="20">
</button>
```

**Expected Output**

Screen readers announce “Search, link” and “Print this page, button.”

**Why This Output Occurs**

The alt text describes the function, not the visual appearance.

#### Real-World Cases

**Case 1: News Article Images**

News images use descriptive alt text that captures the scene.

**Case 2: E-Commerce Product Images**

Product images use descriptive alt text with product details.

**Case 3: Decorative Backgrounds**

Decorative images use `alt=""` to avoid cluttering screen reader output.

---

### 6. Hidden Content Techniques

#### Definitions

**Core Definition**

Hidden content techniques are methods for providing contextual text to screen readers while hiding it visually, or hiding content from screen readers while keeping it visible.

**Technical Definition**

The `.visually-hidden` (also called `.sr-only`) CSS pattern hides content visually while keeping it available to screen readers. It uses `position: absolute`, `width: 1px`, `height: 1px`, `overflow: hidden`, `clip: rect(0, 0, 0, 0)`, and `white-space: nowrap`. The `aria-hidden="true"` attribute hides content from assistive technology while keeping it visible on screen. These techniques are complementary: `.visually-hidden` for screen-reader-only text (like skip links and form labels), and `aria-hidden` for decorative content that should be ignored by AT.

**Beginner-Friendly Explanation**

Sometimes you want text that only screen readers can see — like a “Skip to main content” link that’s invisible until focused, or a label for an icon-only button. The `.visually-hidden` class hides text visually but keeps it available to screen readers. Conversely, `aria-hidden="true"` hides something from screen readers while keeping it visible on screen — useful for decorative icons that would be redundant.

#### Purposes

- To provide accessible names for icon-only controls
- To add context for screen reader users without visual clutter
- To create skip links that are invisible until focused
- To hide decorative elements from assistive technology
- To satisfy WCAG Success Criteria 1.1.1 and 2.4.1

#### Syntax Rules and Structure

**The `.visually-hidden` CSS Pattern**

```css
.visually-hidden {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
}
```

**Skip Link Pattern**

```css
.skip-link {
    position: absolute;
    top: -40px;
    left: 0;
    background: #000;
    color: #fff;
    padding: 8px;
    z-index: 100;
}
.skip-link:focus {
    top: 0;
}
```

**`aria-hidden="true"`**

```html
<button type="button">
    <svg aria-hidden="true" width="16" height="16"><!-- icon --></svg>
    Search
</button>
```

**Syntax Rules**

- Use `.visually-hidden` for text that should be announced but not seen
- Use `aria-hidden="true"` for decorative elements that should be ignored by AT
- Never use `display: none` or `visibility: hidden` for screen-reader-only content (these hide from AT too)
- Skip links should become visible on focus
- Do not use `aria-hidden="true"` on focusable elements

**Constraints and Limitations**

- `aria-hidden="true"` on focusable elements creates a focus trap for keyboard users
- `.visually-hidden` content is still in the accessibility tree and is announced
- Some screen readers may not fully support `clip: rect(0,0,0,0)`; the modern `clip-path` is preferred

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Skip Link**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Skip Link Demo</title>
    <style>
        .skip-link {
            position: absolute;
            top: -40px;
            left: 0;
            background: #000;
            color: #fff;
            padding: 8px;
            z-index: 100;
        }
        .skip-link:focus {
            top: 0;
        }
    </style>
</head>
<body>
    <!-- Skip link visible on focus -->
    <a href="#main-content" class="skip-link">Skip to main content</a>

    <header>
        <nav>
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/about">About</a></li>
            </ul>
        </nav>
    </header>

    <main id="main-content">
        <h1>Main Content</h1>
        <p>This is the main content area.</p>
    </main>
</body>
</html>
```

**Expected Output**

Pressing Tab on page load reveals the skip link at the top of the page. Pressing Enter jumps to the main content.

**Why This Output Occurs**

The skip link is a native `<a>` element that links to `#main-content`. The CSS positions it off-screen and brings it into view on focus.

---

**Example 2: Icon-Only Button with `.visually-hidden` Text**

```html
<style>
    .visually-hidden {
        position: absolute;
        width: 1px;
        height: 1px;
        padding: 0;
        margin: -1px;
        overflow: hidden;
        clip: rect(0, 0, 0, 0);
        white-space: nowrap;
        border: 0;
    }
</style>

<button type="button">
    <svg aria-hidden="true" width="16" height="16"><!-- search icon --></svg>
    <span class="visually-hidden">Search</span>
</button>
```

**Expected Output**

The button displays only the icon. Screen readers announce “Search, button.”

**Why This Output Occurs**

The `.visually-hidden` span provides the accessible name. The `aria-hidden="true"` on the SVG hides it from AT (since it’s decorative).

---

**Example 3: `aria-hidden` for Redundant Decorative Content**

```html
<button type="button">
    <svg aria-hidden="true" width="16" height="16"><!-- icon --></svg>
    Save
</button>
```

**Expected Output**

The button displays an icon and the text “Save.” Screen readers announce “Save, button” and skip the icon.

**Why This Output Occurs**

The `aria-hidden="true"` attribute tells assistive technology to ignore the SVG, since the text “Save” already describes the button.

#### Real-World Cases

**Case 1: Skip Links**

Government and large websites use skip links to help keyboard users bypass navigation.

**Case 2: Icon Buttons**

Icon-only buttons (hamburger menus, close buttons, search) use `.visually-hidden` text for accessible names.

**Case 3: Decorative Icons**

Decorative icons alongside text use `aria-hidden="true"` to avoid redundant announcements.

---

### 7. Choosing the Right Accessible Structure Approach

#### Definitions

**Core Definition**

Choosing the right accessible structure approach means selecting the appropriate HTML elements, ARIA attributes, and CSS techniques based on the content type and the needs of assistive technology users.

**Technical Definition**

The choice depends on the content: headings for outline, landmarks for regions, native controls for interaction, descriptive link text for navigation, alt text for images, and hidden content techniques for context and decoration. Native HTML is always preferred; ARIA is a supplement, not a replacement.

#### Decision Guide

| Content Type | Recommended Approach |
|---|---|
| Page title | `<h1>` |
| Section title | `<h2>`–`<h6>` in sequence |
| Site header | `<header>` |
| Navigation | `<nav>` + `<ul>` + `<a>` |
| Main content | `<main>` |
| Sidebar | `<aside>` |
| Footer | `<footer>` |
| Action button | `<button>` |
| Navigation link | `<a href>` |
| Informative image | `<img alt="description">` |
| Decorative image | `<img alt="">` |
| Functional image | `<img alt="function">` |
| Icon-only button | `.visually-hidden` or `aria-label` |
| Decorative icon | `aria-hidden="true"` |
| Skip link | `.skip-link` + `:focus` |

---

## References

- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.1: Bypass Blocks – https://www.w3.org/WAI/WCAG21/Understanding/bypass-blocks.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.4: Link Purpose (In Context) – https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.6: Headings and Labels – https://www.w3.org/WAI/WCAG21/Understanding/headings-and-labels.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.10: Section Headings – https://www.w3.org/WAI/WCAG21/Understanding/section-headings.html
- W3C – WCAG 2.1 Understanding Success Criterion 4.1.2: Name, Role, Value – https://www.w3.org/WAI/WCAG21/Understanding/name-role-value.html
- W3C – H42: Using h1-h6 to identify headings – https://www.w3.org/WAI/WCAG21/Techniques/html/H42
- W3C – H44: Using label elements to associate text labels with form controls – https://www.w3.org/WAI/WCAG21/Techniques/html/H44
- W3C – G91: Providing link text that describes the purpose of a link – https://www.w3.org/WAI/WCAG21/Techniques/general/G91
- W3C – ARIA11: Using ARIA landmarks to identify regions of a page – https://www.w3.org/WAI/WCAG21/Techniques/aria/ARIA11
- W3C – H37: Using alt attributes on img elements – https://www.w3.org/WAI/WCAG21/Techniques/html/H37
- MDN Web Docs – HTML: A good basis for accessibility – https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML
- MDN Web Docs – ARIA landmarks – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/landmark_role
- MDN Web Docs – `aria-hidden` attribute – https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-hidden
- WebAIM – Skip Navigation Links – https://webaim.org/techniques/skipnav/
- WebAIM – Alternative Text – https://webaim.org/techniques/alttext/
- The A11Y Project – How to hide content accessibly – https://www.a11yproject.com/posts/how-to-hide-content/
- The A11Y Project – Checklist – https://www.a11yproject.com/checklist/