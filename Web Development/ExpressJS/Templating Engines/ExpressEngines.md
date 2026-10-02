# Popular Express Template Engines — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A template engine is a library that enables an Express application to use static template files containing placeholders; at runtime, the engine replaces variables with actual values and transforms the template into an HTML file sent to the client.

**Technical Definition:** Express-compliant template engines export a function named `__express(filePath, options, callback)`, which `res.render()` calls to render the template code. After the view engine is set via `app.set('view engine', ...)`, Express loads the engine module internally and resolves template files from the directory specified by `app.set('views', ...)`. The engine compiles the template source into a JavaScript function, executes it with the merged data context, and produces an HTML string. Engines that do not follow the `__express` convention can be adapted via `@ladjs/consolidate`.

**Beginner-Friendly Explanation:** A template engine lets you write HTML with special placeholders — like `<%= name %>` or `#{name}` — that get replaced with real data when the page is rendered. Instead of building HTML strings in JavaScript, you write a template that looks like HTML and let the engine fill in the blanks. Express handles the plumbing: it finds the template file, loads the engine, and returns the finished HTML to the browser.

### Key Characteristics

- **Separation of concerns:** Presentation (templates) is separated from business logic (routes and controllers).
- **Server-side rendering:** HTML is generated on the server and sent fully rendered to the client.
- **Engine-agnostic:** Express supports any template engine that follows the `__express` convention.
- **Layout and partial support:** Most engines provide mechanisms for reusable fragments and structural inheritance.
- **Data binding:** Variables passed via `res.render()`, `res.locals`, or `app.locals` are interpolated into the template.
- **Caching:** The engine caches compiled template functions (not rendered output), so templates are re-rendered on every request.

### Prerequisites

- **Node.js runtime** (v18 or higher recommended).
- **Express.js installed** (`npm install express`).
- **A template engine package** (e.g., `npm install ejs`, `npm install pug`, `npm install express-handlebars`).
- **Basic JavaScript knowledge:** Functions, objects, and callbacks.
- **Understanding of Express routing:** How `app.get()` and `res.render()` work.

### Related Programming Areas

- **Server-Side Rendering (SSR):** Generating HTML on the server for faster initial page loads and SEO.
- **Middleware pipeline:** Locals set in middleware are available in templates.
- **Static file serving:** CSS, JavaScript, and images are served separately from templates.
- **API development:** Template engines can also render JSON, XML, or plain text.
- **Frontend frameworks:** Template engines are alternatives to React, Vue, and Angular for server-rendered pages.

### Core Concepts

1. **EJS (Embedded JavaScript)** — raw JavaScript execution, includes, performance vs. flexibility.
2. **Pug (formerly Jade)** — whitespace-sensitive syntax, mixins, attribute shortcuts.
3. **Handlebars / Mustache** — logic-less templates, custom block helpers, precompilation.
4. **Alternative Engines** — Nunjucks, Liquid, and raw HTML with minimal interpolation.

---

## Core Concept 1: EJS (Embedded JavaScript)

### Definitions

**Core Definition:** EJS is a simple templating language that lets you generate HTML markup with plain JavaScript, using scriptlet tags to embed logic directly in the template.

**Technical Definition:** EJS provides several tag types: `<% %>` for control flow (no output), `<%= %>` for escaped output, `<%- %>` for unescaped output, and `<%# %>` for comments. It caches intermediate JavaScript functions for fast execution and complies with the Express view system. EJS is effectively a JavaScript runtime; its entire job is to execute JavaScript.

**Beginner-Friendly Explanation:** EJS lets you write regular JavaScript inside your HTML. The `<% %>` tags contain logic that doesn't output anything; the `<%= %>` tags output values. If you know JavaScript, you already know EJS — there's no new syntax to learn.

### Purposes

- To generate dynamic HTML using plain JavaScript without a new template language.
- To leverage existing JavaScript knowledge for server-side rendering.
- To provide fast compilation and rendering through cached intermediate functions.
- To support both server-side and client-side rendering with the same templates.
- To enable template composition through partial includes.

### Sub-Feature 1.1: Raw JavaScript Execution and Include Partials

#### Syntax Rules and Structure

**Tag Types:**

| Tag | Purpose |
|-----|---------|
| `<% %>` | Control flow; no output. |
| `<%= %>` | Escaped output. |
| `<%- %>` | Unescaped output (for partials). |
| `<%# %>` | Comment. |

**Include Syntax:**

```ejs
<%- include('partials/header', { title: 'My Page' }) %>
```

| Component | Breakdown |
|-----------|-----------|
| `<%-` | Outputs unescaped HTML (partials should not be escaped). |
| `include('path')` | Includes the template at the given path. |
| `{ title: 'My Page' }` | Optional object passed to the partial. |

**Constraints and Limitations:**
- `<%-` is unescaped; use `<%=` for escaped output only.
- EJS does not have built-in layout inheritance; use `ejs-mate` or `express-ejs-layouts` for layouts.
- The include path is relative to the current template.
- If you give end-users unfettered access to the EJS render method, you are using EJS in an inherently unsecure way.

#### Annotated Code Example

```html
<!-- views/partials/header.ejs -->
<header>
  <h1><%= title %></h1>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
```

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
  <head><title><%= title %></title></head>
  <body>
    <%- include('partials/header', { title: 'Home' }) %>
    <main>
      <% if (message.includes('Welcome')) { %>
        <p>Thank you for visiting our site!</p>
      <% } %>
    </main>
  </body>
</html>
```

**Expected Output (for rendering `index` with `{ title: 'Home', message: 'Welcome!' }`):**
```html
<!DOCTYPE html>
<html>
  <head><title>Home</title></head>
  <body>
    <header>
      <h1>Home</h1>
      <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
      </nav>
    </header>
    <main>
      <p>Thank you for visiting our site!</p>
    </main>
  </body>
</html>
```

**Why this output:** The `include()` function reads `partials/header.ejs`, renders it with the passed `{ title: 'Home' }` object, and inserts the result. The `if` block conditionally renders the welcome message based on the `message` variable.

#### Real-World Cases

- **Blogs:** A `header.ejs` partial with navigation and a `footer.ejs` partial with copyright.
- **E-commerce:** A `product-card.ejs` partial rendered in a loop on the products page.
- **Admin panels:** A `sidebar.ejs` partial with navigation links, included on every admin page.

---

## Core Concept 2: Pug (formerly Jade)

### Definitions

**Core Definition:** Pug is a high-performance template engine heavily influenced by Haml that uses whitespace-sensitive syntax to write HTML templates with dramatically less code.

**Technical Definition:** Pug compiles its source code into a JavaScript function that takes a data object (called "locals") as an argument. Calling the resultant function returns a string of HTML rendered with the data. Pug supports template inheritance via `extends` and `block`, mixins for reusable code blocks, and implicit tag creation based on indentation.

**Beginner-Friendly Explanation:** Pug uses indentation instead of angle brackets. You write `h1= title` instead of `<h1><%= title %></h1>`. The result is cleaner, more readable templates that look more like a document outline than markup.

### Purposes

- To write HTML templates with minimal syntax and maximum readability.
- To leverage whitespace-sensitive indentation for structural clarity.
- To enable template inheritance through `extends` and `block`.
- To create reusable component blocks through mixins.
- To reduce the amount of code required compared to traditional HTML.

### Sub-Feature 2.1: Whitespace-Sensitive Syntax, Mixins, and Attribute Shortcuts

#### Syntax Rules and Structure

**Implicit Tag Creation:**

```pug
h1= title
p Welcome to #{title}
```

**Mixin Definition and Call:**

```pug
mixin pet(name)
  li.pet= name

ul
  +pet('cat')
  +pet('dog')
```

**Attribute Shortcuts:**

```pug
a(href='/about', class='link') About
//- Shorthand:
a.link(href='/about') About
```

| Feature | Syntax | Purpose |
|---------|--------|---------|
| Implicit tag | `h1= title` | Creates `<h1>` with content. |
| Mixin | `mixin name(args)` / `+name(args)` | Reusable code block. |
| Attribute shortcut | `.class#id` | Shorthand for class and id attributes. |

**Constraints and Limitations:**
- Indentation must be consistent (2 spaces or 1 tab).
- Mixins are compiled to functions and can take arguments.
- Mixin blocks allow passing a block of Pug as content.

#### Annotated Code Example

```pug
//- views/layout.pug
doctype html
html
  head
    title= title
    block scripts
  body
    block content
    block footer
      p Copyright 2026
```

```pug
//- views/page.pug
extends layout.pug

mixin productCard(name, price)
  .card
    h3= name
    p= formatCurrency(price)

block content
  h1= title
  +productCard('Widget', 29.99)
  +productCard('Gadget', 49.99)
```

**Expected Output (for rendering `page` with `{ title: 'Products' }` and a `formatCurrency` helper):**
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Products</title>
  </head>
  <body>
    <h1>Products</h1>
    <div class="card">
      <h3>Widget</h3>
      <p>$29.99</p>
    </div>
    <div class="card">
      <h3>Gadget</h3>
      <p>$49.99</p>
    </div>
    <div id="footer"><p>Copyright 2026</p></div>
  </body>
</html>
```

**Why this output:** The `page.pug` extends `layout.pug` and overrides the `content` block. The `productCard` mixin is called twice with different arguments, generating two card blocks. The `footer` block uses the default content from the layout.

#### Real-World Cases

- **Multi-page websites:** A single `layout.pug` provides the common structure; each page extends it.
- **Admin dashboards:** A `dashboard-layout.pug` with sidebar and navbar blocks; pages override the content block.
- **Blogs:** A `post-layout.pug` with metadata blocks; individual posts override title, date, and body.

---

## Core Concept 3: Handlebars / Mustache

### Definitions

**Core Definition:** Handlebars is a semantic template engine that is largely compatible with Mustache templates, providing logic-less templates with optional custom helpers for complex logic.

**Technical Definition:** Handlebars compiles templates into JavaScript functions, making template execution faster than most other template engines. It uses double-brace syntax (`{{ }}`) for variable interpolation and block helpers (`{{#if}}`, `{{#each}}`) for control flow. Mustache is a logic-less template syntax that can be used for HTML, config files, or source code, with zero dependencies.

**Beginner-Friendly Explanation:** Handlebars looks like regular HTML with `{{ }}` placeholders. It's "logic-less" — you can't write arbitrary JavaScript. Instead, you use built-in helpers like `{{#if}}` and `{{#each}}`, or register your own custom helpers for more complex logic.

### Purposes

- To provide semantic templates that are easy to read and maintain.
- To enforce separation of concerns by restricting arbitrary logic in templates.
- To enable custom logic through registered helpers.
- To support precompilation for faster runtime rendering.
- To be largely compatible with Mustache templates.

### Sub-Feature 3.1: Logic-Less Templates, Custom Block Helpers, and Precompilation

#### Syntax Rules and Structure

**Variable Interpolation:**

```handlebars
{{title}}
{{{rawHtml}}}
```

**Block Helpers:**

```handlebars
{{#if user.isAdmin}}
  <p>Admin Panel</p>
{{else}}
  <p>User Dashboard</p>
{{/if}}

<ul>
  {{#each items}}
    <li>{{this.name}}</li>
  {{/each}}
</ul>
```

**Custom Helper Registration:**

```js
Handlebars.registerHelper('formatCurrency', (amount) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount)
);
```

| Helper | Purpose |
|--------|---------|
| `{{#if}}` | Conditional rendering. |
| `{{#each}}` | Iterate over arrays. |
| `{{#unless}}` | Inverse conditional. |
| `{{this}}` | Current item in `each`. |
| `{{@index}}` | Current index in `each`. |

**Constraints and Limitations:**
- Handlebars is logic-less; complex logic should be handled in the route or with helpers.
- `{{#each}}` does not support `else` for empty arrays by default.
- Helpers must be registered before the template is compiled.
- Mustache is even more restricted than Handlebars; it has no built-in helpers beyond sections and inverted sections.

#### Annotated Code Example

```js
// app.js — Handlebars helpers
const express = require('express');
const exphbs = require('express-handlebars');
const Handlebars = require('handlebars');
const app = express();

app.engine('hbs', exphbs({ defaultLayout: 'main', extname: '.hbs' }));
app.set('view engine', 'hbs');

Handlebars.registerHelper('formatCurrency', (amount) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount)
);

Handlebars.registerHelper('ifCond', function (v1, operator, v2, options) {
  switch (operator) {
    case '===': return v1 === v2 ? options.fn(this) : options.inverse(this);
    case '>': return v1 > v2 ? options.fn(this) : options.inverse(this);
    default: return options.inverse(this);
  }
});

app.get('/', (req, res) => {
  res.render('index', {
    title: 'Helpers Demo',
    price: 29.99,
    stock: 5
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```handlebars
{{! views/index.hbs }}
<h1>{{title}}</h1>
<p>Price: {{formatCurrency price}}</p>
{{#ifCond stock '>' 0}}
  <p>In stock</p>
{{else}}
  <p>Out of stock</p>
{{/ifCond}}
```

**Expected Output (for `GET /`):**
```html
<h1>Helpers Demo</h1>
<p>Price: $29.99</p>
<p>In stock</p>
```

**Why this output:** The `formatCurrency` helper is called with the `price` value. The `ifCond` block helper compares `stock` with `0` using the `>` operator and renders the appropriate block.

#### Real-World Cases

- **E-commerce:** `formatCurrency` for prices, `ifCond` for stock status.
- **Blogs:** `formatDate` for post dates, `truncate` for excerpts.
- **Dashboards:** `formatDate` for timestamps, `ifCond` for status badges.

---

## Core Concept 4: Alternative Engines

### Definitions

**Core Definition:** Alternative template engines provide different trade-offs between syntax familiarity, feature richness, and performance, including Nunjucks (Jinja2-like), Liquid (Shopify-compatible), and raw HTML with minimal interpolation.

**Technical Definition:** Nunjucks is a port of Jinja2, offering template inheritance, filters, and macros with a syntax similar to Python's Jinja2. LiquidJS is a pure JavaScript implementation of the Liquid template language, compatible with Shopify templates, and provides safe, fault-tolerant parsing to an AST with no `eval` or `new Function`. Raw HTML with minimal interpolation uses Express's built-in capabilities or simple string replacement for cases where a full template engine is unnecessary.

**Beginner-Friendly Explanation:** Nunjucks looks like Jinja2 (Python's template engine). Liquid is what Shopify uses. Raw HTML with minimal interpolation is for when you just want to insert a few variables into an HTML file without learning a new template language.

### Purposes

- To provide Jinja2-like syntax for developers familiar with Python ecosystems.
- To offer Shopify-compatible templates for e-commerce integrations.
- To enable safe, sandboxed template rendering without arbitrary code execution.
- To support raw HTML rendering with minimal interpolation for simple use cases.
- To provide streaming rendering for large pages with low memory usage.

### Sub-Feature 4.1: Nunjucks — Jinja2-Like Templating

#### Syntax Rules and Structure

```js
const nunjucks = require('nunjucks');
nunjucks.configure('views', { autoescape: true, express: app });
app.set('view engine', 'njk');
```

**Variable and Filter Syntax:**

```njk
{{ username }}
{{ foo | title }}
{{ foo | join(",") }}
{{ foo | replace("foo", "bar") | capitalize }}
```

| Feature | Syntax |
|---------|--------|
| Variable | `{{ username }}` |
| Filter | `{{ foo | title }}` |
| Inheritance | `{% extends "parent.html" %}` |
| Block | `{% block header %}...{% endblock %}` |

**Constraints and Limitations:**
- Nunjucks does not sandbox execution; it is not safe to run user-defined templates.
- Community has adopted `.njk` file extension.

#### Annotated Code Example

```njk
{# views/parent.njk #}
{% block header %}Default header{% endblock %}
{% block left %}{% endblock %}
{% block right %}More content{% endblock %}
```

```njk
{# views/child.njk #}
{% extends "parent.njk" %}
{% block left %}This is the left side!{% endblock %}
{% block right %}This is the right side!{% endblock %}
```

**Expected Output:**
```html
Default header
This is the left side!
This is the right side!
```

**Why this output:** The child template extends the parent and overrides the `left` and `right` blocks. The `header` block is not overridden, so the default content is used.

---

### Sub-Feature 4.2: LiquidJS — Shopify-Compatible Templating

#### Definitions

**Core Definition:** LiquidJS is a pure JavaScript implementation of the Liquid template language, compatible with Shopify templates, and provides safe parsing to an AST with no `eval` or `new Function`.

**Technical Definition:** LiquidJS supports all filters and tags from Ruby `shopify/liquid`, so Shopify templates work out of the box. It can render directly to a Node.js stream with `renderToNodeStream`, emitting output as it's produced for faster time to first byte and low memory usage. Integration with Express uses `app.engine('liquid', engine.express())`.

**Beginner-Friendly Explanation:** Liquid is the template language used by Shopify. LiquidJS lets you use the same syntax in Node.js. It's designed to be safe — templates are parsed, not executed as code, so you can't accidentally run malicious JavaScript.

#### Syntax Rules and Structure

```js
const { Liquid } = require('liquidjs');
const engine = new Liquid();
app.engine('liquid', engine.express());
app.set('views', './views');
app.set('view engine', 'liquid');
```

**Template Syntax:**

```liquid
{% if user.isAdmin %}
  <p>Admin Panel</p>
{% else %}
  <p>User Dashboard</p>
{% endif %}

<ul>
  {% for item in items %}
    <li>{{ item.name }}</li>
  {% endfor %}
</ul>
```

| Tag | Purpose |
|-----|---------|
| `{% if %}` | Conditional rendering. |
| `{% for %}` | Iteration. |
| `{{ variable }}` | Output variable. |
| `{{ variable \| filter }}` | Apply filter. |

**Constraints and Limitations:**
- LiquidJS caches templates when `cache: true` is set; recommended for production.
- The `root` option specifies the template directory; Express views are also searched.

#### Annotated Code Example

```js
// app.js — LiquidJS with Express
const express = require('express');
const { Liquid } = require('liquidjs');
const app = express();

const engine = new Liquid({ cache: process.env.NODE_ENV === 'production' });
app.engine('liquid', engine.express());
app.set('views', './views');
app.set('view engine', 'liquid');

app.get('/', (req, res) => {
  res.render('index', {
    title: 'Liquid Demo',
    items: [{ name: 'Widget' }, { name: 'Gadget' }]
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```liquid
<!-- views/index.liquid -->
<h1>{{ title }}</h1>
<ul>
  {% for item in items %}
    <li>{{ item.name }}</li>
  {% endfor %}
</ul>
```

**Expected Output (for `GET /`):**
```html
<h1>Liquid Demo</h1>
<ul>
  <li>Widget</li>
  <li>Gadget</li>
</ul>
```

**Why this output:** LiquidJS parses the template, iterates over the `items` array, and outputs each item's `name` property. The `title` variable is interpolated into the `<h1>` tag.

#### Real-World Cases

- **Shopify integrations:** LiquidJS allows Node.js applications to render Shopify-compatible templates.
- **Jekyll and GitHub Pages:** Liquid templates from these platforms work out of the box.
- **E-commerce platforms:** Product listings, order confirmations, and email templates.

---

## Comparison Summary

| Engine | Syntax Style | Logic | Performance | Best For |
|--------|-------------|-------|-------------|----------|
| **EJS** | HTML + `<% %>` | Full JavaScript | Fast (cached functions) | Developers who know JS and want flexibility |
| **Pug** | Indentation-based | Full JavaScript | Fastest (precompiled) | Clean, concise templates |
| **Handlebars** | `{{ }}` | Logic-less + helpers | Fast (precompiled) | Teams wanting enforced separation |
| **Mustache** | `{{ }}` | Logic-less | Fast | Minimal templating needs |
| **Nunjucks** | `{{ }}` + `{% %}` | Jinja2-like | Moderate | Python developers, feature-rich |
| **LiquidJS** | `{{ }}` + `{% %}` | Safe, sandboxed | Moderate | Shopify compatibility, safety |

---

## References

- Using template engines with Express — https://expressjs.com/en/guide/using-template-engines.html
- EJS Official Documentation — https://ejs.co/
- EJS GitHub Repository — https://github.com/mde/ejs
- Pug Official Documentation — https://pugjs.org/api/getting-started.html
- Pug Template Inheritance — https://pugjs.org/language/inheritance.html
- Pug Mixins — https://pugjs.org/language/mixins.html
- Handlebars Official Documentation — https://handlebarsjs.com/
- Handlebars Built-in Helpers — https://handlebarsjs.com/guide/builtin-helpers.html
- Mustache.js GitHub Repository — https://github.com/janl/mustache.js
- Nunjucks Templating Documentation — https://mozilla.github.io/nunjucks/templating.html
- Nunjucks Getting Started — https://mozilla.github.io/nunjucks/getting-started.html
- LiquidJS Official Documentation — https://liquidjs.com/
- LiquidJS Use in Express.js — https://liquidjs.com/tutorials/use-in-expressjs.html
- express-handlebars npm Package — https://www.npmjs.com/package/express-handlebars
- @ladjs/consolidate — https://www.npmjs.com/package/@ladjs/consolidate
- Template Engine Benchmark — https://github.com/itsarnaud/template-engine-bench