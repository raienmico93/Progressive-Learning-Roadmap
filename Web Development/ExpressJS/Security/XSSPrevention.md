# XSS Prevention — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Cross-Site Scripting (XSS)** is a code-injection vulnerability in which untrusted data is interpreted as **executable content** by a victim's browser. It is classified under **CWE-79** and remains one of the most prevalent vulnerability classes in the OWASP Top 10. XSS attacks can lead to account impersonation, theft of sensitive data, observation of user behavior, and loading of external malicious content.

**Technical Definition.** XSS occurs when an application includes untrusted data in a web page **without appropriate output encoding or sanitization**, allowing the data to be parsed as HTML, JavaScript, CSS, or a URL instead of remaining inert text. The vulnerability manifests in three forms: **Reflected XSS** (the payload is immediately returned in the response), **Stored XSS** (the payload is persisted and later rendered to other users), and **DOM-based XSS** (the vulnerability exists entirely in client-side JavaScript, where a source flows into a dangerous sink without sanitization). The definitive defense is **context-aware output encoding**, layered with **Content Security Policy (CSP)**, **HTML sanitization**, and **framework auto-escaping**.

**Beginner-Friendly Explanation.** XSS is **when a website runs code that an attacker injected**. If a comment field accepts `<script>alert('hacked')</script>` and the site displays it without encoding, every visitor's browser executes that script. The script can steal cookies, log keystrokes, or redirect users. The fix is to **encode data before displaying it** so the browser treats it as text, not code — and to use **CSP** as a last line of defense if encoding fails.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Three distinct types** | Reflected (immediate response), Stored (persisted and re-rendered), and DOM-based (client-side source-to-sink flows). |
| **Context-dependent defense** | The correct encoding depends on where the data appears: HTML body, HTML attribute, JavaScript, CSS, or URL. |
| **Framework auto-escaping is default** | React, Angular, and Vue escape bindings by default, but every framework provides **escape hatches** that bypass protection. |
| **CSP is a last line of defense** | A strict, nonce-based CSP stops injected scripts from executing even when encoding fails. |
| **Sanitization is for rich HTML only** | When rich text must be rendered, a whitelist-based sanitizer like DOMPurify removes dangerous markup. |
| **Trusted Types close DOM XSS** | A browser-enforced runtime type system that locks down dangerous DOM sinks. |

---

### Prerequisites

- **HTML, JavaScript, and browser fundamentals**: the DOM, event handlers, and script execution.
- **HTTP fundamentals**: response headers, cookies, and the request/response model.
- **A backend or frontend framework**: Express, React, Angular, or plain JavaScript for the examples.
- **Node.js and npm**: to install DOMPurify and related libraries.
- **A browser with DevTools**: to observe CSP violations and DOM behavior.

---

## 1. XSS (Cross-Site Scripting)

### Definitions

**Core Definition.** **XSS** is a vulnerability that allows an attacker to inject **client-side executable content** into a web page viewed by other users.

**Technical Definition.** XSS arises when an application places untrusted data into a **rendering context** (HTML body, attribute, script, style, or URL) without neutralizing the characters that the browser's parser interprets as syntax. The three types differ in **where the injection occurs**: Reflected and Stored XSS are **server-side injection** issues, while DOM-based XSS is a **client-side injection** issue where the server never sees the payload. XSS executes entirely in the victim's browser and can be used to steal session tokens, perform actions on behalf of the user, or deface the page.

**Beginner-Friendly Explanation.** XSS is **the browser being tricked into running an attacker's code**. There are three ways it happens: **Reflected** — you click a malicious link and the site immediately echoes the payload back; **Stored** — the attacker saves a comment or profile that runs for everyone who views it; **DOM-based** — your own JavaScript reads a value from the URL and writes it to the page in an unsafe way, and the server never sees the attack. The defense is always the same idea: **treat data as data, never as code**.

### Purposes

- **To understand the three attack vectors** so that defenses are applied at the correct layer.
- **To motivate context-aware output encoding** as the primary defense.
- **To identify framework escape hatches** that reintroduce XSS risk.
- **To prioritize CSP and Trusted Types** as defense-in-depth controls.

### Syntax Rules and Structure

**XSS attack surface by type.**

| Type | Where the Payload Enters | Where It Executes | Server Involvement |
|---|---|---|---|
| **Reflected** | HTTP request (query string, form) | Server response HTML | High — the server reflects the input. |
| **Stored** | Database or persistent storage | Any page that renders the stored data | High — the server stores and renders. |
| **DOM-based** | URL fragment, `document.referrer`, `window.name` | Client-side JavaScript (sink) | None — the payload never reaches the server. |

**Dangerous DOM sinks (DOM-based XSS).**

| Sink | Risk |
|---|---|
| `element.innerHTML` | Parses HTML, including `<script>` and event handlers. |
| `element.outerHTML` | Same as `innerHTML` but replaces the element itself. |
| `document.write()` / `document.writeln()` | Parses HTML directly into the document. |
| `eval()` | Executes arbitrary JavaScript. |
| `setTimeout(string)` / `setInterval(string)` | Executes strings as code. |

**Constraints and limitations.**

- **Input validation alone does not prevent XSS**; a value can pass validation and still be dangerous in a specific output context.
- **HTML encoding does not work in JavaScript or URL contexts**; each context requires its own encoding.
- **Frameworks reduce but do not eliminate XSS**; escape hatches such as `dangerouslySetInnerHTML` and `bypassSecurityTrust*` reintroduce risk.

### Annotated Complete Step by Step Code Examples

#### Example 1: Reflected XSS in a search page

```javascript
// ❌ VULNERABLE: reflects the query string without encoding.
app.get('/search', (req, res) => {
  res.send(`<h1>Results for: ${req.query.q}</h1>`);
});
// Request: /search?q=<script>alert('XSS')</script>
// Response: <h1>Results for: <script>alert('XSS')</script></h1>
// The browser executes the script.

// ✅ SAFE: HTML-encode the input before embedding it.
import { encode } from 'html-entities';
app.get('/search', (req, res) => {
  res.send(`<h1>Results for: ${encode(req.query.q)}</h1>`);
});
// Request: /search?q=<script>alert('XSS')</script>
// Response: <h1>Results for: &lt;script&gt;alert('XSS')&lt;/script&gt;</h1>
// The browser renders the script tag as inert text.
```

**Expected Output (conceptual):**
```
Vulnerable: the browser executes alert('XSS').
Safe: the browser displays the literal text <script>alert('XSS')</script>.
```

**Why this output occurs.** In the vulnerable version, the query string is interpolated directly into the HTML, so the `<script>` tag is parsed as executable code. In the safe version, HTML entity encoding converts `<` to `&lt;` and `>` to `&gt;`, so the browser parses the payload as text content, not markup.

---

#### Example 2: DOM-based XSS through `innerHTML`

```javascript
// ❌ VULNERABLE: reads from the URL fragment and writes to innerHTML.
const userInput = location.hash.slice(1);
document.getElementById('output').innerHTML = userInput;
// URL: page.html#<img src=x onerror=alert('XSS')>
// The browser parses the img tag and executes onerror.

// ✅ SAFE: use textContent (no HTML parsing).
document.getElementById('output').textContent = userInput;
// The payload is rendered as literal text, never parsed as HTML.
```

**Expected Output (conceptual):**
```
Vulnerable: the onerror handler executes.
Safe: the string <img src=x onerror=alert('XSS')> is displayed as text.
```

**Why this output occurs.** `innerHTML` parses its argument as HTML, so event-handler attributes like `onerror` execute. `textContent` creates a text node without parsing HTML, so the payload cannot become executable.

---

### Real-World Cases

- **Search pages and error messages**: reflected XSS through query parameters rendered without encoding.
- **Comment sections and forums**: stored XSS that executes for every visitor.
- **Single-page applications**: DOM-based XSS through `innerHTML`, `location.hash`, or `document.referrer`.

---

## 2. Output Encoding

### Definitions

**Core Definition.** **Output encoding** is the process of transforming untrusted data so that it is **rendered as inert content** in a specific browser context, preventing it from being interpreted as executable syntax.

**Technical Definition.** Output encoding must match the **rendering context** where the data appears. The five primary contexts are **HTML body** (HTML entity encoding), **HTML attribute** (attribute encoding, with quoted attributes), **JavaScript** (JavaScript string encoding within a script block), **CSS** (CSS property-value encoding), and **URL** (percent-encoding with scheme validation). Encoding is a **last-mile defense**: it is applied at the moment data is inserted into the page, not when it is stored. Using the wrong encoder for a context can either fail to protect or break functionality.

**Beginner-Friendly Explanation.** Output encoding is **translating data into a form that is safe for the specific place it appears on the page**. If you are putting text between `<div>` tags, you HTML-encode it. If you are putting it inside a `<script>` block, you JavaScript-encode it. If you are putting it in a URL, you URL-encode it. Using the wrong type of encoding is like using a password on a door that needs a key — it does not fit.

### Purposes

- **To neutralize XSS at the point of rendering**, regardless of where the data came from.
- **To preserve the intended display of data** by encoding only the characters that have special meaning in the target context.
- **To cover gaps left by frameworks** when escape hatches or manual DOM manipulation are used.
- **To provide a consistent, auditable defense** that can be applied systematically across an application.

### Syntax Rules and Structure

**Encoding by context.**

| Context | Encoding | Example (Safe Output) |
|---|---|---|
| **HTML body** | HTML entity encoding | `<` → `&lt;`, `>` → `&gt;`, `&` → `&amp;` |
| **HTML attribute** | Attribute encoding (quoted attributes) | `"` → `&quot;`, `'` → `&#x27;` |
| **JavaScript** | JavaScript string encoding | `"` → `\u0022`, `<` → `\u003C` |
| **CSS** | CSS hex encoding | `(` → `\28`, `)` → `\29` |
| **URL** | Percent encoding + scheme allow-list | `javascript:` rejected; `encodeURIComponent` for parameters |

**Encoding rules.**

| Rule | Guidance |
|---|---|
| **Encode at the output boundary** | Apply encoding immediately before inserting data into the page, not when storing it. |
| **Match the encoder to the context** | HTML encoding does not work in JavaScript or URL contexts. |
| **Always quote HTML attributes** | Unquoted attributes allow breakout with spaces. |
| **Validate URL schemes** | Allow only `http:` and `https:`; reject `javascript:`, `data:`, and `vbscript:`. |
| **Do not pre-encode for `textContent`** | `textContent` does not parse HTML; pre-encoding would display entities literally. |

**Constraints and limitations.**

- **Contexts can change**: data encoded for one context may later be placed into another; re-encode at each boundary.
- **Encoding does not protect against all XSS**: DOM sinks like `innerHTML` still parse HTML; use safe DOM APIs or sanitization.
- **Some contexts cannot be made safe by encoding alone**: event-handler attributes and `javascript:` URLs require validation or removal.

### Annotated Complete Step by Step Code Examples

#### Example 1: HTML body encoding

```javascript
// Step 1: Define an HTML encoder.
function encodeHtml(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}

// Step 2: Encode user input before inserting into HTML.
const userComment = '<script>alert("XSS")</script>';
const safeComment = encodeHtml(userComment);

// Step 3: Render the safe output.
console.log(safeComment);
// Expected: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;
```

**Expected Output:**
```
&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;
```

**Why this output occurs.** Each character with special meaning in HTML is replaced by its entity. The browser renders the encoded string as text, so the script tag is never parsed as markup.

---

#### Example 2: HTML attribute encoding

```html
<!-- ✅ SAFE: attribute value is quoted and encoded. -->
<input value="&quot; onmouseover=&quot;alert('XSS')">
<!-- The browser treats the entire value as a single attribute value, not as a new attribute. -->

<!-- ❌ UNSAFE: unquoted attribute. -->
<input value=USER_INPUT>
<!-- Input: x onmouseover=alert('XSS') breaks out of the attribute. -->
```

**Expected Output (conceptual):**
```
Safe: the onmouseover attribute is not created; the payload is displayed as text.
Unsafe: the onmouseover handler is attached and executes on hover.
```

**Why this output occurs.** Quoting the attribute and encoding the quote characters prevents the input from terminating the attribute value. In an unquoted attribute, a space ends the value, allowing the attacker to inject a new attribute.

---

#### Example 3: JavaScript context encoding

```javascript
// ❌ UNSAFE: HTML encoding does not help inside a script block.
const name = "<%= htmlEncode(userInput) %>"; // HTML encoding is insufficient.

// ✅ SAFE: use a JavaScript-specific encoder.
const name = "\u003Cscript\u003Ealert(1)\u003C/script\u003E";
// The Unicode escapes prevent the string from breaking out.
```

**Expected Output (conceptual):**
```
Unsafe: the HTML-encoded payload may still contain characters that break out of the string.
Safe: all non-alphanumeric characters are Unicode-escaped, so the string cannot terminate early.
```

**Why this output occurs.** In a JavaScript string context, the parser interprets `"` and `\` as syntax. JavaScript encoding replaces these with `\uXXXX` escapes, so the string remains a single literal value. HTML encoding is the wrong tool for this context.

---

#### Example 4: URL context validation and encoding

```javascript
// ❌ UNSAFE: the URL scheme is not validated.
const link = `<a href="${userInput}">Click</a>`;
// Input: javascript:alert('XSS')
// The link executes JavaScript when clicked.

// ✅ SAFE: validate the scheme and encode the URL.
function safeUrl(input) {
  try {
    const url = new URL(input);
    if (url.protocol === 'http:' || url.protocol === 'https:') {
      return encodeURI(url.toString());
    }
  } catch {}
  return '#';
}
const link = `<a href="${safeUrl(userInput)}">Click</a>`;
```

**Expected Output (conceptual):**
```
Unsafe: clicking the link executes alert('XSS').
Safe: the javascript: scheme is rejected and replaced with '#'.
```

**Why this output occurs.** URL encoding alone does not prevent `javascript:` URLs; the **scheme must be validated** against an allow-list of safe protocols. Encoding handles special characters in the URL, while validation ensures the URL cannot execute code.

---

### Real-World Cases

- **Server-rendered templates**: HTML-encoding every user-supplied variable before interpolation.
- **Client-side rendering**: using `textContent` instead of `innerHTML` for plain text.
- **URL parameters**: validating and encoding redirect targets and link hrefs.

---

## 3. Content Security Policy (CSP)

### Definitions

**Core Definition.** **Content Security Policy (CSP)** is an HTTP response header that tells the browser **which sources of script, style, and other content are allowed to load and execute**, acting as a last line of defense against XSS.

**Technical Definition.** CSP is a **browser-enforced allow-list**. A **strict, nonce-based CSP** generates a fresh random nonce per response and stamps it on trusted script tags; the browser runs only scripts carrying the correct nonce, so injected scripts (which cannot know the nonce) never execute. Adding **`strict-dynamic`** lets trusted scripts load further scripts, eliminating the need for host allow-lists. **`object-src 'none'`** and **`base-uri 'none'`** close off plugin and base-tag injection vectors. **Trusted Types** (`require-trusted-types-for 'script'`) further locks down DOM sinks like `innerHTML`.

**Beginner-Friendly Explanation.** CSP is **a list of approved script sources that the browser enforces**. Even if an attacker injects a `<script>` tag, the browser refuses to run it because it does not carry the secret nonce that the server generated for that page. A strict CSP is **the safety net under the trapeze**: if output encoding fails, CSP prevents the injected script from executing.

### Purposes

- **To contain XSS when encoding or sanitization fails**, preventing injected scripts from executing.
- **To block inline scripts and event handlers** that are common XSS vectors.
- **To restrict resource loading** (scripts, styles, frames, connections) to trusted sources.
- **To provide visibility into attempted violations** through report-only mode before enforcement.

### Syntax Rules and Structure

**Strict nonce-based CSP header.**

```
Content-Security-Policy: script-src 'nonce-r4nd0m123' 'strict-dynamic'; object-src 'none'; base-uri 'none'; require-trusted-types-for 'script'
```

**Matching markup.**

```html
<script nonce="r4nd0m123" src="/app.js"></script>
```

**CSP rollout rules.**

| Rule | Guidance |
|---|---|
| **Generate a fresh nonce per response** | A static or reused nonce is worthless. |
| **Never add `'unsafe-inline'`** | It defeats the purpose of the CSP and is ignored when a nonce is present. |
| **Start in report-only mode** | `Content-Security-Policy-Report-Only` reports violations without blocking, so you can identify breakage before enforcement. |
| **Close DOM XSS with Trusted Types** | `require-trusted-types-for 'script'` forces dangerous sinks to accept only typed, trusted values. |

**Constraints and limitations.**

- **Host allow-lists are bypassable**; any allowed host with an open redirect or JSONP endpoint becomes a vector.
- **CSP does not fix XSS**; it contains the impact. Output encoding and sanitization remain necessary.
- **`'unsafe-inline'` is a red flag**; its presence usually indicates a policy that was copied rather than designed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Strict nonce-based CSP in Express

```javascript
const crypto = require('crypto');

app.use((req, res, next) => {
  // Step 1: Generate a fresh nonce per request.
  res.locals.nonce = crypto.randomBytes(16).toString('base64');
  // Step 2: Set the CSP header.
  res.setHeader(
    'Content-Security-Policy',
    `script-src 'nonce-${res.locals.nonce}' 'strict-dynamic'; object-src 'none'; base-uri 'none';`
  );
  next();
});

app.get('/', (req, res) => {
  // Step 3: Stamp the nonce on trusted scripts.
  res.send(`
    <script nonce="${res.locals.nonce}" src="/app.js"></script>
  `);
});
```

**Expected Response Header:**
```
Content-Security-Policy: script-src 'nonce-XYZ...' 'strict-dynamic'; object-src 'none'; base-uri 'none';
```

**Expected Browser Behavior:**
```
The trusted script with the matching nonce executes.
An injected <script>alert(1)</script> without the nonce is blocked.
```

**Why this output occurs.** The browser compares the nonce on each script tag against the nonce in the CSP header. Only the trusted script matches, so only it executes. The injected script lacks the nonce and is blocked.

---

#### Example 2: Report-only mode before enforcement

```http
Content-Security-Policy-Report-Only: script-src 'nonce-r4nd0m123' 'strict-dynamic'; object-src 'none'; report-to csp-endpoint
```

**Expected Output (conceptual):**
```
The browser does not block any scripts but sends violation reports to the configured endpoint.
Developers use the reports to fix legitimate sources before switching to enforcement.
```

**Why this output occurs.** Report-only mode allows the team to observe what would break under the policy without disrupting users. Once the reports are clean, the header is changed to `Content-Security-Policy` for enforcement.

---

### Real-World Cases

- **GitHub and Google**: strict nonce-based CSPs with `strict-dynamic` to allow their script loaders.
- **SPAs**: nonce-based CSPs integrated with server-side rendering to stamp nonces on framework-injected scripts.
- **Trusted Types adoption**: Chrome supports `require-trusted-types-for 'script'` to eliminate DOM-based XSS.

---

## 4. Sanitization

### Definitions

**Core Definition.** **Sanitization** is the process of **removing or neutralizing dangerous HTML** from untrusted input when rich HTML must be rendered, using a **whitelist of allowed tags and attributes**.

**Technical Definition.** Sanitization is distinct from encoding: encoding preserves the input as text, while sanitization **parses the HTML and removes disallowed elements and attributes**, returning a safe HTML fragment. **DOMPurify** is the OWASP-recommended sanitizer for HTML; it removes dangerous tags (`<script>`, `<iframe>`), event-handler attributes (`onerror`, `onclick`), and dangerous URL schemes (`javascript:`) while preserving explicitly allowed markup. Sanitization is appropriate for **rich-text content** (comments with formatting, CMS content) where pure text encoding would destroy intended formatting.

**Beginner-Friendly Explanation.** Sanitization is **cleaning HTML so it only contains safe tags**. If you want to let users write bold or italic text, you cannot just encode everything (that would show the tags as text) and you cannot trust the raw HTML (it could contain scripts). A sanitizer like DOMPurify **keeps the safe tags** (`<b>`, `<i>`, `<p>`) and **removes the dangerous ones** (`<script>`, `onerror`), so the content renders with formatting but cannot execute code.

### Purposes

- **To allow safe rich-text formatting** while removing executable content.
- **To protect against stored XSS** in comment systems, CMS platforms, and user profiles.
- **To complement output encoding** when the output context genuinely requires HTML.
- **To enforce a documented allow-list** of tags and attributes rather than trying to block every known attack.

### Syntax Rules and Structure

**DOMPurify basic usage.**

```javascript
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(dirtyHtml);
```

**DOMPurify with an allow-list.**

```javascript
const clean = DOMPurify.sanitize(dirtyHtml, {
  ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong', 'a'],
  ALLOWED_ATTR: ['href', 'title'],
});
```

**Sanitization rules.**

| Rule | Guidance |
|---|---|
| **Use a whitelist, not a blacklist** | Allow only known-safe tags and attributes; everything else is removed. |
| **Sanitize at render time** | Sanitize immediately before inserting into the DOM, not when storing. |
| **Never use `dangerouslySetInnerHTML` without sanitization** | React's escape hatch bypasses JSX auto-escaping. |
| **Validate URL schemes** | Even sanitized `<a href>` values need scheme validation (`http:`/`https:` only). |

**Constraints and limitations.**

- **Sanitization is complex**; hand-rolled sanitizers are error-prone. Use a maintained library like DOMPurify.
- **Sanitization can break legitimate content** if the allow-list is too restrictive.
- **Sanitization does not replace CSP**; it is a complementary layer.

### Annotated Complete Step by Step Code Examples

#### Example 1: DOMPurify removing a script tag

```javascript
import DOMPurify from 'dompurify';

const dirty = '<p>Safe content</p><script>alert("XSS")</script>';
const clean = DOMPurify.sanitize(dirty);

console.log(clean);
// Expected: <p>Safe content</p>
```

**Expected Output:**
```
<p>Safe content</p>
```

**Why this output occurs.** DOMPurify parses the HTML and removes the `<script>` element because it is not in the default allow-list. The safe `<p>` tag is preserved, so the content renders with its intended formatting.

---

#### Example 2: DOMPurify with a custom allow-list

```javascript
const userHtml = '<p>Hello <b>world</b> <script>alert("XSS")</script></p>';
const sanitized = DOMPurify.sanitize(userHtml, {
  ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong'],
});

console.log(sanitized);
// Expected: <p>Hello <b>world</b> </p>
```

**Expected Output:**
```
<p>Hello <b>world</b> </p>
```

**Why this output occurs.** The allow-list permits `p`, `b`, `i`, `em`, and `strong` but not `script`. DOMPurify removes the script tag while preserving the permitted formatting tags. The result is safe HTML that renders as intended.

---

### Real-World Cases

- **Comment systems**: allowing basic formatting while stripping scripts and event handlers.
- **CMS platforms**: sanitizing rich-text editor output before rendering.
- **React applications**: sanitizing every `dangerouslySetInnerHTML` value with DOMPurify.

---

## 5. XSS Prevention Best Practices

### Definitions

**Core Definition.** **XSS prevention best practices** are the layered, empirically established controls that eliminate XSS at the rendering boundary and contain its impact if a defense fails.

**Technical Definition.** Best practices center on **context-aware output encoding** (the primary defense), **strict nonce-based CSP** (the containment layer), **whitelist sanitization** for rich HTML, **framework auto-escaping** with disciplined escape-hatch usage, and **Trusted Types** for DOM sinks. OWASP frames the philosophy as **"perfect injection resistance"**: every variable must be validated and then escaped or sanitized at the output boundary.

**Beginner-Friendly Explanation.** Best practices mean **layering your defenses and never trusting a single control**. Encode data for the exact context where it appears. Add a strict CSP so injected scripts cannot run even if encoding fails. Sanitize rich HTML instead of rendering it raw. Use your framework's default escaping and be extremely careful with its escape hatches. Close DOM XSS with safe APIs or Trusted Types. And **test everything** with real payloads in each context.

### Purposes

- **To provide defense in depth** so a single failure does not lead to XSS.
- **To match the defense to the context**, avoiding the false sense of security that comes from HTML-encoding everything.
- **To eliminate framework-specific escape-hatch risks** through code review and automated checks.
- **To close DOM-based XSS** through safe DOM APIs and Trusted Types.
- **To maintain protection across the application lifecycle** through automated testing and monitoring.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Best Practice | Why It Matters |
|---|---|---|
| **Primary defense** | Context-aware output encoding (HTML, attribute, JS, CSS, URL) | Encoding the wrong context either fails or breaks functionality. |
| **Containment** | Strict nonce-based CSP with `strict-dynamic` | Stops injected scripts from executing even when encoding fails. |
| **Rich HTML** | Whitelist sanitization with DOMPurify | Allows safe formatting while removing executable markup. |
| **Frameworks** | Use auto-escaping; audit every escape hatch | `dangerouslySetInnerHTML`, `bypassSecurityTrust*`, `v-html`, and `|safe` bypass protection. |
| **DOM XSS** | Use `textContent` instead of `innerHTML`; enable Trusted Types | `innerHTML` parses HTML; `textContent` does not. |
| **URLs** | Validate schemes against an allow-list | `javascript:` URLs execute code even when encoded. |
| **Testing** | Test each context with context-specific payloads | HTML body, attribute, JS, and URL contexts require different payloads. |

**Framework escape hatches to audit.**

| Framework | Safe Default | Dangerous Escape Hatch |
|---|---|---|
| **React** | JSX auto-escapes `{variable}` | `dangerouslySetInnerHTML` without DOMPurify |
| **Angular** | Template binding sanitizes by default | `bypassSecurityTrustHtml()` and related methods |
| **Vue** | `{{ }}` interpolation escapes | `v-html` |
| **Django/Jinja2** | Template variables auto-escape | `|safe` filter or `{% autoescape false %}` |

**Constraints and limitations.**

- **No single control is sufficient**; XSS prevention requires a combination of encoding, CSP, and sanitization.
- **CSP is not a substitute for encoding**; it contains the impact but does not fix the vulnerability.
- **Sanitization is not a substitute for encoding**; it is only appropriate for rich HTML, not for plain text.

### Annotated Complete Step by Step Code Examples

#### Example 1: Layered defense in an Express application

```javascript
import { encode } from 'html-entities';
import DOMPurify from 'dompurify';
import crypto from 'crypto';

// Layer 1: Strict nonce-based CSP.
app.use((req, res, next) => {
  res.locals.nonce = crypto.randomBytes(16).toString('base64');
  res.setHeader(
    'Content-Security-Policy',
    `script-src 'nonce-${res.locals.nonce}' 'strict-dynamic'; object-src 'none'; base-uri 'none';`
  );
  next();
});

// Layer 2: HTML encoding for plain text.
app.get('/profile', (req, res) => {
  const bio = req.query.bio || '';
  res.send(`
    <p>${encode(bio)}</p>
    <script nonce="${res.locals.nonce}" src="/app.js"></script>
  `);
});

// Layer 3: Sanitization for rich HTML.
app.get('/comment', (req, res) => {
  const rawHtml = req.query.comment || '';
  const clean = DOMPurify.sanitize(rawHtml, {
    ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong'],
  });
  res.send(`<div>${clean}</div>`);
});
```

**Expected Output (conceptual):**
```
Plain text bio → HTML-encoded, displayed as inert text.
Rich HTML comment → sanitized, safe tags preserved, scripts removed.
Injected script without nonce → blocked by CSP.
```

**Why this output occurs.** Three independent layers protect the application: CSP blocks nonce-less scripts, HTML encoding neutralizes plain-text output, and sanitization removes dangerous markup from rich HTML. An attacker must defeat all three layers, which is impractical without a separate vulnerability.

---

#### Example 2: Safe DOM manipulation in a SPA

```javascript
// ❌ UNSAFE: innerHTML parses the input as HTML.
document.getElementById('output').innerHTML = userInput;

// ✅ SAFE: textContent creates a text node.
document.getElementById('output').textContent = userInput;

// ✅ SAFE: DOMPurify before innerHTML when HTML is required.
document.getElementById('output').innerHTML = DOMPurify.sanitize(userInput);
```

**Expected Output (conceptual):**
```
Unsafe: injected <img src=x onerror=alert(1)> executes.
Safe with textContent: the payload is displayed as literal text.
Safe with DOMPurify: dangerous attributes are removed; safe HTML is preserved.
```

**Why this output occurs.** `textContent` never parses HTML, so no injected markup can execute. When HTML rendering is genuinely required, DOMPurify removes dangerous elements and attributes before `innerHTML` parses the result.

---

### Real-World Cases

- **OWASP ASVS**: requires context-aware output encoding and CSP as part of application security verification.
- **React, Angular, and Vue security guides**: emphasize auditing escape hatches (`dangerouslySetInnerHTML`, `bypassSecurityTrust*`, `v-html`) and sanitizing dynamic HTML.
- **Chrome Trusted Types**: adopted by applications that need to eliminate DOM-based XSS at the browser level.

---

## References

- OWASP — Cross Site Scripting Prevention Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP — DOM based XSS Prevention Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
- OWASP — Content Security Policy Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html
- OWASP — XSS Filter Evasion Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html
- OWASP Top 10 Proactive Controls — C4: Encode and Escape Data - https://top10proactive.owasp.org/archive/2018/c4-encode-escape-data/
- MDN — Content-Security-Policy - https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- W3C — Trusted Types - https://www.w3.org/TR/trusted-types/
- DOMPurify — GitHub Repository - https://github.com/cure53/DOMPurify
- CWE-79: Cross-site Scripting (XSS) - https://cwe.mitre.org/data/definitions/79.html
- Safeguard.sh — Context-Aware Encoding for XSS Prevention (2026) - https://safeguard.sh/resources/blog/cross-site-scripting-xss-prevention
- Safeguard.sh — Content Security Policy Explained (2026) - https://safeguard.sh/resources/blog/content-security-policy-csp-explained
- React — Security Best Practices (dangerouslySetInnerHTML) - https://safeguard.sh/resources/blog/react-security-best-practices
- Angular — DomSanitizer and bypassSecurityTrust* - https://angular.io/api/platform-browser/DomSanitizer