# Handlebars — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Handlebars is a simple templating language that uses a template and an input object to generate HTML or other text formats, with templates that look like regular text containing embedded Handlebars expressions.

**Technical Definition**
Handlebars is a logic-minimized template engine implemented in JavaScript. It compiles templates into render functions, supports HTML-escaped expressions (`{{ }}`), unescaped expressions (`{{{ }}}`), built-in block helpers (`each`, `with`, `if`, `unless`), user-registered helpers, partials for template reuse, and `@data` variables (`@root`, `@index`, `@key`). In Express, it is integrated via the `express-handlebars` view engine, which restores the concepts of layouts and partials that Express removed in version 3.x. Handlebars intentionally restricts template logic—forcing clean separation of data and presentation—and escapes output by default to protect against cross-site scripting (XSS).

**Beginner-Friendly Explanation**
Handlebars lets you write HTML with placeholders like `{{name}}`. When the page is rendered, those placeholders are replaced with real data from your application. It is called “logic-less” because it discourages putting complex code inside templates—you prepare data in your route handlers, and the template simply displays it. Handlebars also automatically escapes data to keep your pages safe from malicious scripts.

---

### Key Characteristics

- **Logic-minimized design** — Handlebars intentionally restricts template logic, encouraging separation of data preparation from presentation.
- **Escaped by default** — Double braces `{{ }}` HTML-escape output, preventing accidental XSS from user data.
- **Triple braces for raw HTML** — `{{{ }}}` disables escaping and must only be used with sanitized, trusted content.
- **Built-in block helpers** — `each`, `with`, `if`, and `unless` handle iteration and conditional rendering without custom JavaScript.
- **Partials for reuse** — Shared template fragments can be registered and included with `{{> partialName}}`.
- **Express integration via layouts** — `express-handlebars` adds layout support, directory-based partials, and production template caching.
- **Context-sensitive escaping** — HTML escaping does not make data safe inside JavaScript, CSS, or URLs; context-specific encoding is required.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- The `handlebars` and `express-handlebars` packages installed via npm.
- Basic familiarity with HTML and JavaScript object notation.
- Understanding of Express routing and `res.render()`.

---

### Related Programming Areas

- **Templating** — Handlebars is one of several Express view engines alongside EJS, Pug, and Nunjucks.
- **Server-side rendering (SSR)** — Handlebars renders full HTML pages on the server.
- **Security (XSS prevention)** — Escaped output and context-aware sanitization are central concerns.
- **Component-based design** — Partials and layouts provide reusable UI fragments.
- **Performance optimization** — Template caching and precompilation reduce rendering overhead.

---

## Core Concept 1: Handlebars

### Definitions

**Core Definition**
Handlebars is a template engine that combines a template containing embedded expressions with an input data object to produce HTML or other text output.

**Technical Definition**
Handlebars compiles a template string into a render function. Expressions delimited by `{{` and `}}` are evaluated against the current context—the input object passed to the template. The engine supports path expressions (`{{person.firstname}}`), block helpers that change the evaluation context (`{{#with person}}`, `{{#each people}}`), user-defined helpers, partials, and `@data` variables. In Express, `express-handlebars` creates an engine instance that handles file I/O, layout resolution, partial registration, and caching. Handlebars is available as both a full package (compiler + runtime) and a runtime-only build for precompiled templates.

**Beginner-Friendly Explanation**
Handlebars is the tool that fills in the blanks in your HTML templates. You write `{{firstname}}` in your template, give Handlebars an object like `{ firstname: "Yehuda" }`, and it produces the finished HTML.

---

### Purposes

- To generate dynamic HTML from templates and data objects without string concatenation.
- To enforce a clean separation between data preparation (controllers) and presentation (templates).
- To provide automatic HTML escaping for safer output by default.
- To enable reusable template fragments through partials and layouts.

---

### Syntax Rules and Structure

#### Complete General Syntax: Basic Expression and Context

```handlebars
<p>{{firstname}} {{lastname}}</p>
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `{{firstname}}` | Expression replaced by the `firstname` property of the context |
| `{{lastname}}` | Expression replaced by the `lastname` property |
| Context | The input object passed to the template |

#### Syntax Rules

- Expressions are surrounded by `{{` and `}}`; the content is a path or helper invocation.
- Dot notation accesses nested properties: `{{person.firstname}}`.
- Block helpers change context: `{{#with person}} ... {{/with}}` and `{{#each people}} ... {{/each}}`.

#### Constraints and Limitations

- Handlebars does not execute arbitrary JavaScript; complex logic must live in helpers or controllers.
- Access to prototype properties is forbidden by default since version 4.6.0.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Basic Express Integration

```javascript
// app.js
const express = require('express');
const { engine } = require('express-handlebars');
const path = require('path');
const app = express();

app.engine('handlebars', engine());
app.set('view engine', 'handlebars');
app.set('views', path.join(__dirname, 'views'));

app.get('/', (req, res) => {
  res.render('home', {
    firstname: 'Yehuda',
    lastname: 'Katz'
  });
});

app.listen(3000);
```

```handlebars
{{!-- views/home.handlebars --}}
<p>{{firstname}} {{lastname}}</p>
```

**Step-by-Step Explanation**

1. **Engine registration** — `app.engine('handlebars', engine())` registers the Handlebars engine factory.
2. **View engine setting** — `app.set('view engine', 'handlebars')` makes Handlebars the default.
3. **Rendering** — `res.render('home', { firstname, lastname })` passes the context object to the template.

**Expected Output**

```html
<p>Yehuda Katz</p>
```

**Why This Output Occurs**

- The expressions `{{firstname}}` and `{{lastname}}` are replaced by the corresponding properties of the input object.

---

### Real-World Cases

- **Server-rendered web applications** — Traditional multi-page apps where each route renders full HTML.
- **Email templates** — Handlebars renders HTML email bodies from shared templates.
- **Prototyping** — Quickly building dynamic pages with minimal template logic.

---

## Core Concept 2: Handlebars Syntax

### Definitions

**Core Definition**
Handlebars syntax is the set of expression delimiters, block helpers, path expressions, and data variables used to embed dynamic values and limited control flow within templates.

**Technical Definition**
Handlebars syntax includes simple expressions (`{{value}}`), block helpers (`{{#helper}} ... {{/helper}}`), inverse sections (`{{else}}`), partials (`{{> partial}}`), comments (`{{! comment }}` and `{{!-- comment --}}`), subexpressions (`{{helper (innerHelper arg)}}`), and `@data` variables (`{{@root}}`, `{{@index}}`, `{{@key}}`). Built-in block helpers include `each` (iteration), `with` (context change), `if`, and `unless`. Whitespace control is available via `{{~` and `~}}`.

**Beginner-Friendly Explanation**
Handlebars syntax is just the special curly-brace tags you put in your HTML. Double curly braces display data, `#` starts a block (like a loop or an if-statement), `/` ends a block, and `>` includes another template file.

---

### Purposes

- To display dynamic values with automatic escaping.
- To iterate over arrays and objects with `{{#each}}`.
- To conditionally render content with `{{#if}}` and `{{#unless}}`.
- To change the evaluation context with `{{#with}}`.

---

### Syntax Rules and Structure

#### Complete General Syntax: Block Helpers and Paths

```handlebars
{{#each people}}
  <li>{{this}} — index {{@index}}</li>
{{/each}}

{{#with person}}
  <p>{{firstname}} {{lastname}}</p>
{{/with}}

{{#if isAdmin}}
  <span>Admin</span>
{{else}}
  <span>User</span>
{{/if}}
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `{{#each array}} ... {{/each}}` | Iterates an array; `this` is the current item, `@index` the position |
| `{{#with object}} ... {{/with}}` | Changes context to the given object |
| `{{#if condition}} ... {{else}} ... {{/if}}` | Conditional block |
| `{{@root}}` | Accesses the initial template context from any depth |
| `{{@key}}` | Current key when iterating an object with `each` |

#### Syntax Rules

- Block helpers must be closed with `{{/helperName}}`.
- `{{else}}` provides an inverse section for `if`, `unless`, and `each`.
- `@data` variables (`@root`, `@index`, `@key`) are available inside block helpers.

#### Constraints and Limitations

- Handlebars does not support arbitrary JavaScript expressions inside templates.
- Subexpressions are delimited by parentheses: `{{outerHelper (innerHelper value)}}`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Iteration with `@index` and `@root`

```handlebars
<ul>
  {{#each items}}
    <li>{{@index}}: {{this}} (site: {{@root.siteName}})</li>
  {{/each}}
</ul>
```

```javascript
// context
{ siteName: 'My Site', items: ['One', 'Two', 'Three'] }
```

**Expected Output**

```html
<ul>
    <li>0: One (site: My Site)</li>
    <li>1: Two (site: My Site)</li>
    <li>2: Three (site: My Site)</li>
</ul>
```

**Why This Output Occurs**

- `@index` provides the zero-based iteration index, and `@root.siteName` reaches the top-level context from inside the loop.

---

### Real-World Cases

- **Data tables** — Rendering rows from database results with `{{#each}}`.
- **Conditional UI** — Showing admin controls only when `isAdmin` is true.
- **Nested object display** — Using `{{#with}}` to simplify access to nested properties.

---

## Core Concept 3: `{{ }}`

### Definitions

**Core Definition**
`{{ }}` is the standard Handlebars expression delimiter that outputs a value **HTML-escaped**, protecting against cross-site scripting when rendering user-controlled data.

**Technical Definition**
By default, the `{{expression}}` syntax HTML-escapes its contents, replacing characters such as `&`, `<`, `>`, `"`, and `'` with HTML entities. This is a deliberate security measure against accidental XSS caused by malicious data passed from the server as JSON. Escaping is context-sensitive to HTML output; it does **not** make data safe inside JavaScript, CSS, URL attributes, or event handlers.

**Beginner-Friendly Explanation**
Double braces are the safe default. When you write `{{comment}}`, Handlebars makes sure any dangerous characters in the comment are converted to harmless text, so a malicious script cannot run.

---

### Purposes

- To safely display untrusted user data in HTML.
- To prevent accidental XSS through automatic HTML escaping.
- To provide the standard, safe output mechanism for template values.

---

### Syntax Rules and Structure

#### Complete General Syntax

```handlebars
<p>{{comment}}</p>
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `{{comment}}` | Outputs the `comment` value with HTML escaping |

#### Syntax Rules

- Use `{{ }}` for all user-controlled data, including names, comments, and search terms.
- HTML escaping protects text content but not attribute contexts without quoting.
- Escaping does not protect against `javascript:` URIs in `href` attributes.

#### Constraints and Limitations

- Escaping is context-dependent; JavaScript and CSS contexts require separate encoding.
- HTML escaping does not prevent unquoted attribute injection.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Escaped vs. Raw Malicious Output

```handlebars
<p>Escaped: {{malicious}}</p>
<p>Raw: {{{malicious}}}</p>
```

```javascript
// context
{ malicious: '<script>alert("xss")</script>' }
```

**Expected Output**

```html
<p>Escaped: &lt;script&gt;alert("xss")&lt;/script&gt;</p>
<p>Raw: <script>alert("xss")</script></p>
```

**Why This Output Occurs**

- `{{ }}` converts `<` and `>` to entities, neutralizing the script tags; `{{{ }}}` inserts the raw string, allowing the script to execute.

---

### Real-World Cases

- **User comments** — Always `{{comment}}` to prevent injected scripts.
- **Profile names** — Displaying usernames safely with escaped output.
- **Search results** — Escaping user-provided search terms in the results page.

---

## Core Concept 4: `{{{ }}}`

### Definitions

**Core Definition**
`{{{ }}}` is the **triple-stash** Handlebars expression that outputs a value **without HTML escaping**, inserting raw content into the rendered page.

**Technical Definition**
The triple-mustache syntax explicitly disables Handlebars’ default escaping. The official Handlebars security guide warns: **do not use triple-stash expressions with untrusted content**, and do not return `Handlebars.SafeString` for unsanitized values, as these disable escaping and can lead to XSS vulnerabilities. If a helper returns HTML, all untrusted values must be escaped before the result is marked as safe. Triple-stash is appropriate only for trusted, sanitized HTML fragments.

**Beginner-Friendly Explanation**
Triple braces mean “insert this exactly as-is, without protection.” Use them only when you are certain the content is safe—like a pre-built HTML snippet from your own server code. Never use them with anything a user typed.

---

### Purposes

- To inject trusted, pre-rendered HTML fragments without double-escaping.
- To render sanitized rich text (e.g., formatted comments) produced by server-side sanitizers.
- To avoid escaping HTML produced by helpers that already return safe markup.

---

### Syntax Rules and Structure

#### Complete General Syntax

```handlebars
{{{trustedHtml}}}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `{{{trustedHtml}}}` | Inserts the value raw, with no HTML escaping |

#### Syntax Rules

- **Only** use `{{{ }}}` for content that has been sanitized or is entirely server-generated.
- **Never** use `{{{ }}}` with `req.body`, `req.query`, or any user-controlled input.
- If a helper returns HTML, escape all untrusted values **before** returning a `SafeString`.

#### Constraints and Limitations

- Triple-stash bypasses all default XSS protection.
- HTML escaping does not protect `javascript:` URIs; validate URL schemes separately.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Trusted HTML Fragment

```handlebars
<div class="bio">{{{sanitizedBio}}}</div>
```

```javascript
// context (bio sanitized server-side before rendering)
{ sanitizedBio: '<strong>Trusted content</strong>' }
```

**Expected Output**

```html
<div class="bio"><strong>Trusted content</strong></div>
```

**Why This Output Occurs**

- `{{{ }}}` inserts the HTML without escaping; the content is trusted because it was sanitized server-side.

---

### Real-World Cases

- **Rich text comments** — Sanitizing Markdown-rendered HTML server-side, then inserting with `{{{ }}}`.
- **Pre-built UI fragments** — Inserting a trusted HTML snippet assembled by a helper.
- **Email templates** — Including trusted boilerplate HTML in transactional emails.

---

## Core Concept 5: Helpers

### Definitions

**Core Definition**
Helpers are JavaScript functions registered with Handlebars that can be invoked from templates to perform custom logic or formatting that the template language itself does not provide.

**Technical Definition**
Helpers are registered with `Handlebars.registerHelper(name, fn)` on a Handlebars environment. Each environment has its own helpers and partials; registering on the global `Handlebars` object makes helpers available to all templates in that environment. Helpers can be **regular helpers** (returning a value) or **block helpers** (receiving an options object with `fn` and `inverse` for the block content). Subexpressions allow helpers to be nested: `{{outerHelper (innerHelper value)}}`. Helpers must validate their arguments and must not convert template input into arbitrary file, network, database, or command access.

**Beginner-Friendly Explanation**
Helpers are custom functions you can call from your templates. If you need to format a date, uppercase a name, or do some calculation, you write a helper and then use it like `{{formatDate date}}` or `{{uppercase name}}`.

---

### Purposes

- To add custom logic and formatting without embedding JavaScript in templates.
- To reuse common transformations across multiple templates.
- To implement block-level behavior (iteration, conditional rendering) through block helpers.
- To extend the template language while keeping templates readable.

---

### Syntax Rules and Structure

#### Complete General Syntax: Registering and Using Helpers

```javascript
const Handlebars = require('handlebars');

Handlebars.registerHelper('uppercase', function (str) {
  return str.toUpperCase();
});
```

```handlebars
{{uppercase name}}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Handlebars.registerHelper(name, fn)` | Registers a helper globally |
| `{{uppercase name}}` | Invokes the helper with `name` as an argument |

#### Complete General Syntax: Block Helper

```javascript
Handlebars.registerHelper('bold', function (options) {
  return '<strong>' + options.fn(this) + '</strong>';
});
```

```handlebars
{{#bold}}Important content{{/bold}}
```

#### Syntax Rules

- Helpers receive the arguments passed in the template, followed by an `options` object.
- Block helpers use `options.fn(context)` to render the block content and `options.inverse(context)` for `{{else}}` content.
- Subexpressions are delimited by parentheses: `{{outer (inner value)}}`.
- Helpers must validate arguments and must not perform arbitrary I/O with template input.

#### Constraints and Limitations

- Helpers are JavaScript functions; they must be registered before rendering.
- Returning raw HTML from a helper requires escaping untrusted values first, or returning a `SafeString` only after sanitization.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Formatting Helper in Express

```javascript
const { engine } = require('express-handlebars');

const hbs = engine({
  helpers: {
    uppercase(str) {
      return String(str).toUpperCase();
    },
    formatPrice(amount) {
      return `$${Number(amount).toFixed(2)}`;
    }
  }
});

app.engine('handlebars', hbs);
app.set('view engine', 'handlebars');
```

```handlebars
<p>Name: {{uppercase user.name}}</p>
<p>Total: {{formatPrice orderTotal}}</p>
```

**Expected Output**

```html
<p>Name: ALICE</p>
<p>Total: $19.99</p>
```

**Why This Output Occurs**

- The `uppercase` and `formatPrice` helpers transform the input values before output.

---

#### Example 2: Block Helper with `{{else}}`

```javascript
Handlebars.registerHelper('ifEven', function (num, options) {
  if (num % 2 === 0) {
    return options.fn(this);
  }
  return options.inverse(this);
});
```

```handlebars
{{#ifEven 4}}Even{{else}}Odd{{/ifEven}}
{{#ifEven 5}}Even{{else}}Odd{{/ifEven}}
```

**Expected Output**

```html
Even
Odd
```

**Why This Output Occurs**

- `options.fn` renders the block when the condition is true; `options.inverse` renders the `{{else}}` block otherwise.

---

### Real-World Cases

- **Date formatting** — A `formatDate` helper for consistent date display.
- **Currency formatting** — A `formatPrice` helper for localized prices.
- **Conditional classes** — A block helper that applies CSS classes based on data.

---

## Core Concept 6: Partials

### Definitions

**Core Definition**
Partials are reusable template fragments that can be included in other templates, enabling code reuse by creating shared templates.

**Technical Definition**
Partials are registered with `Handlebars.registerPartial(name, template)` or loaded from a directory by `express-handlebars`. They are invoked with `{{> partialName}}` (the command character must immediately follow the braces: `{{> partial}}` is valid, `{{ > partial }}` is not). Partials receive the current context by default, but a different context can be passed: `{{> partialName customContext}}`. **Partial blocks** (`{{#> partial}} ... {{/partial}}`) provide fallback content when a partial is missing; the content between the block tags is rendered if the partial cannot be found.

**Beginner-Friendly Explanation**
Partials are reusable pieces of templates—like a header, footer, or product card. You write them once and include them wherever you need them with `{{> partialName}}`. If a partial might be missing, you can provide fallback content with a partial block.

---

### Purposes

- To reuse common template fragments across multiple pages.
- To keep templates DRY and maintainable.
- To pass different contexts to the same partial for flexible reuse.
- To provide fallback content when an optional partial is not available.

---

### Syntax Rules and Structure

#### Complete General Syntax: Partial Invocation and Blocks

```handlebars
{{> header }}
{{> userCard user }}
{{#> optionalPartial}}
  Fallback content when partial is missing
{{/optionalPartial}}
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `{{> partialName}}` | Includes the partial with the current context |
| `{{> partialName context}}` | Includes the partial with a custom context |
| `{{#> partial}} ... {{/partial}}` | Partial block with fallback content |

#### Syntax Rules

- The `>` command character must immediately follow `{{`.
- Partials share the current context unless a custom context is passed.
- Partial blocks render their inner content if the named partial is missing.

#### Constraints and Limitations

- Partials are text-level includes; they do not support block overriding like layouts do.
- Deep partial nesting can complicate debugging.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Header and Footer Partials

```handlebars
{{!-- views/partials/header.handlebars --}}
<header>
  <h1>{{siteName}}</h1>
</header>
```

```handlebars
{{!-- views/home.handlebars --}}
{{> header}}
<main>Welcome!</main>
{{> footer}}
```

**Expected Output**

```html
<header>
  <h1>My Site</h1>
</header>
<main>Welcome!</main>
<footer>...</footer>
```

**Why This Output Occurs**

- The `{{> header}}` partial is replaced with the contents of `header.handlebars`, using the current context (`siteName`).

---

#### Example 2: Partial Block with Fallback

```handlebars
{{#> userBanner}}
  <p>No banner available.</p>
{{/userBanner}}
```

**Expected Output (when `userBanner` partial is missing)**

```html
<p>No banner available.</p>
```

**Why This Output Occurs**

- The partial block renders its inner content as fallback when the named partial is not found.

---

### Real-World Cases

- **Shared navigation** — A header partial included on every page.
- **Product cards** — A card partial rendered for each item in a grid.
- **Optional widgets** — Partial blocks that show fallback content when a widget is not configured.

---

## Core Concept 7: Layouts

### Definitions

**Core Definition**
Layouts provide a shared outer HTML shell—typically containing the `<html>`, `<head>`, and `<body>` structure—that wraps page-specific content, with a placeholder where the page template is injected.

**Technical Definition**
Handlebars itself does not include layout support. In Express, the `express-handlebars` package restores the concept of layouts that Express removed in version 3.x. The layout file contains a `{{{body}}}` placeholder where the rendered page is inserted. The `defaultLayout` option specifies the layout to use when `res.render()` is called without a layout override; `layoutsDir` points to the directory containing layout files. Layouts centralize the page skeleton, reducing duplication across views.

**Beginner-Friendly Explanation**
A layout is the master template that wraps every page. You write the header, footer, and `<html>` structure once in a layout file, and each page only provides its unique content. `express-handlebars` injects the page content into the layout automatically.

---

### Purposes

- To eliminate repetitive HTML skeleton code across pages.
- To centralize `<head>` metadata, CSS links, and script includes.
- To enforce a consistent page structure across an application.
- To simplify maintenance when the global structure changes.

---

### Syntax Rules and Structure

#### Complete General Syntax: Layout Configuration

```javascript
const { engine } = require('express-handlebars');

app.engine('handlebars', engine({
  defaultLayout: 'main',
  layoutsDir: path.join(__dirname, 'views/layouts')
}));
app.set('view engine', 'handlebars');
```

```handlebars
{{!-- views/layouts/main.handlebars --}}
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  {{{body}}}
</body>
</html>
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `defaultLayout: 'main'` | Default layout file for all renders |
| `layoutsDir` | Directory containing layout files |
| `{{{body}}}` | Placeholder where the page content is injected |

#### Syntax Rules

- The layout must contain `{{{body}}}` (triple-stash) to avoid escaping the injected page HTML.
- `defaultLayout` can be overridden per render by passing `layout` in the render options.
- `layoutsDir` tells `express-handlebars` where to find layout files.

#### Constraints and Limitations

- Layouts are provided by `express-handlebars`, not by Handlebars core.
- Only one layout is applied per render; nested layouts require manual composition.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Layout with Partials

```handlebars
{{!-- views/layouts/main.handlebars --}}
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  {{> header}}
  {{{body}}}
  {{> footer}}
</body>
</html>
```

```javascript
app.get('/', (req, res) => {
  res.render('home', {
    title: 'Home',
    siteName: 'My Site'
  });
});
```

```handlebars
{{!-- views/home.handlebars --}}
<h1>Welcome Home</h1>
<p>This is the home page.</p>
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
  <p>This is the home page.</p>
  <footer>...</footer>  <!-- from footer partial -->
</body>
</html>
```

**Why This Output Occurs**

- The page template is rendered and injected into `{{{body}}}`; the layout’s partials surround it.

---

### Real-World Cases

- **Multi-page websites** — Every page shares the same header, footer, and navigation.
- **Dashboard applications** — A sidebar layout with a content placeholder.
- **Marketing sites** — Layout centralizes analytics scripts and meta tags.

---

## Core Concept 8: Handlebars Best Practices

### Definitions

**Core Definition**
Handlebars best practices are the security, performance, and organizational patterns that keep Handlebars-based Express applications safe, fast, and maintainable.

**Technical Definition**
Best practices encompass: using escaped `{{ }}` for all user-controlled data and reserving `{{{ }}}` for sanitized content only; validating URL schemes before placing user input in `href` attributes to prevent `javascript:` execution; enabling production template caching for performance; keeping templates logic-minimal and preparing data in controllers; limiting template size, nesting depth, and iteration counts when users can submit templates to prevent denial of service; and keeping Handlebars up to date while avoiding relaxed prototype access or `allowCallsToHelperMissing`.

**Beginner-Friendly Explanation**
Best practices are the rules that keep Handlebars safe and fast. The biggest rule: always use double braces for user data. The second: turn on caching in production so pages render quickly. The third: keep templates simple and do the complex work in your route handlers.

---

### Purposes

- To prevent XSS through escaped output and context-aware sanitization.
- To improve rendering performance through template caching and precompilation.
- To keep templates maintainable and free of complex logic.
- To protect against denial-of-service when templates are user-supplied.

---

### Syntax Rules and Structure

#### Complete General Syntax: Security and Performance Checklist

```handlebars
{{!-- Safe: escaped output --}}
<p>{{userComment}}</p>

{{!-- Unsafe: raw output --}}
<p>{{{userComment}}}</p>

{{!-- Safe URL: validate scheme in helper --}}
<a href="{{sanitizeUrl userUrl}}">Link</a>
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Escape user data | `{{ }}` for all untrusted input |
| Raw output only trusted | `{{{ }}}` only after server-side sanitization |
| URL sanitization | Validate `https` scheme before rendering in `href` |
| Template caching | Enabled in production by `express-handlebars` |

#### Syntax Rules

- **Never** use `{{{ }}}` with `req.body`, `req.query`, or other user-controlled data.
- **Always** quote HTML attributes containing dynamic values.
- **Never** interpolate user input into `href` without validating the URL scheme.
- Limit template size, nesting depth, and iteration counts if users can submit templates.

#### Constraints and Limitations

- HTML escaping does not make data safe in JavaScript, CSS, or event handler contexts.
- Caching caches compiled templates, not rendered output; dynamic data is evaluated per request.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Secure Template with Caching

```javascript
const { engine } = require('express-handlebars');

app.engine('handlebars', engine({
  defaultLayout: 'main'
}));
app.set('view engine', 'handlebars');
```

```handlebars
{{!-- views/profile.handlebars --}}
<h1>User {{userId}}</h1>
<p>{{bio}}</p>          {{!-- escaped: safe for user data --}}
<div>{{{trustedBio}}}</div> {{!-- raw: sanitized server-side --}}
<a href="{{sanitizeUrl website}}">Website</a>
```

**Expected Output (userId = 42, bio = '<script>alert("xss")</script>', trustedBio = '<strong>Trusted</strong>')**

```html
<h1>User 42</h1>
<p>&lt;script&gt;alert("xss")&lt;/script&gt;</p>
<div><strong>Trusted</strong></div>
<a href="https://example.com">Website</a>
```

**Why This Output Occurs**

- `{{bio}}` escapes the malicious script; `{{{trustedBio}}}` inserts sanitized HTML; the URL helper validates the scheme.

---

### Real-World Cases

- **Production APIs** — Escaped output, sanitized rich text, and URL validation.
- **Security audits** — Verifying no `{{{ }}}` touches user input and no `javascript:` URIs reach `href` attributes.
- **High-traffic sites** — Template caching and precompilation for performance.

---

## References

- Handlebars.js Introduction — https://handlebarsjs.com/guide/
- Handlebars.js Security Guide — https://handlebarsjs.com/zh/guide/security.html
- Handlebars `@data` Variables — https://handlebarsjs.com/guide/expressions.html#data-variables
- Handlebars Partials (core features) — https://handlebarsjs.com/guide/partials.html
- `express-handlebars` npm Package README — https://app.unpkg.com/express-handlebars@4.0.4/files/README.md
- `express-handlebars` API Reference (jsDocs.io) — https://www.jsdocs.io/package/express-handlebars
- Express Template Best Practices (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/5-express-templating/13-express-template-best-practices.mdx
- Handlebars Subexpressions — https://handlebarsjs.com/guide/expressions.html#subexpressions
- Handlebars Partial Blocks (fallback content) — https://handlebarsjs.com/guide/partials.html#partial-blocks