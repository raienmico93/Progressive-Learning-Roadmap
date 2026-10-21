# Other Template Engines in Express.js — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
“Other template engines” in Express.js refers to the non-default, third-party view engines—beyond Pug, EJS, and Handlebars—that can be integrated into Express to render dynamic HTML from templates and data objects.

**Technical Definition**
Express uses a pluggable view-engine architecture. Any template engine that exports a function conforming to `__express(filePath, options, callback)`—or that can be adapted to it—can be registered with `app.engine(ext, callback)` and set as the default engine with `app.set('view engine', ext)`. The engines covered here—Nunjucks, Mustache (via `mustache-express` or `hogan-express`), Liquid (via `liquidjs`), Twig (via `twig`), and Marko—each implement this convention or provide an adapter. Marko notably bypasses Express's view engine entirely through the `@marko/express` package, which adds a `res.marko()` method to enable streaming and modular template organization.

**Beginner-Friendly Explanation**
Express lets you choose from many template engines. Beyond the popular ones like Pug and EJS, there is a whole ecosystem of alternatives. Some, like Nunjucks and Twig, use a Python/Jinja-style syntax. Mustache is the simplest, with almost no logic. Liquid is designed to be safe for user-editable templates. Marko is built for speed and streaming. This cheat sheet covers these five and helps you choose between them.

---

### Key Characteristics

- **Pluggable architecture** — Express does not lock you into a single engine; any compliant engine can be registered and used.
- **Syntax diversity** — Engines range from logic-less (Mustache) to sandboxed (Liquid) to full-featured Jinja-style (Nunjucks, Twig) to HTML-superset streaming (Marko).
- **Security models differ sharply** — Nunjucks explicitly does not sandbox execution and is unsafe for user-defined templates; Mustache is designed to be safe for user consumption; Liquid is sandboxed and recommended for customer-facing template editors.
- **Maintenance status varies** — Some engines (Twig.js) are “work in progress” with limited feature support; others (Marko) are actively developed with modern bundler integrations.
- **Integration approach matters** — Most engines use `app.engine()` + `app.set('view engine')`; Marko uses `res.marko()` via `@marko/express` and bypasses the view engine for streaming.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- Familiarity with `app.engine()`, `app.set('view engine', ...)`, and `res.render()`.
- Basic understanding of HTML and JavaScript object notation.
- Awareness of the security implications of rendering user-controlled data.

---

### Related Programming Areas

- **Templating** — View engines are one component of Express's rendering system.
- **Security (XSS prevention)** — Escaping and sandboxing behavior differs per engine.
- **Performance** — Template compilation, caching, and streaming vary significantly.
- **Ecosystem interoperability** — Some engines are ports of PHP (Twig) or Ruby (Liquid) ecosystems, enabling cross-language template reuse.

---

## Core Concept 1: Nunjucks

### Definitions

**Core Definition**
Nunjucks is a rich and powerful templating language for JavaScript, inspired by Python's Jinja2, built and maintained by Mozilla.

**Technical Definition**
Nunjucks is a server-side (and browser-side) template engine with block inheritance, autoescaping, macros, asynchronous control, and custom filters. It is configured for Express by passing the `express` option to `nunjucks.configure()` or by using the `express-nunjucks` glue package. It supports `{% extends %}`, `{% block %}`, `{% include %}`, `{% for %}`, `{% if %}`, and custom filters registered via `env.addFilter()`. The documentation explicitly states that **Nunjucks does not sandbox execution** and is not safe for running user-defined templates.

**Beginner-Friendly Explanation**
Nunjucks looks a lot like Jinja2 from Python. You write `{% for user in users %}` and `{{ user.name }}`. It has powerful features like template inheritance and custom filters, but you should never let users submit their own Nunjucks templates, because it can execute arbitrary code.

---

### Purposes

- To render dynamic HTML with Jinja2-style control structures and filters.
- To use template inheritance and macros for maintainable, DRY templates.
- To leverage asynchronous filters and custom extensions for advanced rendering needs.
- To share template syntax with Python/Django developers familiar with Jinja2.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Configuration

```javascript
const nunjucks = require('nunjucks');
nunjucks.configure('views', { autoescape: true, express: app });
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `'views'` | Template directory |
| `autoescape: true` | HTML-escapes output by default |
| `express: app` | Wires Nunjucks into Express as a view engine |

#### Complete General Syntax: Template Syntax

```nunjucks
{% extends "base.html" %}
{% block content %}
  <ul>
    {% for user in users %}
      <li>{{ user.name }}</li>
    {% endfor %}
  </ul>
{% endblock %}
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `{% extends %}` / `{% block %}` | Template inheritance |
| `{% for %}` / `{% endfor %}` | Iteration |
| `{{ }}` | Escaped output |

#### Syntax Rules

- `nunjucks.configure(path, options)` must be called before rendering; passing `express: app` enables Express integration.
- `autoescape: true` should always be enabled in production.
- Custom filters are registered with `env.addFilter(name, fn)`.

#### Constraints and Limitations

- **Not sandboxed** — Nunjucks explicitly does not sandbox execution; running user-defined templates is unsafe.
- Autoescaping protects HTML output but not JavaScript/CSS contexts.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Basic Nunjucks Rendering in Express

```javascript
const express = require('express');
const nunjucks = require('nunjucks');
const app = express();

nunjucks.configure('views', { autoescape: true, express: app });

app.get('/', (req, res) => {
  res.render('index.njk', {
    title: 'Nunjucks Demo',
    users: [{ name: 'Alice' }, { name: 'Bob' }]
  });
});

app.listen(3000);
```

```nunjucks
{# views/index.njk #}
<h1>{{ title }}</h1>
<ul>
  {% for user in users %}
    <li>{{ user.name }}</li>
  {% endfor %}
</ul>
```

**Expected Output**

```html
<h1>Nunjucks Demo</h1>
<ul>
    <li>Alice</li>
    <li>Bob</li>
</ul>
```

**Why This Output Occurs**

- The `for` loop iterates the `users` array, and `{{ user.name }}` outputs each name with autoescaping applied.

---

### Real-World Cases

- **Static site generators** — Eleventy (11ty) uses Nunjucks as one of its template languages.
- **Mozilla projects** — Nunjucks was built by Mozilla and is used across Mozilla web properties.
- **Jinja2-compatible workflows** — Teams migrating from Python/Django can reuse template knowledge.

---

## Core Concept 2: Mustache

### Definitions

**Core Definition**
Mustache is a logic-less template syntax that expands tags in a template using values from a data object, with no `if` statements, `else` clauses, or `for` loops—only tags.

**Technical Definition**
Mustache.js is a zero-dependency implementation of the Mustache template system in JavaScript. It is described as “logic-less” because templates contain only tags: some are replaced with a value (`{{name}}`), some with nothing, and others render a series of values via sections (`{{#items}} ... {{/items}}`). Partials are included with `{{> partialName}}`. Lambdas (functions) can be used as values, enabling limited custom behavior. In Express, Mustache is integrated via `mustache-express` or `hogan-express`. The official Mustache documentation notes that all variables are HTML-escaped by default; triple mustaches (`{{{name}}}`) disable escaping.

**Beginner-Friendly Explanation**
Mustache is the simplest template language. You cannot write `if` or `for`—instead, you use special tags called sections that show or repeat content based on the data. It is designed to be safe and portable across many programming languages.

---

### Purposes

- To render templates with minimal logic and maximum portability across languages.
- To enforce strict separation of presentation and logic (logic-less design).
- To use a safe, well-understood template syntax that avoids arbitrary code execution.
- To reuse Mustache templates across JavaScript, Ruby, Python, and other ecosystems.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Configuration

```javascript
const mustacheExpress = require('mustache-express');
app.engine('mustache', mustacheExpress());
app.set('view engine', 'mustache');
```

#### Complete General Syntax: Mustache Tags

```mustache
{{name}}              {{! escaped variable }}
{{{rawHtml}}}         {{! unescaped variable }}
{{#items}} ... {{/items}}  {{! section (iteration/conditional) }}
{{^items}} ... {{/items}}  {{! inverted section }}
{{> partial}}         {{! partial include }}
```

**Component Breakdown**

| Tag | Role |
|-----|------|
| `{{name}}` | Escaped variable |
| `{{{raw}}}` | Unescaped variable |
| `{{#section}}` | Section: iterates arrays or renders if truthy |
| `{{^section}}` | Inverted section: renders if falsy or empty |
| `{{> partial}}` | Partial include |

#### Syntax Rules

- All variables are HTML-escaped by default; triple mustaches disable escaping.
- Sections are used for both iteration and conditional rendering.
- Lambdas can be used as values to produce dynamic content.

#### Constraints and Limitations

- **Strictly logic-less** — No `if`/`else`/`for` statements; only sections and tags.
- Partials must be explicitly passed to the render call or loaded by the Express adapter.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Mustache with Sections and Partials

```javascript
const express = require('express');
const mustacheExpress = require('mustache-express');
const app = express();

app.engine('mustache', mustacheExpress());
app.set('view engine', 'mustache');
app.set('views', __dirname + '/views');

app.get('/', (req, res) => {
  res.render('index', {
    title: 'Mustache Demo',
    users: [{ name: 'Alice' }, { name: 'Bob' }],
    partials: { header: 'header' }
  });
});
```

```mustache
{{!-- views/index.mustache --}}
{{> header}}
<h1>{{title}}</h1>
<ul>
  {{#users}}
    <li>{{name}}</li>
  {{/users}}
</ul>
```

**Expected Output**

```html
<header>...</header>
<h1>Mustache Demo</h1>
<ul>
    <li>Alice</li>
    <li>Bob</li>
</ul>
```

**Why This Output Occurs**

- The `{{#users}}` section iterates the array, rendering the inner content for each item; `{{> header}}` includes the partial.

---

### Real-World Cases

- **Multi-language template reuse** — Mustache templates work in JavaScript, Ruby, Python, Go, and more.
- **Email templates** — Logic-less design reduces the risk of template injection.
- **Static site generators** — Mustache is used in several static site tools.

---

## Core Concept 3: Liquid

### Definitions

**Core Definition**
Liquid is a safe, customer-facing template language created by Shopify, implemented in JavaScript as LiquidJS—a pure JavaScript, Shopify-compatible template engine.

**Technical Definition**
LiquidJS is a simple, safe, Shopify-compatible template engine written in pure JavaScript. It uses two types of markup: **tags** (`{% %}`) for logic and **output** (`{{ }}`) for values with optional filters. It is designed to be sandboxed, preventing unsafe operations and making it suitable for customer-facing template editors. In Express, it is integrated by calling `engine.express()` and registering with `app.engine('liquid', engine.express())`; Express's `views` directory is automatically included when looking up partials and layouts. Caching can be enabled by setting `cache: true` (or a numeric limit) in the `Liquid` constructor.

**Beginner-Friendly Explanation**
Liquid is the template language used by Shopify. It is designed to be safe for non-programmers to edit—like store owners customizing their shop's theme. It cannot run arbitrary JavaScript, so it is much safer than engines like Nunjucks or EJS for user-editable templates.

---

### Purposes

- To provide a safe, sandboxed template language for customer-facing template editing.
- To reuse Shopify-compatible templates and themes in JavaScript applications.
- To render dynamic content with tags, filters, and partials without arbitrary code execution.
- To leverage layout and partial lookup integrated with Express's `views` directory.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Configuration

```javascript
const { Liquid } = require('liquidjs');
const engine = new Liquid({ cache: process.env.NODE_ENV === 'production' });
app.engine('liquid', engine.express());
app.set('view engine', 'liquid');
```

#### Complete General Syntax: Liquid Markup

```liquid
{% for user in users %}
  <li>{{ user.name | upcase }}</li>
{% endfor %}
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `{% tag %}` | Logic tags (for, if, assign, include, layout) |
| `{{ output }}` | Output with optional filters (`| upcase`) |
| `{% include %}` / `{% layout %}` | Partials and layouts |

#### Syntax Rules

- Tags use `{% %}`; output uses `{{ }}`.
- Filters are applied with a pipe: `{{ value | filter: arg }}`.
- Liquid is sandboxed; it cannot execute arbitrary JavaScript.

#### Constraints and Limitations

- Sandboxing limits expressiveness; complex logic must be prepared in controllers.
- Some Shopify-specific tags or filters may not be fully supported by LiquidJS.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: LiquidJS in Express

```javascript
const express = require('express');
const { Liquid } = require('liquidjs');
const app = express();

const engine = new Liquid({ cache: true });
app.engine('liquid', engine.express());
app.set('view engine', 'liquid');
app.set('views', './views');

app.get('/', (req, res) => {
  res.render('index', {
    title: 'Liquid Demo',
    users: [{ name: 'Alice' }, { name: 'Bob' }]
  });
});
```

```liquid
<h1>{{ title }}</h1>
<ul>
  {% for user in users %}
    <li>{{ user.name }}</li>
  {% endfor %}
</ul>
```

**Expected Output**

```html
<h1>Liquid Demo</h1>
<ul>
    <li>Alice</li>
    <li>Bob</li>
</ul>
```

**Why This Output Occurs**

- The `for` tag iterates the `users` array, and `{{ user.name }}` outputs each name within the sandbox.

---

### Real-World Cases

- **Shopify theme development** — Liquid is Shopify's native template language.
- **Customer-facing template editors** — LiquidJS is described as the safest choice for customer-facing template editors.
- **Jekyll-style static sites** — Liquid is used in Jekyll and other static site generators.

---

## Core Concept 4: Twig

### Definitions

**Core Definition**
Twig.js is a pure JavaScript implementation of the Twig PHP templating language, bringing Twig's syntax to Node.js and browsers.

**Technical Definition**
Twig.js is a pure JavaScript port of the Twig PHP templating language. It supports a limited subset of Twig's features, including `extends`, `use`, and `block` tags for template inheritance, and a limited set of filters. It is compatible with Express 2 and 3 (and can be used with later versions via configuration) by setting the view engine to `twig`. It is described as a “work in progress” with more features coming. Its primary appeal is familiarity for developers coming from the Symfony/PHP ecosystem.

**Beginner-Friendly Explanation**
Twig is the template language used in PHP's Symfony framework. Twig.js lets you use the same syntax in JavaScript. If your team already knows Twig from PHP, you can reuse that knowledge. However, Twig.js does not support everything that PHP Twig does.

---

### Purposes

- To reuse Twig syntax and templates across PHP and JavaScript ecosystems.
- To provide template inheritance (`extends`/`block`) in a Twig-compatible syntax.
- To leverage existing Symfony/Twig developer knowledge in Node.js projects.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Configuration

```javascript
const Twig = require('twig');
app.set('view engine', 'twig');
app.set('twig options', { strict_variables: false });
```

#### Complete General Syntax: Twig Markup

```twig
{% extends "base.twig" %}
{% block content %}
  <ul>
    {% for user in users %}
      <li>{{ user.name }}</li>
    {% endfor %}
  </ul>
{% endblock %}
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `{% extends %}` / `{% block %}` | Template inheritance |
| `{% for %}` | Iteration |
| `{{ }}` | Output |

#### Syntax Rules

- Set the view engine to `twig` and optionally configure `twig options`.
- Template inheritance uses `extends` and `block`.
- A limited set of filters is supported.

#### Constraints and Limitations

- **Work in progress** — Supports only a limited subset of the Twig language.
- Documentation is primarily in the GitHub wiki; official Express integration guidance is sparse.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Twig.js in Express

```javascript
const express = require('express');
const Twig = require('twig');
const app = express();

app.set('view engine', 'twig');
app.set('views', './views');

app.get('/', (req, res) => {
  res.render('index.twig', {
    title: 'Twig Demo',
    users: [{ name: 'Alice' }, { name: 'Bob' }]
  });
});
```

```twig
<h1>{{ title }}</h1>
<ul>
  {% for user in users %}
    <li>{{ user.name }}</li>
  {% endfor %}
</ul>
```

**Expected Output**

```html
<h1>Twig Demo</h1>
<ul>
    <li>Alice</li>
    <li>Bob</li>
</ul>
```

**Why This Output Occurs**

- The `for` tag iterates the array, and `{{ user.name }}` outputs each name.

---

### Real-World Cases

- **Symfony/PHP migrations** — Teams moving from PHP to Node.js can reuse Twig knowledge.
- **Cross-language template sharing** — Templates can be shared between PHP and JavaScript services.

---

## Core Concept 5: Marko

### Definitions

**Core Definition**
Marko is a fast and lightweight HTML-based templating engine that compiles templates to CommonJS modules and supports streaming, asynchronous rendering, and custom tags.

**Technical Definition**
Marko is a superset of well-formed HTML: its syntax is based on HTML, and all valid JavaScript statements are allowed. It compiles templates to CommonJS modules and supports streaming directly to the HTTP response stream. Unlike other engines, Marko **bypasses Express's view engine** via the `@marko/express` package, which adds a `res.marko(template, input)` method to the Express response object. This method works like `res.render()` but without the view engine's restrictions, enabling full streaming and modular template organization. By using `res.marko()`, properties from `app.locals` and `res.locals` are automatically available on `$global` in the template.

**Beginner-Friendly Explanation**
Marko is different from the others. It looks like HTML, but you can use JavaScript inside it. It is built for speed: it can stream parts of the page to the browser as soon as they are ready, instead of waiting for the whole page. It also lets you create custom tags (components). It does not use Express's normal view engine; instead, it adds a special `res.marko()` method.

---

### Purposes

- To render HTML with a syntax that is a superset of well-formed HTML.
- To stream rendered output directly to the HTTP response for faster time-to-first-byte.
- To use custom tags (components) and modular template organization.
- To leverage full JavaScript statements inside templates for complex rendering logic.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Integration

```javascript
import express from 'express';
import markoPlugin from '@marko/express';
import template from './template.marko';

const app = express();
app.use(markoPlugin()); // adds res.marko()

app.get('/', (req, res) => {
  res.marko(template, { name: 'Frank', count: 30 });
});
```

#### Complete General Syntax: Marko Template

```marko
$ const { name, count } = input;
<h1>Hello ${name}!</h1>
<if(count > 0)>
  <p>You have ${count} messages.</p>
</if>
```

**Component Breakdown**

| Syntax | Role |
|--------|------|
| `input` | The data object passed to the template |
| `${}` | Output expression (escaped) |
| `<if>`, `<else-if>`, `<else>` | Control flow tags |
| `<custom-tag>` | Reusable components |

#### Syntax Rules

- Use `app.use(markoPlugin())` to enable `res.marko()`.
- Templates are imported as modules and rendered with `res.marko(template, input)`.
- `input` is the data object; `${}` inserts values.
- Control flow uses tags like `<if>`, not JavaScript statements.

#### Constraints and Limitations

- **Bypasses the view engine** — Does not use `app.set('view engine')` or `res.render()`.
- Requires a bundler for most of its API (e.g., Vite, Webpack).
- Older versions of `@marko/express` exposed `app`, `req`, and `res` on `$global`; this is now discouraged due to security risks.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Marko with Streaming and Control Flow

```javascript
import express from 'express';
import markoPlugin from '@marko/express';
import template from './template.marko';

const app = express();
app.use(markoPlugin());

app.get('/', (req, res) => {
  res.marko(template, {
    name: 'Frank',
    count: 3,
    colors: ['red', 'green', 'blue']
  });
});

app.listen(8080);
```

```marko
$ const { name, count, colors } = input;
<h1>Hello ${name}!</h1>
<if(count > 0)>
  <ul>
    <for|color| of=colors>
      <li>${color}</li>
    </for>
  </ul>
</if>
```

**Expected Output**

```html
<h1>Hello Frank!</h1>
<ul>
  <li>red</li>
  <li>green</li>
  <li>blue</li>
</ul>
```

**Why This Output Occurs**

- The `input` object provides the data; `<if>` conditionally renders the list; `<for>` iterates the array; `${}` outputs values.

---

### Real-World Cases

- **eBay** — Marko was created at eBay and is used in eBay's progressive HTML rendering.
- **High-performance server-side rendering** — Streaming enables faster time-to-first-byte for large pages.
- **Component-based applications** — Custom tags provide a component model without a frontend framework.

---

## Core Concept 6: Template Engine Comparison

### Definitions

**Core Definition**
Template engine comparison is the systematic evaluation of Express-compatible engines across syntax style, feature set, security model, performance characteristics, and ecosystem maturity.

**Technical Definition**
The comparison considers: syntax paradigm (logic-less, sandboxed, Jinja-style, HTML-superset); control-flow capabilities; inheritance and partial support; escaping and sandboxing behavior; streaming and async rendering; maintenance status and download volume; and integration approach (view engine vs. custom response method).

---

### Comparison Table

| Engine | Syntax Style | Logic Model | Inheritance / Partials | Security Model | Streaming / Async | Maintenance Status | Best For |
|--------|--------------|-------------|------------------------|----------------|-------------------|--------------------|----------|
| **Nunjucks** | Jinja2-style (`{% %}`) | Full logic (for, if, macros) | `extends` / `block`, `include`, macros | **Not sandboxed**; unsafe for user templates | Async filters supported | Maintained (Mozilla) | Jinja2-familiar teams, static sites (11ty) |
| **Mustache** | Logic-less (`{{ }}`) | Sections only; no if/for | Partials (`{{> }}`) | Designed to be safe for user consumption | No native streaming | Maintained (zero-dependency) | Multi-language portability, simple templates |
| **Liquid (LiquidJS)** | Shopify-style (`{% %}` / `{{ }}`) | Tags and filters; sandboxed | `include`, `layout` | **Sandboxed**; safest for customer-facing editors | No native streaming | Actively maintained | Shopify themes, user-editable templates |
| **Twig (Twig.js)** | Twig/PHP-style (`{% %}`) | Full logic (limited subset) | `extends` / `block`, `use` | Not explicitly sandboxed | No native streaming | Work in progress; limited feature support | Symfony/PHP developers reusing Twig knowledge |
| **Marko** | HTML-superset with JS (`${}`) | Full JavaScript statements + control-flow tags | Custom tags, layouts with nested attributes | Not sandboxed; full JS execution | **Native streaming**, async rendering | Actively developed (eBay) | High-performance streaming SSR, component-based UI |

---

### Purposes

- To select an engine that matches the team's syntax familiarity and security requirements.
- To avoid engines that are unsafe for the intended use case (e.g., Nunjucks for user-defined templates).
- To choose an engine whose performance characteristics (streaming, caching) align with application needs.
- To account for maintenance status and ecosystem longevity.

---

### Syntax Rules and Structure

#### Complete General Syntax: Selection Criteria

| Criterion | Question to Ask |
|-----------|-----------------|
| Syntax familiarity | Does the team know Jinja2, Twig, Liquid, or plain HTML? |
| Security | Will users submit templates? If yes, Liquid or Mustache. |
| Performance | Is streaming or async rendering required? Marko. |
| Feature richness | Are macros, inheritance, and filters needed? Nunjucks or Twig. |
| Ecosystem | Is the engine actively maintained and widely used? |

#### Constraints and Limitations

- No single engine is best for all use cases; trade-offs are inevitable.
- Security model is the most consequential differentiator when templates may be user-supplied.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Same Output Across Engines

**Nunjucks**

```nunjucks
{% for user in users %}<li>{{ user.name }}</li>{% endfor %}
```

**Mustache**

```mustache
{{#users}}<li>{{name}}</li>{{/users}}
```

**Liquid**

```liquid
{% for user in users %}<li>{{ user.name }}</li>{% endfor %}
```

**Twig**

```twig
{% for user in users %}<li>{{ user.name }}</li>{% endfor %}
```

**Marko**

```marko
<for|user| of=input.users><li>${user.name}</li></for>
```

**Expected Output (all engines)**

```html
<li>Alice</li>
<li>Bob</li>
```

**Why This Output Occurs**

- Each engine expresses iteration differently, but all produce the same HTML for the same data.

---

### Real-World Cases

- **Security-sensitive applications** — Choosing Liquid or Mustache when users can edit templates.
- **Performance-critical SSR** — Choosing Marko for streaming and progressive rendering.
- **Team skill alignment** — Choosing Nunjucks or Twig for teams with Jinja2/Symfony experience.
- **Multi-language projects** — Choosing Mustache for template portability across languages.

---

### Template Engine Best Practices

#### Definitions

**Core Definition**
Template engine best practices are the selection, configuration, and security patterns that keep template rendering safe, maintainable, and performant across engines.

**Technical Definition**
Best practices include: matching the engine's security model to the trust level of template authors (sandboxed engines for user templates, non-sandboxed only for trusted developers); enabling autoescaping and never disabling it for user data; enabling production caching; keeping template logic minimal and preparing data in controllers; and evaluating maintenance status before adopting an engine.

---

#### Purposes

- To prevent XSS and code-execution vulnerabilities through appropriate engine selection and configuration.
- To ensure templates remain maintainable by keeping logic in controllers.
- To optimize rendering performance through caching and streaming where appropriate.
- To avoid adopting abandoned or under-maintained engines.

---

#### Syntax Rules and Structure

#### Complete General Syntax: Security-First Selection

```text
Will users submit templates?
├── Yes → Use Liquid (sandboxed) or Mustache (logic-less)
└── No  → Use Nunjucks, Twig, or Marko (trusted developers only)
```

**Component Breakdown**

| Rule | Rationale |
|------|-----------|
| Sandbox for user templates | Nunjucks explicitly does not sandbox; unsafe for user templates |
| Autoescape on | All engines escape by default; keep it on |
| Cache in production | Reduces compilation overhead |
| Minimal template logic | Keeps templates readable and testable |

#### Constraints and Limitations

- Sandboxing reduces expressiveness; some logic must move to controllers.
- Streaming engines like Marko require bundler configuration and are not drop-in view engines.

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Safe Engine Selection

```javascript
// If users can submit templates: use Liquid
const { Liquid } = require('liquidjs');
const engine = new Liquid({ cache: true });
app.engine('liquid', engine.express());
app.set('view engine', 'liquid');

// If only trusted developers write templates: Nunjucks is safe enough
nunjucks.configure('views', { autoescape: true, express: app });
```

**Expected Output**

- User-submitted templates cannot execute arbitrary code with Liquid.
- Nunjucks templates can execute logic but are written by trusted developers.

---

#### Real-World Cases

- **Multi-tenant SaaS** — Liquid for tenant-editable templates.
- **Internal admin tools** — Nunjucks for trusted developer-authored views.
- **Streaming SSR** — Marko for performance-critical pages.

---

## References

- Nunjucks Getting Started (Mozilla) — https://mozilla.github.io/nunjucks/getting-started.html
- Nunjucks Templating Documentation — https://mozilla.github.io/nunjucks/templating.html
- Mustache.js npm Package — https://www.npmjs.com/package/mustache
- Mustache(5) Manpage — https://mustache.github.io/mustache.5.html
- LiquidJS: Use in Express.js — https://liquidjs.com/tutorials/use-in-expressjs.html
- LiquidJS npm Package — https://www.npmjs.com/package/liquidjs
- Twig.js npm Package — https://www.npmjs.com/package/twig
- Twig.js Wiki (Documentation) — https://github.com/twigjs/twig.js/wiki
- Marko + Express (Official Docs) — https://markojs.com/docs/express/
- Marko Syntax Documentation — https://markojs.com/docs/syntax/
- Express.js Template Engines — https://expressjs.com/en/resources/template-engines.html
- Express.js Using Template Engines — https://expressjs.com/en/guide/using-template-engines.html
- JavaScript Templating Engines Comparison (npm-compare) — https://npm-compare.com/dot,ejs,handlebars,hbs,liquidjs,mustache,nunjucks,pug,twig
- “Safe JavaScript Templates” (Stack Overflow, Nunjucks sandboxing discussion) — https://stackoverflow.com/questions/3202513/safe-javascript-templates