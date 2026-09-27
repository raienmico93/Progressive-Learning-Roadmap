# HTML JavaScript Integration: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML JavaScript integration is the set of mechanisms by which executable client-side scripts are embedded in or referenced from an HTML document, and the loading strategies that determine when and how those scripts execute.

**Technical Definition**

JavaScript integration in HTML is defined by the WHATWG HTML Living Standard through the `<script>` element, a metadata content element (when in `<head>`) that is also flow content and phrasing content when placed in the body. The element can contain inline script code or reference an external resource via the `src` attribute. Script execution is governed by the script-processing model, which determines whether execution blocks HTML parsing (classic scripts), is deferred until parsing completes (`defer`), is executed as soon as downloaded (`async`), or is treated as a module (`type="module"`). Module scripts are automatically deferred and execute in strict mode. The `src`, `async`, `defer`, `type`, `crossorigin`, `integrity`, `nomodule`, and `referrerpolicy` attributes control behaviour.

**Beginner-Friendly Explanation**

JavaScript is what makes your web page interactive — buttons that respond to clicks, content that updates without reloading, animations, and more. But you have to connect the JavaScript to your HTML. You do this with the `<script>` tag. You can either write the JavaScript directly inside the tag (inline) or link to a separate `.js` file (external). The tricky part is timing: if the browser runs your script before the HTML is ready, your script can't find the elements it needs. That's why we have `defer`, `async`, and module scripts — they control *when* your JavaScript runs.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Default blocking** | Classic scripts block HTML parsing until downloaded and executed |
| **`defer` preserves order** | Deferred scripts execute after parsing, in document order |
| **`async` is unpredictable** | Async scripts execute as soon as downloaded, in any order |
| **Module scripts are deferred** | `type="module"` scripts are deferred by default and run in strict mode |
| **External is preferred** | External `.js` files are cacheable, maintainable, and CSP-friendly |
| **Inline is discouraged** | Inline scripts (`onclick="..."`) are hard to maintain and a CSP/XSS risk |
| **Placement matters** | Scripts in `<head>` block rendering; scripts before `</body>` do not |
| **CSP-sensitive** | Content Security Policy may block inline scripts and `eval()` |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Basic knowledge of JavaScript syntax and the DOM
- Awareness of how browsers parse HTML and build the DOM
- Familiarity with HTTP requests and the network waterfall (helpful)

---

### Related Programming Areas

- **DOM Manipulation** – JavaScript interacts with the DOM tree built by HTML
- **Web Performance** – Script loading strategy affects First Contentful Paint (FCP), Time to Interactive (TTI), and Largest Contentful Paint (LCP)
- **Content Security Policy (CSP)** – Script loading and inline restrictions are CSP-controlled
- **Progressive Enhancement** – HTML works without JavaScript; scripts enhance
- **Module Systems** – ES Modules, CommonJS, AMD, and bundlers (Webpack, Rollup, Vite)
- **Web Accessibility (A11y)** – Scripts must not break keyboard navigation or screen reader semantics

---

## Core Concepts / Features

---

### 1. The `<script>` Element

#### Definitions

**Core Definition**

The `<script>` element is the HTML element used to embed executable code (typically JavaScript) directly in a document or to reference an external script file.

**Technical Definition**

The `<script>` HTML element is used to embed executable code or data; this is typically used to embed or refer to JavaScript code. It is categorised as metadata content, flow content, and phrasing content. Its content model is dynamic: if the `src` attribute is absent, the content is script text (which must not contain `</script>`); if `src` is present, the content must be empty or whitespace. It supports `src`, `type`, `async`, `defer`, `crossorigin`, `integrity`, `nomodule`, `referrerpolicy`, `blocking`, and `fetchpriority`. Its DOM interface is `HTMLScriptElement`.

**Beginner-Friendly Explanation**

The `<script>` tag is how you put JavaScript into your HTML. You can either write the JavaScript directly between the opening and closing tags, or you can point to an external `.js` file using the `src` attribute. The browser reads the script and runs it.

#### Purposes

- To embed inline JavaScript code in an HTML document
- To reference an external JavaScript file
- To load third-party scripts (analytics, widgets, libraries)
- To define module scripts with `type="module"`
- To provide fallback content for browsers without JavaScript support

#### Syntax Rules and Structure

**Inline Script**

```html
<script>
    console.log('Hello, world!');
</script>
```

**External Script**

```html
<script src="/js/app.js"></script>
```

**Component Breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of an external script |
| `type` | MIME type or `module` (default: classic script) |
| `async` | Download in parallel, execute as soon as ready |
| `defer` | Download in parallel, execute after parsing |
| `crossorigin` | CORS setting for cross-origin scripts |
| `integrity` | Subresource Integrity hash |
| `nomodule` | Fallback for browsers without module support |

**Syntax Rules**

- The `<script>` element requires a closing tag
- Inline scripts must not contain the literal string `</script>` (escape as `<\/script>`)
- `async` and `defer` only apply to external scripts
- The `type` attribute is optional for classic scripts (`text/javascript` is the default)
- `type="module"` creates a module script with different semantics

**Constraints and Limitations**

- Classic scripts block HTML parsing unless `async` or `defer` is used
- Inline scripts are blocked by CSP unless `'unsafe-inline'` is allowed
- `document.write()` in scripts can break the parser and is strongly discouraged

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Inline Script**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Inline Script Demo</title>
</head>
<body>
    <h1 id="greeting">Hello</h1>

    <script>
        // This runs immediately when the parser reaches it
        const el = document.getElementById('greeting');
        el.textContent = 'Hello from inline JavaScript!';
    </script>
</body>
</html>
```

**Expected Output**

The heading text changes from "Hello" to "Hello from inline JavaScript!" as soon as the script runs.

**Why This Output Occurs**

The script is placed after the `<h1>` element, so the element exists in the DOM when the script runs. `document.getElementById()` finds it and updates its text content.

---

**Example 2: External Script**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>External Script Demo</title>
    <script src="/js/app.js" defer></script>
</head>
<body>
    <h1 id="greeting">Hello</h1>
</body>
</html>
```

**app.js**

```javascript
document.addEventListener('DOMContentLoaded', () => {
    document.getElementById('greeting').textContent = 'Hello from external JavaScript!';
});
```

**Expected Output**

The heading text changes to "Hello from external JavaScript!" after the DOM is parsed.

**Why This Output Occurs**

The `defer` attribute delays execution until after parsing. The `DOMContentLoaded` event ensures the DOM is fully available.

#### Real-World Cases

**Case 1: Analytics**

Google Analytics and similar services are loaded via external `<script>` tags.

**Case 2: Framework Bootstrapping**

React, Vue, and Angular applications load their bundles via `<script type="module">` or `<script defer>`.

**Case 3: Third-Party Widgets**

Chat widgets, payment forms, and social embeds are loaded via external scripts.

---

### 2. External JavaScript Files

#### Definitions

**Core Definition**

External JavaScript files are separate `.js` files linked to an HTML document via the `<script src="...">` attribute, separating application logic from structural content.

**Technical Definition**

When the `src` attribute is present on a `<script>` element, the browser fetches the resource at the specified URL and executes it as a script. External scripts are subject to CORS when cross-origin, can be protected by Subresource Integrity (`integrity`), and are cached by the browser according to HTTP cache headers. External scripts can be classic scripts or module scripts (via `type="module"`). The `crossorigin` attribute controls CORS mode: `anonymous` (no credentials) or `use-credentials`. External scripts are the recommended approach for production because they are cacheable, maintainable, and compatible with Content Security Policy.

**Beginner-Friendly Explanation**

Instead of writing all your JavaScript inside your HTML file, you write it in a separate `.js` file and link to it. For example: `<script src="app.js" defer></script>`. This keeps your HTML clean and lets the browser cache the JavaScript file, so it doesn't have to re-download it on every page.

#### Purposes

- To separate application logic from structural content
- To enable browser caching of JavaScript files
- To improve maintainability and team collaboration
- To support Content Security Policy without `'unsafe-inline'`
- To allow Subresource Integrity verification
- To share scripts across multiple pages

#### Syntax Rules and Structure

**General Syntax**

```html
<script src="path/to/script.js"></script>
<script src="path/to/script.js" defer></script>
<script src="path/to/script.js" async></script>
<script src="path/to/module.mjs" type="module"></script>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `src` | URL of the external script |
| `defer` | Execute after parsing, in order |
| `async` | Execute as soon as downloaded |
| `type` | `module` for ES modules |
| `crossorigin` | CORS mode (`anonymous` or `use-credentials`) |
| `integrity` | SRI hash for security |

**Syntax Rules**

- The `<script>` element with `src` must have no content (or only whitespace)
- `async` and `defer` only apply to external scripts
- `crossorigin` is required for `integrity` on cross-origin resources
- Module scripts require a server (they don't work with `file://` due to CORS)

**Constraints and Limitations**

- External scripts add an HTTP request (mitigated by caching)
- Cross-origin scripts require CORS headers if `crossorigin` is set
- Module scripts have stricter loading rules than classic scripts

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Deferred External Script**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>External Script Demo</title>

    <!-- Downloaded in parallel, executed after parsing -->
    <script src="/js/app.js" defer></script>
</head>
<body>
    <button id="btn">Click me</button>
</body>
</html>
```

**app.js**

```javascript
document.getElementById('btn').addEventListener('click', () => {
    alert('Button clicked!');
});
```

**Expected Output**

Clicking the button shows an alert.

**Why This Output Occurs**

The `defer` attribute delays execution until after parsing, so `#btn` exists when the script runs.

---

**Example 2: External Script from a CDN with SRI**

```html
<script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"
        integrity="sha384-abc123..."
        crossorigin="anonymous"
        defer></script>
```

**Expected Output**

The browser verifies the file's integrity before executing it. If the hash doesn't match, the script is blocked.

**Why This Output Occurs**

The `integrity` attribute provides a cryptographic hash. The `crossorigin="anonymous"` attribute enables CORS for the integrity check.

#### Real-World Cases

**Case 1: Framework Bundles**

React, Vue, and Angular build external bundles (e.g., `app.[hash].js`) and load them with `<script defer>`.

**Case 2: CDN Libraries**

jQuery, Lodash, and Bootstrap are loaded from CDNs with `integrity` and `crossorigin`.

**Case 3: Analytics**

Google Analytics, Plausible, and Fathom are loaded as external scripts, often with `async`.

---

### 3. Inline Scripts

#### Definitions

**Core Definition**

Inline scripts are JavaScript expressions written directly inside HTML attributes (e.g., `onclick="..."`) or between `<script>` tags without a `src` attribute.

**Technical Definition**

Inline scripts come in two forms: (1) inline `<script>` blocks (script text between `<script>` and `</script>` tags without a `src` attribute), and (2) inline event handlers (attributes like `onclick`, `onload`, `onchange` containing JavaScript). Inline event handlers are legacy and are discouraged by the WHATWG HTML Living Standard, MDN, and CSP best practices. They are blocked by Content Security Policy unless `'unsafe-inline'` or `'unsafe-hashes'` is allowed. Inline `<script>` blocks are also blocked by CSP unless `'unsafe-inline'` or nonces/hashes are used.

**Beginner-Friendly Explanation**

Inline scripts are JavaScript written directly in your HTML — either inside a `<script>` tag or in an attribute like `onclick="doSomething()"`. Inline event handlers are considered bad practice today because they mix content with behaviour, are hard to maintain, and are a security risk (XSS). Modern development uses external scripts and `addEventListener()` instead.

#### Purposes

- To run a small, one-off script without creating a separate file
- To prototype or test code quickly
- To provide small enhancement snippets (rarely justified)
- To bootstrap critical code before external scripts load (rare)

#### Syntax Rules and Structure

**Inline Event Handler (Discouraged)**

```html
<button onclick="alert('Clicked!')">Click me</button>
```

**Inline Script Block**

```html
<script>
    document.getElementById('btn').addEventListener('click', () => {
        alert('Clicked!');
    });
</script>
```

**Correct Alternative (External + addEventListener)**

```html
<button id="btn">Click me</button>
<script src="app.js" defer></script>
```

**app.js**

```javascript
document.getElementById('btn').addEventListener('click', () => {
    alert('Clicked!');
});
```

**Syntax Rules**

- Inline event handlers use `on` + event name (e.g., `onclick`, `onload`)
- Inline scripts must not contain `</script>` (escape as `<\/script>`)
- CSP blocks inline scripts unless `'unsafe-inline'`, nonces, or hashes are used
- Inline event handlers are blocked by CSP `'unsafe-hashes'` or `'unsafe-inline'`

**Constraints and Limitations**

- Inline event handlers mix HTML and JavaScript, violating separation of concerns
- They are a common XSS vector
- They are hard to test and maintain
- They are blocked by modern CSP configurations
- They do not support multiple listeners for the same event

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Inline Event Handler (Legacy — Discouraged)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Inline Handler (Legacy)</title>
</head>
<body>
    <button onclick="alert('Clicked!')">Click me</button>
</body>
</html>
```

**Expected Output**

Clicking the button shows an alert.

**Why This Output Occurs**

The `onclick` attribute contains JavaScript that runs when the button is clicked. This works but is discouraged.

---

**Example 2: Preferred Alternative**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>External Listener (Preferred)</title>
    <script src="app.js" defer></script>
</head>
<body>
    <button id="btn">Click me</button>
</body>
</html>
```

**app.js**

```javascript
document.getElementById('btn').addEventListener('click', () => {
    alert('Clicked!');
});
```

**Expected Output**

Clicking the button shows an alert.

**Why This Output Occurs**

The event listener is attached in JavaScript, keeping HTML clean and behaviour separate. This approach supports multiple listeners, is testable, and is CSP-friendly.

#### Real-World Cases

**Case 1: Legacy Codebases**

Older codebases often contain inline event handlers that need to be migrated to external scripts.

**Case 2: Quick Prototypes**

Developers use inline scripts during rapid prototyping, then refactor for production.

**Case 3: Email Templates**

HTML emails sometimes use inline scripts, but most clients block them for security.

---

### 4. The `defer` Attribute

#### Definitions

**Core Definition**

The `defer` attribute instructs the browser to download a script in parallel with HTML parsing and execute it only after parsing is complete, preserving the document order of deferred scripts.

**Technical Definition**

The `defer` attribute is a boolean attribute that applies only to classic external scripts. When present, the browser fetches the script in parallel with HTML parsing but does not execute it until the parser has finished parsing the document. Deferred scripts execute in the order they appear in the document, before the `DOMContentLoaded` event fires. The `defer` attribute has no effect on inline scripts or module scripts (module scripts are deferred by default). It is the recommended attribute for scripts that need to run after the DOM is available but must execute in a specific order.

**Beginner-Friendly Explanation**

`defer` tells the browser: "Go ahead and download this script while you're still reading the HTML, but don't run it until you've finished reading the whole page." This means your script can safely access any element in the DOM. And if you have multiple deferred scripts, they run in the order you wrote them.

#### Purposes

- To prevent scripts from blocking HTML parsing
- To ensure the DOM is fully available before the script runs
- To preserve the execution order of multiple scripts
- To improve perceived page load performance
- To eliminate the need for `DOMContentLoaded` wrappers

#### Syntax Rules and Structure

**General Syntax**

```html
<script src="app.js" defer></script>
<script src="library.js" defer></script>
<script src="main.js" defer></script>
```

**Execution Order**

| Script | Download | Execute |
|---|---|---|
| `app.js` | Parallel with parsing | After parsing, 1st |
| `library.js` | Parallel with parsing | After parsing, 2nd |
| `main.js` | Parallel with parsing | After parsing, 3rd |

**Syntax Rules**

- The `defer` attribute is boolean; no value required
- It only applies to external classic scripts (with `src`)
- Module scripts are deferred by default
- Deferred scripts execute in document order
- Deferred scripts execute before `DOMContentLoaded`

**Constraints and Limitations**

- Ignored for inline scripts
- Ignored for module scripts (they are always deferred)
- Cannot be used with `async` (if both are present, `async` wins)

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Deferred Scripts Preserve Order**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Defer Order Demo</title>
    <script src="first.js" defer></script>
    <script src="second.js" defer></script>
    <script src="third.js" defer></script>
</head>
<body>
    <h1 id="output">Waiting...</h1>
</body>
</html>
```

**first.js**

```javascript
document.getElementById('output').textContent = 'First';
```

**second.js**

```javascript
setTimeout(() => {
    document.getElementById('output').textContent += ' → Second';
}, 100);
```

**third.js**

```javascript
document.getElementById('output').textContent += ' → Third';
```

**Expected Output**

The heading text becomes "First → Third" immediately after parsing, then "First → Third → Second" after 100ms.

**Why This Output Occurs**

All three scripts are deferred, so they execute after parsing in document order. The `setTimeout` in `second.js` delays its update, but the script itself runs second.

---

**Example 2: Defer Without DOMContentLoaded**

```html
<head>
    <script src="app.js" defer></script>
</head>
<body>
    <button id="btn">Click</button>
</body>
</html>
```

**app.js**

```javascript
// No DOMContentLoaded wrapper needed — defer guarantees DOM is ready
document.getElementById('btn').addEventListener('click', () => {
    console.log('Clicked');
});
```

**Expected Output**

The button responds to clicks immediately.

**Why This Output Occurs**

`defer` ensures the script runs after parsing, so the DOM is fully available without needing `DOMContentLoaded`.

#### Real-World Cases

**Case 1: Framework Bundles**

React, Vue, and Angular applications use `<script defer>` for their main bundles.

**Case 2: Library + Application**

A page loads jQuery with `defer`, then `app.js` with `defer`, ensuring jQuery is available before `app.js` runs.

**Case 3: Progressive Enhancement**

Deferred scripts add interactivity to server-rendered HTML without blocking the initial render.

---

### 5. The `async` Attribute

#### Definitions

**Core Definition**

The `async` attribute instructs the browser to download a script in parallel with HTML parsing and execute it as soon as it finishes downloading, without waiting for parsing to complete and without preserving order.

**Technical Definition**

The `async` attribute is a boolean attribute that applies only to classic external scripts. When present, the browser fetches the script in parallel with HTML parsing and executes it as soon as it is available, potentially before parsing is complete. Async scripts do not block HTML parsing (download is parallel), but they do block DOM construction momentarily during execution. Multiple async scripts execute in download order, which is unpredictable. The `async` attribute is best for independent scripts that do not depend on the DOM or on other scripts (e.g., analytics, ads).

**Beginner-Friendly Explanation**

`async` tells the browser: "Download this script in the background, and run it as soon as it's ready — don't wait for the page to finish loading." This is great for independent scripts like analytics that don't need the DOM. But it's bad for scripts that depend on other scripts or on the DOM, because the timing is unpredictable.

#### Purposes

- To download scripts without blocking HTML parsing
- To execute independent scripts as early as possible
- To improve page load performance for non-critical scripts
- To load third-party scripts (analytics, ads) without delaying the page

#### Syntax Rules and Structure

**General Syntax**

```html
<script src="analytics.js" async></script>
<script src="ads.js" async></script>
```

**Execution Order**

| Script | Download | Execute |
|---|---|---|
| `analytics.js` | Parallel with parsing | As soon as downloaded |
| `ads.js` | Parallel with parsing | As soon as downloaded |

The order of execution depends on which downloads first.

**Syntax Rules**

- The `async` attribute is boolean; no value required
- It only applies to external classic scripts (with `src`)
- Async scripts execute as soon as downloaded
- Async scripts do not preserve order
- If both `async` and `defer` are present, `async` takes precedence

**Constraints and Limitations**

- Async scripts may execute before the DOM is fully parsed
- Async scripts may execute in any order
- Async scripts cannot rely on other scripts being loaded
- Module scripts with `async` also execute as soon as downloaded

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Async Scripts Execute Unpredictably**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Async Order Demo</title>
    <script src="slow.js" async></script>
    <script src="fast.js" async></script>
</head>
<body>
    <h1>Async Scripts</h1>
</body>
</html>
```

**slow.js** (simulated slow download)

```javascript
console.log('Slow script executed');
```

**fast.js** (simulated fast download)

```javascript
console.log('Fast script executed');
```

**Expected Output (Console)**

```
Fast script executed
Slow script executed
```

**Why This Output Occurs**

The browser downloads both scripts in parallel. The faster one (`fast.js`) executes first, regardless of its position in the HTML. This is the defining characteristic of `async`.

---

**Example 2: Async for Independent Analytics**

```html
<head>
    <!-- Analytics loads asynchronously; doesn't block page rendering -->
    <script async src="https://www.googletagmanager.com/gtag/js?id=GA_ID"></script>
</head>
<body>
    <h1>Welcome</h1>
</body>
</html>
```

**Expected Output**

The page renders immediately. The analytics script loads and executes whenever it finishes downloading.

**Why This Output Occurs**

`async` allows the analytics script to load in the background without delaying the page. Since analytics doesn't depend on the DOM, the unpredictable execution timing is acceptable.

#### Real-World Cases

**Case 1: Google Analytics**

Analytics scripts use `async` because they don't depend on the DOM and shouldn't delay the page.

**Case 2: Ad Networks**

Ad scripts use `async` to load in parallel without blocking content.

**Case 3: Social Embeds**

Twitter, Facebook, and LinkedIn embed scripts use `async`.

---

### 6. Module Scripts (`type="module"`)

#### Definitions

**Core Definition**

Module scripts are scripts loaded with `type="module"` that use ES Module syntax (`import`/`export`), execute in strict mode, are automatically deferred, and have their own scope.

**Technical Definition**

A `<script type="module">` element creates a module script. Module scripts are defined by the ECMAScript Modules specification and the WHATWG HTML Living Standard. They differ from classic scripts in several ways: (1) they are always in strict mode; (2) they are deferred by default (executed after parsing); (3) they have their own top-level scope (variables are not global); (4) they support `import` and `export` statements; (5) they are subject to CORS when cross-origin; (6) they execute only once per URL (module map). Modules are the standard for modern JavaScript development and are supported by all modern browsers.

**Beginner-Friendly Explanation**

A module script is a more modern way to write JavaScript. Instead of putting all your code in one giant file, you split it into modules, and each module can `export` things that other modules `import`. Module scripts run in a special mode that catches more errors (strict mode), and they automatically wait until the HTML is parsed before running. They're the standard way to build JavaScript applications today.

#### Purposes

- To use ES Module syntax (`import`/`export`) in the browser
- To scope variables to the module, avoiding global namespace pollution
- To enable code splitting and modular architecture
- To enforce strict mode automatically
- To defer execution without needing `defer`

#### Syntax Rules and Structure

**General Syntax**

```html
<script type="module" src="main.mjs"></script>
<script type="module">
    import { greet } from './greet.mjs';
    greet('World');
</script>
```

**Module File**

```javascript
// greet.mjs
export function greet(name) {
    console.log(`Hello, ${name}!`);
}
```

**Component Breakdown**

| Feature | Description |
|---|---|
| `type="module"` | Declares a module script |
| `import` | Imports bindings from another module |
| `export` | Exports bindings for other modules |
| Strict mode | Automatic |
| Deferred | Automatic |
| Scope | Module-scoped, not global |

**Syntax Rules**

- Module scripts require a server (CORS blocks `file://`)
- The `import` path must be a valid URL or relative path with extension
- Module scripts are deferred by default
- `async` can be added to modules for immediate execution
- Modules are strict mode by default
- Each module URL is loaded only once (module map)

**Constraints and Limitations**

- Not supported in Internet Explorer
- Require a server (no `file://` support due to CORS)
- Bare specifiers (e.g., `import 'lodash'`) require an import map or bundler
- Cross-origin modules require CORS headers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Module**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Module Demo</title>
</head>
<body>
    <h1 id="output">Loading...</h1>

    <script type="module">
        // Strict mode is automatic; execution is deferred
        const output = document.getElementById('output');
        output.textContent = 'Module script executed!';
    </script>
</body>
</html>
```

**Expected Output**

The heading text changes to "Module script executed!" after parsing.

**Why This Output Occurs**

The module script is deferred by default, so it runs after parsing. The DOM is available, and strict mode is enforced.

---

**Example 2: Multi-File Module Architecture**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Multi-File Modules</title>
</head>
<body>
    <h1 id="output">Loading...</h1>
    <script type="module" src="./js/main.mjs"></script>
</body>
</html>
```

**js/greet.mjs**

```javascript
export function greet(name) {
    return `Hello, ${name}!`;
}
```

**js/main.mjs**

```javascript
import { greet } from './greet.mjs';

document.getElementById('output').textContent = greet('World');
```

**Expected Output**

The heading displays "Hello, World!"

**Why This Output Occurs**

`main.mjs` imports `greet` from `greet.mjs`. The browser resolves the module graph, loads the dependencies, and executes the entry module after parsing.

---

**Example 3: Async Module**

```html
<script type="module" async src="./js/analytics.mjs"></script>
```

**Expected Output**

The module downloads and executes as soon as it's ready, without waiting for parsing.

**Why This Output Occurs**

Adding `async` to a module overrides the default deferral, making it execute as soon as downloaded.

#### Real-World Cases

**Case 1: Modern Web Applications**

React, Vue, and Angular applications use ES modules with bundlers like Vite or Webpack.

**Case 2: NPM Packages on CDNs**

esm.sh, Skypack, and jsDelivr serve NPM packages as ES modules for direct browser import.

**Case 3: Progressive Enhancement**

Modules allow progressive enhancement without global namespace pollution.

---

### 7. Script Placement

#### Definitions

**Core Definition**

Script placement is the strategic positioning of `<script>` elements in an HTML document to control when scripts block rendering and how they interact with the DOM.

**Technical Definition**

The placement of `<script>` elements affects the browser's parsing and rendering pipeline. A classic script in `<head>` without `async` or `defer` blocks HTML parsing: the browser stops parsing, downloads the script, executes it, then resumes parsing. This delays the first render and can cause a "flash of unstyled content" (FOUC) or a blank page. Placing scripts at the end of `<body>` (just before `</body>`) allows the browser to parse and render the HTML first, then download and execute the scripts. With `defer`, scripts can be placed in `<head>` without blocking, gaining the performance benefit of early parallel download while executing after parsing. With `async`, placement is less critical because execution is independent of parsing order.

**Beginner-Friendly Explanation**

Where you put your `<script>` tag matters. If you put it in the `<head>` without `defer` or `async`, the browser stops everything to download and run the script — your page appears blank or delayed. If you put it at the bottom of the `<body>`, the page renders first, then the script runs. Or you can put it in the `<head>` with `defer`, which downloads it early but runs it after the page is ready. Modern best practice is `<script defer>` in `<head>` or `<script type="module">` in `<head>`.

#### Purposes

- To control when scripts block rendering
- To ensure the DOM is available before scripts run
- To optimise the critical rendering path
- To avoid a blank page during script loading
- To improve First Contentful Paint (FCP) and Time to Interactive (TTI)

#### Syntax Rules and Structure

**Placement Comparison**

| Placement | Parsing | Rendering | Best For |
|---|---|---|---|
| `<head>` (no attributes) | Blocked | Blocked | Rarely (critical bootstrapping) |
| `<head>` + `defer` | Not blocked | Not blocked | Most scripts |
| `<head>` + `async` | Not blocked | Not blocked | Independent scripts |
| Before `</body>` | Not blocked | Not blocked | Legacy pattern (still valid) |
| `<head>` + `type="module"` | Not blocked | Not blocked | Modern applications |

**Syntax Rules**

- Classic scripts block parsing unless `async` or `defer` is present
- Deferred scripts can be placed in `<head>` without blocking
- Module scripts are deferred by default, so `<head>` placement is safe
- Placing scripts before `</body>` is a legacy pattern that still works
- Inline scripts block parsing and should be placed at the end of `<body>`

**Constraints and Limitations**

- Scripts in `<head>` without `async`/`defer` delay rendering
- Scripts before `</body>` still block parsing at that point (but the page is already rendered)
- Inline scripts cannot use `defer` or `async`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Blocking Script in `<head>` (Bad)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Blocking Script</title>
    <!-- Blocks parsing: page appears blank until download + execution -->
    <script src="heavy-library.js"></script>
</head>
<body>
    <h1>Hello</h1>
</body>
</html>
```

**Expected Output**

The page appears blank until `heavy-library.js` is downloaded and executed. Then the HTML is parsed and rendered.

**Why This Output Occurs**

Classic scripts in `<head>` block parsing. The browser stops, downloads, and executes the script before continuing with the HTML.

---

**Example 2: Deferred Script in `<head>` (Best Practice)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Deferred Script</title>
    <!-- Download in parallel; execute after parsing -->
    <script src="app.js" defer></script>
</head>
<body>
    <h1>Hello</h1>
</body>
</html>
```

**Expected Output**

The page renders immediately. The script runs after parsing.

**Why This Output Occurs**

The `defer` attribute allows the script to download in parallel without blocking parsing, and execute after parsing.

---

**Example 3: Script Before `</body>` (Legacy Pattern)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>End-of-Body Script</title>
</head>
<body>
    <h1>Hello</h1>

    <!-- HTML is already parsed and rendered; script executes immediately -->
    <script src="app.js"></script>
</body>
</html>
```

**Expected Output**

The page renders first, then the script runs.

**Why This Output Occurs**

By the time the parser reaches the script at the end of `<body>`, the DOM is fully built. The script can safely access any element.

#### Real-World Cases

**Case 1: WordPress**

WordPress enqueues scripts in `<head>` with `defer` or at the end of `<body>` depending on the theme.

**Case 2: Shopify**

Shopify themes load scripts at the end of `<body>` for legacy performance reasons.

**Case 3: Modern SPAs**

Single-page applications use `<script type="module" src="main.js">` in `<head>` (deferred by default).

---

### 8. Choosing the Right Integration Approach

#### Definitions

**Core Definition**

Choosing the right JavaScript integration approach means selecting the appropriate `<script>` attributes, placement, and module strategy based on the script's dependencies, criticality, and performance impact.

**Technical Definition**

The choice depends on whether the script: (1) depends on the DOM, (2) depends on other scripts, (3) is critical for rendering, (4) is independent, and (5) uses modules. The decision matrix recommends `defer` for scripts that depend on the DOM and on other scripts; `async` for independent scripts; `type="module"` for modern modular applications; and inline scripts only when absolutely necessary.

**Beginner-Friendly Explanation**

There's a right tool for every job. If your script needs the DOM and depends on other scripts, use `defer`. If it's independent (like analytics), use `async`. If you're building a modern app with imports and exports, use `type="module"`. Avoid inline scripts and avoid blocking scripts in `<head>`.

#### Decision Guide

| Script Type | Recommended Attributes | Placement |
|---|---|---|
| **Main application script** | `defer` or `type="module"` | `<head>` |
| **Library depended on by others** | `defer` | `<head>` (before dependents) |
| **Analytics / ads** | `async` | `<head>` |
| **Critical bootstrapping** | inline (rare) | end of `<body>` |
| **Modern modular app** | `type="module"` | `<head>` |
| **Legacy jQuery script** | `defer` | `<head>` |
| **Polyfill** | `nomodule` | `<head>` |

---

## References

- MDN Web Docs – `<script>`: The Script element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script
- MDN Web Docs – Script loading strategies – https://developer.mozilla.org/en-US/docs/Learn/Performance/JavaScript
- MDN Web Docs – `defer` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script#defer
- MDN Web Docs – `async` attribute – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script#async
- MDN Web Docs – JavaScript modules – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- MDN Web Docs – ES modules in browsers – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#modules_in_browsers
- WHATWG HTML Living Standard – The script element – https://html.spec.whatwg.org/multipage/scripting.html#the-script-element
- WHATWG HTML Living Standard – Scripting – https://html.spec.whatwg.org/multipage/scripting.html
- web.dev – Efficiently load JavaScript with defer and async – https://web.dev/articles/efficiently-load-third-party-javascript
- web.dev – JavaScript performance – https://web.dev/learn/performance/javascript
- web.dev – ES modules – https://web.dev/articles/module-workers
- Chrome for Developers – Content Security Policy – https://developer.chrome.com/docs/lighthouse/best-practices/csp-xss
- W3C – Subresource Integrity – https://www.w3.org/TR/SRI/
- ECMAScript – Modules – https://tc39.es/ecma262/multipage/ecmascript-language-scripts-and-modules.html
- CSS-Tricks – Preload, Prefetch And Priorities in Chrome – https://css-tricks.com/preload-prefetch-and-priorities-in-chrome/