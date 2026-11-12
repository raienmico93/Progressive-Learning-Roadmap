# Menu — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
The `<menu>` element is an HTML element described in the WHATWG HTML Living Standard as a **semantic alternative to `<ul>`**. It represents an unordered list of items, each represented by an `<li>` element. Browsers render it identically to `<ul>` and expose it through the accessibility tree as a `list`, but its semantic intent is to group **commands, menu items, or interactive controls** rather than general prose items.

**Technical Definition**
In the WHATWG HTML Living Standard, `<menu>` belongs to the **flow content** category and, when it has at least one list item child, to the **palpable content** category. Its content model is **zero or more `<li>` elements** and script-supporting elements (`<script>`, `<template>`). Crucially, when an `<li>` is a child of a `<menu>`, the `<li>`'s content model is **phrasing content** — not the flow content permitted inside `<ul>` or `<ol>`. The element accepts only **global attributes**; the historical `type` and `label` attributes are obsolete and must not be used. The DOM interface is `HTMLMenuElement`, which in the current standard exposes no members beyond those inherited from `HTMLElement`. In the accessibility tree, `<menu>` maps to the ARIA role `list`, and `<li>` maps to `listitem`, exactly as with `<ul>`.

**Beginner-Friendly Explanation**
`<menu>` is a list element that looks and behaves exactly like `<ul>`, but it is intended to represent a menu of commands or interactive items rather than a general bulleted list. Browsers show it with bullets, and screen readers announce it as a list — there is no visual or accessibility difference. In practice, `<ul>` is far more common because `<menu>` has a stricter content model (its `<li>` children can only contain phrasing content, not block-level elements like paragraphs or nested lists) and is not widely known. Use `<menu>` only when the list genuinely represents a set of commands or controls and you want the semantic distinction.

---

### Key Characteristics

- **Semantic alternative to `<ul>`**: The specification explicitly describes `<menu>` as “a semantic alternative to the `ul` element” with “no other distinguishing features”.
- **Identical rendering**: Browsers apply the same default styles as `<ul>` (`list-style-type: disc`, block display, padding for markers).
- **Identical accessibility exposure**: Both `<menu>` and `<ul>` are exposed as ARIA role `list`; `<li>` is exposed as `listitem`.
- **Stricter `<li>` content model**: Inside `<menu>`, an `<li>` may contain only **phrasing content** — not flow content like paragraphs, headings, or nested lists.
- **No element-specific attributes**: The historical `type` and `label` attributes are obsolete; the element accepts only global attributes.
- **Rarely used in practice**: `<ul>` is overwhelmingly more common because it is more flexible and universally understood.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements and content models.
2. **Flow vs. phrasing content**: understanding what may appear inside an `<li>` depending on its parent.
3. **CSS list properties**: awareness of `list-style-type`, `list-style-position`, and `display: flex/grid` on lists.
4. **Accessibility basics**: understanding that screen readers expose list semantics and item counts.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **CSS** | `list-style-type`, `list-style-position`, and `display: flex/grid` on `<menu>` control presentation identically to `<ul>`. |
| **Accessibility (WCAG)** | WCAG 1.3.1 (Info and Relationships) applies; `<menu>` is exposed as `list` like `<ul>`. |
| **Command menus** | The semantic intent is a list of commands or interactive controls, though the spec no longer defines context-menu or toolbar behaviour. |
| **JavaScript** | `<menu>` and `<li>` are common targets for dynamic menu rendering and event delegation. |
| **Framework components** | UI libraries sometimes use `<menu>` for semantic clarity when rendering command lists, but `<ul>` is more common. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<menu>`** — the menu container
2. **`<li>`** — the menu item
3. **Menu best practices** — consolidated guidelines

---

## 1. `<menu>` — The Menu Container

### Definitions

**Core Definition**
The `<menu>` element represents an unordered list of items, each represented by an `<li>` element. It is a semantic alternative to `<ul>`.

**Technical Definition**
`<menu>` belongs to the **flow content** category and, when it has at least one list item child, to the **palpable content** category. Its content model is **zero or more `<li>` elements** and script-supporting elements. The element accepts only **global attributes**; `type` and `label` are obsolete. In the accessibility tree, `<menu>` is exposed as the ARIA role `list`. By default, browsers apply `display: block`, `list-style-type: disc`, and padding for markers — identical to `<ul>`.

**Beginner-Friendly Explanation**
`<menu>` is the container for a menu of commands or interactive items. It looks and behaves like `<ul>`, but the specification reserves it for lists of commands rather than general prose items.

---

### Purposes

- **To represent a list of commands or interactive controls** — for example, a toolbar of buttons or a set of actions.
- **To provide a semantic alternative to `<ul>`** when the list’s items are commands rather than general content.
- **To support CSS-driven menu layouts** such as horizontal toolbars and command palettes.
- **To expose list semantics to assistive technologies**, exactly as `<ul>` does.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<menu>
  <li><button type="button">Cut</button></li>
  <li><button type="button">Copy</button></li>
  <li><button type="button">Paste</button></li>
</menu>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<menu>` | The menu container. |
| `<li>` | One or more menu items; the only permitted non-script direct children. |
| `</menu>` | The end tag; both start and end tags are mandatory. |

#### Syntax Rules

1. **Content model is zero or more `<li>` elements** plus script-supporting elements.
2. **Both start and end tags are mandatory**; tag omission is not permitted.
3. **The element accepts only global attributes**; `type` and `label` are obsolete.
4. **The element may be used wherever flow content is expected**.

#### Constraints and Limitations

- **`<menu>` must not contain elements other than `<li>`** (and `<script>`/`<template>`) as direct children.
- **The `<li>` content model is stricter than inside `<ul>`**: only phrasing content is permitted, so paragraphs, headings, and nested lists are non-conforming.
- **No browser provides menu-specific behaviour**: there is no context-menu, toolbar, or command-palette behaviour attached to `<menu>` in the current standard.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Toolbar of Commands

**Setup Guide**: Save as `menu-toolbar.html` and open it in a browser. Inspect the accessibility tree to confirm the list role.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Menu Toolbar</title>
  <style>
    /* Lay the menu items out horizontally like a toolbar */
    menu {
      list-style: none;
      display: flex;
      gap: 8px;
      padding: 0;
      margin: 0;
    }
    menu button { padding: 8px 16px; }
  </style>
</head>
<body>
  <!-- Each li contains only phrasing content: a button -->
  <menu>
    <li><button type="button">Bold</button></li>
    <li><button type="button">Italic</button></li>
    <li><button type="button">Underline</button></li>
  </menu>
</body>
</html>
```

**Expected output**: A horizontal toolbar of three buttons. A screen reader announces “list, 3 items” and exposes each button as a list item.

**Why this output occurs**: The `<menu>` element groups the command items, and CSS removes the default bullet markers and lays the items out horizontally. The accessibility tree still exposes the list role and item count.

---

#### Example 2: A Simple Command List

**Setup Guide**: Save as `menu-commands.html`. Compare with the same markup using `<ul>` — the rendering is identical.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Command Menu</title>
</head>
<body>
  <h1>Actions</h1>

  <!-- The menu groups commands rather than general items -->
  <menu>
    <li><a href="/edit">Edit this page</a></li>
    <li><a href="/history">View history</a></li>
    <li><a href="/delete">Delete this page</a></li>
  </menu>
</body>
</html>
```

**Expected output**: A bulleted list of three links, rendered identically to the same markup inside a `<ul>`.

**Why this output occurs**: The `<menu>` element provides the same default list styling as `<ul>`, and the `<li>` children contain only phrasing content (links), which is valid for `<menu>`.

---

### Real-World Cases

1. **Toolbars**: A row of formatting buttons where each button is a menu command.
2. **Command palettes**: A list of available actions in an application interface.
3. **Context menus**: A list of actions presented when a user right-clicks (though the browser provides no context-menu behaviour for `<menu>` itself).

---

### References

- MDN Web Docs — `<menu>`: The Menu element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/menu
- WHATWG — The `menu` element — https://html.spec.whatwg.org/multipage/grouping-content.html#the-menu-element
- W3C — HTML 5.2: The `menu` element — https://www.w3.org/TR/html52/grouping-content.html#the-menu-element

---

## 2. `<li>` — The Menu Item

### Definitions

**Core Definition**
The `<li>` element represents a single item in a list. When its parent is a `<menu>`, its content model is **phrasing content** — unlike when its parent is `<ul>` or `<ol>`, where the content model is flow content.

**Technical Definition**
`<li>` has no content category of its own; its role is defined by its parent list. When the parent is `<menu>`, the content model is **phrasing content**, meaning the item may contain text and inline elements (`<a>`, `<button>`, `<span>`, `<em>`, `<img>`, etc.) but **not** flow content such as paragraphs, headings, or nested lists. The `value` attribute is not permitted when the parent is `<menu>`. In the accessibility tree, `<li>` maps to the ARIA role `listitem`.

**Beginner-Friendly Explanation**
`<li>` is one item in a menu. Inside a `<menu>`, it can only contain inline content — text, links, buttons — not block-level things like paragraphs or nested lists. This is stricter than `<li>` inside a `<ul>`.

---

### Purposes

- **To represent a single menu item**.
- **To provide the content of a `<menu>` list**.
- **To support interactive commands** such as buttons and links within menu items.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<menu>
  <li><button type="button">Action</button></li>
</menu>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<li>` | The menu item start tag. |
| Content | **Phrasing content only** when the parent is `<menu>`. |
| `</li>` | The end tag; may be omitted before another `<li>` or at the end of the parent. |

#### Syntax Rules

1. **`<li>` must be a child of `<ul>`, `<ol>`, or `<menu>`**.
2. **When the parent is `<menu>`, the content model is phrasing content** — not flow content.
3. **The `value` attribute is not permitted** when the parent is `<menu>`.
4. **The `li` end tag may be omitted** when followed by another `<li>` or when there is no more content in the parent.

#### Constraints and Limitations

- **Paragraphs, headings, and nested lists are non-conforming** inside an `<li>` inside a `<menu>`.
- **The `value` attribute has no effect inside `<menu>`**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Valid Menu Item with Phrasing Content

**Setup Guide**: Save as `li-menu-valid.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Menu Item</title>
</head>
<body>
  <menu>
    <!-- Valid: the li contains only phrasing content (a button) -->
    <li><button type="button">Save</button></li>
    <li><button type="button">Cancel</button></li>
  </menu>
</body>
</html>
```

**Expected output**: A bulleted list of two buttons.

**Why this output occurs**: The `<li>` children contain only phrasing content, satisfying the `<menu>` content model.

---

#### Example 2: Invalid Menu Item with Flow Content

**Setup Guide**: Save as `li-menu-invalid.html`. Run an HTML validator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Invalid Menu Item</title>
</head>
<body>
  <menu>
    <!-- INVALID: a paragraph is flow content, not phrasing content -->
    <li><p>Save the file</p></li>
  </menu>
</body>
</html>
```

**Expected output**: The browser may render the paragraph, but the markup is non-conforming and a validator will report an error.

**Why this output occurs**: When the parent is `<menu>`, the `<li>` content model is phrasing content; a `<p>` is flow content and violates the content model.

---

### Real-World Cases

1. **Toolbar items**: Each `<li>` wraps a button or link command.
2. **Command lists**: Each `<li>` contains a short action label.
3. **Context menu items**: Each `<li>` contains a single command.

---

### References

- MDN Web Docs — `<li>`: The List Item element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/li
- WHATWG — The `li` element: Content model — https://html.spec.whatwg.org/multipage/grouping-content.html#the-li-element

---

## 3. Menu Best Practices

### Definitions

**Core Definition**
Menu best practices are guidelines for choosing `<menu>` appropriately, structuring menu items correctly, and ensuring that the semantic distinction from `<ul>` is meaningful — so that the element is used only when it adds clarity rather than confusion.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, which explicitly describes `<menu>` as “a semantic alternative to the `ul` element” with “no other distinguishing features”, and by WCAG 1.3.1 (Info and Relationships). They encompass using `<menu>` only when the list genuinely represents commands or interactive controls, respecting the stricter `<li>` content model, avoiding obsolete attributes, and preferring `<ul>` when the semantic distinction is not meaningful.

**Beginner-Friendly Explanation**
Best practices help you decide when to use `<menu>` and when to stick with `<ul>`: use `<menu>` only when the list is genuinely a set of commands or interactive items, keep the items simple (no paragraphs or nested lists), and remember that browsers treat it exactly like `<ul>`.

---

### Purposes

- **To ensure semantic correctness** — `<menu>` for commands, `<ul>` for general items, `<ol>` for sequences.
- **To respect the stricter content model** — only phrasing content inside `<li>` inside `<menu>`.
- **To avoid obsolete attributes** — `type` and `label` must not be used.
- **To maintain accessibility** — the list role and item count are exposed identically to `<ul>`.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `<menu>` only for command lists** | Toolbars, action lists, command palettes. |
| **Respect the phrasing-content restriction** | No paragraphs, headings, or nested lists inside `<li>` inside `<menu>`. |
| **Do not use obsolete `type` or `label` attributes** | They have no effect and are non-conforming. |
| **Prefer `<ul>` when the distinction is not meaningful** | `<ul>` is more flexible (flow content) and universally understood. |
| **Use CSS for presentation** | `list-style-type`, `display: flex/grid`, and `padding` control the visual layout. |
| **Preserve list semantics when styling** | Use `role="list"` if `list-style: none` is applied and semantics may be lost. |

#### Syntax Rules

1. **Semantic correctness**: Use `<menu>` when the list represents commands or interactive controls; use `<ul>` for general unordered items.
2. **Structural correctness**: `<li>` for every item; phrasing content only inside `<li>` inside `<menu>`.
3. **Attribute discipline**: Global attributes only; no `type` or `label`.

#### Constraints and Limitations

- **The distinction from `<ul>` is purely semantic**: there is no rendering or accessibility difference.
- **The stricter content model limits `<menu>` to simple items**, which is why `<ul>` is more commonly used.
- **Browser and screen-reader support is universal**, but the element is rarely encountered, so some developers may not recognise its intent.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct vs Incorrect Use of `<menu>`

**Setup Guide**: Save as `menu-best-practice.html`. Run an HTML validator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Menu Best Practices</title>
</head>
<body>
  <!-- GOOD: menu for a list of commands -->
  <menu>
    <li><button type="button">Cut</button></li>
    <li><button type="button">Copy</button></li>
    <li><button type="button">Paste</button></li>
  </menu>

  <!-- BAD: menu for a general grocery list (use ul instead) -->
  <!--
  <menu>
    <li>Milk</li>
    <li>Eggs</li>
    <li>Bread</li>
  </menu>
  -->

  <!-- BAD: flow content inside an li inside a menu -->
  <!--
  <menu>
    <li><p>Cut the selected text</p></li>
  </menu>
  -->
</body>
</html>
```

**Expected output**: The good example uses `<menu>` for commands and produces valid markup. The bad examples either misuse `<menu>` for general items or violate the phrasing-content restriction.

**Why this output occurs**: `<menu>` is intended for command lists, and its `<li>` children must contain only phrasing content. Violating either rule produces non-conforming markup.

---

#### Example 2: A Styled Menu Toolbar

**Setup Guide**: Save as `menu-styled.html`. Inspect the accessibility tree to confirm the list role.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Styled Menu Toolbar</title>
  <style>
    menu {
      list-style: none;   /* remove bullets */
      display: flex;
      gap: 8px;
      padding: 0;
      margin: 0;
    }
    menu button {
      padding: 8px 16px;
      border: 1px solid #ccc;
      background: #f8f8f8;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <!-- role="list" restores list semantics when list-style: none is used -->
  <menu role="list" aria-label="Formatting options">
    <li><button type="button">Bold</button></li>
    <li><button type="button">Italic</button></li>
    <li><button type="button">Underline</button></li>
  </menu>
</body>
</html>
```

**Expected output**: A horizontal toolbar of formatted buttons. Screen readers announce “Formatting options, list, 3 items” because of the `aria-label` and `role="list"`.

**Why this output occurs**: The `<menu>` retains its list semantics, and the `role="list"` restores them in browsers that may remove them when `list-style: none` is applied. The `aria-label` provides a name for the menu group.

---

### Real-World Cases

1. **Application toolbars**: Formatting, editing, and navigation commands grouped in a toolbar.
2. **Command palettes**: A list of available actions in a productivity app.
3. **Contextual action lists**: A set of commands associated with a selected item.

---

### References

- MDN Web Docs — `<menu>`: Accessibility — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/menu
- WHATWG — The `menu` element — https://html.spec.whatwg.org/multipage/grouping-content.html#the-menu-element
- W3C — HTML 5.2: The `menu` element — https://www.w3.org/TR/html52/grouping-content.html#the-menu-element
- W3C — Technique H48: Using ol, ul and dl for lists or groups of links — https://www.w3.org/WAI/WCAG22/Techniques/html/H48

---

## Summary Table of Menu Concepts

| Concept | Key Takeaway |
|---|---|
| **`<menu>`** | Semantic alternative to `<ul>`; same rendering and accessibility; intended for command lists; stricter `<li>` content model (phrasing content only). |
| **`<li>`** | One menu item; phrasing content when the parent is `<menu>`; end tag may be omitted before another `<li>`. |
| **Best practices** | Use `<menu>` only for genuine command lists; respect the phrasing-content restriction; no obsolete attributes; prefer `<ul>` when the distinction is not meaningful. |

---

## Final Notes

- **`<menu>` is semantically distinct but practically identical to `<ul>`**: The specification says it is “a semantic alternative to the `ul` element” with “no other distinguishing features”.
- **The stricter content model is the key difference**: `<li>` inside `<menu>` accepts only phrasing content, not the flow content permitted inside `<li>` inside `<ul>`.
- **Obsolete attributes must not be used**: `type` and `label` have no effect and are non-conforming.
- **Browser and screen-reader support is universal**: Both expose `<menu>` as a `list`, exactly like `<ul>`.
- **`<ul>` is the safer default**: Because `<menu>` is rarely used and less flexible, `<ul>` is the more common choice for almost every list — including navigation menus and general item lists.
- **Validate and test**: Use HTML validators to catch flow content inside `<li>` inside `<menu>`, and accessibility checkers to verify list semantics.

---

This cheat sheet has covered the `<menu>` element, its `<li>` children, and menu best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate menu markup against the WHATWG HTML Living Standard and WCAG 1.3.1 (Info and Relationships).