# CSRF Protection — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Cross-Site Request Forgery (CSRF)** is a web security vulnerability in which a malicious site, email, or program causes an authenticated user’s browser to perform an **unwanted state-changing action** on a trusted site. The attack exploits the browser’s automatic attachment of **ambient credentials** — session cookies, HTTP authentication, and client certificates — to requests, without the user’s knowledge or consent.

**Technical Definition.** CSRF occurs when three conditions align: (1) the target uses **ambient credentials** (cookies) for authentication, (2) the attacker can **reconstruct a valid request** without knowing a secret, and (3) the request has a **state-changing effect** on the server. Defenses work by breaking the second condition: introducing a value into the request that the attacker cannot predict, obtain, or reconstruct from outside the application — typically a **CSRF token** bound to the user’s session or browser context, or by restricting credential attachment through the **`SameSite` cookie attribute**.

**Beginner-Friendly Explanation.** CSRF is **tricking a logged-in user’s browser into doing something they did not intend**. Imagine you are logged into your bank in one tab and visit a malicious site in another. That malicious site can silently submit a form to your bank’s “transfer money” endpoint. Because your browser automatically sends your session cookie with the request, the bank thinks **you** initiated it. CSRF protection is the set of techniques that lets the bank tell the difference between a request you actually made and one forged by another site.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Browser-mediated** | The attack relies on the browser’s automatic credential attachment; non-browser clients (curl, Postman, server-to-server) are not CSRF vectors. |
| **State-changing focus** | CSRF targets requests that **modify state** (POST, PUT, PATCH, DELETE), not safe reads. |
| **Ambient credential dependency** | The attack works because cookies are sent automatically; APIs authenticated with bearer tokens in headers are **not vulnerable** to classic CSRF. |
| **Defense-in-depth** | No single control is sufficient; tokens, `SameSite` cookies, `Origin`/`Sec-Fetch-Site` validation, and custom headers layer together. |
| **XSS defeats CSRF defenses** | A cross-site scripting vulnerability can bypass **all** CSRF mitigations, because the attacker can read the token or issue requests directly from the trusted origin. |
| **Framework-dependent** | Many frameworks (Django, Rails, ASP.NET Core, Laravel) provide built-in CSRF protection; Express historically relied on the now-deprecated `csurf`. |

---

### Prerequisites

- **HTTP fundamentals**: cookies, `Set-Cookie`, `SameSite`, `Origin`, `Referer`, and state-changing methods.
- **Browser security model**: the same-origin policy (SOP) and how cookies are attached to cross-site requests.
- **Express.js basics**: middleware, routing, and session management (for the Node.js examples).
- **Node.js and npm**: to install `csrf-csrf`, `cookie-parser`, and related packages.
- **A browser and DevTools**: to observe `Set-Cookie` headers and request behavior.

---

### Related Programming Areas

- **Security fundamentals**: CSRF is a direct consequence of the browser’s ambient-credential model.
- **Authentication and session management**: cookie-based sessions are the primary CSRF vector.
- **CORS**: cross-origin resource sharing and CSRF are related but distinct; CORS does not prevent CSRF by itself.
- **SameSite cookies**: the browser-level defense that reduces (but does not eliminate) CSRF risk.
- **Content Security Policy (CSP)**: prevents XSS, which would otherwise defeat CSRF tokens.

---

## Core Concepts / Features

---

## 1. CSRF (Cross-Site Request Forgery)

### Definitions

**Core Definition.** **CSRF** is an attack that forces an authenticated user’s browser to execute an unintended state-changing request against a web application where the user is currently authenticated.

**Technical Definition.** The attack exploits the browser’s **credential-attachment model**: when the browser sends a request to a target origin, it automatically includes cookies scoped to that origin, regardless of which site initiated the request. If the server authenticates solely by the presence of a session cookie and does not verify **request intent** (via a token, origin check, or custom header), it cannot distinguish a legitimate user action from a forged cross-site request.

**Beginner-Friendly Explanation.** CSRF is **a forged request that carries your credentials**. You are logged in; an attacker makes your browser send a request to a site you trust; the site sees your cookie and thinks it is you. The defense is to require **proof of intent** that the attacker cannot forge — a token, a header, or a cookie attribute that blocks cross-site sending.

### Purposes

- **To understand the attack surface** that cookie-based authentication creates.
- **To motivate the deployment of token-based, origin-based, and cookie-attribute defenses**.
- **To recognize that CSRF is a browser-only vector** and that APIs using header-based bearer tokens are not classically vulnerable.
- **To prioritize CSRF protection** for any endpoint that performs state-changing operations on behalf of an authenticated user.

### Syntax Rules and Structure

CSRF is an **attack pattern**, not a syntax. The defense “syntax” consists of the mechanisms that break the attack:

| Defense Mechanism | What It Breaks |
|---|---|
| **CSRF token** (synchronizer or double-submit) | The attacker cannot reconstruct a valid request without the token. |
| **`SameSite` cookie attribute** | The browser does not attach the session cookie to cross-site requests. |
| **`Origin` / `Sec-Fetch-Site` header validation** | The server can detect that the request originated from a different site. |
| **Custom request header requirement** | The attacker cannot set custom headers cross-origin without CORS permission. |

**Constraints and limitations.**

- **CSRF protection does not protect against XSS**; an XSS vulnerability can read the token or issue same-origin requests directly.
- **`SameSite` alone is not a complete solution**; `Lax` permits top-level GET requests, and `None` disables the protection entirely.
- **Non-browser clients are unaffected**; CSRF is strictly a browser-mediated attack.

### Annotated Complete Step by Step Code Examples

#### Example 1: The vulnerable pattern (no CSRF protection)

```javascript
// ❌ VULNERABLE: state-changing endpoint with no CSRF defense
app.post("/transfer", (req, res) => {
  // Authenticates solely via session cookie.
  const user = req.session.user;
  const { to, amount } = req.body;
  bank.transfer(user.id, to, amount);
  res.json({ success: true });
});
```

**Expected Attack (conceptual):**
```
A malicious site at evil.example hosts:
<form action="https://bank.example/transfer" method="POST">
  <input name="to" value="attacker">
  <input name="amount" value="1000">
</form>
<script>document.forms[0].submit()</script>

The victim's browser sends the request with the bank's session cookie.
The bank sees the valid cookie and executes the transfer.
```

**Why this output occurs.** The server authenticates only by cookie presence. The browser automatically includes the cookie because the request targets the bank’s origin. The server cannot distinguish the forged request from a legitimate one, so the transfer executes.

---

### Real-World Cases

- **Banking and payment endpoints**: transferring funds, changing account details, or authorizing payments.
- **Administrative actions**: creating users, changing permissions, or deleting resources.
- **Social media**: posting content, changing privacy settings, or following accounts on behalf of the victim.

---

## 2. `csurf` (Deprecated)

### Definitions

**Core Definition.** **`csurf`** was the most widely used CSRF protection middleware for Express.js. It is **officially deprecated** by the Express.js Technical Committee and is no longer maintained.

**Technical Definition.** `csurf` provided Express middleware that generated and validated CSRF tokens, typically stored in the session and submitted via a hidden form field or request header. It was deprecated because it had **long been difficult to maintain**, had known design limitations, and the maintainers concluded that CSRF protection is better handled by **frameworks or custom implementations aligned with the application’s architecture**. A vulnerability was identified that the maintainers could not fix, prompting the deprecation and redirection to alternatives like `csrf-csrf`.

**Beginner-Friendly Explanation.** `csurf` was **the standard way to add CSRF protection to Express for years**. It is now **officially retired**. It had maintenance problems and a security issue the maintainers could not fix. **Do not start new projects with `csurf`**. If you are maintaining a legacy Express app that uses it, plan a migration to a modern alternative.

### Purposes

- **To understand why legacy Express applications may still reference `csurf`** and why that is a security concern.
- **To recognize the migration imperative** for applications still using `csurf`.
- **To avoid adopting `csurf` in new development** despite its prevalence in older tutorials and guides.

### Syntax Rules and Structure

**Legacy `csurf` syntax (do not use for new projects).**

```javascript
// ❌ DEPRECATED — do not use in new projects
const csrf = require("csurf");
const csrfProtection = csrf({ cookie: true });

app.get("/form", csrfProtection, (req, res) => {
  res.render("form", { csrfToken: req.csrfToken() });
});

app.post("/submit", csrfProtection, (req, res) => {
  // csurf validates req.body._csrf or the CSRF-TOKEN header
});
```

**Constraints and limitations.**

- **No longer maintained**; security fixes are not being issued.
- **Known design limitations** that contributed to its deprecation.
- **Migration is required** for continued security assurance.

### Annotated Complete Step by Step Code Examples

#### Example 1: Migrating away from `csurf`

```javascript
// ❌ OLD (deprecated): csurf
const csrf = require("csurf");
app.use(csrf({ cookie: true }));

// ✅ NEW (modern): csrf-csrf with double-submit cookie pattern
const { doubleCsrf } = require("csrf-csrf");

const { generateCsrfToken, doubleCsrfProtection } = doubleCsrf({
  getSecret: () => process.env.CSRF_SECRET,
  cookieName: "__Host-csrf",
  cookieOptions: {
    sameSite: "lax",
    secure: true,
    httpOnly: true,
  },
  getTokenFromRequest: (req) => req.headers["x-csrf-token"],
});

app.get("/form", (req, res) => {
  const token = generateCsrfToken(req, res);
  res.json({ csrfToken: token });
});

app.post("/submit", doubleCsrfProtection, (req, res) => {
  res.json({ success: true });
});
```

**Expected Output (conceptual):**
```
GET /form → returns a CSRF token and sets the CSRF cookie.
POST /submit with the correct X-CSRF-Token header → 200 OK.
POST /submit without the token or with a mismatched token → 403 Forbidden.
```

**Why this output occurs.** `csrf-csrf` implements the **double-submit cookie pattern**: the server issues a signed token in a cookie and the client echoes it in a request header. The server compares the header value against the cookie value and validates the signature. Unlike `csurf`, this implementation is stateless and actively maintained.

---

### Real-World Cases

- **Legacy Express applications**: many older tutorials and codebases still reference `csurf`; these must be migrated.
- **Express ecosystem deprecation**: the official Express blog explicitly recommends finding a modern CSRF protection strategy.
- **Security audits**: `csurf` is flagged as an unmaintained dependency with known issues.

---

## 3. `csurf` Alternatives

### Definitions

**Core Definition.** **`csurf` alternatives** are actively maintained packages and patterns that implement CSRF protection for Express and other Node.js frameworks, replacing the deprecated `csurf` middleware.

**Technical Definition.** The principal alternatives implement one of the two canonical token architectures: the **synchronizer token pattern** (stateful; token bound to the session, recommended for stateful software) and the **double-submit cookie pattern** (stateless; token stored in a cookie and echoed in a request header, recommended for stateless software). Notable packages include **`csrf-csrf`** (double-submit), **`csrf-sync`** (synchronizer token), **`@pfeiferio/express-csrf`** (short-lived, single-use tokens), and **`small-csrf`**. Frameworks like NestJS document `csrf-csrf` as the recommended Express-compatible solution.

**Beginner-Friendly Explanation.** Since `csurf` is retired, you need a **modern replacement**. If your app has **sessions** (server-side login state), use a **synchronizer token** package like `csrf-sync`. If your app is **stateless** (no server-side session store, e.g., API-driven with JWTs), use a **double-submit cookie** package like `csrf-csrf`. Both are actively maintained and designed for modern Express.

### Purposes

- **To replace `csurf` with an actively maintained implementation** that receives security updates.
- **To match the CSRF defense to the application’s architecture** — stateful sessions vs. stateless APIs.
- **To avoid the design pitfalls** that contributed to `csurf`’s deprecation.
- **To integrate CSRF protection cleanly** with modern Express patterns and TypeScript.

### Syntax Rules and Structure

**Alternative packages and their patterns.**

| Package | Pattern | Statefulness | Best For |
|---|---|---|---|
| **`csrf-csrf`** | Double-submit cookie (HMAC-signed) | Stateless | APIs, SPAs, serverless |
| **`csrf-sync`** | Synchronizer token | Stateful (session) | Traditional server-rendered apps |
| **`@pfeiferio/express-csrf`** | Short-lived, single-use tokens | Stateless (external store optional) | Multi-instance deployments |
| **`small-csrf`** | Token-based | Configurable | Lightweight applications |

**`csrf-csrf` configuration syntax.**

```javascript
const { doubleCsrf } = require("csrf-csrf");

const { generateCsrfToken, doubleCsrfProtection } = doubleCsrf({
  getSecret: () => process.env.CSRF_SECRET,
  getTokenFromRequest: (req) => req.headers["x-csrf-token"],
  cookieName: "__Host-csrf",
  cookieOptions: { sameSite: "lax", secure: true, httpOnly: true },
});
```

**Constraints and limitations.**

- **The double-submit pattern requires the token to be readable by JavaScript** (non-`HttpOnly` cookie) or delivered via a separate mechanism; the `csrf-csrf` package uses a signed cookie and expects the token in a header.
- **The synchronizer token pattern requires a session store**; stateless deployments cannot use it without external session storage.
- **All alternatives must be kept updated**; dependency management is part of the security posture.

### Annotated Complete Step by Step Code Examples

#### Example 1: `csrf-csrf` with Express (double-submit cookie)

```javascript
const express = require("express");
const cookieParser = require("cookie-parser");
const { doubleCsrf } = require("csrf-csrf");

const app = express();
app.use(express.json());
app.use(cookieParser());

const {
  generateCsrfToken,
  doubleCsrfProtection,
} = doubleCsrf({
  getSecret: () => process.env.CSRF_SECRET, // at least 32 chars
  getTokenFromRequest: (req) => req.headers["x-csrf-token"],
  cookieName: "__Host-csrf",
  cookieOptions: {
    sameSite: "lax",
    secure: true,
    httpOnly: true,
  },
});

// Step 1: Endpoint to obtain a token.
app.get("/csrf-token", (req, res) => {
  const token = generateCsrfToken(req, res);
  res.json({ csrfToken: token });
});

// Step 2: State-changing endpoint protected by CSRF middleware.
app.post("/transfer", doubleCsrfProtection, (req, res) => {
  res.json({ success: true, message: "Transfer executed" });
});

app.listen(3000);
```

**Setup.** `npm install express cookie-parser csrf-csrf`.

**Expected Output (conceptual):**
```
GET /csrf-token → { "csrfToken": "abc123..." } and sets __Host-csrf cookie.
POST /transfer with X-CSRF-Token: abc123... → 200 OK.
POST /transfer without the header → 403 Forbidden (invalid CSRF token).
```

**Why this output occurs.** `generateCsrfToken` creates an HMAC-signed token and sets it in a `__Host-` cookie. The client must read the token (typically from a non-`HttpOnly` source or a meta tag) and send it in the `X-CSRF-Token` header. The `doubleCsrfProtection` middleware compares the header token against the cookie token and verifies the signature. A missing or mismatched token is rejected.

---

#### Example 2: `csrf-sync` with Express sessions (synchronizer token)

```javascript
const session = require("express-session");
const { csrfSync } = require("csrf-sync");

app.use(session({ secret: "session-secret", resave: false, saveUninitialized: true }));

const {
  generateToken,
  csrfSynchronisedProtection,
} = csrfSync();

app.get("/form", (req, res) => {
  const token = generateToken(req);
  res.json({ csrfToken: token });
});

app.post("/submit", csrfSynchronisedProtection, (req, res) => {
  res.json({ success: true });
});
```

**Expected Output (conceptual):**
```
GET /form → { "csrfToken": "..." } and the token is stored in the session.
POST /submit with the token in the body or header → 200 OK.
POST /submit without the token → 403 Forbidden.
```

**Why this output occurs.** `csrf-sync` implements the **synchronizer token pattern**: the token is generated server-side and stored in the user’s session. The client submits it in the request body or header, and the middleware compares it against the session-stored value. This is the recommended pattern for **stateful** applications.

---

### Real-World Cases

- **Express APIs with JWTs**: `csrf-csrf` with double-submit cookies for stateless deployments.
- **Traditional server-rendered apps**: `csrf-sync` with session-backed synchronizer tokens.
- **NestJS applications**: the NestJS documentation recommends `csrf-csrf` for Express-based CSRF protection.

---

## 4. CSRF Tokens

### Definitions

**Core Definition.** A **CSRF token** is a **cryptographically random, unpredictable value** that the server generates and associates with the user’s session or browser context. The client must include the token in state-changing requests, and the server must validate it before processing the request.

**Technical Definition.** CSRF tokens break the attacker’s ability to **reconstruct a valid request**. The token must satisfy three properties: **unpredictability** (generated with cryptographically sufficient randomness), **confidentiality** (never exposed to a cross-origin attacker), and **server-side binding and verification** (checked against a stored, session-associated value using constant-time comparison). The two dominant architectures are the **synchronizer token pattern** (token stored in the session, embedded in HTML forms) and the **double-submit cookie pattern** (token stored in a cookie, echoed in a request header or body).

**Beginner-Friendly Explanation.** A CSRF token is **a secret handshake that only your site and the user’s browser know**. When the user loads a form, the server gives the browser a random token. When the form is submitted, the token must come back. An attacker on another site cannot see the token (because the browser’s same-origin policy blocks reading the response), so they cannot include it in a forged request. The server rejects any state-changing request without the correct token.

### Purposes

- **To prove that a state-changing request originated from the application’s own pages**, not from a cross-site attacker.
- **To provide the primary defense against CSRF** for cookie-authenticated applications.
- **To enable stateless CSRF protection** through the double-submit cookie pattern when no session store is available.
- **To support defense in depth** when combined with `SameSite` cookies and origin validation.

### Syntax Rules and Structure

**Synchronizer token pattern (stateful).**

```
1. Server generates a random token and stores it in the session.
2. Server embeds the token in the rendered HTML form as a hidden field.
3. Client submits the form; the token is included in the body.
4. Server compares the submitted token against the session-stored token.
5. Match → proceed; mismatch or missing → reject.
```

**Double-submit cookie pattern (stateless).**

```
1. Server generates a random (often HMAC-signed) token and sets it as a cookie.
2. Client reads the token (from a non-HttpOnly cookie or a meta tag).
3. Client sends the token in a request header (e.g., X-CSRF-Token) or body.
4. Server compares the header/body token against the cookie token and verifies the signature.
5. Match and valid signature → proceed; otherwise → reject.
```

**Token rules.**

| Rule | Guidance |
|---|---|
| **Use cryptographically secure randomness** | At least 32 bytes of entropy; never use `Math.random()`. |
| **Use constant-time comparison** | Prevent timing side channels during token validation. |
| **Reject missing and empty tokens explicitly** | Do not treat “no token” as a separate code path that may accidentally allow the request. |
| **Bind the token to the session (synchronizer) or sign it (double-submit)** | Unsigned double-submit cookies are vulnerable to cookie injection from subdomains. |
| **Rotate tokens on privilege changes** | Regenerate after login, logout, and role changes. |

**Constraints and limitations.**

- **Tokens do not protect against XSS**; an XSS vulnerability can read the token or bypass it entirely.
- **Per-request token rotation reduces the attack window** but can break back-button usability and increase server load.
- **Double-submit cookies require the token to be readable by JavaScript** (non-`HttpOnly`), which introduces XSS risk if not handled carefully.

### Annotated Complete Step by Step Code Examples

#### Example 1: Synchronizer token pattern (pseudocode)

```python
# Synchronizer token pattern — stateful
def render_transfer_form(request):
    session = get_session(request)
    if "csrf_token" not in session:
        session["csrf_token"] = generate_random_token(32)
    return render_template("transfer_form.html",
                           csrf_token=session["csrf_token"])

def handle_transfer(request):
    session = get_session(request)
    submitted = request.form.get("csrf_token")
    expected = session.get("csrf_token")
    if not submitted or not expected:
        return reject(400, "Missing CSRF token")
    if not constant_time_compare(submitted, expected):
        return reject(403, "Invalid CSRF token")
    # proceed with transfer logic
    return execute_transfer(request)
```

**Expected Output (conceptual):**
```
Legitimate form submission with the correct token → transfer executes.
Forged cross-site submission without the token → 400 or 403 rejected.
Submission with a guessed token → 403 rejected (astronomically improbable).
```

**Why this output occurs.** The token is generated once per session and stored server-side. It is embedded in the form and submitted with the request. An attacker on another origin cannot read the token because the same-origin policy blocks cross-origin response access. The constant-time comparison prevents timing attacks, and the explicit rejection of missing tokens prevents bypasses.

---

#### Example 2: Double-submit cookie pattern with `csrf-csrf` (Express)

```javascript
const { doubleCsrf } = require("csrf-csrf");

const { generateCsrfToken, doubleCsrfProtection } = doubleCsrf({
  getSecret: () => process.env.CSRF_SECRET,
  getTokenFromRequest: (req) => req.headers["x-csrf-token"],
  cookieName: "__Host-csrf",
  cookieOptions: { sameSite: "lax", secure: true, httpOnly: true },
});

app.get("/token", (req, res) => {
  const token = generateCsrfToken(req, res);
  res.json({ csrfToken: token });
});

app.post("/action", doubleCsrfProtection, (req, res) => {
  res.json({ ok: true });
});
```

**Expected Output (conceptual):**
```
GET /token → sets the __Host-csrf cookie and returns the token.
POST /action with X-CSRF-Token matching the cookie → 200 OK.
POST /action with a missing or forged token → 403 Forbidden.
```

**Why this output occurs.** The server signs the token with a secret and sets it as a `__Host-` cookie. The client echoes the token in the `X-CSRF-Token` header. The middleware validates the signature and compares the header value with the cookie value. An attacker cannot forge the signature without the secret, and cannot read the cookie cross-origin.

---

### Real-World Cases

- **Django**: uses the synchronizer token pattern by default (`{% csrf_token %}`).
- **Rails**: uses `protect_from_forgery` with a per-session token.
- **ASP.NET Core**: uses antiforgery tokens (synchronizer pattern) and `Sec-Fetch-Site`/`Origin` validation.
- **Stateless SPAs**: use double-submit cookie patterns via `csrf-csrf` or similar libraries.

---

## 5. SameSite Cookies

### Definitions

**Core Definition.** The **`SameSite` cookie attribute** instructs the browser whether to attach a cookie to **cross-site requests**. It is a browser-enforced **defense-in-depth** mechanism against CSRF, reducing the automatic credential attachment that CSRF relies on.

**Technical Definition.** `SameSite` accepts three values: **`Strict`** (the cookie is never sent in any cross-site context, including top-level navigation), **`Lax`** (the cookie is sent for same-site requests and top-level cross-site **GET** navigations, but not for cross-site POST or embedded requests), and **`None`** (the cookie is sent in all contexts; **no CSRF protection**). As of late 2024, **`Lax` is the default** in Chrome, Edge, and Opera, but **not** in Firefox or Safari.

**Beginner-Friendly Explanation.** `SameSite` tells the browser **when to include the cookie on requests coming from other sites**. `Strict` is the safest: the cookie is never sent when you arrive from another site, even by clicking a link. `Lax` is the practical default: the cookie is sent when you click a link to the site (a top-level GET) but **not** when another site tries to POST a form. `None` turns off the protection entirely. `Lax` stops most CSRF attacks, but it is **not a complete solution** because it still allows top-level GET requests.

### Purposes

- **To prevent the browser from attaching session cookies to cross-site state-changing requests**.
- **To provide a browser-level defense that requires no server-side token logic**.
- **To reduce CSRF risk as part of a defense-in-depth strategy** alongside tokens and origin validation.
- **To align with modern browser defaults** (`Lax` by default) and avoid relying on legacy behavior.

### Syntax Rules and Structure

**General syntax (`Set-Cookie` header).**

```
Set-Cookie: session=abc123; SameSite=Lax; Secure; HttpOnly
```

**Values and behavior.**

| Value | Cross-Site Behavior | CSRF Protection |
|---|---|---|
| **`Strict`** | Cookie **never** sent in any cross-site context, including link navigation. | Strongest; may break usability (e.g., user appears logged out when arriving from a link). |
| **`Lax`** | Cookie sent for same-site requests and **top-level cross-site GET navigations**; not sent for cross-site POST or embedded requests. | Good for POST-based CSRF; **does not protect state-changing GET endpoints**. |
| **`None`** | Cookie sent in **all** cross-site contexts. | **No protection**; required when the cookie must be sent in cross-site iframes or POSTs. |

**SameSite rules.**

| Rule | Guidance |
|---|---|
| **Explicitly set `SameSite`** | Do not rely on browser defaults; explicitly set `Lax` or `Strict`. |
| **Use `Lax` as a practical default** | It balances security and usability for most applications. |
| **Use `Strict` for sensitive transactional pages** | Banking and high-value actions may warrant `Strict`. |
| **`None` requires `Secure`** | Cookies with `SameSite=None` must be marked `Secure` and sent over HTTPS. |
| **Do not set a cookie specifically for a parent domain** | Subdomain sharing can undermine the protection. |

**Constraints and limitations.**

- **`Lax` does not protect state-changing GET requests**; the application must never use GET for state changes.
- **`None` provides no CSRF protection**; it is appropriate only when cross-site cookie sending is genuinely required (e.g., embedded third-party widgets).
- **`SameSite` alone is insufficient** for high-value actions; combine with tokens and origin validation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Setting a `Lax` session cookie in Express

```javascript
app.use(session({
  secret: "session-secret",
  resave: false,
  saveUninitialized: true,
  cookie: {
    sameSite: "lax",
    secure: true,
    httpOnly: true,
  },
}));
```

**Expected `Set-Cookie` Header:**
```
Set-Cookie: connect.sid=...; Path=/; HttpOnly; Secure; SameSite=Lax
```

**Why this output occurs.** The `sameSite: "lax"` option sets the `SameSite=Lax` attribute. The browser will send the cookie for same-site requests and top-level cross-site GET navigations, but **not** for cross-site POST requests. This blocks the most common CSRF vector (a cross-site POST form) while preserving usability for link-based navigation.

---

#### Example 2: `Strict` for a banking session cookie

```javascript
app.use(session({
  secret: "bank-session-secret",
  cookie: {
    sameSite: "strict",
    secure: true,
    httpOnly: true,
  },
}));
```

**Expected `Set-Cookie` Header:**
```
Set-Cookie: connect.sid=...; Path=/; HttpOnly; Secure; SameSite=Strict
```

**Why this output occurs.** `SameSite=Strict` prevents the cookie from being sent in **any** cross-site context, including link navigation. A user who clicks a link to the banking site from an external forum will appear logged out until they navigate within the site. This is a deliberate trade-off: maximum CSRF protection for high-value sessions.

---

### Real-World Cases

- **GitHub**: uses `SameSite=Lax` for session cookies, balancing security and link usability.
- **Banking sites**: often use `SameSite=Strict` for transactional session cookies.
- **Third-party embeds**: cookies that must work in cross-site iframes require `SameSite=None; Secure`, but must be paired with token-based CSRF protection.

---

## 6. CSRF Best Practices

### Definitions

**Core Definition.** **CSRF best practices** are empirically established guidelines for designing and configuring CSRF defenses so that they are **layered, complete, and maintainable** across different application architectures.

**Technical Definition.** Best practices center on **defense in depth** (combining tokens, `SameSite` cookies, and origin validation), **architecture alignment** (synchronizer tokens for stateful apps, double-submit cookies for stateless apps), **correct token lifecycle management** (generation, binding, rotation, and constant-time validation), **header-based origin verification** (`Origin` and `Sec-Fetch-Site`), and **elimination of state-changing GET requests**. OWASP recommends using **built-in framework protection** where available and implementing **at least one defense-in-depth mitigation** beyond the primary token.

**Beginner-Friendly Explanation.** Best practices mean **layering your defenses**. Do not rely on `SameSite` alone; add a token. Do not rely on a token alone; add `Origin` validation and `SameSite`. Use the pattern that matches your architecture: sessions get synchronizer tokens, stateless APIs get double-submit cookies. Always reject state-changing requests that lack a token, use constant-time comparison, and rotate tokens after login. And **never** use GET requests to change state, because `SameSite=Lax` still allows top-level GET navigations.

### Purposes

- **To provide complete CSRF protection** that does not depend on a single control.
- **To match the defense to the application’s architecture**, avoiding the `csurf`-style one-size-fits-all pitfalls.
- **To eliminate bypasses** caused by missing tokens, timing side channels, or state-changing GET endpoints.
- **To maintain protection across deployments and infrastructure changes** through stateless or externally stored designs.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Best Practice |
|---|---|
| **Primary defense** | Synchronizer token (stateful) or signed double-submit cookie (stateless). |
| **Defense in depth** | Combine with `SameSite=Lax` or `Strict` and `Origin`/`Sec-Fetch-Site` validation. |
| **Token generation** | Use cryptographically secure randomness (≥32 bytes) and a separate CSRF secret. |
| **Token validation** | Constant-time comparison; explicitly reject missing tokens. |
| **Token rotation** | Regenerate on login, logout, and privilege changes. |
| **HTTP methods** | Never use GET for state-changing operations; use POST/PUT/PATCH/DELETE. |
| **Framework protection** | Use built-in framework CSRF protection when available. |
| **XSS prevention** | CSRF is defeated by XSS; prevent XSS with CSP, output encoding, and input validation. |

**Header-based origin validation.**

```
Origin: https://app.example.com
Sec-Fetch-Site: same-origin | same-site | cross-site | none
```

- **`Origin`** is sent by browsers on cross-origin requests (and some same-origin POSTs). Compare it against an allow-list of expected origins.
- **`Sec-Fetch-Site`** is sent by modern browsers on every request. Reject `cross-site` for state-changing endpoints.

**Constraints and limitations.**

- **No single control is sufficient**; defense in depth is mandatory.
- **`SameSite` is not a complete solution**; it must be paired with tokens for sensitive actions.
- **Framework protection may not fit every architecture**; custom implementations must follow the same token properties.

### Annotated Complete Step by Step Code Examples

#### Example 1: Layered CSRF defense in Express

```javascript
const { doubleCsrf } = require("csrf-csrf");

// Layer 1: SameSite cookie for session.
app.use(session({
  cookie: { sameSite: "lax", secure: true, httpOnly: true },
}));

// Layer 2: Signed double-submit CSRF token.
const { generateCsrfToken, doubleCsrfProtection } = doubleCsrf({
  getSecret: () => process.env.CSRF_SECRET,
  getTokenFromRequest: (req) => req.headers["x-csrf-token"],
});

app.get("/csrf-token", (req, res) => {
  res.json({ csrfToken: generateCsrfToken(req, res) });
});

// Layer 3: Origin / Sec-Fetch-Site validation for state-changing routes.
function verifyOrigin(req, res, next) {
  const origin = req.headers.origin;
  const allowed = ["https://app.example.com"];
  if (origin && !allowed.includes(origin)) {
    return res.status(403).json({ error: "Origin not allowed" });
  }
  const site = req.headers["sec-fetch-site"];
  if (site && !["same-origin", "same-site"].includes(site)) {
    return res.status(403).json({ error: "Cross-site request blocked" });
  }
  next();
}

app.post("/transfer",
  verifyOrigin,
  doubleCsrfProtection,
  (req, res) => res.json({ success: true })
);
```

**Expected Output (conceptual):**
```
Legitimate same-origin POST with token → 200 OK.
Cross-site POST (Sec-Fetch-Site: cross-site) → 403 blocked by origin check.
Same-origin POST without token → 403 blocked by CSRF middleware.
Same-origin POST with token → 200 OK.
```

**Why this output occurs.** Three independent layers protect the endpoint: `SameSite=Lax` reduces cross-site cookie attachment; `verifyOrigin` rejects requests whose `Origin` or `Sec-Fetch-Site` indicates a cross-site initiator; and `doubleCsrfProtection` rejects requests lacking a valid signed token. An attacker must defeat all three layers, which is impractical without an XSS vulnerability.

---

#### Example 2: Token rotation on login

```javascript
app.post("/login", (req, res) => {
  // ... authenticate ...
  req.session.regenerate((err) => {
    // Session regenerated; issue a new CSRF token.
    const token = generateCsrfToken(req, res);
    res.json({ success: true, csrfToken: token });
  });
});
```

**Expected Output (conceptual):**
```
Successful login → session regenerated and a new CSRF token issued.
The old token is invalidated; requests using it are rejected.
```

**Why this output occurs.** Session fixation and token reuse are mitigated by **regenerating the session and the CSRF token on privilege transitions**. OWASP recommends token rotation on login, logout, and role changes.

---

### Real-World Cases

- **OWASP CSRF Prevention Cheat Sheet**: recommends framework protection, token-based mitigation, `SameSite` cookies, and `Origin`/`Sec-Fetch-Site` validation as layered defenses.
- **ASP.NET Core**: automatically uses `Sec-Fetch-Site` and `Origin` validation alongside antiforgery tokens.
- **Modern Express applications**: use `csrf-csrf` or `csrf-sync` together with `SameSite=Lax` cookies and `Origin` validation for defense in depth.

---

## References

- OWASP — Cross-Site Request Forgery Prevention Cheat Sheet (raw Markdown, verified 2026) - https://raw.githubusercontent.com/OWASP/CheatSheetSeries/master/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md
- Express.js — Spring Cleaning in Express.js: Deprecations and the Path Ahead (May 2025) - https://expressjs.com/es/blog/2025-05-16-express-cleanup-legacy-packages/
- OWASP — SameSite Cookie Attribute - https://community.owasp.org/SameSite
- csrf-csrf — npm Package (v4.x, Double Submit Cookie) - https://www.npmjs.com/package/csrf-csrf
- CSRF Token Mechanisms — Synchronizer Token Pattern and Double-Submit Cookie Pattern (vulnerability notes) - https://raw.githubusercontent.com/parsamajidipour/vulnerability-notes/refs/heads/main/CSRF/11-CSRF-Token-Mechanisms.md
- @pfeiferio/express-csrf — npm Package (modern csurf alternative) - https://www.npmjs.com/package/@pfeiferio/express-csrf
- MDN — SameSite Cookies - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- Microsoft Learn — Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks in ASP.NET Core - https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- CWE — Cross-Site Request Forgery (CWE-352) - https://cwe.mitre.org/data/definitions/352.html
- NestJS — CSRF Protection Documentation - https://docs.nestjs.com/security/csrf
- csrf-sync — npm Package (Synchronizer Token Pattern) - https://www.npmjs.com/package/csrf-sync