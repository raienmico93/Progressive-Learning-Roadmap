# Helmet — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Helmet** is an open-source **Express/Connect middleware** for Node.js that automatically sets **HTTP response security headers** to mitigate common browser-based attacks, including cross-site scripting (XSS), clickjacking, MIME sniffing, and protocol downgrade attacks. It is not a security framework or a silver bullet; it is a **low-effort, high-coverage baseline** that makes the secure header configuration the default rather than an afterthought.

**Technical Definition.** Helmet exports a middleware factory that, when mounted with `app.use(helmet())`, sets **13 HTTP response headers** by default. Each header can be individually configured, disabled, or replaced with custom logic. The middleware operates by mutating the `res` object's headers before the response is sent, intercepting `res.writeHead` and `res.end`. Helmet's header suite includes **Content-Security-Policy**, **Cross-Origin-Opener-Policy**, **Cross-Origin-Resource-Policy**, **Origin-Agent-Cluster**, **Referrer-Policy**, **Strict-Transport-Security**, **X-Content-Type-Options**, **X-DNS-Prefetch-Control**, **X-Download-Options**, **X-Frame-Options**, **X-Permitted-Cross-Domain-Policies**, and the removal of **X-Powered-By** and **X-XSS-Protection**.

**Beginner-Friendly Explanation.** Helmet is **a package you install to make your Express app safer with one line of code**. Instead of remembering a dozen obscure HTTP headers and their correct values, you call `app.use(helmet())` and it sets them for you. Think of it as **a safety checklist that runs automatically** on every response, telling the browser “don't sniff MIME types, don't allow framing from other sites, prefer HTTPS, only load scripts from approved sources.” It does not fix insecure code, but it closes a large class of browser-level attacks that would otherwise be open by default.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Zero-config baseline** | `app.use(helmet())` sets 13 headers with sensible defaults. |
| **Granular control** | Every header can be configured, disabled, or replaced individually. |
| **Express/Connect middleware** | Designed for Express, but usable with other Node.js frameworks and standalone HTTP servers. |
| **Not a silver bullet** | Helmet sets headers; it does not validate input, enforce authorization, or fix application logic. |
| **Development-aware** | Several headers (notably HSTS and `upgrade-insecure-requests`) interfere with `localhost` development and should be adjusted in dev. |
| **Ecosystem-backed** | Maintained by the Helmet.js organization; version 8.x is current (8.3.0 at time of writing). |

---

### Prerequisites

- **Node.js and npm**: to install and run Helmet.
- **Express.js basics**: `app.use()`, middleware ordering, and route definitions.
- **HTTP response headers**: understanding how headers influence browser behavior.
- **A tool to inspect responses**: browser DevTools (Network tab) or `curl -I`.

---

### Related Programming Areas

- **Security fundamentals**: Helmet is a direct application of secure defaults and defense in depth.
- **Content Security Policy (CSP)**: Helmet's most powerful and most configuration-sensitive header.
- **API security**: Helmet is primarily for browser-facing applications; APIs may need a subset of headers.
- **Node.js/Express security hardening**: Helmet is the standard first step in Express hardening guides.

---

### Core Concepts / Features

The following eight sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules and structure, annotated executable examples with expected outputs, and real-world cases.

---

## 1. Helmet

### Definitions

**Core Definition.** **Helmet** is a Node.js middleware that sets security-related HTTP response headers to harden Express applications against common web vulnerabilities.

**Technical Definition.** Helmet is a **middleware factory** that returns a function `(req, res, next)` which calls `res.setHeader()` for each configured security header. It is typically mounted with `app.use(helmet())` **before routes are defined**, ensuring every response — including errors — carries the headers. Helmet exports named middleware functions for each header (e.g., `helmet.contentSecurityPolicy()`, `helmet.hsts()`, `helmet.frameguard()`), allowing standalone use or selective composition.

**Beginner-Friendly Explanation.** Helmet is **the one package that covers the security headers you would otherwise forget**. You install it, add one line, and your app immediately sends safer responses to every browser. You can still customize each header if your app has special needs (for example, allowing scripts from a CDN), but the default configuration already protects against several common attacks.

### Purposes

- **To set secure HTTP response headers by default**, eliminating the need for manual header configuration.
- **To protect against browser-based attacks** such as clickjacking, MIME sniffing, and cross-site scripting.
- **To enforce HTTPS usage** through the Strict-Transport-Security header.
- **To reduce the attack surface** by removing information-leaking headers like `X-Powered-By`.
- **To provide a maintainable, auditable security baseline** that can be customized per application.

### Syntax Rules and Structure

**General syntax (quick start).**

```javascript
import helmet from "helmet";
const app = express();
app.use(helmet());
```

**Component breakdown.**

- `helmet()` — the middleware factory; calling it returns the middleware function.
- `app.use(helmet())` — mounts the middleware globally; must be called **before routes** and other middleware that may send responses.
- This sets **13 headers** by default.

**Configuration syntax (per-header).**

```javascript
app.use(helmet({
  contentSecurityPolicy: { directives: { ... } },
  hsts: { maxAge: 31536000, includeSubDomains: true },
  frameguard: { action: "deny" },
}));
```

**Disabling a header.**

```javascript
app.use(helmet({
  contentSecurityPolicy: false,
  xDownloadOptions: false,
}));
```

**Constraints and limitations.**

- **Helmet does not validate CSP correctness**; you must use a CSP checker like CSP Evaluator.
- **Helmet is not a substitute for secure coding**; it does not prevent SQL injection or broken access control.
- **Middleware order matters**: Helmet must run before routes to ensure headers are set on all responses.

### Annotated Complete Step by Step Code Examples

#### Example 1: Minimal Helmet setup

```javascript
// server.js
import express from "express";
import helmet from "helmet";

const app = express();

// Step 1: Mount Helmet before routes.
app.use(helmet());

// Step 2: Define a route.
app.get("/", (req, res) => {
  res.json({ message: "Hello, Helmet!" });
});

app.listen(3000, () => console.log("http://localhost:3000"));
```

**Setup.** `npm install express helmet`.

**Expected Response (inspect with `curl -I http://localhost:3000`):**
```
HTTP/1.1 200 OK
Content-Security-Policy: default-src 'self';base-uri 'self';font-src 'self' https: data:;...
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
... (13 headers total)
```

**Why this output occurs.** `helmet()` returns middleware that sets each security header on the response before the route handler runs. The browser receives all 13 headers and enforces the corresponding protections. No header configuration was required beyond calling `helmet()`.

---

#### Example 2: Selective configuration

```javascript
app.use(helmet({
  // Step 1: Allow scripts from a CDN while keeping the rest of the CSP defaults.
  contentSecurityPolicy: {
    directives: {
      "script-src": ["'self'", "cdn.example.com"],
    },
  },
  // Step 2: Tighten framing to DENY.
  frameguard: { action: "deny" },
}));
```

**Expected Output (conceptual):**
```
Content-Security-Policy includes script-src 'self' cdn.example.com (merged with defaults).
X-Frame-Options: DENY (replaces the default SAMEORIGIN).
Other headers retain their defaults.
```

**Why this output occurs.** Helmet **merges** custom directives with default directives unless `useDefaults: false` is set. The `frameguard` option replaces the default `SAMEORIGIN` action with `DENY`, which prevents framing entirely.

---

### Real-World Cases

- **Every Express application**: Helmet is the first security middleware added in virtually every Express security guide.
- **API gateways and reverse proxies**: Helmet-style headers are also set at the proxy layer for defense in depth.
- **Compliance baselines**: OWASP and CIS benchmarks recommend security headers that Helmet sets by default.

---

## 2. Security Headers

### Definitions

**Core Definition.** **Security headers** are HTTP response headers that instruct the browser to enable or enforce specific security policies — restricting resource loading, controlling framing, enforcing HTTPS, and limiting information leakage.

**Technical Definition.** Security headers operate at the **browser enforcement layer**. They do not replace server-side controls; they add a **defense-in-depth layer** that the browser honors. Helmet's default header suite addresses XSS (CSP), clickjacking (`X-Frame-Options`, `frame-ancestors`), MIME sniffing (`X-Content-Type-Options`), protocol downgrade (HSTS), referrer leakage (`Referrer-Policy`), and cross-origin isolation (`Cross-Origin-Opener-Policy`, `Cross-Origin-Resource-Policy`).

**Beginner-Friendly Explanation.** Security headers are **instructions you send to the browser** that say “protect me from these specific attacks.” For example, `X-Frame-Options: DENY` tells the browser “never let another site embed me in a frame,” which prevents clickjacking. `Strict-Transport-Security` tells the browser “always use HTTPS for this site.” Helmet sets these instructions for you.

### Purposes

- **To instruct the browser to enforce security policies** that the server cannot enforce alone.
- **To mitigate XSS** by restricting which scripts can execute (CSP).
- **To prevent clickjacking** by controlling framing (`X-Frame-Options`, `frame-ancestors`).
- **To block MIME confusion attacks** by disabling content sniffing.
- **To protect user privacy** by limiting referrer information and DNS prefetching.

### Syntax Rules and Structure

**Helmet default headers (abbreviated) .**

| Header | Default Value | Purpose |
|---|---|---|
| `Content-Security-Policy` | `default-src 'self'; script-src 'self'; ...` | Restricts resource loading; mitigates XSS. |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Forces HTTPS. |
| `X-Frame-Options` | `SAMEORIGIN` | Mitigates clickjacking (legacy). |
| `X-Content-Type-Options` | `nosniff` | Disables MIME sniffing. |
| `Referrer-Policy` | `no-referrer` | Limits referrer leakage. |
| `Cross-Origin-Opener-Policy` | `same-origin` | Process isolation. |
| `Cross-Origin-Resource-Policy` | `same-origin` | Blocks cross-origin resource loading. |
| `Origin-Agent-Cluster` | `?1` | Origin-based process isolation. |
| `X-DNS-Prefetch-Control` | `off` | Disables DNS prefetching. |
| `X-Download-Options` | `noopen` | IE-specific download safety. |
| `X-Permitted-Cross-Domain-Policies` | `none` | Adobe cross-domain policy control. |
| `X-Powered-By` | **Removed** | Hides server technology. |
| `X-XSS-Protection` | **Disabled (`0`)** | Prevents harmful legacy XSS filters. |

**Constraints and limitations.**

- **Headers are not enforcement on the server**; a client can ignore them. They protect well-behaved browsers.
- **Some headers are legacy** (`X-Frame-Options`, `X-XSS-Protection`, `X-Download-Options`) but still useful for older clients.
- **Header conflicts can break functionality** if configured incorrectly (e.g., a CSP that blocks required scripts).

### Annotated Complete Step by Step Code Examples

#### Example 1: Inspecting Helmet's default headers

```javascript
// Step 1: Mount Helmet with defaults.
app.use(helmet());

// Step 2: Add a route.
app.get("/headers", (req, res) => {
  res.json({ ok: true });
});
```

**Expected Output (conceptual, from `curl -I`):**
```
HTTP/1.1 200 OK
Content-Security-Policy: default-src 'self';base-uri 'self';font-src 'self' https: data:;...
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Origin-Agent-Cluster: ?1
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-DNS-Prefetch-Control: off
X-Download-Options: noopen
X-Frame-Options: SAMEORIGIN
X-Permitted-Cross-Domain-Policies: none
```

**Why this output occurs.** `helmet()` sets each header in the default suite. `X-Powered-By` is removed, and `X-XSS-Protection` is disabled (`0`), because modern browsers ignore it and it can introduce vulnerabilities.

---

### Real-World Cases

- **Browser security audits**: tools like SecurityHeaders.com and Mozilla Observatory grade sites based on these headers.
- **PCI DSS and OWASP compliance**: security headers are part of the baseline requirements.
- **Defense in depth**: headers complement server-side controls; they do not replace them.

---

## 3. CSP (Content-Security-Policy)

### Definitions

**Core Definition.** **Content-Security-Policy (CSP)** is a security header that **restricts which resources a page is allowed to load and execute**, mitigating cross-site scripting (XSS), data injection, and other code-execution attacks.

**Technical Definition.** CSP is a **declarative allow-list** expressed as a set of directives (`default-src`, `script-src`, `style-src`, `img-src`, etc.), each with a list of allowed sources. Sources can be keywords (`'self'`, `'none'`), hostnames, schemes, or cryptographic nonces/hashes. Helmet's default CSP is **restrictive** but **requires customization** for applications that load third-party scripts, use inline styles, or embed frames. Helmet performs **very little validation** on CSP; correctness must be verified with a CSP checker.

**Beginner-Friendly Explanation.** CSP is **a guest list for your web page**. It tells the browser “only run scripts from this list of sources, only load images from these sources, and refuse anything else.” If an attacker injects a malicious script, the browser refuses to run it because the script's origin is not on the list. CSP is the **most powerful header in Helmet's suite**, but it is also the one most likely to break your site if configured incorrectly.

### Purposes

- **To mitigate XSS** by blocking inline scripts and scripts from untrusted origins.
- **To control resource loading** — images, styles, fonts, frames, and connections.
- **To prevent data exfiltration** by restricting `connect-src` and `form-action`.
- **To enforce HTTPS** through the `upgrade-insecure-requests` directive.
- **To support nonce-based or hash-based script authorization** for legitimate inline scripts.

### Syntax Rules and Structure

**General syntax (Helmet CSP configuration).**

```javascript
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      "script-src": ["'self'", "example.com"],
      "style-src": ["'self'", "'unsafe-inline'"],
    },
  },
}));
```

**Component breakdown.**

- `contentSecurityPolicy` — the header-specific configuration object.
- `directives` — a nested object; each key is a **directive name** (camelCase or kebab-case).
- Each value is an **array** of source strings or functions.
- Sources can include `'self'`, `'none'`, hostnames (`example.com`), schemes (`https:`), and nonces (`'nonce-...'`).

**Helmet's default CSP .**

```
default-src 'self';
base-uri 'self';
font-src 'self' https: data:;
form-action 'self';
frame-ancestors 'self';
img-src 'self' data:;
object-src 'none';
script-src 'self';
script-src-attr 'none';
style-src 'self' https: 'unsafe-inline';
upgrade-insecure-requests
```

**Nonce-based CSP syntax.**

```javascript
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(32).toString("hex");
  next();
});

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
    },
  },
}));
```

**Constraints and limitations.**

- **Default CSP blocks inline scripts and third-party scripts**; most apps require customization.
- **`upgrade-insecure-requests` breaks `localhost` development**; disable it in development.
- **Helmet does not validate CSP**; use CSP Evaluator to check for weaknesses.
- **CSP is not a complete XSS defense**; server-side output encoding is still required.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic CSP customization

```javascript
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      // Step 1: Allow scripts from self and a CDN.
      "script-src": ["'self'", "cdn.example.com"],
      // Step 2: Disable the default style-src (allows nothing from style-src).
      "style-src": null,
    },
  },
}));
```

**Expected Output (conceptual):**
```
Content-Security-Policy: default-src 'self'; ...; script-src 'self' cdn.example.com; ...
The style-src directive is omitted, so no styles are allowed by default.
```

**Why this output occurs.** Helmet **merges** the custom `script-src` with the default directives. Setting `style-src: null` **disables** the default style directive entirely, which may break the page unless styles are provided another way.

---

#### Example 2: Nonce-based CSP for inline scripts

```javascript
import crypto from "crypto";

// Step 1: Generate a per-request nonce.
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(32).toString("hex");
  next();
});

// Step 2: Include the nonce in the CSP.
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
    },
  },
}));

// Step 3: Use the nonce in the HTML.
app.get("/", (req, res) => {
  res.send(`<script nonce="${res.locals.cspNonce}">console.log("allowed")</script>`);
});
```

**Expected Output (conceptual):**
```
Content-Security-Policy: script-src 'self' 'nonce-e33cc...';
The inline script runs because its nonce matches the CSP.
Inline scripts without the nonce are blocked.
```

**Why this output occurs.** The nonce is generated per request and added to both the CSP header and the `<script>` tag. The browser allows the script because its `nonce` attribute matches the CSP's `'nonce-...'` source. This is the **recommended approach for legitimate inline scripts** without resorting to `'unsafe-inline'`.

---

#### Example 3: Disabling `upgrade-insecure-requests` in development

```javascript
const isDevelopment = app.get("env") === "development";

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      "upgrade-insecure-requests": isDevelopment ? null : [],
    },
  },
}));
```

**Expected Output (development):**
```
Content-Security-Policy does not include upgrade-insecure-requests.
`http://localhost` is not forced to HTTPS.
```

**Why this output occurs.** Safari upgrades `http://localhost` to `https://localhost` when `upgrade-insecure-requests` is active, breaking local development. Disabling it in development and enabling it in production preserves both developer productivity and production security.

---

### Real-World Cases

- **GitHub**: uses a strict CSP with nonces and `strict-dynamic`.
- **Google**: uses CSP to restrict script sources on many properties.
- **SPAs and SSR apps**: CSP must be tuned for the framework's script and style injection patterns.

---

## 4. HSTS (Strict-Transport-Security)

### Definitions

**Core Definition.** **HTTP Strict Transport Security (HSTS)** is a response header that tells browsers to **always connect to the site over HTTPS**, even if the user types `http://` or follows an insecure link, for a specified period.

**Technical Definition.** HSTS is expressed as `Strict-Transport-Security: max-age=<seconds>; includeSubDomains; preload`. The `max-age` directive specifies how long (in seconds) the browser should remember to use HTTPS. `includeSubDomains` extends the policy to all subdomains. `preload` expresses intent to be included in browser preload lists, which hardcode the policy before the first visit. Helmet's default is `max-age=31536000; includeSubDomains` (365 days), with `preload` disabled by default.

**Beginner-Friendly Explanation.** HSTS is **a promise to the browser that your site always uses HTTPS**. Once the browser sees the header, it refuses to connect over plain HTTP for the specified duration. This prevents attackers from downgrading the connection to HTTP and stealing cookies or injecting content. The `preload` directive asks browsers to include your domain in a built-in list, so even the very first visit is protected.

### Purposes

- **To prevent protocol downgrade attacks** by forcing HTTPS connections.
- **To protect against SSL-stripping** attacks that intercept the initial HTTP request.
- **To secure cookies and authentication tokens** from being sent over plaintext connections.
- **To improve performance** by avoiding unnecessary HTTP-to-HTTPS redirects after the first visit.
- **To prepare for HSTS preload** by including `includeSubDomains` and, optionally, `preload`.

### Syntax Rules and Structure

**General syntax (Helmet HSTS configuration).**

```javascript
app.use(helmet({
  strictTransportSecurity: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: false,
  },
}));
```

**Component breakdown.**

- `maxAge` — seconds the browser remembers to use HTTPS; defaults to `31536000` (365 days).
- `includeSubDomains` — boolean; applies the policy to all subdomains; defaults to `true`.
- `preload` — boolean; adds the `preload` directive; defaults to `false`.

**Default header value.**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**Constraints and limitations.**

- **HSTS is a one-way commitment**; once a browser sees it, reverting to HTTP is difficult until `max-age` expires.
- **Disable HSTS in development** if you use `http://localhost`, or the browser will force HTTPS and break local development.
- **Preload requires a long max-age and includeSubDomains**; the minimum for preload submission is typically 1 year, and the OWASP recommendation is 2 years.
- **HSTS does not protect the very first request** unless the domain is in the browser's preload list.

### Annotated Complete Step by Step Code Examples

#### Example 1: Default HSTS (production-safe)

```javascript
app.use(helmet());
```

**Expected Header:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**Why this output occurs.** Helmet's default HSTS configuration sets a 365-day max-age and includes subdomains. This is a strong default for production but may interfere with `localhost` development.

---

#### Example 2: Custom HSTS with preload

```javascript
app.use(helmet({
  strictTransportSecurity: {
    maxAge: 63072000,        // 2 years
    includeSubDomains: true,
    preload: true,
  },
}));
```

**Expected Header:**
```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

**Why this output occurs.** The `maxAge` is set to 2 years (the OWASP recommendation), `includeSubDomains` remains enabled, and `preload: true` adds the `preload` directive. This configuration qualifies the domain for submission to the HSTS preload list.

---

#### Example 3: Disabling HSTS in development

```javascript
const isDevelopment = app.get("env") === "development";

app.use(helmet({
  strictTransportSecurity: isDevelopment ? false : undefined,
}));
```

**Expected Output (development):**
```
No Strict-Transport-Security header is sent.
`http://localhost` continues to work.
```

**Why this output occurs.** HSTS forces HTTPS and can break local development. Disabling it only in development preserves production security while allowing local HTTP testing.

---

### Real-World Cases

- **All production HTTPS sites**: HSTS is a baseline requirement for secure deployment.
- **HSTS preload list**: domains with `preload` are hardcoded into browsers, protecting even the first visit.
- **Payment and authentication flows**: HSTS prevents downgrade attacks that would expose credentials.

---

## 5. X-Frame-Options

### Definitions

**Core Definition.** **X-Frame-Options** is a legacy HTTP header that controls whether a page can be embedded in a `<frame>`, `<iframe>`, or `<object>`, mitigating **clickjacking** attacks.

**Technical Definition.** X-Frame-Options accepts three values: `DENY` (never allow framing), `SAMEORIGIN` (allow framing only by pages on the same origin), and the legacy `ALLOW-FROM uri` (allow framing from a specific URI; not supported by Helmet). Helmet's default is `SAMEORIGIN`. The header is **superseded by the CSP `frame-ancestors` directive** but remains useful for older browsers that do not support CSP.

**Beginner-Friendly Explanation.** X-Frame-Options is **a rule about who can put your website inside a frame on their website**. Clickjacking attacks trick users by overlaying an invisible frame of your site on a malicious page, so when the user clicks what they think is a button, they are actually clicking your site's button. `X-Frame-Options: DENY` says “no one can frame me.” `SAMEORIGIN` says “only pages on my own site can frame me.”

### Purposes

- **To prevent clickjacking** by blocking framing from other origins.
- **To provide defense in depth** alongside the CSP `frame-ancestors` directive.
- **To support older browsers** that do not implement CSP `frame-ancestors`.
- **To allow same-origin framing** when the application legitimately embeds its own pages.

### Syntax Rules and Structure

**General syntax (Helmet frameguard configuration).**

```javascript
app.use(helmet({
  frameguard: {
    action: "deny",   // or "sameorigin"
  },
}));
```

**Component breakdown.**

- `frameguard.action` — `"deny"` or `"sameorigin"`; defaults to `"sameorigin"`.
- `ALLOW-FROM` is **not supported** by Helmet.

**Default header value.**

```
X-Frame-Options: SAMEORIGIN
```

**Constraints and limitations.**

- **`ALLOW-FROM` is obsolete** and unsupported by modern browsers and Helmet.
- **CSP `frame-ancestors` is the modern replacement**; use both for maximum coverage.
- **`DENY` may break legitimate embedding** if your own application uses iframes for internal pages.

### Annotated Complete Step by Step Code Examples

#### Example 1: Default SAMEORIGIN

```javascript
app.use(helmet());
```

**Expected Header:**
```
X-Frame-Options: SAMEORIGIN
```

**Why this output occurs.** Helmet's default allows same-origin framing, which is safe for most applications. Cross-origin framing is blocked, mitigating clickjacking from external sites.

---

#### Example 2: DENY (no framing at all)

```javascript
app.use(helmet({
  frameguard: { action: "deny" },
}));
```

**Expected Header:**
```
X-Frame-Options: DENY
```

**Why this output occurs.** `DENY` prevents any framing, including same-origin. This is the strictest setting and is appropriate for applications that never need to be embedded.

---

### Real-World Cases

- **Banking and financial sites**: `DENY` is common to prevent any framing.
- **SaaS dashboards**: `SAMEORIGIN` allows internal embedding while blocking external clickjacking.
- **Legacy browser support**: X-Frame-Options remains necessary because some older browsers do not support CSP `frame-ancestors`.

---

## 6. X-Content-Type-Options

### Definitions

**Core Definition.** **X-Content-Type-Options** is a security header that, when set to `nosniff`, **disables MIME type sniffing** in browsers, forcing them to trust the declared `Content-Type` rather than guessing.

**Technical Definition.** MIME sniffing is a browser behavior that inspects a response body to determine its type, even if the `Content-Type` header says otherwise. This can allow an attacker to upload a malicious script disguised as an image or text file. `X-Content-Type-Options: nosniff` disables this behavior, ensuring that a response declared as `text/css` is treated as CSS, a response declared as `image/png` is treated as an image, and so on.

**Beginner-Friendly Explanation.** `X-Content-Type-Options: nosniff` tells the browser: **“trust what I say the file type is; do not guess.”** Without it, a browser might look at a file, decide it looks like JavaScript, and execute it even though the server said it was a harmless image. This header closes that loophole.

### Purposes

- **To prevent MIME confusion attacks** where a file's actual content is misinterpreted as executable code.
- **To enforce strict content-type handling** by browsers.
- **To protect against content-sniffing vulnerabilities** in uploaded files and API responses.
- **To align with OWASP and browser security recommendations**.

### Syntax Rules and Structure

**Helmet configuration.**

```javascript
app.use(helmet());   // nosniff is set by default
```

The header takes **no options**; it is either enabled (`nosniff`) or disabled.

**Default header value.**

```
X-Content-Type-Options: nosniff
```

**Constraints and limitations.**

- **`nosniff` is binary**: it is either on or off; there are no configurable values.
- **It may break applications** that rely on browsers sniffing types for legacy assets; these should be fixed to serve correct `Content-Type` headers.
- **It applies to script and style loading as well**, enforcing that `Content-Type` matches the actual type.

### Annotated Complete Step by Step Code Examples

#### Example 1: Default nosniff

```javascript
app.use(helmet());
```

**Expected Header:**
```
X-Content-Type-Options: nosniff
```

**Why this output occurs.** Helmet sets `nosniff` by default. The browser will not sniff content types for scripts, styles, or other resources, reducing the risk of MIME confusion attacks.

---

### Real-World Cases

- **File upload services**: `nosniff` prevents uploaded HTML files disguised as images from being executed.
- **API responses**: JSON responses are not misinterpreted as scripts.
- **CDN-served assets**: correct `Content-Type` is enforced, preventing type confusion.

---

## 7. Referrer-Policy

### Definitions

**Core Definition.** **Referrer-Policy** controls how much **referrer information** (the `Referer` header) is sent when a user navigates from one page to another or loads a resource.

**Technical Definition.** The `Referer` header tells the destination site where the user came from. This can leak sensitive information such as search queries, internal URLs, or session tokens. Referrer-Policy defines what is included in the `Referer` header: `no-referrer` (send nothing), `origin` (send only the origin), `strict-origin-when-cross-origin` (send the full URL for same-origin, only the origin for cross-origin), and others. Helmet's default is `no-referrer`, the **most privacy-preserving** option.

**Beginner-Friendly Explanation.** Referrer-Policy is **a privacy setting for the “where did you come from” information** that browsers send when you click a link. If you visit your bank's internal page and then click an external link, the external site normally learns the internal URL. `no-referrer` stops that leak entirely. `strict-origin-when-cross-origin` is a middle ground that shares the domain but not the full path with external sites.

### Purposes

- **To protect user privacy** by preventing sensitive URLs from leaking to third parties.
- **To reduce the risk of token leakage** when URLs contain sensitive parameters.
- **To balance privacy and analytics** by choosing a policy that shares only what is necessary.
- **To comply with privacy regulations** that restrict cross-site data sharing.

### Syntax Rules and Structure

**General syntax (Helmet referrerPolicy configuration).**

```javascript
app.use(helmet({
  referrerPolicy: {
    policy: "no-referrer",   // or "strict-origin-when-cross-origin", etc.
  },
}));
```

**Component breakdown.**

- `policy` — a string or array of strings; defaults to `"no-referrer"`.
- Arrays are joined with commas for fallback policies.

**Common policy values.**

| Value | Behavior |
|---|---|
| `no-referrer` | Send no referrer information. |
| `origin` | Send only the origin (scheme + host + port). |
| `strict-origin-when-cross-origin` | Full URL for same-origin; origin only for cross-origin; nothing for HTTP downgrades. |
| `same-origin` | Send full URL for same-origin; nothing cross-origin. |

**Default header value.**

```
Referrer-Policy: no-referrer
```

**Constraints and limitations.**

- **`no-referrer` may break analytics** that rely on referrer data; choose a policy that balances privacy and business needs.
- **`strict-origin-when-cross-origin` is the modern browser default** in many cases; explicitly setting it ensures consistency.

### Annotated Complete Step by Step Code Examples

#### Example 1: Default no-referrer

```javascript
app.use(helmet());
```

**Expected Header:**
```
Referrer-Policy: no-referrer
```

**Why this output occurs.** Helmet defaults to the most privacy-preserving policy. No referrer information is sent, preventing URL leakage.

---

#### Example 2: Strict-origin-when-cross-origin

```javascript
app.use(helmet({
  referrerPolicy: {
    policy: "strict-origin-when-cross-origin",
  },
}));
```

**Expected Header:**
```
Referrer-Policy: strict-origin-when-cross-origin
```

**Why this output occurs.** This policy sends the full URL for same-origin requests (useful for internal analytics) and only the origin for cross-origin requests (protecting path privacy). It is a balanced default for applications that need referrer data without leaking sensitive paths.

---

### Real-World Cases

- **Privacy-sensitive sites**: `no-referrer` for healthcare, finance, and government sites.
- **Analytics-dependent sites**: `strict-origin-when-cross-origin` to preserve referral data while protecting paths.
- **Modern browser default**: many browsers default to `strict-origin-when-cross-origin` when no policy is set.

---

## 8. Permissions-Policy

### Definitions

**Core Definition.** **Permissions-Policy** (formerly **Feature-Policy**) is a security header that **enables or disables browser features and APIs** (camera, microphone, geolocation, etc.) for a page and its embedded frames.

**Technical Definition.** Permissions-Policy is expressed as a comma-separated list of directives, each specifying a feature and an allow-list of origins: `geolocation=()`, `camera=(self)`, `microphone=(*)`. Empty parentheses `()` disable the feature entirely; `(self)` allows the current origin; `(*)` allows all origins. Helmet does **not set Permissions-Policy by default** in its main suite; it must be configured separately or via a dedicated middleware.

**Beginner-Friendly Explanation.** Permissions-Policy is **a way to turn off browser features you do not need**. If your app does not use the camera or microphone, you can tell the browser “block these features entirely.” This reduces the attack surface: even if an attacker injects a script, it cannot access the camera if the policy forbids it. It also gives users confidence that unnecessary permissions are not being requested.

### Purposes

- **To disable unnecessary browser features** (camera, microphone, geolocation, etc.).
- **To reduce the attack surface** for script-based attacks that might abuse APIs.
- **To protect user privacy** by preventing silent access to sensitive sensors.
- **To control feature availability in embedded frames** by specifying which origins may use which features.

### Syntax Rules and Structure

**General syntax (manual header or dedicated middleware).**

```
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

**Component breakdown.**

- `geolocation=()` — disable geolocation entirely.
- `camera=(self)` — allow the camera only for the current origin.
- `microphone=(*)` — allow the microphone for all origins (not recommended).

**Helmet integration.** Helmet does not set this header by default. It can be set manually via `res.setHeader` or through a dedicated middleware such as `permissions-policy`.

**Constraints and limitations.**

- **Helmet does not include Permissions-Policy by default**; it must be added separately.
- **Feature names and browser support vary**; not all directives are supported in all browsers.
- **Overly restrictive policies may break legitimate features**; audit which features your app actually uses.

### Annotated Complete Step by Step Code Examples

#### Example 1: Setting Permissions-Policy manually

```javascript
// Step 1: Set the header in a middleware.
app.use((req, res, next) => {
  res.setHeader(
    "Permissions-Policy",
    "geolocation=(), microphone=(), camera=()"
  );
  next();
});
```

**Expected Header:**
```
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

**Why this output occurs.** The middleware sets the header on every response. The browser disables geolocation, microphone, and camera for the page. Scripts cannot access these APIs, reducing the attack surface.

---

#### Example 2: Allowing a feature for same-origin only

```javascript
app.use((req, res, next) => {
  res.setHeader(
    "Permissions-Policy",
    "camera=(self), geolocation=(self), microphone=()"
  );
  next();
});
```

**Expected Header:**
```
Permissions-Policy: camera=(self), geolocation=(self), microphone=()
```

**Why this output occurs.** The camera and geolocation are allowed only for the current origin (`self`), while the microphone is disabled entirely. This is a common pattern for apps that need some features but not others.

---

### Real-World Cases

- **Privacy-focused apps**: disable all sensors that are not needed.
- **Video conferencing apps**: allow camera and microphone for `self` only.
- **Embedded widgets**: restrict features for third-party frames while allowing the main page.

---

## 9. Helmet Best Practices

### Definitions

**Core Definition.** **Helmet best practices** are empirically established guidelines for configuring Helmet so that it provides **maximum security without breaking application functionality**, particularly in development and with content-heavy applications.

**Technical Definition.** Best practices center on **middleware ordering** (Helmet before routes), **environment-aware configuration** (disable HSTS and `upgrade-insecure-requests` in development), **CSP customization** (use nonces or hashes for inline scripts, avoid `'unsafe-inline'`), **CSP validation** (use CSP Evaluator), **header tuning** (set `X-Frame-Options: DENY` if framing is unnecessary), and **dependency management** (keep Helmet updated).

**Beginner-Friendly Explanation.** Best practices mean **using Helmet correctly, not just installing it**. Put it before your routes so every response gets the headers. Turn off HSTS and `upgrade-insecure-requests` during local development so `localhost` keeps working. Use nonces instead of `'unsafe-inline'` for inline scripts. Check your CSP with a validator. Use `DENY` for framing if you never embed your own pages. And keep Helmet updated.

### Purposes

- **To ensure every response carries security headers** by mounting Helmet first.
- **To avoid breaking local development** by environment-aware header configuration.
- **To maintain a strong CSP** by using nonces/hashes instead of unsafe directives.
- **To validate CSP correctness** with external tools.
- **To keep the security baseline current** by updating Helmet regularly.

### Syntax Rules and Structure

**Best-practice checklist.**

| Area | Best Practice |
|---|---|
| **Ordering** | `app.use(helmet())` before all routes and response-sending middleware. |
| **Development** | Disable HSTS and `upgrade-insecure-requests` in development. |
| **CSP** | Use nonces or hashes; avoid `'unsafe-inline'` and `'unsafe-eval'`. |
| **CSP validation** | Use CSP Evaluator; Helmet does minimal validation. |
| **Framing** | Use `DENY` if framing is not needed; `SAMEORIGIN` otherwise. |
| **HSTS** | Use at least 1-year max-age; add `preload` only after confirming subdomain readiness. |
| **Referrer** | Choose a policy that balances privacy and analytics needs. |
| **Permissions** | Disable unused features; allow only what the app needs. |
| **Updates** | Keep Helmet updated to receive new headers and fixes. |

**Constraints and limitations.**

- **Best practices are context-dependent**; a strict CSP for a static site may be inappropriate for a complex SPA.
- **Helmet is not a complete security solution**; it complements input validation, authentication, and authorization.

### Annotated Complete Step by Step Code Examples

#### Example 1: Environment-aware Helmet configuration

```javascript
import helmet from "helmet";

const isDevelopment = app.get("env") === "development";

app.use(helmet({
  // Step 1: Disable HSTS in development.
  strictTransportSecurity: isDevelopment ? false : {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  // Step 2: Disable upgrade-insecure-requests in development.
  contentSecurityPolicy: {
    directives: {
      "upgrade-insecure-requests": isDevelopment ? null : [],
    },
  },
  // Step 3: Use DENY for framing unless same-origin embedding is needed.
  frameguard: { action: "deny" },
}));
```

**Expected Output (development):**
```
No Strict-Transport-Security header.
No upgrade-insecure-requests in CSP.
X-Frame-Options: DENY.
```

**Expected Output (production):**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy includes upgrade-insecure-requests.
X-Frame-Options: DENY.
```

**Why this output occurs.** The `isDevelopment` flag toggles HSTS and `upgrade-insecure-requests` off in development and on in production. The `frameguard` action is set to `DENY` because the application does not need to be framed. This configuration provides strong production security without breaking local development.

---

#### Example 2: CSP with nonce and validation guidance

```javascript
import crypto from "crypto";

// Step 1: Generate a per-request nonce.
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(32).toString("hex");
  next();
});

// Step 2: Configure CSP with the nonce.
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
}));

// Step 3: Validate the CSP with CSP Evaluator externally.
```

**Expected Output (conceptual):**
```
Content-Security-Policy: script-src 'self' 'nonce-...'; object-src 'none'; upgrade-insecure-requests;
Inline scripts with the correct nonce are allowed; others are blocked.
```

**Why this output occurs.** The nonce authorizes legitimate inline scripts without `'unsafe-inline'`. `object-src 'none'` blocks plugin content. The CSP should still be validated with CSP Evaluator to catch weaknesses that Helmet does not detect.

---

### Real-World Cases

- **Production Express apps**: use environment-aware configuration to keep development fast and production secure.
- **CSP-strict applications**: use nonces and validate with CSP Evaluator before deployment.
- **API services**: may disable CSP and framing headers if the service returns only JSON and is never rendered in a browser.

---

## References

- Helmet.js — Official Documentation - https://helmet.js.org/
- helmet — npm Package (v8.x README) - https://www.npmjs.com/package/helmet
- Helmet — GitHub Repository (README) - https://github.com/helmetjs/helmet
- helmet-csp — npm Package (CSP Middleware) - https://www.npmjs.com/package/helmet-csp
- MDN — Content-Security-Policy - https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- MDN — Strict-Transport-Security - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
- MDN — X-Frame-Options - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
- MDN — X-Content-Type-Options - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
- MDN — Referrer-Policy - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
- MDN — Permissions-Policy - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy
- OWASP — HTTP Security Response Headers Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html
- OWASP — Content Security Policy Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html
- OWASP — HSTS Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html
- CSP Evaluator (Google) - https://csp-evaluator.withgoogle.com/