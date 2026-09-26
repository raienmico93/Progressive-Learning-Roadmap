Here’s a structured guide to HTML attributes covering the topics you listed.

## 1. Attribute syntax

Attributes appear **inside the opening/start tag** of an element:

```html
<element attribute="value" another-attribute='value'>
```

Basic rules:

- Written after the element name, separated by whitespace.
- Form is usually `name="value"`.
- Spaces around `=` are tolerated in HTML but not conventional:
  ```html
  <a href = "/about">About</a> <!-- works, but avoid -->
  ```
- Attribute names are **case-insensitive in HTML**, but lowercase is strongly recommended:
  ```html
  <input TYPE="text"> <!-- works in HTML -->
  <input type="text"> <!-- preferred -->
  ```
- In XHTML/XML, attribute names are case-sensitive and must be lowercase, and values must be quoted.
- Attributes go in the start tag, not the end tag:
  ```html
  <p class="note">Text</p> <!-- correct -->
  <p>Text</p class="note"> <!-- invalid -->
  ```

## 2. Attribute values

Values may be:

```html
<input type="text">
<input type='text'>
<input type=text>
```

Preferred style:

```html
<input type="text">
```

Rules:

- Quote values with double quotes by default.
- Use single quotes if the value contains double quotes:
  ```html
  <div title='He said "hello"'>
  ```
- Or escape with entities:
  ```html
  <div title="He said &quot;hello&quot;">
  ```
- Unquoted values are allowed only if they contain no spaces, quotes, `=`, `<`, `>`, or backticks. Quoting is safer.
- Character references are parsed in attribute values:
  ```html
  <a title="Tom &amp; Jerry">Link</a>
  ```
- Empty values are valid and sometimes meaningful:
  ```html
  <img src="decorative.png" alt="">
  ```
- Attribute names are generally case-insensitive in HTML, but values may be case-sensitive depending on the attribute. For example, `id` and `class` values are generally case-sensitive, while some enumerated values like `type="TEXT"` are treated case-insensitively.

## 3. Global attributes

Global attributes can be used on almost all HTML elements.

Common global attributes:

| Attribute | Purpose |
|---|---|
| `id` | Unique identifier for one element |
| `class` | Space-separated class names |
| `style` | Inline CSS |
| `title` | Advisory tooltip text |
| `lang` | Language of element content |
| `dir` | Text direction: `ltr`, `rtl`, `auto` |
| `hidden` | Hides element |
| `tabindex` | Keyboard focus order |
| `accesskey` | Keyboard shortcut |
| `contenteditable` | Makes content editable |
| `draggable` | Enables drag behavior |
| `spellcheck` | Enables/disables spellcheck |
| `translate` | Whether content should be translated |
| `data-*` | Custom data attributes |
| `role` | ARIA role |
| `aria-*` | ARIA accessibility attributes |

Examples:

```html
<div id="main" class="container" lang="en" data-user-id="42">
  ...
</div>

<button aria-label="Close dialog">×</button>
```

Notes:

- `id` should be unique in the document.
- `class` can be reused.
- Custom data should use `data-*`:
  ```html
  <div data-user-id="42" data-role="admin"></div>
  ```
- Prefer native HTML semantics before adding ARIA.

## 4. Element-specific attributes

These are valid only on certain elements.

Examples:

| Element | Common attributes |
|---|---|
| `<a>` | `href`, `target`, `rel`, `download`, `hreflang`, `type` |
| `<img>` | `src`, `alt`, `width`, `height`, `srcset`, `sizes`, `loading`, `decoding` |
| `<input>` | `type`, `name`, `value`, `placeholder`, `required`, `disabled`, `readonly`, `checked`, `min`, `max`, `step`, `pattern`, `autocomplete` |
| `<form>` | `action`, `method`, `enctype`, `target`, `novalidate` |
| `<script>` | `src`, `async`, `defer`, `type`, `crossorigin`, `integrity` |
| `<video>` | `src`, `controls`, `autoplay`, `loop`, `muted`, `poster`, `preload` |
| `<meta>` | `charset`, `name`, `content`, `http-equiv` |
| `<ol>` | `type`, `start`, `reversed` |
| `<td>` | `colspan`, `rowspan`, `headers` |

Example:

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Visit example
</a>

<img src="photo.jpg" alt="A cat" width="600" height="400" loading="lazy">
```

Using an attribute on the wrong element is usually invalid. For example, `href` on a `<div>` has no standard meaning.

## 5. Boolean attributes

A boolean attribute is **true when present** and **false when absent**.

Examples:

```html
<input disabled>
<input required>
<input checked>
<select multiple>...</select>
<video controls></video>
<script async></script>
<script defer></script>
<details open>...</details>
```

These are all treated as true:

```html
<input disabled>
<input disabled="">
<input disabled="disabled">
```

This is misleading and non-conforming, but browsers still treat it as true:

```html
<input disabled="false"> <!-- still disabled -->
```

To make it false, omit the attribute:

```html
<input> <!-- not disabled -->
```

Common boolean attributes include:

`disabled`, `checked`, `selected`, `required`, `readonly`, `multiple`, `autofocus`, `hidden`, `async`, `defer`, `loop`, `muted`, `controls`, `autoplay`, `open`, `novalidate`, `formnovalidate`, `reversed`, `allowfullscreen`.

In JavaScript, use the property or `removeAttribute()`:

```js
input.disabled = false;
input.removeAttribute("disabled");
```

## 6. Attribute ordering

Attribute order has **no semantic effect** in HTML.

These are equivalent:

```html
<a href="/about" class="nav-link" id="about-link">About</a>
<a id="about-link" class="nav-link" href="/about">About</a>
```

Conventions for readability:

```html
<input
  type="text"
  id="username"
  name="username"
  class="form-input"
  placeholder="Username"
  required
  autocomplete="username"
>
```

A common order is:

1. `type`
2. `id`
3. `name`
4. `class`
5. `data-*`
6. `src` / `href`
7. other functional attributes
8. boolean attributes
9. ARIA attributes

Minifiers and frameworks may reorder attributes. That does not change behavior.

## 7. Multiple attributes

Multiple attributes are separated by whitespace:

```html
<input type="email" id="email" name="email" required>
```

Whitespace can include spaces, tabs, or newlines:

```html
<img
  src="photo.jpg"
  alt="A cat"
  width="600"
  height="400"
>
```

Duplicate attributes:

```html
<div id="first" id="second"></div>
```

In HTML parsing, the **first occurrence usually wins**; later duplicates are ignored. This is a parse error, so avoid duplicates.

## 8. Valid attribute usage

Checklist:

- Use only global attributes plus attributes valid for that element.
- Use `data-*` for custom data.
- Keep `id` values unique.
- Quote attribute values, especially if they contain spaces or special characters.
- Do not put attributes in closing tags.
- Do not invent boolean values like `disabled="false"`; omit the attribute instead.
- Use lowercase attribute names by convention.
- Prefer CSS over obsolete presentational attributes like `align`, `bgcolor`, or `border` where possible.
- Use ARIA attributes correctly and only when native HTML cannot express the meaning.
- For links opening a new tab, consider:
  ```html
  <a href="..." target="_blank" rel="noopener noreferrer">...</a>
  ```
- Validate with the W3C HTML Validator when precision matters.

Quick valid example:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attribute Demo</title>
</head>
<body>
  <a href="/about" class="nav-link" id="about-link">About</a>

  <img src="cat.jpg" alt="A cat" width="600" height="400" loading="lazy">

  <form action="/subscribe" method="post">
    <input type="email" name="email" required>
    <button type="submit">Subscribe</button>
  </form>
</body>
</html>
```