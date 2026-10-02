# Express.js Security in Template Rendering — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Security in template rendering is the practice of ensuring that dynamic data inserted into server-side templates is handled safely, preventing cross-site scripting (XSS), template injection, and prototype pollution attacks from compromising the application or its users.

**Technical Definition:** Template rendering security encompasses four interlocking controls: (1) auto-escaping mechanisms that encode output based on its destination context, (2) contextual sanitisation that applies the correct encoding for HTML bodies, attributes, JavaScript, and URL contexts, (3) Content Security Policy (CSP) integration with per-request cryptographic nonces to restrict inline script execution, and (4) prototype pollution defence to prevent malicious object mutations from reaching template engine compilation and runtime logic. These controls operate at the boundary between application data and browser-executable code.

**Beginner-Friendly Explanation:** When you build a web page on the server, you're combining a template (the fixed HTML structure) with data (user names, comments, prices). If that data contains malicious code — like a `<script>` tag — and you insert it into the page without checking, the browser will run it as if you wrote it yourself. Template rendering security is the set of rules and tools that ensure user data is always treated as plain text, never as executable code, no matter where it appears in the page.

### Key Characteristics

- **Context-dependent:** The correct encoding depends entirely on where the data appears — HTML body, HTML attribute, JavaScript string, CSS, or URL.
- **Engine-specific:** EJS uses `<%= %>` for escaped output and `<%- %>` for raw; Pug uses `=` and `!=`; Handlebars escapes `{{ }}` by default and uses `{{{ }}}` for raw.
- **Defence-in-depth:** Auto-escaping alone is insufficient; CSP, sanitisation, and prototype pollution prevention are complementary layers.
- **Vulnerability-prone when misused:** The difference between `<%=` and `<%-` is a single character; a mistype can open an XSS vulnerability.
- **Version-sensitive:** Template engines have a history of prototype pollution and template injection CVEs; version discipline is non-negotiable.

### Prerequisites

- **Node.js runtime** (v18 or higher recommended).
- **Express.js with a template engine** (EJS, Pug, or Handlebars).
- **Basic understanding of XSS:** How malicious scripts execute in browsers.
- **Familiarity with HTML contexts:** Body, attributes, script tags, URLs.
- **Understanding of JavaScript prototypes:** For prototype pollution defence.

### Related Programming Areas

- **Content Security Policy (CSP):** Restricting script execution to trusted sources with nonces.
- **Input validation and sanitisation:** Cleaning data before it reaches the template.
- **Output encoding libraries:** DOMPurify, escape-html, and context-specific encoders.
- **Dependency management:** Pinning template engine versions to avoid known CVEs.
- **Server-side template injection (SSTI):** Preventing user input from becoming template source code.

### Core Concepts

1. **Auto-Escaping & XSS Prevention** — safe outputs vs. raw HTML execution.
2. **Contextual Sanitisation** — encoding for HTML bodies, attributes, and script tags.
3. **Content Security Policy (CSP) Integration** — injecting dynamic nonces into `<script>` blocks.
4. **Preventing Prototype Pollution in Views** — defending against malicious object mutations.

---

## Core Concept 1: Auto-Escaping & XSS Prevention

### Definitions

**Core Definition:** Auto-escaping is a template engine feature that automatically encodes dynamic values before inserting them into HTML, neutralising characters that could be interpreted as markup or script.

**Technical Definition:** In EJS, `<%= variable %>` outputs the value with HTML entity encoding (converting `<` to `&lt;`, `>` to `&gt;`, `&` to `&amp;`, `"` to `&quot;`, and `'` to `&#39;`), while `<%- variable %>` outputs the raw, unescaped value. In Pug, `= variable` escapes and `!= variable` does not. In Handlebars, `{{ variable }}` escapes and `{{{ variable }}}` does not. Auto-escaping is the primary defence against reflected and stored XSS, but it does not protect against context-specific injection (e.g., unquoted attributes or JavaScript contexts).

**Beginner-Friendly Explanation:** Auto-escaping is like a translator who rewrites dangerous words so they can't be misunderstood as commands. If someone writes `<script>alert(1)</script>` in a comment, the template engine rewrites it to `&lt;script&gt;alert(1)&lt;/script&gt;`, which the browser displays as text instead of executing.

### Purposes

- To prevent reflected and stored XSS by neutralising HTML special characters in dynamic output.
- To provide a safe default that protects developers who might otherwise forget to encode.
- To establish a clear distinction between safe output (escaped) and deliberately unsafe output (raw HTML).
- To serve as the first layer of defence, complemented by CSP and contextual sanitisation.

### Sub-Feature 1.1: `<%= %>` vs `<%- %>` in EJS

#### Syntax Rules and Structure

| Tag | Output | Safety |
|-----|--------|--------|
| `<%= variable %>` | HTML-escaped string | Safe for HTML body and quoted attributes |
| `<%- variable %>` | Raw, unescaped string | **Dangerous with user input** |

**Constraints and Limitations:**
- `<%- %>` is an XSS sink when the expression contains user-controlled data; static analysis tools like Semgrep flag it as a vulnerability.
- HTML escaping does **not** protect against injection in unquoted attributes, JavaScript contexts, or URL contexts.
- The characters `<` and `=` are adjacent on the keyboard, increasing the risk of accidental mistyping.

#### Annotated Code Example

```ejs
<!-- views/comment.ejs -->

<!-- ✅ SAFE: Escaped output — HTML characters are neutralised -->
<p>Comment: <%= userComment %></p>

<!-- ❌ DANGEROUS: Raw output — scripts execute -->
<!-- <p>Comment: <%- userComment %></p> -->

<!-- ✅ SAFE: Escaped output inside a quoted attribute -->
<input type="text" value="<%= userComment %>">

<!-- ❌ DANGEROUS: Unquoted attribute — escaping does not help -->
<!-- <input type="text" value=<%= userComment %>> -->
```

**Expected Output (for `userComment = '<script>alert(1)</script>'` with `<%= %>`):**
```html
<p>Comment: &lt;script&gt;alert(1)&lt;/script&gt;</p>
```

**Expected Output (with `<%- %>`):**
```html
<p>Comment: <script>alert(1)</script></p>
```

**Why this output:** `<%= %>` converts the angle brackets to HTML entities, so the browser renders the text literally. `<%- %>` inserts the string verbatim, and the browser executes the script. In the unquoted attribute case, even escaped output is insufficient because the attacker can inject an event handler like `onmouseover=alert(1)` without using any escaped characters.

#### Real-World Cases

- **Comment sections:** A stored XSS vulnerability in a comment system can affect every visitor who views the comment.
- **User profiles:** Displaying a user's "bio" or "display name" with `<%- %>` allows script injection.
- **Search results:** Reflecting a search query with `<%- %>` enables reflected XSS via crafted links.

---

### Sub-Feature 1.2: Preventing DOM-Based Injection

#### Definitions

**Core Definition:** DOM-based XSS occurs entirely in the browser, where client-side JavaScript reads untrusted data from the DOM (URL, `location.hash`, `document.referrer`) and writes it to a sink (e.g., `innerHTML`) without encoding.

**Technical Definition:** The OWASP DOM-based XSS Prevention Cheat Sheet identifies safe and unsafe sinks and sources. Safe sinks include `textContent`, `setAttribute` (for non-event attributes), and `createElement`. Unsafe sinks include `innerHTML`, `outerHTML`, `document.write`, and `eval`. The appropriate encoding for DOM-based XSS depends on the execution context, not the rendering context.

**Beginner-Friendly Explanation:** DOM-based XSS is like a recipe that tells the browser to read a note from the fridge (the URL) and follow its instructions. If the note says "set the kitchen on fire," the browser does it. Preventing DOM-based XSS means never following instructions from untrusted sources — always treat them as text to display.

#### Purposes

- To prevent client-side JavaScript from executing untrusted data as code.
- To enforce the use of safe DOM APIs (`textContent`, `createElement`) over unsafe ones (`innerHTML`, `document.write`).
- To complement server-side escaping with client-side discipline.

#### Syntax Rules and Structure

```js
// ❌ UNSAFE: innerHTML with untrusted data
element.innerHTML = userInput;

// ✅ SAFE: textContent renders as text
element.textContent = userInput;

// ✅ SAFE: createElement + textContent
const div = document.createElement('div');
div.textContent = userInput;
container.appendChild(div);
```

| Sink | Safety | Alternative |
|------|--------|-------------|
| `innerHTML` | Unsafe | `textContent` |
| `outerHTML` | Unsafe | `replaceWith` + `textContent` |
| `document.write` | Unsafe | `insertAdjacentText` |
| `eval` | Unsafe | `JSON.parse` |

**Constraints and Limitations:**
- `textContent` is safe but loses HTML formatting; if rich text is required, use DOMPurify.
- The OWASP rule states: "The general rule is to HTML Attribute encode untrusted data placed in an HTML Attribute. This is the appropriate step to take when outputting data in a rendering context, however using HTML Attribute encoding in an execution context will break the application display of data."

#### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<body>
  <div id="output"></div>
  <script>
    // ❌ VULNERABLE: innerHTML executes scripts
    // document.getElementById('output').innerHTML =
    //   new URLSearchParams(location.search).get('name');

    // ✅ SAFE: textContent renders as text
    const userInput = new URLSearchParams(location.search).get('name') || '';
    document.getElementById('output').textContent = userInput;
  </script>
</body>
</html>
```

**Expected Output (for URL `?name=<img src=x onerror=alert(1)>`):**
```
The page displays the literal text: <img src=x onerror=alert(1)>
No alert is triggered.
```

**Why this output:** `textContent` treats the entire string as text. The `<img>` tag is not parsed as HTML, so the `onerror` event never fires.

#### Real-World Cases

- **Single-page applications:** Client-side routers that read URL fragments and inject them into the DOM.
- **Search pages:** Client-side search that writes the query to the page without encoding.
- **Comment previews:** Live preview of user input using `innerHTML`.

---

## Core Concept 2: Contextual Sanitisation

### Definitions

**Core Definition:** Contextual sanitisation is the process of applying the correct encoding or sanitisation technique based on the specific location in the HTML document where untrusted data is inserted.

**Technical Definition:** OWASP identifies four primary output encoding contexts: **HTML Entity Encoding** (for data between HTML tags), **HTML Attribute Encoding** (for data inside attribute values), **JavaScript Encoding** (for data inside `<script>` blocks), and **URL Encoding** (for data in URL parameters). Each context requires a different encoding scheme because different characters are dangerous in different positions. Layered encoding is required for nested contexts (e.g., JavaScript inside an HTML attribute).

**Beginner-Friendly Explanation:** If you're writing a message on a whiteboard, you use a marker. If you're writing on a glass window, you use a different type of pen. Contextual sanitisation means using the right "pen" for the right surface — HTML encoding for HTML, JavaScript encoding for JavaScript, and URL encoding for URLs.

### Purposes

- To prevent XSS in contexts where auto-escaping alone is insufficient (attributes, JavaScript, URLs).
- To ensure that data is rendered correctly regardless of its destination context.
- To handle nested contexts (JavaScript inside an HTML attribute) with layered encoding.
- To complement auto-escaping with context-aware manual encoding where needed.

### Sub-Feature 2.1: HTML Body and Attribute Contexts

#### Definitions

**Core Definition:** HTML body context refers to data inserted between HTML tags, while HTML attribute context refers to data inserted inside an attribute value.

**Technical Definition:** In HTML body context, the characters `<`, `>`, `&`, `"`, and `'` must be HTML-entity encoded. In HTML attribute context, the same characters must be encoded, but the attribute value must also be quoted. The OWASP rule states: "For attributes, HTML Attribute encode untrusted data (data from the database, HTTP request, user, back-end system, etc.) placed in an HTML Attribute." Unquoted attributes are particularly dangerous because an attacker can inject an event handler (e.g., `onmouseover=alert(1)`) without using any characters that would be escaped.

**Beginner-Friendly Explanation:** In an HTML body, you encode `<` and `>` so the browser doesn't interpret them as tags. In an attribute, you encode the same characters AND always use quotes around the value. Without quotes, the attacker can add their own attributes.

#### Syntax Rules and Structure

```ejs
<!-- HTML body context: escape < > & " ' -->
<p><%= userContent %></p>

<!-- HTML attribute context: escape AND quote -->
<input type="text" value="<%= userContent %>">

<!-- ❌ NEVER: unquoted attribute -->
<!-- <input type="text" value=<%= userContent %>> -->
```

| Context | Encoding | Quoting |
|---------|----------|---------|
| HTML body | HTML entity | Not applicable |
| HTML attribute | HTML attribute | **Required** |

**Constraints and Limitations:**
- HTML escaping does not neutralise event handlers in unquoted attributes.
- The OWASP DOM-based XSS Cheat Sheet notes that "using HTML Attribute encoding in an execution context will break the application display of data."

#### Annotated Code Example

```ejs
<!-- views/form.ejs -->

<!-- ✅ SAFE: Quoted attribute with escaped output -->
<input type="text" name="username" value="<%= username %>">

<!-- ✅ SAFE: HTML body context -->
<p>Welcome, <%= username %></p>

<!-- ❌ DANGEROUS: Unquoted attribute -->
<!-- <input type="text" value=<%= username %>> -->
```

**Expected Output (for `username = 'Alice" onmouseover="alert(1)'`):**
```html
<input type="text" name="username" value="Alice&quot; onmouseover=&quot;alert(1)">
```

**Why this output:** The `"` character is converted to `&quot;`, so the attribute value remains intact. The `onmouseover` text is treated as part of the attribute value, not as a separate event handler. Without quoting, the `"` would close the attribute and allow the attacker to add `onmouseover` as a real event handler.

#### Real-World Cases

- **Login forms:** Pre-filling the username field with a reflected value.
- **Search inputs:** Preserving the search query in the input field.
- **Comment edit forms:** Loading the existing comment into a textarea.

---

### Sub-Feature 2.2: JavaScript and URL Contexts

#### Definitions

**Core Definition:** JavaScript context refers to data inserted inside a `<script>` block or event handler, while URL context refers to data inserted into `href`, `src`, or `action` attributes.

**Technical Definition:** In JavaScript context, data must be JavaScript-encoded (escaping characters that can break out of string literals or introduce new statements). In URL context, data must be URL-encoded (percent-encoding all characters except unreserved characters). The OWASP guidance states: "The appropriate encoding to use in the above case would be only JavaScript encoding to disallow an attacker from closing out the single quotes and in-lining code, or escaping to HTML and opening a new script tag."

**Beginner-Friendly Explanation:** In a JavaScript block, you can't just HTML-encode because the browser parses JavaScript differently. You need to escape characters that would end the string or start a new command. In a URL, you need to encode characters that would change the URL's meaning (like `&` and `?`).

#### Syntax Rules and Structure

```ejs
<!-- ❌ DANGEROUS: User data in JavaScript context without JS encoding -->
<script>
  var username = "<%= username %>";
</script>

<!-- ✅ SAFE: JavaScript-encoded (escape quotes, backslashes, newlines) -->
<script>
  var username = "<%= JSON.stringify(username).slice(1, -1) %>";
</script>

<!-- ❌ DANGEROUS: User data in href without URL encoding -->
<a href="/user/<%= userId %>">Profile</a>

<!-- ✅ SAFE: URL-encoded -->
<a href="/user/<%= encodeURIComponent(userId) %>">Profile</a>
```

| Context | Encoding | Example |
|---------|----------|---------|
| JavaScript string | JavaScript escape | `JSON.stringify` |
| URL parameter | Percent encoding | `encodeURIComponent` |
| HTML attribute (URL) | URL encoding + quoting | `encodeURIComponent` + quotes |

**Constraints and Limitations:**
- HTML escaping does not protect JavaScript contexts; `<%= %>` is insufficient inside `<script>`.
- Layered encoding is required for nested contexts: JavaScript encode **then** HTML attribute encode.
- The OWASP example shows: `Encoder.encodeForHtml(Encoder.encodeForJavaScript(request.getParameter("error")))` for JavaScript inside an HTML attribute.

#### Annotated Code Example

```ejs
<!-- views/profile.ejs -->

<!-- ✅ SAFE: URL-encoded user ID in href -->
<a href="/user/<%= encodeURIComponent(userId) %>">View Profile</a>

<!-- ❌ DANGEROUS: Raw user data in href (javascript: URL injection) -->
<!-- <a href="<%= userUrl %>">Click</a> -->

<!-- ✅ SAFE: JavaScript-encoded data in script block -->
<script>
  var config = {
    userId: <%- JSON.stringify(userId) %>,
    theme: <%- JSON.stringify(theme) %>
  };
</script>
```

**Expected Output (for `userId = 'abc/../../admin'`):**
```html
<a href="/user/abc%2F..%2F..%2Fadmin">View Profile</a>
```

**Expected Output (for `userId = '123'` and `theme = 'dark'`):**
```html
<script>
  var config = {
    userId: "123",
    theme: "dark"
  };
</script>
```

**Why this output:** `encodeURIComponent` encodes the `/` characters as `%2F`, preventing path traversal. `JSON.stringify` wraps the values in quotes and escapes any special characters, producing valid JavaScript literals. Using `<%- %>` with `JSON.stringify` is safe because `JSON.stringify` already produces a properly escaped string.

#### Real-World Cases

- **User profile URLs:** Encoding user IDs in `href` attributes to prevent path traversal.
- **Configuration objects:** Injecting server-side data into client-side JavaScript configuration.
- **Redirect URLs:** Encoding redirect targets to prevent open redirect attacks.

---

## Core Concept 3: Content Security Policy (CSP) Integration

### Definitions

**Core Definition:** Content Security Policy (CSP) is a response header that instructs the browser on which sources of scripts, styles, and other resources are permitted to load, with nonces providing a mechanism to allow specific inline scripts.

**Technical Definition:** A CSP nonce is a cryptographically random value generated per request and included in both the `Content-Security-Policy` header (`script-src 'nonce-RANDOM'`) and the `nonce` attribute of permitted `<script>` tags. The browser executes only scripts whose `nonce` attribute matches the header value. In Express, the nonce is generated in middleware, stored in `res.locals`, and injected into templates via template variables. Helmet's `contentSecurityPolicy` middleware supports nonce-based policies.

**Beginner-Friendly Explanation:** A CSP nonce is like a one-time password for scripts. The server generates a random code, puts it in the page's security policy header, and stamps it on the scripts it trusts. The browser checks the stamp before running any script. Scripts without the correct stamp — including ones an attacker injected — are blocked.

### Purposes

- To allow legitimate inline scripts while blocking injected scripts.
- To provide a strong second layer of defence against XSS when output encoding fails.
- To enable strict CSP policies (`script-src 'nonce-...' 'strict-dynamic'`) that do not rely on allowlists.
- To support modern single-page applications that require inline hydration scripts.

### Sub-Feature 3.1: Generating and Injecting Nonces

#### Syntax Rules and Structure

```js
// Middleware: generate nonce per request
const crypto = require('node:crypto');

app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(32).toString('base64');
  next();
});

// Helmet CSP with nonce
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`]
    }
  }
}));
```

```ejs
<!-- Template: inject nonce into script tag -->
<script nonce="<%= cspNonce %>">
  // Inline script allowed by CSP
</script>
```

| Component | Breakdown |
|-----------|-----------|
| `crypto.randomBytes(32)` | Cryptographically secure random nonce. |
| `res.locals.cspNonce` | Request-scoped nonce available in templates. |
| `'nonce-...'` | CSP directive referencing the nonce. |
| `nonce="<%= cspNonce %>"` | Template injection of the nonce. |

**Constraints and Limitations:**
- The nonce must be unique per request and unpredictable.
- The nonce must be base64-encoded and without characters that would break HTML attributes.
- Helmet must be configured to use the nonce; the default CSP does not include it.
- `'unsafe-inline'` is ignored when a nonce or hash is present in the same directive.

#### Annotated Code Example

```js
// app.js — CSP nonce with Helmet and EJS
const express = require('express');
const helmet = require('helmet');
const crypto = require('node:crypto');
const app = express();

app.set('view engine', 'ejs');

// ✅ Generate nonce per request
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(32).toString('base64');
  next();
});

// ✅ Configure Helmet CSP with nonce
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        (req, res) => `'nonce-${res.locals.cspNonce}'`
      ],
      styleSrc: ["'self'", "'unsafe-inline'"],
      objectSrc: ["'none'"]
    }
  }
}));

app.get('/', (req, res) => {
  res.render('index', { title: 'CSP Demo' });
});

app.listen(3000, () => console.log('CSP server on 3000'));
```

```ejs
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1><%= title %></h1>

  <!-- ✅ Allowed: nonce matches CSP header -->
  <script nonce="<%= cspNonce %>">
    console.log('This inline script is allowed.');
  </script>

  <!-- ❌ Blocked: no nonce -->
  <script>
    console.log('This script is blocked by CSP.');
  </script>
</body>
</html>
```

**Expected Response Header:**
```
Content-Security-Policy: default-src 'self';script-src 'self' 'nonce-a1b2c3d4...';style-src 'self' 'unsafe-inline';object-src 'none'
```

**Expected Output (browser console):**
```
This inline script is allowed.
(Refused to execute inline script because it violates CSP)
```

**Why this output:** The nonce in the `script-src` directive matches the `nonce` attribute on the first script tag, so the browser executes it. The second script tag has no nonce attribute, so the browser blocks it. If an attacker injects a script tag without the correct nonce, it will also be blocked.

#### Real-World Cases

- **Single-page applications:** Nonces allow the hydration script to execute while blocking injected scripts.
- **Payment pages:** Strict CSP with nonces protects against card-skimming attacks.
- **Admin dashboards:** Inline scripts for charts and widgets are allowed via nonce.

---

## Core Concept 4: Preventing Prototype Pollution in Views

### Definitions

**Core Definition:** Prototype pollution in views occurs when an attacker manipulates JavaScript's `Object.prototype` through malicious input, causing polluted properties to influence template engine compilation, option resolution, or runtime rendering — potentially leading to remote code execution (RCE).

**Technical Definition:** Template engines resolve configuration options and partial references through property lookups on plain objects. If `Object.prototype` has been polluted with a property whose key matches a template engine option (e.g., `outputFunctionName` in EJS, or a partial name in Handlebars), the polluted value is used instead of the intended default. This is the mechanism behind CVE-2022-29078 (EJS RCE) and CVE-2026-33916 (Handlebars XSS via partial template injection). Defence requires freezing `Object.prototype`, validating input types before merging, and never spreading untrusted objects into render options.

**Beginner-Friendly Explanation:** JavaScript objects inherit properties from a shared prototype. If an attacker can add a property to that shared prototype — like `outputFunctionName` — then every object in the application suddenly has that property. Template engines read their configuration from objects, so a polluted prototype can trick the engine into executing attacker-controlled code. Preventing prototype pollution means locking the shared prototype so it can't be modified.

### Purposes

- To prevent attackers from injecting properties into `Object.prototype` through malicious input.
- To protect template engine configuration and partial resolution from prototype chain traversal.
- To break the attack chain that links prototype pollution to RCE via template gadgets.
- To ensure that template rendering options are resolved from safe, validated sources.

### Sub-Feature 4.1: Known Vulnerabilities and Defences

#### Definitions

**Core Definition:** Known template engine prototype pollution vulnerabilities include CVE-2022-29078 (EJS RCE via `outputFunctionName`) and CVE-2026-33916 (Handlebars XSS via partial template injection).

**Technical Definition:** CVE-2022-29078 allows an attacker to overwrite EJS's `outputFunctionName` option with an arbitrary OS command, which executes when the template is compiled. Exploitation requires the application to already be vulnerable to prototype pollution, allowing attacker-controlled data to reach `settings[view options][outputFunctionName]`. CVE-2026-33916 allows a polluted `Object.prototype` property to be used as a partial body in Handlebars, rendering the polluted string without HTML escaping. Both are fixed by upgrading to EJS 3.1.7+ and Handlebars 4.7.9+ respectively.

**Beginner-Friendly Explanation:** Think of the template engine's configuration as a set of instructions on a clipboard. An attacker who pollutes the prototype can slip a new instruction onto every clipboard in the building. The template engine reads the clipboard, sees the fake instruction, and follows it — running the attacker's code. Freezing the prototype is like laminating the clipboard so no one can add instructions.

#### Purposes

- To understand the specific attack mechanisms that link prototype pollution to template engine RCE.
- To implement version upgrades and defensive coding patterns that break the attack chain.
- To ensure that render options are never populated from user-controlled objects.

#### Syntax Rules and Structure

```js
// ✅ Defence 1: Freeze Object.prototype at startup
Object.freeze(Object.prototype);

// ✅ Defence 2: Use null-prototype objects for dictionaries
const safeDict = Object.create(null);

// ✅ Defence 3: Validate input types before merging
function safeMerge(target, source) {
  for (const key of Object.keys(source)) {
    if (['__proto__', 'constructor', 'prototype'].includes(key)) continue;
    target[key] = source[key];
  }
  return target;
}

// ❌ NEVER: Spread untrusted objects into render options
// res.render('view', { ...req.body }); // DANGEROUS
```

| Defence | Implementation | Effect |
|---------|---------------|--------|
| Freeze prototype | `Object.freeze(Object.prototype)` | Prevents mutation of base prototype. |
| Null-prototype objects | `Object.create(null)` | Breaks the prototype chain. |
| Key filtering | Skip `__proto__`, `constructor`, `prototype` | Prevents pollution during merge. |
| Version discipline | EJS 3.1.7+, Handlebars 4.7.9+ | Removes known gadgets. |

**Constraints and Limitations:**
- `Object.freeze(Object.prototype)` may break libraries that rely on prototype modification; test thoroughly.
- Filtering keys during merge is not sufficient if the application uses `Object.assign` or unsafe deep-merge functions elsewhere.
- Freezing the prototype does not prevent pollution of other objects; it only protects the base prototype.

#### Annotated Code Example

```js
// app.js — Prototype pollution defence
const express = require('express');
const app = express();

// ✅ Freeze Object.prototype early (before any other module loads)
Object.freeze(Object.prototype);

app.use(express.json());

// ✅ Safe render: pass user data as values, never as options
app.get('/profile', (req, res) => {
  const name = typeof req.query.name === 'string' ? req.query.name : 'Guest';

  // ✅ SAFE: user data is a value, not spread into options
  res.render('profile', { name });

  // ❌ DANGEROUS: spreading user input into render options
  // res.render('profile', { ...req.query });
});

app.listen(3000, () => console.log('Prototype-safe server on 3000'));
```

```ejs
<!-- views/profile.ejs -->
<h1>Profile: <%= name %></h1>
```

**Expected Output (for `GET /profile?name=Alice`):**
```html
<h1>Profile: Alice</h1>
```

**Expected Output (for `GET /profile?__proto__[outputFunctionName]=x;process.mainModule.require('child_process').execSync('id');s`):**
```html
<h1>Profile: Guest</h1>
```

**Why this output:** The `name` parameter is validated as a string and passed as a value in the locals object. The `__proto__` key in the query string is not spread into the render options, so it cannot reach EJS's internal configuration. `Object.freeze(Object.prototype)` provides an additional layer of defence by preventing the prototype from being modified at all.

#### Real-World Cases

- **User registration:** An attacker submits a JSON body with `__proto__` keys to pollute the prototype.
- **Query string parameters:** Express's `qs` parser can create nested objects from query strings, enabling prototype pollution.
- **API endpoints:** Any endpoint that merges user input into an object without filtering dangerous keys.

---

## References

- EJS Official Documentation — https://ejs.co/
- EJS GitHub Repository — https://github.com/mde/ejs
- OWASP XSS Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP DOM-based XSS Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
- OWASP Output Encoding Contexts — https://wiki.owasp.org/index.php/OWASP_Encoding_Project
- Helmet Content Security Policy Documentation — https://helmetjs.github.io/
- Express.js Security Best Practices — https://expressjs.com/en/advanced/best-practice-security/
- CVE-2022-29078 (EJS RCE) — https://nvd.nist.gov/vuln/detail/CVE-2022-29078
- CVE-2026-33916 (Handlebars prototype pollution XSS) — https://nvd.nist.gov/vuln/detail/CVE-2026-33916
- CVE-2021-23383 (Handlebars prototype pollution) — https://nvd.nist.gov/vuln/detail/CVE-2021-23383
- Snyk EJS Vulnerability Database — https://security.snyk.io/vuln/SNYK-JS-EJS-2803307
- Handlebars Security Review — https://safeguard.sh/resources/blog/npm-handlebars
- EJS Security Review — https://safeguard.sh/resources/blog/ejs-npm
- Node.js Prototype Pollution Glossary — https://pentesterlab.com/glossary/nodejs-prototype-pollution
- MDN Content Security Policy — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP
- Semgrep EJS XSS Rules — https://github.com/semgrep/semgrep-rules
- "Undefined-oriented Programming" (Academic Paper on Template Engine Prototype Pollution) — https://www.semanticscholar.org/paper/Undefined-oriented-Programming%3A-Detecting-and-in-Liu/