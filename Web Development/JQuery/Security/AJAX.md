# jQuery AJAX Security — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery AJAX security is the discipline of ensuring that asynchronous HTTP requests initiated by jQuery — and the responses they receive — do not introduce vulnerabilities such as cross-site scripting (XSS), cross-site request forgery (CSRF), data leakage, or unauthorised access, by applying transport-layer encryption, origin controls, authentication and authorisation checks, and secure coding practices.

**Technical Definition**
AJAX security operates across multiple layers. **Transport security** (HTTPS) ensures that requests and responses are encrypted and authenticated, preventing man-in-the-middle attacks and data interception. **Origin control** (CORS) restricts which domains can read responses from cross-origin requests, using a server-driven allowlist and browser-enforced preflight checks. **Authentication** verifies identity (via cookies, tokens, or other credentials), while **authorisation** enforces access decisions on every request, including AJAX calls, on the server. jQuery-specific risks include the automatic fallback to **JSONP** for cross-domain requests (which works by injecting `<script>` tags and executes arbitrary JavaScript), the use of unsafe sinks such as `.html()` with untrusted data, and the failure to protect cookie-authenticated state-changing requests against CSRF. OWASP's Web Frontend Security Cheat Sheet (formerly the AJAX Security Cheat Sheet) provides a foundational starting point for these controls.

**Beginner-Friendly Explanation**
AJAX lets your page talk to the server without reloading. Security for AJAX is about making sure those conversations are private, that only the right people can start them, and that the answers cannot be poisoned to attack your users. Think of it like sending letters: HTTPS is the sealed envelope, CORS is the list of people allowed to read the reply, authentication is your signature, and authorisation is the rule about what you are allowed to ask for. jQuery makes AJAX easy, but it also has some sharp edges — especially JSONP, which turns the reply into executable code. AJAX security is about knowing those edges and avoiding them.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Multi-layered** | Security depends on transport (HTTPS), origin control (CORS), identity (authentication), and permission (authorisation) working together. |
| **Server-enforced** | Client-side jQuery code is fully visible and modifiable; all security decisions must be enforced on the server. |
| **JSONP is fundamentally unsafe** | jQuery may silently fall back to JSONP for cross-domain requests; JSONP executes the response as JavaScript and must be disabled for untrusted endpoints. |
| **Cookie-based authentication requires CSRF protection** | AJAX requests using cookie auth must include CSRF tokens; header-based authentication (e.g., Bearer tokens) is not vulnerable to CSRF. |
| **CORS is server-controlled** | The browser enforces CORS, but the server decides which origins are allowed; wildcard (`*`) origins with credentials are invalid and dangerous. |
| **HTTPS must originate from an HTTPS page** | An AJAX call from an HTTP page cannot be fully secured against SSL stripping even if it targets an HTTPS URL. |

---

### Prerequisites

- Working knowledge of jQuery AJAX (`$.ajax()`, `$.get()`, `$.post()`, `$.getJSON()`).
- Understanding of the same-origin policy and cross-origin requests.
- Familiarity with HTTP headers (Origin, Access-Control-Allow-Origin, Authorization).
- Awareness of authentication mechanisms (cookies, tokens) and the difference between authentication and authorisation.

---

### Related Programming Areas

- **XSS prevention** — AJAX responses are a common source of untrusted data that reaches DOM sinks.
- **CSRF prevention** — cookie-based AJAX requests require token-based protection.
- **HTTPS and TLS** — transport-layer security for data in transit.
- **CORS configuration** — server-side control of cross-origin access.
- **Session management** — secure cookie attributes and token storage.
- **API security** — authorisation checks on every endpoint, including AJAX-exposed ones.

---

### Core Concepts / Features (Index)

1. AJAX Security (General)
2. CORS (Cross-Origin Resource Sharing)
3. JSONP Security
4. HTTPS
5. Authentication
6. Authorization
7. AJAX Security Best Practices


## Core Concept 1: AJAX Security (General)

### Definitions

**Core Definition**
AJAX security is the set of principles and controls that prevent asynchronous requests from being used to read, modify, or exfiltrate data without authorization, or to introduce executable content into the client.

**Technical Definition**
AJAX security encompasses the full request-response lifecycle: the client constructs a request (method, URL, headers, body), the browser applies same-origin and CORS rules, the server authenticates and authorises the request, and the client processes the response. Vulnerabilities arise at each stage: an unsafe method (GET used for state changes), an over-permissive CORS policy, a missing CSRF token, an unvalidated redirect, or a response rendered through `.html()`. OWASP's Web Frontend Security Cheat Sheet emphasises that **all data should be considered untrusted** — client input, API responses, third-party integrations, cached responses, and browser storage — unless explicitly validated and safely handled. It also warns against relying on client-side logic for security and against using `eval()` or `new Function()`.

**Beginner-Friendly Explanation**
AJAX security is not one thing — it is a chain. A weak link anywhere lets an attacker in. If your AJAX call uses HTTP instead of HTTPS, someone on the network can read it. If your CORS policy says “allow everyone”, any site can read your users' data. If you use cookies for authentication and do not use CSRF tokens, another site can make requests on your users' behalf. If you take the response and put it into `.html()`, the response can run scripts. AJAX security is about checking every link in the chain.

---

### Purposes

- **To protect data in transit** from interception and tampering.
- **To prevent unauthorised cross-origin reads** of sensitive responses.
- **To ensure that state-changing requests cannot be forged** by third-party sites.
- **To prevent AJAX responses from becoming an XSS vector** when rendered.
- **To enforce access decisions on the server**, not the client.

---

### Syntax Rules and Structure

| Risk | Control |
|---|---|
| Data interception | HTTPS for all AJAX endpoints |
| Cross-origin data theft | CORS with explicit origin allowlist |
| Forged state-changing requests | CSRF tokens (cookie auth) or header-based auth |
| XSS via response rendering | `.text()` / sanitisation, never `.html()` with untrusted data |
| Client-side logic bypass | Server-side enforcement of all rules |

---

### Constraints and Limitations

- **Client-side validation is not security**; it is UX. Attackers can bypass it.
- **AJAX requests are visible** in DevTools and can be replayed or modified.
- **The browser is not a trust boundary**; the server is.

---

### Annotated Code Example

```javascript
// ❌ INSECURE: HTTP endpoint, no CORS control, .html() sink
$.ajax({
  url: "http://api.example.com/data",     // cleartext
  type: "GET",
  success: function (data) {
    $("#output").html(data);               // XSS sink
  }
});

// ✅ SECURE: HTTPS, explicit dataType, safe rendering
$.ajax({
  url: "https://api.example.com/data",    // encrypted
  type: "GET",
  dataType: "json",                        // parse as JSON, not script
  success: function (data) {
    $("#output").text(data.message);       // safe sink
  }
});
```

**Expected Output**
Both calls fetch data. The insecure version transmits in cleartext and renders the response as HTML, creating an XSS risk. The secure version encrypts the request and renders the response as text, preventing script execution.

**Why this output occurs**: HTTPS encrypts the transport; `dataType: "json"` ensures the response is parsed as data, not executed; `.text()` treats the content as plain text.

---

### Real-World Cases

- **API endpoints**: All AJAX APIs should be served over HTTPS with CORS configured to allow only trusted origins.
- **User-generated content**: AJAX responses containing user text must be rendered with `.text()`, not `.html()`.
- **State-changing operations**: POST/PUT/DELETE via AJAX with cookie auth require CSRF tokens.

---

## Core Concept 2: CORS (Cross-Origin Resource Sharing)

### Definitions

**Core Definition**
CORS is a browser-enforced security mechanism that allows a server to declare which origins (scheme + host + port) are permitted to read responses from cross-origin requests, by returning specific `Access-Control-Allow-*` headers.

**Technical Definition**
CORS works through a negotiation between the browser and the server. For **simple requests** (GET, HEAD, POST with limited content types), the browser sends the request and the server responds with `Access-Control-Allow-Origin`. For **preflighted requests** (custom headers, methods like PUT/DELETE, certain content types), the browser first sends an `OPTIONS` request; the server must respond with `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, and optionally `Access-Control-Allow-Credentials`. If the request includes credentials (cookies, HTTP auth), the client must set `xhrFields: { withCredentials: true }` in jQuery, and the server must respond with `Access-Control-Allow-Credentials: true` and an **explicit origin** — the wildcard `*` is invalid when credentials are allowed. jQuery's `$.ajax()` handles CORS transparently, but the security policy is entirely server-side.

**Beginner-Friendly Explanation**
CORS is the browser's way of asking the server: “Is it okay for this other website to read your response?” The server answers with a header. If the answer is no, the browser blocks the page from seeing the data. The important part for developers is that **CORS does not stop the request from being sent** — it stops the response from being read. And if you allow credentials, you cannot use a wildcard; you must name the specific origin. jQuery can send credentials with `withCredentials: true`, but the server must explicitly allow it.

---

### Purposes

- **To control which origins can read AJAX responses**.
- **To enable legitimate cross-origin API access** without disabling the same-origin policy globally.
- **To support credentialed cross-origin requests** when necessary, with explicit server-side consent.
- **To prevent malicious sites from reading sensitive data** returned by APIs.

---

### Syntax Rules and Structure

#### jQuery Client-Side Configuration

```javascript
$.ajax({
  url: "https://api.example.com/data",
  type: "GET",
  dataType: "json",
  xhrFields: {
    withCredentials: true          // send cookies with cross-origin request
  }
});
```

#### Server-Side Response Headers (Conceptual)

```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, X-CSRF-Token
```

---

### Constraints and Limitations

- **Wildcard `*` cannot be used with credentials**; an explicit origin is required.
- **CORS is enforced by the browser**, not the server; non-browser clients can ignore it.
- **Preflight requests add latency**; cache preflight responses where possible.
- **Misconfigured CORS is a common data-leak vector**; review the origin allowlist carefully.

---

### Annotated Code Example

```javascript
// ✅ CREDENTIALED CROSS-ORIGIN AJAX WITH CORS
$.ajax({
  url: "https://api.example.com/user",
  type: "GET",
  dataType: "json",
  xhrFields: {
    withCredentials: true            // include cookies
  },
  success: function (data) {
    $("#profile").text(data.name);
  },
  error: function (xhr) {
    // CORS failure often surfaces as a network error
    console.log("Request failed or CORS blocked:", xhr.status);
  }
});
```

**Expected Output**
If the server responds with `Access-Control-Allow-Origin: https://app.example.com` and `Access-Control-Allow-Credentials: true`, the response is readable and the profile is displayed. If the origin is not allowed, the browser blocks the response and the `error` callback fires.

**Why this output occurs**: The `withCredentials` flag tells the browser to include cookies; the server’s CORS headers determine whether the response is exposed to the calling script.

---

### Real-World Cases

- **Third-party APIs**: Allow specific partner origins with credentials disabled and wildcard origins.
- **Single-page applications**: The SPA origin is explicitly allowlisted on the API server.
- **Microservices**: Each service configures CORS independently; ensure consistency.

---

## Core Concept 3: JSONP Security

### Definitions

**Core Definition**
JSONP (JSON with Padding) is a legacy cross-domain technique that works by injecting a `<script>` tag whose `src` points to a remote endpoint, which responds with a JavaScript function call wrapping the data — meaning the response is **executed as JavaScript** in the page's context.

**Technical Definition**
JSONP bypasses the same-origin policy by exploiting the fact that `<script>` tags can load resources from any domain. The remote server wraps its JSON response in a callback function name, and the browser executes the response as JavaScript. This is inherently unsafe with untrusted endpoints because the endpoint can return **arbitrary JavaScript** that runs with full access to the page's DOM, cookies, and data. jQuery's `$.getJSON()` and `$.ajax()` may automatically fall back to JSONP for cross-domain requests under certain conditions, and the jQuery frontend security specification explicitly states: **"JSONP MUST be disabled unless the endpoint is fully trusted (and even then, avoid)"** . The recommended fix is to set `jsonp: false` and use `dataType: "json"` with CORS.

**Beginner-Friendly Explanation**
JSONP is a trick from the days before CORS. Instead of asking the server for data, you ask it for a script. The server responds with something like `myCallback({"data": "value"})`. Your page runs that script, and the callback receives the data. The problem is that the server can put **anything** in that script — not just data. If the server is malicious or compromised, it can run code that steals your users' cookies, reads their data, or performs actions on their behalf. jQuery might use JSONP automatically for cross-domain requests, so you must explicitly disable it for any endpoint you do not fully control.

---

### Purposes

- **To understand why JSONP is dangerous** and why it should not be used with untrusted endpoints.
- **To recognise jQuery's automatic JSONP fallback** and disable it.
- **To migrate from JSONP to CORS** for legitimate cross-origin data access.
- **To identify JSONP usage in existing code** (`dataType: "jsonp"`, `callback=?`).

---

### Syntax Rules and Structure

#### Unsafe (JSONP)

```javascript
$.ajax({
  url: "https://untrusted.example.com/data",
  dataType: "jsonp",                  // executes response as JavaScript
  jsonp: "callback"
});
```

#### Safe (JSON + CORS)

```javascript
$.ajax({
  url: "https://trusted.example.com/data",
  dataType: "json",                    // parse as data, not script
  jsonp: false,                         // explicitly disable JSONP fallback
  success: function (data) { … }
});
```

---

### Constraints and Limitations

- **JSONP has no error handling** for script execution failures.
- **JSONP cannot send custom headers** or use non-GET methods.
- **jQuery may fall back to JSONP automatically** in some cross-domain scenarios; explicit `jsonp: false` is required for defense in depth.
- **JSONP endpoints can execute arbitrary code**; they must be fully trusted.

---

### Annotated Code Example

```javascript
// ❌ DANGEROUS: JSONP with an untrusted endpoint
$.getJSON("https://untrusted.example.com/data?callback=?", function (data) {
  $("#output").text(data.value);
});
// The response could be: alert(document.cookie); // not just data

// ✅ SAFE: JSON with CORS, JSONP explicitly disabled
$.ajax({
  url: "https://api.example.com/data",
  type: "GET",
  dataType: "json",
  jsonp: false,                         // prevent automatic fallback
  success: function (data) {
    $("#output").text(data.value);
  }
});
```

**Expected Output**
The JSONP version executes whatever JavaScript the remote server returns. The CORS version receives only parsed JSON data and never executes the response as code.

**Why this output occurs**: JSONP works by script injection; the browser executes the response. JSON with CORS is parsed as data; the browser does not execute it.

---

### Real-World Cases

- **Legacy APIs**: Some older APIs still only support JSONP; migrate to CORS where possible.
- **Third-party widgets**: Avoid JSONP for any endpoint not fully controlled by the application.
- **Code audits**: Search for `dataType: "jsonp"` and `callback=?` to identify legacy JSONP usage.

---

## Core Concept 4: HTTPS

### Definitions

**Core Definition**
HTTPS (HTTP over TLS) is the secure version of HTTP that encrypts all communication between the browser and the server, preventing eavesdropping, tampering, and man-in-the-middle attacks on AJAX requests and responses.

**Technical Definition**
HTTPS uses TLS (Transport Layer Security) to provide confidentiality (encryption), integrity (message authentication), and authentication (server identity via certificates). For AJAX security, HTTPS is mandatory: without it, an attacker on the network can read request bodies and response data, inject malicious content, or redirect requests. The **only secure way to make an HTTPS AJAX call is from an HTTPS page**; if the page itself is served over HTTP, an attacker can use SSL stripping to force the AJAX call to an insecure endpoint even if the URL specifies `https://`. Best practice is to serve the entire application over HTTPS, use HSTS to enforce it, and never mix HTTP and HTTPS content.

**Beginner-Friendly Explanation**
HTTPS is the sealed envelope for your AJAX requests. Without it, anyone on the same Wi-Fi — or any network between you and the server — can read what you send and what the server sends back. They can also change the response to include malicious code. But there is a catch: if your page is loaded over HTTP, an attacker can strip the “S” from HTTPS and force your AJAX call to go over HTTP, even if you typed `https://` in the URL. The only reliable way to secure AJAX is to serve the entire page over HTTPS and use HTTPS URLs for all API calls.

---

### Purposes

- **To encrypt AJAX request and response data** in transit.
- **To prevent man-in-the-middle attacks** and response tampering.
- **To authenticate the server** via TLS certificates.
- **To satisfy compliance and privacy requirements** for sensitive data.
- **To avoid mixed-content warnings and SSL-stripping vulnerabilities**.

---

### Syntax Rules and Structure

```javascript
// ✅ SECURE: HTTPS page + HTTPS AJAX URL
$.ajax({
  url: "https://api.example.com/data",
  type: "GET",
  dataType: "json"
});

// ❌ UNSAFE: HTTP page or HTTP AJAX URL
$.ajax({
  url: "http://api.example.com/data",   // cleartext
  type: "GET"
});
```

**Server configuration**: Redirect all HTTP to HTTPS; set `Strict-Transport-Security` (HSTS); use secure cookies.

---

### Constraints and Limitations

- **HTTPS must be end-to-end**; a page served over HTTP cannot securely call an HTTPS endpoint.
- **Mixed content** (HTTP resources on an HTTPS page) is blocked by modern browsers.
- **Certificate management** is required; expired or misconfigured certificates break AJAX calls.

---

### Annotated Code Example

```javascript
// ✅ ENFORCE HTTPS FOR ALL AJAX REQUESTS
function secureAjax(options) {
  options.url = options.url.replace(/^http:\/\//, "https://");
  return $.ajax(options);
}

secureAjax({
  url: "http://api.example.com/data",   // automatically upgraded
  type: "GET",
  dataType: "json"
});
```

**Expected Output**
The request is sent to `https://api.example.com/data` even though the caller supplied an HTTP URL.

**Why this output occurs**: The helper replaces the `http://` scheme with `https://` before dispatching the request. This is a defence-in-depth measure, but the page itself must still be served over HTTPS to prevent SSL stripping.

---

### Real-World Cases

- **Login and payment flows**: All authentication and transaction AJAX calls must use HTTPS.
- **API gateways**: Enforce HTTPS at the edge and reject HTTP requests.
- **HSTS deployment**: Instruct browsers to always use HTTPS for the domain.

---

## Core Concept 5: Authentication

### Definitions

**Core Definition**
Authentication is the process of verifying the identity of the user or client making an AJAX request, typically through credentials such as session cookies, bearer tokens, or API keys.

**Technical Definition**
In AJAX contexts, authentication mechanisms fall into two broad categories. **Cookie-based authentication** relies on the browser automatically sending session cookies with same-origin and (if configured) cross-origin requests; it is convenient but vulnerable to CSRF unless protected by tokens or SameSite attributes. **Token-based authentication** (e.g., JWT Bearer tokens) requires the client to explicitly attach an `Authorization` header to each request; it is not vulnerable to CSRF because the browser does not automatically attach the token. For single-page applications, storing tokens in **HttpOnly, Secure, SameSite cookies** is recommended to protect them from XSS, or keeping access tokens in **JavaScript memory only** and using refresh tokens in secure cookies. The Auth0 guidance describes using a backend-for-frontend pattern where the backend authenticates the user and sets a session cookie for subsequent AJAX calls.

**Beginner-Friendly Explanation**
Authentication answers the question: “Who are you?” When you log in, the server gives you something that proves your identity — a session cookie or a token. For AJAX, the key question is how that proof is attached to each request. With cookies, the browser attaches it automatically, which is convenient but means another site can trick the browser into sending it (CSRF). With tokens, your JavaScript must attach it explicitly, which means another site cannot forge requests — but you must store the token safely. The safest modern pattern is to keep tokens out of JavaScript-accessible storage and use secure, HttpOnly cookies or a backend proxy.

---

### Purposes

- **To verify the identity of the requester** on every AJAX call.
- **To protect against credential theft** through secure storage and transmission.
- **To choose the right mechanism** (cookies vs. tokens) based on the threat model.
- **To avoid CSRF vulnerabilities** by using header-based tokens or CSRF protection with cookies.
- **To integrate with identity providers** (OAuth 2.0, OpenID Connect) securely.

---

### Syntax Rules and Structure

| Mechanism | How AJAX Sends It | CSRF Risk |
|---|---|---|
| **Session cookie** | Browser automatically attaches | High — needs CSRF token or SameSite |
| **Bearer token (header)** | `headers: { Authorization: "Bearer " + token }` | Low — not automatically attached |
| **HttpOnly cookie** | Browser automatically attaches | Same as session cookie |

#### jQuery Bearer Token Example

```javascript
$.ajax({
  url: "https://api.example.com/data",
  type: "GET",
  headers: {
    Authorization: "Bearer " + accessToken
  },
  dataType: "json"
});
```

---

### Constraints and Limitations

- **Storing tokens in `localStorage` is vulnerable to XSS**; prefer HttpOnly cookies or in-memory storage.
- **Cookie-based auth requires CSRF protection**; header-based auth does not have CSRF risk.
- **Tokens must be short-lived** and refreshed securely.

---

### Annotated Code Example

```javascript
// ✅ BEARER TOKEN AUTHENTICATION (header-based, no CSRF risk)
var accessToken = sessionStorage.getItem("access_token"); // short-lived token

$.ajax({
  url: "https://api.example.com/profile",
  type: "GET",
  dataType: "json",
  headers: {
    Authorization: "Bearer " + accessToken
  },
  success: function (data) {
    $("#profile").text(data.name);
  },
  error: function (xhr) {
    if (xhr.status === 401) {
      // token expired — redirect to login
      window.location.href = "/login";
    }
  }
});
```

**Expected Output**
The request carries the `Authorization` header. The server validates the token and returns the profile. If the token is invalid or expired, the server returns `401`, and the client redirects to login.

**Why this output occurs**: The token is attached explicitly via the header; the browser does not attach it automatically, so a cross-site request would not include it.

---

### Real-World Cases

- **SPA authentication**: Use Auth0 or a similar provider with HttpOnly cookies or a backend-for-frontend pattern.
- **API clients**: Use short-lived JWT access tokens and refresh tokens.
- **Legacy cookie auth**: Add CSRF tokens and `SameSite` attributes.

---

## Core Concept 6: Authorization

### Definitions

**Core Definition**
Authorization is the process of determining what an authenticated user is permitted to do, and enforcing those permissions on every AJAX request on the server side.

**Technical Definition**
Authorization must be **deny-by-default** and **checked on every request**, including AJAX calls. OWASP's Authorization Cheat Sheet states that permission should be validated correctly on every request, regardless of whether the request was initiated by an AJAX script, server-side code, or any other source. Client-side hiding of UI elements is not authorization; an attacker can craft AJAX requests directly. The server must verify that the authenticated user has permission to perform the requested action on the specific resource (preventing insecure direct object references, IDOR). Application-wide mechanisms such as filters or middleware are preferred over per-method checks.

**Beginner-Friendly Explanation**
Authorization answers: “What are you allowed to do?” Authentication proves you are who you say you are; authorization decides whether you can delete that post, view that invoice, or change that setting. The critical rule for AJAX is that **the server must check permissions on every request**. If your jQuery code hides the “Delete” button for non-admin users, that is not security — an attacker can still send the AJAX request directly. The server must reject it.

---

### Purposes

- **To enforce access control on every AJAX endpoint**.
- **To prevent privilege escalation** and insecure direct object references.
- **To implement deny-by-default policies**.
- **To centralise authorization logic** in middleware or filters.
- **To ensure that hiding UI elements is never treated as a security control**.

---

### Syntax Rules and Structure

| Principle | Implementation |
|---|---|
| **Deny by default** | Reject unless explicitly permitted. |
| **Check every request** | Authorise AJAX calls the same as server-side calls. |
| **Object-level checks** | Verify the user can access the specific resource (prevent IDOR). |
| **Centralised enforcement** | Use middleware/filters, not scattered per-method checks. |

---

### Constraints and Limitations

- **Client-side checks are bypassable**; never rely on them.
- **Authorisation must be server-side**; the client cannot be trusted.
- **Role-based and attribute-based models** must be applied consistently.

---

### Annotated Code Example (Conceptual)

```javascript
// ❌ INSECURE: client-side hiding only
if (user.role !== "admin") {
  $("#deleteBtn").hide();             // attacker can still send the AJAX request
}

// ✅ SECURE: server-side authorisation on every request
// Client sends the request; server rejects if not authorised.
$.ajax({
  url: "/api/posts/123",
  type: "DELETE",
  success: function () { console.log("Deleted"); },
  error: function (xhr) {
    if (xhr.status === 403) {
      console.log("Not authorised");   // server enforced the rule
    }
  }
});
```

**Expected Output**
The insecure version hides the button but the server may still process the request if called directly. The secure version relies on the server to return `403 Forbidden` when the user lacks permission.

**Why this output occurs**: The server checks the user's permissions before executing the action. The client-side hiding is only a UX convenience.

---

### Real-World Cases

- **Admin panels**: Every AJAX action must be authorised server-side.
- **API endpoints**: Use middleware to enforce role-based access control.
- **Object-level permissions**: Verify that the user owns or has access to the specific record.

---

## Core Concept 7: AJAX Security Best Practices

### Definitions

**Core Definition**
jQuery AJAX security best practices are the consolidated conventions — HTTPS everywhere, CORS with explicit origins, JSONP disabled, authentication with secure token storage, server-side authorisation, CSRF protection for cookie auth, and safe response rendering — that together prevent the most common AJAX-related vulnerabilities.

**Technical Definition**
Best practices synthesise the individual controls: (1) **serve all pages and AJAX endpoints over HTTPS**; (2) **configure CORS with an explicit origin allowlist**, never `*` with credentials; (3) **disable JSONP** (`jsonp: false`) and use `dataType: "json"` with CORS; (4) **use header-based authentication** (Bearer tokens) where possible, or **protect cookie-based auth with CSRF tokens**; (5) **store tokens securely** (HttpOnly cookies or in-memory) and keep them short-lived; (6) **enforce authorisation on the server** for every AJAX request; (7) **render responses safely** with `.text()` or sanitisation, never `.html()` with untrusted data; and (8) **validate all input and encode all output** at the appropriate context. OWASP's "TOP 10 Secure Coding Requirements for AJAX" additionally requires server-side validation of all data entry points and AJAX URLs for injection vulnerabilities, and enforcement of business logic on the server.

**Beginner-Friendly Explanation**
Best practices are the habits that make AJAX secure. Use HTTPS. Lock CORS down to your own sites. Turn off JSONP. Use tokens or CSRF protection. Store tokens safely. Check permissions on the server. Display responses as text, not HTML. Validate input and encode output. None of these is optional if the data matters. Together they close the gaps that attackers use.

---

### Purposes

- **To provide a single actionable checklist** for secure AJAX.
- **To prevent the most common AJAX vulnerabilities** (data interception, XSS, CSRF, CORS misconfiguration, JSONP injection).
- **To balance security with usability** through correct mechanism selection.
- **To establish a baseline for code reviews and security audits**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Transport** | HTTPS for all pages and endpoints | HTTP or mixed content |
| **CORS** | Explicit origin allowlist | `*` with credentials |
| **JSONP** | `jsonp: false`, `dataType: "json"` | `dataType: "jsonp"` |
| **Auth** | Bearer tokens or CSRF-protected cookies | Tokens in `localStorage` |
| **Authorisation** | Server-side check on every request | Client-side hiding only |
| **Response rendering** | `.text()` or sanitised `.html()` | `.html(untrusted)` |
| **Input** | Server-side validation | Trusting client validation |

---

### Annotated Code Example (Consolidated)

```javascript
// ✅ AJAX SECURITY BEST PRACTICES — CONSOLIDATED

// 1. HTTPS endpoint
// 2. CORS with explicit origin (server-side configuration)
// 3. JSONP disabled
// 4. Bearer token authentication
var token = sessionStorage.getItem("access_token"); // short-lived

$.ajax({
  url: "https://api.example.com/data",
  type: "GET",
  dataType: "json",                      // parse as data
  jsonp: false,                          // disable JSONP fallback
  headers: {
    Authorization: "Bearer " + token     // header-based auth, no CSRF risk
  },
  success: function (data) {
    // 5. Safe rendering — no .html() with untrusted data
    $("#output").text(data.message);
  },
  error: function (xhr) {
    if (xhr.status === 401) {
      // handle token expiry
      window.location.href = "/login";
    }
  }
});

// 6. Server-side authorisation is enforced independently
// 7. Server validates all input and enforces business logic
```

**Expected Output**
The request is encrypted, CORS-controlled, JSONP-disabled, authenticated with a header token, and rendered safely. The server authorises the request and returns data. If any control fails, the request is rejected or the response is not exposed.

**Why this output occurs**: Each practice closes a specific gap: HTTPS encrypts; CORS restricts origins; `jsonp: false` prevents script execution; the Bearer header avoids CSRF; `.text()` avoids XSS; server-side checks prevent unauthorised access.

---

### Real-World Cases

- **Code reviews**: A checklist ensures that new AJAX code follows secure patterns.
- **Legacy migration**: Replace JSONP with CORS, move tokens out of `localStorage`, add CSRF tokens.
- **Security audits**: Verify HTTPS, CORS policies, authentication mechanisms, and authorisation checks.

---

## References

- Web Frontend Security Cheat Sheet (formerly AJAX Security Cheat Sheet) | OWASP — https://cheatsheetseries.owasp.org/cheatsheets/Web_Frontend_Security_Cheat_Sheet.html
- jQuery Frontend Security Spec (jQuery 4.0.x, modern browsers) — https://cdn.jsdelivr.net/gh/openai/skills@e6afb0d74cc75d220df2faf3dd6c635c2dc6a108/skills/.curated/security-best-practices/references/javascript-jquery-web-frontend-security.md
- Cross-Site Request Forgery Prevention Cheat Sheet | OWASP — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- Authorization Cheat Sheet | OWASP — https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- Enable Cross-Origin Requests (CORS) in ASP.NET Core | Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/cors
- Authenticate Single-Page Apps With Cookies | Auth0 — https://auth0.com/docs/manage-users/cookies/spa-authenticate-with-cookies
- Is getJSON() safe to call on untrusted URL? | Stack Overflow — https://stackoverflow.com/questions/29023066/is-getjson-safe-to-call-on-untrusted-url
- If on a page using https, if that page does a post with the ajax command in jquery, does that post use https? | Stack Overflow — https://stackoverflow.com/questions/36160977
- TOP 10 Secure Coding Requirements for AJAX | OWASP — https://wiki.owasp.org/index.php/TOP_10_Secure_Coding_Requirements_for_AJAX
- JWT Best Practices for Web & Mobile Apps | Duende — https://duendesoftware.com/products/identityserver/jwt-best-practices
- Secure Token Storage for SPAs: Avoiding XSS Theft | Safeguard — https://safeguard.sh/resources/blog/secure-token-storage-spas