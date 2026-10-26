# Secure Cookies — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Secure cookies** are HTTP cookies configured with security attributes — **`HttpOnly`**, **`Secure`**, **`SameSite`**, cookie name **prefixes** (`__Host-` / `__Secure-`), and narrowly scoped **`Domain`** and **`Path`** — that constrain how, where, and over which transport the browser stores and transmits them. A session cookie is a **bearer credential**: whoever holds it is treated as the logged-in user until it expires, which makes cookie configuration one of the most consequential security decisions in a web application.

**Technical Definition.** A cookie is set via the **`Set-Cookie`** response header. Each attribute is **enforced by the browser**, not by the server: `HttpOnly` blocks JavaScript access; `Secure` restricts transmission to HTTPS; `SameSite` controls cross-site attachment; `Max-Age`/`Expires` bound lifetime; `Domain`/`Path` constrain scope. The **`__Host-`** prefix requires the cookie to be `Secure`, host-only (no `Domain`), and `Path=/`; the **`__Secure-`** prefix requires `Secure` and a secure origin. These rules are codified in **RFC 6265bis**, the cookie specification draft that adds `SameSite` and cookie prefixes.

**Beginner-Friendly Explanation.** Secure cookies are **cookies with safety locks**. A cookie is a small piece of data your browser stores and sends back to your site — usually a session identifier that proves you are logged in. If that identifier is stolen, an attacker can impersonate you. Secure cookies add locks: one lock hides the cookie from JavaScript (`HttpOnly`), one ensures it only travels over encrypted HTTPS (`Secure`), one stops other websites from silently using it (`SameSite`), and naming rules lock the cookie to your exact domain (`__Host-`). Getting these right is cheap; getting them wrong is how account takeovers happen.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Browser-enforced** | The server sets attributes; the browser decides whether to store, expose, and transmit the cookie. |
| **Opt-in security** | Browsers accept insecure cookies without warnings; security flags are **not** enabled by default. |
| **Bearer-credential risk** | A stolen session cookie grants immediate account access without a password. |
| **Layered defense** | `HttpOnly` mitigates XSS token theft, `Secure` mitigates MITM, `SameSite` mitigates CSRF, and prefixes lock scope. |
| **Scope-sensitive** | `Domain` and `Path` determine which requests carry the cookie; over-broad scope increases exposure. |
| **Parse-sensitive** | Browser/server parsing discrepancies can undermine prefixes and flags if libraries are outdated. |

---

### Prerequisites

- **HTTP fundamentals**: request/response headers, `Set-Cookie`, `Cookie`, and status codes.
- **HTTPS/TLS basics**: why `Secure` requires a secure channel.
- **Same-origin policy and sites**: how “same-site” differs from “same-origin.”
- **Session management**: how session identifiers are issued and validated.
- **A backend framework**: Express, Django, or PHP for the code examples.

---

### Related Programming Areas

- **Session management**: cookies are the dominant browser session mechanism.
- **XSS prevention**: `HttpOnly` limits the impact of cross-site scripting.
- **CSRF prevention**: `SameSite` is a primary CSRF mitigation.
- **Transport security**: `Secure` and HSTS enforce HTTPS.
- **Input validation and output encoding**: protect the values stored in cookies.

---

## Core Concepts / Features

---

## 1. Secure Cookies

### Definitions

**Core Definition.** **Secure cookies** are cookies whose `Set-Cookie` header includes security attributes that restrict access, transport, and cross-site transmission, reducing the risk of theft, tampering, and forgery.

**Technical Definition.** A secure cookie is constructed by combining the cookie name/value pair with one or more attributes: `HttpOnly`, `Secure`, `SameSite`, `Max-Age`/`Expires`, `Domain`, `Path`, and optionally a **cookie prefix** in the name. The browser enforces each attribute independently. The combination is greater than the sum of its parts: `HttpOnly` stops `document.cookie` reads, `Secure` stops plaintext transmission, `SameSite` stops cross-site attachment, and `__Host-` stops subdomain overwrites.

**Beginner-Friendly Explanation.** A secure cookie is **a cookie with the right locks for its job**. Most cookies on a site are session identifiers, so they need all the locks. A cookie that only stores a UI theme preference needs fewer locks, but a session cookie should almost always be `HttpOnly; Secure; SameSite=Lax` and, when possible, named `__Host-`.

### Purposes

- **To prevent session-token theft** from JavaScript via XSS (`HttpOnly`) and from network eavesdropping (`Secure`).
- **To mitigate cross-site request forgery** by controlling cross-site cookie attachment (`SameSite`).
- **To lock cookie scope** to the exact host and path, preventing subdomain overwrites (`__Host-`, narrow `Domain`/`Path`).
- **To bound credential lifetime** through `Max-Age`/`Expires`, limiting the window of misuse.
- **To provide a defense-in-depth baseline** that remains effective even when other controls fail.

### Syntax Rules and Structure

**General syntax (`Set-Cookie`).**

```http
Set-Cookie: <cookie-name>=<cookie-value>; <attribute1>; <attribute2>; ...
```

**Component breakdown.**

```http
Set-Cookie: __Host-session=abc123; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

- `__Host-session` — **cookie name** with the `__Host-` prefix, which enforces `Secure`, no `Domain`, and `Path=/`.
- `abc123` — **cookie value** (ideally an opaque, high-entropy session identifier).
- `HttpOnly` — hides the cookie from JavaScript.
- `Secure` — sends the cookie only over HTTPS.
- `SameSite=Lax` — sends the cookie on same-site requests and top-level cross-site GET navigations, but not cross-site POSTs.
- `Path=/` — scopes the cookie to the entire host.
- `Max-Age=3600` — expires the cookie after one hour.

**Attribute reference.**

| Attribute | Effect | Typical Secure Value |
|---|---|---|
| **`HttpOnly`** | Blocks `document.cookie` access | Always on session cookies |
| **`Secure`** | HTTPS-only transmission | Always in production |
| **`SameSite`** | Cross-site attachment policy | `Lax` or `Strict` |
| **`Max-Age` / `Expires`** | Lifetime bound | Short for sessions |
| **`Domain`** | Subdomain scope | Omit unless sharing is required |
| **`Path`** | Path scope | Narrowest possible |
| **`__Host-` prefix** | Host-only, `Secure`, `Path=/` | Strongest scope |
| **`__Secure-` prefix** | `Secure` required | Other secure cookies |

**Syntax rules.**

- The `Set-Cookie` header may appear **multiple times** in one response to set multiple cookies.
- Attributes are **case-insensitive** but conventionally written in camel case (`HttpOnly`, `SameSite`).
- `SameSite=None` **requires** `Secure`; otherwise the browser rejects the cookie.
- Cookie prefixes are **name-level rules**: the browser rejects a `__Host-` cookie that has a `Domain`, is not `Secure`, or does not use `Path=/`.

**Constraints and limitations.**

- **Defaults are permissive**: browsers accept cookies without flags and emit no warning.
- **`__Host-` and `__Secure-` can be undermined** by server-side parsing inconsistencies (e.g., Unicode whitespace normalization) if cookie-parsing libraries are outdated.
- **`SameSite` is not a complete CSRF defense**; it does not protect state-changing GET endpoints and is not uniformly defaulted across browsers.
- **`Secure` has no effect on plaintext HTTP**; a cookie without `Secure` can be sent over HTTP.

### Annotated Complete Step by Step Code Examples

#### Example 1: Insecure vs. secure `Set-Cookie`

```http
# ❌ Insecure: no security attributes.
Set-Cookie: sessionid=abc123
# The cookie is exposed to JavaScript, sent over HTTP, and attached to cross-site requests.

# ✅ Secure baseline: HttpOnly, Secure, SameSite.
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict
# JavaScript cannot read it; it is sent only over HTTPS; cross-site requests do not carry it.
```

**Expected Output (conceptual):**
```
Insecure cookie: document.cookie can read "sessionid=abc123"; HTTP requests carry it.
Secure cookie: document.cookie returns nothing for sessionid; HTTPS-only; cross-site POSTs lack the cookie.
```

**Why this output occurs.** The browser enforces each attribute independently. Without `HttpOnly`, the cookie is visible to scripts. Without `Secure`, it travels over plaintext. Without `SameSite=Strict`, it is attached to cross-site requests. The secure baseline closes all three windows.

---

#### Example 2: Hardened session cookie with `__Host-` prefix (Express)

```javascript
// server.js
const express = require("express");
const session = require("express-session");
const app = express();

app.use(session({
  name: "__Host-session",          // __Host- prefix enforces host-only scope
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,                // no JavaScript access
    secure: true,                  // HTTPS only
    sameSite: "lax",               // cross-site POSTs excluded
    path: "/",                     // required by __Host-
    maxAge: 3600_000,              // 1 hour
  },
}));

app.get("/", (req, res) => res.send("ok"));
app.listen(3000);
```

**Expected `Set-Cookie` Header:**
```http
Set-Cookie: __Host-session=s%3A...; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

**Why this output occurs.** The `name` option sets the `__Host-` prefix. The browser accepts the cookie only because it is `Secure`, has `Path=/`, and has **no `Domain`** attribute. The cookie is therefore bound to the exact host and cannot be overwritten by a subdomain.

---

#### Example 3: Full secure cookie with explicit attributes (raw header)

```http
Set-Cookie: __Host-session=8f14e45fceea167a5a36dedd4bea2543; Path=/; Secure; HttpOnly; SameSite=Strict; Max-Age=7200
```

**Expected Browser Behavior:**
```
The cookie is stored only for the exact host.
It is sent only over HTTPS.
It is not readable from JavaScript.
It is not attached to any cross-site request, including top-level link navigation.
It expires after 2 hours.
```

**Why this output occurs.** `__Host-` enforces host-only scope; `Secure` enforces HTTPS; `HttpOnly` blocks script access; `SameSite=Strict` blocks all cross-site attachment; `Max-Age` bounds lifetime. Each attribute is enforced independently by the browser.

---

### Real-World Cases

- **Session cookies**: the most common secure-cookie use case; a stolen session cookie is an immediate account takeover.
- **Authentication tokens**: refresh tokens and remember-me tokens stored in `HttpOnly; Secure` cookies.
- **CSRF tokens**: double-submit cookies with `SameSite` and `Secure`.
- **Preference cookies**: may omit `HttpOnly` if JavaScript must read them, but should still use `Secure` and `SameSite`.

---

## 2. `HttpOnly`

### Definitions

**Core Definition.** **`HttpOnly`** is a cookie attribute that **prevents client-side JavaScript from accessing the cookie** through APIs such as `document.cookie`, while still allowing the browser to send it with HTTP requests.

**Technical Definition.** When a cookie is marked `HttpOnly`, the browser excludes it from the `document.cookie` string and from other script-accessible cookie APIs. The cookie remains fully available to the server via the `Cookie` request header. `HttpOnly` is a **mitigation for XSS token theft**, not a complete XSS defense: an attacker who achieves XSS can still issue requests on the user’s behalf, but cannot directly read the token.

**Beginner-Friendly Explanation.** `HttpOnly` is **a lock that hides the cookie from JavaScript**. If an attacker injects a script into your page, that script can normally read `document.cookie` and steal the session token. With `HttpOnly`, the script sees nothing — the browser keeps the cookie hidden. The script can still make requests, but it cannot copy the token and send it to an attacker’s server.

### Purposes

- **To prevent XSS from directly stealing session identifiers** via `document.cookie`.
- **To reduce the blast radius of an XSS vulnerability** by keeping the credential out of script reach.
- **To protect authentication and session cookies** that never need JavaScript access.
- **To satisfy security baselines** that mandate `HttpOnly` on session cookies.

### Syntax Rules and Structure

**General syntax.**

```http
Set-Cookie: <name>=<value>; HttpOnly
```

**Component breakdown.**

- `HttpOnly` — a **valueless flag**; its presence enables the protection.
- It may appear alongside other attributes: `HttpOnly; Secure; SameSite=Lax`.

**Syntax rules.**

- `HttpOnly` is a **boolean flag**; there is no `HttpOnly=true`.
- It applies to the entire cookie; there is no per-path or per-domain variant.
- It must be set **at cookie creation time**; it cannot be added later by JavaScript.

**Constraints and limitations.**

- **`HttpOnly` does not prevent XSS from making requests**; it only prevents reading the cookie value.
- **Cookies that must be read by JavaScript cannot use `HttpOnly`**; this is a direct trade-off between functionality and security.
- **Some legacy clients or proxies may strip unknown attributes**, though modern browsers support `HttpOnly` universally.

### Annotated Complete Step by Step Code Examples

#### Example 1: `HttpOnly` in Express

```javascript
// server.js
app.get("/login", (req, res) => {
  res.cookie("session", "abc123", {
    httpOnly: true,   // JavaScript cannot read this cookie
    secure: true,
    sameSite: "lax",
  });
  res.send("logged in");
});
```

**Expected `Set-Cookie` Header:**
```http
Set-Cookie: session=abc123; Path=/; HttpOnly; Secure; SameSite=Lax
```

**Expected Browser Behavior:**
```javascript
// In the page console:
document.cookie; // returns "" (the HttpOnly cookie is hidden)
```

**Why this output occurs.** The `httpOnly: true` option adds the `HttpOnly` attribute. The browser stores the cookie but excludes it from `document.cookie`, so scripts cannot read it. The cookie is still sent with same-site HTTPS requests.

---

#### Example 2: Comparing `HttpOnly` and non-`HttpOnly` cookies

```http
# Cookie A: HttpOnly
Set-Cookie: session=secret123; HttpOnly; Secure

# Cookie B: not HttpOnly
Set-Cookie: theme=dark; Secure
```

**Expected Browser Behavior:**
```javascript
document.cookie;
// Returns: "theme=dark"
// The session cookie is absent because it is HttpOnly.
```

**Why this output occurs.** The browser includes only non-`HttpOnly` cookies in `document.cookie`. The session cookie remains inaccessible to scripts, while the theme cookie — which is intentionally readable — is visible. This demonstrates the selective nature of `HttpOnly`.

---

### Real-World Cases

- **Session cookies**: always `HttpOnly` in production applications.
- **Authentication tokens**: refresh tokens and remember-me cookies should be `HttpOnly`.
- **CSRF double-submit cookies**: often **not** `HttpOnly` because the client must read the token to echo it in a header; this is a deliberate exception.

---

## 3. `Secure`

### Definitions

**Core Definition.** **`Secure`** is a cookie attribute that instructs the browser to send the cookie **only over encrypted HTTPS connections**, never over plaintext HTTP.

**Technical Definition.** When `Secure` is present, the browser attaches the cookie only to requests whose scheme is `https` (or otherwise considered secure by the user agent). If the site is served over HTTP, the cookie is either not set or not sent. `Secure` protects against **man-in-the-middle interception** of the cookie on the network.

**Beginner-Friendly Explanation.** `Secure` is **an HTTPS-only lock**. It tells the browser: “never send this cookie over an unencrypted connection.” If someone on the same Wi-Fi tries to sniff your traffic, the cookie never appears in plaintext because the browser refuses to send it over HTTP.

### Purposes

- **To prevent cookie interception over plaintext HTTP** by network attackers.
- **To mitigate man-in-the-middle attacks** that would otherwise capture session tokens.
- **To enforce HTTPS usage** for sensitive cookies in production.
- **To satisfy compliance and security baselines** that require encrypted transport for credentials.

### Syntax Rules and Structure

**General syntax.**

```http
Set-Cookie: <name>=<value>; Secure
```

**Component breakdown.**

- `Secure` — a **valueless flag**; its presence restricts transmission to HTTPS.
- It is frequently combined with `HttpOnly` and `SameSite`.

**Syntax rules.**

- `Secure` is a **boolean flag**; there is no `Secure=true`.
- `SameSite=None` **requires** `Secure`; a cookie with `SameSite=None` but without `Secure` is rejected by the browser.
- The `__Host-` and `__Secure-` prefixes **require** `Secure`.

**Constraints and limitations.**

- **`Secure` has no effect on HTTP**: the cookie is simply not sent over plaintext, which may break development on `http://localhost`.
- **`Secure` does not encrypt the cookie**; it only restricts the transport. The cookie value itself is still visible to the browser and, if `HttpOnly` is absent, to scripts.
- **Some frameworks default `secure` to `false`**; it must be explicitly enabled in production.

### Annotated Complete Step by Step Code Examples

#### Example 1: `Secure` behavior over HTTP vs. HTTPS

```javascript
// server.js
app.get("/set-cookie", (req, res) => {
  res.cookie("session", "abc123", {
    secure: true,    // HTTPS only
    httpOnly: true,
    sameSite: "lax",
  });
  res.send("cookie set");
});
```

**Expected Behavior:**
```
HTTPS request: Set-Cookie is honored; the browser stores the cookie.
HTTP request: the browser refuses to store or send the cookie.
```

**Why this output occurs.** The `Secure` flag is enforced by the browser based on the request scheme. Over HTTPS, the cookie is accepted; over HTTP, the browser ignores it, preventing plaintext exposure.

---

#### Example 2: `Secure` with a `__Secure-` prefix

```http
Set-Cookie: __Secure-session=abc123; Secure; HttpOnly; SameSite=Lax; Path=/
```

**Expected Browser Behavior:**
```
The browser accepts the cookie only if it was set from a secure origin and carries the Secure flag.
The __Secure- prefix itself does not enforce host-only scope; the cookie may still have a Domain attribute if one is provided (but should not, for session cookies).
```

**Why this output occurs.** The `__Secure-` prefix adds a name-level requirement that the cookie must be `Secure` and set from a secure origin. It is weaker than `__Host-` because it permits a `Domain` attribute, but it still prevents insecure-origin overwrites.

---

### Real-World Cases

- **Production session cookies**: always `Secure` when the application is served over HTTPS.
- **Authentication cookies**: `Secure` prevents token leakage on mixed-content or downgrade paths.
- **Development**: `Secure` should be disabled or conditionally enabled for `http://localhost` to avoid breaking local sessions.

---

## 4. `SameSite`

### Definitions

**Core Definition.** **`SameSite`** is a cookie attribute that controls whether the browser attaches the cookie to **cross-site requests**. It is the primary cookie-level defense against **cross-site request forgery (CSRF)**.

**Technical Definition.** `SameSite` accepts three values: **`Strict`** (the cookie is never sent in a cross-site context, including top-level link navigation), **`Lax`** (the cookie is sent on same-site requests and safe top-level cross-site GET navigations, but not cross-site POSTs or subresource requests), and **`None`** (the cookie is sent in all contexts, including cross-site, and **requires `Secure`**). As of late 2024, `Lax` is the default in Chrome, Edge, and Opera, but **not** in Firefox or Safari; the attribute should therefore be set explicitly.

**Beginner-Friendly Explanation.** `SameSite` tells the browser **when to send the cookie when the request comes from another site**. `Strict` is the most protective: the cookie is never sent when you arrive from another site, even by clicking a link. `Lax` is the practical default: the cookie is sent when you click a link to the site, but **not** when another site tries to POST a form or load a hidden image. `None` turns off the protection and requires `Secure`; it is only for genuine cross-site needs.

### Purposes

- **To mitigate CSRF attacks** by preventing cookies from riding on cross-site state-changing requests.
- **To control cross-site cookie attachment** for embedded widgets, iframes, and third-party integrations.
- **To balance security and usability** through `Lax` (the practical default) and `Strict` (the high-security option).
- **To reduce cross-origin information leakage** from cookies attached to cross-site requests.

### Syntax Rules and Structure

**General syntax.**

```http
Set-Cookie: <name>=<value>; SameSite=Strict
Set-Cookie: <name>=<value>; SameSite=Lax
Set-Cookie: <name>=<value>; SameSite=None; Secure
```

**Component breakdown.**

- `SameSite` — the attribute name.
- `Strict` — no cross-site attachment, including top-level navigations.
- `Lax` — same-site plus top-level cross-site GET navigations; excludes cross-site POST and subresources.
- `None` — all contexts; **must** be paired with `Secure`.

**Behavior matrix.**

| Value | Same-Site Requests | Top-Level Cross-Site GET | Cross-Site POST / Subresource | CSRF Protection |
|---|---|---|---|---|
| **`Strict`** | Sent | **Not sent** | Not sent | Strongest |
| **`Lax`** | Sent | Sent | **Not sent** | Good for POST-based CSRF |
| **`None`** | Sent | Sent | Sent | **None** |

**Syntax rules.**

- `SameSite=None` **requires** `Secure`; the combination `SameSite=None` without `Secure` is invalid.
- `SameSite` values are **case-insensitive** in modern browsers, but canonical forms are `Strict`, `Lax`, and `None`.
- Omitting `SameSite` leaves the default to the browser, which varies by vendor.

**Constraints and limitations.**

- **`Lax` does not protect state-changing GET requests**; the application must never use GET for mutations.
- **`Strict` can break usability** for link-based navigation (the user appears logged out after clicking a link from another site).
- **`None` provides no CSRF protection**; it is appropriate only when cross-site cookie sending is genuinely required.
- **`SameSite` is not a complete CSRF solution**; it must be combined with CSRF tokens and origin validation for sensitive actions.

### Annotated Complete Step by Step Code Examples

#### Example 1: `Strict`, `Lax`, and `None` in Express

```javascript
// Strict: no cross-site attachment.
res.cookie("strictSession", token, {
  httpOnly: true,
  secure: true,
  sameSite: "strict",
});

// Lax: top-level GET navigation allowed, cross-site POST blocked.
res.cookie("laxSession", token, {
  httpOnly: true,
  secure: true,
  sameSite: "lax",
});

// None: all cross-site contexts allowed; Secure is mandatory.
res.cookie("crossSiteSession", token, {
  httpOnly: true,
  secure: true,
  sameSite: "none",
});
```

**Expected `Set-Cookie` Headers:**
```http
Set-Cookie: strictSession=...; Path=/; HttpOnly; Secure; SameSite=Strict
Set-Cookie: laxSession=...; Path=/; HttpOnly; Secure; SameSite=Lax
Set-Cookie: crossSiteSession=...; Path=/; HttpOnly; Secure; SameSite=None
```

**Why this output occurs.** Each option sets the corresponding `SameSite` value. The browser then enforces the behavior matrix: `Strict` cookies are absent from all cross-site requests; `Lax` cookies are present on top-level GET navigations but absent from cross-site POSTs; `None` cookies are present in all cross-site contexts.

---

#### Example 2: `Lax` in a cross-site POST scenario

```html
<!-- Attacker site: cross-site form POST to victim site. -->
<form action="https://victim.example/transfer" method="POST">
  <input name="to" value="attacker">
  <input name="amount" value="1000">
</form>
<script>document.forms[0].submit()</script>
```

**Expected Behavior with `SameSite=Lax`:**
```
The browser does NOT attach the victim's Lax session cookie to the cross-site POST.
The request reaches the server without authentication.
The server rejects the request (or treats it as anonymous).
```

**Why this output occurs.** `SameSite=Lax` excludes cookies from cross-site POST requests, which is the primary CSRF vector. The attacker’s form submission cannot ride on the victim’s session. If the cookie were `SameSite=None`, the browser would attach it and the CSRF would succeed unless a token-based defense blocked it.

---

### Real-World Cases

- **Session cookies**: `Lax` is the practical default for most applications.
- **Banking and high-value transactions**: `Strict` is appropriate because even link-based navigation should not carry the session.
- **Third-party embedded widgets**: `None; Secure` is required when the cookie must be sent in cross-site iframes, but it must be paired with token-based CSRF protection.

---

## 5. Cookie Security Best Practices

### Definitions

**Core Definition.** **Cookie security best practices** are the layered configuration and lifecycle guidelines that prevent session-token theft, forgery, and leakage across browsers, frameworks, and deployment environments.

**Technical Definition.** Best practices combine **attribute hygiene** (`HttpOnly`, `Secure`, `SameSite`), **scope minimization** (`__Host-` prefix, narrow `Domain`/`Path`), **lifetime bounds** (`Max-Age`/`Expires`), **payload safety** (opaque identifiers, no sensitive data), **session lifecycle management** (regeneration on login, rotation), and **dependency hygiene** (patching cookie-parsing libraries). These controls are complementary: a missing attribute can turn an otherwise solid authentication system into one where a cross-site script reads the token or a forged request rides on it.

**Beginner-Friendly Explanation.** Best practices are **the checklist that keeps cookies from becoming the weakest link**. Put all three flags on every session cookie. Use the `__Host-` prefix whenever you do not need to share the cookie across subdomains. Set a sensible expiry. Do not store sensitive data directly in the cookie — store an opaque identifier and keep the data server-side. Regenerate the session when the user logs in. And keep your cookie-parsing library updated, because parsing bugs can undermine even a well-configured cookie.

### Purposes

- **To prevent cookie theft** through XSS (`HttpOnly`) and network interception (`Secure`).
- **To prevent CSRF** through `SameSite` and, where necessary, token-based defenses.
- **To minimize cookie scope** so that a compromise on one subdomain or path does not affect others.
- **To bound the damage window** through short lifetimes and session rotation.
- **To avoid introducing vulnerabilities through outdated parsing dependencies**.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Best Practice | Why It Matters |
|---|---|---|
| **`HttpOnly`** | On every session/auth cookie | Stops XSS from reading the token |
| **`Secure`** | Always in production | Stops plaintext interception |
| **`SameSite`** | `Lax` or `Strict`; `None` only with `Secure` | Mitigates CSRF |
| **Prefix** | `__Host-` for session cookies | Host-only, `Secure`, `Path=/` |
| **`Domain`** | Omit unless sharing is required | Prevents subdomain leakage |
| **`Path`** | Narrowest possible | Reduces exposure surface |
| **`Max-Age`/`Expires`** | Explicit, short lifetime | Bounds misuse window |
| **Payload** | Opaque ID; no sensitive data | A stolen cookie reveals nothing directly |
| **Session lifecycle** | Regenerate on login; rotate on privilege change | Prevents session fixation |
| **Dependencies** | Patch cookie-parsing libraries | Parsing bugs can bypass prefixes |

**Syntax rules for a hardened session cookie (Express).**

```javascript
res.cookie("__Host-session", token, {
  httpOnly: true,
  secure: true,
  sameSite: "lax",
  path: "/",
  maxAge: 3600_000,
});
```

- The `__Host-` prefix requires `path: "/"` and no `domain` option.
- `secure` must be `true`; in development, it may be conditionally disabled for `localhost`.

**Constraints and limitations.**

- **No single attribute is sufficient**; the combination is required for defense in depth.
- **`__Host-` prevents sharing across subdomains**; if subdomain sharing is required, `__Secure-` is the fallback, but it is weaker.
- **Browser defaults vary**; always set `SameSite` explicitly.
- **Parsing discrepancies can undermine prefixes**; keep server-side cookie parsers updated and validate assumptions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Hardened Express session configuration

```javascript
// server.js
const session = require("express-session");

app.use(session({
  name: "__Host-session",
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    path: "/",
    maxAge: 3600_000,
  },
}));
```

**Expected `Set-Cookie` (production):**
```http
Set-Cookie: __Host-session=s%3A...; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

**Why this output occurs.** The configuration applies all three essential flags, the `__Host-` prefix, a narrow path, and an explicit lifetime. In production, `secure` is enabled; in development, it is disabled only to support `http://localhost`.

---

#### Example 2: Django secure cookie configuration

```python
# settings.py
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = "Lax"
SESSION_COOKIE_NAME = "__Host-session"
SESSION_COOKIE_AGE = 3600
```

**Expected `Set-Cookie` Header:**
```http
Set-Cookie: __Host-session=...; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

**Why this output occurs.** Django applies the configured attributes to the session cookie. The `__Host-` prefix is honored because the cookie is `Secure`, has `Path=/`, and does not set a `Domain`. The explicit `SameSite` avoids browser-default variability.

---

#### Example 3: Full hardened cookie with `__Host-` prefix and short lifetime

```http
Set-Cookie: __Host-session=a1b2c3d4e5; Path=/; Secure; HttpOnly; SameSite=Strict; Max-Age=1800
```

**Expected Browser Behavior:**
```
Host-only scope (no subdomain access).
HTTPS-only transmission.
JavaScript cannot read it.
No cross-site attachment (including top-level links).
Expires after 30 minutes.
```

**Why this output occurs.** `__Host-` enforces host-only, `Secure`, and `Path=/`. `HttpOnly` blocks script access. `SameSite=Strict` blocks all cross-site attachment. `Max-Age=1800` bounds the credential’s lifetime to 30 minutes, reducing the window of misuse if it is stolen.

---

### Real-World Cases

- **OWASP WSTG**: recommends `Secure` and `HttpOnly` on session cookies, narrow `Domain`/`Path`, and the `__Host-` prefix for session identifiers.
- **Safeguard.sh 2026 guidance**: `HttpOnly; Secure; SameSite=Lax` plus `__Host-` prefix as the baseline for session cookies; never store sensitive data directly in cookies.
- **Acunetix**: describes the secure baseline as `HttpOnly; Secure; SameSite=Strict` and warns that browsers do not enforce these flags by default.
- **PortSwigger research**: demonstrates that `__Host-` and `__Secure-` can be undermined by parsing inconsistencies, reinforcing the need to patch cookie libraries and validate server-side parsing.

---

## References

- MDN — Secure cookie configuration - https://developer.mozilla.org/en-US/docs/Web/Security/Practical_implementation_guides/Cookies
- MDN — Set-Cookie header - https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie
- OWASP WSTG — Testing for Cookies Attributes - https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/02-Testing_for_Cookies_Attributes
- OWASP — SameSite - https://community.owasp.org/SameSite
- RFC 6265bis — Cookies: HTTP State Management Mechanism (draft) - https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis
- Safeguard.sh — Cookie Security Best Practices (2026): Flags, Prefixes, Pitfalls - https://safeguard.sh/resources/blog/cookie-security-best-practices
- Acunetix — Cookie Security Flags: How to Secure Cookies with HttpOnly, Secure, and SameSite - https://www.acunetix.com/blog/web-security-zone/cookie-security-flags/
- PortSwigger — Cookie Chaos: How to bypass __Host and __Secure cookie prefixes - https://portswigger.net/research/cookie-chaos-how-to-bypass-host-and-secure-cookie-prefixes
- Express.js — express-session cookie options - https://expressjs.com/en/resources/middleware/session.html
- MDN — SameSite cookies - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite