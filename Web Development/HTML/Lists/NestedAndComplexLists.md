# Nested Lists — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Nested lists are lists placed inside other lists. An inner list is created by placing a new `<ul>`, `<ol>`, or `<menu>` element inside an `<li>` element of an outer list. The nesting expresses a hierarchical relationship: the inner items belong to, or are sub-categories of, the outer item that contains them.

**Technical Definition**
In the WHATWG HTML Living Standard, nesting is expressed structurally: a list element (`<ul>`, `<ol>`, `<menu>`) is a valid child of an `<li>` element, and that inner list contains its own `<li>` children. The parent–child relationship in the DOM is reflected in the accessibility tree, where screen readers announce the nesting depth and the number of items at each level. Nested lists may mix types — an `<ol>` inside a `<ul>`, a `<ul>` inside an `<ol>`, or a `<dl>` alongside either — as long as each inner list is placed inside an `<li>` (or, for `<dl>`, inside the parent list item as appropriate). The common structural error is placing a list element directly inside another list element instead of inside an `<li>`, which violates the content model and triggers parser error recovery.

**Beginner-Friendly Explanation**
A nested list is a list inside a list. Imagine a table of contents: “Chapter 1” is an item, and under it are sections “1.1”, “1.2”, and “1.3”. The chapters form the outer list, and the sections form the inner list. You create a nested list by putting the inner list **inside the list item** that owns it — not next to it. This preserves the parent–child relationship for screen readers and search engines.

---

### Key Characteristics

- **Structure encodes hierarchy**: The inner list is a child of the `<li>` it belongs to, so the nesting relationship is explicit in the DOM.
- **Mixed list types are permitted**: An `<ol>` may contain a nested `<ul>`, and vice versa; a `<ul>` may contain a nested `<dl>` inside an `<li>`, and so on.
- **Accessibility depth**: Screen readers announce the nesting level, the number of items at each level, and allow navigation in and out of sub-lists.
- **CSS controls visual depth**: Indentation, marker styles, and layout are presentation concerns; the semantic nesting is independent of the visual indentation.
- **Depth is a usability trade-off**: Two or three levels are manageable; deeper nesting becomes difficult for screen-reader and keyboard users to navigate.
- **Invalid nesting is common**: Placing a `<ul>` directly inside another `<ul>` (rather than inside an `<li>`) is a frequent and easily caught error.

---

### Prerequisites

1. **Basic HTML list syntax**: familiarity with `<ul>`, `<ol>`, `<dl>`, and `<li>`.
2. **Content models**: understanding that `<li>` accepts flow content when the parent is `<ul>` or `<ol>`.
3. **CSS list styling**: awareness of `list-style-type`, `list-style-position`, and `margin`/`padding` on lists.
4. **Accessibility basics**: understanding that screen readers expose list depth and item counts.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS** | `list-style-type` can vary by depth (disc → circle → square) to reinforce the visual hierarchy. |
| **Accessibility (WCAG)** | WCAG 1.3.1 (Info and Relationships) and Technique H48 require semantic list markup; nested lists must preserve the parent–child relationship. |
| **Navigation menus** | Multi-level site navigation is the most common nested-list pattern. |
| **Tables of contents and outlines** | Document outlines use nested lists to express section hierarchy. |
| **SEO** | Search engines parse nested lists to understand category trees and site structure. |
| **JavaScript** | Nested lists are common targets for recursive rendering, tree views, and event delegation. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Nested lists** — the general concept
2. **Multi-level structures** — two, three, and deeper levels
3. **Mixed list types** — `<ul>` inside `<ol>`, `<dl>` alongside lists
4. **Semantic list usage** — choosing the right list type at each level
5. **Navigation lists** — nested `<ul>` inside `<nav>`
6. **Content lists** — nested lists in prose and documentation
7. **Nesting best practices** — consolidated guidelines

---

## 1. Nested Lists — The General Concept

### Definitions

**Core Definition**
A nested list is a list element placed inside an `<li>` element of another list. The outer list contains the parent items, and the inner list contains the sub-items belonging to one of those parents.

**Technical Definition**
Nesting is expressed by placing a `<ul>`, `<ol>`, or `<menu>` element as a child of an `<li>` element. The inner list contains its own `<li>` children, which may themselves contain further nested lists. The DOM tree therefore mirrors the conceptual hierarchy, and the accessibility tree exposes the nesting depth. The `<li>` element's content model is flow content when the parent is `<ul>` or `<ol>`, so the inner list is a valid child.

**Beginner-Friendly Explanation**
A nested list is a list inside a list item. The inner list goes inside the `<li>` of the parent item it belongs to — not next to it.

---

### Purposes

- **To represent hierarchical relationships** between items.
- **To preserve the parent–child structure** for assistive technologies and search engines.
- **To enable multi-level navigation menus** and document outlines.
- **To support recursive rendering** in scripts and components.

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

1. **The nested list must be a child of an `<li>`**, not a direct child of the outer list.
2. **The nested list may follow the parent item's content** (text, link, or other content) within the same `<li>`.
3. **Each level of nesting is exposed as a nested list** in the accessibility tree, with its own item count.

#### Constraints and Limitations

- **A list element directly inside another list element is invalid** and triggers parser error recovery.
- **Very deep nesting** can be difficult for screen-reader users to navigate.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Two-Level Nested List

**Setup Guide**: Save as `nested-basic.html`. Inspect the accessibility tree to confirm the nesting depth.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested List</title>
</head>
<body>
  <ul>
    <li>
      Fruits
      <!-- The nested ul is inside the li, preserving the parent-child relationship -->
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

**Expected output**: A bulleted list with two top-level items (“Fruits” and “Vegetables”), each containing a nested bulleted sub-list.

**Why this output occurs**: The inner `<ul>` is placed inside the parent `<li>`, so the browser renders it as a sub-list. The accessibility tree exposes the nesting, and screen readers announce the parent item followed by the nested list and its item count.

---

#### Example 2: Invalid Nesting (Anti-Pattern)

**Setup Guide**: Save as `nested-invalid.html` and inspect the parsed DOM.

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

**Expected output**: The browser renders something, but the DOM inspector reveals that the parser has restructured the markup; the inner list is not associated with the “Fruits” item, and the nesting relationship is lost.

**Why this output occurs**: `<ul>` accepts only `<li>` and script-supporting elements as direct children. A direct `<ul>` child violates the content model and triggers parser error recovery.

---

### Real-World Cases

1. **Category trees**: Product categories with sub-categories.
2. **Document outlines**: Chapters with nested sections.
3. **Multi-level navigation**: Dropdown menus within primary navigation.

---

### References

- MDN Web Docs — `<ul>`: Nesting lists — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul#nesting_lists
- WHATWG — The `ul` element: Content model — https://html.spec.whatwg.org/dev/grouping-content.html#the-ul-element
- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG22/Techniques/html/H48

---

## 2. Multi-Level Structures — Two, Three, and Deeper Levels

### Definitions

**Core Definition**
A multi-level structure is a nested list with more than two levels of depth — for example, a chapter containing sections, which contain subsections, which contain topics.

**Technical Definition**
Each additional level is created by placing another list inside an `<li>` of the previous level. The accessibility tree exposes each level separately, and screen readers announce the depth and item counts. Best practice is to keep nesting to two or three levels for navigability; deeper structures are better represented with headings, tables, or dedicated tree-view components.

**Beginner-Friendly Explanation**
A multi-level structure is a list with sub-lists inside sub-lists. Two levels is a list with sub-items; three levels is a list with sub-items that themselves have sub-items. Deeper than that becomes hard to navigate.

---

### Purposes

- **To represent deeper hierarchies** such as book → chapter → section → subsection.
- **To support complex navigation menus** with sub-menus.
- **To provide a navigable outline** for assistive technologies.

---

### Syntax Rules and Structure

#### Complete General Syntax (Three Levels)

```html
<ul>
  <li>
    Level 1
    <ul>
      <li>
        Level 2
        <ul>
          <li>Level 3</li>
        </ul>
      </li>
    </ul>
  </li>
</ul>
```

**Component breakdown**

| Level | Element | Container |
|---|---|---|
| Level 1 | Outer `<ul>` | Parent element |
| Level 2 | Nested `<ul>` | `<li>` of level 1 |
| Level 3 | Nested `<ul>` | `<li>` of level 2 |

#### Syntax Rules

1. **Each nested list must be inside the `<li>` of its parent item**.
2. **Each level should use an appropriate list type** — `<ol>` for sequences, `<ul>` for unordered groups.
3. **Use headings or other structural elements when depth exceeds three levels**; deep lists are difficult to navigate.

#### Constraints and Limitations

- **Deep nesting increases cognitive load** for screen-reader and keyboard users.
- **CSS indentation must reflect the nesting depth**; otherwise the visual hierarchy contradicts the semantic hierarchy.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Three-Level Document Outline

**Setup Guide**: Save as `nested-three-levels.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Three-Level Outline</title>
</head>
<body>
  <ol>
    <li>
      Introduction
      <ul>
        <li>Background</li>
        <li>
          Scope
          <ul>
            <li>In scope</li>
            <li>Out of scope</li>
          </ul>
        </li>
      </ul>
    </li>
    <li>Methods</li>
    <li>Results</li>
  </ol>
</body>
</html>
```

**Expected output**: A numbered outer list with a nested bulleted list under “Introduction”, and a further nested bulleted list under “Scope”.

**Why this output occurs**: Each nested `<ul>` is placed inside the `<li>` of its parent, so the browser renders three distinct levels. The accessibility tree exposes the depth, and screen readers announce the nesting.

---

#### Example 2: Mixed Depth with Different Markers

**Setup Guide**: Save as `nested-markers.html`. Use CSS to vary marker styles by depth.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Marker Styles by Depth</title>
  <style>
    ul { list-style-type: disc; }
    ul ul { list-style-type: circle; }
    ul ul ul { list-style-type: square; }
  </style>
</head>
<body>
  <ul>
    <li>
      Level 1 (disc)
      <ul>
        <li>
          Level 2 (circle)
          <ul>
            <li>Level 3 (square)</li>
          </ul>
        </li>
      </ul>
    </li>
  </ul>
</body>
</html>
```

**Expected output**: A three-level list with disc markers at level 1, circle markers at level 2, and square markers at level 3.

**Why this output occurs**: The CSS selectors `ul`, `ul ul`, and `ul ul ul` apply different `list-style-type` values to each nesting depth, reinforcing the visual hierarchy without altering the HTML structure.

---

### Real-World Cases

1. **Book tables of contents**: Part → chapter → section.
2. **Documentation sidebars**: Category → page → sub-section.
3. **Organisational charts**: Department → team → role.

---

### References

- MDN Web Docs — `<ul>`: Nesting lists — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul#nesting_lists
- WHATWG — The `ul` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-ul-element

---

## 3. Mixed List Types — `<ul>` Inside `<ol>`, `<dl>` Alongside Lists

### Definitions

**Core Definition**
Mixed list types occur when a nested list uses a different list element from its parent — for example, an `<ol>` inside a `<ul>`, a `<ul>` inside an `<ol>`, or a `<dl>` placed inside an `<li>` alongside either.

**Technical Definition**
The WHATWG HTML Living Standard permits any list element (`<ul>`, `<ol>`, `<menu>`) to be a child of an `<li>`, and `<dl>` is flow content that may also appear inside an `<li>`. Mixing types is semantically meaningful: an ordered sequence can contain an unordered sub-list of notes, or an unordered category can contain an ordered sub-list of steps. The accessibility tree exposes each list with its own role (`list`), and the nesting depth is preserved regardless of type.

**Beginner-Friendly Explanation**
Mixed list types mean using a numbered list inside a bulleted list, or a bulleted list inside a numbered list. This is perfectly valid and often clearer than forcing every level to use the same type. For example, a numbered recipe step might contain a bulleted list of ingredients.

---

### Purposes

- **To match the list type to the meaning at each level** — order matters at one level, not at another.
- **To combine sequences and groups** in a single structure.
- **To represent complex hierarchies** such as a numbered process with unordered notes.

---

### Syntax Rules and Structure

#### Complete General Syntax (`<ul>` Inside `<ol>`)

```html
<ol>
  <li>
    Step 1
    <ul>
      <li>Note A</li>
      <li>Note B</li>
    </ul>
  </li>
</ol>
```

#### Complete General Syntax (`<ol>` Inside `<ul>`)

```html
<ul>
  <li>
    Category
    <ol>
      <li>First step</li>
      <li>Second step</li>
    </ol>
  </li>
</ul>
```

#### Complete General Syntax (`<dl>` Inside `<li>`)

```html
<ul>
  <li>
    Product
    <dl>
      <dt>Price</dt>
      <dd>$29.99</dd>
      <dt>Weight</dt>
      <dd>250 g</dd>
    </dl>
  </li>
</ul>
```

**Component breakdown**

| Pattern | Outer | Inner |
|---|---|---|
| Ordered with unordered notes | `<ol>` | `<ul>` |
| Unordered category with ordered steps | `<ul>` | `<ol>` |
| List item with metadata | `<ul>` or `<ol>` | `<dl>` |

#### Syntax Rules

1. **The inner list must be inside an `<li>` of the outer list**.
2. **Choose the list type that matches the meaning at each level**.
3. **A `<dl>` inside an `<li>` is valid flow content** and may be used for metadata.

#### Constraints and Limitations

- **Do not mix types purely for visual effect**; the type must match the meaning.
- **A `<dl>` inside an `<li>`** is valid, but do not use it as a substitute for a nested `<ul>` when the content is an unordered group.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Ordered Steps with Unordered Notes

**Setup Guide**: Save as `nested-mixed-ol-ul.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Ordered with Unordered Notes</title>
</head>
<body>
  <ol>
    <li>
      Preheat the oven to 180°C.
      <ul>
        <li>Use the fan setting if available.</li>
        <li>Place the rack in the middle.</li>
      </ul>
    </li>
    <li>Mix the ingredients.</li>
    <li>Bake for 25 minutes.</li>
  </ol>
</body>
</html>
```

**Expected output**: A numbered list of steps, with the first step containing a nested bulleted list of notes.

**Why this output occurs**: The outer `<ol>` conveys the ordered sequence of steps, and the inner `<ul>` conveys the unordered notes that apply to the first step. The accessibility tree exposes both levels and their roles.

---

#### Example 2: Unordered Categories with Ordered Steps

**Setup Guide**: Save as `nested-mixed-ul-ol.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Unordered with Ordered Steps</title>
</head>
<body>
  <ul>
    <li>
      Account setup
      <ol>
        <li>Create a username.</li>
        <li>Set a password.</li>
        <li>Verify your email.</li>
      </ol>
    </li>
    <li>
      Profile configuration
      <ol>
        <li>Upload a photo.</li>
        <li>Write a bio.</li>
      </ol>
    </li>
  </ul>
</body>
</html>
```

**Expected output**: A bulleted list of categories, each containing a numbered list of steps.

**Why this output occurs**: The outer `<ul>` groups the categories (order does not matter), and the inner `<ol>` sequences the steps within each category (order matters). The mixed types match the meaning at each level.

---

#### Example 3: List Item with Metadata Using `<dl>`

**Setup Guide**: Save as `nested-dl.html`.

```html
<ul>
  <li>
    Wireless Headphones
    <dl>
      <dt>Price</dt>
      <dd>$299.99</dd>
      <dt>Battery life</dt>
      <dd>30 hours</dd>
      <dt>Weight</dt>
      <dd>250 g</dd>
    </dl>
  </li>
  <li>
    Charging Case
    <dl>
      <dt>Price</dt>
      <dd>$49.99</dd>
      <dt>Connector</dt>
      <dd>USB-C</dd>
    </dl>
  </li>
</ul>
```

**Expected output**: A bulleted list of products, each with a nested description list of metadata.

**Why this output occurs**: The outer `<ul>` groups the products, and the inner `<dl>` provides the key–value metadata for each product. The nested `<dl>` is valid flow content inside the `<li>`.

---

### Real-World Cases

1. **Recipe instructions**: Numbered steps with bulleted ingredient notes.
2. **Product catalogs**: Bulleted categories with numbered configuration steps.
3. **Documentation**: Bulleted feature lists with nested metadata descriptions.

---

### References

- MDN Web Docs — `<li>`: Content model — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/li
- WHATWG — The `li` element — https://html.spec.whatwg.org/dev/grouping-content.html#the-li-element

---

## 4. Semantic List Usage — Choosing the Right List Type at Each Level

### Definitions

**Core Definition**
Semantic list usage is the practice of choosing `<ul>`, `<ol>`, `<dl>`, or `<menu>` at each level based on the **meaning** of the items at that level, not on visual preference.

**Technical Definition**
The choice of list element is a semantic decision governed by the WHATWG HTML Living Standard: `<ul>` for unordered groups, `<ol>` for sequences where order matters, `<dl>` for name–value pairs, and `<menu>` for command lists. In a nested structure, each level may use a different type if the meaning differs; the outer type and inner type are independent. WCAG 1.3.1 (Info and Relationships) requires that the structure conveyed visually is also conveyed programmatically, which means the list types must match the intended semantics.

**Beginner-Friendly Explanation**
Semantic list usage means picking the right list type for what the items actually are. If order matters, use `<ol>`. If it does not, use `<ul>`. If you have term–value pairs, use `<dl>`. In a nested list, you can mix types when the meaning changes from one level to the next.

---

### Purposes

- **To convey the correct structure** to assistive technologies and search engines.
- **To satisfy WCAG 1.3.1 (Info and Relationships)**.
- **To support correct rendering** of markers and numbering.
- **To make nested structures understandable** to screen-reader users.

---

### Syntax Rules and Structure

| Content at the level | Correct list type |
|---|---|
| Unordered group of related items | `<ul>` |
| Ordered sequence (steps, ranks) | `<ol>` |
| Term–value pairs, metadata | `<dl>` |
| Commands or interactive controls | `<menu>` |

#### Syntax Rules

1. **Choose the list type based on meaning**, not on the desired visual marker.
2. **Each level is independent**; a `<ul>` may contain an `<ol>` and vice versa.
3. **Use `<dl>` for name–value pairs** even when nested inside another list.
4. **Use CSS to change marker appearance**, not a different list element.

#### Constraints and Limitations

- **Using `<ol>` for unordered items misrepresents the structure** and confuses screen-reader users.
- **Using `<ul>` for ordered items loses the sequence information** that assistive technologies rely on.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct Semantic Choices in a Nested Structure

**Setup Guide**: Save as `semantic-nested.html`.

```html
<ol>
  <li>
    Prepare the workspace
    <ul>
      <li>Clear the desk.</li>
      <li>Gather the tools.</li>
    </ul>
  </li>
  <li>
    Assemble the model
    <dl>
      <dt>Part A</dt>
      <dd>Attach to Part B.</dd>
      <dt>Part C</dt>
      <dd>Insert into the base.</dd>
    </dl>
  </li>
</ol>
```

**Expected output**: A numbered sequence of steps, with an unordered checklist inside step 1 and a description list of parts inside step 2.

**Why this output occurs**: The outer `<ol>` conveys the step sequence; the inner `<ul>` conveys an unordered checklist; the inner `<dl>` conveys part–instruction pairs. Each level uses the type that matches its meaning.

---

#### Example 2: Incorrect Semantic Choice

**Setup Guide**: Save as `semantic-nested-bad.html`.

```html
<!-- BAD: an ol is used for unordered notes -->
<ol>
  <li>
    Step 1
    <ol>
      <li>Note A</li>
      <li>Note B</li>
    </ol>
  </li>
</ol>
```

**Expected output**: The inner notes are numbered, implying a sequence that does not exist.

**Why this output occurs**: The inner `<ol>` misrepresents unordered notes as an ordered sequence, violating the semantic meaning of the content.

---

### Real-World Cases

1. **Recipe instructions**: Ordered steps with unordered ingredient notes.
2. **Technical documentation**: Unordered categories with ordered procedures.
3. **Product catalogs**: Unordered products with metadata description lists.

---

### References

- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG22/Techniques/html/H48
- MDN Web Docs — `<ul>`, `<ol>`, `<dl>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements

---

## 5. Navigation Lists — Nested `<ul>` Inside `<nav>`

### Definitions

**Core Definition**
Navigation lists are nested `<ul>` structures placed inside a `<nav>` element, used to represent multi-level site navigation menus.

**Technical Definition**
The standard navigation pattern is `<nav><ul><li><a href>…</a></li></ul></nav>`, with sub-menus represented as nested `<ul>` elements inside the parent `<li>`. The `<nav>` element creates a `navigation` landmark, and the `<ul>` provides list semantics that screen readers announce (“list, 5 items”) and that allow users to jump between items. Sub-menus are typically hidden by default and revealed on hover or focus via CSS or JavaScript, but the nested `<ul>` structure remains in the DOM and is announced by assistive technologies when the sub-menu is expanded.

**Beginner-Friendly Explanation**
Navigation lists are the standard way to build site menus: a `<nav>` landmark containing a `<ul>`, with sub-menus as nested `<ul>` elements. Screen readers announce the menu as a list and let users navigate item by item, including into sub-menus.

---

### Purposes

- **To provide a semantic navigation landmark** with list structure.
- **To support multi-level menus** with nested sub-menus.
- **To expose menu structure to assistive technologies** for navigation and item counting.
- **To enable CSS-driven dropdowns** and keyboard-accessible sub-menus.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<nav aria-label="Primary">
  <ul>
    <li><a href="/">Home</a></li>
    <li>
      <a href="/products">Products</a>
      <ul>
        <li><a href="/products/headphones">Headphones</a></li>
        <li><a href="/products/speakers">Speakers</a></li>
      </ul>
    </li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<nav>` | The navigation landmark; `aria-label` distinguishes multiple navs. |
| `<ul>` | The top-level menu list. |
| `<li>` | A menu item; sub-menus are nested `<ul>` inside the `<li>`. |
| `<a>` | The navigable link. |

#### Syntax Rules

1. **Use `<nav>` with an `aria-label`** when multiple navigation regions exist.
2. **Use `<ul>` for the menu structure** and `<li>` for each item.
3. **Nest sub-menus inside the parent `<li>`** as a `<ul>`.
4. **Ensure keyboard accessibility** for sub-menus; focus styles and `:focus-within` reveal the sub-menu.

#### Constraints and Limitations

- **Sub-menus hidden with `display: none` are removed from the accessibility tree**; use focus-within or JavaScript to reveal them on keyboard focus.
- **Hover-only sub-menus are inaccessible** to keyboard and touch users.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Multi-Level Navigation Menu

**Setup Guide**: Save as `nav-nested.html`. Tab through the menu to verify keyboard access.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested Navigation</title>
  <style>
    nav ul { list-style: none; padding: 0; margin: 0; }
    nav > ul { display: flex; gap: 16px; }
    nav li { position: relative; }
    nav li > ul {
      display: none;
      position: absolute;
      top: 100%;
      left: 0;
      background: white;
      border: 1px solid #ccc;
      min-width: 160px;
    }
    nav li:hover > ul,
    nav li:focus-within > ul { display: block; }
    nav a:focus-visible { outline: 2px solid #005fcc; }
  </style>
</head>
<body>
  <nav aria-label="Primary">
    <ul>
      <li><a href="/">Home</a></li>
      <li>
        <a href="/products">Products</a>
        <ul>
          <li><a href="/products/headphones">Headphones</a></li>
          <li><a href="/products/speakers">Speakers</a></li>
        </ul>
      </li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</body>
</html>
```

**Expected output**: A horizontal top-level menu with a dropdown sub-menu under “Products”. The sub-menu appears on hover and on keyboard focus, and screen readers announce the nested list when it is revealed.

**Why this output occurs**: The `<nav>` landmark and `<ul>` list structure provide semantics, and the nested `<ul>` inside the “Products” `<li>` provides the sub-menu. The `:focus-within` rule reveals the sub-menu on keyboard focus, making it accessible to keyboard users.

---

#### Example 2: A Breadcrumb Navigation List

**Setup Guide**: Save as `nav-breadcrumb.html`.

```html
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="/">Home</a></li>
    <li><a href="/products">Products</a></li>
    <li><a href="/products/headphones" aria-current="page">Headphones</a></li>
  </ol>
</nav>
```

**Expected output**: A breadcrumb trail rendered as an ordered list, with the current page marked by `aria-current="page"`.

**Why this output occurs**: Breadcrumbs are an ordered sequence, so `<ol>` is the correct list type. The `<nav>` landmark and `aria-label` provide context, and `aria-current` identifies the current page.

---

### Real-World Cases

1. **Site navigation menus**: Primary and secondary navigation with dropdowns.
2. **Documentation sidebars**: Category trees with nested pages.
3. **Breadcrumb trails**: Ordered sequences of navigation links.

---

### References

- MDN Web Docs — `<nav>`: The Navigation section element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/nav
- W3C — ARIA Authoring Practices: Disclosure Navigation — https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/
- WebAIM — Creating Accessible Lists — https://webaim.org/techniques/semanticstructure/

---

## 6. Content Lists — Nested Lists in Prose and Documentation

### Definitions

**Core Definition**
Content lists are nested lists used within prose or documentation — for example, a bulleted list of features where one feature has a nested list of sub-features, or a numbered procedure where one step has a nested list of notes.

**Technical Definition**
Content lists are nested lists inside `<article>`, `<section>`, `<main>`, or other prose containers. They follow the same structural rules as navigation lists: the inner list is a child of the `<li>` it belongs to, and each level uses the list type that matches its meaning. Content lists often combine types — an ordered procedure with unordered notes, or an unordered feature list with a description list of specifications.

**Beginner-Friendly Explanation**
Content lists are nested lists in articles and documentation. They are the same structure as navigation lists, but they live in the body of a page rather than in the site menu. They help organise features, steps, notes, and specifications.

---

### Purposes

- **To organise prose content** into scannable, hierarchical groups.
- **To represent procedures with notes and sub-steps**.
- **To present feature lists with specifications**.
- **To improve readability** for all users.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<article>
  <h2>Features</h2>
  <ul>
    <li>
      Wireless connectivity
      <ul>
        <li>Bluetooth 5.3</li>
        <li>Multipoint pairing</li>
      </ul>
    </li>
    <li>30-hour battery life</li>
  </ul>
</article>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<article>` | The prose container. |
| Outer `<ul>` | The top-level feature list. |
| Parent `<li>` | The feature with sub-features. |
| Inner `<ul>` | The sub-features. |

#### Syntax Rules

1. **Use nested lists inside `<li>` elements** as with any other nested list.
2. **Choose the list type that matches the meaning** at each level.
3. **Keep nesting shallow**; use headings for deep hierarchies.

#### Constraints and Limitations

- **Deep nesting in prose** can be difficult to read; prefer headings for deep hierarchy.
- **Nested lists should not replace headings** when the structure is a major section hierarchy.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Feature List with Sub-Features

**Setup Guide**: Save as `content-nested.html`.

```html
<article>
  <h2>Product Features</h2>
  <ul>
    <li>
      Wireless connectivity
      <ul>
        <li>Bluetooth 5.3</li>
        <li>Multipoint pairing</li>
        <li>10-metre range</li>
      </ul>
    </li>
    <li>30-hour battery life</li>
    <li>Active noise cancellation</li>
  </ul>
</article>
```

**Expected output**: A bulleted feature list with a nested bulleted sub-list of connectivity details.

**Why this output occurs**: The outer `<ul>` groups the features, and the inner `<ul>` groups the details of one feature. The accessibility tree exposes both levels.

---

#### Example 2: A Procedure with Nested Notes

**Setup Guide**: Save as `content-procedure.html`.

```html
<article>
  <h2>Setup Instructions</h2>
  <ol>
    <li>
      Connect the power cable.
      <ul>
        <li>Use the included adapter.</li>
        <li>Ensure the outlet is grounded.</li>
      </ul>
    </li>
    <li>Turn on the device.</li>
    <li>
      Configure the network.
      <dl>
        <dt>SSID</dt>
        <dd>Your network name.</dd>
        <dt>Password</dt>
        <dd>Your Wi-Fi password.</dd>
      </dl>
    </li>
  </ol>
</article>
```

**Expected output**: A numbered procedure with an unordered note list inside step 1 and a description list of network settings inside step 3.

**Why this output occurs**: The outer `<ol>` conveys the procedure sequence, the inner `<ul>` conveys unordered notes, and the inner `<dl>` conveys key–value settings. Each level uses the type that matches its meaning.

---

### Real-World Cases

1. **Product feature lists**: Features with sub-features.
2. **Setup procedures**: Steps with nested notes and settings.
3. **Documentation**: Categories with nested specifications.

---

### References

- MDN Web Docs — `<ul>`: Nesting lists — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul#nesting_lists
- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG22/Techniques/html/H48

---

## 7. Nesting Best Practices

### Definitions

**Core Definition**
Nesting best practices are guidelines for structuring nested lists correctly, choosing the right list type at each level, preserving accessibility, and keeping the hierarchy shallow enough to be navigable.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG 1.3.1 (Info and Relationships), and W3C Technique H48. They encompass placing the nested list inside the parent `<li>`, choosing list types based on meaning, ensuring CSS indentation matches the semantic depth, preserving list semantics when styling away markers, keeping nesting to two or three levels, and making sub-menus keyboard-accessible.

**Beginner-Friendly Explanation**
Best practices help you build nested lists that are correct and accessible: put the inner list inside the parent `<li>`, use the right list type for each level, keep nesting shallow, make sure keyboard users can reach sub-menus, and do not break list semantics when you style away bullets.

---

### Purposes

- **To ensure semantic correctness** and valid markup.
- **To preserve accessibility** for screen-reader and keyboard users.
- **To keep hierarchies navigable** by limiting depth.
- **To maintain visual and structural consistency** across a site.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Place nested lists inside `<li>`** | Never as a direct child of another list element. |
| **Choose the list type by meaning** | `<ul>`, `<ol>`, `<dl>`, or `<menu>` as appropriate at each level. |
| **Keep nesting to two or three levels** | Deeper hierarchies are better represented with headings or tree components. |
| **Make sub-menus keyboard-accessible** | Use `:focus-within` or JavaScript to reveal sub-menus on focus. |
| **Preserve list semantics when styling** | Use `role="list"` if `list-style: none` is applied and semantics may be lost. |
| **Match CSS indentation to nesting depth** | Visual hierarchy must reflect semantic hierarchy. |
| **Use `<dfn>` for glossary terms** | Nested description lists are ideal for glossaries. |

#### Syntax Rules

1. **Structural correctness**: Inner lists inside `<li>`; each level uses the correct list type.
2. **Accessibility correctness**: Sub-menus must be keyboard-reachable; list semantics must be preserved.
3. **Depth discipline**: Keep nesting shallow; use headings for deep hierarchies.

#### Constraints and Limitations

- **Deep nesting increases cognitive load** and is difficult for screen-reader users to navigate.
- **Hover-only sub-menus are inaccessible** to keyboard and touch users.
- **`list-style: none` can remove list semantics** in some browsers; use `role="list"` to restore them.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible Nested Navigation with Keyboard Support

**Setup Guide**: Save as `best-practice-nested.html`. Tab through the menu to verify keyboard access.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Nested Navigation</title>
  <style>
    nav ul { list-style: none; padding: 0; margin: 0; }
    nav > ul { display: flex; gap: 16px; }
    nav li { position: relative; }
    nav li > ul {
      display: none;
      position: absolute;
      top: 100%;
      left: 0;
      background: white;
      border: 1px solid #ccc;
      min-width: 160px;
    }
    nav li:hover > ul,
    nav li:focus-within > ul { display: block; }
    nav a:focus-visible { outline: 2px solid #005fcc; }
  </style>
</head>
<body>
  <nav aria-label="Primary">
    <!-- role="list" restores list semantics when list-style: none is used -->
    <ul role="list">
      <li><a href="/">Home</a></li>
      <li>
        <a href="/products">Products</a>
        <ul role="list">
          <li><a href="/products/headphones">Headphones</a></li>
          <li><a href="/products/speakers">Speakers</a></li>
        </ul>
      </li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</body>
</html>
```

**Expected output**: A horizontal navigation menu with a dropdown sub-menu. The sub-menu appears on hover and on keyboard focus, and screen readers announce the nested list when it is revealed.

**Why this output occurs**: The `role="list"` attributes restore list semantics after `list-style: none` is applied, and the `:focus-within` rule reveals the sub-menu on keyboard focus, making it accessible to keyboard users.

---

#### Example 2: Common Nesting Failures

**Setup Guide**: Save as `best-practice-failures.html`. Run an HTML validator and an accessibility checker.

```html
<!-- FAILURE 1: ul directly inside ul -->
<!--
<ul>
  <li>Fruits</li>
  <ul>
    <li>Apples</li>
  </ul>
</ul>
-->

<!-- FAILURE 2: div as a direct child of ul -->
<!--
<ul>
  <div>Item 1</div>
  <div>Item 2</div>
</ul>
-->

<!-- FAILURE 3: hover-only sub-menu without keyboard access -->
<!--
<nav>
  <ul>
    <li>
      <a href="/products">Products</a>
      <ul class="submenu">
        <li><a href="/products/headphones">Headphones</a></li>
      </ul>
    </li>
  </ul>
</nav>
<style>
  .submenu { display: none; }
  li:hover .submenu { display: block; }
</style>
-->
```

**Expected output**: Each failure breaks the structure or accessibility: invalid nesting loses the parent–child relationship, `<div>` children lose list semantics, and hover-only sub-menus are inaccessible to keyboard users.

**Why this output occurs**: The failures violate the content model (nested lists must be inside `<li>`), the list structure (direct children must be `<li>`), and the keyboard accessibility requirement (sub-menus must be focusable).

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG 1.3.1 compliance requires semantic nested lists and keyboard-accessible sub-menus.
2. **E-commerce**: Category trees with nested sub-categories and keyboard-accessible dropdowns.
3. **Documentation sites**: Nested sidebars with keyboard-navigable categories.

---

### References

- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG22/Techniques/html/H48
- MDN Web Docs — `<ul>`: Nesting lists — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/ul#nesting_lists
- WebAIM — Creating Accessible Lists — https://webaim.org/techniques/semanticstructure/

---

## Summary Table of Nested List Concepts

| Concept | Key Takeaway |
|---|---|
| **Nested lists** | Inner list must be a child of an `<li>`, not a direct child of another list. |
| **Multi-level structures** | Two or three levels are manageable; deeper hierarchies are better represented with headings. |
| **Mixed list types** | `<ul>` inside `<ol>`, `<ol>` inside `<ul>`, and `<dl>` inside `<li>` are all valid when the meaning differs by level. |
| **Semantic list usage** | Choose `<ul>`, `<ol>`, `<dl>`, or `<menu>` based on meaning at each level, not visual preference. |
| **Navigation lists** | `<nav><ul><li><a>` with nested `<ul>` for sub-menus; sub-menus must be keyboard-accessible. |
| **Content lists** | Nested lists in prose and documentation; keep nesting shallow and use the right type at each level. |
| **Nesting best practices** | Inner lists inside `<li>`; correct list type by meaning; shallow depth; keyboard-accessible sub-menus; preserve list semantics when styling. |

---

## Final Notes

- **Nest inside `<li>`**: A nested list belongs inside the parent `<li>`, not directly inside another list element.
- **Match list type to meaning**: Use `<ul>`, `<ol>`, `<dl>`, or `<menu>` based on what the items at each level actually are.
- **Keep nesting shallow**: Two or three levels are navigable; deeper hierarchies should use headings or dedicated tree components.
- **Make sub-menus keyboard-accessible**: Hover-only dropdowns exclude keyboard and touch users; use `:focus-within` or JavaScript.
- **Preserve list semantics when styling**: `list-style: none` can remove list semantics in some browsers; add `role="list"` to restore them.
- **Validate and test**: Use HTML validators to catch invalid nesting and accessibility checkers to verify keyboard access and list semantics.

---

This cheat sheet has covered nested lists — the general concept, multi-level structures, mixed list types, semantic list usage, navigation lists, content lists, and nesting best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate nested list markup against the WHATWG HTML Living Standard and WCAG 1.3.1 (Info and Relationships) with Technique H48.