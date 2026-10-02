# Express.js Template Engine Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A template engine is a library that enables an Express application to use static template files containing placeholders; at runtime, the engine replaces variables with actual values and transforms the template into an HTML file sent to the client.

**Technical Definition:** Express-compliant template engines export a function named `__express(filePath, options, callback)`, which `res.render()` calls to render the template code. The engine compiles the template source into a JavaScript function, executes it with the merged data context (locals), and produces an HTML string. Express manages template file resolution, caching, and engine loading internally once configured via `app.set('view engine', ...)` and `app.set('views', ...)`.

**Beginner-Friendly Explanation:** A template engine lets you write HTML with special placeholders — like `{{name}}` or `<%= name %>` — that get replaced with real data when the page is rendered. Instead of building HTML strings in JavaScript, you write a template that looks like HTML and let the engine fill in the blanks.

### Key Characteristics

- **Separation of concerns:** Presentation (templates) is separated from business logic (routes and controllers).
- **Server-side rendering:** HTML is generated on the server and sent fully rendered to the client.
- **Engine-agnostic:** Express supports any template engine that follows the `__express` convention, including Pug, EJS, and Handlebars.
- **Layout and partial support:** Most engines provide mechanisms for reusable fragments and structural inheritance.
- **Data binding:** Variables passed via `res.render()`, `res.locals`, or `app.locals` are interpolated into the template.
- **Caching:** The engine caches compiled template functions (not rendered output), so templates are re-rendered on every request.

### Prerequisites

- **Node.js runtime** (v18 or higher recommended).
- **Express.js installed** (`npm install express`).
- **A template engine package** (e.g., `npm install pug`, `npm install ejs`, `npm install express-handlebars`).
- **Basic JavaScript knowledge:** Functions, objects, and callbacks.
- **Understanding of Express routing:** How `app.get()` and `res.render()` work.

### Related Programming Areas

- **Server-Side Rendering (SSR):** Generating HTML on the server for faster initial page loads and SEO.
- **Middleware pipeline:** Locals set in middleware are available in templates.
- **Static file serving:** CSS, JavaScript, and images are served separately from templates.
- **API development:** Template engines can also render JSON, XML, or plain text.
- **Frontend frameworks:** Template engines are alternatives to React, Vue, and Angular for server-rendered pages.

### Core Concepts

1. **Template Engine Architecture** — how Express compiles templates into HTML, and `app.set('view engine')` vs `app.set('views')`.
2. **Core Layout Patterns** — base layouts, partials, fragments, block overrides, and structural inheritance.
3. **Data Binding & Context Interpolation** — local variables, `app.locals`, and `res.locals`.
4. **Control Structures** — conditionals, loops, and list iterations within markup.
5. **Custom Helper Functions** — registering globally accessible utility functions for formatting.

---

## Core Concept 1: Template Engine Architecture

### Definitions

**Core Definition:** Template engine architecture describes how Express discovers template files, loads the engine module, compiles templates into functions, and renders them with data to produce HTML.

**Technical Definition:** When `res.render(view, locals)` is called, Express resolves the view file path using the `views` setting and the `view engine` setting, loads the engine module (cached after first load), calls the engine's `__express` function with the file path, merged options (locals + `app.locals` + `res.locals`), and a callback, and receives the rendered HTML string.

**Beginner-Friendly Explanation:** Express is like a post office. When you call `res.render('index')`, Express looks in the `views` directory for a file called `index` with the extension of your chosen engine. It hands the file and your data to the engine, which fills in the blanks and returns the finished HTML.

### Purposes

- To configure the template engine and view directory for the application.
- To enable automatic engine loading and file extension resolution.
- To control caching behaviour for development and production.
- To allow multiple engines within a single application via `app.engine()`.

### Sub-Feature 1.1: `app.set('view engine')` vs `app.set('views')`

#### Definitions

**Core Definition:** `app.set('views')` specifies the directory where template files are stored, and `app.set('view engine')` specifies which template engine to use.

**Technical Definition:** The `views` setting defaults to the `views` directory in the application root. The `view engine` setting tells Express which module to load for rendering. Once set, the file extension can be omitted in `res.render()`; Express appends the engine's extension automatically. If no view engine is set, the extension must be specified.

**Beginner-Friendly Explanation:** `app.set('views', './templates')` tells Express "my HTML templates live in the `templates` folder." `app.set('view engine', 'pug')` tells Express "these templates are written in Pug." Once both are set, you can write `res.render('index')` instead of `res.render('index.pug')`.

#### Purposes

- To configure the location and type of template files for the application.
- To enable extension-free rendering via `res.render()`.
- To allow custom view directory paths for different project structures.
- To support multiple engines within one application.

#### Syntax Rules and Structure

```js
app.set('views', './views');        // Template directory
app.set('view engine', 'pug');      // Engine name
```

| Setting | Breakdown |
|---------|-----------|
| `'views'` | Directory where template files are located. Defaults to `views` in the app root. |
| `'view engine'` | Engine name (e.g., `'pug'`, `'ejs'`, `'hbs'`). Express loads the module internally. |

**Constraints and Limitations:**
- The engine module must be installed separately (`npm install pug`).
- Some engines do not follow the `__express` convention; use `@ladjs/consolidate` for those.
- The `views` path can be an array of directories, searched in order.

#### Annotated Code Example

```js
// app.js — Configuring the view engine
const express = require('express');
const path = require('node:path');
const app = express();

// ✅ Set the views directory (absolute path recommended)
app.set('views', path.join(__dirname, 'views'));

// ✅ Set the view engine
app.set('view engine', 'pug');

// ✅ Create a route that renders a template
app.get('/', (req, res) => {
  res.render('index', { title: 'Home', message: 'Welcome!' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```pug
//- views/index.pug
doctype html
html
  head
    title= title
  body
    h1= message
```

**Expected Output (for `GET /`):**
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Home</title>
  </head>
  <body>
    <h1>Welcome!</h1>
  </body>
</html>
```

**Why this output:** Express resolves `index` to `views/index.pug`, loads the Pug engine, and calls its `__express` function with the file path and the locals object `{ title: 'Home', message: 'Welcome!' }`. Pug compiles the template, interpolates the variables, and returns the HTML string.

#### Real-World Cases

- **Multi-page websites:** Different routes render different templates from the same `views` directory.
- **Custom directory structures:** `app.set('views', path.join(__dirname, 'templates', 'pages'))` for non-standard layouts.
- **Multiple engines:** `app.engine('html', require('ejs').renderFile)` to use EJS with `.html` extensions.

---

### Sub-Feature 1.2: The `__express` Convention and `app.engine()`

#### Definitions

**Core Definition:** The `__express` convention is the interface that Express-compliant template engines implement, allowing Express to call the engine's rendering function directly.

**Technical Definition:** An Express-compliant engine exports a function named `__express(filePath, options, callback)`. `res.render()` invokes this function. For engines that do not follow this convention, `app.engine(extension, callback)` registers a custom rendering function for a specific file extension.

**Beginner-Friendly Explanation:** Think of `__express` as a standard plug shape. Any template engine that has this plug can connect to Express. If an engine has a different plug shape, `app.engine()` lets you adapt it.

#### Purposes

- To enable Express to render templates without loading the engine module directly.
- To support engines that do not follow the `__express` convention.
- To register custom rendering functions for specific file extensions.

#### Syntax Rules and Structure

```js
// Standard: Express loads the engine automatically
app.set('view engine', 'pug');

// Custom: Register a rendering function for an extension
app.engine('html', require('ejs').renderFile);
app.set('view engine', 'html');
```

| Component | Breakdown |
|-----------|-----------|
| `app.engine(ext, fn)` | Registers a rendering function for the given extension. |
| `fn(filePath, options, callback)` | The rendering function signature. |
| `app.set('view engine', ext)` | Sets the registered engine as the default. |

**Constraints and Limitations:**
- `app.engine()` must be called before `app.set('view engine')`.
- The rendering function must accept `(filePath, options, callback)` and call the callback with `(err, html)`.

#### Annotated Code Example

```js
// app.js — Custom engine registration
const express = require('express');
const ejs = require('ejs');
const app = express();

// ✅ Register EJS for .html files
app.engine('html', ejs.renderFile);

// ✅ Set .html as the default view engine
app.set('view engine', 'html');
app.set('views', './views');

app.get('/', (req, res) => {
  res.render('index', { name: 'World' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```html
<!-- views/index.html -->
<h1>Hello, <%= name %>!</h1>
```

**Expected Output (for `GET /`):**
```html
<h1>Hello, World!</h1>
```

**Why this output:** `app.engine('html', ejs.renderFile)` tells Express to use EJS's `renderFile` function whenever it encounters an `.html` file. `app.set('view engine', 'html')` makes `.html` the default extension, so `res.render('index')` resolves to `views/index.html`.

#### Real-World Cases

- **Using EJS with `.html` extensions:** Many teams prefer `.html` over `.ejs` for editor syntax highlighting.
- **Custom template formats:** Registering a custom engine for `.md` files that render Markdown to HTML.
- **Legacy migrations:** Registering an adapter for an old template engine while migrating to a new one.

---

## Core Concept 2: Core Layout Patterns

### Definitions

**Core Definition:** Layout patterns are reusable template structures — base layouts, partials, fragments, and blocks — that promote DRY (Don't Repeat Yourself) principles across multiple pages.

**Technical Definition:** A **layout** is a master template defining the common HTML skeleton (header, footer, navigation) with placeholders for page-specific content. **Partials** are reusable template fragments (header, footer, sidebar) included in multiple pages. **Blocks** are named regions in a layout that child templates can override, append to, or prepend to. **Structural inheritance** allows templates to extend other templates, overriding specific blocks while inheriting the rest.

**Beginner-Friendly Explanation:** A layout is like a picture frame. The frame (header, footer, sidebar) stays the same, but the picture (page content) changes. Partials are like reusable stickers that you can place on any picture. Blocks are like windows in the frame where you can insert different content.

### Purposes

- To eliminate duplication of common HTML structure across pages.
- To provide consistent navigation, headers, and footers.
- To enable page-specific content insertion through block overrides.
- To support multi-level inheritance for complex site hierarchies.
- To keep templates modular and maintainable.

### Sub-Feature 2.1: Pug Template Inheritance (Block/Extends)

#### Definitions

**Core Definition:** Pug's template inheritance uses the `block` and `extends` keywords to define replaceable regions in a layout and override them in child templates.

**Technical Definition:** A layout template defines named blocks using the `block` keyword. A child template uses `extends layout` to inherit from the layout and then overrides specific blocks. Blocks can have default content that is used if the child does not override them. Pug also supports `block append` and `block prepend` to add content without replacing the default.

**Beginner-Friendly Explanation:** Think of a layout as a form with blank fields. The child template fills in those fields. If the child doesn't fill in a field, the default value is used.

#### Purposes

- To provide a single layout for all pages of a website.
- To override only the parts of the layout that change per page.
- To add scripts or styles for specific pages via `append`.
- To support multi-level inheritance (sub-layouts).

#### Syntax Rules and Structure

```pug
//- layout.pug
doctype html
html
  head
    title= title
    block head
  body
    block content
    block footer
      p Default footer
```

```pug
//- page.pug
extends layout.pug

block content
  h1= title
  p Welcome to #{title}
```

| Keyword | Purpose |
|---------|---------|
| `block name` | Defines a replaceable region. |
| `extends layout` | Inherits from the specified layout. |
| `block append name` | Adds content after the block's default content. |
| `block prepend name` | Adds content before the block's default content. |

**Constraints and Limitations:**
- Only named blocks and mixin definitions can appear at the top level of a child template.
- A child template cannot add content outside of a block.
- Variables defined in the layout are inherited by child templates.

#### Annotated Code Example

```pug
//- views/layout.pug
doctype html
html
  head
    title My Site - #{title}
    block scripts
      script(src='/vendor/jquery.js')
  body
    block content
    block foot
      #footer
        p Copyright 2026
```

```pug
//- views/page-a.pug
extends layout.pug

block scripts
  script(src='/pets.js')

block content
  h1= title
  - var pets = ['cat', 'dog']
  each petName in pets
    include pet.pug
```

```pug
//- views/pet.pug
p= petName
```

**Expected Output (for rendering `page-a` with `{ title: 'Pets' }`):**
```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Site - Pets</title>
    <script src="/pets.js"></script>
  </head>
  <body>
    <h1>Pets</h1>
    <p>cat</p>
    <p>dog</p>
    <div id="footer"><p>Copyright 2026</p></div>
  </body>
</html>
```

**Why this output:** `page-a.pug` extends `layout.pug` and overrides the `scripts` block with a different script. The `content` block is replaced with the pets list. The `foot` block is not overridden, so the default footer is used.

#### Real-World Cases

- **Multi-page websites:** A single `layout.pug` provides the common structure; each page extends it.
- **Admin dashboards:** A `dashboard-layout.pug` with sidebar and navbar blocks; pages override the content block.
- **Blogs:** A `post-layout.pug` with metadata blocks; individual posts override title, date, and body.

---

### Sub-Feature 2.2: EJS Partials and Includes

#### Definitions

**Core Definition:** EJS partials are reusable template fragments included in other templates using the `<%- include() %>` syntax.

**Technical Definition:** EJS provides an `include()` function that reads another template file, renders it with the current context (or a passed object), and inserts the result at the inclusion point. Partials are typically stored in a `partials` subdirectory within `views`.

**Beginner-Friendly Explanation:** A partial is like a LEGO block that you can snap into any page. The header, footer, and navigation are common partials that you include on every page.

#### Purposes

- To reuse common HTML fragments (header, footer, nav) across pages.
- To pass different data to the same partial from different pages.
- To keep templates DRY and maintainable.

#### Syntax Rules and Structure

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
- The include path is relative to the current template unless configured otherwise.
- EJS does not have built-in layout inheritance; use `ejs-mate` or `express-ejs-layouts` for layouts.

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
      <p>Welcome to the home page.</p>
    </main>
    <%- include('partials/footer') %>
  </body>
</html>
```

**Expected Output (for rendering `index` with `{ title: 'Home' }`):**
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
      <p>Welcome to the home page.</p>
    </main>
    <footer><p>© 2026</p></footer>
  </body>
</html>
```

**Why this output:** The `include()` function reads `partials/header.ejs`, renders it with the passed `{ title: 'Home' }` object, and inserts the result. The `title` variable is available in the partial because it is passed explicitly.

#### Real-World Cases

- **Blogs:** A `header.ejs` partial with navigation and a `footer.ejs` partial with copyright.
- **E-commerce:** A `product-card.ejs` partial rendered in a loop on the products page.
- **Admin panels:** A `sidebar.ejs` partial with navigation links, included on every admin page.

---

### Sub-Feature 2.3: Handlebars Layouts and Blocks

#### Definitions

**Core Definition:** `express-handlebars` provides layout support via the `{{{body}}}` placeholder and block helpers for named content regions.

**Technical Definition:** The `express-handlebars` package adds back the concept of layouts that was removed in Express 3.x. A layout template contains the `{{{body}}}` placeholder where the rendered page content is inserted. Named blocks are defined with `{{{block "name"}}}` in layouts and filled with `{{#contentFor "name"}}...{{/contentFor}}` in pages. Partials are included with `{{> partialName}}` and are loaded from a `partials` directory.

**Beginner-Friendly Explanation:** A layout is a picture frame with a hole in the middle (`{{{body}}}`). The page content fills that hole. Blocks are additional smaller holes (for page-specific scripts or styles) that you can fill from the page.

#### Purposes

- To provide layout inheritance for Handlebars templates.
- To allow pages to inject scripts or styles into the layout's `<head>`.
- To reuse partials across pages.
- To support multi-level layouts.

#### Syntax Rules and Structure

```js
// app.js
const exphbs = require('express-handlebars');
app.engine('hbs', exphbs({ defaultLayout: 'main', extname: '.hbs' }));
app.set('view engine', 'hbs');
```

```handlebars
{{! views/layouts/main.hbs }}
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
  {{{block "pageStyles"}}}
</head>
<body>
  {{{body}}}
  {{{block "pageScripts"}}}
</body>
</html>
```

```handlebars
{{! views/home.hbs }}
{{#contentFor "pageScripts"}}
  <script src="/home.js"></script>
{{/contentFor}}
<h1>{{title}}</h1>
<p>Welcome!</p>
```

| Component | Breakdown |
|-----------|-----------|
| `{{{body}}}` | Placeholder for the rendered page content. |
| `{{{block "name"}}}` | Named block for page-specific content. |
| `{{#contentFor "name"}}` | Fills the named block from the page. |
| `{{> partialName}}` | Includes a partial. |

**Constraints and Limitations:**
- `defaultLayout` must be configured or passed to `res.render()`.
- Layouts are resolved from the `layouts` directory (default: `views/layouts`).
- Handlebars escapes HTML by default; use triple braces `{{{ }}}` for unescaped content.

#### Annotated Code Example

```js
// app.js
const express = require('express');
const exphbs = require('express-handlebars');
const app = express();

app.engine('hbs', exphbs({
  defaultLayout: 'main',
  extname: '.hbs',
  layoutsDir: 'views/layouts',
  partialsDir: 'views/partials'
}));
app.set('view engine', 'hbs');

app.get('/', (req, res) => {
  res.render('home', { title: 'Home', message: 'Hello, Handlebars!' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```handlebars
{{! views/layouts/main.hbs }}
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
  {{{block "pageStyles"}}}
</head>
<body>
  {{> header}}
  {{{body}}}
  {{{block "pageScripts"}}}
</body>
</html>
```

```handlebars
{{! views/partials/header.hbs }}
<header><h1>{{title}}</h1></header>
```

```handlebars
{{! views/home.hbs }}
{{#contentFor "pageScripts"}}
  <script src="/home.js"></script>
{{/contentFor}}
<p>{{message}}</p>
```

**Expected Output (for `GET /`):**
```html
<!DOCTYPE html>
<html>
<head>
  <title>Home</title>
</head>
<body>
  <header><h1>Home</h1></header>
  <p>Hello, Handlebars!</p>
  <script src="/home.js"></script>
</body>
</html>
```

**Why this output:** The layout `main.hbs` provides the HTML skeleton with `{{{body}}}` and block placeholders. The `home.hbs` template fills the `pageScripts` block with a script tag and provides the body content. The `header` partial is included in the layout.

#### Real-World Cases

- **Multi-page websites:** A single `main.hbs` layout with partials for header and footer.
- **Admin dashboards:** A `dashboard.hbs` layout with sidebar and content blocks.
- **Marketing pages:** A `landing.hbs` layout with different block overrides for each page.

---

## Core Concept 3: Data Binding & Context Interpolation

### Definitions

**Core Definition:** Data binding is the process of passing data from the Express application to the template engine, where variables are interpolated into the rendered HTML.

**Technical Definition:** Express provides three levels of data binding: `res.render(view, locals)` passes data for a single render call; `res.locals` holds data for the lifetime of a single request-response cycle; `app.locals` holds data for the lifetime of the application. The template engine merges these objects (with `res.render` locals taking precedence over `res.locals`, which take precedence over `app.locals`) and makes them available as variables in the template.

**Beginner-Friendly Explanation:** Think of templates as forms. `app.locals` is like a company-wide directory available to every form. `res.locals` is like a folder for the current customer, available only while they're being served. `res.render` locals are like sticky notes for that specific form.

### Purposes

- To pass dynamic data from route handlers to templates.
- To provide application-wide data (site name, version) to all templates.
- To expose request-specific data (current user, settings) to templates.
- To provide helper functions to all templates.

### Sub-Feature 3.1: `res.render()` Locals

#### Definitions

**Core Definition:** `res.render(view, locals)` passes a locals object to the template for a single rendering operation.

**Technical Definition:** The locals object is merged with `res.locals` and `app.locals` and passed to the template engine. Keys in the `res.render` locals object take precedence over the same keys in `res.locals` and `app.locals`.

**Beginner-Friendly Explanation:** When you call `res.render('index', { title: 'Home' })`, the template receives a `title` variable with the value `'Home'`. This is the most direct way to pass data to a template.

#### Purposes

- To pass route-specific data to a template.
- To override application-level or request-level locals for a specific render.
- To provide data that changes with every request.

#### Syntax Rules and Structure

```js
res.render('index', { title: 'Home', user: req.user });
```

| Component | Breakdown |
|-----------|-----------|
| `'index'` | Template name (extension optional if view engine is set). |
| `{ title: 'Home', user: req.user }` | Locals object passed to the template. |

**Constraints and Limitations:**
- The locals object is merged with `res.locals` and `app.locals`; conflicting keys are resolved in favour of `res.render` locals.
- Locals should not contain user-controlled keys that could affect the view engine's operation.

#### Annotated Code Example

```js
// app.js
app.get('/user/:id', (req, res) => {
  const user = { id: req.params.id, name: 'Alice', role: 'admin' };
  res.render('profile', { title: 'User Profile', user });
});
```

```pug
//- views/profile.pug
h1= title
p Name: #{user.name}
p Role: #{user.role}
```

**Expected Output (for `GET /user/42`):**
```html
<h1>User Profile</h1>
<p>Name: Alice</p>
<p>Role: admin</p>
```

**Why this output:** The `res.render` call passes `title` and `user` to the `profile` template. Pug interpolates `title` as `'User Profile'` and accesses `user.name` and `user.role` from the nested object.

#### Real-World Cases

- **User profiles:** Passing the user object to a profile template.
- **Product pages:** Passing product details to a product template.
- **Search results:** Passing the query and results array to a search template.

---

### Sub-Feature 3.2: `res.locals` — Request-Scoped Variables

#### Definitions

**Core Definition:** `res.locals` is an object whose properties are available to templates rendered during a single request-response cycle.

**Technical Definition:** `res.locals` is populated in middleware and route handlers. It is merged with `app.locals` and `res.render` locals when a template is rendered. Variables set on `res.locals` are available only for the current request and are not shared between requests.

**Beginner-Friendly Explanation:** `res.locals` is like a folder that belongs to the current customer. Everything you put in the folder is available while you're serving that customer, but when the next customer arrives, you get a fresh folder.

#### Purposes

- To expose request-level information (authenticated user, request path) to templates.
- To set variables in middleware that are consumed in templates.
- To provide a clean separation between application-wide and request-specific data.

#### Syntax Rules and Structure

```js
// Middleware: set res.locals
app.use((req, res, next) => {
  res.locals.currentUser = req.user;
  res.locals.path = req.path;
  next();
});
```

| Component | Breakdown |
|-----------|-----------|
| `res.locals.currentUser` | Variable available in all templates for this request. |
| `res.locals.path` | The request path, available in templates. |

**Constraints and Limitations:**
- `res.locals` is reset for each request.
- Keys should not be user-controlled to avoid view engine manipulation.

#### Annotated Code Example

```js
// app.js — Middleware sets res.locals
const express = require('express');
const app = express();

app.use((req, res, next) => {
  res.locals.siteName = 'My App';
  res.locals.currentPath = req.path;
  next();
});

app.get('/', (req, res) => {
  res.render('index', { title: 'Home' });
});
```

```pug
//- views/layout.pug
doctype html
html
  head
    title= siteName + ' - ' + title
  body
    nav
      a(href='/', class=currentPath === '/' ? 'active' : '') Home
    block content
```

**Expected Output (for `GET /`):**
```html
<!DOCTYPE html>
<html>
  <head>
    <title>My App - Home</title>
  </head>
  <body>
    <nav>
      <a href="/" class="active">Home</a>
    </nav>
  </body>
</html>
```

**Why this output:** The middleware sets `res.locals.siteName` and `res.locals.currentPath` before the route handler renders the template. The layout uses these variables alongside the `title` from `res.render` locals.

#### Real-World Cases

- **Authentication:** Setting `res.locals.user` in authentication middleware, available in all templates.
- **Navigation highlighting:** Setting `res.locals.currentPath` to highlight the active link.
- **Flash messages:** Setting `res.locals.flash` from session data, consumed in the layout.

---

### Sub-Feature 3.3: `app.locals` — Application-Wide Variables and Helpers

#### Definitions

**Core Definition:** `app.locals` is an object whose properties persist throughout the life of the application and are available in all templates.

**Technical Definition:** `app.locals` is a JavaScript object (technically a Function object in older Express versions) whose properties are merged into the locals for every `res.render` call. It is useful for providing helper functions and application-level data such as site name, version, and configuration values.

**Beginner-Friendly Explanation:** `app.locals` is like a company directory that every employee (template) can access. The site name, the current year, and helper functions for formatting dates are stored here.

#### Purposes

- To provide application-wide data (site name, version, environment) to all templates.
- To register helper functions accessible from any template.
- To avoid passing the same data to every `res.render` call.

#### Syntax Rules and Structure

```js
app.locals.siteName = 'My App';
app.locals.version = '1.0.0';
app.locals.formatDate = (date) => new Date(date).toLocaleDateString();
```

| Component | Breakdown |
|-----------|-----------|
| `app.locals.siteName` | String available in all templates. |
| `app.locals.formatDate` | Function callable from templates. |

**Constraints and Limitations:**
- Values persist for the life of the application; changing them affects all subsequent requests.
- Do not use native function property names (`name`, `call`, `apply`, `bind`, `length`, `constructor`) as keys.
- `app.locals` is a Function object in Express 3.x; this is not an issue in Express 4.x and 5.x.

#### Annotated Code Example

```js
// app.js — Application-wide locals and helpers
const express = require('express');
const app = express();

app.locals.siteName = 'My App';
app.locals.currentYear = new Date().getFullYear();
app.locals.formatCurrency = (amount) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount);

app.get('/', (req, res) => {
  res.render('index', { title: 'Home' });
});
```

```pug
//- views/index.pug
h1= siteName
p © #{currentYear}
p Price: #{formatCurrency(29.99)}
```

**Expected Output (for `GET /`):**
```html
<h1>My App</h1>
<p>© 2026</p>
<p>Price: $29.99</p>
```

**Why this output:** `app.locals.siteName`, `app.locals.currentYear`, and `app.locals.formatCurrency` are merged into the locals for every render. The template accesses them directly by name.

#### Real-World Cases

- **Site-wide metadata:** `app.locals.siteName` and `app.locals.description` used in the layout's `<title>` and meta tags.
- **Helper functions:** `app.locals.formatDate` used in blog post templates.
- **Configuration:** `app.locals.environment` used to conditionally load analytics scripts.

---

## Core Concept 4: Control Structures

### Definitions

**Core Definition:** Control structures are template syntax constructs that allow conditional logic and iteration within templates.

**Technical Definition:** Template engines provide syntax for `if/else` conditionals, `for`/`forEach` loops, and other control flow constructs. EJS uses standard JavaScript inside `<% %>` tags; Pug uses indentation-based `if`, `else`, `each`, and `while` keywords; Handlebars uses `{{#if}}`, `{{#each}}`, and `{{#unless}}` block helpers.

**Beginner-Friendly Explanation:** Control structures let your templates make decisions ("show this if the user is logged in") and repeat content ("show this card for each product"). They bring programming logic into your HTML.

### Purposes

- To conditionally render HTML based on data values.
- To iterate over arrays and objects to generate repetitive markup.
- To provide fallback content when data is missing.
- To keep templates dynamic and data-driven.

### Sub-Feature 4.1: EJS Conditionals and Loops

#### Definitions

**Core Definition:** EJS embeds JavaScript logic inside `<% %>` tags for control flow, with `<%= %>` for escaped output and `<%- %>` for unescaped output.

**Technical Definition:** `<% if (condition) { %>` opens a conditional block; `<% } %>` closes it. `<% array.forEach(function(item) { %>` iterates over an array. `<%= variable %>` outputs an escaped value; `<%- variable %>` outputs unescaped HTML.

**Beginner-Friendly Explanation:** EJS lets you write regular JavaScript inside your HTML. The `<% %>` tags contain logic that doesn't output anything; the `<%= %>` tags output values.

#### Purposes

- To conditionally display content (e.g., admin panel only for admins).
- To loop over arrays and generate lists.
- To handle empty states with `else` blocks.

#### Syntax Rules and Structure

```ejs
<% if (user.isAdmin) { %>
  <p>Admin Panel</p>
<% } else { %>
  <p>User Dashboard</p>
<% } %>

<ul>
  <% items.forEach(function(item) { %>
    <li><%= item.name %></li>
  <% }); %>
</ul>
```

| Tag | Purpose |
|-----|---------|
| `<% %>` | Executes JavaScript without output. |
| `<%= %>` | Outputs escaped value. |
| `<%- %>` | Outputs unescaped HTML. |

**Constraints and Limitations:**
- The opening and closing tags must be balanced.
- `<%=` escapes HTML; use `<%-` only for trusted content.

#### Annotated Code Example

```html
<!-- views/users.ejs -->
<h1>Users</h1>
<% if (users.length === 0) { %>
  <p>No users found.</p>
<% } else { %>
  <ul>
    <% users.forEach(function(user) { %>
      <li>
        <strong><%= user.name %></strong>
        <% if (user.role === 'admin') { %>
          <span class="badge">Admin</span>
        <% } %>
      </li>
    <% }); %>
  </ul>
<% } %>
```

**Expected Output (for `users = [{ name: 'Alice', role: 'admin' }, { name: 'Bob', role: 'user' }]`):**
```html
<h1>Users</h1>
<ul>
  <li>
    <strong>Alice</strong>
    <span class="badge">Admin</span>
  </li>
  <li>
    <strong>Bob</strong>
  </li>
</ul>
```

**Why this output:** The outer `if` checks if the array is empty. The `forEach` loop iterates over each user. The inner `if` conditionally renders the admin badge only for users with the `admin` role.

#### Real-World Cases

- **Product listings:** Looping over a products array to generate cards.
- **User tables:** Displaying user data with conditional action buttons.
- **Empty states:** Showing "No results found" when a search returns nothing.

---

### Sub-Feature 4.2: Pug Conditionals and Iteration

#### Definitions

**Core Definition:** Pug uses indentation-based syntax for conditionals (`if`, `else if`, `else`) and iteration (`each`, `while`).

**Technical Definition:** Pug's `if`/`else if`/`else` keywords create conditional blocks. `each item in items` iterates over arrays and objects. `while` creates a loop with a condition. Code lines beginning with `-` are unbuffered (no output), and `=` outputs an expression.

**Beginner-Friendly Explanation:** Pug uses indentation instead of angle brackets. You write `if user.isAdmin` on one line, and indent the content that should be shown.

#### Purposes

- To conditionally render content with clean, indentation-based syntax.
- To iterate over arrays with `each`.
- To write loops with `while` when the iteration count is known.

#### Syntax Rules and Structure

```pug
if user.isAdmin
  p Admin Panel
else
  p User Dashboard

ul
  each item in items
    li= item.name
```

| Keyword | Purpose |
|---------|---------|
| `if` / `else if` / `else` | Conditional rendering. |
| `each item in items` | Iterate over arrays and objects. |
| `while condition` | Loop while condition is true. |

**Constraints and Limitations:**
- Indentation must be consistent (2 spaces or 1 tab).
- `each` also supports `each val, index in items` for index access.

#### Annotated Code Example

```pug
//- views/users.pug
h1 Users
if users.length === 0
  p No users found.
else
  ul
    each user in users
      li
        strong= user.name
        if user.role === 'admin'
          span.badge Admin
```

**Expected Output (for the same users array):**
```html
<h1>Users</h1>
<ul>
  <li><strong>Alice</strong><span class="badge">Admin</span></li>
  <li><strong>Bob</strong></li>
</ul>
```

**Why this output:** The `if` checks if the array is empty. The `each` loop iterates over each user. The inner `if` conditionally renders the admin badge.

#### Real-World Cases

- **Navigation menus:** `each link in links` to generate the navigation.
- **Blog archives:** `each post in posts` to generate post previews.
- **Tables:** `each row in data` to generate table rows.

---

### Sub-Feature 4.3: Handlebars Conditionals and Each

#### Definitions

**Core Definition:** Handlebars uses block helpers (`{{#if}}`, `{{#each}}`, `{{#unless}}`) for control flow.

**Technical Definition:** `{{#if condition}}...{{else}}...{{/if}}` renders content conditionally. `{{#each items}}...{{/each}}` iterates over arrays and objects. `{{#unless condition}}` is the inverse of `if`. Inside `each`, `{{this}}` refers to the current item and `{{@index}}` to the current index.

**Beginner-Friendly Explanation:** Handlebars uses `{{#if}}` and `{{#each}}` blocks. You write `{{#if user.isAdmin}}` and close with `{{/if}}`.

#### Purposes

- To provide declarative conditionals without embedding JavaScript.
- To iterate over arrays with built-in index and first/last helpers.
- To handle empty states with `{{else}}`.

#### Syntax Rules and Structure

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

| Helper | Purpose |
|--------|---------|
| `{{#if}}` | Conditional rendering. |
| `{{#each}}` | Iterate over arrays. |
| `{{#unless}}` | Inverse conditional. |
| `{{this}}` | Current item in `each`. |
| `{{@index}}` | Current index in `each`. |

**Constraints and Limitations:**
- Handlebars is logic-less; complex logic should be handled in the route or with helpers.
- `{{#each}}` does not support `else` for empty arrays by default; use `{{#if items.length}}` or a custom helper.

#### Annotated Code Example

```handlebars
{{! views/users.hbs }}
<h1>Users</h1>
{{#if users.length}}
  <ul>
    {{#each users}}
      <li>
        <strong>{{this.name}}</strong>
        {{#if this.isAdmin}}
          <span class="badge">Admin</span>
        {{/if}}
      </li>
    {{/each}}
  </ul>
{{else}}
  <p>No users found.</p>
{{/if}}
```

**Expected Output (for the same users array):**
```html
<h1>Users</h1>
<ul>
  <li>
    <strong>Alice</strong>
    <span class="badge">Admin</span>
  </li>
  <li>
    <strong>Bob</strong>
  </li>
</ul>
```

**Why this output:** The outer `{{#if users.length}}` checks if the array has items. The `{{#each}}` block iterates over each user. The inner `{{#if this.isAdmin}}` conditionally renders the badge.

#### Real-World Cases

- **E-commerce:** `{{#each products}}` to generate product cards.
- **Blogs:** `{{#each posts}}` to generate post previews.
- **Admin panels:** `{{#if user.isAdmin}}` to show admin-only controls.

---

## Core Concept 5: Custom Helper Functions

### Definitions

**Core Definition:** Custom helper functions are JavaScript functions registered on `app.locals` (or the engine's helper system) that can be called from templates to perform formatting, calculation, or other utility tasks.

**Technical Definition:** In Express, `app.locals` accepts both data values and functions. Functions registered on `app.locals` are available as callable helpers in templates. In Handlebars, helpers are registered via `Handlebars.registerHelper()`. In EJS, any function in the locals object is callable. In Pug, functions are callable as mixins or directly.

**Beginner-Friendly Explanation:** A helper is like a calculator that lives inside your template. You give it a value (like a date or a price), and it returns a formatted result (like "January 15, 2026" or "$29.99").

### Purposes

- To format dates, currency, and strings consistently across templates.
- To compute derived values (e.g., full name from first and last).
- To conditionally render content based on complex logic.
- To avoid duplicating formatting logic in multiple templates.

### Sub-Feature 5.1: Registering Helpers on `app.locals`

#### Definitions

**Core Definition:** Helpers registered on `app.locals` are available in all templates rendered by the application.

**Technical Definition:** Any function assigned to a property of `app.locals` is merged into the locals object for every `res.render` call. The function can accept arguments and return a value that is interpolated into the template.

**Beginner-Friendly Explanation:** You attach a helper function to `app.locals`, and from then on, every template can call it by name.

#### Purposes

- To provide consistent formatting across all templates.
- To centralise utility logic in one place.
- To make templates cleaner by moving complex expressions into helpers.

#### Syntax Rules and Structure

```js
app.locals.formatDate = (date) => {
  return new Date(date).toLocaleDateString('en-US', {
    year: 'numeric', month: 'long', day: 'numeric'
  });
};

app.locals.formatCurrency = (amount) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount);
```

| Component | Breakdown |
|-----------|-----------|
| `app.locals.formatDate` | Helper name available in templates. |
| `(date) => { ... }` | Function accepting a value and returning a formatted string. |

**Constraints and Limitations:**
- Helpers must be synchronous; asynchronous helpers require engine-specific support.
- Do not use native function property names (`name`, `call`, `apply`, `bind`, `length`, `constructor`) as helper names.
- Helpers are registered once at application startup.

#### Annotated Code Example

```js
// app.js — Registering helpers on app.locals
const express = require('express');
const app = express();

// ✅ Date formatting helper
app.locals.formatDate = (date) => {
  if (!date) return '';
  return new Date(date).toLocaleDateString('en-US', {
    year: 'numeric', month: 'long', day: 'numeric'
  });
};

// ✅ Currency formatting helper
app.locals.formatCurrency = (amount) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount);

// ✅ Truncation helper
app.locals.truncate = (str, len = 50) =>
  str.length > len ? str.slice(0, len) + '...' : str;

app.get('/', (req, res) => {
  res.render('index', {
    title: 'Helpers Demo',
    created: new Date(),
    price: 29.99,
    description: 'A very long description that needs to be truncated for display purposes.'
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```pug
//- views/index.pug
h1= title
p Created: #{formatDate(created)}
p Price: #{formatCurrency(price)}
p Description: #{truncate(description, 30)}
```

**Expected Output (for `GET /`):**
```html
<h1>Helpers Demo</h1>
<p>Created: January 15, 2026</p>
<p>Price: $29.99</p>
<p>Description: A very long description that...</p>
```

**Why this output:** The `formatDate` helper converts the `Date` object to a readable string. The `formatCurrency` helper formats the number as USD. The `truncate` helper shortens the description to 30 characters. All helpers are called with arguments and return values that are interpolated into the template.

#### Real-World Cases

- **Blogs:** `formatDate` for post publication dates.
- **E-commerce:** `formatCurrency` for product prices.
- **User profiles:** `truncate` for bio snippets.
- **Admin panels:** `formatDate` for audit log timestamps.

---

### Sub-Feature 5.2: Handlebars Helpers

#### Definitions

**Core Definition:** Handlebars helpers are functions registered with `Handlebars.registerHelper()` that can be called from within templates using `{{helperName arg}}`.

**Technical Definition:** Handlebars helpers receive arguments and an options object (for block helpers). They return a string that is inserted into the template. Helpers can be inline (`{{helper arg}}`) or block (`{{#helper}}...{{/helper}}`).

**Beginner-Friendly Explanation:** Handlebars helpers are like custom tools you add to your template toolkit. Once registered, you can use them anywhere in your Handlebars templates.

#### Purposes

- To add custom formatting logic to Handlebars templates.
- To create reusable block helpers for conditional rendering.
- To extend Handlebars' built-in helpers with application-specific logic.

#### Syntax Rules and Structure

```js
Handlebars.registerHelper('formatCurrency', (amount) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount)
);
```

```handlebars
{{formatCurrency price}}
```

| Component | Breakdown |
|-----------|-----------|
| `Handlebars.registerHelper(name, fn)` | Registers a helper. |
| `fn(arg1, arg2, options)` | Helper function signature. |
| `{{helperName arg}}` | Calls the helper in a template. |

**Constraints and Limitations:**
- Helpers must be registered before the template is compiled.
- Block helpers receive an `options.fn()` function to render the block content.

#### Annotated Code Example

```js
// app.js — Handlebars helpers
const express = require('express');
const exphbs = require('express-handlebars');
const Handlebars = require('handlebars');
const app = express();

app.engine('hbs', exphbs({ defaultLayout: 'main', extname: '.hbs' }));
app.set('view engine', 'hbs');

// ✅ Register helpers
Handlebars.registerHelper('formatDate', (date) => {
  if (!date) return '';
  return new Date(date).toLocaleDateString('en-US', {
    year: 'numeric', month: 'long', day: 'numeric'
  });
});

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
    created: new Date(),
    price: 29.99,
    stock: 5
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

```handlebars
{{! views/index.hbs }}
<h1>{{title}}</h1>
<p>Created: {{formatDate created}}</p>
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
<p>Created: January 15, 2026</p>
<p>Price: $29.99</p>
<p>In stock</p>
```

**Why this output:** The `formatDate` and `formatCurrency` helpers are called with the `created` and `price` values. The `ifCond` block helper compares `stock` with `0` using the `>` operator and renders the appropriate block.

#### Real-World Cases

- **E-commerce:** `formatCurrency` for prices, `ifCond` for stock status.
- **Blogs:** `formatDate` for post dates, `truncate` for excerpts.
- **Dashboards:** `formatDate` for timestamps, `ifCond` for status badges.

---

## References

- Using template engines with Express — https://expressjs.com/en/guide/using-template-engines.html
- Express Application Object (`app.locals`) — https://expressjs.com/en/5x/api/application.html#app.locals
- Express Response Object (`res.locals`) — https://expressjs.com/en/5x/api/response.html#res.locals
- Pug Template Inheritance — https://pugjs.org/language/inheritance.html
- EJS Documentation — https://ejs.co/
- express-handlebars Documentation — https://www.npmjs.com/package/express-handlebars
- express-hbs Documentation — https://www.npmjs.com/package/express-hbs
- @ladjs/consolidate — https://www.npmjs.com/package/@ladjs/consolidate
- GeeksforGeeks: How to create helper functions with EJS in Express — https://origin.geeksforgeeks.org/how-to-create-helper-functions-with-ejs-in-express/