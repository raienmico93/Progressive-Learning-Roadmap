# Pug — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Pug (formerly Jade) is a high-performance, whitespace-sensitive template engine for Node.js and browsers that compiles concise, indentation-based markup into HTML.

**Technical Definition**
Pug is a template engine heavily influenced by Haml and implemented in JavaScript. It uses indentation to express tag nesting, eliminates closing tags, and supports JavaScript expressions, template inheritance, includes, and mixins. Pug compiles `.pug` files into JavaScript render functions and is Express-compliant: it exports a function named `__express(filePath, options, callback)` that `res.render()` invokes to render template code. As of version 2, “pug” is the official package name.

**Beginner-Friendly Explanation**
Pug lets you write HTML much more concisely. Instead of typing `<div class="card"><h1>Title</h1></div>`, you write `.card` and indent `h1 Title` beneath it. Pug figures out the tags and closing brackets for you, making templates shorter and easier to scan.

---

### Key Characteristics

- **Whitespace-sensitive** — Indentation defines parent-child relationships; closing tags are omitted.
- **JavaScript-native** — Inline JavaScript, conditionals, loops, and expressions are first-class features.
- **Express-first** — The default view engine for Express's application generator.
- **Composable** — Includes, template inheritance (`extends` + `block`), and mixins enable modular, reusable templates.
- **Secure-by-default output** — Buffered output (`=`) is HTML-escaped automatically; unescaped output (`!=`) must be used deliberately.
- **Performance-conscious** — Template caching prevents recompilation on every request in production.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- The `pug` package installed via npm (`npm install pug`).
- Basic familiarity with HTML structure and JavaScript expressions.
- Understanding of Express routing and `res.render()`.

---

### Related Programming Areas

- **Templating** — Pug is one of several Express view engines alongside EJS and Handlebars.
- **Server-side rendering (SSR)** — Pug renders full HTML pages on the server.
- **Security (XSS prevention)** — Escaped interpolation and careful attribute handling are critical.
- **Component-based design** — Mixins provide reusable UI blocks without a frontend framework.
- **Build tooling** — Pug can be compiled ahead of time via the `pug-cli` package.

---

## Core Concept 1: Pug

### Definitions

**Core Definition**
Pug is a template engine that compiles indentation-based markup into HTML, using JavaScript for dynamic logic and data insertion.

**Technical Definition**
Pug transforms `.pug` templates into HTML through a compilation pipeline that lexes, parses, links, and generates JavaScript render functions. It is implemented for Node.js and browsers, supports filters (e.g., Markdown), and integrates with Express via the `__express` convention. Pug was renamed from Jade in version 2 due to a trademark issue; the syntax also changed, so migrating from Jade requires attention to deprecations and removals.

**Beginner-Friendly Explanation**
Pug is a tool that takes a very short, indentation-based file and turns it into full HTML. You write less, and Pug fills in the brackets. It works especially well with Express.

---

### Purposes

- To write HTML more concisely by eliminating closing tags and repetitive markup.
- To embed JavaScript logic (loops, conditionals, variables) directly in templates.
- To integrate seamlessly with Express as a view engine.
- To support modular, reusable template structures through includes, inheritance, and mixins.

---

### Syntax Rules and Structure

#### Complete General Syntax: Basic Pug Document

```pug
doctype html
html(lang="en")
  head
    title= pageTitle
  body
    h1 Pug Demo
    p Welcome, #{name}!
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `doctype html` | Emits `<!DOCTYPE html>`. |
| `html(lang="en")` | Tag with parenthesized attributes. |
| Indentation | Defines nesting. |
| `=` and `#{}` | Evaluate and interpolate JavaScript. |

#### Syntax Rules

- Indentation must be consistent (spaces or tabs, but not mixed).
- Tags do not require closing tags; nesting is determined by indentation.
- Attribute values can be JavaScript expressions.

#### Constraints and Limitations

- Inconsistent indentation causes compilation errors.
- Pug is whitespace-sensitive, so copying and pasting into environments that normalize whitespace can break templates.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Express Application with Pug

```javascript
// app.js
const express = require('express');
const path = require('path');
const app = express();

app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');

app.get('/', (req, res) => {
  res.render('index', {
    pageTitle: 'Pug Demo',
    name: 'Alice',
    items: ['One', 'Two', 'Three']
  });
});

app.listen(3000);
```

```pug
//- views/index.pug
doctype html
html(lang="en")
  head
    title= pageTitle
  body
    h1 Welcome, #{name}!
    ul
      each item in items
        li= item
```

**Step-by-Step Explanation**

1. **Engine registration** — `app.set('view engine', 'pug')` enables Pug rendering.
2. **Rendering** — `res.render('index', { ... })` passes locals to Pug.
3. **Compilation** — Pug compiles the template into a function and executes it with the locals.

**Expected Output**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Pug Demo</title>
  </head>
  <body>
    <h1>Welcome, Alice!</h1>
    <ul>
      <li>One</li>
      <li>Two</li>
      <li>Three</li>
    </ul>
  </body>
</html>
```

**Why This Output Occurs**

- `each` iterates the array; `=` emits escaped values; indentation determines the nesting inside `<ul>` and `<body>`.

---

### Real-World Cases

- **Express-generated projects** — Pug is the default view engine.
- **Server-rendered dashboards** — Concise templates for complex layouts.
- **Prototyping** — Faster markup authoring with fewer characters.

---

## Core Concept 2: Pug Syntax

### Definitions

**Core Definition**
Pug syntax is the indentation-based markup language that Pug compiles into HTML, consisting of tags, attributes, text, and JavaScript expressions.

**Technical Definition**
Pug syntax includes tag declarations (bare words like `div` or `h1`), implicit `div` shorthand (`.` and `#`), parenthesized attributes, pipe text, block text (`. `), unbuffered code (`-`), buffered code (`=`), and interpolation (`#{}` and `!{}`). Control flow uses `if`, `else`, `each`, `while`, and `case`. Comments are `//` (rendered) and `//-` (silent).

**Beginner-Friendly Explanation**
Pug syntax is the set of rules for writing Pug. Tags are words on their own line, indentation nests them, attributes go in parentheses, and JavaScript goes after `=` or inside `#{}`.

---

### Purposes

- To provide a concise, readable alternative to HTML.
- To embed JavaScript control flow and expression evaluation.
- To support comments and text blocks without HTML noise.

---

### Syntax Rules and Structure

#### Complete General Syntax: Syntax Overview

```pug
//- Silent comment (not rendered)
// Rendered comment

doctype html
html
  body
    h1.title#main Heading
    p Text with #{variable}
    - const x = 10
    p= x
    if condition
      p Condition true
    else
      p Condition false
```

**Component Breakdown**

| Syntax | Meaning |
|--------|---------|
| `//-` / `//` | Silent / rendered comment. |
| `.class` / `#id` | Class and ID shorthand. |
| `-` | Unbuffered JavaScript (no output). |
| `=` | Buffered, HTML-escaped output. |
| `#{}` | Escaped string interpolation. |
| `!{}` | Unescaped interpolation (dangerous with user input). |

#### Syntax Rules

- Tags may omit parentheses for implicit `div` when using `.` or `#`.
- Buffered code (`=`) is HTML-escaped by default.
- Unescaped output (`!=` or `!{}`) must only be used with sanitized content.

#### Constraints and Limitations

- Pug's conciseness can obscure structure for developers new to indentation-based syntax.
- Large templates without includes/mixins become hard to maintain.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Control Flow and Code

```pug
- const users = ['Alice', 'Bob', 'Charlie']
ul
  each user in users
    li= user

if users.length > 0
  p There are #{users.length} users.
else
  p No users found.
```

**Expected Output**

```html
<ul>
  <li>Alice</li>
  <li>Bob</li>
  <li>Charlie</li>
</ul>
<p>There are 3 users.</p>
```

**Why This Output Occurs**

- `each` loops over the array; `=` emits each value; `#{}` interpolates the length inside a string.

---

### Real-World Cases

- **Data-driven lists** — Rendering tables and lists from database results.
- **Conditional UI** — Showing/hiding navigation based on authentication state.
- **Layout composition** — Building page structure with nested tags.

---

## Core Concept 3: Indentation

### Definitions

**Core Definition**
Indentation in Pug is the use of leading whitespace to define the parent-child nesting relationship between tags.

**Technical Definition**
Pug is whitespace-sensitive: the level of indentation determines nesting. The exact number of spaces or tabs does not matter, as long as the level of indentation remains consistent throughout the template. Mixing tabs and spaces or inconsistent indentation levels causes compilation errors. Nesting is expressed by indenting a line beneath its parent; dedenting ends the nesting.

**Beginner-Friendly Explanation**
Indentation is how Pug knows which tags are inside which. If you indent a line under another, it becomes a child. If you don't, it becomes a sibling. As long as you are consistent, Pug does not care whether you use two spaces or four.

---

### Purposes

- To express HTML nesting without closing tags.
- To make template structure visually obvious.
- To reduce markup noise and improve readability.

---

### Syntax Rules and Structure

#### Complete General Syntax

```pug
ul
  li Item 1
  li Item 2
p Paragraph
```

**Component Breakdown**

| Indentation | Effect |
|-------------|--------|
| `ul` at level 0 | Parent list. |
| `li` at level 1 | Children of `ul`. |
| `p` at level 0 | Sibling of `ul`. |

#### Syntax Rules

- Use consistent indentation (spaces or tabs, not both).
- Nesting is determined solely by indentation level.
- Dedenting closes the previous nesting level.

#### Constraints and Limitations

- Inconsistent indentation produces `Pug: unexpected token` errors.
- Editors that auto-format or normalize whitespace can silently break Pug templates.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Nested Structure

```pug
div.container
  h1 Title
  p Description
  ul
    li Item A
    li Item B
```

**Expected Output**

```html
<div class="container">
  <h1>Title</h1>
  <p>Description</p>
  <ul>
    <li>Item A</li>
    <li>Item B</li>
  </ul>
</div>
```

**Why This Output Occurs**

- Each indented line becomes a child of the previous less-indented line.

---

### Real-World Cases

- **Any Pug template** — Indentation is foundational to every Pug file.
- **Layouts and partials** — Deep nesting reflects page hierarchy.

---

## Core Concept 4: Tags

### Definitions

**Core Definition**
Tags in Pug are bare words at the start of a line that declare HTML elements, with implicit `div` shorthand and shorthand class/ID syntax.

**Technical Definition**
A tag declaration consists of a tag name, optional class and ID shorthand, and optional attributes in parentheses. If no tag name is provided but a `.` or `#` shorthand is used, the tag defaults to `div`. Classes use `.classname` and IDs use `#idname`. Text content follows the tag on the same line or on indented lines below.

**Beginner-Friendly Explanation**
Tags are the HTML elements you write. In Pug you just type `h1`, `p`, `div`, and so on. You can add classes with `.` and IDs with `#` right next to the tag name.

---

### Purposes

- To declare HTML elements concisely.
- To attach classes and IDs without repetitive `class=` and `id=` attributes.
- To structure document hierarchy.

---

### Syntax Rules and Structure

#### Complete General Syntax

```pug
h1 Heading
p Text
div.card#main Content
a(href="/home") Home
```

**Component Breakdown**

| Syntax | Output |
|--------|--------|
| `h1 Heading` | `<h1>Heading</h1>` |
| `.card` | `<div class="card"></div>` |
| `#main` | `<div id="main"></div>` |
| `div.card#main` | `<div class="card" id="main"></div>` |
| `a(href="/home")` | `<a href="/home"></a>` |

#### Syntax Rules

- Omit the tag name to default to `div` when using `.` or `#`.
- Multiple classes are chained: `div.card.big-shadow`.
- Tag text can be inline or indented (block text).

#### Constraints and Limitations

- Self-closing tags (e.g., `img`, `input`) do not need a trailing slash; Pug handles void elements automatically.
- Deeply chained shorthand can become hard to read.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Shorthand and Attributes

```pug
.card.shadow#product-1
  h2 Product Name
  p Price: $19.99
  a(href="/product/1") View details
```

**Expected Output**

```html
<div class="card shadow" id="product-1">
  <h2>Product Name</h2>
  <p>Price: $19.99</p>
  <a href="/product/1">View details</a>
</div>
```

**Why This Output Occurs**

- `.card.shadow` and `#product-1` create the `div` with two classes and an ID.

---

### Real-World Cases

- **UI components** — Cards, buttons, and navigation items.
- **Semantic structure** — Headings, sections, and lists.
- **Forms** — Inputs and labels with classes for styling.

---

## Core Concept 5: Attributes

### Definitions

**Core Definition**
Attributes in Pug are specified in parentheses after a tag, using JavaScript expressions for dynamic values, with shorthand syntax for classes and IDs.

**Technical Definition**
Attributes are written as `tag(attr="value" attr2=expression)`. Values are JavaScript expressions, enabling dynamic attributes such as `body(class=authenticated ? 'authed' : 'anon')`. Classes and IDs have dedicated shorthand (`.class` and `#id`). The `&attributes()` syntax can spread an object into the attribute list, but it carries security risks if the object is user-controlled.

**Beginner-Friendly Explanation**
Attributes go inside parentheses right after the tag. You can use plain values like `"text"` or JavaScript expressions like `isActive ? 'on' : 'off'`. Classes and IDs have their own shortcuts so you don't have to write `class=` every time.

---

### Purposes

- To set HTML attributes with static or dynamic values.
- To conditionally apply classes or attributes based on data.
- To keep attribute syntax compact and readable.

---

### Syntax Rules and Structure

#### Complete General Syntax

```pug
input(type="text" name="q" autofocus)
body(class=authenticated ? 'authed' : 'anon')
a(href=url) Link
```

**Component Breakdown**

| Syntax | Meaning |
|--------|---------|
| `type="text"` | Static attribute. |
| `class=expression` | Dynamic attribute value. |
| `autofocus` | Boolean attribute (presence = true). |

#### Syntax Rules

- Attribute values may be any JavaScript expression.
- Boolean attributes are set by presence; `disabled=false` omits the attribute.
- `&attributes(obj)` spreads an object into attributes; avoid with untrusted input.

#### Constraints and Limitations

- Using `&attributes` with user-controlled objects can introduce XSS or unexpected attributes.
- Attribute values in `href` can still contain `javascript:` URIs; HTML escaping alone does not prevent this.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Dynamic Class and Attributes

```pug
- const isLoggedIn = true
- const user = { name: 'Alice' }

div(class=isLoggedIn ? 'user' : 'guest')
  input(type="text" value=user.name disabled=!isLoggedIn)
  a(href="/logout") Logout
```

**Expected Output**

```html
<div class="user">
  <input type="text" value="Alice">
  <a href="/logout">Logout</a>
</div>
```

**Why This Output Occurs**

- `isLoggedIn` is true, so the class is `"user"` and `disabled` is `false`, which omits the attribute. `value` is dynamically set from `user.name`.

---

### Real-World Cases

- **Form inputs** — Dynamic `value`, `disabled`, and `checked` attributes.
- **Navigation states** — Active classes based on the current route.
- **Accessibility** — `aria-*` attributes driven by application state.

---

## Core Concept 6: Interpolation

### Definitions

**Core Definition**
Interpolation is the embedding of JavaScript values inside Pug text or attributes, using `#{}` for escaped output and `!{}` for unescaped output.

**Technical Definition**
Pug provides several interpolation operators. `#{}` performs **escaped string interpolation**: it evaluates the expression, converts it to a string, and HTML-escapes the result, making it suitable for untrusted input. `!{}` performs **unescaped interpolation**, inserting raw HTML or JavaScript values without escaping, which is dangerous with user input. `#[tag]` interpolates a Pug element inline within text. Buffered code (`=`) is also escaped; unescaped buffered code (`!=`) does not escape.

**Beginner-Friendly Explanation**
`#{}` is the safe way to put data into text: it escapes HTML so malicious scripts become harmless text. `!{}` puts raw content in, which is useful for trusted HTML but dangerous for anything the user controls. Use `#{}` by default; use `!{}` only for content you have sanitized.

---

### Purposes

- `#{}` — To safely embed dynamic values in text.
- `!{}` — To inject trusted HTML fragments without double-escaping.
- `#[tag]` — To embed inline Pug elements within text.

---

### Syntax Rules and Structure

#### Complete General Syntax

```pug
p Hello, #{name}!
p !{trustedHtml}
p This is #[strong important] text.
```

**Component Breakdown**

| Syntax | Output | Security |
|--------|--------|----------|
| `#{expr}` | HTML-escaped string | Safe for user input. |
| `!{expr}` | Unescaped output | XSS risk with untrusted input. |
| `#[tag]` | Inline Pug element | Safe (escaped as needed). |

#### Syntax Rules

- `#{}` is the default choice for user data.
- `!{}` must only be used for sanitized or trusted HTML.
- To output a literal `#{`, escape it as `\#{`.

#### Constraints and Limitations

- Escaping does not protect against `javascript:` URIs in `href` attributes.
- `!{}` in `<script>` contexts requires additional JSON-stringification safeguards.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Escaped vs. Unescaped Interpolation

```pug
- const userInput = '<script>alert("xss")</script>'
- const trustedHtml = '<strong>Safe content</strong>'

p Escaped: #{userInput}
p Unescaped: !{trustedHtml}
```

**Expected Output**

```html
<p>Escaped: &lt;script&gt;alert("xss")&lt;/script&gt;</p>
<p>Unescaped: <strong>Safe content</strong></p>
```

**Why This Output Occurs**

- `#{}` escapes the script tags into entities; `!{}` inserts the trusted HTML raw.

---

### Real-World Cases

- **User comments** — Always `#{comment}` to prevent XSS.
- **Rich text content** — Sanitize server-side, then `!{sanitizedHtml}`.
- **Inline formatting** — `#[strong bold]` within paragraphs.

---

## Core Concept 7: Includes

### Definitions

**Core Definition**
Includes allow one Pug template to insert the contents of another Pug file at a specified point, enabling reuse of partials such as headers and footers.

**Technical Definition**
The `include` keyword inserts another template's content at the current indentation level. Includes can target Pug files or apply filters (e.g., `include:markdown article.md`). Included files share the parent's data scope, but variables defined in the parent with `-` may need to be passed explicitly depending on scope. Includes are resolved relative to the including file.

**Beginner-Friendly Explanation**
An include pastes another file into your template. If every page needs the same navigation bar, you write it once in a separate file and include it everywhere.

---

### Purposes

- To reuse common template fragments across pages.
- To keep templates DRY and maintainable.
- To compose large pages from smaller, focused files.

---

### Syntax Rules and Structure

#### Complete General Syntax

```pug
include ./partials/header.pug
main Page content
include ./partials/footer.pug
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `include path` | Inserts the target file's content. |
| `include:filter path` | Applies a filter (e.g., Markdown) to the included file. |

#### Syntax Rules

- Paths are relative to the including template.
- Included content shares the parent template's local variables.
- Includes can be nested.

#### Constraints and Limitations

- Includes are textual; they do not provide block overriding (use `extends` for that).
- Deep include chains can complicate debugging.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Header and Footer Includes

```pug
//- partials/header.pug
header
  h1 My Site
  nav
    a(href="/") Home
    a(href="/about") About

//- partials/footer.pug
footer
  p Copyright 2026

//- page.pug
include ./partials/header.pug
main
  p Welcome to the page.
include ./partials/footer.pug
```

**Expected Output**

```html
<header>
  <h1>My Site</h1>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
<main>
  <p>Welcome to the page.</p>
</main>
<footer>
  <p>Copyright 2026</p>
</footer>
```

**Why This Output Occurs**

- The includes are replaced with the contents of the referenced files at the current indentation level.

---

### Real-World Cases

- **Site navigation** — A shared header/nav partial.
- **Analytics scripts** — A shared footer snippet.
- **Form components** — Reusable input groups.

---

## Core Concept 8: Extends

### Definitions

**Core Definition**
`extends` enables template inheritance: a child template extends a base layout and overrides named `block` sections, allowing shared page structure with per-page content.

**Technical Definition**
A base template defines `block` placeholders. A child template begins with `extends layout.pug` and then overrides blocks by name. Blocks can contain default content and can themselves contain nested blocks for deeper customization. `extends` is a single inheritance mechanism; Pug supports one parent per template.

**Beginner-Friendly Explanation**
`extends` lets you create a master layout with holes (blocks) that child pages fill in. Every page shares the same header and footer, but each page can replace the main content area with its own content.

---

### Purposes

- To share a common page shell (doctype, head, header, footer) across pages.
- To override specific sections per page without duplicating the layout.
- To support nested block structures for flexible customization.

---

### Syntax Rules and Structure

#### Complete General Syntax

```pug
//- layout.pug
doctype html
html
  head
    block title
      title Default Title
  body
    block content

//- page.pug
extends layout.pug

block title
  title Page Title

block content
  h1 Page Content
  p This replaces the parent block.
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `extends layout.pug` | Declares the parent template. |
| `block name` | Defines an overridable section. |
| Child `block name` | Replaces the parent block's content. |

#### Syntax Rules

- `extends` must be the first thing in the child template (except comments).
- Blocks are overridden by matching names.
- A child can add content around a block using `append` or `prepend` (where supported).

#### Constraints and Limitations

- Only single inheritance is supported.
- Deeply nested blocks can become difficult to track.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Layout and Child Template

```pug
//- layout.pug
doctype html
html
  head
    block title
      title My Site
  body
    header
      h1 My Site
    block content
    footer
      p Footer

//- home.pug
extends layout.pug

block title
  title Home Page

block content
  h2 Welcome Home
  p This is the home page.
```

**Expected Output**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <header>
      <h1>My Site</h1>
    </header>
    <h2>Welcome Home</h2>
    <p>This is the home page.</p>
    <footer>
      <p>Footer</p>
    </footer>
  </body>
</html>
```

**Why This Output Occurs**

- The child overrides `title` and `content`; the parent's header and footer remain.

---

### Real-World Cases

- **Multi-page websites** — A shared layout with per-page content blocks.
- **Dashboard layouts** — A sidebar layout with a content block.
- **Marketing pages** — Shared header/footer with page-specific hero sections.

---

## Core Concept 9: Mixins

### Definitions

**Core Definition**
Mixins are reusable Pug blocks, compiled into functions, that can accept arguments and an optional block of nested content, enabling component-based template design.

**Technical Definition**
A mixin is declared with `mixin name(args)` and invoked with `+name(args)`. Mixins can accept rest arguments (`...`) for variable-length argument lists. A mixin can also take a **block** of Pug content, rendered where the `block` keyword appears inside the mixin. Mixins are compiled into functions, so they can be reused across templates and included via `include`.

**Beginner-Friendly Explanation**
A mixin is a reusable piece of template code, like a function for HTML. You define it once with `mixin`, give it arguments, and call it with `+name`. If you want the caller to supply inner content, you use a `block` inside the mixin.

---

### Purposes

- To create reusable UI components without a frontend framework.
- To pass data into reusable blocks via arguments.
- To allow callers to supply nested content through mixin blocks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```pug
mixin button(text, href)
  a.btn(href=href)= text

+button('Click Me', '/submit')

mixin article(title)
  article
    h2= title
    block

+article('Hello')
  p Article content here.
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `mixin name(args)` | Declares the mixin. |
| `+name(args)` | Invokes the mixin. |
| `block` | Placeholder for caller-supplied content. |

#### Syntax Rules

- Mixins are invoked with `+`; arguments are passed in parentheses.
- `block` inside a mixin renders the content provided by the caller.
- Rest arguments (`...`) allow variable-length argument lists.

#### Constraints and Limitations

- Mixins support only a single unnamed block; multiple named blocks are not supported.
- Mixin definitions are not automatically available across files unless included.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Button Mixin with Block

```pug
mixin card(title)
  .card
    h3= title
    block

+card('Product 1')
  p Price: $19.99

+card('Product 2')
  p Price: $29.99
```

**Expected Output**

```html
<div class="card">
  <h3>Product 1</h3>
  <p>Price: $19.99</p>
</div>
<div class="card">
  <h3>Product 2</h3>
  <p>Price: $29.99</p>
</div>
```

**Why This Output Occurs**

- The mixin is called twice with different titles; the `block` renders the caller-supplied paragraph each time.

---

### Real-World Cases

- **UI component libraries** — Buttons, cards, alerts, and modals.
- **Form field groups** — Label + input + error message bundles.
- **Repeated page sections** — Testimonials, pricing tiers, and feature cards.

---

## Core Concept 10: Pug Best Practices

### Definitions

**Core Definition**
Pug best practices are the security, performance, and organizational patterns that keep Pug-based Express applications safe, fast, and maintainable.

**Technical Definition**
Best practices encompass: using escaped output (`=` and `#{}`) for all user-controlled data and reserving unescaped output (`!=` and `!{}`) for sanitized content; enabling view caching in production to avoid recompilation on every request; quoting attributes and avoiding `javascript:` URIs in `href` values; validating and sanitizing all user input in controllers; organizing templates into `layouts/`, `partials/`, and `pages/` directories; and keeping mixins focused and composable.

**Beginner-Friendly Explanation**
Best practices are the rules that keep Pug safe and fast. The biggest rule: always escape user data. The second: turn on caching in production so pages load quickly. The third: keep your templates organized so you can find things.

---

### Purposes

- To prevent XSS through escaped output and safe attribute handling.
- To improve rendering performance by enabling template caching.
- To keep templates organized and maintainable as the project grows.
- To separate data preparation from presentation.

---

### Syntax Rules and Structure

#### Complete General Syntax: Security and Performance Checklist

```pug
//- Escaped output (safe for user input)
p= userInput
p Hello, #{userInput}

//- Unescaped output (trusted only)
p!= sanitizedHtml

//- Quoted attribute
a(href="/user/" + userId) Profile
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Escape user data | `=` and `#{}`. |
| Unescape only trusted | `!=` and `!{}` for sanitized content. |
| Quote attributes | Always quote dynamic attribute values. |
| Cache templates | `app.enable('view cache')` in production. |

#### Syntax Rules

- **Never** use `!=` or `!{}` with user-controlled input.
- **Never** interpolate user input into `href` without validating the scheme (avoid `javascript:`).
- Enable template caching in production.
- Organize templates into `layouts/`, `partials/`, and `pages/`.

#### Constraints and Limitations

- Caching caches the compiled template, not the rendered output; dynamic data is still evaluated per request.
- Mixins with blocks support only one unnamed block.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Secure and Cached Pug Setup

```javascript
// app.js
app.set('view engine', 'pug');
app.set('views', path.join(__dirname, 'views'));
app.enable('view cache'); // production

app.get('/profile/:id', (req, res) => {
  const userId = Number(req.params.id);
  if (!Number.isInteger(userId)) {
    return res.status(400).send('Invalid ID');
  }
  res.render('profile', {
    title: 'Profile',
    userId,
    bio: sanitize(user.bio) // sanitized before template
  });
});
```

```pug
//- views/profile.pug
extends layouts/main

block content
  h1 User #{userId}
  p= bio
```

**Expected Output**

```html
<h1>User 42</h1>
<p>Sanitized bio text</p>
```

**Why This Output Occurs**

- `#{userId}` and `= bio` are escaped; the template is cached in production; validation and sanitization happen in the controller.

---

### Real-World Cases

- **Production Express apps** — Caching enabled, escaped output, sanitized rich text.
- **Security audits** — Verifying no `!=` touches user input and no `javascript:` URIs reach `href`.
- **Team onboarding** — Consistent template organization and mixin usage.

---

## References

- Pug Official Documentation — https://pugjs.org/
- Pug GitHub Repository — https://github.com/pugjs/pug
- Pug Syntax (official) — https://pugjs.org/language/
- Pug Attributes — https://pugjs.org/language/attributes.html
- Pug Interpolation — https://pugjs.org/language/interpolation.html
- Pug Includes — https://pugjs.org/language/includes.html
- Pug Template Inheritance — https://pugjs.org/language/inheritance.html
- Pug Mixins — https://pugjs.org/language/mixins.html
- Express.js Using Template Engines — https://expressjs.com/en/guide/using-template-engines.html
- Prevent XSS in ExpressJS (Semgrep) — https://docs.semgrep.dev/cheat-sheets/express-xss
- Express.js Insecure Template Engine Usage (Sourcery) — https://www.sourcery.ai
- Pug Best Practices (Zaira Labs) — https://zairalabs.ai
- Pug Cheatsheet (devhints) — https://devhints.io/pug
- Pug Security Policy (GitHub) — https://github.com/pugjs/pug/security