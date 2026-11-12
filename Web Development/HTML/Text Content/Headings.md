# Headings — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Headings are the HTML elements `<h1>` through `<h6>` that represent six levels of section titles in a document. They create a hierarchical outline of the page's content, with `<h1>` as the highest (most important) level and `<h6>` as the lowest.

**Technical Definition**
In the WHATWG HTML Living Standard, the `h1`–`h6` elements are **heading content** elements belonging to the **flow content**, **heading content**, and **palpable content** categories. Their content model is **phrasing content**. By default, all heading elements create a **block-level box** in the layout, starting on a new line and taking up the full width available in their containing block. Each heading element has a **heading level**, an unsigned integer from 1 to 6, where `h1` has the highest rank and `h6` the lowest. The first heading element inside a sectioning content element (such as `<article>`, `<aside>`, `<nav>`, or `<section>`) or a sectioning root element (such as `<blockquote>`, `<body>`, `<details>`, `<dialog>`, `<fieldset>`, `<figure>`, or `<td>`) represents the heading for that section.

**Beginner-Friendly Explanation**
Headings are the titles and subtitles of a web page. `<h1>` is the main title — the biggest, most important heading. `<h2>` is a major section title, `<h3>` is a subsection, and so on down to `<h6>`. Think of headings like a book's table of contents: the book title is the `h1`, chapter titles are `h2`, section titles within chapters are `h3`. Headings help readers scan the page, help screen readers navigate, and help search engines understand what the page is about.

---

### Key Characteristics

- **Six hierarchical levels**: `<h1>` is the highest rank and `<h6>` the lowest.
- **Block-level by default**: Headings start on a new line and take up the full available width.
- **Semantic structure**: Headings convey the logical hierarchy of content, not just visual size.
- **Screen-reader navigation**: Assistive technologies use headings to build a navigable outline and allow users to jump between sections.
- **SEO signal**: Search engines use headings to understand content structure and topical relevance.
- **Single `<h1>` recommended**: While multiple `<h1>` elements are technically allowed when not nested, best practice is one `<h1>` per page describing the page's main topic.
- **No heading-level skipping**: Best practice is to use levels in sequence without gaps (e.g. `h1` → `h2` → `h3`).

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<html>`, `<head>`, `<body>`, and elements.
2. **Flow content and phrasing content**: understanding that headings accept phrasing content only.
3. **The concept of semantic HTML**: knowing that elements carry meaning beyond presentation.
4. **Browser developer tools**: inspecting the accessibility tree and heading outline.
5. **A screen reader or accessibility inspection tool**: observing how headings are announced and navigated.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | Headings satisfy WCAG 1.3.1 (Info and Relationships) and 2.4.10 (Section Headings); proper markup is essential for screen-reader navigation. |
| **SEO** | Headings provide structure, context, and topical signals to search engines. |
| **CSS** | Headings have default font-size and margin styles; CSS controls their visual presentation. |
| **JavaScript / DOM** | Headings are DOM nodes that scripts can query and manipulate; the document outline is derived from them. |
| **Document outline** | The theoretical HTML outline algorithm uses headings and sectioning elements to build a document outline, though browser support is inconsistent. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<h1>` to `<h6>`** — the heading elements
2. **Heading hierarchy** — nesting and outline construction
3. **Heading best practices** — authoring guidelines
4. **Heading accessibility** — screen-reader navigation and WCAG conformance
5. **Heading SEO** — search-engine optimisation
6. **Heading pitfalls** — common mistakes and anti-patterns

---

## 1. `<h1>` to `<h6>` — The Heading Elements

### Definitions

**Core Definition**
The `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, and `<h6>` elements represent six levels of section headings, with `<h1>` being the highest section level and `<h6>` the lowest.

**Technical Definition**
The heading elements belong to the **flow content**, **heading content**, and **palpable content** categories. Their content model is **phrasing content**. They accept only **global attributes**. By default, all heading elements create a block-level box in the layout, starting on a new line and taking up the full width available in their containing block. The elements have a **heading level** (1–6), and the first heading element inside a sectioning content or sectioning root element represents the heading for that section.

**Beginner-Friendly Explanation**
There are six heading tags. `<h1>` is the most important — the main title. `<h2>` is for major sections, `<h3>` for subsections, and so on down to `<h6>`. The number in the tag tells you the level.

---

### Purposes

- **To define section titles** and convey content hierarchy.
- **To enable screen-reader navigation** by creating a navigable outline.
- **To provide structure for search engines** to understand page topics.
- **To allow user agents to construct a table of contents** automatically.
- **To support visual scanning** by breaking content into manageable sections.

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
| `<h4>` | Fourth | Sub-subsection heading. |
| `<h5>` | Fifth | Deep subsection heading. |
| `<h6>` | Lowest | Deepest practical heading level. |

#### Syntax Rules

1. **Content model is phrasing content**: headings cannot contain block-level elements.
2. **Only global attributes are permitted**; there are no element-specific attributes.
3. **Do not use heading elements to resize text**: use CSS `font-size` instead.
4. **Do not skip heading levels**: always start from `<h1>`, followed by `<h2>`, and so on.
5. **Prefer a single `<h1>` per page**: while multiple `<h1>` elements are allowed when not nested, one `<h1>` describing the page's content is best practice.

#### Constraints and Limitations

- **Multiple `<h1>` elements**: Allowed by the standard when not nested, but not considered best practice; nesting multiple `<h1>` elements in nested sectioning elements was allowed in older HTML versions but is now non-conforming.
- **Heading-level offsets**: The HTML standard specifies that `<h1>` elements in a `<section>`, `<article>`, `<aside>`, or `<nav>` element should render as an `<h2>` (smaller font size) by default.
- **Headings are not for styling**: Using headings purely for visual formatting misleads users who rely on headings for navigation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Correctly Nested Heading Hierarchy

**Setup Guide**: Save as `headings-correct.html` and inspect the heading outline in the browser's accessibility tools.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Heading Hierarchy</title>
</head>
<body>
  <!-- h1 is the main page title -->
  <h1>Web Development Guide</h1>

  <!-- h2 for major sections -->
  <h2>HTML Fundamentals</h2>
  <p>HTML provides the structure of web pages.</p>

  <!-- h3 for subsections under HTML Fundamentals -->
  <h3>Elements</h3>
  <p>Elements are the building blocks of HTML.</p>

  <h3>Attributes</h3>
  <p>Attributes modify element behaviour.</p>

  <!-- h2 for another major section -->
  <h2>CSS Fundamentals</h2>
  <p>CSS controls the presentation of web pages.</p>
</body>
</html>
```

**Expected output**: A page with "Web Development Guide" as the largest heading, "HTML Fundamentals" and "CSS Fundamentals" as second-level headings, and "Elements" and "Attributes" as third-level headings nested under "HTML Fundamentals". The outline is logical and non-skipping.

**Why this output occurs**: The heading levels follow the recommended hierarchy: `h1` → `h2` → `h3` → `h3` → `h2`. Screen readers can present this as a structured outline, and the visual rendering follows the browser's default heading styles. Technique H42 recommends this structure for conveying content hierarchy.

---

#### Example 2: Headings in a Multi-Column Layout

**Setup Guide**: Save as `headings-layout.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Multi-Column Headings</title>
</head>
<body>
  <!-- Navigation column uses h2 (less important than main content) -->
  <nav>
    <h2>Site Navigation</h2>
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
    </ul>
  </nav>

  <!-- Main content uses h1 (the page's main topic) -->
  <main>
    <h1>Stock Market Up Today</h1>
    <p>Markets rallied on positive earnings reports.</p>
  </main>

  <!-- Sidebar uses h2 (less important than main content) -->
  <aside>
    <h2>Related Links</h2>
    <ul>
      <li><a href="/analysis">Market Analysis</a></li>
    </ul>
  </aside>
</body>
</html>
```

**Expected output**: The main content's `h1` is the highest-level heading on the page. The navigation and sidebar use `h2` because they are relatively less important than the main content, even though they may appear earlier in the source order.

**Why this output occurs**: Technique H42 permits headings in a multi-column layout to reflect relative importance. The main content's `h1` is the page's primary topic, while the navigation and sidebar headings use `h2` to indicate their secondary role.

---

### Real-World Cases

1. **Documentation sites**: `h1` for the page title, `h2` for major topics, and `h3` for sub-topics.
2. **Blog posts**: The post title is typically the page's `h1`; section headings within the post use `h2` and `h3`.
3. **E-commerce category pages**: `h1` names the category, `h2` names product groups, and `h3` names individual products or sub-groups.

---

### References

- MDN Web Docs — `<h1>`–`<h6>`: The HTML Section Heading elements — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements
- W3C — Technique H42: Using h1-h6 to identify headings — https://www.w3.org/WAI/WCAG21/Techniques/html/H42
- WHATWG — The h1, h2, h3, h4, h5, and h6 elements — https://html.spec.whatwg.org/multipage/sections.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements

---

## 2. Heading Hierarchy

### Definitions

**Core Definition**
Heading hierarchy is the logical nesting of heading levels that reflects the structure of the content, with `<h1>` at the top and progressively deeper levels (`<h2>`–`<h6>`) representing nested subsections.

**Technical Definition**
Heading hierarchy is expressed through the **heading level** of each heading element. The WHATWG standard assigns each `h1`–`h6` element a **heading level** equal to its number. The first heading element inside a **sectioning content** element (`<article>`, `<aside>`, `<nav>`, `<section>`) or a **sectioning root** element (`<blockquote>`, `<body>`, `<details>`, `<dialog>`, `<fieldset>`, `<figure>`, `<td>`) represents the heading for that section. The theoretical **document outline algorithm** would use sectioning elements and headings to build an outline, but it is not consistently implemented in browsers; explicit heading levels remain the primary structural signal.

**Beginner-Friendly Explanation**
Heading hierarchy means using heading levels in order: `h1` first, then `h2`, then `h3`, and so on. When you move to a new subsection, you go one level deeper. When you finish a subsection, you go back up. This creates a clear table of contents for your page.

---

### Purposes

- **To convey logical structure**: headings nested hierarchically show the relationship between content sections.
- **To enable screen-reader navigation**: users can jump between heading levels to find content.
- **To improve comprehension**: a clear hierarchy helps all users understand the organisation of the page.
- **To satisfy WCAG requirements**: Technique H69 requires headings at the beginning of each section of content.

---

### Syntax Rules and Structure

#### Complete General Syntax (Hierarchical Example)

```html
<h1>Plant Foods that Humans Eat</h1>
<h2>Fruit</h2>
<h3>Apple</h3>
<h3>Orange</h3>
<h3>Banana</h3>
<h2>Vegetables</h2>
<h3>Broccoli</h3>
<h3>Brussels sprouts</h3>
<h3>Green beans</h3>
```

**Component breakdown**

| Level | Role in hierarchy |
|---|---|
| `<h1>` | The page's main topic. |
| `<h2>` | A major section under the `h1`. |
| `<h3>` | A subsection under an `h2`. |
| `<h4>`–`<h6>` | Progressively deeper subsections. |

#### Syntax Rules

1. **Do not skip heading levels**: an `h3` must not be followed by an `h5` without an intervening `h4`.
2. **Nest hierarchically**: `h3` is a subsection of `h2`, which is a subsection of `h1`.
3. **Skipping levels is not a WCAG failure** as long as each heading is marked up as a heading and assigned a level consistent with its relative position in the hierarchy.
4. **Fixed sections** (navigation, sidebars) may start at `h2` or `h3` if they are relatively less important than the main content and are used consistently across the site.

#### Constraints and Limitations

- **Skipping levels confuses screen-reader users**: if a screen-reader user cycles through headings and a level is missing, they may think they missed content.
- **The document outline algorithm is not reliably implemented**: do not rely on sectioning elements alone to create an outline; explicit heading levels are the primary signal.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Hierarchical Outline

**Setup Guide**: Save as `hierarchy.html`. Use a screen reader or heading-navigation tool to jump between levels.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hierarchical Headings</title>
</head>
<body>
  <h1>Plant Foods that Humans Eat</h1>
  <p>There are an abundant number of plants that humans eat.</p>

  <h2>Fruit</h2>
  <p>A fruit is a structure of a plant that contains its seeds.</p>

  <h3>Apple</h3>
  <p>The apple is the pomaceous fruit of the apple tree.</p>

  <h3>Orange</h3>
  <p>The orange is a hybrid of ancient cultivated origin.</p>

  <h3>Banana</h3>
  <p>Banana is the common name for herbaceous plants.</p>

  <h2>Vegetables</h2>
  <p>A vegetable is an edible plant or part of a plant.</p>

  <h3>Broccoli</h3>
  <p>Broccoli is a plant of the mustard/cabbage family.</p>
</body>
</html>
```

**Expected output**: A clear outline: `h1` → `h2` (Fruit) → `h3` (Apple, Orange, Banana) → `h2` (Vegetables) → `h3` (Broccoli). Screen-reader users can jump to any level and understand the structure.

**Why this output occurs**: The headings follow the hierarchical pattern recommended by Technique H42 and H69. Each `h3` is a logical subsection of its preceding `h2`, and the `h2` sections are subsections of the `h1`.

---

#### Example 2: Skipped Level (Anti-Pattern)

**Setup Guide**: Save as `hierarchy-skip.html`. Compare the outline with the previous example.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Skipped Heading Level</title>
</head>
<body>
  <h1>Main Title</h1>
  <p>Introduction.</p>

  <!-- WRONG: jumps from h1 directly to h3 -->
  <h3>Skipped h2</h3>
  <p>This heading skips level 2.</p>
</body>
</html>
```

**Expected output**: The browser renders "Skipped h2" smaller than "Main Title", but a screen reader's heading list shows a gap where an `h2` should be, potentially confusing users.

**Why this output occurs**: Skipping from `h1` to `h3` violates the recommended hierarchical nesting. While not a strict WCAG failure, it creates a usability problem for screen-reader users who navigate by heading level.

---

### Real-World Cases

1. **Long-form articles**: Chapters use `h2`, sections within chapters use `h3`, and sub-sections use `h4`.
2. **Documentation pages**: API references use `h1` for the page title, `h2` for method categories, and `h3` for individual methods.
3. **E-commerce category pages**: `h1` for the category, `h2` for product groups, `h3` for individual products.

---

### References

- W3C — Technique H69: Providing heading elements at the beginning of each section of content — https://www.w3.org/WAI/WCAG22/Techniques/html/H69
- W3C — Technique G141: Organizing a page using headings — https://www.w3.org/WAI/WCAG22/Techniques/general/G141
- Government NZ — Make headings accessible — https://govtnz.github.io/web-a11y-guidance/wct/headings/make-headings-accessible.html

---

## 3. Heading Best Practices

### Definitions

**Core Definition**
Heading best practices are authoring guidelines for using heading elements semantically, hierarchically, and consistently to maximise readability, accessibility, and search-engine understanding.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG techniques H42 and H69, and search-engine guidance. They encompass using a single `<h1>` per page, maintaining a non-skipping hierarchy, using headings for structure rather than styling, writing descriptive headings, and keeping headings concise.

**Beginner-Friendly Explanation**
Best practices are the "good habits" for headings. They ensure your headings help users and search engines understand your page without causing confusion.

---

### Purposes

- **To maximise accessibility** through a logical, navigable outline.
- **To improve SEO** by clearly signalling content structure and topics.
- **To enhance readability** by breaking content into scannable sections.
- **To maintain consistency** across pages and components.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use one `<h1>` per page** | The `h1` describes the page's main topic; multiple `h1` elements are allowed but not best practice. |
| **Do not skip heading levels** | Use `h1` → `h2` → `h3` in sequence. |
| **Use headings for structure, not styling** | Do not use an `h3` just to make text bold or large; use CSS. |
| **Write descriptive headings** | A heading should clearly describe the content that follows. |
| **Keep headings concise** | Avoid vague labels and keyword stuffing. |
| **Place headings at the start of sections** | Each section should begin with a heading. |
| **Avoid empty headings** | Completely empty headings are ignored by assistive technologies and fail conformance. |

#### Syntax Rules

1. **Semantic markup**: Always use `<h1>`–`<h6>` elements, not styled `<div>` or `<span>` elements.
2. **Hierarchy discipline**: Nest headings without skipping levels.
3. **Single `<h1>`**: Use one `<h1>` describing the page's content, similar to the `<title>` element.
4. **Descriptive text**: Headings should be meaningful out of context.

#### Constraints and Limitations

- **Fixed sections** (navigation, sidebars) may use `h2` or `h3` consistently across the site, even if they precede the `h1` in source order.
- **Heading-level offsets** in sectioning elements may alter default rendering, but the semantic level remains the same.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Heading Structure

**Setup Guide**: Save as `best-practice-headings.html` and inspect the outline.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Headings</title>
</head>
<body>
  <header>
    <h1>Company Annual Report</h1>
  </header>

  <main>
    <h2>Financial Highlights</h2>
    <p>Revenue increased by 12% this year.</p>

    <h3>Revenue Breakdown</h3>
    <p>Q4 saw the highest revenue.</p>

    <h3>Operating Costs</h3>
    <p>Costs remained stable.</p>

    <h2>Future Outlook</h2>
    <p>The company plans to expand.</p>
  </main>
</body>
</html>
```

**Expected output**: A clear outline: `h1` → `h2` (Financial Highlights) → `h3` (Revenue Breakdown, Operating Costs) → `h2` (Future Outlook). The `h1` describes the page's main topic.

**Why this output occurs**: The structure follows best practices: one `h1`, a non-skipping hierarchy, and descriptive headings that clearly label their sections.

---

#### Example 2: Incorrect Use of Headings for Styling (Anti-Pattern)

**Setup Guide**: Save as `heading-styling.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Headings for Styling</title>
</head>
<body>
  <h1>Page Title</h1>

  <!-- WRONG: using h3 just because the text should be smaller/bold -->
  <h3>This is not a real subsection</h3>
  <p>This text is not part of a subsection, but the h3 makes it look like one.</p>

  <h2>A Real Section</h2>
  <p>Content.</p>
</body>
</html>
```

**Expected output**: The `h3` appears visually smaller than the `h1`, but the outline shows a gap where an `h2` should be. Users relying on headings are misled about the content structure.

**Why this output occurs**: The `h3` is used for visual formatting rather than structural meaning. This creates a misleading content outline and fails the principle that headings must be used for structure, not styling.

---

### Real-World Cases

1. **Content management systems**: Templates enforce a single `h1` and non-skipping hierarchy to maintain accessibility and SEO.
2. **Government websites**: WCAG compliance requires proper heading structure; audits check for skipped levels and styling-only headings.
3. **Design systems**: Heading components are designed to enforce semantic levels independent of visual size.

---

### References

- MDN Web Docs — `<h1>`–`<h6>`: Usage notes — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements#usage_notes
- W3C — Technique H69: Providing heading elements at the beginning of each section of content — https://www.w3.org/WAI/WCAG22/Techniques/html/H69
- Government NZ — Make headings accessible — https://govtnz.github.io/web-a11y-guidance/wct/headings/make-headings-accessible.html
- Acquia — All heading elements are not used for formatting — http://dev-docs.acquia.com/web-governance/all-heading-elements-are-not-used-formatting

---

## 4. Heading Accessibility

### Definitions

**Core Definition**
Heading accessibility refers to the use of heading elements in ways that allow assistive technologies — especially screen readers — to present the page's structure, announce heading levels, and enable efficient navigation between sections.

**Technical Definition**
Heading accessibility is grounded in WCAG Success Criterion 1.3.1 (Info and Relationships) and 2.4.10 (Section Headings). Technique H42 (Using `h1`–`h6` to identify headings) is a sufficient technique for 1.3.1; it allows assistive technologies to present the heading status of text, announce the heading level, and navigate by heading markup. Technique H69 (Providing heading elements at the beginning of each section of content) is a sufficient technique for 2.4.1 (Bypass Blocks) and 2.4.10 (Section Headings). WebAIM's Screen Reader User Survey #9 found that 85.7% of respondents find heading levels very or somewhat useful for navigation.

**Beginner-Friendly Explanation**
Heading accessibility means using headings properly so that people who use screen readers can understand your page's structure and jump directly to the section they need. Screen readers announce headings with their level and allow users to press keys to move between headings — like a built-in table of contents.

---

### Purposes

- **To enable screen-reader navigation** by heading level.
- **To convey document structure** programmatically (WCAG 1.3.1).
- **To satisfy WCAG 2.4.10** (Section Headings) and 2.4.1 (Bypass Blocks).
- **To support cognitive accessibility** by chunking content into clearly labelled sections.

---

### Syntax Rules and Structure

#### Complete General Syntax (Accessible Heading Structure)

```html
<h1>Page Title</h1>
<h2>Section One</h2>
<h3>Subsection A</h3>
<h3>Subsection B</h3>
<h2>Section Two</h2>
```

**Component breakdown**

| Requirement | Description |
|---|---|
| Semantic markup | Use `<h1>`–`<h6>`, not styled `<div>` or `<span>`. |
| Non-skipping hierarchy | Use levels in sequence. |
| Descriptive headings | Headings must describe the content that follows. |
| Single `<h1>` | The `h1` represents the page's main topic. |

#### Syntax Rules

1. **Always use semantic heading markup**: never fake headings with styled text.
2. **Maintain a logical hierarchy**: do not skip levels.
3. **Keep one `<h1>` per page**: the `h1` conventionally represents the page's main topic.
4. **Use headings consistently**: fixed sections (navigation, sidebars) may use `h2` or `h3` consistently across the site.

#### Constraints and Limitations

- **Skipping levels is not a strict WCAG failure** as long as each heading is marked up as a heading and assigned a level consistent with its relative position.
- **Empty headings** are ignored by assistive technologies and fail conformance.
- **Multiple `<h1>` elements** are allowed by the standard when not nested but can confuse screen-reader users; one `<h1>` is best practice.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible Article Structure

**Setup Guide**: Save as `accessible-headings.html`. Test with NVDA, JAWS, or VoiceOver.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Headings</title>
</head>
<body>
  <article>
    <h1>How to Structure Accessible Content</h1>

    <h2>Why Headings Matter</h2>
    <p>Headings provide a navigable outline for screen-reader users.</p>

    <h3>Screen-Reader Navigation</h3>
    <p>Users can jump between headings using keyboard shortcuts.</p>

    <h3>Visual Scanning</h3>
    <p>Sighted users also benefit from clear section titles.</p>

    <h2>Best Practices</h2>
    <p>Use one h1 and avoid skipping levels.</p>
  </article>
</body>
</html>
```

**Expected output**: A screen reader announces the `h1` as "heading level 1", the `h2` sections as "heading level 2", and the `h3` subsections as "heading level 3". Users can navigate by heading level.

**Why this output occurs**: The semantic heading markup allows assistive technologies to present the heading status and level. Technique H42 ensures that screen readers can recognise and announce headings correctly.

---

#### Example 2: Empty Heading (Accessibility Failure)

**Setup Guide**: Save as `empty-heading.html`. Inspect with a screen reader.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Empty Heading</title>
</head>
<body>
  <!-- FAIL: empty heading -->
  <h2></h2>
  <p>Content under an empty heading.</p>
</body>
</html>
```

**Expected output**: The empty heading is ignored by assistive technologies; screen-reader users hear no heading and may not realise a new section was intended.

**Why this output occurs**: Completely empty headings fail Technique H42 and are consistently ignored by assistive technologies, even though they may still affect the visual layout.

---

### Real-World Cases

1. **Government and public-sector websites**: WCAG compliance requires semantic heading markup and logical hierarchy.
2. **Screen-reader user testing**: Users frequently navigate by headings; poor heading structure is one of the most common accessibility complaints.
3. **Cognitive accessibility**: Clear headings help users with cognitive disabilities understand and navigate content.

---

### References

- W3C — Technique H42: Using h1-h6 to identify headings — https://www.w3.org/WAI/WCAG21/Techniques/html/H42
- W3C — Technique H69: Providing heading elements at the beginning of each section of content — https://www.w3.org/WAI/WCAG22/Techniques/html/H69
- Government NZ — Make headings accessible — https://govtnz.github.io/web-a11y-guidance/wct/headings/make-headings-accessible.html
- W3C — ACT Rule: Heading has non-empty accessible name — https://www.w3.org/WAI/standards-guidelines/act/rules/heading-non-empty/

---

## 5. Heading SEO

### Definitions

**Core Definition**
Heading SEO is the practice of using heading elements strategically to help search engines understand the topic, structure, and relevance of a page's content, thereby improving search visibility.

**Technical Definition**
Search engines use headings to interpret the topic and structure of content. The `h1` usually signals the page's main topic; `h2` and `h3` headings introduce related subtopics, providing structure and context. Headings also support **semantic SEO** by naturally containing related terms and concepts that reinforce the page's overall topic. Search engines and AI-driven search systems use clean heading structure to extract accurate answers.

**Beginner-Friendly Explanation**
Heading SEO means using headings to tell search engines what your page is about. Your `h1` should clearly state the main topic, and your `h2` and `h3` headings should organise related subtopics. Good headings help search engines rank your page for relevant queries.

---

### Purposes

- **To signal the page's main topic** through the `h1`.
- **To organise related subtopics** through `h2` and `h3` headings.
- **To support semantic SEO** with naturally related terms and concepts.
- **To help AI-driven search systems** extract accurate answers.
- **To improve readability and engagement**, which are quality signals.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **One clear `h1`** | The `h1` should state the page's main topic and primary keyword. |
| **Logical hierarchy** | Use `h2` for main sections and `h3` for subsections. |
| **Descriptive headings** | Avoid vague labels; headings should clearly describe their sections. |
| **Natural keyword use** | Include relevant secondary keywords in `h2` and `h3` headings without stuffing. |
| **Do not skip levels** | A clean hierarchy helps search engines interpret content correctly. |

#### Syntax Rules

1. **The `h1` should align with the page's primary topic**: it is the strongest heading signal for search engines.
2. **Use headings to structure content logically**: this supports both human readers and crawlers.
3. **Avoid keyword stuffing**: unnatural repetition can harm trust and rankings.
4. **Keep headings concise and meaningful**: long or vague headings reduce clarity.

#### Constraints and Limitations

- **Headings alone do not guarantee rankings**: they are one signal among many.
- **AI-driven search systems** may interpret headings differently than traditional crawlers, but clean structure remains beneficial.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: SEO-Optimised Heading Structure

**Setup Guide**: Save as `seo-headings.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Wireless Headphones Buying Guide</title>
</head>
<body>
  <!-- h1 states the main topic and primary keyword -->
  <h1>Wireless Headphones Buying Guide</h1>

  <!-- h2 sections cover major subtopics -->
  <h2>Types of Wireless Headphones</h2>
  <h3>Over-Ear Headphones</h3>
  <h3>In-Ear Headphones</h3>

  <h2>Key Features to Consider</h2>
  <h3>Battery Life</h3>
  <h3>Noise Cancellation</h3>

  <h2>Top Brands Compared</h2>
  <p>Comparison content.</p>
</body>
</html>
```

**Expected output**: Search engines interpret the page as being about "wireless headphones," with clear subtopics for types, features, and brands. The structure supports both traditional and AI-driven search systems.

**Why this output occurs**: The `h1` signals the main topic; the `h2` and `h3` headings organise related subtopics in a logical hierarchy, supporting semantic SEO and clean content extraction.

---

### Real-World Cases

1. **E-commerce product pages**: `h1` states the product name; `h2` covers features, specifications, and reviews.
2. **Blog posts**: `h1` is the post title; `h2` sections cover major points; `h3` covers subpoints.
3. **Category pages**: `h1` names the category; `h2` groups related products or topics.

---

### References

- Yoast — How to use headings on your site — https://yoast.com/how-to-use-headings-on-your-site/
- Semrush — What Are Heading Tags? & Why They're Important for SEO — https://www.semrush.com/blog/heading-tags/
- Prerender — Heading elements — https://docs.prerender.io/article/106-heading-elements

---

## 6. Heading Pitfalls

### Definitions

**Core Definition**
Heading pitfalls are common mistakes in heading usage that degrade accessibility, SEO, or readability. They include skipping levels, using headings for styling, empty headings, multiple `h1` elements, and vague or keyword-stuffed labels.

**Technical Definition**
Heading pitfalls are deviations from semantic and hierarchical heading use that create barriers for assistive technologies or reduce the clarity of the document outline. While some pitfalls (such as skipping levels) are not strict WCAG failures, they cause usability problems for screen-reader users and weaken the semantic structure of the page. Other pitfalls, such as using headings purely for visual formatting, violate the principle that headings must be used for structure, not styling.

**Beginner-Friendly Explanation**
Heading pitfalls are the common ways people get headings wrong. They include using headings just to make text bigger, skipping levels, leaving headings empty, using too many `h1` elements, or writing vague headings like "More" instead of "Pricing Details."

---

### Purposes

- **To identify and avoid common heading mistakes**.
- **To prevent accessibility regressions** caused by improper heading use.
- **To maintain SEO clarity** by avoiding misleading or vague headings.
- **To guide content reviews and audits**.

---

### Common Pitfalls

| Pitfall | Explanation |
|---|---|
| **Skipping heading levels** | Jumping from `h1` to `h3` creates a gap in the outline and confuses screen-reader users. |
| **Using headings for styling** | Using an `h3` just to make text bold or large misleads users who rely on headings for navigation. |
| **Empty headings** | `<h2></h2>` is ignored by assistive technologies and fails conformance. |
| **Multiple `<h1>` elements** | More than one `h1` on a page can confuse users and dilute the page's main topic signal. |
| **Vague headings** | Headings like "More" or "Info" fail to describe the content that follows. |
| **Keyword stuffing in headings** | Overloading headings with keywords harms readability and trust. |
| **Headings not at section start** | A section should begin with its heading; misplaced headings break the outline. |
| **Using `<div>` or `<span>` instead of headings** | Styled text without heading markup is invisible to screen-reader navigation. |

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Skipped Levels and Styling-Only Headings

**Setup Guide**: Save as `pitfalls.html`. Inspect with an accessibility checker.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Heading Pitfalls</title>
</head>
<body>
  <h1>Main Title</h1>

  <!-- PITFALL: skipped level (h1 → h3) -->
  <h3>Skipped h2</h3>
  <p>This heading skips level 2.</p>

  <!-- PITFALL: heading used only for visual styling -->
  <h4>Not a real subsection</h4>
  <p>This text is bold and large, but it is not a logical subsection.</p>

  <!-- PITFALL: empty heading -->
  <h2></h2>
  <p>Content under an empty heading.</p>
</body>
</html>
```

**Expected output**: The visual rendering may look fine, but an accessibility checker or screen reader reveals: a missing `h2` level, a misleading `h4` with no logical parent `h3`, and an empty heading that is ignored by assistive technologies.

**Why this output occurs**: Each pitfall breaks a different aspect of heading semantics: skipping levels breaks hierarchy, styling-only headings misrepresent structure, and empty headings provide no navigable content.

---

### Real-World Cases

1. **Content migrations**: Legacy pages often have skipped levels or headings used for styling; audits reveal these during accessibility reviews.
2. **Template errors**: CMS templates may generate multiple `h1` elements or empty headings when fields are left blank.
3. **Marketing pages**: Keyword-stuffed headings can reduce trust and readability.

---

### References

- W3C — Failure Example: A heading used only for visual effect — https://www.w3.org/WAI/WCAG21/Techniques/failures/F2
- Government NZ — Make headings accessible — https://govtnz.github.io/web-a11y-guidance/wct/headings/make-headings-accessible.html
- W3C — ACT Rule: Heading has non-empty accessible name — https://www.w3.org/WAI/standards-guidelines/act/rules/heading-non-empty/
- Acquia — Is the heading descriptive enough? — https://docs.acquia.com/cloud-platform/accessibility/heading-descriptive/

---

## Summary Table of Heading Concepts

| Concept | Key Takeaway |
|---|---|
| **`<h1>`–`<h6>`** | Six heading levels; `h1` is highest, `h6` is lowest. Content model is phrasing content. |
| **Heading hierarchy** | Nest headings without skipping levels; `h3` is a subsection of `h2`, which is a subsection of `h1`. |
| **Best practices** | One `h1` per page, descriptive headings, no styling-only headings, headings at section starts. |
| **Accessibility** | Semantic markup enables screen-reader navigation; WCAG 1.3.1 and 2.4.10; Technique H42 and H69. |
| **SEO** | `h1` signals the main topic; `h2`/`h3` organise subtopics; clean structure supports semantic SEO. |
| **Pitfalls** | Skipped levels, styling-only headings, empty headings, multiple `h1`s, vague labels, keyword stuffing. |

---

## Final Notes

- **Semantic headings are essential**: They are the backbone of document structure for both assistive technologies and search engines.
- **One `<h1>` per page is best practice**: While multiple `h1` elements are technically allowed when not nested, a single descriptive `h1` provides the clearest signal.
- **Do not skip levels**: While not a strict WCAG failure, skipping levels confuses screen-reader users and weakens the outline.
- **Headings are not styling tools**: Use CSS for visual presentation; use headings for structure.
- **Test with assistive technologies**: Automated checkers catch some issues, but screen-reader testing is essential to verify heading navigation.

---

This cheat sheet has covered headings — the `<h1>`–`<h6>` elements, heading hierarchy, best practices, accessibility, SEO, and pitfalls — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate heading usage against the WHATWG HTML Living Standard, WCAG techniques, and search-engine guidance.