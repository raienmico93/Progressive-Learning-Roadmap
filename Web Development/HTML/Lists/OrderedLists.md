# Ordered Lists — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Ordered lists are HTML structures that represent a sequence of items in which the order is meaningful — changing the order would change the meaning of the list or document. They are created with the `<ol>` element, which contains one or more `<li>` list-item elements. Browsers typically render ordered lists with sequential markers (numbers, letters, or Roman numerals), but the semantics are "an intentionally ordered sequence", not "a numbered list".

**Technical Definition**
In the WHATWG HTML Living Standard, `<ol>` belongs to the **flow content** and **palpable content** categories. Its content model is **zero or more `<li>` elements** and script-supporting elements. The `<li>` element's content model is **flow content** when the parent is `<ol>` or `<ul>`, and **phrasing content** when the parent is `<menu>`. The `<ol>` element supports three element-specific attributes: `type` (marker type), `start` (ordinal value of the first item), and `reversed` (boolean; descending order). In the accessibility tree, `<ol>` maps to the ARIA role `list` and `<li>` maps to `listitem`; screen readers announce the item count and, in some cases, the position within the list.

**Beginner-Friendly Explanation**
An ordered list is a group of things where the sequence matters. A recipe's steps, driving directions, or a ranked list of results are all ordered lists. You wrap the group in `<ol>` and each item in `<li>`. Browsers number the items automatically, but the important part is the meaning: screen readers announce "list, 5 items" and let users navigate through the sequence.

---

### Key Characteristics

- **Order is meaningful**: If reordering the items would not change the meaning, use `<ul>` instead.
- **Direct children are `<li>` elements**: The `<ol>` element accepts only `<li>` and script-supporting elements as direct children.
- **Automatic sequential markers**: Browsers apply `list-style-type: decimal` by default; the `type` attribute and CSS `list-style-type` can change the marker style.
- **Custom numbering**: The `start` attribute sets the ordinal value of the first item; the `reversed` boolean attribute numbers the list in descending order.
- **Accessibility semantics**: `<ol>` is exposed as `list` and `<li>` as `listitem`; screen readers announce the item count and support list navigation.
- **Nesting via `<li>`**: Sub-lists must be placed inside a parent `<li>` element, never as a direct child of another `<ol>`.

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
| **CSS** | `list-style-type`, `list-style-position`, and `display: flex/grid` on `<ol>` control presentation. |
| **Accessibility (WCAG)** | WCAG 1.3.1 (Info and Relationships) and Technique H48 require semantic list markup. |
| **Legal and technical documents** | Numbered clauses and steps often require ordered lists with specific numbering. |
| **SEO** | Ordered lists can be eligible for featured snippets and step-by-step rich results. |
| **JavaScript** | `<ol>` and `<li>` are common targets for dynamic list rendering and event delegation. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<ol>`** — the ordered list container
2. **`<li>`** — the list item
3. **`type` attribute** — the marker style
4. **`start` attribute** — the first ordinal value
5. **`reversed` attribute** — descending order
6. **Ordered list best practices** — consolidated guidelines

---

## 1. `<ol>` — The Ordered List Container

### Definitions

**Core Definition**
The `<ol>` element represents an ordered list of items — a list in which the order is intentional, such that changing the order would change the meaning of the list.

**Technical Definition**
`<ol>` belongs to the **flow content** and **palpable content** categories. Its content model is **zero or more `<li>` elements** plus script-supporting elements. The element accepts global attributes plus `type`, `start`, and `reversed`. The `compact` attribute is obsolete and must not be used. In the accessibility tree, `<ol>` is exposed as the ARIA role `list`. By default, browsers apply `display: block` and `list-style-type: decimal`.

**Beginner-Friendly Explanation**
`<ol>` is the container for an ordered list. You put your items inside it as `<li>` elements, and the browser numbers them in sequence.

---

### Purposes

- **To represent a sequence of items** where order matters — steps, directions, ranked results, legal clauses.
- **To provide semantic list structure** so screen readers can announce the item count and allow list navigation.
- **To enable CSS-driven layouts** for numbered navigation and step indicators.
- **To support custom numbering** via `type`, `start`, and `reversed`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ol>
  <li>First step</li>
  <li>Second step</li>
  <li>Third step</li>
</ol>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<ol>` | The ordered list container. |
| `<li>` | One or more list items; the only permitted non-script direct children. |
| `</ol>` | The end tag. |

#### Syntax Rules

1. **Content model is zero or more `<li>` elements** plus script-supporting elements.
2. **Use `<ol>` when order matters**; use `<ul>` when it does not.
3. **The `compact` attribute is obsolete**; use CSS instead.
4. **Place nested lists inside an `<li>`**, never as a direct child of `<ol>`.

#### Constraints and Limitations

- **`<ol>` must not contain elements other than `<li>`** (and `<script>`/`<template>`) as direct children.
- **Empty `<ol>` elements are not palpable** and should be avoided.
- **Screen readers distinguish ordered from unordered lists**; using `<ol>` for non-sequential content misrepresents the structure.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Ordered Sequence

**Setup Guide**: Save as `ol-basic.html` and open it in a browser. Inspect the accessibility tree to confirm the list role.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Ordered List</title>
</head>
<body>
  <!-- The ol groups items where the sequence matters -->
  <h2>How to Make Tea</h2>
  <ol>
    <!-- Each li is one step in the sequence -->
    <li>Boil water.</li>
    <li>Place a tea bag in a cup.</li>
    <li>Pour the hot water over the tea bag.</li>
    <li>Steep for 3–5 minutes.</li>
  </ol>
</body>
</html>
```

**Expected output**: A numbered list (1–4) of steps. A screen reader announces "list, 4 items" and allows navigation between items.

**Why this output occurs**: The `<ol>` element creates an ordered list container, and the browser applies default `list-style-type: decimal` styling. The accessibility tree exposes the list role and item count, which screen readers use for navigation.

---

#### Example 2: An Ordered List with Rich List Items

**Setup Guide**: Save as `ol-rich.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Rich Ordered List</title>
</head>
<body>
  <ol>
    <!-- Each li can contain flow content: headings, paragraphs, links -->
    <li>
      <h3>Preheat the oven</h3>
      <p>Set the oven to 180°C (350°F).</p>
    </li>
    <li>
      <h3>Mix the ingredients</h3>
      <p>Combine flour, sugar, and butter in a bowl.</p>
      <a href="/recipe/details">See full ingredient list</a>
    </li>
  </ol>
</body>
</html>
```

**Expected output**: A numbered list where each step contains a heading, a paragraph, and a link.

**Why this output occurs**: When the parent is `<ol>`, `<li>`'s content model is flow content, so headings, paragraphs, and links are all valid children. This enables rich step-by-step instructions without losing list semantics.

---

### Real-World Cases

1. **Recipe instructions**: Steps that must be followed in order.
2. **Driving directions**: Turn-by-turn navigation.
3. **Legal and technical documents**: Numbered clauses that are referenced by number.
4. **Ranked lists**: Leaderboards and top-ten lists.

---

### References

- MDN Web Docs — `<ol>`: The Ordered List element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ol
- WHATWG — The `ol` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-ol-element
- W3C — HTML 5: The `ol` element — https://w3c.github.io/html-reference/ol.html

---

## 2. `<li>` — The List Item Element

### Definitions

**Core Definition**
The `<li>` element represents a single item in a list. It must be a child of an `<ol>`, `<ul>`, or `<menu>` element, and its content model depends on which of those parents it belongs to.

**Technical Definition**
`<li>` has no content category of its own; its role is defined by its parent list. Its content model is **flow content** (prose content) when the parent is `<ol>` or `<ul>`, and **phrasing content** when the parent is `<menu>`. The `value` attribute is permitted only when the parent is `<ol>`, where it sets the item's ordinal value. In the accessibility tree, `<li>` maps to the ARIA role `listitem`. The `li` end tag may be omitted when the item is immediately followed by another `<li>` or when there is no more content in the parent.

**Beginner-Friendly Explanation**
`<li>` is one item in a list. Inside an `<ol>`, it is a numbered step. The optional `value` attribute lets you override the number for a specific item.

---

### Purposes

- **To represent a single item in an ordered list**.
- **To allow custom numbering** within an ordered list via the `value` attribute.
- **To serve as the container for nested sub-lists**.
- **To provide rich content** (headings, paragraphs, links) within a list item.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ol>
  <li>Item content</li>
</ol>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<li>` | The list item start tag. |
| `value` | Optional integer; overrides the ordinal value for this item (only valid when the parent is `<ol>`). |
| Content | Flow content when the parent is `<ol>` or `<ul>`. |
| `</li>` | The end tag; may be omitted before another `<li>` or at the end of the parent. |

#### Syntax Rules

1. **`<li>` must be a child of `<ol>`, `<ul>`, or `<menu>`**.
2. **Content model is flow content** when the parent is `<ol>` or `<ul>`; `<li>` may contain paragraphs, nested lists, and other flow-level elements.
3. **The `value` attribute is only valid when the parent is `<ol>`**; it sets the ordinal value of that specific item.
4. **The `li` end tag may be omitted** when followed by another `<li>` or when there is no more content in the parent.

#### Constraints and Limitations

- **`<li>` outside a list parent has no defined list relationship**.
- **The `value` attribute has no effect inside `<ul>`**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Numbering with `value`

**Setup Guide**: Save as `li-value.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Custom List Values</title>
</head>
<body>
  <ol>
    <!-- The value attribute overrides the ordinal value for this item -->
    <li value="10">Item numbered 10</li>
    <li>Item numbered 11</li>
    <li value="20">Item numbered 20</li>
    <li>Item numbered 21</li>
  </ol>
</body>
</html>
```

**Expected output**: A numbered list displaying 10, 11, 20, 21.

**Why this output occurs**: The `value` attribute on `<li>` sets the ordinal value for that item. Subsequent items continue from that value, so the list numbers become 10, 11, then reset to 20, then 21.

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
  <ol>
    <!-- The </li> end tags are omitted; the parser closes them automatically -->
    <li>First item
    <li>Second item
    <li>Third item
  </ol>
</body>
</html>
```

**Expected output**: Three list items render correctly; the DOM inspector shows that each `<li>` was automatically closed before the next one.

**Why this output occurs**: The `<li>` end tag may be omitted when the item is immediately followed by another `<li>`. The parser applies the omission rule and creates sibling list items.

---

### Real-World Cases

1. **Recipe steps**: Each step is an `<li>` with the instruction text.
2. **Legal clauses**: Numbered clauses with custom `value` attributes for cross-referencing.
3. **Ranked lists**: Each ranked item is an `<li>` in an `<ol>`.

---

### References

- MDN Web Docs — `<li>`: The List Item element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/li
- WHATWG — The `li` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-li-element

---

## 3. `type` Attribute — The Marker Style

### Definitions

**Core Definition**
The `type` attribute on `<ol>` specifies the kind of marker (numbering style) to use for the list items — Arabic numerals, uppercase or lowercase letters, or uppercase or lowercase Roman numerals.

**Technical Definition**
`type` is an enumerated attribute accepting five values: `1` (decimal, default), `a` (lowercase alphabet), `A` (uppercase alphabet), `i` (lowercase Roman numerals), and `I` (uppercase Roman numerals). The attribute is **not purely presentational**; it carries meaning when the list items are referenced by their number or letter, as in legal or technical documents. CSS `list-style-type` offers more marker types and overrides the `type` attribute in most browsers.

**Beginner-Friendly Explanation**
`type` changes how the list is numbered. `type="a"` gives you a, b, c; `type="A"` gives A, B, C; `type="i"` gives i, ii, iii; and `type="1"` gives 1, 2, 3 (the default). You should use it when the actual numbering style matters — for example, in a legal document where clauses are referenced by letter.

---

### Purposes

- **To specify the marker style** for the list items.
- **To preserve meaningful numbering** in legal, technical, and academic documents where items are referenced by their marker.
- **To support multi-level numbering schemes** (e.g. 1, a, i) across nested lists.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ol type="a">
  <li>First item</li>
  <li>Second item</li>
</ol>
```

**Component breakdown**

| Value | Numbering style | Example |
|---|---|---|
| `1` | Decimal (default) | 1, 2, 3 |
| `a` | Lowercase alphabet | a, b, c |
| `A` | Uppercase alphabet | A, B, C |
| `i` | Lowercase Roman numerals | i, ii, iii |
| `I` | Uppercase Roman numerals | I, II, III |

#### Syntax Rules

1. **`type` accepts one of five values**: `1`, `a`, `A`, `i`, `I`.
2. **The attribute is not deprecated**; it has semantic meaning when the marker type matters.
3. **CSS `list-style-type` overrides `type`** in most browsers; do not use both with conflicting values.

#### Constraints and Limitations

- **CSS offers more marker types** than the HTML `type` attribute; for purely visual styling, use CSS.
- **If the marker type is purely decorative**, prefer CSS `list-style-type`; use the `type` attribute only when the numbering style is meaningful.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Alphabetical Numbering

**Setup Guide**: Save as `ol-type-a.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Alphabetical List</title>
</head>
<body>
  <ol type="a">
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
  </ol>
</body>
</html>
```

**Expected output**: A list numbered a, b, c.

**Why this output occurs**: The `type="a"` attribute instructs the browser to use lowercase letters as markers.

---

#### Example 2: Roman Numerals

**Setup Guide**: Save as `ol-type-i.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Roman Numeral List</title>
</head>
<body>
  <ol type="I">
    <li>Introduction</li>
    <li>Methods</li>
    <li>Results</li>
  </ol>
</body>
</html>
```

**Expected output**: A list numbered I, II, III.

**Why this output occurs**: The `type="I"` attribute instructs the browser to use uppercase Roman numerals as markers.

---

### Real-World Cases

1. **Legal documents**: Clauses numbered with letters or Roman numerals for cross-referencing.
2. **Academic outlines**: Multi-level outlines using 1, a, i, etc.
3. **Technical specifications**: Numbered sections referenced by their marker.

---

### References

- W3C — HTML: The Markup Language: `ol` `type` attribute — https://w3c.github.io/html-reference/ol.html#ol.attrs.type
- MDN Web Docs — `<ol>`: `type` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ol#type

---

## 4. `start` Attribute — The First Ordinal Value

### Definitions

**Core Definition**
The `start` attribute specifies the ordinal value of the first item in an ordered list. Numbering begins at that value and increments (or decrements, with `reversed`) from there.

**Technical Definition**
`start` is an integer content attribute. If present, it must be a valid integer giving the ordinal value of the first list item. The value is always an Arabic numeral, even when `type` is set to letters or Roman numerals. For example, `start="4"` with `type="a"` produces d, e, f.

**Beginner-Friendly Explanation**
`start` changes the number the list begins with. `start="5"` makes the list start at 5 instead of 1. This is useful when a list is split across pages or when you need to continue numbering from a previous list.

---

### Purposes

- **To begin numbering at a value other than 1**.
- **To continue a list across multiple containers** (e.g. paginated content).
- **To align numbering with external references** (e.g. starting at clause 10).

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ol start="5">
  <li>Fifth item</li>
  <li>Sixth item</li>
</ol>
```

**Component breakdown**

| Attribute | Value | Example |
|---|---|---|
| `start` | Integer | `start="5"` → 5, 6, 7… |

#### Syntax Rules

1. **`start` must be a valid integer**.
2. **The value is always an Arabic numeral**, even when `type` is letters or Roman numerals.
3. **`start` is not deprecated**; it has semantic meaning and is not purely presentational.

#### Constraints and Limitations

- **The `start` attribute on `<li>` was deprecated** and should not be used; use `value` on `<li>` for individual overrides instead.
- **`start` does not change the marker style**; it only changes the starting number.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Starting at 5

**Setup Guide**: Save as `ol-start.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Start Attribute</title>
</head>
<body>
  <ol start="5">
    <li>Fifth item</li>
    <li>Sixth item</li>
    <li>Seventh item</li>
  </ol>
</body>
</html>
```

**Expected output**: A list numbered 5, 6, 7.

**Why this output occurs**: The `start="5"` attribute sets the first item's ordinal value to 5, and subsequent items increment from there.

---

#### Example 2: Continuing a List Across Pages

**Setup Guide**: Save as `ol-start-continue.html`.

```html
<!-- Page 1 -->
<ol start="1">
  <li>Item 1</li>
  <li>Item 2</li>
  <li>Item 3</li>
</ol>

<!-- Page 2: continues from item 4 -->
<ol start="4">
  <li>Item 4</li>
  <li>Item 5</li>
</ol>
```

**Expected output**: Page 1 shows 1–3; Page 2 shows 4–5.

**Why this output occurs**: The `start="4"` on the second list continues the numbering from where the first list left off.

---

### Real-World Cases

1. **Paginated lists**: Continuing numbering across pages.
2. **Legal documents**: Starting at a specific clause number.
3. **Split lists**: Resuming numbering after an interruption.

---

### References

- W3C — HTML: The Markup Language: `ol` `start` attribute — https://w3c.github.io/html-reference/ol.html#ol.attrs.start
- MDN Web Docs — `<ol>`: `start` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ol#start

---

## 5. `reversed` Attribute — Descending Order

### Definitions

**Core Definition**
The `reversed` attribute is a boolean attribute that indicates the list should be numbered in descending order — the first item has the highest number, and the last item has the lowest.

**Technical Definition**
`reversed` is a boolean attribute on `<ol>`. If present, the list is a descending list (…, 3, 2, 1); if absent, the list is ascending (1, 2, 3, …). When `reversed` is present and `start` is also specified, numbering begins at the `start` value and counts down. If `start` is omitted, the browser counts the number of list items and counts down from that number to 1.

**Beginner-Friendly Explanation**
`reversed` makes the list count backwards. The first item gets the highest number, and the last item gets 1. This is useful for countdowns, top-ten lists, or reverse-chronological content.

---

### Purposes

- **To number a list in descending order**.
- **To support countdowns and reverse-chronological lists**.
- **To allow ranking lists where the highest number is most important**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<ol reversed>
  <li>Third item</li>
  <li>Second item</li>
  <li>First item</li>
</ol>
```

**Component breakdown**

| Attribute | Type | Effect |
|---|---|---|
| `reversed` | Boolean | Numbers the list in descending order. |

#### Syntax Rules

1. **`reversed` is a boolean attribute**; presence means descending order.
2. **If `start` is also present, numbering begins at `start` and counts down**.
3. **If `start` is absent, the browser counts the items and counts down from that number to 1**.

#### Constraints and Limitations

- **`reversed` does not change the order of the items in the DOM**; it only changes the numbering. The first item remains first in reading order.
- **Browser support is broad**; the attribute is part of the HTML Living Standard.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Reversed List

**Setup Guide**: Save as `ol-reversed.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Reversed List</title>
</head>
<body>
  <ol reversed>
    <li>Third item</li>
    <li>Second item</li>
    <li>First item</li>
  </ol>
</body>
</html>
```

**Expected output**: A list numbered 3, 2, 1.

**Why this output occurs**: The `reversed` boolean attribute instructs the browser to number the list in descending order.

---

#### Example 2: Reversed List with a Start Value

**Setup Guide**: Save as `ol-reversed-start.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Reversed with Start</title>
</head>
<body>
  <ol reversed start="10">
    <li>Item at 10</li>
    <li>Item at 9</li>
    <li>Item at 8</li>
  </ol>
</body>
</html>
```

**Expected output**: A list numbered 10, 9, 8.

**Why this output occurs**: The `start="10"` sets the highest number, and `reversed` counts down from there.

---

### Real-World Cases

1. **Countdown lists**: Top 10 lists where 10 is first.
2. **Reverse-chronological timelines**: Most recent first, numbered in descending order.
3. **Ranked results**: Highest score first, with the rank number descending.

---

### References

- W3C — HTML: The Markup Language: `ol` `reversed` attribute — https://w3c.github.io/html-reference/ol.html#ol.attrs.reversed
- MDN Web Docs — `<ol>`: `reversed` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ol#reversed

---

## 6. Ordered List Best Practices

### Definitions

**Core Definition**
Ordered list best practices are guidelines for choosing `<ol>` appropriately, structuring list items correctly, using attributes meaningfully, and presenting lists accessibly — so that list semantics are preserved for assistive technologies and search engines.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG 1.3.1 (Info and Relationships), and W3C Technique H48 (Using `ol`, `ul` and `dl` for lists or groups of links). They encompass choosing `<ol>` only when order is meaningful, using `<li>` for every item, nesting sub-lists inside `<li>`, using `type` only when the marker style has semantic meaning, and preserving list semantics when styling away markers.

**Beginner-Friendly Explanation**
Best practices help you build ordered lists that are correct and accessible: use `<ol>` only when the sequence matters, put every item in an `<li>`, use `type` only when the numbering style is meaningful, and do not break the list semantics when you style the list.

---

### Purposes

- **To ensure semantic correctness** and valid markup.
- **To preserve accessibility** for screen-reader users who rely on list semantics and item counts.
- **To support SEO** by providing clear sequential structure that search engines can parse.
- **To maintain visual and structural consistency** across a site.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `<ol>` only when order matters** | If reordering does not change meaning, use `<ul>`. |
| **Every item must be an `<li>`** | Do not use `<div>` or `<span>` as direct children of `<ol>`. |
| **Nest sub-lists inside `<li>`** | A nested `<ol>` or `<ul>` must be a child of the parent `<li>`. |
| **Use `type` only when the numbering style is meaningful** | For purely visual styling, use CSS `list-style-type`. |
| **Do not use the obsolete `compact` attribute** | Use CSS instead. |
| **Use `start` to continue numbering across containers** | Semantic continuation across paginated content. |
| **Use `reversed` for descending sequences** | Countdowns, reverse-chronological lists. |
| **Preserve list semantics when styling** | `list-style: none` can remove list semantics in some browsers; use `role="list"` as a fallback. |

#### Syntax Rules

1. **Semantic correctness**: `<ol>` for ordered sequences, `<ul>` for unordered groups, `<dl>` for term–definition pairs.
2. **Structural correctness**: `<li>` for every item; nested lists inside `<li>`.
3. **Attribute discipline**: Use `type`, `start`, and `reversed` only when they carry meaning, not for purely visual effects.

#### Constraints and Limitations

- **Using `<ol>` for non-sequential content misrepresents the structure** and confuses screen-reader users.
- **CSS `list-style-type` overrides `type`**; do not use both with conflicting values.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct vs Incorrect Ordered List Usage

**Setup Guide**: Save as `ol-best-practice.html`. Run an HTML validator and inspect the accessibility tree.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Ordered List Best Practices</title>
</head>
<body>
  <!-- GOOD: ol for a sequence where order matters -->
  <ol>
    <li>Preheat the oven.</li>
    <li>Mix the ingredients.</li>
    <li>Bake for 25 minutes.</li>
  </ol>

  <!-- BAD: ol used for a non-sequential group (should be ul) -->
  <!--
  <ol>
    <li>Milk</li>
    <li>Eggs</li>
    <li>Bread</li>
  </ol>
  -->

  <!-- BAD: div as a direct child of ol -->
  <!--
  <ol>
    <div>Step 1</div>
    <div>Step 2</div>
  </ol>
  -->
</body>
</html>
```

**Expected output**: The good example produces a numbered list of steps with proper list semantics. The bad examples misrepresent the content or break the list structure.

**Why this output occurs**: `<ol>` is reserved for ordered sequences, and `<li>` is the only permitted direct child of `<ol>`. Violating these rules triggers parser error recovery and misleads assistive technologies.

---

#### Example 2: Preserving Semantics When Styling Away Markers

**Setup Guide**: Save as `ol-styled.html`. Inspect the accessibility tree.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Styled Ordered List</title>
  <style>
    ol.steps {
      list-style: none;
      counter-reset: step;
    }
    ol.steps li::before {
      counter-increment: step;
      content: counter(step) ". ";
      font-weight: bold;
    }
  </style>
</head>
<body>
  <!-- role="list" preserves list semantics when list-style: none is used -->
  <ol class="steps" role="list">
    <li>Preheat the oven.</li>
    <li>Mix the ingredients.</li>
    <li>Bake for 25 minutes.</li>
  </ol>
</body>
</html>
```

**Expected output**: A custom-styled numbered list that still exposes list semantics to screen readers.

**Why this output occurs**: The `role="list"` attribute restores the list semantics that `list-style: none` may remove in some browsers, ensuring that screen readers announce the list and item count.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG 1.3.1 compliance requires semantic list markup; audits check for `<div>`-based fake lists.
2. **Recipe sites**: Ordered lists for step-by-step instructions, with `type` used only when the numbering style matters.
3. **Legal and technical documentation**: Ordered lists with `type` and `start` for meaningful numbering schemes.

---

### References

- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG22/Techniques/html/H48
- MDN Web Docs — `<ol>`: Accessibility — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ol#accessibility
- WHATWG — The `ol` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-ol-element
- WebAIM — Creating Accessible Lists — https://webaim.org/techniques/semanticstructure/

---

## Summary Table of Ordered List Concepts

| Concept | Key Takeaway |
|---|---|
| **`<ol>`** | Container for ordered sequences; content model is zero or more `<li>` plus script-supporting elements; exposed as `list`. |
| **`<li>`** | One list item; flow content when the parent is `<ol>`; `value` overrides the ordinal value; end tag may be omitted. |
| **`type`** | Marker style: `1`, `a`, `A`, `i`, `I`; use only when the numbering style is meaningful; CSS `list-style-type` overrides it. |
| **`start`** | Ordinal value of the first item; always an Arabic numeral; used to continue lists across containers. |
| **`reversed`** | Boolean; numbers the list in descending order; can be combined with `start`. |
| **Best practices** | Use `<ol>` only when order matters; every item must be `<li>`; nest inside `<li>`; preserve semantics when styling. |

---

## Final Notes

- **Order matters**: If reordering the items would not change the meaning, use `<ul>` instead of `<ol>`.
- **Every item must be an `<li>`**: `<div>`-based fake lists lose the accessibility semantics that screen readers rely on.
- **`type` is meaningful, not just visual**: Use it when the numbering style has semantic significance; otherwise use CSS `list-style-type`.
- **`start` and `reversed` control numbering**: Use them to continue lists across pages or to create descending sequences.
- **Preserve semantics when styling**: `list-style: none` can remove list semantics in some browsers; add `role="list"` to restore them.
- **Validate and test**: Use HTML validators and accessibility checkers to catch non-`<li>` children and invalid nesting.

---

This cheat sheet has covered ordered lists — the `<ol>` container, the `<li>` list item, the `type`, `start`, and `reversed` attributes, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate list markup against the WHATWG HTML Living Standard and WCAG 1.3.1 (Info and Relationships) with Technique H48.