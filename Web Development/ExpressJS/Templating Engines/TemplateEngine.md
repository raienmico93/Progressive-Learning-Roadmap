# Express.js Templating Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Templating in Express.js is the mechanism by which static template files are transformed into dynamic HTML at runtime, with variables and logic replaced by actual values before the result is sent to the client.

**Technical Definition**
Express uses a pluggable **view engine** architecture. A template engine enables the use of static template files in an application; at runtime, the engine replaces variables in the template with actual values and transforms the template into an HTML file sent to the client. Express supports any Express-compliant template engine that exports a function named `__express(filePath, options, callback)`, which `res.render()` calls to render the template code. The two application settings that control this system are `views` (the directory containing template files) and `view engine` (the default engine to use).

**Beginner-Friendly Explanation**
Templating lets you write HTML pages with placeholders—like “Hello, {{name}}”—and then fill in those placeholders with real data when a user requests the page. Instead of building HTML strings by hand in your JavaScript code, you write a template file, and Express uses a template engine to combine the template with your data and send the finished HTML to the browser.

---

### Key Characteristics

- **Pluggable engines** — Express does not force a specific template engine; Pug, EJS, Handlebars, Nunjucks, LiquidJS, and over a dozen others are supported via `@ladjs/consolidate` or native `__express` compliance.
- **Two-setting configuration** — The `views` directory and the `view engine` default are the only two settings required to enable templating.
- **Layered locals** — Template variables are merged from three sources with a defined precedence: `res.render()` options override `res.locals`, which override `app.locals`.
- **Environment-aware caching** — View caching is enabled by default in production (`NODE_ENV=production`) and disabled in development; only the compiled template is cached, not the rendered output.
- **Security-sensitive** — `res.render()` performs filesystem operations and evaluates modules; the `view` argument and the `locals` object must not contain unsanitized user input.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A template engine package installed via npm (e.g., `ejs`, `pug`, `hbs`).
- Basic familiarity with HTML and JavaScript expressions.
- Understanding of Express routing and the `res` object.

---

### Related Programming Areas

- **Routing** — Routes invoke `res.render()` to respond with rendered HTML.
- **Middleware** — Middleware populates `res.locals` with request-scoped data for templates.
- **Security** — Auto-escaping and XSS prevention are templating concerns.
- **Static file serving** — CSS, JavaScript, and images referenced by templates are served via `express.static()`.
- **Error handling** — Error-handling middleware can render dedicated error templates.

---

## Core Concept 1: Template Engines

### Definitions

**Core Definition**
A template engine is a library that combines template files containing placeholders and logic with data to produce finished HTML output.

**Technical Definition**
An Express-compliant template engine exports a function named `__express(filePath, options, callback)` that `res.render()` invokes to render the template. Some engines do not follow this convention natively; the `@ladjs/consolidate` library maps all popular Node.js template engines to the Express convention and works seamlessly within Express. Engines can be registered explicitly with `app.engine(ext, callback)`, where the callback has the signature `(path, options, callback)`.

**Beginner-Friendly Explanation**
A template engine is the tool that takes your template file and your data and produces the final HTML. EJS, Pug, and Handlebars are different template engines with different syntaxes, but they all do the same basic job.

---

### Purposes

- To separate presentation (HTML structure) from application logic (route handlers and business logic).
- To enable dynamic content generation without manual string concatenation.
- To provide automatic HTML escaping and reduce XSS risk.
- To support reusable partials, layouts, and components across pages.

---

### Syntax Rules and Structure

#### Complete General Syntax: Popular Engine Comparison

| Engine | Syntax Style | Learning Curve | Key Traits |
|--------|--------------|----------------|------------|
| **EJS** | HTML with `<% %>` tags | Low | Simple, similar to PHP/ASP |
| **Pug** | Indentation-based, minimal | Medium | Concise, powerful, no closing tags |
| **Handlebars** | HTML with `{{}}` expressions | Low | Logic-less templates |

**Component Breakdown**

| Engine | Escaped Output | Unescaped Output |
|--------|----------------|-------------------|
| EJS | `<%= value %>` | `<%- value %>`（trusted only） |
| Pug | `#{value}` or `p= value` | `!{value}` or `p!= value` |
| Handlebars | `{{value}}` | `{{{value}}}` |

#### Syntax Rules

- The engine must either natively export `__express` or be wrapped by `@ladjs/consolidate`.
- The default engine is set via `app.set('view engine', 'ejs')`; Express loads the module internally once the setting is configured.
- When the view engine is set, file extensions can be omitted in `res.render()` calls; otherwise, the extension is required.

#### Constraints and Limitations

- Pug is indentation-sensitive and can produce confusing errors if whitespace is inconsistent.
- Handlebars is “logic-less”; complex logic must be implemented in helpers or pre-processed data.
- EJS allows arbitrary JavaScript, which increases flexibility but also risk if misused.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: EJS Template Rendering

```javascript
// app.js
const express = require('express');
const path = require('path');
const app = express();

app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.get('/', (req, res) => {
  res.render('index', {
    title: 'Home',
    user: { name: 'Alice' },
    isAdmin: true
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
  <h1>Welcome, <%= user.name %>!</h1>
  <% if (isAdmin) { %>
    <div class="admin-panel">Admin controls</div>
  <% } %>
</body>
</html>
```

**Step-by-Step Explanation**

1. **Engine selection** — `app.set('view engine', 'ejs')` tells Express to use EJS and load the `ejs` module internally.
2. **Directory configuration** — `app.set('views', ...)` points to the folder containing `index.ejs`.
3. **Rendering** — `res.render('index', { ... })` passes locals to the template, and EJS substitutes them into the HTML.

**Expected Output**

```html
<!DOCTYPE html>
<html>
<head><title>Home</title></head>
<body>
  <h1>Welcome, Alice!</h1>
  <div class="admin-panel">Admin controls</div>
</body>
</html>
```

**Why This Output Occurs**

- EJS evaluates the `<%= %>` expressions and the `<% if %>` block, producing the final HTML.

---

### Real-World Cases

- **Server-rendered web applications** — Traditional multi-page apps render full HTML on each request.
- **Email templates** — The same engines render HTML emails using `app.render()` for programmatic string generation.
- **Error pages** — Dedicated error templates render status-specific pages for `404` and `500` responses.

---

## Core Concept 2: View Engine

### Definitions

**Core Definition**
The view engine is the default template engine that Express uses to render views when a file extension is not explicitly provided in the `res.render()` call.

**Technical Definition**
The `view engine` application setting specifies the default template engine. Once set, Express loads the corresponding module internally, and `res.render()` can omit the file extension. Express can also be configured with multiple engines registered via `app.engine(ext, callback)`, allowing different file extensions to be handled by different engines. If the `view engine` property is not set, the extension must be specified in every `res.render()` call.

**Beginner-Friendly Explanation**
The view engine is the “default engine” Express uses when you say `res.render('home')` without telling it which type of template it is. Setting it once saves you from repeating the engine name on every render call.

---

### Purposes

- To establish a default rendering engine so file extensions can be omitted.
- To enable Express to load the engine module automatically without explicit `require()` in every route.
- To support multiple engines for different file types in the same application.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.set('view engine', 'ejs');
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `'view engine'` | Application setting name |
| `'ejs'` | Name of the installed engine package |

#### Syntax Rules

- The engine package must be installed via npm and Express-compatible.
- Once set, `res.render('home')` resolves to `home.ejs` without the extension.
- Multiple engines can be registered with `app.engine('ext', engineFunction)` and used by specifying the extension.

#### Constraints and Limitations

- The engine must follow the `__express` convention or be wrapped by `@ladjs/consolidate`.
- Changing the view engine at runtime affects subsequent render calls.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Multiple Engines

```javascript
const express = require('express');
const exphbs = require('express-handlebars');
const app = express();

// Register Handlebars for .handlebars files
app.engine('handlebars', exphbs());
// Set EJS as the default engine
app.set('view engine', 'ejs');

app.get('/ejs', (req, res) => res.render('example.ejs', { title: 'EJS' }));
app.get('/hbs', (req, res) => res.render('example.handlebars', { title: 'HBS' }));
```

**Expected Output**

```
GET /ejs  → rendered example.ejs
GET /hbs  → rendered example.handlebars
```

**Why This Output Occurs**

- The default engine handles extension-less renders, while explicit extensions route to their registered engines.

---

### Real-World Cases

- **Gradual migrations** — Using Handlebars for new pages while keeping legacy EJS pages working.
- **Email versus web** — Using a lightweight engine for web pages and a feature-rich engine for emails.

---

## Core Concept 3: `app.set('view engine', ...)`

### Definitions

**Core Definition**
`app.set('view engine', name)` configures the default template engine that Express uses to render views and to determine file extensions for extension-less render calls.

**Technical Definition**
This application setting tells Express which template engine module to load internally. Once set, Express does not require the engine to be explicitly required in application code; it loads the module when rendering. The setting works in conjunction with `app.set('views', ...)` to fully configure the view system.

**Beginner-Friendly Explanation**
This line tells Express, “Use this template engine as your default.” After setting it, you can render templates without typing the file extension every time.

---

### Purposes

- To set the default engine for all extension-less `res.render()` calls.
- To let Express load the engine module automatically, reducing boilerplate.
- To pair with the `views` setting to complete view configuration.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.set('view engine', 'pug');
```

**Component Breakdown**

| Argument | Role |
|----------|------|
| `'view engine'` | The setting key |
| `'pug'` | The engine name (matching the npm package) |

#### Syntax Rules

- The engine package must be installed.
- The setting must be configured before any `res.render()` call.
- It works alongside `app.set('views', ...)`.

#### Constraints and Limitations

- If the engine does not follow the `__express` convention, wrap it with `@ladjs/consolidate`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Setting the Engine Before Routes

```javascript
const express = require('express');
const path = require('path');
const app = express();

app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');

app.get('/', (req, res) => {
  res.render('index', { title: 'Pug Example' });
});
```

**Expected Output**

```
GET / → rendered index.pug
```

**Why This Output Occurs**

- The engine setting allows `res.render('index')` to resolve to `index.pug`.

---

### Real-World Cases

- **Express generator projects** — The default generator sets Pug as the view engine.
- **API-plus-web hybrids** — The engine is set once, and JSON routes remain unaffected.

---

## Core Concept 4: `app.set('views', ...)`

### Definitions

**Core Definition**
`app.set('views', path)` specifies the directory (or directories) where Express looks for template files.

**Technical Definition**
The `views` setting defaults to the `views` directory in the application root. It can be set to an absolute path or a relative path, and it can be an array of directories, in which case Express searches them in order until the template is found. This setting is essential for view resolution when templates are organized outside the default location.

**Beginner-Friendly Explanation**
This setting tells Express where your template files live. If your templates are in a folder called `views`, you usually don't need to change it—but if you organize them differently, this is how you point Express to the right place.

---

### Purposes

- To tell Express where template files are located.
- To support custom directory structures (e.g., `templates/` instead of `views/`).
- To allow multiple view directories searched in priority order.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.set('views', path.join(__dirname, 'views'));
// Or multiple directories
app.set('views', [path.join(__dirname, 'views'), path.join(__dirname, 'admin-views')]);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `path.join(__dirname, 'views')` | Absolute path to the template directory |
| Array | Multiple directories searched in order |

#### Syntax Rules

- Use absolute paths (`path.join(__dirname, ...)`) for reliability across environments.
- The default is the `views` directory in the application root.
- Multiple directories are supported and searched in order.

#### Constraints and Limitations

- Relative paths are resolved from the current working directory, which can vary in deployments.
- If a template exists in multiple directories, the first match wins.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Multiple View Directories

```javascript
const express = require('express');
const path = require('path');
const app = express();

app.set('views', [
  path.join(__dirname, 'views'),
  path.join(__dirname, 'admin-views'),
  path.join(__dirname, 'email-templates')
]);
app.set('view engine', 'ejs');

app.get('/admin', (req, res) => {
  // Express searches all three directories for dashboard.ejs
  res.render('dashboard', { title: 'Admin' });
});
```

**Expected Output**

```
GET /admin → rendered dashboard.ejs (found in whichever directory contains it)
```

**Why This Output Occurs**

- Express searches the directories in order and uses the first matching template.

---

### Real-World Cases

- **Modular applications** — Feature-specific view directories searched alongside shared views.
- **Email templates** — A separate directory for email templates kept apart from web views.

---

## Core Concept 5: `res.render()`

### Definitions

**Core Definition**
`res.render(view [, locals] [, callback])` renders a view template with the provided local variables and sends the rendered HTML string to the client.

**Technical Definition**
The `view` argument is a string path to the template file, either absolute or relative to the `views` setting. If the path lacks an extension, the `view engine` setting determines it; if an extension is present, Express loads the corresponding engine module and uses its `__express` function. The optional `locals` object defines variables for the view, and the optional `callback` receives both the possible error and the rendered string without performing an automated response; when an error occurs, `next(err)` is invoked internally if no callback is provided.

**Beginner-Friendly Explanation**
`res.render()` is the command that says “take this template, fill it with this data, and send the result to the browser.” You give it the template name and an object containing the data, and Express handles the rest.

---

### Purposes

- To render a template with data and send the resulting HTML as the response.
- To support callback-based rendering for custom processing or testing.
- To integrate with error-handling middleware through internal `next(err)` invocation.
- To enable programmatic rendering via `app.render()` when the response should not be sent automatically.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
res.render(view [, locals] [, callback]);
```

**Component Breakdown**

| Parameter | Role |
|-----------|------|
| `view` | Template file path (relative to `views` or absolute) |
| `locals` | Object of variables for the template |
| `callback` | Optional `(err, html)` callback; suppresses automatic response |

#### Syntax Rules

- If `view` has no extension, the `view engine` setting determines it.
- If `view` has an extension, Express loads the module for that engine.
- Without a callback, `res.render()` sends the HTML automatically.
- With a callback, the rendered string is passed to the callback instead of being sent.

#### Constraints and Limitations

- The `view` argument performs filesystem operations and evaluates modules; it must not contain user input.
- The `locals` object keys may be sensitive and should not contain user-controlled input that could affect the engine or enable XSS.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Basic Rendering with Locals

```javascript
app.get('/user', (req, res) => {
  res.render('user', { name: 'Tobi' });
});
```

**Expected Output**

```
GET /user → rendered user template with name = "Tobi"
```

**Why This Output Occurs**

- The `locals` object passes `name` to the template, and `res.render()` sends the rendered HTML.

---

#### Example 2: Callback-Based Rendering

```javascript
app.get('/user', (req, res) => {
  res.render('user', { name: 'Tobi' }, (err, html) => {
    if (err) return res.status(500).send('Render error');
    res.send(html); // must send explicitly when callback is provided
  });
});
```

**Expected Output**

```
GET /user → rendered HTML sent explicitly via res.send()
```

**Why This Output Occurs**

- The callback suppresses the automatic response, so the developer must send the HTML explicitly.

---

### Real-World Cases

- **Web pages** — Standard HTML responses for browser requests.
- **Email generation** — `app.render()` renders templates to strings for email bodies without sending an HTTP response.
- **Error pages** — `res.status(404).render('errors/404', { ... })` sends a status code with a rendered error template.

---

## Core Concept 6: View Locals

### Definitions

**Core Definition**
View locals are the variables made available to a template during rendering, merged from three sources with a defined precedence.

**Technical Definition**
Express merges local variables from three sources when rendering templates, with the following precedence from highest to lowest: (1) variables passed directly to `res.render()` or `app.render()`, (2) `res.locals` (request-scoped variables), and (3) `app.locals` (application-wide variables). `app.locals` is initialized during application setup and includes a reference to `app.settings`; `res.locals` is initialized as an empty object per request. This layered system allows global defaults, request-specific data, and per-render overrides to coexist cleanly.

**Beginner-Friendly Explanation**
View locals are the pieces of data you give to a template. There are three “levels”: `app.locals` for things that are always the same (like the site name), `res.locals` for things that depend on the request (like the logged-in user), and the object you pass directly to `res.render()` for things that only apply to that one page. The most specific level wins.

---

### Purposes

- To provide application-wide constants (site name, version) to all templates via `app.locals`.
- To attach request-scoped data (current user, flash messages) to templates via `res.locals`.
- To pass page-specific data directly to `res.render()`.
- To avoid repeating common variables in every render call.

---

### Syntax Rules and Structure

#### Complete General Syntax: Layered Locals

```javascript
// Application-wide
app.locals.siteName = 'My App';

// Request-scoped middleware
app.use((req, res, next) => {
  res.locals.user = req.user;
  res.locals.isAuthenticated = !!req.user;
  next();
});

// Page-specific
res.render('home', { title: 'Home' });
```

**Component Breakdown**

| Source | Scope | Precedence |
|--------|-------|------------|
| `res.render()` locals | Single render call | Highest |
| `res.locals` | Current request | Middle |
| `app.locals` | Entire application | Lowest |

#### Syntax Rules

- `app.locals` persists for the application lifetime; use it for constants.
- `res.locals` is per-request; set it in middleware before rendering.
- Direct `res.render()` locals override both other sources.

#### Constraints and Limitations

- `res.locals` is not shared across requests; do not use it for global state.
- Mutating `app.locals` at runtime affects all subsequent renders.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Layered Locals in Action

```javascript
app.locals.siteName = 'My App';

app.use((req, res, next) => {
  res.locals.user = { name: 'Alice' };
  next();
});

app.get('/page', (req, res) => {
  res.render('page', { title: 'Page Title', user: { name: 'Override' } });
});
```

```html
<!-- page.ejs -->
<h1><%= siteName %></h1>     <!-- from app.locals -->
<p>User: <%= user.name %></p> <!-- from res.render locals (overrides res.locals) -->
<p>Title: <%= title %></p>    <!-- from res.render locals -->
```

**Expected Output**

```html
<h1>My App</h1>
<p>User: Override</p>
<p>Title: Page Title</p>
```

**Why This Output Occurs**

- `siteName` comes from `app.locals`; `user.name` is overridden by the `res.render()` locals because render locals have the highest precedence.

---

### Real-World Cases

- **Authentication-aware navigation** — `res.locals.isAuthenticated` and `res.locals.user` drive conditional navigation rendering.
- **Site-wide constants** — `app.locals.siteTitle` used in every layout.
- **Flash messages** — `res.locals.flash` set by middleware and displayed once per request.

---

## Core Concept 7: Templating Best Practices

### Definitions

**Core Definition**
Templating best practices are the organizational, security, and performance patterns that keep template-based Express applications maintainable, safe, and efficient.

**Technical Definition**
Best practices encompass: selecting an engine appropriate to team and project needs; organizing templates into `layouts/`, `partials/`, and `pages/` directories; enabling automatic HTML escaping and never disabling it for user input; pre-processing data in route handlers rather than performing database queries or complex logic in templates; caching compiled templates in production; and rendering error-specific templates with correct status codes.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your templates clean and safe. They include organizing your template files, always letting the engine escape user data, doing heavy calculations in your JavaScript code instead of inside the template, and turning on caching in production for speed.

---

### Purposes

- To prevent XSS by relying on automatic escaping and avoiding unescaped output tags for user data.
- To keep templates readable and maintainable through consistent directory structure and partials.
- To improve performance by caching compiled templates and minimizing work inside templates.
- To provide consistent, user-friendly error pages.

---

### Syntax Rules and Structure

#### Complete General Syntax: Recommended Structure

```text
project-root/
├── views/
│   ├── layouts/
│   │   └── main.ejs
│   ├── partials/
│   │   ├── header.ejs
│   │   └── footer.ejs
│   └── pages/
│       ├── home.ejs
│       └── about.ejs
├── public/
└── app.js
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Directory structure | `layouts/`, `partials/`, `pages/` |
| Auto-escaping | Use `<%= %>` (EJS), `#{}` (Pug), `{{}}` (Handlebars) for user data |
| View caching | `app.set('view cache', true)` in production |
| Error templates | `res.status(statusCode).render('errors/error', { ... })` |

#### Syntax Rules

- **Never** use unescaped output tags (`<%- %>`, `!{}`, `{{{}}}`) with user-controlled data.
- **Never** disable auto-escaping globally.
- Pre-process data in route handlers; do not run database queries inside templates.
- Enable `view cache` in production (default when `NODE_ENV=production`).
- Do not pass user input to the `view` argument or as top-level `locals` keys.

#### Constraints and Limitations

- View caching caches only the compiled template, not the rendered output; dynamic data is still evaluated on every request.
- Template engines that are logic-less (Handlebars) require helpers for any non-trivial logic.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Safe Rendering with Layouts and Partials

```javascript
// app.js
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');
app.set('view cache', process.env.NODE_ENV === 'production');

app.use((req, res, next) => {
  res.locals.siteName = 'My App';
  res.locals.user = req.user;
  next();
});

app.get('/', (req, res) => {
  res.render('pages/home', {
    title: 'Home',
    bio: '<script>alert("xss")</script>Safe bio text'
  });
});
```

```html
<!-- views/pages/home.ejs -->
<%- include('../partials/header', { title: title }) %>
<h1>Welcome to <%= siteName %></h1>
<p>Bio: <%= bio %></p>  <!-- Escaped: script tags rendered as text -->
<%- include('../partials/footer') %>
```

**Expected Output**

```html
<!-- Header with title "Home" -->
<h1>Welcome to My App</h1>
<p>Bio: &lt;script&gt;alert("xss")&lt;/script&gt;Safe bio text</p>
<!-- Footer -->
```

**Why This Output Occurs**

- `<%= bio %>` escapes the HTML, neutralizing the script tags; the layout and partials provide structure while `siteName` comes from `res.locals`.

---

### Real-World Cases

- **Production web applications** — View caching, organized templates, and escaped output.
- **Error handling** — Dedicated `errors/` templates rendered with the correct status codes.
- **Internationalized sites** — Locale data passed as locals and rendered with an i18n helper.

---

## References

- Using template engines with Express · Express.js — https://expressjs.com/en/guide/using-template-engines/
- `res.render()` API Reference (Express 4.x) — https://github.com/Jankyboy/expressjs.com/blob/fcaca7fcc6f182a8821352a94a88f9fc10320cbb/_includes/api/en/4x/res-render.md
- Template Rendering (Express internals, DeepWiki) — https://deepwiki.com/expressjs/express/6.2-template-rendering
- Express View Configuration (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/5-express-templating/11-express-view-configuration.mdx
- Express Template Best Practices (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/5-express-templating/13-express-template-best-practices.mdx
- Express Template Caching (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/5-express-templating/9-express-template-caching.mdx
- Prevent XSS in ExpressJS (Semgrep) — https://docs.semgrep.dev/cheat-sheets/express-xss
- Express.js Insecure Template Engine Usage (Sourcery) — https://www.sourcery.ai
- `app.locals` vs `res.locals` (Express documentation and community) — https://expressjs.com/en/5x/api/application/#applocals
- `@ladjs/consolidate` (Express-compatible engine mapping) — https://www.npmjs.com/package/@ladjs/consolidate