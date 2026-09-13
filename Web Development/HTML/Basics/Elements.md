# HTML Elements

HTML elements are the **building blocks** of every web page. Understanding how they're structured, how they nest, and how they relate to one another is fundamental to writing correct and maintainable HTML.

---

## 1. What Is an HTML Element?

An **element** is a complete unit of markup that typically consists of:

```
Opening tag + Content + Closing tag
```

**Example:**

```html
<p>Hello, World!</p>
```

| Part | Code | Purpose |
|---|---|---|
| Opening tag | `<p>` | Marks the start of the element |
| Content | `Hello, World!` | The element's data |
| Closing tag | `</p>` | Marks the end of the element |

Together, these three parts form **one `<p>` element**.

### Elements vs. Tags

These terms are often confused:

| Term | Meaning |
|---|---|
| **Tag** | The markup itself — `<p>` or `</p>` |
| **Element** | The complete structure — `<p>Hello</p>` |

**Rule of thumb:** A tag is a piece; an element is the whole thing.

---

## 2. Opening Tags (Start Tags)

An **opening tag** (also called a **start tag**) marks the beginning of an element.

### Syntax

```html
<tagname>
```

Or with attributes:

```html
<tagname attribute="value">
```

### Examples

```html
<p>
<div class="container">
<a href="https://example.com">
<img src="photo.jpg" alt="A photo">
```

### Rules

- Begins with `<` and ends with `>`
- Contains the element name immediately after `<`
- Attributes (if any) follow the tag name, separated by spaces
- Attribute values are enclosed in quotes (single or double; double is conventional)
- **No forward slash** before the `>` in HTML5 (that's XHTML style)

```html
<!-- HTML5 — correct -->
<br>

<!-- XHTML — valid but unnecessary in HTML5 -->
<br />
```

---

## 3. Closing Tags (End Tags)

A **closing tag** (also called an **end tag**) marks the end of an element.

### Syntax

```html
</tagname>
```

### Examples

```html
</p>
</div>
</a>
</body>
```

### Rules

- Begins with `</` and ends with `>`
- Contains the same element name as the opening tag
- **No attributes** in closing tags
- Must match the opening tag's name exactly (case-insensitive in HTML)

```html
<!-- Correct -->
<p>Text</p>

<!-- Incorrect: mismatched tag names -->
<p>Text</div>
```

### Case Sensitivity

HTML tag names are **case-insensitive**:

```html
<P>Text</P>
<p>Text</p>
<P>Text</p>
```

All three are valid and equivalent. However, **lowercase is the universal convention** and strongly recommended for consistency and readability.

---

## 4. Content

**Content** is everything between the opening and closing tags.

### Types of Content

| Type | Description | Example |
|---|---|---|
| **Text** | Plain character data | `<p>Hello</p>` |
| **Child elements** | Nested HTML elements | `<div><p>Hi</p></div>` |
| **Mixed content** | Both text and elements | `<p>Hello <strong>World</strong></p>` |
| **Empty** | No content (see void elements) | `<br>` |

### Examples

```html
<!-- Text content -->
<h1>Welcome</h1>

<!-- Element content -->
<div>
  <p>Nested paragraph</p>
</div>

<!-- Mixed content -->
<p>This is <em>emphasized</em> text.</p>
```

---

## 5. Element Nesting

**Nesting** means placing one element inside another.

### Example

```html
<div>
  <p>
    This is a <strong>bold</strong> word.
  </p>
</div>
```

Here:
- `<strong>` is nested inside `<p>`
- `<p>` is nested inside `<div>`

### Nesting Rules

1. **Proper closing order (LIFO):** The last element opened must be the first closed.

```html
<!-- CORRECT -->
<p>Text with <em>emphasis</em>.</p>

<!-- INCORRECT: overlapping tags -->
<p>Text with <em>emphasis.</p></em>
```

2. **Block vs. inline nesting:** Some elements can only contain certain types of children.

```html
<!-- CORRECT: block inside block -->
<div>
  <p>Paragraph</p>
</div>

<!-- INCORRECT: block inside inline -->
<span>
  <p>Paragraph</p>  <!-- <p> is block, <span> is inline -->
</span>
```

3. **Certain elements have restricted content models** — e.g., `<ul>` should only contain `<li>` elements.

```html
<!-- CORRECT -->
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>

<!-- INCORRECT -->
<ul>
  <div>Item 1</div>
</ul>
```

### Why Nesting Matters

- **Validity:** Improper nesting produces invalid HTML
- **Rendering:** Browsers auto-correct bad nesting unpredictably
- **CSS/JS:** Selectors and DOM traversal depend on correct hierarchy

---

## 6. Parent-Child Relationships

Nesting creates a **tree hierarchy** with parent-child relationships.

### Terminology

| Term | Definition | Example |
|---|---|---|
| **Parent** | An element that directly contains another | `<div>` is the parent of `<p>` |
| **Child** | An element directly contained by another | `<p>` is the child of `<div>` |
| **Ancestor** | Any element above in the tree (parent, grandparent, etc.) | `<body>` is an ancestor of `<p>` |
| **Descendant** | Any element below in the tree | `<strong>` is a descendant of `<div>` |

### Example Tree

```html
<body>
  <div>
    <p>
      Hello <strong>World</strong>
    </p>
  </div>
</body>
```

**Tree representation:**

```
body
└── div
    └── p
        └── strong
```

**Relationships:**
- `<body>` is the **parent** of `<div>`
- `<div>` is the **child** of `<body>` and the **parent** of `<p>`
- `<p>` is the **child** of `<div>` and the **parent** of `<strong>`
- `<strong>` is the **child** of `<p>` and a **descendant** of `<body>`

### CSS and JavaScript Implications

CSS selectors use these relationships:

```css
/* Direct child */
div > p { color: blue; }

/* Descendant */
div p { color: red; }
```

JavaScript uses them too:

```javascript
element.parentElement       // Parent
element.children            // Children
element.firstElementChild   // First child
element.lastElementChild    // Last child
```

---

## 7. Sibling Elements

**Siblings** are elements that share the same parent.

### Example

```html
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
  <li>Item 3</li>
</ul>
```

All three `<li>` elements are **siblings** — they share the same parent `<ul>`.

### Tree View

```
ul
├── li (Item 1)
├── li (Item 2)
└── li (Item 3)
```

### Adjacent Siblings

Elements that are **directly next to each other** are **adjacent siblings**.

```html
<h1>Title</h1>
<p>First paragraph</p>   <!-- adjacent to <h1> -->
<p>Second paragraph</p>  <!-- adjacent to the first <p> -->
```

### CSS and JavaScript

```css
/* Adjacent sibling */
h1 + p { margin-top: 0; }

/* General sibling */
h1 ~ p { color: gray; }
```

```javascript
element.nextElementSibling       // Next sibling
element.previousElementSibling   // Previous sibling
```

---

## 8. Void Elements

**Void elements** (also called **empty elements** or **self-closing elements**) are elements that **cannot have content** and **do not have a closing tag**.

### Why They Exist

Void elements represent things that don't wrap around content — like a line break, an image, or a horizontal rule. There's nothing to put *between* an opening and closing tag.

### Complete List of Void Elements in HTML5

| Element | Purpose |
|---|---|
| `<area>` | Image map area |
| `<base>` | Base URL for relative links |
| `<br>` | Line break |
| `<col>` | Table column |
| `<embed>` | External content embed |
| `<hr>` | Thematic break (horizontal rule) |
| `<img>` | Image |
| `<input>` | Form input |
| `<link>` | External resource link |
| `<meta>` | Metadata |
| `<param>` | Plugin parameter (deprecated) |
| `<source>` | Media source |
| `<track>` | Text track for media |
| `<wbr>` | Word break opportunity |

### Syntax

```html
<!-- Correct HTML5 -->
<br>
<img src="photo.jpg" alt="A photo">
<input type="text" name="username">
<hr>
<meta charset="UTF-8">
<link rel="stylesheet" href="style.css">
```

### The XHTML Self-Closing Style

XHTML required a trailing slash:

```html
<br />
<img src="photo.jpg" alt="A photo" />
```

This is **valid in HTML5 but unnecessary**. The HTML5 spec allows it for compatibility, but the trailing slash has **no meaning** in HTML — it doesn't make the element "self-closing" (void elements are always void).

### Common Mistake

```html
<!-- WRONG: void elements must NOT have closing tags -->
<br></br>
<img src="photo.jpg"></img>
```

Adding a closing tag to a void element is invalid HTML. Browsers will ignore the closing tag, but validators will flag it as an error.

### Nesting and Void Elements

Void elements **cannot contain any content or child elements** — they have no "inside."

```html
<!-- CORRECT -->
<img src="photo.jpg" alt="A photo">

<!-- IMPOSSIBLE: nothing can go inside a void element -->
<img src="photo.jpg">
  <p>This is invalid</p>
</img>
```

---

## 9. Complete Example

Here's a document that demonstrates every concept covered:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">                         <!-- void element -->
  <title>Element Structure Demo</title>
  <link rel="stylesheet" href="style.css">        <!-- void element -->
</head>
<body>
  <header>                                        <!-- parent of h1, nav -->
    <h1>My Site</h1>                              <!-- child of header -->
    <nav>                                         <!-- sibling of h1 -->
      <a href="/">Home</a>                        <!-- child of nav -->
      <a href="/about">About</a>                  <!-- sibling of first a -->
    </nav>
  </header>

  <main>                                          <!-- sibling of header -->
    <article>                                     <!-- child of main -->
      <h2>First Post</h2>                         <!-- child of article -->
      <p>                                         <!-- sibling of h2 -->
        Welcome to <strong>my blog</strong>.      <!-- child of p -->
      </p>
      <hr>                                        <!-- void element -->
      <img src="photo.jpg" alt="A photo">         <!-- void element -->
    </article>
  </main>
</body>
</html>
```

**Tree structure:**

```
html
├── head
│   ├── meta
│   ├── title
│   └── link
└── body
    ├── header
    │   ├── h1
    │   └── nav
    │       ├── a
    │       └── a
    └── main
        └── article
            ├── h2
            ├── p
            │   └── strong
            ├── hr
            └── img
```

---

## 10. Quick Reference Table

| Concept | Definition | Example |
|---|---|---|
| **Element** | Opening tag + content + closing tag | `<p>Hi</p>` |
| **Opening tag** | Marks the start | `<p>` |
| **Closing tag** | Marks the end | `</p>` |
| **Content** | Data between tags | `Hi` |
| **Nesting** | Element inside another | `<p><em>Hi</em></p>` |
| **Parent** | Directly contains another element | `<div>` in `<div><p></p></div>` |
| **Child** | Directly contained | `<p>` in `<div><p></p></div>` |
| **Sibling** | Shares the same parent | Two `<li>` in a `<ul>` |
| **Void element** | No content, no closing tag | `<br>`, `<img>` |

---

## Key Takeaways

1. An **element** = opening tag + content + closing tag.
2. A **tag** is markup; an **element** is the complete structure.
3. **Closing order matters** — last opened, first closed.
4. **Nesting** creates a **tree hierarchy** with parent-child relationships.
5. **Siblings** share the same parent.
6. **Void elements** have no content and no closing tag — never write `<br></br>`.
7. Proper structure affects **validity, rendering, CSS, JavaScript, and accessibility**.
8. The browser builds a **DOM tree** from these relationships — this tree is what CSS and JavaScript interact with.

---

Would you like me to continue with **"4. HTML Attributes"** next, or expand on any part (e.g., the full content model rules, or how browsers handle invalid nesting)?