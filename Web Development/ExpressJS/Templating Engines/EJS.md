# EJS (Embedded JavaScript Templates) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
EJS (Embedded JavaScript) is a simple templating language that lets you generate HTML markup with plain JavaScript, using special tags embedded directly inside HTML files.

**Technical Definition**
EJS is a zero-dependency template engine that compiles templates into JavaScript functions. It complies with the Express view system and works out of the box once the view engine is set. Unlike template languages that invent their own syntax for loops and conditionals, EJS uses plain JavaScript inside scriptlet tags (`<% %>`), meaning there are no new keywords or helper APIs to learn. The engine caches compiled template functions for speed and supports sub-template includes, custom delimiters, and both server-side and client-side rendering.

**Beginner-Friendly Explanation**
EJS lets you write normal HTML and sprinkle JavaScript inside it using special tags. If you already know JavaScript, you already know EJS. You write `<h1><%= title %></h1>` and EJS replaces `<%= title %>` with the actual title when the page is rendered. There is no new language to learn — just JavaScript inside HTML.

---

### Key Characteristics

- **Plain JavaScript, no new syntax** — Loops, conditionals, and variables use the JavaScript you already know; no custom helpers or directives are required.
- **Zero dependencies** — EJS has no external dependencies, making it lightweight and easy to audit.
- **Express-compatible out of the box** — EJS complies with Express's view system; setting `app.set('view engine', 'ejs')` is sufficient to begin.
- **Multiple tag types** — Distinct tags exist for escaped output, unescaped output, control flow, comments, and includes.
- **Caching and CLI support** — Compiled templates can be cached for performance, and EJS includes a command-line interface for rendering templates to files.
- **Security-sensitive by design** — EJS is effectively a JavaScript runtime; giving end-users unfettered access to the render method is explicitly described by the maintainers as inherently insecure.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- The `ejs` package installed via npm (`npm install ejs`).
- Basic familiarity with HTML and JavaScript (variables, conditionals, loops, functions).
- Understanding of Express routing and `res.render()`.

---

### Related Programming Areas

- **Templating** — EJS is one of several Express view engines alongside Pug, Handlebars, and Nunjucks.
- **Server-side rendering (SSR)** — EJS renders full HTML on the server for traditional web applications.
- **Security (XSS prevention)** — EJS's escaping behavior is a primary defense against cross-site scripting.
- **Performance optimization** — Template caching and compilation reuse directly affect rendering speed.
- **Email generation** — EJS can render templates to strings for HTML email bodies via `app.render()`.

---

## Core Concept 1: EJS

### Definitions

**Core Definition**
EJS is a template engine that embeds JavaScript within HTML to produce dynamic markup, using the existing JavaScript language rather than introducing a new templating dialect.

**Technical Definition**
EJS compiles template strings or files into JavaScript functions that accept a data object (locals) and return an HTML string. It can be used standalone via `ejs.render()` or `ejs.compile()`, as an Express view engine via `app.set('view engine', 'ejs')`, or from the command line via `ejs ./template.ejs -f data.json`. As of EJS v7, the engine does **not** read its options from Express's render data, `app.locals`, or `app.set('view options')`; to set EJS options for an Express app, a render function must be registered. EJS is explicitly described by its maintainers as effectively a JavaScript runtime, meaning the security of rendered output depends on the application's input handling.

**Beginner-Friendly Explanation**
EJS is a tool that takes an HTML file with JavaScript placeholders and fills them in with real data. It works with Express, and because it just uses JavaScript, you don't have to learn anything new. You can use it on the server or even in the browser.

---

### Purposes

- To generate dynamic HTML using plain JavaScript without learning a new template syntax.
- To integrate seamlessly with Express's view rendering system.
- To separate HTML structure from application data while keeping logic expressed in JavaScript.
- To enable reusable components through includes and partials.

---

### Syntax Rules and Structure

#### Complete General Syntax: Standalone and Express Usage

```javascript
// Standalone rendering
const ejs = require('ejs');
const html = ejs.render('<%= name %>', { name: 'Alice' });

// Express view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));
app.get('/', (req, res) => res.render('index', { title: 'Home' }));
```

**Component Breakdown**

| Usage | API | Role |
|-------|-----|------|
| Standalone string | `ejs.render(str, data, options)` | Renders a template string directly |
| Compile once, render many | `ejs.compile(str, options)` | Returns a reusable template function |
| Express | `app.set('view engine', 'ejs')` | Registers EJS as the default view engine |
| CLI | `ejs ./template.ejs -f data.json` | Renders a template file to an output file |

#### Syntax Rules

- Install EJS via npm and either set it as the Express view engine or require it directly.
- As of EJS v7, Express-level EJS options require a registered render function, not `app.set('view options')`.
- Templates use `.ejs` extension by convention when used with Express.

#### Constraints and Limitations

- EJS is **not** a sandbox; it is a JavaScript runtime, and unfettered user access to the render method is inherently insecure.
- In Express, EJS options (delimiters, caching, etc.) are not automatically inherited from Express settings in v7+; explicit configuration is required.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Express Application with EJS

```javascript
// app.js
const express = require('express');
const path = require('path');
const app = express();

app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.get('/', (req, res) => {
  res.render('index', {
    title: 'EJS Demo',
    users: ['Alice', 'Bob', 'Charlie']
  });
});

app.listen(3000);
```

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head><title><%= title %></title></head>
<body>
  <h1><%= title %></h1>
  <ul>
    <% users.forEach(function(user) { %>
      <li><%= user %></li>
    <% }); %>
  </ul>
</body>
</html>
```

**Step-by-Step Explanation**

1. **Engine registration** — `app.set('view engine', 'ejs')` tells Express to use EJS and load the module internally.
2. **Directory configuration** — `app.set('views', ...)` points to the folder containing `index.ejs`.
3. **Rendering** — `res.render('index', { title, users })` passes locals to the template.
4. **Template execution** — EJS runs the embedded JavaScript (`forEach`) and substitutes escaped values.

**Expected Output**

```html
<!DOCTYPE html>
<html>
<head><title>EJS Demo</title></head>
<body>
  <h1>EJS Demo</h1>
  <ul>
      <li>Alice</li>
      <li>Bob</li>
      <li>Charlie</li>
  </ul>
</body>
</html>
```

**Why This Output Occurs**

- The `<% %>` scriptlet executes the `forEach` loop without producing output; each `<%= user %>` emits an escaped `<li>` for every user.

---

### Real-World Cases

- **Server-rendered web applications** — Traditional multi-page apps where each route renders a full HTML page.
- **Email templates** — Using `app.render()` to generate HTML email bodies without sending an HTTP response.
- **Prototyping** — Quickly building dynamic pages without adopting a heavier template language.

---

## Core Concept 2: EJS Syntax

### Definitions

**Core Definition**
EJS syntax is the set of tag delimiters and conventions used to embed JavaScript logic and output expressions within HTML templates.

**Technical Definition**
An EJS "tag" is the primary functioning unit in a template. With the exception of literal tags, all tags follow the format `<starting content closing>`, where the starting and closing delimiters are `%` by default (customizable). The starting tag determines the tag's behavior: `<%` for scriptlets (control flow, no output), `<%=` for escaped output, `<%-` for unescaped output, and `<%#` for comments. Ending tags include `%>` (regular), `-%>` (removes the trailing newline, "newline slurping"), and `_%>` (removes all trailing whitespace). Delimiters can be customized per template or globally.

**Beginner-Friendly Explanation**
EJS syntax is just special tags that tell the template engine what to do. `<% %>` runs JavaScript without printing anything. `<%= %>` prints a value and makes it safe. `<%- %>` prints raw HTML. `<%# %>` is a comment. You can also customize the delimiters if you don't like `%`.

---

### Purposes

- To embed JavaScript control flow (loops, conditionals) directly in HTML.
- To output dynamic values with automatic HTML escaping for safety.
- To output trusted pre-formatted HTML without escaping.
- To organize templates with comments and whitespace control.

---

### Syntax Rules and Structure

#### Complete General Syntax: Tag Overview

```html
<% code %>          <!-- Scriptlet: control flow, no output -->
<%= value %>        <!-- Escaped output (HTML escaped) -->
<%- value %>        <!-- Unescaped output (raw HTML) -->
<%# comment %>      <!-- Comment, no execution, no output -->
<%%                 <!-- Outputs a literal "<%" -->
```

**Component Breakdown**

| Tag | Behavior | Use Case |
|-----|----------|----------|
| `<% %>` | Runs JavaScript; produces **no output** | Loops, conditionals, variable declarations |
| `<%= %>` | Outputs the value **HTML-escaped** | User data, dynamic text |
| `<%- %>` | Outputs the value **unescaped (raw)** | Trusted pre-rendered HTML fragments |
| `<%# %>` | Comment; no execution, no output | Annotating templates |
| `<%%` | Outputs a literal `<%` | Escaping EJS delimiters themselves |

#### Syntax Rules

- The content of a tag can be any valid JavaScript expression; tags like `<%= name ? name : (lastName || 'John Doe') %>` work as expected.
- The default delimiter is `%`; it can be customized per template or globally.
- Ending tags `-%>` and `_%>` control newline and whitespace trimming.

#### Constraints and Limitations

- `<%- %>` with untrusted user input creates an XSS vulnerability; it must only be used for sanitized, trusted HTML.
- Custom delimiters can reduce readability for developers unfamiliar with the template.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Control Flow and Output

```html
<!-- views/demo.ejs -->
<% if (user) { %>
  <h2>Hello, <%= user.name %>!</h2>
  <% if (user.isAdmin) { %>
    <span class="badge">Admin</span>
  <% } %>
<% } else { %>
  <h2>Hello, Guest!</h2>
<% } %>

<% items.forEach(function(item) { %>
  <li><%= item %></li>
<% }); %>
```

**Expected Output (user = { name: 'Alice', isAdmin: true }, items = ['A', 'B'])**

```html
  <h2>Hello, Alice!</h2>
    <span class="badge">Admin</span>

  <li>A</li>
  <li>B</li>
```

**Why This Output Occurs**

- The `<% if %>` and `<% forEach %>` tags execute control flow without emitting output; `<%= %>` emits the escaped values.

---

### Real-World Cases

- **Conditional UI** — Showing admin controls only to administrators.
- **List rendering** — Looping over database results to build tables or lists.
- **Layout composition** — Using includes (see Core Concept 4) to assemble pages from fragments.

---

## Core Concept 3: Output Tags — `<%= %>`, `<%- %>`, `<% %>`

### Definitions

**Core Definition**
The three primary EJS output tags serve distinct purposes: `<%= %>` escapes and outputs a value, `<%- %>` outputs a value without escaping, and `<% %>` executes JavaScript without producing output.

**Technical Definition**
`<%= %>` is the escaped-output tag; forbidden characters such as `<` and `&` are automatically replaced with HTML entities, mitigating cross-site scripting. `<%- %>` is the unescaped-output tag; it inserts the raw string into the HTML, which is appropriate only for trusted, pre-sanitized HTML fragments. `<% %>` is the scriptlet tag; it executes JavaScript statements (conditionals, loops, variable assignments) and produces no direct output. EJS's default escaping function is configurable, but the default is the safe choice for user data.

**Beginner-Friendly Explanation**
`<%= %>` is your safe default: it prints a value and automatically makes it safe to display. `<%- %>` prints raw HTML — use it only when you are certain the content is safe, like a pre-built HTML snippet. `<% %>` runs code but does not print anything, so you use it for loops and if-statements.

---

### Purposes

- `<%= %>` — To safely display user-supplied and dynamic data in HTML.
- `<%- %>` — To inject trusted, pre-rendered HTML (partials, sanitized rich text) without double-escaping.
- `<% %>` — To execute control flow and logic without emitting output.

---

### Syntax Rules and Structure

#### Complete General Syntax: Escaped vs. Unescaped vs. Scriptlet

```html
<p>Hello, <%= name %>.</p>          <!-- escaped: safe for user data -->
<p>Hello, <%- myHtml %>.</p>        <!-- unescaped: trusted HTML only -->
<% if (showGreeting) { %>            <!-- scriptlet: no output -->
  <p>Welcome!</p>
<% } %>
```

**Component Breakdown**

| Tag | Escaping | Security Implication |
|-----|----------|----------------------|
| `<%= %>` | HTML-escaped | Safe for untrusted input |
| `<%- %>` | **None** | XSS risk if input is untrusted |
| `<% %>` | N/A (no output) | No injection surface from the tag itself |

#### Syntax Rules

- **Always** use `<%= %>` for user-controlled data, including names, emails, comments, and search terms.
- **Only** use `<%- %>` for HTML that has been sanitized or produced entirely by trusted server code.
- `<% %>` statements must be syntactically valid JavaScript; unbalanced braces cause template compilation errors.

#### Constraints and Limitations

- HTML escaping does **not** protect against unquoted HTML attributes; variables rendered inside attributes must always be quoted.
- `<%- %>` with malicious input produces script execution; the EJS documentation explicitly warns to be “100% sure” the rendered local is sanitized.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Escaped vs. Unescaped Output

```html
<!-- views/security.ejs -->
<p>Escaped: <%= myHtml %></p>
<p>Unescaped: <%- myHtml %></p>

<p>Escaped malicious: <%= myMaliciousHtml %></p>
<p>Unescaped malicious: <%- myMaliciousHtml %></p>
```

```javascript
// locals
{
  myHtml: '<strong>Timothy</strong>',
  myMaliciousHtml: '</p><script>document.write()</script><p>'
}
```

**Expected Output**

```html
<p>Escaped: &lt;strong&gt;Timothy&lt;/strong&gt;</p>
<p>Unescaped: <strong>Timothy</strong></p>

<p>Escaped malicious: &lt;/p&gt;&lt;script&gt;document.write()&lt;/script&gt;&lt;p&gt;</p>
<p>Unescaped malicious: </p><script>document.write()</script><p></p>
```

**Why This Output Occurs**

- `<%= %>` converts `<` and `>` to `&lt;` and `&gt;`, neutralizing the script tags.
- `<%- %>` inserts the raw string, allowing the malicious script to execute — a demonstration of the XSS risk.

---

### Real-World Cases

- **Comment systems** — Rendering user comments with `<%= %>` to prevent injected scripts.
- **Rich text editors** — Sanitizing user-generated HTML server-side, then rendering it with `<%- %>`.
- **Layout partials** — Including header/footer fragments with `<%- include(...) %>` to avoid double-escaping.

---

## Core Concept 4: Includes

### Definitions

**Core Definition**
Includes allow one EJS template to embed the rendered output of another template file, enabling reusable fragments such as headers, footers, and navigation bars.

**Technical Definition**
EJS provides an `include()` function that renders another template and returns its HTML string. The modern syntax is `<%- include('path', { locals }) %>`, which uses the unescaped-output tag to avoid double-escaping the included HTML. Includes are resolved relative to the including template, which requires the `filename` option to be set (Express sets this automatically). Variables from the top-level data object are available to all includes, but local variables must be explicitly passed down. The legacy preprocessor syntax `<% include user/show %>` is still supported but the function form is recommended.

**Beginner-Friendly Explanation**
An include lets you paste one template file inside another. If every page needs the same header and footer, you write them once as separate files and include them on each page. This keeps your templates DRY (Don't Repeat Yourself).

---

### Purposes

- To reuse common template fragments (headers, footers, navigation) across pages.
- To reduce duplication and keep templates maintainable.
- To pass page-specific data into reusable fragments via locals.
- To compose larger pages from smaller, focused template files.

---

### Syntax Rules and Structure

#### Complete General Syntax: Modern Include

```html
<%- include('partials/header', { title: 'Home' }) %>
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `<%-` | Unescaped output tag to avoid double-escaping the included HTML |
| `include('path', locals)` | Renders the specified template with optional locals |
| `'partials/header'` | Path relative to the including template |

#### Syntax Rules

- Use `<%- include(...) %>` (not `<%= include(...) %>`) to prevent the included HTML from being escaped.
- Paths are relative to the template that calls `include()`; the `filename` option must be set.
- Locals from the parent template's data object are visible inside includes, but variables created with `<% const x = ... %>` must be passed explicitly.

#### Constraints and Limitations

- Includes are rendered at runtime; extremely deep or recursive includes can impact performance.
- The legacy `<% include user/show %>` syntax is supported but discouraged in favor of the function form.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Header and Footer Includes

```html
<!-- views/partials/header.ejs -->
<header>
  <h1><%= title %></h1>
  <nav><a href="/">Home</a> | <a href="/about">About</a></nav>
</header>
```

```html
<!-- views/partials/footer.ejs -->
<footer>
  <p>&copy; <%= new Date().getFullYear() %> My Site</p>
</footer>
```

```html
<!-- views/page.ejs -->
<%- include('partials/header', { title: 'Welcome' }) %>
<main>
  <p>Page content goes here.</p>
</main>
<%- include('partials/footer') %>
```

**Expected Output**

```html
<header>
  <h1>Welcome</h1>
  <nav><a href="/">Home</a> | <a href="/about">About</a></nav>
</header>
<main>
  <p>Page content goes here.</p>
</main>
<footer>
  <p>&copy; 2026 My Site</p>
</footer>
```

**Why This Output Occurs**

- The header include receives the `title` local; the footer include uses no locals. The `<%-` tags insert the rendered HTML without escaping.

---

### Real-World Cases

- **Site-wide navigation** — A single header/nav partial included on every page.
- **Reusable form components** — Input field partials included in multiple forms.
- **Email layouts** — Shared header/footer fragments across transactional emails.

---

## Core Concept 5: Layouts

### Definitions

**Core Definition**
Layouts provide a shared outer shell (typically containing the `<html>`, `<head>`, and `<body>` structure) that wraps multiple page templates, with a placeholder where page-specific content is inserted.

**Technical Definition**
EJS does **not** have built-in layout support. The standard solution is the `express-ejs-layouts` package, which adds layout, partial, and block template functions for EJS in Express. The layout defines a `<%- body %>` region where the rendered page content is injected. Layouts reduce boilerplate by centralizing the HTML skeleton, while each page template contains only its unique content.

**Beginner-Friendly Explanation**
A layout is a master template that wraps every page. Instead of copying the same `<html>`, `<head>`, and `<body>` tags into every file, you write them once in a layout, and each page only provides its unique content. `express-ejs-layouts` makes this work in EJS.

---

### Purposes

- To eliminate repetitive HTML skeleton code across pages.
- To centralize `<head>` metadata, CSS links, and script includes.
- To enforce a consistent page structure across an application.
- To simplify maintenance when the global structure changes.

---

### Syntax Rules and Structure

#### Complete General Syntax: `express-ejs-layouts`

```javascript
// app.js
const expressLayouts = require('express-ejs-layouts');
app.use(expressLayouts);
app.set('layout', 'layouts/main'); // default layout
```

```html
<!-- views/layouts/main.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <%- include('../partials/header') %>
  <%- body %>  <!-- page content is injected here -->
  <%- include('../partials/footer') %>
</body>
</html>
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `expressLayouts` middleware | Enables layout support in Express |
| `<%- body %>` | Placeholder for the rendered page content |
| `layout` setting | Default layout file path |

#### Syntax Rules

- Install `express-ejs-layouts` and register it with `app.use(expressLayouts)`.
- The layout must contain a `<%- body %>` placeholder.
- Individual pages can override the default layout by setting `layout: false` or a different layout in their locals.

#### Constraints and Limitations

- Layouts are not native to EJS; the package adds this functionality.
- Layout variables (like `title`) must be provided by the route or page template.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Layout with Header and Footer Partials

```javascript
// app.js
const express = require('express');
const expressLayouts = require('express-ejs-layouts');
const app = express();

app.set('view engine', 'ejs');
app.use(expressLayouts);
app.set('layout', 'layouts/main');

app.get('/', (req, res) => {
  res.render('pages/home', { title: 'Home', bodyClass: 'home' });
});

app.listen(3000);
```

```html
<!-- views/pages/home.ejs -->
<h1>Welcome Home</h1>
<p>This is the home page content.</p>
```

**Expected Output**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Home</title>
</head>
<body>
  <header>...</header>  <!-- from header partial -->
  <h1>Welcome Home</h1>
  <p>This is the home page content.</p>
  <footer>...</footer>  <!-- from footer partial -->
</body>
</html>
```

**Why This Output Occurs**

- The page content (`home.ejs`) is rendered and injected into `<%- body %>` within the layout; the layout’s partials surround it.

---

### Real-World Cases

- **Multi-page websites** — Every page shares the same header, footer, and navigation.
- **Dashboard applications** — A layout provides the sidebar and top bar; each view supplies the main content.
- **Marketing sites** — Layout centralizes analytics scripts and meta tags.

---

## Core Concept 6: Partials

### Definitions

**Core Definition**
Partials are reusable template fragments that represent a piece of a page—such as a header, footer, card, or form field group—and are included within layouts or other templates.

**Technical Definition**
Partials in EJS are plain `.ejs` files stored in a dedicated directory (conventionally `partials/`) and included via `<%- include('partials/name', locals) %>`. They differ from layouts in scope: a layout wraps an entire page, while a partial represents a smaller, repeatable component. Partials can receive their own locals and can themselves include other partials, enabling hierarchical composition. The EJS documentation recommends using the raw output tag (`<%-`) with includes to avoid double-escaping.

**Beginner-Friendly Explanation**
A partial is a small, reusable piece of a page. Think of it like a LEGO brick: a navigation bar, a product card, or a form field. You build these pieces once and snap them into pages wherever you need them.

---

### Purposes

- To reuse small UI components across multiple pages.
- To keep templates focused and readable by breaking them into manageable pieces.
- To pass component-specific data to reusable fragments.
- To enable hierarchical composition of complex pages.

---

### Syntax Rules and Structure

#### Complete General Syntax: Partial with Locals

```html
<%- include('partials/product-card', { product: item }) %>
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `include('partials/product-card', { product: item })` | Renders the partial with the given local |
| `item` | Data from the parent template’s scope |

#### Syntax Rules

- Store partials in a dedicated directory (e.g., `views/partials/`).
- Pass required locals explicitly; do not rely on parent-scope variables being automatically available inside the partial.
- Use `<%-` for the include tag to avoid escaping the partial’s HTML.

#### Constraints and Limitations

- Partials cannot define layout-level structure; use layouts for page shells.
- Deeply nested partials can make debugging harder; keep the hierarchy shallow.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Product Card Partial

```html
<!-- views/partials/product-card.ejs -->
<div class="product-card">
  <h3><%= product.name %></h3>
  <p><%= product.price %></p>
</div>
```

```html
<!-- views/products.ejs -->
<h1>Products</h1>
<div class="product-grid">
  <% products.forEach(function(product) { %>
    <%- include('partials/product-card', { product: product }) %>
  <% }); %>
</div>
```

**Expected Output**

```html
<h1>Products</h1>
<div class="product-grid">
    <div class="product-card">
      <h3>Widget</h3>
      <p>9.99</p>
    </div>
    <div class="product-card">
      <h3>Gadget</h3>
      <p>19.99</p>
    </div>
</div>
```

**Why This Output Occurs**

- The loop in `products.ejs` calls the partial once per product, passing each `product` object as a local.

---

### Real-World Cases

- **E-commerce product grids** — A product card partial reused for every item.
- **Form fields** — Input partials with label, error, and help text.
- **Comment lists** — A comment partial rendered for each comment.

---

## Core Concept 7: EJS Best Practices

### Definitions

**Core Definition**
EJS best practices are the security, performance, and organizational patterns that keep EJS-based Express applications safe, fast, and maintainable.

**Technical Definition**
Best practices encompass: always using `<%= %>` for user-controlled data and reserving `<%- %>` for sanitized or trusted HTML; quoting all HTML attributes that contain dynamic values to prevent attribute-based XSS; enabling template caching in production while disabling it in development; organizing templates into `layouts/`, `partials/`, and `pages/` directories; keeping complex logic in route handlers rather than templates; and never passing unsanitized user input as the template name or as top-level locals keys.

**Beginner-Friendly Explanation**
Best practices are the rules of thumb that keep EJS from becoming a security hole or a performance bottleneck. The most important rule: always escape user data with `<%= %>`. The second: turn on caching in production so your pages render fast. The third: organize your template files so you can find things.

---

### Purposes

- To prevent XSS vulnerabilities through correct escaping and attribute quoting.
- To improve rendering performance through compiled-template caching.
- To keep templates organized and maintainable as the application grows.
- To separate logic from presentation by preparing data in controllers.

---

### Syntax Rules and Structure

#### Complete General Syntax: Security and Performance Checklist

```javascript
// Enable view caching in production
app.set('view cache', process.env.NODE_ENV === 'production');
```

```html
<!-- Safe: escaped output -->
<p><%= userComment %></p>

<!-- Safe: quoted attribute -->
<a href="/user/<%= userId %>">Profile</a>

<!-- UNSAFE: unquoted attribute -->
<a href=/user/<%= userId %>>Profile</a>  <!-- XSS risk -->
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Escape user data | `<%= value %>` for all untrusted input |
| Quote attributes | Always wrap dynamic attribute values in quotes |
| Cache templates | `app.set('view cache', true)` in production |
| Organize templates | `layouts/`, `partials/`, `pages/` directories |
| Pre-process data | Compute values in route handlers, not in templates |

#### Syntax Rules

- **Never** pass user input directly to `res.render()` as the template name or as top-level locals keys.
- **Always** quote HTML attributes containing dynamic values; HTML escaping does not mitigate unquoted attribute injection.
- Enable caching in production and disable it in development for easier debugging.

#### Constraints and Limitations

- Caching caches the **compiled template**, not the rendered output; dynamic data is still evaluated on every request.
- Cache invalidation requires restarting the process or clearing the cache when templates change in non-development environments.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Secure Template with Caching

```javascript
// app.js
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));
app.set('view cache', process.env.NODE_ENV === 'production');

app.get('/profile/:id', (req, res) => {
  const userId = Number(req.params.id);
  // Validate in the controller, not the template
  if (!Number.isInteger(userId)) {
    return res.status(400).send('Invalid ID');
  }

  res.render('profile', {
    title: 'Profile',
    userId: userId,
    bio: '<strong>Trusted bio</strong>' // sanitized server-side
  });
});
```

```html
<!-- views/profile.ejs -->
<h1>User <%= userId %></h1>
<p><%= bio %></p>          <!-- escaped: safe even if bio were untrusted -->
<div><%- bio %></div>       <!-- unescaped: trusted HTML only -->
<a href="/user/<%= userId %>">Permalink</a>  <!-- quoted attribute -->
```

**Expected Output (userId = 42, bio = '<strong>Trusted bio</strong>')**

```html
<h1>User 42</h1>
<p>&lt;strong&gt;Trusted bio&lt;/strong&gt;</p>
<div><strong>Trusted bio</strong></div>
<a href="/user/42">Permalink</a>
```

**Why This Output Occurs**

- The escaped `<%= bio %>` renders HTML entities; the unescaped `<%- bio %>` renders the trusted HTML. The attribute is quoted for safety.

---

### Real-World Cases

- **Production web applications** — View caching enabled, escaped output, quoted attributes.
- **Security audits** — Verifying that no `<%- %>` tags touch user input and that all attributes are quoted.
- **Team onboarding** — Consistent template directory structure reduces confusion.

---

## References

- EJS Official Site — https://ejs.co/
- EJS Syntax Reference (official) — https://github.com/mde/ejs/blob/main/docs/syntax.md
- EJS Syntax Reference (v2.5.x) — https://raw.githubusercontent.com/mde/ejs/refs/tags/v2.5.1/docs/syntax.md
- Express.js Template Engines Guide — https://expressjs.com/en/guide/using-template-engines.html
- Express.js `res.render()` API — https://expressjs.com/en/5x/api.html#res.render
- Prevent XSS in ExpressJS (Semgrep) — https://docs.semgrep.dev/cheat-sheets/express-xss
- Express Template Best Practices (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/5-express-templating/13-express-template-best-practices.mdx
- Express Template Layouts (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/5-express-templating/7-express-template-layouts.mdx
- EJS Caching System (DeepWiki) — https://deepwiki.com/mde/ejs/5-caching-system
- `express-ejs-layouts` (npm) — https://www.npmjs.com/package/express-ejs-layouts
- EJS Security Policy (SECURITY.md) — https://github.com/mde/ejs/blob/main/SECURITY.md