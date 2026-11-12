# Unordered Lists — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Unordered lists are HTML structures that group a set of related items whose order is not meaningful. They are created with the `<ul>` element, which contains one or more `<li>` list-item elements. Browsers typically render unordered lists with bullet markers, but the semantics are "a group of items with no defined sequence", not "a bulleted list".

**Technical Definition**
In the WHATWG HTML Living Standard, `<ul>` belongs to the **flow content** and **palpable content** categories. Its content model is **zero or more `<li>` elements** and script-supporting elements (`<script>`, `<template>`). The `<li>` element has no category of its own; its role is defined by its parent, and its content model is **flow content** (prose content) when the parent is `<ul>` or `<ol>`, and **phrasing content** when the parent is `<menu>`. In the accessibility tree, `<ul>` maps to the ARIA role `list`, and `<li>` maps to `listitem`. The `compact` and `type` attributes are obsolete; visual presentation is controlled by CSS (`list-style-type`, `list-style-position`, and `list-style-image`).

**Beginner-Friendly Explanation**
An unordered list is a group of related things where the order does not matter. You wrap the group in `<ul>` and each item in `<li>`. Browsers show bullets by default, but the important part is the meaning: screen readers announce "list, 4 items" and let users jump between them. Use `<ul>` for shopping lists, feature lists, navigation menus, and any collection where sequence is irrelevant.

---

### Key Characteristics

- **Order is not meaningful**: If reordering the items changes the meaning, use `<ol>` instead.
- **Direct children are `<li>` elements**: The `<ul>` element accepts only `<li>` and script-supporting elements as direct children.
- **Automatic bullet markers**: Browsers apply `list-style-type: disc` by default; CSS can change or remove markers.
- **Accessibility semantics**: `<ul>` is exposed as `list` and `<li>` as `listitem`; screen readers announce the item count and allow list navigation.
- **Nesting via `<li>`**: Sub-lists must be placed inside a parent `<li>` element, never as a direct child of another `<ul>`.
- **Universal support**: `<ul>` and `<li>` are supported in every browser and every screen reader.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements and content models.
2. **Flow vs. phrasing content**: understanding what may appear inside a list item.
3. **CSS list properties**: awareness of `list-style-type`, `list-style-position`, and `margin`/`padding` on lists.
4. **Accessibility basics**: understanding that screen readers expose list semantics and item counts.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS** | `list-style-type`, `list-style-position`, and `display: flex/grid` on `<ul>` control presentation. |
| **Accessibility (WCAG)** | WCAG 1.3.1 (Info and Relationships) and Technique H48 require semantic list markup. |
| **Navigation menus** | `<ul>` inside `<nav>` is the standard pattern for site navigation. |
| **SEO** | Lists can be eligible for featured snippets, FAQ rich results, and other search features. |
| **JavaScript** | `<ul>` and `<li>` are common targets for dynamic list rendering and event delegation. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<ul>`** — the unordered list container
2. **`<li>`** — the list item
3. **Nested unordered lists** — sub-lists inside list items
4. **Unordered list best practices** — consolidated guidelines

---

## 1. `<ul>` — The Unordered List Container

### Definitions

**Core Definition**
The `<ul>` element represents an unordered list of items — a list in which changing the order of the items would not change the meaning.

**Technical Definition**
`<ul>` belongs to the **flow content** and **palpable content** categories. Its content model is **zero or more `<li>` elements** plus script-supporting elements. The element has no element-specific content attributes; `compact` and `type` are obsolete. In the accessibility tree, `<ul>` is exposed as the ARIA role `list`. By default, browsers apply `display: block`, `list-style-type: disc`, and padding to reserve space for markers.

**Beginner-Friendly Explanation**
`<ul>` is the container for an unordered list. You put your items inside it as `<li>` elements, and the browser shows them as a bulleted group.

---

### Purposes

- **To group items without a meaningful order** — a shopping list, a feature list, a set of tags.
- **To provide semantic list structure** so screen readers can announce the number of items and allow list navigation.
- **To serve as the structural base for navigation menus** and other link groups.
- **To enable CSS-driven layouts** such as flexbox navigation bars and grid-based card lists.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ul>
  <li>Item one</li>
  <li>Item two</li>
  <li>Item three</li>
</ul>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<ul>` | The unordered list container. |
| `<li>` | One or more list items; the only permitted non-script direct children. |
| `</ul>` | The end tag. |

#### Syntax Rules

1. **Content model is zero or more `<li>` elements** plus script-supporting elements.
2. **Use `<ul>` when order does not matter**; use `<ol>` when it does.
3. **The `type` and `compact` attributes are obsolete**; use CSS `list-style-type` instead.
4. **Place nested lists inside an `<li>`**, never as a direct child of `<ul>`.

#### Constraints and Limitations

- **`<ul>` must not contain elements other than `<li>`** (and `<script>`/`<template>`) as direct children.
- **Empty `<ul>` elements are not palpable** and should be avoided.
- **Styling `<ul>` for layout** can remove list semantics if `list-style: none` is applied without a fallback; Safari historically removed list semantics when `list-style: none` was applied, though this has been corrected in modern versions.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Feature List

**Setup Guide**: Save as `ul-basic.html` and open it in a browser. Inspect the accessibility tree to see the list role.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Unordered List</title>
</head>
<body>
  <!-- The ul groups related items with no meaningful order -->
  <h2>Product Features</h2>
  <ul>
    <!-- Each li is a single item in the list -->
    <li>Lightweight design</li>
    <li>Long battery life</li>
    <li>Water-resistant</li>
    <li>Two-year warranty</li>
  </ul>
</body>
</html>
```

**Expected output**: A bulleted list of four features. A screen reader announces "list, 4 items" and allows navigation between items.

**Why this output occurs**: The `<ul>` element creates a list container, and the browser applies default `list-style-type: disc` styling. The accessibility tree exposes the list role and item count, which screen readers use for navigation.

---

#### Example 2: A Navigation Menu Built from `<ul>`

**Setup Guide**: Save as `ul-nav.html`. Tab through the page to verify keyboard access.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>List-Based Navigation</title>
  <style>
    /* Remove default list styling and lay items out horizontally */
    nav ul {
      list-style: none;   /* remove bullet markers */
      display: flex;      /* lay items out in a row */
      gap: 16px;          /* spacing between items */
      padding: 0;
      margin: 0;
    }
    nav a { text-decoration: none; }
  </style>
</head>
<body>
  <!-- The ul provides list semantics for the navigation links -->
  <nav aria-label="Primary">
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</body>
</html>
```

**Expected output**: A horizontal navigation bar with three links and no bullet markers. Screen readers announce "Primary navigation, list, 3 items" and expose each link as a list item.

**Why this output occurs**: The `<ul>` retains its semantic list role even when CSS removes the visual markers and changes the layout to flexbox. This is the standard navigation pattern: semantic structure with CSS presentation.

---

### Real-World Cases

1. **Feature and benefit lists**: Marketing pages list product features with `<ul>`.
2. **Navigation menus**: `<ul>` inside `<nav>` is the standard site navigation pattern.
3. **Shopping and ingredient lists**: Items without a required order.

---

### References

- MDN Web Docs — `<ul>`: The Unordered List element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul
- WHATWG — The `ul` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-ul-element
- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG21/Techniques/html/H48

---

## 2. `<li>` — The List Item Element

### Definitions

**Core Definition**
The `<li>` element represents a single item in a list. It must be a child of an `<ul>`, `<ol>`, or `<menu>` element, and its content model depends on which of those parents it belongs to.

**Technical Definition**
`<li>` has no content category of its own; its role is defined by its parent list. Its content model is **flow content** (prose content) when the parent is `<ul>` or `<ol>`, and **phrasing content** when the parent is `<menu>`. The `value` attribute is permitted only when the parent is `<ol>`, where it sets the item's ordinal value. In the accessibility tree, `<li>` maps to the ARIA role `listitem`. The `li` end tag may be omitted when the item is immediately followed by another `<li>` or when there is no more content in the parent.

**Beginner-Friendly Explanation**
`<li>` is one item in a list. It always lives inside a `<ul>`, `<ol>`, or `<menu>`. Its meaning depends on its parent: inside `<ul>` it is a bulleted item; inside `<ol>` it is a numbered step.

---

### Purposes

- **To represent a single item in a list**.
- **To provide the content of unordered, ordered, or menu lists**.
- **To serve as the container for nested sub-lists**.
- **To allow custom numbering** in ordered lists via the `value` attribute (not applicable to `<ul>`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ul>
  <li>Item content</li>
</ul>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<li>` | The list item start tag. |
| Content | Flow content when the parent is `<ul>` or `<ol>`; phrasing content when the parent is `<menu>`. |
| `</li>` | The end tag; may be omitted before another `<li>` or at the end of the parent. |

#### Syntax Rules

1. **`<li>` must be a child of `<ul>`, `<ol>`, or `<menu>`** (or a script-supporting element in a list context).
2. **Content model is flow content** when the parent is `<ul>` or `<ol>`; `<li>` may contain paragraphs, nested lists, and other flow-level elements.
3. **The `value` attribute is only valid when the parent is `<ol>`**; it has no effect inside `<ul>`.
4. **The `li` end tag may be omitted** when followed by another `<li>` or when there is no more content in the parent.

#### Constraints and Limitations

- **`<li>` outside a list parent has no defined list relationship**.
- **`<li>` cannot be a direct child of a `<ul>` inside another `<ul>`**; nested lists go inside an `<li>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `<li>` with Rich Flow Content

**Setup Guide**: Save as `li-rich.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Rich List Items</title>
</head>
<body>
  <ul>
    <!-- Each li can contain flow content: headings, paragraphs, links -->
    <li>
      <h3>Wireless Headphones</h3>
      <p>Lightweight design with 30-hour battery life.</p>
      <a href="/products/wireless-headphones">View details</a>
    </li>
    <li>
      <h3>Charging Case</h3>
      <p>Compact case with USB-C charging.</p>
      <a href="/products/charging-case">View details</a>
    </li>
  </ul>
</body>
</html>
```

**Expected output**: A list of two products, each with a heading, a paragraph, and a link inside its `<li>`.

**Why this output occurs**: When the parent is `<ul>`, `<li>`'s content model is flow content, so headings, paragraphs, and links are all valid children. This enables rich product or card lists without losing list semantics.

---

#### Example 2: End-Tag Omission

**Setup Guide**: Save as `li-omission.html` and inspect the parsed DOM.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Li Tag Omission</title>
</head>
<body>
  <ul>
    <!-- The </li> end tags are omitted; the parser closes them automatically -->
    <li>First item
    <li>Second item
    <li>Third item
  </ul>
</body>
</html>
```

**Expected output**: Three list items render correctly; the DOM inspector shows that each `<li>` was automatically closed before the next one.

**Why this output occurs**: The `<li>` end tag may be omitted when the item is immediately followed by another `<li>`. The parser applies the omission rule and creates sibling list items.

---

### Real-World Cases

1. **Navigation menus**: Each navigation link is wrapped in an `<li>` inside a `<ul>`.
2. **Product cards**: Each card is an `<li>` containing a heading, image, price, and link.
3. **Comment threads**: Each comment is an `<li>` in a nested `<ul>`.

---

### References

- MDN Web Docs — `<li>`: The List Item element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/li
- WHATWG — The `li` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-li-element
- W3C — HTML 5.2: The `li` element — https://www.w3.org/TR/html52/grouping-content.html#the-li-element

---

## 3. Nested Unordered Lists — Sub-Lists Inside List Items

### Definitions

**Core Definition**
A nested unordered list is a `<ul>` element placed inside an `<li>` element of another list. It represents a sub-list of items belonging to the parent item.

**Technical Definition**
Nesting is expressed structurally: a `<ul>` is a child of an `<li>`, and that `<ul>` contains its own `<li>` children. This preserves the parent–child relationship in the accessibility tree, so screen readers announce the nesting depth and the number of items at each level. The common mistake is placing a `<ul>` directly inside another `<ul>` (invalid) rather than inside an `<li>` (valid).

**Beginner-Friendly Explanation**
A nested unordered list is a list inside a list. For example, a "Fruits" item might contain a sub-list of "Apples", "Bananas", and "Cherries". The sub-list goes inside the "Fruits" list item, not directly inside the outer list.

---

### Purposes

- **To represent hierarchical relationships** within a group of items.
- **To create multi-level navigation menus** with dropdown sub-menus.
- **To express categories and sub-categories** in a list context.
- **To preserve the nesting structure** for assistive technologies and search engines.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ul>
  <li>
    Parent item
    <ul>
      <li>Child item</li>
      <li>Child item</li>
    </ul>
  </li>
</ul>
```

**Component breakdown**

| Component | Description |
|---|---|
| Outer `<ul>` | The top-level list. |
| Parent `<li>` | The item that owns the sub-list. |
| Inner `<ul>` | The nested list; must be a child of the `<li>`, not of the outer `<ul>`. |
| Inner `<li>` | The child items. |

#### Syntax Rules

1. **The nested `<ul>` must be a child of an `<li>`**, not a direct child of the outer `<ul>`.
2. **The nested `<ul>` should follow the parent item's content** (text, link, or other content) within the same `<li>`.
3. **Each level of nesting is exposed as a nested list** in the accessibility tree, with its own item count.

#### Constraints and Limitations

- **A `<ul>` directly inside another `<ul>` is invalid** and produces parser error recovery.
- **Very deep nesting** can be difficult for screen-reader users to navigate; keep nesting to two or three levels where possible.
- **CSS must not collapse the visual hierarchy**; indentation or other visual cues should reflect the nesting depth.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Two-Level Category List

**Setup Guide**: Save as `ul-nested.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested Unordered List</title>
</head>
<body>
  <ul>
    <li>
      <!-- Parent item contains the sub-list -->
      Fruits
      <ul>
        <li>Apples</li>
        <li>Bananas</li>
        <li>Cherries</li>
      </ul>
    </li>
    <li>
      Vegetables
      <ul>
        <li>Carrots</li>
        <li>Broccoli</li>
      </ul>
    </li>
  </ul>
</body>
</html>
```

**Expected output**: A bulleted list with two top-level items, each containing a nested bulleted sub-list.

**Why this output occurs**: The inner `<ul>` is placed inside the parent `<li>`, so the browser renders it as a sub-list. The accessibility tree exposes the nesting, and screen readers announce the parent item followed by the nested list and its item count.

---

#### Example 2: Invalid Nesting (Anti-Pattern)

**Setup Guide**: Save as `ul-nested-invalid.html` and inspect the parsed DOM.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Invalid Nesting</title>
</head>
<body>
  <!-- INVALID: the inner ul is a direct child of the outer ul -->
  <ul>
    <li>Fruits</li>
    <ul>
      <li>Apples</li>
      <li>Bananas</li>
    </ul>
  </ul>
</body>
</html>
```

**Expected output**: The browser renders something, but the DOM inspector reveals that the parser has restructured the markup; the inner list is not associated with the "Fruits" item, and the nesting relationship is lost.

**Why this output occurs**: `<ul>` accepts only `<li>` and script-supporting elements as direct children. A direct `<ul>` child violates the content model and triggers parser error recovery.

---

### Real-World Cases

1. **Multi-level navigation menus**: Top-level links with dropdown sub-menus.
2. **Documentation tables of contents**: Chapters with nested sections.
3. **Category trees**: Product categories with sub-categories.

---

### References

- MDN Web Docs — `<ul>`: Nesting lists — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul#nesting_lists
- WHATWG — The `ul` element: Content model — https://html.spec.whatwg.org/dev/grouping-content.html#the-ul-element

---

## 4. Unordered List Best Practices

### Definitions

**Core Definition**
Unordered list best practices are guidelines for choosing `<ul>` appropriately, structuring list items correctly, nesting safely, and presenting lists accessibly — so that list semantics are preserved for assistive technologies and search engines.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG 1.3.1 (Info and Relationships), and W3C Technique H48 (Using `ol`, `ul` and `dl` for lists or groups of links). They encompass choosing `<ul>` only when order is irrelevant, using `<li>` for every item, nesting sub-lists inside `<li>`, avoiding non-`<li>` direct children, preserving list semantics when removing markers with CSS, and ensuring that visual presentation matches the underlying structure.

**Beginner-Friendly Explanation**
Best practices help you build lists that are correct and accessible: use `<ul>` only when order does not matter, put every item in an `<li>`, nest sub-lists inside the parent `<li>`, and do not break the list semantics when you style away the bullets.

---

### Purposes

- **To ensure semantic correctness** and valid markup.
- **To preserve accessibility** for screen-reader users who rely on list semantics and item counts.
- **To support SEO** by providing clear structure that search engines can parse.
- **To maintain visual and structural consistency** across a site.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `<ul>` only when order is irrelevant** | If reordering changes the meaning, use `<ol>`. |
| **Every item must be an `<li>`** | Do not use `<div>` or `<span>` as direct children of `<ul>`. |
| **Nest sub-lists inside `<li>`** | A nested `<ul>` must be a child of the parent `<li>`. |
| **Use `<ul>` inside `<nav>` for navigation** | The standard pattern for site menus. |
| **Do not remove list semantics with CSS** | `list-style: none` removes bullets; ensure the list role is preserved or use alternative styling. |
| **Use `<ul>` for groups of links** | Technique H48 recommends lists for groups of related links. |
| **Keep nesting shallow** | Two or three levels are manageable; deeper nesting is harder for screen-reader users. |
| **Do not use empty `<ul>` elements** | Empty lists are not palpable and provide no value. |

#### Syntax Rules

1. **Semantic correctness**: `<ul>` for unordered groups, `<ol>` for ordered sequences, `<dl>` for term–definition pairs.
2. **Structural correctness**: `<li>` for every item; nested `<ul>` inside `<li>`.
3. **Accessibility preservation**: When styling lists, ensure the list role and item count remain exposed to assistive technologies.

#### Constraints and Limitations

- **`list-style: none` can remove list semantics in some browsers**, particularly older Safari; adding `role="list"` to the `<ul>` restores the semantics.
- **CSS `display: flex` or `display: grid` on `<ul>`** changes the layout but should preserve the list role in modern browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible Styled Navigation List

**Setup Guide**: Save as `best-practice-nav.html`. Inspect the accessibility tree to confirm the list role is preserved.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Navigation</title>
  <style>
    nav ul {
      list-style: none;   /* remove bullets */
      display: flex;
      gap: 16px;
      padding: 0;
      margin: 0;
    }
    nav a { text-decoration: none; }
  </style>
</head>
<body>
  <nav aria-label="Primary">
    <!-- role="list" restores list semantics when list-style: none is used -->
    <ul role="list">
      <li><a href="/">Home</a></li>
      <li><a href="/products">Products</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</body>
</html>
```

**Expected output**: A horizontal navigation bar with no bullets. Screen readers still announce "Primary navigation, list, 3 items" because the `role="list"` restores the semantics that `list-style: none` can remove in some browsers.

**Why this output occurs**: The `role="list"` attribute explicitly exposes the list semantics, ensuring that screen readers announce the item count even after the visual markers are removed.

---

#### Example 2: Correct vs Incorrect List Structure

**Setup Guide**: Save as `best-practice-structure.html`. Run an HTML validator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>List Structure</title>
</head>
<body>
  <!-- GOOD: every item is an li; nesting is inside li -->
  <ul>
    <li>Fruits
      <ul>
        <li>Apples</li>
        <li>Bananas</li>
      </ul>
    </li>
    <li>Vegetables</li>
  </ul>

  <!-- BAD: div as a direct child of ul -->
  <!--
  <ul>
    <div>Fruits</div>
    <div>Vegetables</div>
  </ul>
  -->

  <!-- BAD: ul directly inside ul -->
  <!--
  <ul>
    <li>Fruits</li>
    <ul>
      <li>Apples</li>
    </ul>
  </ul>
  -->
</body>
</html>
```

**Expected output**: The good example produces valid, accessible markup. The bad examples fail validation and lose the parent–child list relationship.

**Why this output occurs**: `<ul>` accepts only `<li>` and script-supporting elements as direct children, and nested lists must be inside an `<li>`. Violating these rules triggers parser error recovery and breaks the accessibility tree.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG 1.3.1 compliance requires semantic list markup; audits check for `<div>`-based fake lists.
2. **E-commerce**: Product feature lists and category trees use `<ul>` and nested `<ul>` for structure.
3. **Documentation sites**: Navigation sidebars and tables of contents use `<ul>` with nested sub-lists.

---

### References

- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG21/Techniques/html/H48
- MDN Web Docs — `<ul>`: Accessibility — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul#accessibility
- WHATWG — The `ul` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-ul-element
- WebAIM — Creating Accessible Lists — https://webaim.org/techniques/semanticstructure/

---

## Summary Table of Unordered List Concepts

| Concept | Key Takeaway |
|---|---|
| **`<ul>`** | Container for unordered groups; content model is zero or more `<li>` plus script-supporting elements; exposed as `list`. |
| **`<li>`** | One list item; flow content when the parent is `<ul>` or `<ol>`; end tag may be omitted before another `<li>`. |
| **Nested unordered lists** | A nested `<ul>` must be inside an `<li>`, never a direct child of another `<ul>`. |
| **Best practices** | Use `<ul>` only when order is irrelevant; every item must be `<li>`; preserve list semantics when styling. |

---

## Final Notes

- **Order matters**: If reordering the items changes the meaning, use `<ol>` instead of `<ul>`.
- **Every item must be an `<li>`**: `<div>`-based fake lists lose the accessibility semantics that screen readers rely on.
- **Nest inside `<li>`**: A nested `<ul>` belongs inside the parent `<li>`, not directly inside the outer `<ul>`.
- **Preserve semantics when styling**: `list-style: none` can remove list semantics in some browsers; add `role="list"` to restore them.
- **Navigation menus are lists**: The standard pattern is `<nav><ul><li><a>…</a></li></ul></nav>`, which combines landmark and list semantics.
- **Validate and test**: Use HTML validators and accessibility checkers to catch non-`<li>` children and invalid nesting.

---

This cheat sheet has covered unordered lists — the `<ul>` container, the `<li>` list item, nested unordered lists, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate list markup against the WHATWG HTML Living Standard and WCAG 1.3.1 (Info and Relationships) with Technique H48.