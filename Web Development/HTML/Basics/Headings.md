# HTML Headings: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML headings are semantic elements (`<h1>` through `<h6>`) that define section titles and establish a hierarchical content structure within an HTML document.

**Technical Definition**

The `<h1>`–`<h6>` elements are section heading elements that represent six levels of section headings, where `<h1>` is the highest section level and `<h6>` is the lowest. They are flow content and heading content, with phrasing content as their permitted content model. Their DOM interface is `HTMLHeadingElement`.

**Beginner-Friendly Explanation**

Think of HTML headings like the table of contents in a book. The `<h1>` is the book's title, `<h2>` are chapter titles, `<h3>` are sections within chapters, and so on down to `<h6>`. They tell browsers, search engines, and screen readers what each part of your page is about and how the parts relate to each other.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Six levels** | `<h1>` (most important) through `<h6>` (least important) |
| **Semantic, not presentational** | Headings convey meaning and structure, not visual size. Use CSS for styling |
| **Accessibility-critical** | Screen reader users navigate pages primarily by jumping between headings using the `h` key |
| **SEO-relevant** | Search engines use headings to understand page content and hierarchy |
| **Global attributes only** | Headings support only global attributes (e.g., `id`, `class`, `lang`); presentational attributes like `align` are obsolete |
| **Cannot be empty** | Headings must contain content; empty headings are invalid and harmful to accessibility |
| **Cannot be nested** | Heading elements must not be nested inside other heading elements |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements and tags
- Awareness of semantic HTML principles
- Basic knowledge of accessibility concepts (helpful but not required)

---

### Related Programming Areas

- **Semantic HTML** – Headings are a core semantic element category
- **Web Accessibility (A11y)** – Headings are the primary navigation mechanism for screen reader users
- **Search Engine Optimization (SEO)** – Heading hierarchy influences content indexing
- **CSS Styling** – Visual presentation of headings is controlled via CSS
- **Document Outlining** – Historical concept (algorithm removed from spec) related to heading-based structure
- **HTML5 Sectioning Elements** – `<article>`, `<section>`, `<aside>`, `<nav>` interact with headings historically

---

## Core Concepts / Features

---

### 1. The Six Heading Levels (`<h1>` – `<h6>`)

#### Definitions

**Core Definition**

The `<h1>` through `<h6>` elements represent six levels of section headings, with `<h1>` being the highest rank and `<h6>` being the lowest.

**Technical Definition**

These elements represent headings for their sections. The `h1` element has the highest rank, the `h6` element has the lowest rank, and two elements with the same name have equal rank. They are categorised as flow content, heading content, and formatBlock candidates, with `hgroup` or flow content as their permitted parents.

**Beginner-Friendly Explanation**

HTML gives you six sizes of headings. `<h1>` is the most important (like a book title), and `<h6>` is the least important (like a tiny subsection deep in a chapter). You choose the level based on how important the content is, not how big you want it to look.

#### Purposes

- To establish a clear hierarchical structure within an HTML document
- To indicate the relative importance of content sections
- To provide navigational landmarks for screen reader users
- To help search engines understand page content organisation
- To enable consistent visual styling across content levels

#### Syntax Rules and Structure

**General Syntax**

```html
<h1>Heading level 1 content</h1>
<h2>Heading level 2 content</h2>
<h3>Heading level 3 content</h3>
<h4>Heading level 4 content</h4>
<h5>Heading level 5 content</h5>
<h6>Heading level 6 content</h6>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<h1>` … `<h6>` | The opening tag; the number indicates the heading level (1 = highest, 6 = lowest) |
| `Heading content` | The text (or phrasing content) that appears as the heading |
| `</h1>` … `</h6>` | The closing tag; required for all heading elements |

**Syntax Rules**

- Each heading must have a closing tag
- Headings must contain at least some content (they cannot be empty)
- Headings may contain phrasing content (text, `<em>`, `<strong>`, `<code>`, `<a>`, etc.)
- Headings must not be nested inside other heading elements
- Headings must not appear as descendants of the `<address>` element
- Only global attributes (`id`, `class`, `lang`, `dir`, `title`, etc.) are permitted

**Constraints and Limitations**

- The `align` attribute is **obsolete** — use CSS `text-align` instead
- There are exactly six levels; no `<h7>` exists
- Browsers apply default font sizes that decrease from `<h1>` to `<h6>`, but these are presentational defaults only
- Legacy behaviour where nested `<h1>` elements inside sectioning elements appeared smaller has been removed from modern browsers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Heading Levels**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Heading Levels Demo</title>
</head>
<body>
    <!-- h1: The most important heading on the page -->
    <h1>Main Page Title</h1>

    <!-- h2: A major section heading -->
    <h2>Introduction</h2>
    <p>This is the introduction section.</p>

    <!-- h3: A subsection within the h2 section -->
    <h3>Background</h3>
    <p>Background information goes here.</p>

    <!-- h4: A deeper subsection -->
    <h4>Historical Context</h4>
    <p>Historical details go here.</p>

    <!-- h5: An even deeper level -->
    <h5>Early Developments</h5>
    <p>Early development details.</p>

    <!-- h6: The least important heading level -->
    <h6>Minor Detail</h6>
    <p>A very specific detail.</p>
</body>
</html>
```

**Expected Output (Visual Rendering)**

The browser renders each heading at progressively smaller default font sizes:

```
Main Page Title          (largest, bold)
Introduction             (large, bold)
Background               (medium, bold)
Historical Context       (slightly smaller, bold)
Early Developments       (small, bold)
Minor Detail             (smallest, bold)
```

**Why This Output Occurs**

Browsers apply default user-agent stylesheets that set `font-size` values decreasing from `h1` (typically `2em`) to `h6` (typically `0.67em`). These are purely presentational defaults and can be overridden with CSS.

---

**Example 2: Structured Document with Realistic Hierarchy**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>The Crushing Bore - Story</title>
</head>
<body>
    <!-- h1: The story title (only one per page recommended) -->
    <h1>The Crushing Bore</h1>
    <p>By Chris Mills</p>

    <!-- h2: Chapter headings (siblings at the same level) -->
    <h2>Chapter 1: The dark night</h2>
    <p>It was a dark night. Somewhere, an owl hooted.</p>

    <!-- h3: A subsection within Chapter 2 -->
    <h2>Chapter 2: The eternal silence</h2>
    <p>Our protagonist could not so much as a whisper out of the shadowy figure…</p>
    <h3>The specter speaks</h3>
    <p>Several more hours had passed, when all of a sudden the specter sat bolt upright…</p>
</body>
</html>
```

**Expected Output**

The document outline would appear as:

```
1. The Crushing Bore
   1. Chapter 1: The dark night
   2. Chapter 2: The eternal silence
      1. The specter speaks
```

**Why This Output Occurs**

The `<h1>` establishes the document title. The two `<h2>` elements are siblings that represent chapters. The `<h3>` element nests under "Chapter 2" because it follows an `<h2>` and represents a subsection of that chapter. This follows the logical hierarchy where each heading level sits below the most recent heading of a higher level.

---

**Example 3: Multiple Top-Level Headings (Allowed but Not Best Practice)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Alphabetic Fruit</title>
</head>
<body>
    <!-- Multiple h1 elements are valid HTML but not recommended -->
    <h1>Apples</h1>
    <p>Pomaceous.</p>

    <h1>Bananas</h1>
    <p>Edible.</p>

    <h1>Carambola</h1>
    <p>Star.</p>
</body>
</html>
```

**Expected Output**

The document outline contains three separate top-level headings:

```
1. Apples
2. Bananas
3. Carambola
```

**Why This Output Occurs**

The HTML specification permits multiple `<h1>` elements on one page as long as they are not nested. However, this is **not considered best practice** because it creates multiple top-level headings that can confuse screen reader users and dilute SEO signals. The WHATWG specification explicitly demonstrates this pattern in its sample outlines but notes that authors are encouraged to use a single `<h1>` or appropriate ranks for section nesting.

#### Real-World Cases

**Case 1: Documentation Websites**

Technical documentation platforms (e.g., MDN Web Docs, Read the Docs) use heading hierarchies to create navigable table-of-contents structures. The single `<h1>` represents the page title, `<h2>` elements represent major sections (Installation, Usage, API Reference), and `<h3>` elements represent subsections within those topics. Screen reader users navigate through these headings to find specific information quickly.

**Case 2: E-Commerce Product Pages**

An online store might structure a product page as:

- `<h1>` – Product name
- `<h2>` – Description
- `<h2>` – Specifications
- `<h3>` – Dimensions
- `<h3>` – Materials
- `<h2>` – Customer Reviews

This structure allows all users to scan the page efficiently and helps search engines index the content appropriately.

**Case 3: Blog Articles**

A news article might use:

- `<h1>` – Article headline
- `<h2>` – Introduction
- `<h2>` – Main argument
- `<h3>` – Supporting evidence
- `<h2>` – Counterarguments
- `<h2>` – Conclusion

This creates a logical reading flow and enables assistive technology users to jump to the sections that interest them most.

---

### 2. Heading Hierarchy

#### Definitions

**Core Definition**

Heading hierarchy is the ordered relationship between heading levels that reflects the structural organisation of content, where headings must follow a logical, nested order without skipping levels.

**Technical Definition**

Heading hierarchy refers to the sequential rank order of heading elements, where each heading after the first must have a rank (h1–h6) that is equal to or one greater than the preceding heading. Sections may contain headings of any rank, but authors are strongly encouraged to use only `h1` elements or elements of the appropriate rank for the section‘s nesting level.

**Beginner-Friendly Explanation**

Heading hierarchy is like the numbering in a formal document outline. You start with 1 (h1), then 1.1 (h2), then 1.1.1 (h3), and so on. You don’t jump from 1 to 1.1.1 without using 1.1 first. This logical order helps everyone understand how your content is organised.

#### Purposes

- To communicate the structural relationships between different content sections
- To enable screen reader users to understand content organisation at a glance
- To facilitate efficient navigation for all users
- To improve search engine comprehension of page structure
- To maintain semantic correctness and professional quality standards

#### Syntax Rules and Structure

**General Syntax**

There is no special syntax for hierarchy beyond correct ordering. The hierarchy is determined by the sequence and levels of heading elements:

```html
<!-- Correct: sequential progression -->
<h1>Top Level</h1>
<h2>Second Level</h2>
<h3>Third Level</h3>
<h2>Back to Second Level</h2>
<h3>Third Level Again</h3>

<!-- Incorrect: skipped level (h2 to h4) -->
<h1>Top Level</h1>
<h2>Second Level</h2>
<h4>Skipped h3 — WRONG</h4>
```

**Component Breakdown**

| Component | Description |
|---|---|
| Sequential order | Each heading must be at most one level deeper than the previous heading |
| Return capability | You can always return to a higher level (e.g., `<h2>` after `<h3>`) |
| Single root | Preferably one `<h1>` per page acts as the document root |

**Syntax Rules**

- Do **not** skip heading levels. An `<h2>` should never be followed directly by an `<h4>` without an intervening `<h3>`
- You may use multiple headings at the same level consecutively (e.g., multiple `<h2>` elements)
- You may always go back to a higher level
- The first heading in the main content should be an `<h1>`
- The series `h1 → h2 → h1` is acceptable; `h1 → h3 → h1` is not

**Constraints and Limitations**

- The HTML specification **recommends** sequential ordering but does not strictly enforce it as a validity requirement
- WCAG (Web Content Accessibility Guidelines) guidelines consider skipped heading levels a confusing practice
- HTML validators may flag skipped levels as warnings
- The former document outline algorithm (which would have handled nesting automatically) has been **removed** from the WHATWG Living Standard

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct Heading Hierarchy**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Tax Book</title>
</head>
<body>
    <!-- Root heading -->
    <h1>The Tax Book</h1>

    <!-- Major section: Earning Money -->
    <h2>Earning Money</h2>
    <p>Earning money is good.</p>

    <!-- Subsection of Earning Money -->
    <h3>Getting a Job</h3>
    <p>To earn money you typically need a job.</p>

    <!-- Major section: Spending Money -->
    <h2>Spending Money</h2>
    <p>Spending is what money is mainly used for.</p>

    <!-- Subsections of Spending Money (siblings at h3 level) -->
    <h3>Cheap Things</h3>
    <p>Buying cheap things often is not cost-effective.</p>

    <h3>Expensive Things</h3>
    <p>The most expensive thing is often not the most cost-effective either.</p>

    <!-- Major section: Investing Money -->
    <h2>Investing Money</h2>
    <p>You can lend your money to other people.</p>

    <!-- Major section: Losing Money -->
    <h2>Losing Money</h2>
    <p>If you spend money or invest money, sooner or later you will lose money.</p>
    <h3>Poor Judgement</h3>
    <p>Usually if you lose money it's because you made a mistake.</p>
</body>
</html>
```

**Expected Output (Document Outline)**

```
1. The Tax Book
   1. Earning Money
      1. Getting a Job
   2. Spending Money
      1. Cheap Things
      2. Expensive Things
   3. Investing Money
   4. Losing Money
      1. Poor Judgement
```

**Why This Output Occurs**

Each heading follows the sequential order: `<h1>` starts the document, `<h2>` elements introduce major sections, and `<h3>` elements appear only after an `<h2>` (their parent section). No levels are skipped. This exact structure is used as an example in the WHATWG HTML specification’s sample outlines.

---

**Example 2: Incorrect Hierarchy (Skipped Level)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Incorrect Hierarchy Demo</title>
</head>
<body>
    <h1>Document Title</h1>

    <h2>First Section</h2>
    <p>Content of first section.</p>

    <!-- ERROR: Skipped h3, jumped directly to h4 -->
    <h4>Subsection That Skips Level 3</h4>
    <p>This heading level skips h3, breaking the logical hierarchy.</p>

    <h2>Second Section</h2>
    <p>Content of second section.</p>
</body>
</html>
```

**Expected Output**

Browsers will still render the `<h4>` at the appropriate visual size, but the document outline is broken. Screen readers will announce the heading level as “heading, level 4,” which is confusing when the previous heading was level 2.

**Why This Output Occurs**

The hierarchy is broken because `<h4>` requires an `<h3>` parent context. Without an `<h3>`, the `<h4>` appears to be misplaced. WCAG guidelines state that skipping heading levels “may create the impression that the structure of the document has not been properly thought through”.

#### Real-World Cases

**Case 1: Government Websites**

Government accessibility standards (e.g., Section 508 in the US, EN 301 549 in the EU) require sequential heading hierarchy for compliance. A typical government page structure:

- `<h1>` – Agency name
- `<h2>` – Services
- `<h3>` – Apply for Benefits
- `<h3>` – Check Status
- `<h2>` – About Us

Skipping levels would fail accessibility audits.

**Case 2: Learning Management Systems**

Educational platforms use headings to structure course content. A course page might have:

- `<h1>` – Course Title
- `<h2>` – Module 1
- `<h3>` – Lesson 1.1
- `<h3>` – Lesson 1.2
- `<h2>` – Module 2

This enables students using screen readers to navigate directly to specific lessons.

**Case 3: API Documentation**

Developer documentation for APIs (e.g., Stripe, Twilio) uses:

- `<h1>` – API Name
- `<h2>` – Authentication
- `<h3>` – API Keys
- `<h3>` – OAuth
- `<h2>` – Endpoints
- `<h3>` – GET /users
- `<h3>` – POST /users

This structure allows developers to quickly locate relevant information.

---

### 3. Document Outline Concepts

#### Definitions

**Core Definition**

Document outline refers to the structural representation of a document derived from its heading elements, historically intended to be generated automatically by browsers and assistive technologies.

**Technical Definition**

The document outline was an algorithm defined in early HTML5 drafts that would construct a hierarchical view of a document based on its headings and sectioning elements. The algorithm was intended to allow user agents to generate a table of contents and provide navigational affordances. **This algorithm was never implemented by any major browser or screen reader and was officially removed from the WHATWG HTML Living Standard**.

**Beginner-Friendly Explanation**

Imagine if your web browser could automatically create a table of contents from your headings. That was the idea behind the document outline — a feature that would let browsers “understand” your page structure and offer quick navigation. Unfortunately, it never worked in practice, and the feature has been removed from the official HTML standard. Today, we rely on manual heading hierarchy to achieve the same goal.

#### Purposes

- To explain the historical concept of automatic document structure detection
- To clarify why heading hierarchy must be manually maintained
- To understand why modern browsers no longer adjust heading levels based on nesting
- To inform authors that they cannot rely on sectioning elements to “fix” heading levels

#### Syntax Rules and Structure

**Historical Algorithm (Removed)**

The removed algorithm operated as follows:

1. The first heading in a sectioning element (e.g., `<section>`, `<article>`) would become the heading for that section
2. Subsequent headings of equal or higher rank would start new (implied) sections
3. Headings of lower rank would start implied subsections of the previous section

**Current State (Post-Removal)**

- No browser implements document outline generation
- Browsers have removed legacy CSS rules that made nested `<h1>` elements appear smaller
- The AOM (Accessibility Object Model) and screen readers never modified heading levels based on nesting — a nested `<h1>` was always reported as “heading, level 1”
- Authors must manually maintain correct heading levels

**Constraints and Limitations**

- The `hgroup` element‘s intended role in outlining is no longer supported by any browser
- You **cannot** rely on nesting `<h1>` inside `<section>` to create sub-headings
- You **must** explicitly use `<h1>`, `<h2>`, `<h3>`, etc. for hierarchy
- The algorithm was never a final W3C recommendation

#### Annotated Complete Step-by-Step Code Examples

**Example 1: What the Outline Algorithm Would Have Done (Historical)**

```html
<!-- This pattern was INTENDED to work with the outline algorithm -->
<body>
    <h1>My Website</h1>

    <section>
        <h1>About Us</h1>  <!-- Intended to be treated as h2 by the algorithm -->
        <p>We are a company...</p>
    </section>

    <section>
        <h1>Contact</h1>   <!-- Intended to be treated as h2 by the algorithm -->
        <p>Email us at...</p>
    </section>
</body>
```

**What the Algorithm Would Have Produced (Historically)**

```
1. My Website
   1. About Us
   2. Contact
```

**Why This Output Occurs**

The algorithm would have determined that the `<h1>` inside each `<section>` starts a new section nested within the outer document. Therefore, the nested `<h1>` elements would be treated as level 2 headings in the outline.

**What Actually Happens Now**

The browser renders both `<h1>` elements at the same visual size (the default `<h1>` size). Both are announced by screen readers as “heading, level 1.” There is no automatic hierarchy. This creates **confusion** because the visual hierarchy and the semantic hierarchy are misaligned. This is why the pattern is now **strongly discouraged**.

---

**Example 2: The Recommended Modern Approach**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Modern Heading Approach</title>
</head>
<body>
    <!-- Single h1 for the page -->
    <h1>My Website</h1>

    <!-- Explicit h2 for sections -->
    <section>
        <h2>About Us</h2>
        <p>We are a company...</p>
    </section>

    <section>
        <h2>Contact</h2>
        <p>Email us at...</p>
    </section>
</body>
</html>
```

**Expected Output**

```
1. My Website
   1. About Us
   2. Contact
```

**Why This Output Occurs**

The explicit `<h2>` elements convey the correct hierarchy regardless of nesting. Screen readers announce them as “heading, level 2,” which accurately reflects their position in the document structure. This approach works consistently across all browsers and assistive technologies.

#### Real-World Cases

**Case 1: Legacy Codebases**

Older websites built between 2010 and 2016 may have used the “nested h1” pattern in the hope that the outline algorithm would eventually be implemented. These sites now require remediation to replace nested `<h1>` elements with proper sequential headings.

**Case 2: CSS Frameworks**

Some CSS frameworks historically included styles for nested `<h1>` elements inside sectioning elements to simulate smaller headings. These workarounds are no longer necessary and should be removed.

**Case 3: Automated Accessibility Tools**

Tools like axe-core and WAVE now flag multiple `<h1>` elements and skipped heading levels as issues because they cannot rely on the (removed) outline algorithm to infer correct structure.

---

### 4. Meaningful Heading Structure

#### Definitions

**Core Definition**

Meaningful heading structure refers to the practice of choosing heading levels based on the semantic importance and logical organisation of content, rather than visual appearance.

**Technical Definition**

Meaningful heading structure requires that heading elements accurately reflect the content hierarchy, that heading text is descriptive of the content it introduces, and that heading levels are used consistently throughout a document. This is a requirement for WCAG 2.1 Success Criterion 1.3.1 (Info and Relationships) and 2.4.6 (Headings and Labels).

**Beginner-Friendly Explanation**

Don‘t pick a heading level because it “looks right” — pick it because it “means right.” If something is a main section, use `<h2>`. If it’s a subsection, use `<h3>`. Don‘t use `<h4>` just because you want smaller text. Use CSS to change how headings look, and use heading levels to show how content is organised.

#### Purposes

- To ensure content is understandable regardless of visual presentation
- To provide accurate navigational landmarks for assistive technology users
- To meet WCAG accessibility compliance requirements
- To improve SEO by providing clear content signals
- To maintain consistency across pages and content authors

#### Syntax Rules and Structure

**General Syntax**

Meaningful structure does not require special syntax, but it requires discipline in choosing heading levels:

```html
<!-- Meaningful: levels reflect content importance -->
<h1>Product Name</h1>
<h2>Description</h2>
<h3>Key Features</h3>
<h3>Technical Specifications</h3>
<h2>Pricing</h2>
<h3>Monthly Plan</h3>
<h3>Annual Plan</h3>

<!-- NOT Meaningful: levels chosen for visual size -->
<h1 style="font-size: 24px;">Section</h1>
<h3>Subsection (used because h3 looks “right”)</h3>
<h5>Another Subsection (used because h5 is small enough)</h5>
```

**Component Breakdown**

| Component | Description |
|---|---|
| Heading level | Determined by content nesting depth, not visual preference |
| Heading text | Must be descriptive of the content it introduces |
| Consistency | Same level used for same types of content across the site |

**Syntax Rules**

- Choose heading level based on the content’s position in the document hierarchy
- Use CSS (`font-size`, `font-weight`, `color`, etc.) to control visual appearance
- Heading text should be concise and descriptive (aim for under 80 characters as a guideline)
- Avoid using headings solely for styling purposes (e.g., using `<h4>` to make text bold and small)
- Maintain consistent heading usage across similar pages

**Constraints and Limitations**

- The `style` attribute can override visual appearance, but does not change semantic meaning
- Deprecated attributes like `align` must not be used; CSS is the correct mechanism for presentation
- Overuse of heading levels (e.g., going down to `<h6>` on a simple page) can fragment content excessively
- Guidelines suggest aiming to use no more than three heading levels per page for simplicity

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Meaningful Structure with CSS Styling**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Meaningful Structure Demo</title>
    <style>
        /* Reset visual hierarchy to a custom design */
        h1 {
            font-size: 2.5rem;
            color: #1a1a1a;
            border-bottom: 3px solid #0066cc;
            padding-bottom: 0.5rem;
        }

        h2 {
            font-size: 1.8rem;
            color: #333;
            margin-top: 2rem;
        }

        h3 {
            font-size: 1.3rem;
            color: #555;
            margin-top: 1.5rem;
        }

        /* Remove default browser heading margins for custom design */
        h1, h2, h3 {
            margin-top: 0;
            margin-bottom: 0.5rem;
        }
    </style>
</head>
<body>
    <!-- h1: The single top-level heading -->
    <h1>Web Development Guide</h1>

    <!-- h2: Major topic areas -->
    <h2>HTML Fundamentals</h2>
    <p>Learn the building blocks of the web.</p>

    <!-- h3: Subtopics within HTML Fundamentals -->
    <h3>Elements and Tags</h3>
    <p>Understanding how HTML elements work.</p>

    <h3>Attributes</h3>
    <p>Adding extra information to elements.</p>

    <!-- h2: Another major topic -->
    <h2>CSS Styling</h2>
    <p>Make your pages look great.</p>

    <h3>Selectors</h3>
    <p>Target elements precisely.</p>
</body>
</html>
```

**Expected Output**

The page displays with a custom visual design where all headings are styled via CSS, but the semantic hierarchy remains intact:

- “Web Development Guide” (h1) — largest, blue underline
- “HTML Fundamentals” (h2) — medium-large
- “Elements and Tags” (h3) — medium
- “Attributes” (h3) — medium
- “CSS Styling” (h2) — medium-large
- “Selectors” (h3) — medium

**Why This Output Occurs**

The CSS rules override the browser’s default heading styles, applying a custom visual design. However, the semantic meaning of each heading level is preserved. A screen reader still announces the correct heading levels, and the document hierarchy remains logical. This separation of concerns (semantics in HTML, presentation in CSS) is the cornerstone of modern web development.

---

**Example 2: Poor Meaningful Structure (Heading Levels Chosen for Style)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Poor Structure Demo</title>
</head>
<body>
    <!-- h1 is used for the site name -->
    <h1>My Blog</h1>

    <!-- h4 is used here because the author wanted smaller text -->
    <h4>Recent Posts</h4>

    <!-- h6 is used for post titles because they look like captions -->
    <h6>How to Learn HTML</h6>
    <p>HTML is the foundation of the web...</p>

    <h6>CSS Tips for Beginners</h6>
    <p>CSS controls the visual presentation...</p>

    <!-- h3 is used for a section that is visually important -->
    <h3>About the Author</h3>
    <p>I have been blogging since 2010...</p>
</body>
</html>
```

**Expected Output (Visual)**

- “My Blog” — largest (h1 default)
- “Recent Posts” — small (h4 default)
- “How to Learn HTML” — very small (h6 default)
- “CSS Tips for Beginners” — very small (h6 default)
- “About the Author” — medium (h3 default)

**Why This Output Occurs**

The author has chosen heading levels based on visual appearance rather than content importance. “Recent Posts” is a major section but appears as `<h4>` (small). “How to Learn HTML” is a top-level post title but appears as `<h6>` (very small). “About the Author” is a minor section but appears as `<h3>` (medium). This creates a **misleading document outline** where screen reader users cannot understand the actual structure of the page. The correct approach would be to use `<h2>` for “Recent Posts,” `<h3>` for post titles, and `<h2>` or `<h3>` for “About the Author,” then use CSS to adjust visual sizes.

#### Real-World Cases

**Case 1: News Websites**

Major news organisations like BBC News and The Guardian use heading levels semantically. Their article pages typically have:

- `<h1>` – Article headline
- `<h2>` – Section subheadings (e.g., “Background,” “Analysis”)
- `<h3>` – Sub-subsections within those

Visual styling (font size, weight, colour) is applied via CSS, allowing the same semantic structure to render differently across devices and contexts.

**Case 2: E-Learning Platforms**

Platforms like Coursera and edX structure course content with meaningful headings:

- `<h1>` – Course title
- `<h2>` – Weekly module titles
- `<h3>` – Individual lesson titles
- `<h4>` – Topics within lessons

This structure allows learners using screen readers to navigate directly to specific lessons, and it enables the platform to generate automatic navigation menus.

**Case 3: Government Forms**

Accessible government forms use headings to group related fields:

- `<h1>` – Form title
- `<h2>` – Personal Information
- `<h3>` – Name
- `<h3>` – Address
- `<h2>` – Employment Details

This structure helps all users understand the form’s organisation and enables screen reader users to jump between sections.

---

## References

- MDN Web Docs – HTML text fundamentals – https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/HTML_text_fundamentals
- MDN Web Docs – `<h1>`–`<h6>`: The HTML Section Heading elements – https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements
- WHATWG HTML Living Standard – Headings and outlines – https://html.spec.whatwg.org/multipage/sections.html#headings-and-outlines
- WHATWG HTML Living Standard – Sample outlines – https://html.spec.whatwg.org/multipage/sections.html#sample-outlines
- W3C – H42: Using h1-h6 to identify headings – https://www.w3.org/WAI/WCAG21/Techniques/html/H42
- W3C – WAI Web Accessibility Tutorials: Headings – https://www.w3.org/WAI/tutorials/page-structure/headings/
- web.dev – Headings and landmarks – https://web.dev/learn/accessibility/headings-landmarks
- web.dev – Text basics – https://web.dev/learn/html/text-basics
- CSS-Tricks – The final nail in the HTML5 document outline coffin – https://css-tricks.com/the-final-nail-in-the-html5-document-outline-coffin/
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – WCAG 2.1 Understanding Success Criterion 2.4.6: Headings and Labels – https://www.w3.org/WAI/WCAG21/Understanding/headings-and-labels.html
- W3C – Headings and sections (HTML 5 Markup Language) – https://w3c.github.io/html/sections.html#headings-and-sections
- WHATWG Blog – The document outline algorithm – https://blog.whatwg.org/