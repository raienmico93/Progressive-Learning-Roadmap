# jQuery XSS Prevention — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** jQuery XSS Prevention is the discipline of safely handling untrusted data when using jQuery's DOM manipulation methods, ensuring that user-supplied content is never interpreted as executable HTML, JavaScript, or other active content by the browser.

**Technical Definition:** Cross-Site Scripting (XSS) is a vulnerability class in which an attacker injects malicious scripts into content that is then rendered and executed in a victim's browser. In jQuery, XSS vulnerabilities arise when untrusted data is passed to DOM manipulation methods — `.html()`, `.append()`, `.after()`, `.before()`, `.prepend()`, `.replaceWith()`, `$(htmlString)`, and others — that parse their input as HTML. jQuery's documentation explicitly states: "By design, any jQuery constructor or method that accepts an HTML string — jQuery(), .append(), .after(), etc. — can potentially execute code. This can occur by injection of script tags or use of HTML attributes that execute code (for example, `<img onload="">`). Do not use these methods to insert strings obtained from untrusted sources such as URL query parameters, cookies, or form inputs. Doing so can introduce cross-site-scripting (XSS) vulnerabilities". The primary defenses are: use `.text()` instead of `.html()` for untrusted text; perform contextual output encoding; implement Content Security Policy (CSP); use the Trusted Types API; and validate URL attributes to block `javascript:` schemes.

**Beginner-Friendly Explanation:** XSS is what happens when a malicious user enters code instead of text, and your website accidentally runs that code. Imagine a guestbook where someone writes `<script>stealPasswords()</script>` as their message. If you display that message with `.html()`, the browser thinks it is a real script and runs it. If you use `.text()`, the browser shows the literal text, and the script is never executed. This cheat sheet is about using the safe methods and understanding why the unsafe ones are dangerous.

### Key Characteristics

- **`.text()` is safe by default:** It escapes HTML characters so they are displayed as literal text, never parsed as markup.
- **`.html()` is a dangerous sink:** It parses its input as HTML, allowing script execution and event-handler injection.
- **The jQuery constructor `$()` is also a sink:** Passing an HTML string to `$()` immediately evaluates it.
- **Output encoding must be contextual:** Data placed in HTML body, HTML attributes, JavaScript, CSS, and URLs each require different encoding rules.
- **CSP is a defense-in-depth layer:** Content Security Policy blocks inline script execution and restricts script sources, mitigating XSS even if encoding fails.
- **Trusted Types enforce object-based policies:** The Trusted Types API requires that data passed to dangerous sinks be wrapped in a `TrustedHTML` object, preventing string-based injection.
- **URL attributes need scheme validation:** `.attr("href", userInput)` and `.attr("src", userInput)` can execute `javascript:` URIs if not validated.

### Prerequisites

- Proficiency in jQuery fundamentals: `.html()`, `.text()`, `.append()`, `.attr()`, and the `$()` constructor.
- Understanding of the browser's HTML parsing and JavaScript execution model.
- Familiarity with OWASP's XSS Prevention Cheat Sheet and contextual output encoding.
- Awareness of CSP headers and their directives.

### Related Programming Areas

- **Web Security (OWASP Top 10):** XSS is consistently one of the most critical web application security risks.
- **Content Security Policy (CSP):** A browser security mechanism that restricts resource loading and script execution.
- **Trusted Types API:** A browser API that prevents DOM-based XSS by enforcing typed policies.
- **Output Encoding:** The process of converting special characters to their HTML entity equivalents.
- **DOM Manipulation:** The jQuery methods that can introduce XSS if used with untrusted data.

### Core Concepts / Features

This cheat sheet covers five core concepts and two enhanced topics: unsafe HTML insertion, `.html()` considerations, `.text()` for untrusted text, output encoding, CSP, Trusted Types API integration, and safe attribute modification.

---

## Core Concept 1: Unsafe HTML Insertion — Avoiding Raw String Concatenation with Unvalidated User Inputs

### Definitions

**Core Definition:** Unsafe HTML insertion is the practice of passing unvalidated user input to jQuery methods that parse HTML strings, allowing attackers to inject executable scripts or event handlers into the page.

**Technical Definition:** jQuery methods that accept HTML strings — `.html()`, `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()`, `.wrap()`, and the `$()` constructor itself — parse their input as HTML and insert the resulting DOM nodes into the document. If the input contains `<script>` tags or elements with event-handler attributes (e.g., `<img src=x onerror=alert(1)>`), the browser executes the associated code. jQuery's own documentation warns that these methods "can potentially execute code" and that developers should not use them to insert strings from "untrusted sources such as URL query parameters, cookies, or form inputs". Even `.append()` and similar methods are vulnerable: the EdgeScan jQuery XSS reference table lists `.html()`, `.append()`, `.after()`, `.before()`, `.prepend()`, `.insertAfter()`, `.insertBefore()`, and `$()` as methods that directly update the DOM and can execute JavaScript.

**Beginner-Friendly Explanation:** Think of your web page as a house. `.text()` is like putting a note on the wall — it stays a note no matter what is written on it. `.html()` is like letting a guest write on the wall with permanent marker — if the guest writes "open the door to strangers," the house might actually do it. Unsafe HTML insertion is what happens when you let untrusted guests write on the wall with permanent marker.

### Purposes

- To understand which jQuery methods are dangerous when used with untrusted data.
- To identify the specific attack vectors (script tags, event-handler attributes, `javascript:` URIs) that can be injected through HTML strings.
- To establish a baseline of safe coding practices for jQuery DOM manipulation.
- To recognize that the `$()` constructor is itself a dangerous sink when passed an HTML string.

### Syntax Rules and Structure

**Dangerous jQuery Methods (Sinks):**

| Method | Accepts HTML String | Executes Script | Safe Alternative |
|--------|---------------------|-----------------|------------------|
| `.html()` | Yes | Yes | `.text()` |
| `.append()` | Yes | Yes | `.append(document.createTextNode())` |
| `.prepend()` | Yes | Yes | `.prepend(document.createTextNode())` |
| `.after()` | Yes | Yes | `.after(document.createTextNode())` |
| `.before()` | Yes | Yes | `.before(document.createTextNode())` |
| `.replaceWith()` | Yes | Yes | `.replaceWith(document.createTextNode())` |
| `$()` | Yes | Yes | `$("<div>").text(str)` |
| `.text()` | No | No | — (safe) |
| `.val()` | No | No | — (safe) |

**Complete General Syntax (Unsafe — Avoid):**
```javascript
// DANGEROUS: user input is parsed as HTML
$("#output").html(userInput);
$("#list").append("<li>" + userInput + "</li>");
$("<div>" + userInput + "</div>").appendTo("#container");
```

**Complete General Syntax (Safe — Preferred):**
```javascript
// SAFE: user input is treated as text
$("#output").text(userInput);
$("#list").append($("<li>").text(userInput));
$("<div>").text(userInput).appendTo("#container");
```

**Syntax Rules:**

- Never pass strings from untrusted sources (URL parameters, form inputs, cookies, AJAX responses from third parties) to `.html()`, `.append()`, `.prepend()`, `.after()`, `.before()`, `.replaceWith()`, or `$()`.
- Use `.text()` for plain text content.
- When building DOM elements, create the element first and set its text with `.text()`: `$("<li>").text(userInput)`.
- If HTML must be inserted, sanitize it with DOMPurify using the `SAFE_FOR_JQUERY` option before passing it to jQuery.

**Constraints and Limitations:**

- `.text()` escapes HTML characters but does not decode HTML entities; it displays them literally.
- Some jQuery methods accept both HTML strings and DOM nodes; when in doubt, pass a DOM node instead of a string.
- The `$()` constructor evaluates HTML strings immediately upon creation, before the element is inserted into the DOM.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Unsafe `.append()` vs. Safe `.append()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Unsafe vs Safe Append Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <ul id="unsafeList"></ul>
  <ul id="safeList"></ul>
  
  <input type="text" id="userInput" value="<img src=x onerror=alert('XSS')>">

  <button id="appendUnsafe">Append Unsafe</button>
  <button id="appendSafe">Append Safe</button>

  <script>
    $(function() {
      // Step 1: Unsafe — user input is parsed as HTML
      $("#appendUnsafe").click(function() {
        var input = $("#userInput").val();
        $("#unsafeList").append("<li>" + input + "</li>");
        // The <img onerror> will execute, alerting 'XSS'
      });

      // Step 2: Safe — user input is treated as text
      $("#appendSafe").click(function() {
        var input = $("#userInput").val();
        $("#safeList").append($("<li>").text(input));
        // The input is displayed as literal text; no script executes
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Append Unsafe" triggers an alert box with "XSS" because the `<img onerror>` attribute executes. Clicking "Append Safe" displays the literal text `<img src=x onerror=alert('XSS')>` as a list item with no script execution.

**Why this output:** The unsafe append concatenates user input into an HTML string, which the browser parses and executes. The safe append creates a `<li>` element and sets its text content with `.text()`, which escapes the HTML characters so they are displayed literally.

### Real-World Cases

- **Comment systems:** Displaying user comments with `.text()` instead of `.html()` to prevent script injection.
- **User profiles:** Rendering usernames and bios as text content, never as HTML.
- **Chat applications:** Inserting chat messages using `.text()` or sanitized HTML.
- **Search results:** Displaying search query text with `.text()` to prevent reflected XSS.

---

## Core Concept 2: `.html()` Considerations — Treating `.html()` as a Dangerous XSS Sink

### Definitions

**Core Definition:** `.html()` is a jQuery method that sets or gets the inner HTML of an element. When used as a setter, it parses its argument as HTML and replaces the element's content, making it a dangerous sink for XSS if the argument contains untrusted data.

**Technical Definition:** The `.html()` method is functionally equivalent to setting `element.innerHTML`. Both parse their input as HTML, create DOM nodes, and insert them into the document. If the input contains `<script>` tags, the browser executes the script. If the input contains elements with event-handler attributes (e.g., `<img onload="">`), the browser executes the handler when the event fires. jQuery's documentation explicitly warns against using `.html()` with untrusted strings. A specific historical vulnerability, CVE-2020-11023, affected jQuery versions 1.0.3 through 3.4.1: passing HTML containing `<option>` elements from untrusted sources — even after sanitizing them — to `.html()`, `.append()`, or similar methods could execute untrusted code. This vulnerability was fixed in jQuery 3.5.0, and CISA added it to the Known Exploited Vulnerabilities catalog in January 2025.

**Beginner-Friendly Explanation:** `.html()` is like telling the browser "here is some HTML, please build it and put it on the page." If the HTML contains a script, the browser builds and runs it. `.text()` is like saying "here is some text, please display it." The browser displays it without running anything. Use `.text()` unless you absolutely need to insert HTML, and if you do need HTML, sanitize it first.

### Purposes

- To understand why `.html()` is inherently dangerous when used with untrusted data.
- To recognize the specific historical vulnerabilities (CVE-2020-11023) associated with `.html()`.
- To establish the policy of using `.text()` as the default and `.html()` only with sanitized input.
- To implement sanitization (DOMPurify with `SAFE_FOR_JQUERY`) when HTML insertion is unavoidable.

### Syntax Rules and Structure

**Complete General Syntax (Unsafe — Avoid with Untrusted Data):**
```javascript
$(selector).html(userInput);
```

**Complete General Syntax (Safe — Sanitize First):**
```javascript
// Sanitize with DOMPurify using SAFE_FOR_JQUERY
var clean = DOMPurify.sanitize(userInput, { SAFE_FOR_JQUERY: true });
$(selector).html(clean);
```

| Component | Description |
|-----------|-------------|
| `DOMPurify.sanitize(dirty, { SAFE_FOR_JQUERY: true })` | Removes dangerous HTML while preserving safe markup. |
| `SAFE_FOR_JQUERY` | Specifically designed to work around jQuery's HTML parsing quirks. |

**Syntax Rules:**

- Never pass untrusted data directly to `.html()`.
- If HTML insertion is required, sanitize the input with DOMPurify before passing it to `.html()`.
- Starting with jQuery 3.5.0, the `SAFE_FOR_JQUERY` option is no longer necessary for jQuery-specific bugs, but DOMPurify should still be used to sanitize HTML from users.
- Upgrade to jQuery 3.5.0 or later to benefit from the fix for CVE-2020-11023.

**Constraints and Limitations:**

- DOMPurify sanitizes HTML but may not preserve all formatting or attributes; test with your specific content.
- Sanitization adds processing overhead; for plain text, `.text()` is both safer and faster.
- The `SAFE_FOR_JQUERY` option is specific to older jQuery versions and should be used only when necessary.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Sanitizing HTML with DOMPurify Before `.html()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>DOMPurify .html() Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dompurify@3.4.15/dist/purify.min.js"></script>
</head>
<body>
  <div id="output"></div>
  <input type="text" id="userInput" value="<b>Bold</b><script>alert('XSS')</script>">
  <button id="render">Render HTML</button>

  <script>
    $(function() {
      $("#render").click(function() {
        var input = $("#userInput").val();

        // Step 1: Sanitize the input
        var clean = DOMPurify.sanitize(input, { SAFE_FOR_JQUERY: true });

        // Step 2: Insert the sanitized HTML
        $("#output").html(clean);
        // The <b> tag is preserved; the <script> tag is removed
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Render HTML" displays "Bold" in bold text. The `<script>` tag is removed, so no alert box appears.

**Why this output:** DOMPurify with `SAFE_FOR_JQUERY` removes dangerous elements and attributes while preserving safe formatting tags like `<b>`. The sanitized HTML is then safely inserted with `.html()`.

### Real-World Cases

- **Rich text editors:** Sanitizing user-authored HTML before displaying it with `.html()`.
- **CMS content rendering:** Sanitizing stored HTML from untrusted sources.
- **Email clients:** Sanitizing HTML email bodies before rendering.
- **Markdown renderers:** Converting Markdown to HTML and sanitizing the result.

---

## Core Concept 3: `.text()` for Untrusted Text — Using `.text()` to Force Browser Contextual Text Encoding

### Definitions

**Core Definition:** `.text()` is a jQuery method that sets or gets the text content of an element. When used as a setter, it converts its argument to a string and assigns it to the element's `textContent` property, automatically escaping HTML special characters so they are displayed as literal text rather than parsed as markup.

**Technical Definition:** The `.text()` method corresponds to the DOM `textContent` property. Unlike `.html()`, which parses its input as HTML, `.text()` treats its input as plain text. Any HTML tags in the input are escaped: `<` becomes `<`, `>` becomes `>`, `&` becomes `&`, and so on. This makes `.text()` inherently safe for inserting untrusted data, because the browser cannot execute escaped text as markup or script. The EdgeScan jQuery XSS reference table explicitly notes that `.text()` "updates DOM, but is safe". Stack Overflow answers confirm: "If you use jQuery's `.text(untrustedString)` method, you'll be fine. That method will escape any html or tags".

**Beginner-Friendly Explanation:** `.text()` is like putting a note inside a glass display case. Whatever you write on the note — even if it says "explode" — it stays a note. Nobody can read it as an instruction. `.html()` is like handing the note to someone who might follow whatever is written on it. Use `.text()` when you want to show user input as text.

### Purposes

- To safely insert untrusted text content into the DOM without the risk of XSS.
- To escape HTML special characters automatically, eliminating the need for manual encoding.
- To display user-generated content (names, comments, messages) exactly as the user typed it.
- To avoid the performance overhead of HTML parsing when only text content is needed.
- To comply with OWASP's recommendation to use contextually appropriate encoding for the HTML body context.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
$(selector).text( untrustedString );
```

| Component | Description |
|-----------|-------------|
| `$(selector)` | The element whose text content will be set. |
| `.text( untrustedString )` | Sets the element's `textContent` to the string, escaping HTML characters. |

**Syntax Rules:**

- `.text()` converts its argument to a string automatically; numbers, Booleans, and objects are coerced.
- HTML special characters (`<`, `>`, `&`, `"`, `'`) are escaped and displayed literally.
- `.text()` does not decode HTML entities; if the input contains `&amp;`, it is displayed as `&amp;`, not `&`.
- For building DOM elements with text content, use `$("<tag>").text(untrustedString)` rather than string concatenation.

**Constraints and Limitations:**

- `.text()` cannot insert HTML formatting; if bold text or links are needed, a different approach (sanitized `.html()` or DOM construction) is required.
- `.text()` replaces all child nodes of the target element, including any existing HTML content.
- `.text()` does not work on `<input>` or `<textarea>` elements; use `.val()` instead.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Displaying User Input with `.text()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>.text() Safety Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <input type="text" id="input" value="<script>alert('XSS')</script>">
  <button id="display">Display</button>
  <div id="output"></div>

  <script>
    $(function() {
      $("#display").click(function() {
        var userInput = $("#input").val();

        // Step 1: Use .text() to safely display the input
        $("#output").text(userInput);

        // The script tag is displayed as literal text, not executed
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Display" shows the literal text `<script>alert('XSS')</script>` in the output div. No alert box appears.

**Why this output:** `.text()` escapes the `<` and `>` characters, so the browser treats the input as plain text rather than parsing it as a script tag. The script is never executed.

### Real-World Cases

- **User comments:** Displaying comment text with `.text()` to prevent script injection.
- **Search results:** Showing the search query with `.text()` to prevent reflected XSS.
- **Profile information:** Rendering usernames, bios, and locations as text.
- **Error messages:** Displaying error text that may contain user input.

---

## Core Concept 4: Output Encoding — Escaping Special HTML Characters Before Rendering

### Definitions

**Core Definition:** Output encoding (also called output escaping) is the process of converting special characters in untrusted data to their corresponding HTML entity representations before inserting the data into the DOM, ensuring that the browser interprets the data as text rather than markup or code.

**Technical Definition:** OWASP's XSS Prevention Cheat Sheet defines a positive model for preventing XSS using contextual output encoding. The core rule is: "Escape the following characters with HTML entity encoding to prevent switching into any execution context, such as script, style, or event handlers". The specific characters and their encodings are: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`, `'` → `&#x27;`, `/` → `&#x2F;`. Different contexts require different encoding: HTML body content requires HTML entity encoding; HTML attributes require attribute encoding; JavaScript contexts require JavaScript escaping; URL contexts require URL encoding; and CSS contexts require CSS escaping. jQuery's `.text()` method performs HTML entity encoding automatically, making it the easiest way to implement output encoding for the HTML body context.

**Beginner-Friendly Explanation:** Output encoding is like putting on a disguise. You take a dangerous character — like `<` which could start an HTML tag — and disguise it as `&lt;`, which the browser reads as "the less-than symbol" instead of "start a tag." The dangerous character is still visible to the user, but the browser does not treat it as code.

### Purposes

- To prevent untrusted data from escaping its intended context and being interpreted as code.
- To comply with OWASP's XSS Prevention Cheat Sheet, which recommends contextual output encoding as the primary defense against XSS.
- To handle data that must be inserted in different contexts (HTML body, attributes, JavaScript, URLs) with the appropriate encoding scheme.
- To provide a defense-in-depth layer even when other security measures (CSP, Trusted Types) are in place.
- To ensure that data is rendered correctly and safely regardless of its source.

### Syntax Rules and Structure

**Complete General Syntax (Manual HTML Entity Encoding):**
```javascript
function encodeHTML(str) {
    return String(str)
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#x27;")
        .replace(/\//g, "&#x2F;");
}
```

**Context-Specific Encoding Rules:**

| Context | Encoding Scheme | Example |
|---------|----------------|---------|
| HTML Body | HTML Entity Encoding | `&amp;`, `&lt;`, `&gt;`, `&quot;`, `&#x27;`, `&#x2F;` |
| HTML Attribute | HTML Attribute Encoding | All non-alphanumeric < 256 → `&#xHH;` |
| JavaScript | JavaScript Escaping | All non-alphanumeric < 256 → `\xHH` |
| URL | URL Encoding | All non-alphanumeric < 256 → `%HH` |
| CSS | CSS Escaping | All non-alphanumeric < 256 → `\HH` |

**Syntax Rules:**

- Encode on output, not on input; encoding at the last moment before inserting data into the DOM ensures that the data is safe in its specific context.
- Use different encoding for different contexts; HTML entity encoding is not sufficient for JavaScript or URL contexts.
- Do not use shortcuts like escaping only quotes; the quote character may be matched by the HTML attribute parser which runs first.
- For HTML body content, jQuery's `.text()` performs the necessary encoding automatically.

**Constraints and Limitations:**

- Manual encoding is error-prone; use `.text()` or a proven library (DOMPurify) whenever possible.
- Double-encoding can occur if data is encoded more than once; encode exactly once, at the point of output.
- Encoding does not prevent all XSS vectors; some attacks use character encodings or browser quirks that bypass simple escaping.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Contextual Encoding for an HTML Attribute**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Contextual Encoding Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <input type="text" id="userInput" value='"><script>alert(1)</script>'>
  <button id="apply">Apply</button>
  <a id="link" href="#">Link</a>

  <script>
    $(function() {
      $("#apply").click(function() {
        var input = $("#userInput").val();

        // Step 1: Encode for HTML attribute context
        var encoded = String(input)
          .replace(/&/g, "&amp;")
          .replace(/"/g, "&quot;")
          .replace(/'/g, "&#x27;")
          .replace(/</g, "&lt;")
          .replace(/>/g, "&gt;");

        // Step 2: Set the title attribute with encoded data
        $("#link").attr("title", encoded);
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** The link's `title` attribute contains the encoded string. Hovering over the link shows the literal text `"><script>alert(1)</script>` without executing the script.

**Why this output:** The encoding converts `"` to `&quot;` and `<` to `&lt;`, preventing the input from breaking out of the attribute context. The browser treats the encoded characters as data, not as markup.

### Real-World Cases

- **URL parameter rendering:** Encoding URL parameters before inserting them into HTML attributes.
- **Form field values:** Encoding user input before placing it in `value` attributes.
- **Data attributes:** Encoding data passed to `data-*` attributes.
- **Internationalization:** Ensuring that special characters in translated strings are safely rendered.

---

## Core Concept 5: Content Security Policy (CSP) — Configuring Headers to Block Unauthorized Script Execution

### Definitions

**Core Definition:** Content Security Policy (CSP) is an HTTP response header that allows website administrators to restrict the sources from which scripts, styles, images, and other resources can be loaded, and to block inline script and style execution, thereby mitigating XSS attacks even if output encoding fails.

**Technical Definition:** CSP is implemented via the `Content-Security-Policy` HTTP header. The `script-src` directive specifies valid sources for JavaScript, including not only URLs loaded directly into `<script>` elements but also inline script event handlers (e.g., `onclick`) and XSLT stylesheets that can trigger script execution. When `script-src 'self'` is set, only scripts from the same origin are allowed; inline scripts (`<script>...</script>`) and inline event handlers are blocked. To allow inline scripts, developers must use a `nonce` (a cryptographically random token generated per request) or a `hash` (a SHA-256 hash of the script content). jQuery dynamically executed scripts via `globalEval` or `getScript` may fail under strict CSP unless a nonce is explicitly provided. A common CSP header for jQuery applications is: `Content-Security-Policy: default-src 'self'; script-src 'self' https://code.jquery.com; style-src 'self'`.

**Beginner-Friendly Explanation:** CSP is like a bouncer at a club. It has a list of approved guests (trusted script sources). If someone tries to bring in a script from an unapproved source, the bouncer blocks it. Even if an attacker manages to inject a script into your page, the bouncer checks the script's source against the approved list and refuses to let it run if it is not on the list.

### Purposes

- To provide a defense-in-depth layer against XSS by restricting script execution to trusted sources.
- To block inline script execution, which is the most common XSS payload delivery mechanism.
- To prevent the execution of scripts from untrusted third-party domains.
- To mitigate the impact of XSS vulnerabilities even when output encoding is incomplete.
- To comply with modern security best practices and browser security features.

### Syntax Rules and Structure

**Complete General Syntax (CSP Header):**
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://code.jquery.com; style-src 'self'
```

**Complete General Syntax (CSP with Nonce for Inline Scripts):**
```http
Content-Security-Policy: script-src 'nonce-abc123' https://code.jquery.com
```

```html
<script nonce="abc123">
    // Inline script with matching nonce
</script>
```

| Directive | Purpose | Example |
|-----------|---------|---------|
| `default-src 'self'` | Default source for all resource types | Only same-origin resources |
| `script-src 'self' https://code.jquery.com` | Allowed script sources | jQuery CDN and same-origin |
| `style-src 'self'` | Allowed style sources | Same-origin stylesheets |
| `'nonce-abc123'` | Allows specific inline scripts | Inline script with `nonce="abc123"` |
| `'unsafe-inline'` | Allows all inline scripts (NOT recommended) | Avoid |

**Syntax Rules:**

- Use `'self'` to allow resources from the same origin only.
- Add trusted CDN domains to `script-src` (e.g., `https://code.jquery.com`).
- Avoid `'unsafe-inline'` and `'unsafe-eval'`; these defeat much of CSP's protection.
- Use `'nonce-<random>'` for inline scripts that must execute; generate a new nonce for each request.
- jQuery's `.css()` and `.attr('style', ...)` methods set inline styles, which are blocked by `style-src 'self'`; use CSS classes instead.

**Constraints and Limitations:**

- CSP does not prevent all XSS attacks; it is a mitigation layer, not a complete solution.
- Inline event handlers (`onclick`, `onerror`, etc.) are blocked by CSP; they must be replaced with `addEventListener()` or jQuery's `.on()`.
- jQuery's dynamic script execution (`$.getScript`, `$.globalEval`) may fail under strict CSP unless a nonce is provided.
- Implementing CSP requires careful planning and testing; an overly restrictive policy can break legitimate functionality.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Configuring CSP for a jQuery Application**

```http
# Server response header
Content-Security-Policy: default-src 'self'; script-src 'self' https://code.jquery.com https://cdn.jsdelivr.net; style-src 'self' https://code.jquery.com; img-src 'self' data:; object-src 'none'
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>CSP Demo</title>
  <!-- jQuery loaded from an approved CDN -->
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <!-- No inline scripts allowed; all scripts must be in external files -->
  <script src="/js/app.js"></script>
</head>
<body>
  <div id="output"></div>
</body>
</html>
```

**Expected Output:** The jQuery library loads from the approved CDN. Inline scripts and event handlers are blocked by the browser, and a CSP violation is logged to the console. Legitimate scripts loaded from approved sources execute normally.

**Why this output:** The `script-src` directive allows scripts from `'self'`, `https://code.jquery.com`, and `https://cdn.jsdelivr.net`. Any inline script or script from an unapproved domain is blocked. This prevents injected scripts from executing even if they are inserted into the DOM.

### Real-World Cases

- **Enterprise applications:** Enforcing CSP to prevent XSS in applications that handle sensitive data.
- **Content management systems:** Adding CSP headers to reduce the impact of stored XSS vulnerabilities.
- **Third-party integrations:** Restricting which external scripts are allowed to load.
- **Payment pages:** Using strict CSP to protect against script injection on checkout pages.

---

## Enhanced Topic: Trusted Types API Integration — Enforcing Strict Object-Based Policies

### Definitions

**Core Definition:** The Trusted Types API is a browser security mechanism that prevents DOM-based XSS by requiring that data passed to dangerous DOM sinks be wrapped in a `TrustedHTML` (or `TrustedScript`, `TrustedScriptURL`) object created by a developer-defined policy, rather than being passed as a raw string.

**Technical Definition:** The Trusted Types API is implemented via the `trustedTypes` global object. Developers create a policy using `trustedTypes.createPolicy(name, { createHTML: function(input) { ... } })`. The policy's `createHTML` function receives the raw string and returns a sanitized `TrustedHTML` object. When Trusted Types are enforced (via the `require-trusted-types-for 'script'` CSP directive), DOM sinks like `.html()` and `element.innerHTML` reject plain strings and only accept `TrustedHTML` objects. DOMPurify supports Trusted Types via the `RETURN_TRUSTED_TYPE: true` option, which returns a `TrustedHTML` object instead of a string: `var clean = DOMPurify.sanitize(dirty, { RETURN_TRUSTED_TYPE: true })`. Developers can also supply a custom Trusted Types policy to DOMPurify: `DOMPurify.sanitize(dirty, { TRUSTED_TYPES_POLICY: trustedTypes.createPolicy({ createHTML: s => s, createScriptURL: s => s }) })`.

**Beginner-Friendly Explanation:** Trusted Types is like a security checkpoint at the airport. Before any data can be passed to a dangerous DOM method, it must go through the checkpoint and be stamped as "TrustedHTML." If you try to pass a raw string, the checkpoint rejects it. This forces developers to sanitize data before it can be inserted into the DOM, preventing XSS at the browser level.

### Purposes

- To enforce sanitization at the browser level, making it impossible to bypass by passing raw strings to dangerous sinks.
- To provide a stronger defense than output encoding alone, because it cannot be accidentally bypassed.
- To integrate with DOMPurify for automatic `TrustedHTML` generation.
- To comply with the `require-trusted-types-for 'script'` CSP directive, which enforces Trusted Types.
- To prevent DOM-based XSS in modern browsers that support the API.

### Syntax Rules and Structure

**Complete General Syntax (Creating a Trusted Types Policy):**
```javascript
if (window.trustedTypes && trustedTypes.createPolicy) {
    var policy = trustedTypes.createPolicy("myPolicy", {
        createHTML: function(input) {
            return input.replace(/<script>/gi, "");
        }
    });
    var trustedHTML = policy.createHTML(userInput);
    document.getElementById("output").innerHTML = trustedHTML;
}
```

**Complete General Syntax (DOMPurify with Trusted Types):**
```javascript
var clean = DOMPurify.sanitize(dirty, { RETURN_TRUSTED_TYPE: true });
// clean is a TrustedHTML object
document.getElementById("output").innerHTML = clean;
```

| Component | Description |
|-----------|-------------|
| `trustedTypes.createPolicy(name, { createHTML })` | Creates a policy that produces `TrustedHTML` objects. |
| `DOMPurify.sanitize(dirty, { RETURN_TRUSTED_TYPE: true })` | Returns a `TrustedHTML` object. |
| `require-trusted-types-for 'script'` | CSP directive that enforces Trusted Types. |

**Syntax Rules:**

- Create a Trusted Types policy with `createHTML` (and optionally `createScriptURL`) to produce trusted objects.
- Use DOMPurify's `RETURN_TRUSTED_TYPE: true` option to generate `TrustedHTML` automatically.
- Trusted Types require browser support (Chrome and Edge; Firefox and Safari have partial or no support).
- Combine Trusted Types with CSP's `require-trusted-types-for 'script'` directive for enforcement.
- Provide a fallback for browsers that do not support Trusted Types.

**Constraints and Limitations:**

- Trusted Types are not supported in all browsers; Safari and Firefox have limited support.
- Enforcing Trusted Types can break existing code that passes raw strings to `.html()` or `innerHTML`; a migration plan is required.
- jQuery itself does not currently enforce Trusted Types; the application must enforce them via CSP and DOMPurify.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: DOMPurify with Trusted Types**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Trusted Types Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dompurify@3.4.15/dist/purify.min.js"></script>
</head>
<body>
  <div id="output"></div>
  <input type="text" id="input" value="<b>Safe</b><script>alert('XSS')</script>">
  <button id="render">Render</button>

  <script>
    $(function() {
      $("#render").click(function() {
        var dirty = $("#input").val();

        // Step 1: Sanitize and return a TrustedHTML object
        var clean = DOMPurify.sanitize(dirty, { RETURN_TRUSTED_TYPE: true });

        // Step 2: Insert the TrustedHTML object
        document.getElementById("output").innerHTML = clean;
        // The <b> tag is preserved; the <script> tag is removed
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking "Render" displays "Safe" in bold text. The script tag is removed. In browsers that enforce Trusted Types, the `innerHTML` assignment accepts the `TrustedHTML` object but would reject a raw string.

**Why this output:** DOMPurify with `RETURN_TRUSTED_TYPE: true` sanitizes the input and returns a `TrustedHTML` object. When Trusted Types are enforced, `innerHTML` requires a `TrustedHTML` object, preventing raw string injection.

### Real-World Cases

- **Modern web applications:** Enforcing Trusted Types to prevent DOM-based XSS at the browser level.
- **Content management systems:** Using DOMPurify with Trusted Types to sanitize user-authored HTML.
- **Third-party widget integration:** Creating policies that allow specific trusted sources while blocking others.
- **Progressive migration:** Gradually adopting Trusted Types by creating policies and monitoring violations before enforcing them.

---

## Enhanced Topic: Safe Attribute Modification — Preventing Malicious URI Schemes

### Definitions

**Core Definition:** Safe attribute modification is the practice of validating and sanitizing URL-valued attributes (e.g., `href`, `src`, `action`, `formaction`) before passing untrusted data to jQuery's `.attr()` method, to prevent the execution of `javascript:` URIs and other malicious schemes.

**Technical Definition:** jQuery's `.attr()` method sets attributes on DOM elements. When the attribute is a URL-valued attribute like `href` or `src`, passing an untrusted value can allow an attacker to inject a `javascript:` URI, which executes script when the user clicks the link or the browser loads the resource. The Stack Overflow discussion confirms: `$('#link001').attr("href", "javascript:alert('evil')")` will execute the script when the link is clicked. Simple checks like `encodeURI()` are insufficient, and even checking for `javascript:` is error-prone because attackers can prepend control characters and whitespace: `$('#link001').attr("href", "\n\b\r\t jAvAsCrIpT:alert('evil')")`. The recommended approach is a whitelist: require that the URL starts with `http://` or `https://` (case-insensitive), or use a library like DOMPurify with the `ALLOWED_URI_REGEXP` option to validate URLs.

**Beginner-Friendly Explanation:** A URL attribute like `href` tells the browser where to go when you click a link. If someone puts `javascript:alert('hacked')` in that attribute, the browser runs the script instead of navigating. Safe attribute modification means checking the URL before setting it, and only allowing safe schemes like `http` and `https`.

### Purposes

- To prevent `javascript:` URI injection through `href`, `src`, and other URL-valued attributes.
- To block other dangerous schemes like `vbscript:` and `livescript:` that may execute in some browsers.
- To enforce a whitelist of allowed URL schemes (http, https, mailto, tel, etc.).
- To protect against URL-based XSS vectors that bypass HTML encoding.
- To comply with OWASP's recommendation to encode URI attributes with URL encoding.

### Syntax Rules and Structure

**Complete General Syntax (Whitelist Validation):**
```javascript
function safeUrl(url) {
    // Allow only http, https, and relative URLs
    if (/^https?:\/\//i.test(url) || /^\//.test(url)) {
        return url;
    }
    return "#"; // Fallback for unsafe URLs
}

$("#link").attr("href", safeUrl(userInput));
```

**Complete General Syntax (DOMPurify URI Validation):**
```javascript
var clean = DOMPurify.sanitize(userInput, {
    ALLOWED_URI_REGEXP: /^(?:(?:(?:f|ht)tps?|mailto|tel|callto|sms|cid|xmpp):|[^a-z]|[a-z+.\-]+(?:[^a-z+.\-:]|$))/i
});
```

| Approach | Description |
|----------|-------------|
| Whitelist regex | Allow only `http://`, `https://`, and relative URLs. |
| DOMPurify `ALLOWED_URI_REGEXP` | Default regex allows safe schemes; customize as needed. |
| `encodeURI()` | NOT sufficient; does not prevent `javascript:` schemes. |

**Syntax Rules:**

- Use a whitelist approach; allow only known-safe schemes (`http`, `https`, `mailto`, `tel`).
- Allow relative URLs (starting with `/` or `#`) if needed for internal navigation.
- Do not rely on `encodeURI()` or simple string matching to block `javascript:`; attackers can obfuscate the scheme.
- For `src` attributes on `<img>`, `<script>`, and `<iframe>`, use the same validation.
- Consider using DOMPurify's `ALLOWED_URI_REGEXP` for comprehensive URL validation.

**Constraints and Limitations:**

- Whitelisting can break legitimate URLs that use non-standard schemes; customize the whitelist for your application's needs.
- Relative URLs are generally safe but should be validated to prevent path traversal in some contexts.
- Some browsers may execute `data:` URIs in certain contexts; consider blocking `data:` for `href` attributes.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Safe URL Validation for `.attr("href")`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Safe Attribute Demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <a id="link" href="#">Click me</a>
  <input type="text" id="urlInput" value="javascript:alert('XSS')">
  <button id="setUrl">Set URL</button>

  <script>
    $(function() {
      $("#setUrl").click(function() {
        var userUrl = $("#urlInput").val().trim();

        // Step 1: Validate the URL against a whitelist
        var safeUrl = "#";
        if (/^https?:\/\//i.test(userUrl) || /^\//.test(userUrl)) {
          safeUrl = userUrl;
        }

        // Step 2: Set the validated URL
        $("#link").attr("href", safeUrl);
      });
    });
  </script>
</body>
</html>
```

**Expected Output:** Entering `javascript:alert('XSS')` and clicking "Set URL" sets the link's `href` to `#` (the fallback). Clicking the link does nothing. Entering `https://example.com` sets the link's `href` to that URL, and clicking it navigates to the site.

**Why this output:** The whitelist regex only allows URLs starting with `http://` or `https://`, or relative URLs starting with `/`. The `javascript:` URI fails the check and is replaced with the safe fallback `#`.

### Real-World Cases

- **User profile links:** Validating URLs submitted by users for their website or social media profiles.
- **Image sources:** Validating `src` attributes for user-uploaded images.
- **Redirect parameters:** Validating `returnUrl` or `redirect` parameters to prevent open redirects.
- **Form actions:** Validating `action` attributes to prevent form submission to malicious endpoints.

---

## References

- XSS (Cross Site Scripting) Prevention Cheat Sheet — OWASP — https://wiki.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet
- jQuery html() method a security risk like innerHTML? — Stack Overflow — https://stackoverflow.com/questions/49396862/jquery-html-method-a-security-risk-like-innerhtml
- jQuery will evaluate <script> tags and execute script in a variety of API's — EdgeScan — https://www.edgescan.com/wp-content/uploads/2018/08/04.-XSS-and-Encoding-edgescan.pdf
- Is it possible to perform a XSS attack using jQuery's .attr("href", value) method? — Stack Overflow — https://stackoverflow.com/questions/54626970/is-it-possible-to-perform-a-xss-attack-using-jquerys-attrhref-value-metho
- U.S. CISA adds JQuery flaw to its Known Exploited Vulnerabilities catalog — Security Affairs — https://securityaffairs.com/173388/breaking-news/u-s-cisa-adds-jquery-flaw-known-exploited-vulnerabilities-catalog.html
- CSP: script-src — MDN Web Docs — http://developer.typescripts.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/script-src
- DOMPurify README — https://raw.githubusercontent.com/cure53/DOMPurify/c70f8c57e60d2b4eb29d560682e7832febd62eaa/README.md
- jQuery .html() Documentation — https://api.jquery.com/html/
- jQuery .text() Documentation — https://api.jquery.com/text/
- jQuery .attr() Documentation — https://api.jquery.com/attr/
- CSP Compatibility Issue with jQuery 3.7.1 under Strict Content-Security-Policy — GitHub — https://github.com/jquery/jquery/discussions/5680
- Revision b75a5c50-5411-4aa6-9ba3-66810dd52be8 — Stack Overflow — https://stackoverflow.com/revisions/b75a5c50-5411-4aa6-9ba3-66810dd52be8/view-source
- Revision 5e970ae8-4215-4ed7-9f3b-45ec41cd926f — Stack Overflow — https://stackoverflow.com/revisions/5e970ae8-4215-4ed7-9f3b-45ec41cd926f/view-source
- jQuery插件如何进行安全防护 — 亿速云 — http://www.yisu.com/jc/1058819.html
- Trusted Types API — MDN Web Docs — https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API
- require-trusted-types-for — MDN Web Docs — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/require-trusted-types-for
- HTML Sanitization — OWASP — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html