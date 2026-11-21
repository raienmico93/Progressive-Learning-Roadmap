# jQuery XSS Prevention — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery XSS prevention is the discipline of ensuring that untrusted data — user input, URL parameters, API responses, or stored content — never reaches a jQuery method that interprets strings as HTML markup or executable code, thereby preventing attackers from injecting and executing malicious scripts in a victim's browser.

**Technical Definition**
Cross‑Site Scripting (XSS) is an injection vulnerability in which untrusted data is interpreted as code by the browser. In jQuery applications, XSS arises when untrusted data flows from a **source** (URL fragment, `location.search`, form input, AJAX response, database content) to a **sink** — a jQuery method that parses its argument as HTML. The primary unsafe sinks are `.html()`, `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()`, and the `$()` constructor when passed a string containing markup. Safe alternatives such as `.text()` and `.val()` treat their arguments as **data**, not markup, and do not execute scripts. `.attr()` is context‑dependent: setting ordinary attributes (e.g., `title`, `class`) is generally safe, but setting URL‑context attributes (`href`, `src`) or event‑handler attributes (`onclick`) can create XSS or open‑redirect vectors. The three XSS categories — reflected, stored, and DOM‑based — differ in where the payload lives and where it is rendered, but all are preventable through a combination of **output encoding** at the sink, **input validation** at the boundary, **sanitisation** for rich HTML, and **secure coding practices** that default to safe APIs.

**Beginner‑Friendly Explanation**
XSS is what happens when a bad guy manages to sneak a script into your page and make it run in someone else's browser. Imagine a guestbook where visitors type comments. If you display those comments using jQuery's `.html()`, and a visitor types `<script>stealCookies()</script>`, that script will run for every person who views the guestbook. XSS prevention is about never letting user‑provided text be treated as code. jQuery gives you safe tools — `.text()` and `.val()` — that display content as plain text, no matter what it contains. The unsafe tools — `.html()`, `.append()`, and friends — should only be used with HTML you trust, or with HTML you have **sanitised** through a library like DOMPurify. The rule is simple: if the data came from a user, an API, or a database, treat it as hostile until proven otherwise.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Source → sink data flow** | XSS requires untrusted data (source) to reach a dangerous API (sink); breaking the flow breaks the attack. |
| **`.html()` is the primary jQuery sink** | `.html()`, `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()`, and `$(string)` all parse HTML and can execute scripts. |
| **`.text()` and `.val()` are safe** | These methods set text content or form values as data, escaping or ignoring markup. |
| **`.attr()` is context‑dependent** | Safe for non‑URL, non‑event attributes; unsafe for `href`, `src`, `onclick`, and similar. |
| **Encoding is the primary defence** | Contextual output encoding (HTML entity encoding for HTML content, attribute encoding for attributes) neutralises payloads. |
| **Sanitisation is for rich HTML** | When HTML must be preserved, DOMPurify with the `SAFE_FOR_JQUERY` flag is the recommended library. |
| **Client‑side validation is not security** | Validation improves UX but must be backed by server‑side enforcement; attackers can bypass the client. |

---

### Prerequisites

- Working knowledge of jQuery selection, DOM manipulation, and AJAX.
- Familiarity with HTML structure and how browsers parse markup.
- Awareness of the three XSS categories (reflected, stored, DOM‑based).
- Understanding that client‑side code runs in an attacker‑controlled environment.

---

### Related Programming Areas

- **Input validation** — enforcing expected formats at trust boundaries.
- **Output encoding** — neutralising data at the sink.
- **Sanitisation** — allowing safe HTML while removing dangerous constructs.
- **Content Security Policy (CSP)** — defence in depth that restricts script sources.
- **Secure cookies and session management** — limiting the impact of successful XSS.

---

### Core Concepts / Features (Index)

1. XSS (Cross‑Site Scripting)
2. Reflected XSS
3. Stored XSS
4. DOM‑based XSS
5. `.html()` — Unsafe Sink
6. `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()` — Unsafe Sinks
7. `.text()` — Safe Sink
8. `.val()` — Safe Sink
9. `.attr()` — Context‑Dependent Sink
10. Output Encoding
11. Input Validation
12. Sanitisation
13. DOMPurify
14. XSS Prevention Best Practices


## Core Concept 1: XSS (Cross‑Site Scripting)

### Definitions

**Core Definition**
Cross‑Site Scripting (XSS) is an injection vulnerability that occurs when an application includes untrusted data in a web page without proper validation or escaping, allowing an attacker to execute malicious scripts in a victim's browser.

**Technical Definition**
XSS is classified as an injection attack in which the attacker's script is executed in the context of the vulnerable application's origin, granting access to cookies, session tokens, DOM content, and the ability to perform actions on behalf of the victim. XSS is categorised into three types: **reflected** (the payload is immediately returned by the server in a response), **stored** (the payload is persisted on the server and served to other users), and **DOM‑based** (the vulnerability exists entirely in client‑side code, where data from a source flows to a dangerous sink without server involvement). XSS is listed in the OWASP Top 10 as part of **Injection** (A03:2021 and A05:2025).

**Beginner‑Friendly Explanation**
XSS is what happens when an attacker finds a way to make your website run their code. It could be a malicious link (reflected), a poisoned comment that affects everyone who reads it (stored), or a flaw in your own JavaScript that lets attacker‑controlled data slip into a dangerous place (DOM‑based). The result is always the same: the attacker's script runs as if it were part of your site, and it can steal data, impersonate users, or damage the page.

---

### Purposes

- **To understand the fundamental mechanism** of XSS (untrusted data reaching a code‑interpreting sink).
- **To classify XSS into reflected, stored, and DOM‑based types** so that the correct defence can be selected.
- **To recognise that XSS is an injection problem**, not merely a “JavaScript problem”, and is addressed by breaking the data‑to‑code flow.
- **To motivate the use of safe APIs and output encoding** rather than attempting to filter individual characters.

---

### Syntax Rules and Structure

XSS is not expressed in a single syntax; it is a pattern:

```
Untrusted Source → (lack of encoding/validation) → Dangerous Sink → Script Execution
```

| Source Examples | Sink Examples (jQuery) |
|---|---|
| `location.hash`, `location.search` | `.html()`, `.append()` |
| Form input, `document.cookie` | `$(userInput)` |
| AJAX response, database content | `.after()`, `.replaceWith()` |

---

### Constraints and Limitations

- **Filtering blacklists is ineffective**; attackers use encoding, polyglots, and browser quirks to bypass naive filters.
- **Client‑side validation alone does not prevent XSS**; attackers can disable JavaScript or intercept requests.
- **XSS can exist even in “safe” frameworks** if developers bypass escaping mechanisms.

---

### Annotated Code Example

```javascript
// ❌ XSS VULNERABILITY: untrusted data reaches .html()
var userComment = "<img src=x onerror=alert('XSS')>";
$("#commentBox").html(userComment);    // script executes

// ✅ XSS PREVENTED: .text() treats data as text
$("#commentBox").text(userComment);     // displays literal text, no execution
```

**Expected Output**
The unsafe version executes the `onerror` handler. The safe version displays the literal string `<img src=x onerror=alert('XSS')>` without executing it.

**Why this output occurs**: `.html()` parses the string as HTML and creates an `<img>` element whose `onerror` attribute contains JavaScript. `.text()` sets the text content, and the browser renders the angle brackets as visible characters rather than interpreting them as markup.

---

### Real‑World Cases

- **Comment sections**: Stored XSS through unsanitised comments that execute for every visitor.
- **Search results pages**: Reflected XSS through the search query echoed back into the page.
- **Single‑page applications**: DOM‑based XSS through `location.hash` written into `.html()`.

---

## Core Concept 2: Reflected XSS

### Definitions

**Core Definition**
Reflected XSS occurs when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way, causing the attacker's payload to execute in the victim's browser when they click a crafted link or submit a crafted form.

**Technical Definition**
In reflected XSS, the malicious script is **not stored** on the server; it is reflected from the current request. The attacker delivers the payload via a URL parameter, form field, or HTTP header, and the server includes it in the response HTML without proper encoding. The payload executes in the victim's browser as part of the legitimate page. The primary defence is **contextual output encoding** at the server, ensuring that any reflected data is treated as text, not markup.

**Beginner‑Friendly Explanation**
Reflected XSS is like a boomerang: the attacker throws the payload at the server, and the server throws it right back into the page. The victim only needs to click a malicious link for the attack to work. The payload does not stay on the server — it is “reflected” from the request. The fix is to encode the data before putting it into the page, so the browser sees it as text rather than code.

---

### Purposes

- **To understand the “immediate echo” attack pattern** and why URL parameters are a common vector.
- **To prioritise output encoding** as the primary defence for reflected data.
- **To recognise that reflected XSS does not require persistence** and can affect any user who clicks a crafted link.
- **To motivate server‑side encoding** rather than relying on client‑side filters.

---

### Syntax Rules and Structure

```
Request with payload → Server reflects payload in response → Browser executes payload
```

**Defence**: Encode the reflected data with HTML entity encoding before inserting it into the response.

---

### Constraints and Limitations

- **Server‑side encoding is mandatory**; client‑side filters can be bypassed.
- **Reflected XSS can occur in any part of the response** — HTML body, attributes, JavaScript, CSS, or URLs.
- **The payload is not stored**, so the attack requires delivery to the victim (phishing link, malicious form).

---

### Annotated Code Example (Conceptual)

```javascript
// ❌ REFLECTED XSS (server-side pseudo-code)
app.get("/search", function (req, res) {
  res.send("<h1>Results for: " + req.query.q + "</h1>");
});
// Attacker link: /search?q=<script>alert('XSS')</script>

// ✅ REFLECTED XSS PREVENTED (server-side pseudo-code)
app.get("/search", function (req, res) {
  var safeQuery = escapeHtml(req.query.q);   // encode before insertion
  res.send("<h1>Results for: " + safeQuery + "</h1>");
});
```

**Expected Output**
The vulnerable endpoint executes the script when the crafted URL is visited. The safe endpoint displays the query as text (e.g., `&lt;script&gt;alert('XSS')&lt;/script&gt;`), preventing execution.

**Why this output occurs**: HTML entity encoding converts `<` to `&lt;` and `>` to `&gt;`, so the browser treats the payload as text rather than markup.

---

### Real‑World Cases

- **Search pages**: Query parameters echoed into results.
- **Error messages**: Unencoded error text reflected from request parameters.
- **Login forms**: “Invalid username” messages that include the submitted value.

---

## Core Concept 3: Stored XSS

### Definitions

**Core Definition**
Stored XSS (also called persistent XSS) occurs when an application stores untrusted data — in a database, file system, or other persistent store — and later includes that data in a page without proper encoding, causing the attacker's script to execute for every user who views the affected content.

**Technical Definition**
Stored XSS is the most damaging category because the payload is persisted and served to multiple victims without requiring individual delivery. Common vectors include comment fields, user profiles, forum posts, product reviews, and log viewers. The defence requires **output encoding at the point of rendering** (not merely input filtering), because the same stored data may be rendered in different contexts (HTML body, attribute, JavaScript string) requiring different encodings. Server‑side sanitisation of rich HTML is also necessary when stored content is intended to contain markup.

**Beginner‑Friendly Explanation**
Stored XSS is like leaving a cursed note in a guestbook. Everyone who reads the guestbook is affected. The attacker does not need to trick each victim individually — they just plant the payload once, and the server serves it to everyone. The fix is to encode the stored data every time it is rendered, and to sanitise rich content before storing it.

---

### Purposes

- **To understand why persistent storage amplifies the impact** of XSS.
- **To prioritise output encoding at every rendering point**, not just at input.
- **To motivate server‑side sanitisation** for content that is intended to contain HTML.
- **To recognise that stored XSS can affect administrators and privileged users**, leading to full compromise.

---

### Syntax Rules and Structure

```
Attacker submits payload → Server stores payload → Victim views page → Server renders stored payload unsafely → Script executes
```

**Defence**: Encode stored data at output, and sanitise rich HTML on input with a vetted library.

---

### Constraints and Limitations

- **Stored XSS can persist for long periods** if not detected.
- **Different rendering contexts require different encodings**; a single input filter is insufficient.
- **Database content from external sources** (APIs, integrations) must also be treated as untrusted.

---

### Annotated Code Example (Conceptual)

```javascript
// ❌ STORED XSS (server-side pseudo-code)
app.post("/comment", function (req, res) {
  db.saveComment(req.body.text);           // payload stored unsanitised
});
app.get("/comments", function (req, res) {
  var comments = db.getComments();
  res.send(comments.map(c => "<div>" + c.text + "</div>").join(""));
  // payload rendered unsafely for every visitor
});

// ✅ STORED XSS PREVENTED
app.post("/comment", function (req, res) {
  var cleanText = sanitizeHtml(req.body.text);   // sanitise on input
  db.saveComment(cleanText);
});
app.get("/comments", function (req, res) {
  var comments = db.getComments();
  res.send(comments.map(c => "<div>" + escapeHtml(c.text) + "</div>").join(""));
  // double protection: sanitised storage + encoded output
});
```

**Expected Output**
The vulnerable application executes the stored script for every visitor. The safe application stores sanitised HTML and encodes it at output, preventing execution while preserving safe formatting.

**Why this output occurs**: Sanitisation removes dangerous constructs (e.g., `<script>`, event handlers) from the stored content. Output encoding provides a second layer of defence by neutralising any residual markup at render time.

---

### Real‑World Cases

- **Comment systems**: Payload executed for every reader.
- **User profiles**: “About me” fields that render HTML.
- **Support tickets**: Attacker‑submitted content viewed by support staff.

---

## Core Concept 4: DOM‑based XSS

### Definitions

**Core Definition**
DOM‑based XSS is a cross‑site scripting vulnerability that exists entirely in client‑side JavaScript, where attacker‑controlled data from a **source** (e.g., `location.hash`, `document.URL`, `window.name`) flows into a dangerous **sink** (e.g., `innerHTML`, jQuery `.html()`, `$(string)`) that interprets the data as code.

**Technical Definition**
In DOM‑based XSS, the server may never see the payload; the vulnerability is created by the client‑side code itself. The classic pattern is `$(location.hash)` or `$("#target").html(location.hash)`. jQuery introduces additional sinks beyond the standard DOM APIs: `.html()`, `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()`, and the `$()` constructor when passed a string containing markup. The defence is to use **safe APIs** (`.text()`, `setAttribute` with non‑URL attributes, `textContent`) and to encode or sanitise data before it reaches any sink.

**Beginner‑Friendly Explanation**
DOM‑based XSS is a bug in your own JavaScript. The attacker does not need to involve the server at all — they just craft a URL fragment (the part after `#`) or another client‑side source, and your code takes that data and puts it into a dangerous jQuery method. For example, if your code does `$("#content").html(location.hash)`, an attacker can send a link like `page.html#<script>alert(1)</script>` and the script runs. The fix is to never pass client‑side data to `.html()` or similar methods, and to use `.text()` instead.

---

### Purposes

- **To recognise that client‑side code can create XSS independently of the server**.
- **To identify jQuery‑specific sinks** that are not part of the standard DOM API.
- **To prefer safe APIs** (`.text()`, `.val()`, safe attribute setting) over HTML‑parsing sinks.
- **To understand why sanitisation or encoding is required even when data never leaves the browser**.

---

### Syntax Rules and Structure

```
Client-side source → (no encoding/sanitisation) → jQuery sink → Script execution
```

| Common Sources | jQuery Sinks |
|---|---|
| `location.hash`, `location.search` | `.html()`, `.append()` |
| `document.referrer` | `.prepend()`, `.after()` |
| `window.name` | `$()` with markup |
| `document.cookie` | `.replaceWith()` |

---

### Constraints and Limitations

- **The server cannot protect against DOM‑based XSS** unless it also controls the client code.
- **Client‑side validation can be bypassed**; the sink is in the browser.
- **Sanitisation must occur before the sink**; encoding after the sink is too late.

---

### Annotated Code Example

```javascript
// ❌ DOM-BASED XSS
var hash = location.hash.substring(1);
$("#content").html(hash);                 // payload executes

// ✅ DOM-BASED XSS PREVENTED
var hash = location.hash.substring(1);
$("#content").text(hash);                  // payload displayed as text

// ✅ ALTERNATIVE: sanitise before using .html()
var clean = DOMPurify.sanitize(hash, { SAFE_FOR_JQUERY: true });
$("#content").html(clean);                 // safe HTML rendered
```

**Expected Output**
A URL like `page.html#<img src=x onerror=alert(1)>` executes the `onerror` handler in the vulnerable version. The safe version displays the literal string without execution. The DOMPurify version renders any safe markup but strips the dangerous handler.

**Why this output occurs**: `.html()` parses the string as HTML. `.text()` sets text content. DOMPurify removes dangerous elements and attributes while preserving safe markup.

---

### Real‑World Cases

- **Hash‑based navigation**: `location.hash` written into the DOM.
- **Client‑side templating**: User‑provided data rendered with `.html()` without sanitisation.
- **Single‑page applications**: API responses inserted into the DOM without encoding.

---

## Core Concept 5: `.html()` — Unsafe Sink

### Definitions

**Core Definition**
`.html()` is a jQuery method that sets or returns the HTML content of an element. When used as a setter with an untrusted string, it parses the string as HTML and can execute embedded scripts.

**Technical Definition**
`.html(htmlString)` removes all existing content and replaces it with the parsed HTML representation of the string. Internally, jQuery uses the browser's HTML parser (via `innerHTML` or a DocumentFragment), which means `<script>` tags, event‑handler attributes (`onerror`, `onclick`), and other executable constructs are processed and may execute. `.html()` is therefore the most direct and most dangerous jQuery XSS sink.

**Beginner‑Friendly Explanation**
`.html()` is the “insert HTML here” method. It is useful when you genuinely want to add markup — but it is also the easiest way to introduce XSS. If you pass user input to `.html()`, you are telling the browser: “treat this as code.” The safe alternative is `.text()`, which treats the input as plain text.

---

### Purposes

- **To insert trusted HTML** into an element.
- **To understand why `.html()` is unsafe with untrusted data**.
- **To motivate the use of `.text()` or sanitisation** when the data is not fully trusted.
- **To identify `.html()` as a high‑risk sink in security audits**.

---

### Syntax Rules and Structure

```
$( selector ).html( htmlString )
$( selector ).html()                 // getter — returns HTML string
```

**Unsafe usage**: `.html(untrustedString)`.

**Safe usage**: `.html(sanitisedString)` where the string has been processed by DOMPurify or a server‑side sanitiser.

---

### Constraints and Limitations

- **`.html()` cannot be made safe by blacklist filtering**; use an allowlist‑based sanitiser.
- **`.html()` executes scripts in the string**, including `<script>` tags and event handlers.
- **`.html()` is not equivalent to `.text()`**; the choice has security implications.

---

### Annotated Code Example

```javascript
// ❌ XSS: user input passed to .html()
var userInput = "<img src=x onerror=alert('XSS')>";
$("#output").html(userInput);             // script executes

// ✅ SAFE: use .text() for untrusted data
$("#output").text(userInput);              // displayed as text

// ✅ SAFE (when HTML is needed): sanitise with DOMPurify
var clean = DOMPurify.sanitize(userInput, { SAFE_FOR_JQUERY: true });
$("#output").html(clean);                  // dangerous constructs removed
```

**Expected Output**
The unsafe version executes the `onerror` handler. The `.text()` version displays the literal string. The DOMPurify version removes the `onerror` attribute, preventing execution while possibly preserving safe markup.

**Why this output occurs**: `.html()` parses the string as HTML. `.text()` sets text content. DOMPurify strips dangerous elements and attributes according to its allowlist.

---

### Real‑World Cases

- **Comment rendering**: Unsanitised `.html(comment)` creates stored XSS.
- **Search result highlighting**: `.html()` used to insert highlighted terms can execute scripts if terms contain markup.
- **Template rendering**: Client‑side templates that use `.html()` with user data.

---

## Core Concept 6: `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()` — Unsafe Sinks

### Definitions

**Core Definition**
These jQuery methods insert content relative to an element — inside it (`.append()`, `.prepend()`), adjacent to it (`.after()`, `.before()`), or in place of it (`.replaceWith()`). When passed an untrusted HTML string, they parse and execute embedded scripts, making them XSS sinks.

**Technical Definition**
All of these methods accept HTML strings and use the same underlying HTML‑parsing machinery as `.html()`. The OWASP DOM XSS guidance explicitly lists `.after()`, `.append()`, `.prepend()`, `.before()`, and `.replaceWith()` as methods that directly update the DOM and can execute JavaScript. The risk is identical to `.html()`: untrusted data is interpreted as markup. The safe alternatives are `.text()` for text insertion, or sanitisation with DOMPurify before insertion.

**Beginner‑Friendly Explanation**
These methods are variations on “put content here.” `.append()` adds inside the element at the end; `.prepend()` at the beginning; `.after()` and `.before()` add siblings; `.replaceWith()` swaps the element out. They are all useful — but they all treat strings as HTML. If the string comes from a user, it can contain a script. The rule is the same as for `.html()`: use `.text()` if you just want to show text, or sanitise first if you genuinely need HTML.

---

### Purposes

- **To insert content in specific positions** relative to an element.
- **To understand that all HTML‑inserting methods share the same XSS risk**.
- **To choose the correct method** (`.text()` vs. sanitised `.append()`) based on whether markup is required.
- **To audit code for all HTML‑inserting sinks**, not just `.html()`.

---

### Syntax Rules and Structure

| Method | Position |
|---|---|
| `.append(content)` | Inside, at the end |
| `.prepend(content)` | Inside, at the beginning |
| `.after(content)` | Immediately after the element |
| `.before(content)` | Immediately before the element |
| `.replaceWith(content)` | Replaces the element |

**All accept HTML strings and are therefore unsafe with untrusted data.**

---

### Constraints and Limitations

- **None of these methods escape their arguments**; they are HTML parsers.
- **The safe alternative depends on the goal**: `.text()` for text, sanitisation for HTML, DOM construction for dynamic elements.

---

### Annotated Code Example

```javascript
// ❌ XSS: untrusted string passed to .append()
var userInput = "<script>alert('XSS')</script>";
$("#container").append(userInput);         // script executes

// ✅ SAFE: create a text node / use .text()
$("#container").append($("<div>").text(userInput)); // displayed as text

// ✅ SAFE: sanitise before appending HTML
var clean = DOMPurify.sanitize(userInput, { SAFE_FOR_JQUERY: true });
$("#container").append(clean);
```

**Expected Output**
The unsafe version executes the script. The `.text()` version displays the literal string. The DOMPurify version removes the `<script>` tag and appends only safe content.

**Why this output occurs**: `.append()` parses the string as HTML. `$("<div>").text(userInput)` creates an element and sets its text content, treating the input as data. DOMPurify sanitises the string before it reaches the HTML parser.

---

### Real‑World Cases

- **Dynamic lists**: Appending user‑submitted items without encoding.
- **Chat messages**: `.append()` with message content from the server.
- **Notifications**: `.prepend()` with untrusted notification text.

---

## Core Concept 7: `.text()` — Safe Sink

### Definitions

**Core Definition**
`.text()` is a jQuery method that sets or returns the text content of an element, treating its argument as plain text and never parsing it as HTML. It is the primary safe alternative to `.html()` and the HTML‑inserting methods when displaying untrusted data.

**Technical Definition**
`.text(textString)` sets the text content of each element in the jQuery set. The string is **not parsed as HTML**; angle brackets, quotes, and other markup characters are rendered as literal text (or escaped by the browser as necessary). `.text()` does not execute scripts and does not create elements from markup. It is therefore the recommended method for displaying user‑generated content that should not be interpreted as HTML.

**Beginner‑Friendly Explanation**
`.text()` is the “show this as plain text” method. If the text contains `<script>` tags, they will be displayed on the page as visible text — not executed. It is the safest way to display anything that came from a user. Use `.text()` whenever you want to show data, and reserve `.html()` and its relatives for trusted markup.

---

### Purposes

- **To display untrusted text safely** without executing scripts.
- **To avoid the need for manual escaping** when the goal is plain text display.
- **To serve as the default safe replacement** for `.html()` in most user‑content scenarios.
- **To demonstrate that safety does not require complex escaping** — the right method is often the simplest one.

---

### Syntax Rules and Structure

```
$( selector ).text( textString )
$( selector ).text()                 // getter — returns text content
```

**`textString` is treated as data, never as HTML.**

---

### Constraints and Limitations

- **`.text()` cannot insert markup**; if HTML is required, use sanitisation.
- **`.text()` replaces all existing content** with a single text node.
- **Performance is excellent**; there is no HTML parsing overhead.

---

### Annotated Code Example

```javascript
// ✅ SAFE: .text() displays untrusted data as plain text
var userInput = "<b>Bold?</b> <script>alert('XSS')</script>";
$("#output").text(userInput);
// Displays: <b>Bold?</b> <script>alert('XSS')</script>   (literal text)
```

**Expected Output**
The literal string is displayed; no bold formatting is applied and no script executes.

**Why this output occurs**: `.text()` sets the text content of the element. The browser renders the characters as text, not as markup, so `<b>` is not interpreted as a bold tag and `<script>` is not executed.

---

### Real‑World Cases

- **Displaying usernames, comments, and messages**.
- **Showing error messages** that may contain user input.
- **Rendering API data** that should be treated as text.

---

## Core Concept 8: `.val()` — Safe Sink

### Definitions

**Core Definition**
`.val()` is a jQuery method that sets or returns the value of form elements (input, select, textarea). When used as a setter, it sets the element's `value` property via the DOM, not via HTML parsing, and is therefore safe from XSS for ordinary form fields.

**Technical Definition**
`.val(value)` sets the `value` property of the underlying DOM element. This is a DOM property assignment, not an HTML string assignment. The browser treats the value as data for the form control and does not parse it as markup. The Stack Overflow consensus and OWASP guidance confirm that `.val()` is safe for setting form field values with untrusted data. However, `.val()` should not be used to set the `value` of an element in a context where the value is later interpreted as HTML.

**Beginner‑Friendly Explanation**
`.val()` is for form fields — text inputs, textareas, dropdowns. It sets what the user sees inside the field. Because it sets a DOM property rather than injecting HTML, it cannot execute scripts. If you are putting user data into an input field, `.val()` is safe.

---

### Purposes

- **To safely populate form fields** with untrusted data.
- **To read user input** from form controls.
- **To distinguish form‑value setting from HTML‑content setting** in security reviews.

---

### Syntax Rules and Structure

```
$( selector ).val( value )
$( selector ).val()                   // getter — returns current value
```

**`value` is assigned to the DOM `value` property; no HTML parsing occurs.**

---

### Constraints and Limitations

- **`.val()` is for form elements only**; it does not make sense for arbitrary elements.
- **`.val()` does not sanitise the content** for later use; it only sets the form value safely.
- **If the value is later inserted into the page with `.html()`**, the safety is lost.

---

### Annotated Code Example

```javascript
// ✅ SAFE: .val() sets a form field value
var untrusted = "<script>alert('XSS')</script>";
$("#username").val(untrusted);          // value is set as data
console.log($("#username").val());       // "<script>alert('XSS')</script>"
```

**Expected Output**
The input field displays the literal string `<script>alert('XSS')</script>`; no script executes.

**Why this output occurs**: `.val()` sets the `value` property of the input element. The browser treats the value as a string for the form control and does not parse it as HTML.

---

### Real‑World Cases

- **Pre‑filling edit forms** with data from a database.
- **Setting default values** from URL parameters.
- **Populating hidden fields** with tokens or identifiers.

---

## Core Concept 9: `.attr()` — Context‑Dependent Sink

### Definitions

**Core Definition**
`.attr()` is a jQuery method that gets or sets attributes on elements. Its safety depends entirely on **which attribute** is being set: ordinary attributes (e.g., `title`, `class`, `data-*`) are safe with untrusted data, but URL‑context attributes (`href`, `src`, `action`) and event‑handler attributes (`onclick`, `onerror`) are dangerous.

**Technical Definition**
`.attr(name, value)` sets the attribute `name` to `value` on each element. If `name` is a URL attribute (e.g., `href`, `src`), an attacker can supply a `javascript:` URL that executes when the user clicks or when the browser loads the resource. If `name` is an event‑handler attribute (e.g., `onclick`), the value is treated as JavaScript code and executes on the corresponding event. Setting non‑URL, non‑event attributes (e.g., `title`, `alt`, `class`) with untrusted strings is generally safe.

**Beginner‑Friendly Explanation**
`.attr()` is like setting a property on an HTML tag. If you set the `title` attribute, the browser displays a tooltip — safe. If you set the `href` attribute and the user clicks the link, the browser navigates to that URL — so if the URL starts with `javascript:`, it executes code. If you set `onclick`, you are literally writing JavaScript into the tag. The rule is: never set URL attributes or event‑handler attributes with untrusted data.

---

### Purposes

- **To set ordinary attributes safely** with untrusted data.
- **To understand the specific attributes that create XSS risk**.
- **To choose safe alternatives** (e.g., `.text()` for link text, URL validation for `href`).
- **To audit `.attr()` calls** for dangerous attribute names.

---

### Syntax Rules and Structure

| Attribute Type | Safety | Recommendation |
|---|---|---|
| `title`, `alt`, `class`, `data-*` | Safe | Use `.attr()` freely with untrusted data. |
| `href`, `src`, `action` | Unsafe | Validate URL scheme (allow only `https:`, `http:`, `mailto:`, etc.); reject `javascript:`. |
| `onclick`, `onerror`, `onload` | Unsafe | Never set event‑handler attributes with untrusted data; use `.on()` instead. |

---

### Constraints and Limitations

- **`.attr()` does not validate the attribute name**; the developer must know which attributes are dangerous.
- **`javascript:` URLs are the primary URL‑context threat**.
- **Event‑handler attributes should be avoided entirely** in modern code; use `.on()` for event binding.

---

### Annotated Code Example

```javascript
// ✅ SAFE: ordinary attribute
$("#tooltip").attr("title", untrustedText);   // tooltip displays as text

// ❌ UNSAFE: URL attribute with untrusted data
$("#link").attr("href", untrustedUrl);         // javascript: URLs execute on click

// ✅ SAFE: validate URL before setting href
function isSafeUrl(url) {
  return /^https?:\/\//i.test(url) || /^mailto:/i.test(url);
}
if (isSafeUrl(untrustedUrl)) {
  $("#link").attr("href", untrustedUrl);
} else {
  $("#link").removeAttr("href");
}
```

**Expected Output**
The ordinary attribute is set safely. The unsafe URL attribute can execute a `javascript:` payload. The validated version only sets `href` for safe URL schemes.

**Why this output occurs**: `title` is a display‑only attribute. `href` is a navigation attribute; a `javascript:` URL executes code when the link is clicked. Validation rejects dangerous schemes.

---

### Real‑World Cases

- **Profile links**: `href` set from user‑provided website URLs.
- **Image galleries**: `src` set from untrusted image URLs.
- **Dynamic buttons**: `onclick` set instead of using `.on()`.

---

## Core Concept 10: Output Encoding

### Definitions

**Core Definition**
Output encoding is the process of converting untrusted data into a safe representation before inserting it into an HTML document, so that the browser interprets it as data rather than executable code.

**Technical Definition**
Output encoding transforms characters that have special meaning in a given context — such as `<`, `>`, `&`, `"`, and `'` in HTML — into their corresponding entities (`&lt;`, `&gt;`, `&amp;`, `&quot;`, `&#x27;`). The encoding must be **context‑specific**: HTML entity encoding for HTML body content, attribute encoding for HTML attributes, JavaScript encoding for script blocks, CSS encoding for styles, and URL encoding for URL parameters. Encoding is applied **at the point of output**, immediately before the data is written to the page, not at input time.

**Beginner‑Friendly Explanation**
Output encoding is like putting a disguise on dangerous characters so the browser does not recognise them as instructions. If you encode `<` as `&lt;`, the browser displays a literal less‑than sign instead of starting a tag. The key is to encode in the right way for the right place: HTML content needs HTML encoding, an attribute needs attribute encoding, and a URL needs URL encoding. If you use the safe jQuery methods (`.text()`, `.val()`), the browser handles the encoding for you. When you must use HTML‑inserting methods, encoding is the primary defence.

---

### Purposes

- **To neutralise untrusted data at the sink** so it cannot break out of its intended context.
- **To prevent both reflected and stored XSS** when applied at the server and client.
- **To provide a context‑appropriate transformation** for each insertion point.
- **To complement input validation** — encoding handles the “output” half of the defence.

---

### Syntax Rules and Structure

| Context | Encoding |
|---|---|
| HTML body | HTML entity encode `<`, `>`, `&`, `"`, `'` |
| HTML attribute | Attribute encode all non‑alphanumeric characters |
| JavaScript string | JavaScript encode `<` as `\u003C`, etc. |
| URL parameter | URL encode (`encodeURIComponent`) |
| CSS | CSS encode |

**jQuery safe methods handle encoding for you**: `.text()`, `.val()`.

---

### Constraints and Limitations

- **Encoding must match the context**; HTML encoding in a JavaScript string does not prevent XSS.
- **Encoding must be applied at output**, not stored encoded in the database.
- **Client‑side encoding can be bypassed**; server‑side encoding is mandatory.

---

### Annotated Code Example (Server‑Side Conceptual)

```javascript
// ❌ NO ENCODING
res.send("<div>" + userComment + "</div>");

// ✅ HTML ENTITY ENCODING
function escapeHtml(str) {
  return str.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;")
            .replace(/"/g, "&quot;")
            .replace(/'/g, "&#x27;");
}
res.send("<div>" + escapeHtml(userComment) + "</div>");
```

**Expected Output**
The encoded version renders the comment as text, even if it contains `<script>` tags.

**Why this output occurs**: The `<` and `>` characters are replaced with `&lt;` and `&gt;`, so the browser does not recognise them as the start and end of an HTML tag.

---

### Real‑World Cases

- **Server‑side templates**: Auto‑escaping template engines (e.g., Jinja2, ERB with `<%= %>`) apply HTML encoding by default.
- **API responses**: JSON encoding prevents script injection when data is later rendered.
- **Client‑side rendering**: Use `.text()` instead of `.html()`.

---

## Core Concept 11: Input Validation

### Definitions

**Core Definition**
Input validation is the process of ensuring that data received by an application conforms to expected syntax, format, type, and length before it is processed, stored, or used.

**Technical Definition**
Input validation enforces **syntactic and semantic correctness** at trust boundaries. It uses allowlists (permitted characters, formats, lengths) rather than blacklists, and rejects data that does not conform. However, OWASP and PortSwigger emphasise that **input validation is not a substitute for output encoding**; XSS is prevented primarily by context‑aware output encoding, while input validation reduces the attack surface and improves data quality. Client‑side validation improves UX but must be backed by server‑side enforcement.

**Beginner‑Friendly Explanation**
Input validation is checking that what the user typed makes sense. If a field expects an email address, validation rejects anything that does not look like an email. If a username should only contain letters and numbers, validation rejects symbols. This reduces the amount of dangerous data that enters the system, but it does not make output encoding unnecessary — a valid input can still be dangerous if displayed unsafely.

---

### Purposes

- **To enforce expected data formats** at the boundary.
- **To reduce the attack surface** by rejecting malformed or unexpected input.
- **To improve data quality** and prevent downstream errors.
- **To complement output encoding**, not replace it.

---

### Syntax Rules and Structure

```javascript
// Client-side validation (UX layer)
function isValidUsername(u) {
  return /^[a-zA-Z0-9_]{3,20}$/.test(u);   // allowlist
}

// Server-side enforcement (mandatory)
app.post("/register", function (req, res) {
  if (!isValidUsername(req.body.username)) {
    return res.status(400).send("Invalid username");
  }
  // proceed
});
```

---

### Constraints and Limitations

- **Client‑side validation can be bypassed**; server‑side enforcement is mandatory.
- **Validation alone does not prevent XSS**; output encoding is still required.
- **Blacklists are fragile**; prefer allowlists.

---

### Annotated Code Example

```javascript
// ✅ INPUT VALIDATION (allowlist)
function validateComment(text) {
  if (text.length > 1000) return false;
  if (!/^[a-zA-Z0-9\s.,!?'-]*$/.test(text)) return false;  // restrict characters
  return true;
}

// But validation does not make .html() safe:
if (validateComment(userText)) {
  $("#comment").text(userText);    // still use safe output method
}
```

**Expected Output**
Only comments matching the allowlist are accepted. The comment is then displayed with `.text()`, providing defence in depth.

**Why this output occurs**: Validation restricts the input to a safe character set, but output encoding (via `.text()`) ensures that even unexpected characters cannot execute.

---

### Real‑World Cases

- **Registration forms**: Username and email format validation.
- **Comment systems**: Length limits and character allowlists.
- **API endpoints**: Schema validation of JSON payloads.

---

## Core Concept 12: Sanitisation

### Definitions

**Core Definition**
Sanitisation is the process of cleaning HTML input by removing or neutralising dangerous elements and attributes while preserving safe markup, typically using an allowlist‑based library.

**Technical Definition**
Sanitisation parses HTML and applies an allowlist policy: permitted tags (e.g., `<b>`, `<i>`, `<p>`, `<a>`) and attributes (e.g., `href` with safe schemes) are retained, while dangerous constructs (`<script>`, event‑handler attributes, `javascript:` URLs, `<iframe>`, `<object>`) are removed or neutralised. Sanitisation is necessary when the application must accept rich HTML (e.g., WYSIWYG editors, comments with formatting) but must prevent XSS. **DOMPurify** is the recommended client‑side sanitiser. Server‑side sanitisation is also required for stored content.

**Beginner‑Friendly Explanation**
Sanitisation is like a security guard at the door of an HTML party. The guard has a guest list (the allowlist). Safe tags like `<b>` and `<i>` are allowed in. Dangerous tags like `<script>` and suspicious characters are turned away. If you must let users submit HTML — for example, in a rich‑text editor — you sanitise it before displaying it. DOMPurify is the most trusted guard.

---

### Purposes

- **To allow safe HTML** while removing dangerous constructs.
- **To protect rich‑text and user‑generated content** from XSS.
- **To provide a defence when `.html()` or similar methods must be used**.
- **To enforce a clear, auditable allowlist policy**.

---

### Syntax Rules and Structure

```javascript
// Client-side sanitisation with DOMPurify
var clean = DOMPurify.sanitize(dirtyHtml, {
  SAFE_FOR_JQUERY: true             // required for .html() compatibility
});
$("#output").html(clean);
```

**Server‑side sanitisation** should use an equivalent library (e.g., OWASP Java HTML Sanitizer, Bleach for Python).

---

### Constraints and Limitations

- **Sanitisation is complex**; custom regex‑based sanitisers are error‑prone.
- **The SAFE_FOR_JQUERY flag is required** when the sanitised output will be passed to jQuery's `.html()`.
- **Sanitisation must be kept up to date**; new bypasses are discovered regularly.

---

### Annotated Code Example

```javascript
// ✅ SANITISE BEFORE INSERTING RICH HTML
var userHtml = "<p>Hello <b>world</b></p><script>alert('XSS')</script>";
var clean = DOMPurify.sanitize(userHtml, { SAFE_FOR_JQUERY: true });
$("#content").html(clean);
// Result: <p>Hello <b>world</b></p>  (script removed)
```

**Expected Output**
The safe markup (`<p>`, `<b>`) is preserved; the `<script>` tag is removed.

**Why this output occurs**: DOMPurify parses the HTML, applies its allowlist, and strips disallowed elements and attributes. The `SAFE_FOR_JQUERY` flag ensures compatibility with jQuery's HTML parsing.

---

### Real‑World Cases

- **Rich‑text editors**: Comments, forum posts, and messaging content.
- **Markdown rendering**: Sanitising the HTML output of a Markdown parser.
- **CMS content**: User‑submitted articles with formatting.

---

## Core Concept 13: DOMPurify

### Definitions

**Core Definition**
DOMPurify is a DOM‑only, fast, and tolerant XSS sanitiser for HTML, MathML, and SVG, written in JavaScript, that removes dangerous content according to a configurable allowlist and is the recommended client‑side sanitiser for jQuery applications.

**Technical Definition**
DOMPurify sanitises HTML by parsing it in a detached DOM, traversing the resulting tree, and removing elements and attributes that are not on its allowlist. It is designed by security professionals with deep XSS expertise and is used by major organisations. When the sanitised output will be passed to jQuery's `.html()` or similar methods, the **`SAFE_FOR_JQUERY`** flag must be set to ensure compatibility and to prevent jQuery‑specific bypasses. DOMPurify can be included via a `<script>` tag or as an npm module.

**Beginner‑Friendly Explanation**
DOMPurify is the industry‑standard tool for cleaning HTML. It is fast, well‑tested, and designed specifically to stop XSS. If you need to accept HTML from users — for example, in a comment with bold and italic formatting — you run it through DOMPurify first. The `SAFE_FOR_JQUERY` option is a special setting that makes the cleaned HTML safe specifically for jQuery's parsing behaviour. Without it, certain edge cases could still be dangerous when passed to `.html()`.

---

### Purposes

- **To provide a reliable, vetted sanitiser** for rich HTML.
- **To handle the complexity of HTML parsing and allowlisting** so developers do not have to write custom filters.
- **To enable safe use of `.html()` and HTML‑inserting methods** when markup is genuinely required.
- **To serve as a defence‑in‑depth measure** alongside output encoding and input validation.

---

### Syntax Rules and Structure

```javascript
// Include DOMPurify (script tag or npm)
// <script src="https://cdn.jsdelivr.net/npm/dompurify@3/dist/purify.min.js"></script>

var clean = DOMPurify.sanitize(dirty, {
  SAFE_FOR_JQUERY: true
});
```

**Key options**: `ALLOWED_TAGS`, `ALLOWED_ATTR`, `FORBID_TAGS`, `FORBID_ATTR`, `SAFE_FOR_JQUERY`.

---

### Constraints and Limitations

- **`SAFE_FOR_JQUERY` is required** when using jQuery's `.html()`; without it, jQuery‑specific bypasses may exist.
- **DOMPurify runs in the browser**, so server‑side sanitisation is still required for stored content.
- **Configuration should be reviewed** to ensure the allowlist matches the application's needs.

---

### Annotated Code Example

```javascript
// ✅ DOMPURIFY WITH SAFE_FOR_JQUERY
var dirty = '<p onclick="alert(1)">Click</p><script>alert(2)</script>';
var clean = DOMPurify.sanitize(dirty, { SAFE_FOR_JQUERY: true });
$("#output").html(clean);
// Result: <p>Click</p>
```

**Expected Output**
The `onclick` attribute and `<script>` tag are removed; the safe `<p>` element is preserved.

**Why this output occurs**: DOMPurify's allowlist permits `<p>` but forbids event‑handler attributes and `<script>`. The `SAFE_FOR_JQUERY` flag ensures the output is safe for jQuery's `.html()` parsing.

---

### Real‑World Cases

- **Comment systems with formatting**: DOMPurify allows `<b>`, `<i>`, `<a>` while removing scripts.
- **Markdown rendering**: DOMPurify sanitises the HTML generated from user Markdown.
- **CMS and WYSIWYG editors**: Server‑side and client‑side sanitisation of rich content.

---

## Core Concept 14: XSS Prevention Best Practices

### Definitions

**Core Definition**
jQuery XSS prevention best practices are the consolidated conventions — prefer safe APIs, encode at output, validate at input, sanitise rich HTML, use DOMPurify with `SAFE_FOR_JQUERY`, deploy CSP, and never trust client‑side validation as security — that systematically eliminate the conditions for XSS in jQuery applications.

**Technical Definition**
Best practices synthesise the individual techniques: (1) **use `.text()` and `.val()`** for untrusted data, reserving `.html()` and its relatives for trusted markup; (2) **encode output contextually** at the server and client; (3) **validate input on the server** with allowlists, treating client‑side validation as UX only; (4) **sanitise rich HTML with DOMPurify** using the `SAFE_FOR_JQUERY` flag before passing it to jQuery methods; (5) **avoid setting URL and event‑handler attributes** with untrusted data via `.attr()`; (6) **deploy a Content Security Policy (CSP)** as defence in depth, restricting `script-src` and avoiding `unsafe-inline`; (7) **keep jQuery and libraries updated**, as historical versions have contained XSS vulnerabilities; and (8) **audit code for unsafe sinks** using automated scanners and manual review.

**Beginner‑Friendly Explanation**
Best practices are the habits that keep XSS out. Use `.text()` and `.val()` by default — they are safe. Only use `.html()` when you genuinely need HTML, and then sanitise it with DOMPurify. Validate input on the server, not just the browser. Never set `href` or `onclick` with user data. Add a CSP as a safety net. Keep your libraries up to date. And if you are reviewing code, look for `.html()`, `.append()`, and `.attr()` with untrusted data — those are the red flags.

---

### Purposes

- **To provide a single actionable checklist** for XSS‑safe jQuery development.
- **To prevent the most common XSS vulnerabilities** identified by OWASP.
- **To combine multiple layers of defence** (safe APIs, encoding, validation, sanitisation, CSP).
- **To establish a baseline for code reviews and security audits**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Text display** | `.text(untrusted)` | `.html(untrusted)` |
| **Form values** | `.val(untrusted)` | `.attr("value", untrusted)` |
| **Rich HTML** | `DOMPurify.sanitize(html, { SAFE_FOR_JQUERY: true })` | Raw `.html()` with user input |
| **URL attributes** | Validate scheme; allow only `https`, `http`, `mailto` | `.attr("href", untrusted)` |
| **Event handlers** | `.on("click", handler)` | `.attr("onclick", untrusted)` |
| **Input validation** | Server‑side allowlist + client UX | Client‑side only |
| **Defence in depth** | CSP with `script-src` restrictions | No CSP or `unsafe-inline` |
| **Dependencies** | Keep jQuery and libraries updated | Pinned, outdated versions |

---

### Constraints and Limitations

- **Best practices are context‑dependent**; a static page has different needs from a banking application.
- **CSP is defence in depth**, not a primary control; it reduces impact but does not eliminate XSS.
- **Sanitisation is complex**; rely on vetted libraries, not custom regex.

---

### Annotated Code Example (Consolidated)

```javascript
// ✅ XSS PREVENTION BEST PRACTICES — CONSOLIDATED

// 1. Prefer safe APIs for untrusted text
var userComment = "<script>alert('XSS')</script>";
$("#comment").text(userComment);                    // safe

// 2. Use .val() for form fields
$("#username").val(userComment);                     // safe

// 3. Sanitise rich HTML with DOMPurify
var richHtml = "<p>Hello <b>world</b></p><script>alert(1)</script>";
var clean = DOMPurify.sanitize(richHtml, { SAFE_FOR_JQUERY: true });
$("#richContent").html(clean);                       // safe HTML

// 4. Validate URLs before setting href
function isSafeUrl(url) {
  return /^https?:\/\//i.test(url) || /^mailto:/i.test(url);
}
if (isSafeUrl(userUrl)) {
  $("#link").attr("href", userUrl);                  // safe
}

// 5. Server-side validation and encoding (conceptual)
// Always enforce validation and encoding on the server as well.
```

**Expected Output**
The comment and username are displayed safely. The rich HTML is sanitised, preserving safe markup while removing the script. The URL is set only if it uses a safe scheme. Each practice corresponds to a specific XSS defence.

**Why this output occurs**: `.text()` and `.val()` treat input as data. DOMPurify removes dangerous constructs. URL validation rejects `javascript:` schemes. Together, these practices break the source‑to‑sink flow that XSS requires.

---

### Real‑World Cases

- **Code reviews**: A checklist ensures that new jQuery code follows XSS‑safe patterns.
- **Legacy refactoring**: Identifying `.html()` with user input and replacing it with `.text()` or sanitised `.html()`.
- **CI/CD pipelines**: Automated scanning for unsafe sinks (e.g., Semgrep rules for `jquery-insecure-selector`).

---

## References

- OWASP XSS (Cross Site Scripting) Prevention Cheat Sheet — https://wiki.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet
- OWASP DOM based XSS Prevention Cheat Sheet — https://wiki.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet
- OWASP: Unraveling some of the Mysteries around DOM-based XSS — https://wiki.owasp.org/index.php/Unraveling_some_of_the_Mysteries_around_DOM-based_XSS
- OWASP: XSS Filter Evasion Cheat Sheet — https://wiki.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet
- DOMPurify — https://www.npmjs.com/package/dompurify
- DOMPurify GitHub Repository — https://github.com/cure53/DOMPurify
- PortSwigger: How to prevent XSS — https://portswigger.net/web-security/cross-site-scripting/preventing
- Snyk: jQuery XSS vulnerabilities and remediation — https://snyk.io
- EdgeScan: jQuery API's and XSS — https://www.edgescan.com/wp-content/uploads/2018/08/04.-XSS-and-Encoding-edgescan.pdf
- jQuery Frontend Security Spec (jQuery 4.0.x, modern browsers) — https://github.com/tech-leads-club/agent-skills/blob/main/packages/skills-catalog/skills/(security)/security-best-practices/references/javascript-jquery-web-frontend-security.md
- Unsafe HTML sinks and Desk DOM XSS — https://github.com/frappe/skills/blob/d213dce39942c969293e9fe7feb96ab33cbba25c/skills/deep-app-audit/security/C-xss-client/C02-unsafe-html-sinks.md
- Telerik: Security and Validation Useful Tips (Kendo UI for jQuery) — https://www.telerik.com/kendo-jquery-ui/documentation/security/security-and-validation
- Semgrep rule: jquery-insecure-selector — https://github.com/semgrep/semgrep-rules/blob/develop/javascript/jquery/security/audit/jquery-insecure-selector.yaml
- MDN: Cross-site scripting (XSS) — https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting