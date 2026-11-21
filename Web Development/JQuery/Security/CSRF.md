# jQuery CSRF Prevention — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery CSRF prevention is the discipline of ensuring that state‑changing HTTP requests initiated from a jQuery‑powered client cannot be forged by a malicious third‑party site, by combining server‑side token validation, cookie attribute hardening, request‑origin verification, and secure AJAX configuration.

**Technical Definition**
Cross‑Site Request Forgery (CSRF) is an attack that occurs when a malicious web site, email, blog, instant message, or program tricks an authenticated user's web browser into performing an unwanted action on a trusted site. Because browser requests automatically include all cookies — including session cookies — the target site cannot distinguish between legitimate authorised requests and forged authenticated requests unless a proper challenge‑response mechanism verifies the identity and authority of the requester. In jQuery applications, CSRF risk is amplified because AJAX requests (`$.ajax()`, `$.post()`, `$.get()`) are commonly used for state‑changing operations, and the client‑side code is fully visible to attackers. Defence requires a layered approach: **CSRF tokens** (synchronizer token pattern or signed double‑submit cookie), **SameSite cookie attributes** (Lax or Strict), **Origin and Referer header validation**, **custom request headers** for AJAX, and **secure coding practices** that prevent XSS from defeating these controls.

**Beginner‑Friendly Explanation**
Imagine you are logged into your bank's website, and you visit a malicious page in another tab. That malicious page could contain a hidden form that automatically submits a request to your bank — for example, to transfer money to the attacker. Because your browser automatically includes your bank's session cookie with the request, the bank thinks the request came from you. That is CSRF. jQuery CSRF prevention is about making sure the bank can tell the difference between a request you actually intended to make and one that was forged by another site. The main tools are: a secret token that only your bank's pages know (CSRF token), a cookie setting that tells the browser not to send your session cookie on cross‑site requests (`SameSite`), and server‑side checks on where the request came from (`Origin` and `Referer` headers).

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **CSRF exploits ambient authentication** | The browser automatically attaches cookies to requests, so authentication alone cannot distinguish legitimate from forged requests. |
| **Tokens are the primary defence** | Stateful applications use the synchronizer token pattern; stateless applications use the double‑submit cookie pattern. |
| **jQuery requires explicit AJAX configuration** | Tokens must be attached to AJAX requests via `$.ajaxSetup()` or per‑request `headers`, because jQuery does not do this automatically. |
| **SameSite is defence in depth, not a complete solution** | Lax enforcement provides reasonable defence against POST‑based CSRF but does not offer a robust defence against CSRF as a general category of attack. |
| **Origin is preferred over Referer** | The `Origin` header is more reliable; the `Referer` header may be omitted, especially over HTTPS, and is considered a weaker form of protection. |
| **XSS can defeat all CSRF mitigations** | If an attacker can execute JavaScript in the victim's browser via XSS, they can read CSRF tokens and bypass every control. |

---

### Prerequisites

- Working knowledge of jQuery AJAX (`$.ajax()`, `$.ajaxSetup()`, `$.post()`).
- Understanding of HTTP cookies, sessions, and the same‑origin policy.
- Familiarity with HTML forms and how browsers submit them.
- Awareness of server‑side request handling and middleware.

---

### Related Programming Areas

- **XSS prevention** — XSS defeats CSRF tokens; the two must be addressed together.
- **Session management** — secure cookie attributes (Secure, HttpOnly, SameSite).
- **CORS** — cross‑origin request handling and preflight behaviour.
- **Authentication and authorisation** — verifying identity and enforcing access rules.
- **API security** — token‑based authentication and stateless request validation.

---

### Core Concepts / Features (Index)

1. CSRF (Cross‑Site Request Forgery)
2. CSRF Tokens
3. AJAX CSRF
4. SameSite Cookies
5. Origin Validation
6. Referer Validation
7. CSRF Prevention Best Practices


## Core Concept 1: CSRF (Cross‑Site Request Forgery)

### Definitions

**Core Definition**
CSRF is an attack that forces an authenticated user's browser to execute unwanted actions on a web application in which the user is currently authenticated, by exploiting the browser's automatic inclusion of credentials (cookies, HTTP authentication) with cross‑site requests.

**Technical Definition**
A CSRF attack occurs when a malicious web site, email, blog, instant message, or program tricks an authenticated user's web browser into performing an unwanted action on a trusted site. Unprotected target sites cannot distinguish between legitimate authorised requests and forged authenticated requests because browser requests automatically include all cookies including session cookies. Successful CSRF attacks can only exploit the capabilities exposed by the vulnerable application and the user's privileges — depending on the user's credentials, the attacker can transfer funds, change a password, make an unauthorised purchase, or elevate privileges. CSRF is mitigated by verifying that the request originated from the legitimate application, typically through unpredictable tokens, SameSite cookies, or Origin/Referer header validation.

**Beginner‑Friendly Explanation**
CSRF is like someone using your stamped, pre‑signed letterhead to send a letter you never wrote. You are logged into a site, so your browser has a “stamp” (session cookie) that proves who you are. A malicious site can trick your browser into sending a request with that stamp — and the server, seeing a valid stamp, carries out the action. The defence is to require something the attacker cannot forge: a secret token that is embedded in the legitimate page, a cookie attribute that stops the browser from sending your stamp on cross‑site requests, or a server‑side check of where the request came from.

---

### Purposes

- **To understand the fundamental mechanism** by which CSRF exploits ambient authentication.
- **To recognise that authentication alone does not prevent CSRF** — the server must verify intent, not just identity.
- **To motivate the use of CSRF tokens, SameSite cookies, and header validation** as complementary defences.
- **To prioritise state‑changing operations** (POST, PUT, DELETE, PATCH) as the primary targets of CSRF protection.
- **To understand why XSS must also be prevented**, since XSS can bypass all CSRF controls.

---

### Syntax Rules and Structure

CSRF is not expressed in a single syntax; it is an attack pattern:

```
Victim is authenticated to target site
→ Victim visits attacker-controlled site
→ Attacker site sends a request to target site (form, image, AJAX)
→ Browser automatically includes target-site cookies
→ Target site executes the action as the victim
```

**Defence**: Ensure the request includes a value the attacker cannot know or set (CSRF token), or ensure the browser does not send credentials on cross‑site requests (SameSite), or validate the request origin (Origin/Referer).

---

### Constraints and Limitations

- **CSRF only affects state‑changing requests**; read‑only GET requests that do not modify state are generally not targets, though they can be used for information leakage.
- **CSRF does not require the attacker to see the response**; the action is performed even if the attacker cannot read the result.
- **XSS defeats CSRF protections** because the attacker can read tokens and make requests from within the trusted origin.

---

### Annotated Code Example (Vulnerable vs. Protected)

```html
<!-- ❌ VULNERABLE: no CSRF protection on a state-changing form -->
<form action="/transfer" method="POST">
  <input type="text" name="to" value="attacker-account">
  <input type="text" name="amount" value="1000">
  <button type="submit">Transfer</button>
</form>

<!-- ✅ PROTECTED: synchronizer token included as a hidden field -->
<form action="/transfer" method="POST">
  <input type="hidden" name="csrf_token" value="a1b2c3d4e5f6...">
  <input type="text" name="to" value="recipient">
  <input type="text" name="amount" value="100">
  <button type="submit">Transfer</button>
</form>
```

**Expected Output**
The vulnerable form can be triggered from any site. The protected form includes a token that the server validates; a forged request without the valid token is rejected.

**Why this output occurs**: The server compares the submitted `csrf_token` with the token stored in the user's session. If the values do not match, the request is denied. The attacker cannot guess or obtain the token because it is unpredictable and embedded in the legitimate page.

---

### Real‑World Cases

- **Banking transfers**: Unprotected transfer forms allow attackers to move funds via a hidden form on a malicious site.
- **Password changes**: CSRF can change a victim's password without their knowledge.
- **Admin actions**: CSRF against administrative interfaces can create rogue accounts or modify settings.

---

## Core Concept 2: CSRF Tokens

### Definitions

**Core Definition**
A CSRF token is a unique, unpredictable, and secret value generated by the server and associated with the user's session, which must be included in every state‑changing request and validated by the server before the request is executed.

**Technical Definition**
Two primary token patterns exist. The **Synchronizer Token Pattern** (stateful) generates a random challenge token associated with the user's current session; the token is embedded in HTML forms and links, and the server verifies its existence and correctness on each state‑changing request. The **Double‑Submit Cookie Pattern** (stateless) sets a random token as a cookie and requires the same token to be submitted in the request body or header; the server compares the two values without maintaining server‑side state. The most secure double‑submit variant is the **Signed Double‑Submit Cookie**, which cryptographically ties the token to the user's authenticated session. Tokens must be unpredictable, unique per session, and validated on every state‑changing request.

**Beginner‑Friendly Explanation**
A CSRF token is like a numbered ticket that the bank gives you when you log in. Every time you want to make a transaction, you must show the ticket. The attacker does not have your ticket, so their forged request is rejected. The token is secret and unpredictable — the attacker cannot guess it. In a stateful application, the server remembers your ticket; in a stateless application, the server gives you a ticket in a cookie and asks you to show the same ticket in the request, then compares the two.

---

### Purposes

- **To provide an unpredictable challenge** that the attacker cannot forge.
- **To bind the request to the user's session** so that a request from another user is rejected.
- **To enable stateless protection** through the double‑submit cookie pattern when server‑side state is impractical.
- **To serve as the primary defence against CSRF** in state‑changing operations.
- **To complement SameSite cookies and header validation** in a defence‑in‑depth strategy.

---

### Syntax Rules and Structure

#### Synchronizer Token Pattern (Stateful)

**Server**:
1. Generate a random token.
2. Store it in the user's session.
3. Embed it in HTML forms (hidden field) or expose it to JavaScript (meta tag).

**Client**:
1. Include the token in every state‑changing request (form field or header).

**Server**:
1. Compare the submitted token with the session token.
2. Reject if missing or mismatched.

#### Double‑Submit Cookie Pattern (Stateless)

**Server**:
1. Set a random token as a cookie (not HttpOnly, so JavaScript can read it).
2. Do not store the token server‑side.

**Client**:
1. Read the token from the cookie.
2. Send the same token in the request body or header.

**Server**:
1. Compare the cookie token with the request token.
2. Reject if missing or mismatched.

---

### Constraints and Limitations

- **Tokens must be unpredictable**; use a cryptographically secure random generator.
- **Tokens must be unique per session**; reusing a global token defeats the purpose.
- **The synchronizer pattern requires server‑side state**; the double‑submit pattern does not, but must be signed or bound to the session to prevent cookie injection.
- **XSS can read tokens**; CSRF tokens do not protect against XSS.

---

### Annotated Code Example (Server‑Side Conceptual)

```javascript
// ✅ SYNCHRONIZER TOKEN PATTERN (server-side pseudo-code)

// Step 1: Generate and store a token when the session is created
app.use(session({
  secret: "...",
  genid: function () {
    return crypto.randomBytes(32).toString("hex"); // session ID
  }
}));

// Step 2: Generate a CSRF token and store it in the session
app.use(function (req, res, next) {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString("hex");
  }
  res.locals.csrfToken = req.session.csrfToken; // expose to templates
  next();
});

// Step 3: Include the token in forms
// <input type="hidden" name="csrf_token" value="<%= csrfToken %>">

// Step 4: Validate the token on state-changing requests
app.post("/transfer", function (req, res) {
  if (req.body.csrf_token !== req.session.csrfToken) {
    return res.status(403).send("Invalid CSRF token");
  }
  // proceed with the transfer
});
```

**Expected Output**
A forged request without the correct `csrf_token` is rejected with `403 Forbidden`. A legitimate request containing the session‑bound token succeeds.

**Why this output occurs**: The token is unpredictable and stored server‑side. The attacker cannot obtain it because it is embedded only in the legitimate page. The server compares the submitted token with the session token and rejects mismatches.

---

### Real‑World Cases

- **Django**: Uses the synchronizer token pattern with a `csrftoken` cookie and `csrfmiddlewaretoken` form field.
- **Ruby on Rails**: Uses `protect_from_forgery` with an authenticity token.
- **Stateless APIs**: Use signed double‑submit cookies to avoid server‑side session storage.

---

## Core Concept 3: AJAX CSRF

### Definitions

**Core Definition**
AJAX CSRF prevention is the practice of configuring jQuery's AJAX machinery to automatically attach the CSRF token to every state‑changing request — typically via a custom HTTP header — while ensuring that the token is not sent to cross‑domain endpoints.

**Technical Definition**
jQuery provides `$.ajaxSetup()` to set default options for all AJAX requests, and a `beforeSend` callback that runs before each request is sent. By checking `settings.type` against a list of safe HTTP methods (GET, HEAD, OPTIONS, TRACE) and `this.crossDomain`, the client can conditionally add the token header (e.g., `X-CSRF-Token`, `X-XSRF-TOKEN`, or `X-CSRFToken`) only for same‑origin, state‑changing requests. The token itself is typically retrieved from a `<meta>` tag rendered by the server, or from a cookie (for the double‑submit pattern). This approach ensures that all AJAX requests — including those made by plugins and third‑party code — are protected without requiring per‑request configuration.

**Beginner‑Friendly Explanation**
When you use jQuery to send AJAX requests, you need to make sure the CSRF token is included. Doing this manually for every request is error‑prone. jQuery lets you set a global rule: “before any request is sent, if it is a POST (or PUT/DELETE) and it is going to our own server, add the token as a header.” The server checks that header. The `crossDomain` check is important: it prevents your token from being sent to a different domain, which would leak it. The token is usually placed in a `<meta>` tag by the server, and jQuery reads it from there.

---

### Purposes

- **To ensure that every state‑changing AJAX request carries the CSRF token** without manual per‑request configuration.
- **To avoid leaking the token to cross‑domain endpoints** by checking `this.crossDomain`.
- **To protect AJAX‑based operations** (form submissions, deletions, updates) that may not use traditional HTML forms.
- **To centralise CSRF configuration** so that plugins and third‑party code are also covered.

---

### Syntax Rules and Structure

```javascript
// Retrieve the token from a meta tag
var csrfToken = $('meta[name="csrf-token"]').attr('content');

// Configure jQuery to add the token to state-changing requests
$.ajaxSetup({
  beforeSend: function (xhr, settings) {
    if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
      xhr.setRequestHeader("X-CSRF-Token", csrfToken);
    }
  }
});
```

**Component Breakdown**

| Component | Description |
|---|---|
| `csrfToken` | The token value, read from a `<meta>` tag or cookie. |
| `settings.type` | The HTTP method of the request. |
| `csrfSafeMethod()` | A helper that returns `true` for GET, HEAD, OPTIONS, TRACE. |
| `this.crossDomain` | `true` if the request targets a different domain; the token must not be sent. |
| `xhr.setRequestHeader()` | Attaches the token as a custom header. |

---

### Constraints and Limitations

- **The `<meta>` tag must exist before the script runs**; place the script after the `<head>` or use `$(document).ready()`.
- **The `crossDomain` check is essential**; without it, the token could be sent to an attacker‑controlled domain.
- **The server must validate the custom header**; the client‑side configuration alone is insufficient.
- **CORS preflight** may be triggered by custom headers; ensure the server allows the header.

---

### Annotated Code Example

```html
<!DOCTYPE html>
<html>
<head>
  <!-- Server renders the CSRF token into a meta tag -->
  <meta name="csrf-token" content="a1b2c3d4e5f6...">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="deleteBtn">Delete Item</button>

  <script>
    // Step 1: Read the token from the meta tag
    var csrfToken = $('meta[name="csrf-token"]').attr("content");

    // Step 2: Configure all AJAX requests
    $.ajaxSetup({
      beforeSend: function (xhr, settings) {
        // Step 3: Only add the header for unsafe methods and same-origin requests
        if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
          xhr.setRequestHeader("X-CSRF-Token", csrfToken);
        }
      }
    });

    // Step 4: Send a state-changing AJAX request
    $("#deleteBtn").on("click", function () {
      $.ajax({
        url: "/api/items/123",
        type: "DELETE",
        success: function () { console.log("Deleted"); },
        error: function (xhr) { console.log("Failed:", xhr.status); }
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
Clicking the button sends a `DELETE` request with the `X-CSRF-Token` header. The server validates the token and, if valid, deletes the item and returns `200 OK`. If the token is missing or invalid, the server returns `403 Forbidden`.

**Why this output occurs**: `$.ajaxSetup()` registers a `beforeSend` callback that runs before every AJAX request. The callback checks the HTTP method and cross‑domain status, and attaches the token header only when appropriate. The server compares the header value with the session token.

---

### Real‑World Cases

- **ASP.NET Core**: Uses `AutoValidateAntiforgeryToken` and configures `$.ajaxSetup` to send `X-XSRF-TOKEN`.
- **Django**: Uses `X-CSRFToken` header with a cookie‑based token and the `csrfSafeMethod` check.
- **Rails**: Uses `X-CSRF-Token` header with `$.ajaxSetup` and `beforeSend`.

---

## Core Concept 4: SameSite Cookies

### Definitions

**Core Definition**
The `SameSite` cookie attribute instructs the browser whether to send a cookie with cross‑site requests, providing a browser‑level defence against CSRF by restricting when session cookies are attached to requests initiated by other origins.

**Technical Definition**
`SameSite` has three values. **`Strict`** prevents the cookie from being sent by the browser to the target site in all cross‑site browsing contexts, even when following a regular link. **`Lax`** provides a reasonable balance between security and usability: the cookie is sent on same‑site requests and on top‑level cross‑site navigations that use a safe HTTP method (typically GET), but it is blocked on CSRF‑prone methods such as POST. **`None`** disables the restriction entirely, sending the cookie in all contexts; it requires the `Secure` attribute and provides no CSRF protection. As of December 2024, `Lax` is the default in Chrome, Edge, and Opera but not in Firefox or Safari, so developers should explicitly set the attribute. Lax enforcement provides reasonable defence in depth against CSRF attacks that rely on unsafe methods like POST, but it does not offer a robust defence against CSRF as a general category of attack.

**Beginner‑Friendly Explanation**
`SameSite` is a label you put on a cookie that tells the browser: “Only send this cookie when the request comes from my own site.” With `Strict`, the cookie is never sent from another site — even if the user clicks a link to your site, they will not be logged in on the first request. With `Lax`, the cookie is sent when the user navigates to your site via a normal link (GET), but not when a malicious site tries to POST a form. `Lax` is a good balance for most applications, but it is not a complete defence on its own. `None` turns the protection off and should only be used when the cookie must work in cross‑site embeds, and then only with `Secure` and additional CSRF defences.

---

### Purposes

- **To provide a browser‑level, defence‑in‑depth control** against CSRF without server‑side changes.
- **To block the automatic inclusion of session cookies** on cross‑site state‑changing requests (with `Lax` or `Strict`).
- **To balance security and usability** by choosing `Lax` for most applications or `Strict` for high‑security contexts.
- **To reduce reliance on tokens alone** by adding an independent layer of protection.

---

### Syntax Rules and Structure

| Value | Behaviour | CSRF Protection |
|---|---|---|
| `Strict` | Cookie sent only on same‑site requests; never on cross‑site requests, including link navigation. | Strong, but may affect usability (user appears logged out after following an external link). |
| `Lax` | Cookie sent on same‑site requests and on top‑level cross‑site GET navigations; blocked on cross‑site POST. | Reasonable defence in depth; not a complete solution. |
| `None` | Cookie sent on all requests, including cross‑site. Requires `Secure`. | No CSRF protection. |

**Server‑Side Setting** (conceptual):
```http
Set-Cookie: session=abc123; SameSite=Lax; Secure; HttpOnly
```

---

### Constraints and Limitations

- **`Lax` does not block all CSRF**: Attackers can still trigger top‑level navigations or use prerendering to create same‑site requests.
- **`Strict` can break usability** for users arriving from external links; consider the trade‑off.
- **`None` must be paired with `Secure` and additional CSRF defences**; it provides no CSRF protection.
- **Browser defaults vary**; explicitly set the attribute rather than relying on defaults.

---

### Annotated Code Example

```javascript
// ✅ SECURE COOKIE CONFIGURATION (server-side pseudo-code)
app.use(session({
  name: "session",
  secret: "...",
  cookie: {
    sameSite: "lax",      // or "strict" for high-security applications
    secure: true,          // HTTPS only
    httpOnly: true         // not accessible via JavaScript
  }
}));
```

**Expected Output**
The session cookie is sent on same‑site requests and on top‑level cross‑site GET navigations, but not on cross‑site POST requests. CSRF attacks that rely on cross‑site POST are blocked by the browser.

**Why this output occurs**: The `SameSite=Lax` attribute instructs the browser to withhold the cookie on cross‑site requests that use unsafe methods, while allowing it on safe top‑level navigations.

---

### Real‑World Cases

- **GitHub**: Uses `SameSite=Lax` to allow logged‑in sessions when users follow links from external sites while blocking cross‑site POST.
- **Banking applications**: May use `SameSite=Strict` because they do not want transactional pages linked from external sites.
- **Third‑party embeds**: Require `SameSite=None; Secure` and must implement their own CSRF tokens.

---

## Core Concept 5: Origin Validation

### Definitions

**Core Definition**
Origin validation is a server‑side defence that inspects the `Origin` HTTP header — and, as a fallback, the `Referer` header — to determine whether a state‑changing request originated from a trusted origin, rejecting requests whose origin does not match the application's expected host.

**Technical Definition**
The `Origin` header is sent by browsers on cross‑origin requests and on same‑origin POST requests, and contains the scheme, host, and port of the request's origin. Server‑side validation compares the `Origin` (or, if absent, the `Referer`) against an allowlist of trusted origins. If the header is present but does not match, the request is rejected (typically with `400 Bad Request` or `403 Forbidden`). If both `Origin` and `Referer` are missing, best practice is to reject the request as potentially malicious, though some frameworks treat header‑less requests as non‑browser clients and allow them. Origin validation is a **defence‑in‑depth** measure and is not a substitute for CSRF tokens; it is particularly useful for AJAX and API endpoints.

**Beginner‑Friendly Explanation**
When your browser sends a request to a server, it often includes a note saying where the request came from — this is the `Origin` header. The server can check: “Did this request come from my own site, or from somewhere else?” If it came from somewhere else, the server rejects it. This blocks many CSRF attacks because the attacker's site will have a different origin. The `Origin` header is more reliable than `Referer`, and if both are missing, the safest choice is to reject the request. However, Origin validation is a backup, not a replacement for tokens — some older browsers or privacy settings may omit these headers.

---

### Purposes

- **To provide a lightweight, stateless defence** against CSRF for API endpoints and AJAX requests.
- **To block requests whose origin is not on the trusted allowlist**.
- **To complement CSRF tokens and SameSite cookies** in a defence‑in‑depth strategy.
- **To detect and reject header‑less requests** that may be forged or originate from non‑browser clients.
- **To protect endpoints that cannot easily use tokens** (e.g., some legacy APIs).

---

### Syntax Rules and Structure

**Server‑Side Validation Logic** (conceptual):

1. Read the `Origin` header.
2. If `Origin` is present, compare it against the allowlist of trusted origins.
3. If `Origin` is absent, fall back to `Referer` and compare its origin.
4. If both are absent, reject the request (recommended).
5. If the origin matches, allow the request; otherwise, reject with `400` or `403`.

```javascript
// Conceptual middleware
function validateOrigin(req, res, next) {
  const origin = req.headers.origin || req.headers.referer;
  if (!origin) {
    return res.status(400).send("Missing Origin and Referer");
  }
  const parsed = new URL(origin);
  if (parsed.host !== req.headers.host) {
    return res.status(403).send("Invalid origin");
  }
  next();
}
```

---

### Constraints and Limitations

- **`Origin` is not sent on all requests** (e.g., some same‑origin GET requests); it is primarily present on POST and cross‑origin requests.
- **Privacy settings may suppress `Referer`**, especially over HTTPS.
- **Header‑less requests must be handled deliberately**; allowing them by default can bypass the protection.
- **Origin validation does not protect against same‑site attacks** (e.g., XSS).

---

### Annotated Code Example (Conceptual)

```javascript
// ✅ ORIGIN VALIDATION MIDDLEWARE (Node.js/Express pseudo-code)
const TRUSTED_ORIGINS = ["https://app.example.com", "https://admin.example.com"];

app.use(function (req, res, next) {
  // Step 1: Only check state-changing methods
  if (/^(GET|HEAD|OPTIONS|TRACE)$/i.test(req.method)) return next();

  // Step 2: Read Origin, fall back to Referer
  const origin = req.headers.origin;
  const referer = req.headers.referer;

  if (!origin && !referer) {
    // Step 3: Reject header-less requests (recommended)
    return res.status(400).send("Missing Origin and Referer");
  }

  // Step 4: Validate the origin
  const requestOrigin = origin || new URL(referer).origin;
  if (!TRUSTED_ORIGINS.includes(requestOrigin)) {
    return res.status(403).send("Invalid origin");
  }

  // Step 5: Allow the request
  next();
});
```

**Expected Output**
A POST request from `https://app.example.com` is allowed. A POST request from `https://evil.example.com` is rejected with `403 Forbidden`. A POST request with no `Origin` or `Referer` is rejected with `400 Bad Request`.

**Why this output occurs**: The middleware compares the request's origin against the allowlist. Missing headers are treated as suspicious and rejected, consistent with best practice.

---

### Real‑World Cases

- **ASP.NET Core**: Uses `Origin` and `Sec-Fetch-Site` headers alongside antiforgery tokens.
- **Apache Wicket**: `CsrfPreventionRequestCycleListener` rejects requests with mismatched or missing Origin/Referer.
- **API gateways**: Validate `Origin` to block cross‑site requests before they reach the application.

---

## Core Concept 6: Referer Validation

### Definitions

**Core Definition**
Referer validation is a server‑side check that inspects the `Referer` HTTP header to confirm that a state‑changing request originated from a page on the same application, rejecting requests whose referrer does not match the expected host or allowlist.

**Technical Definition**
The `Referer` header contains the URL of the page that linked to the requested resource. Referer validation compares this URL's origin against the application's expected origin. However, it is considered a **weaker form of CSRF protection** than Origin validation or tokens. Browsers may omit the `Referer` header for privacy reasons, especially when navigating from HTTPS to HTTP, or when the user has disabled referrer sending. Attackers cannot spoof the `Referer` in a CSRF attack because the browser sets it, but they can sometimes suppress it (e.g., by initiating the request from an FTP page or using `rel="noreferrer"`). Lenient Referer validation that accepts requests without a `Referer` is easy to bypass. Therefore, Referer validation should be used as a fallback to Origin validation, not as the sole defence.

**Beginner‑Friendly Explanation**
The `Referer` header is like a return address on an envelope — it tells the server which page the request came from. The server can check: “Did this request come from one of my own pages?” If not, it rejects the request. But `Referer` is less reliable than `Origin` because browsers sometimes leave it blank for privacy reasons. If you accept requests with no `Referer`, an attacker can suppress it and bypass your check. So `Referer` validation is a backup, not a primary defence — use it as a fallback when `Origin` is absent, and reject requests where both are missing.

---

### Purposes

- **To provide a fallback check** when the `Origin` header is absent.
- **To add a defence‑in‑depth layer** alongside tokens and SameSite cookies.
- **To detect requests that originate from external sites** when headers are present.
- **To complement Origin validation** in environments where `Origin` may not be sent.

---

### Syntax Rules and Structure

**Validation Logic** (conceptual):

1. Read the `Referer` header.
2. If absent, reject the request (recommended).
3. Parse the `Referer` URL and compare its origin against the trusted allowlist.
4. If the origin matches, allow; otherwise, reject.

```javascript
function validateReferer(req, res, next) {
  if (/^(GET|HEAD|OPTIONS|TRACE)$/i.test(req.method)) return next();

  const referer = req.headers.referer;
  if (!referer) {
    return res.status(400).send("Missing Referer");
  }
  const parsed = new URL(referer);
  if (parsed.origin !== "https://app.example.com") {
    return res.status(403).send("Invalid Referer");
  }
  next();
}
```

---

### Constraints and Limitations

- **`Referer` may be omitted** for privacy, security (HTTPS→HTTP), or user configuration.
- **Lenient validation (allowing missing Referer) is easily bypassed**.
- **Attackers can suppress the `Referer`** in some contexts (e.g., FTP pages, `rel="noreferrer"`).
- **Referer validation is weaker than Origin validation and tokens**; use it only as a fallback.

---

### Annotated Code Example (Conceptual)

```javascript
// ✅ REFERER VALIDATION (fallback to Origin)
app.use(function (req, res, next) {
  if (/^(GET|HEAD|OPTIONS|TRACE)$/i.test(req.method)) return next();

  // Prefer Origin; fall back to Referer
  const origin = req.headers.origin;
  const referer = req.headers.referer;
  const requestOrigin = origin || (referer ? new URL(referer).origin : null);

  if (!requestOrigin) {
    return res.status(400).send("Missing Origin and Referer");
  }

  if (requestOrigin !== "https://app.example.com") {
    return res.status(403).send("Invalid origin");
  }

  next();
});
```

**Expected Output**
A request with a valid `Origin` or `Referer` from `https://app.example.com` is allowed. A request with a `Referer` from another site is rejected. A request with neither header is rejected.

**Why this output occurs**: The middleware uses `Origin` when available and falls back to `Referer`. Missing headers are rejected to prevent suppression attacks. The comparison ensures the request originated from the trusted application.

---

### Real‑World Cases

- **Legacy applications**: Use Referer validation as a fallback when Origin is not available.
- **Form submissions**: Combine Referer checks with tokens for defence in depth.
- **Privacy‑conscious environments**: Must account for omitted Referer headers and reject missing values.

---

## Core Concept 7: CSRF Prevention Best Practices

### Definitions

**Core Definition**
jQuery CSRF prevention best practices are the consolidated conventions — using CSRF tokens on all state‑changing requests, setting SameSite cookies, validating Origin and Referer headers, configuring jQuery AJAX securely, preventing XSS, and layering defences — that systematically eliminate the conditions for CSRF in jQuery‑based applications.

**Technical Definition**
Best practices synthesise the individual techniques: (1) **use the synchronizer token pattern** for stateful applications and the **signed double‑submit cookie pattern** for stateless applications; (2) **attach tokens to every state‑changing request** — form fields for HTML forms, custom headers for AJAX; (3) **configure jQuery's `$.ajaxSetup()`** with a `beforeSend` callback that adds the token header for unsafe methods and same‑origin requests; (4) **set `SameSite=Lax` or `Strict`** on session cookies and explicitly specify the attribute; (5) **validate `Origin`** on the server, falling back to `Referer`, and reject requests where both are missing; (6) **prevent XSS**, because XSS can bypass all CSRF controls; (7) **do not use GET for state‑changing operations**; and (8) **defence in depth**: combine tokens, SameSite, and header validation rather than relying on any single control.

**Beginner‑Friendly Explanation**
Best practices are the habits that make CSRF protection reliable. Use a secret token on every form and AJAX request. Set your session cookie to `SameSite=Lax`. Check the `Origin` header on the server. Configure jQuery once so every AJAX request automatically includes the token. Prevent XSS, because XSS lets attackers steal tokens. Never use GET to change data. And layer your defences — do not rely on just one mechanism. If one fails, another still protects you.

---

### Purposes

- **To provide a single actionable checklist** for CSRF‑safe jQuery development.
- **To combine multiple independent defences** (tokens, SameSite, header validation) so that no single failure compromises the application.
- **To prevent the most common CSRF vulnerabilities** in jQuery‑based applications.
- **To establish a baseline for code reviews and security audits**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Token pattern** | Synchronizer token (stateful) or signed double‑submit (stateless) | No token on state‑changing requests |
| **AJAX configuration** | `$.ajaxSetup` with `beforeSend` and `crossDomain` check | Manual per‑request token handling |
| **Cookie attribute** | `SameSite=Lax` or `Strict`; explicit `Secure` and `HttpOnly` | `SameSite=None` without additional defences |
| **Origin validation** | Validate `Origin`; fall back to `Referer`; reject if both missing | Trusting header‑less requests |
| **HTTP methods** | POST/PUT/DELETE/PATCH for state changes | GET for state‑changing operations |
| **XSS prevention** | Output encoding, CSP, safe jQuery APIs | Unescaped `.html()` with user input |
| **Defence in depth** | Tokens + SameSite + header validation | Relying on a single control |

---

### Constraints and Limitations

- **Best practices are context‑dependent**; a stateless API has different needs from a traditional server‑rendered application.
- **`SameSite=Lax` is not a complete defence**; it must be combined with tokens or header validation.
- **Origin and Referer headers may be absent**; the application must decide how to handle header‑less requests.
- **XSS prevention is a prerequisite**; CSRF controls are ineffective if XSS is present.

---

### Annotated Code Example (Consolidated)

```html
<!DOCTYPE html>
<html>
<head>
  <!-- Server injects CSRF token into a meta tag -->
  <meta name="csrf-token" content="a1b2c3d4e5f6...">
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <form id="myForm" action="/update" method="POST">
    <!-- Synchronizer token in hidden field -->
    <input type="hidden" name="csrf_token" value="a1b2c3d4e5f6...">
    <input type="text" name="data" value="new value">
    <button type="submit">Update</button>
  </form>

  <script>
    // ✅ jQuery AJAX CSRF configuration
    var csrfToken = $('meta[name="csrf-token"]').attr("content");
    $.ajaxSetup({
      beforeSend: function (xhr, settings) {
        if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
          xhr.setRequestHeader("X-CSRF-Token", csrfToken);
        }
      }
    });

    // Any AJAX POST will now include the header automatically
    $("#myForm").on("submit", function (e) {
      e.preventDefault();
      $.ajax({
        url: "/update",
        type: "POST",
        data: $(this).serialize(),
        success: function () { console.log("Updated"); }
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
The form includes a synchronizer token in a hidden field. AJAX POST requests automatically include the `X-CSRF-Token` header. The server validates the token and rejects forged requests. The session cookie is set with `SameSite=Lax` (server‑side configuration).

**Why this output occurs**: The hidden field provides the token for traditional form submissions. `$.ajaxSetup` provides the token for AJAX requests. The server validates the token in both cases. `SameSite=Lax` adds a browser‑level layer that blocks cross‑site POST requests from including the session cookie. Together, these practices implement defence in depth.

---

### Real‑World Cases

- **Code reviews**: A checklist ensures that new endpoints include CSRF tokens and proper AJAX configuration.
- **Framework integration**: Use built‑in CSRF protection (Django, Rails, ASP.NET Core) and configure jQuery to send the token.
- **Security audits**: Verify that SameSite attributes are set, Origin validation is enforced, and XSS prevention is in place.

---

## References

- OWASP Cross‑Site Request Forgery Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP SameSite — https://community.owasp.org/SameSite
- OWASP CSRFGuard — https://wiki.owasp.org/index.php/CSRFGuard
- IETF RFC 6265bis: Cookies: HTTP State Management Mechanism (SameSite) — https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis/22/
- MDN: SameSite cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- jQuery `$.ajaxSetup()` — https://api.jquery.com/jQuery.ajaxSetup/
- jQuery `$.ajax()` — https://api.jquery.com/jQuery.ajax/
- Django CSRF documentation — https://docs.djangoproject.com/en/stable/ref/csrf/
- ASP.NET Core Anti‑Forgery — https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Spring Security CSRF — https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html
- PortSwigger: CSRF — https://portswigger.net/web-security/csrf
- CWE‑352: Cross‑Site Request Forgery (CSRF) — https://cwe.mitre.org/data/definitions/352.html
- OWASP Top 10:2021 – A01: Broken Access Control (includes CSRF) — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- Yisu: jQuery Plugin Security Protection (CSRF section) — https://m.yisu.com/zixun/1058819.html
- Stack Overflow: jQuery AJAX CSRF token header setup — https://stackoverflow.com/questions/32613782/jquery-ajax-csrf-token-header-setup