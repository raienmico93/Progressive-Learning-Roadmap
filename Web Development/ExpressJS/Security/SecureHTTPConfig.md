# Express.js Secure HTTP Configuration — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Secure HTTP configuration is the set of server-side settings, protocol choices, and response headers that protect data in transit, control how browsers handle cookies, and constrain the capabilities of client-side code.

**Technical Definition:** Secure HTTP configuration in Express.js encompasses TLS termination and protocol enforcement (TLS 1.2 minimum, TLS 1.3 preferred), HTTP Strict Transport Security (HSTS) to prevent protocol downgrade attacks, cookie attribute configuration (Secure, HttpOnly, SameSite), and the manual or middleware-driven setting of security response headers such as Permissions-Policy, Referrer-Policy, and the Reporting API headers (Reporting-Endpoints, Report-To).

**Beginner-Friendly Explanation:** Think of your web application as a house. Secure HTTP configuration is the set of locks, reinforced doors, and security cameras you install. HTTPS/TLS encrypts the conversation between the browser and your server so no one can eavesdrop. HSTS tells browsers "always use the secure door, never the old one." Cookie flags control who can read and send your session keys. Security headers tell the browser which features the page is allowed to use. Together, they form the transport and browser-level foundation of your application's security.

### Key Characteristics

- **Transport-layer foundation:** HTTPS/TLS is the prerequisite for every other secure HTTP feature; without it, Secure cookies and HSTS are meaningless.
- **Header-driven:** Most browser-side protections are implemented through HTTP response headers that the browser enforces.
- **Defence-in-depth:** No single configuration is sufficient; TLS, HSTS, cookie flags, and security headers must be layered.
- **Version-sensitive:** TLS 1.0 and 1.1 are deprecated; TLS 1.2 is the minimum, and TLS 1.3 is preferred. Cookie SameSite behaviour has evolved across browser versions.
- **Middleware-assisted:** Helmet provides sensible defaults for most security headers, but complex headers like Permissions-Policy, Referrer-Policy, and Reporting-Endpoints often require manual configuration.

### Prerequisites

- **Node.js runtime** (v18 or higher recommended for current TLS defaults).
- **Express.js installed** (`npm install express`).
- **SSL/TLS certificate** (from a trusted CA or self-signed for development).
- **Basic understanding of HTTP:** request/response headers, status codes, cookies.
- **Familiarity with Express middleware:** how `app.use()` and `res.setHeader()` work.

### Related Programming Areas

- **Cryptography and PKI:** Certificate management, cipher suites, key exchange.
- **Reverse proxy configuration:** Nginx, Cloudflare, AWS ALB terminating TLS.
- **Browser security model:** Same-Origin Policy, cookie scope, feature policies.
- **Web application firewalls (WAF):** Header inspection and protocol enforcement.
- **Observability:** Reporting API for CSP violations and deprecation reports.

### Core Concepts

1. **HTTPS & TLS Configuration** — enforcing TLS 1.3, managing certificates, and HSTS.
2. **Secure Cookies** — the `Secure` flag and protocol stripping prevention.
3. **HTTP-only Cookies** — the `HttpOnly` flag and session token protection.
4. **SameSite Cookies** — `Strict`, `Lax`, and `None` configurations.
5. **Security Headers** — Permissions-Policy, Referrer-Policy, and Reporting API.

---

## Core Concept 1: HTTPS & TLS Configuration

### Definitions

**Core Definition:** HTTPS is HTTP transmitted over TLS (Transport Layer Security), which encrypts and authenticates the connection between the client and server. TLS configuration involves selecting protocol versions, cipher suites, and certificate management.

**Technical Definition:** In Node.js, HTTPS is implemented via the `https` module, which extends `tls.createServer()`. The `https.ServerOptions` object accepts `key`, `cert`, and `ca` for certificate configuration, `minVersion` and `maxVersion` for protocol version control, `ciphers` for cipher suite selection, and `honorCipherOrder` to prefer the server's cipher preference. TLS 1.3 cipher suites are automatically negotiated when both peers support the protocol and are not configured via the `ciphers` option in the same way as TLS 1.2 suites.

**Beginner-Friendly Explanation:** HTTPS is like sending a letter in a sealed, tamper-proof envelope instead of a postcard. TLS is the sealing mechanism. TLS 1.3 is the newest and strongest seal. Configuring TLS means choosing which seals to accept (protocol versions), which materials are allowed (cipher suites), and proving your identity with a certificate.

### Purposes

- To encrypt data in transit, preventing eavesdropping and man-in-the-middle attacks.
- To authenticate the server's identity via X.509 certificates issued by trusted CAs.
- To enforce modern protocol versions (TLS 1.2 minimum, TLS 1.3 preferred) and reject deprecated versions.
- To prevent protocol downgrade attacks via HTTP Strict Transport Security (HSTS).

### Sub-Feature 1.1: TLS Protocol and Cipher Configuration

#### Definitions

**Core Definition:** TLS protocol configuration selects which versions of the TLS protocol and which cipher suites the server will accept.

**Technical Definition:** In Node.js, `minVersion` and `maxVersion` control the acceptable TLS protocol versions. The `ciphers` option specifies a colon-separated string of cipher suite names. For TLS 1.3, only five cipher suites are defined (TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256, TLS_AES_128_GCM_SHA256, TLS_AES_128_CCM_SHA256, TLS_AES_128_CCM_8_SHA256), and Node.js enables the recommended ones by default. TLS 1.2 cipher suites must be explicitly configured for defence-in-depth.

**Beginner-Friendly Explanation:** Protocol versions are like different generations of a lock design. Older locks (TLS 1.0, 1.1) have known weaknesses, so you disable them. Cipher suites are the specific mechanisms inside the lock. You choose the strongest ones and tell the server to prefer them.

#### Purposes

- To enforce the minimum acceptable security level for encrypted connections.
- To reject connections from clients that only support deprecated protocols.
- To select strong cipher suites and disable weak ones (RC4, DES, 3DES).
- To optimise performance by preferring modern, efficient cipher suites.

#### Syntax Rules and Structure

```js
const https = require('node:https');
const fs = require('node:fs');

const options = {
  key: fs.readFileSync('/path/to/private.key'),
  cert: fs.readFileSync('/path/to/certificate.crt'),
  minVersion: 'TLSv1.2',
  maxVersion: 'TLSv1.3',
  ciphers: [
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384',
    'ECDHE-ECDSA-CHACHA20-POLY1305',
    'ECDHE-RSA-CHACHA20-POLY1305',
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256'
  ].join(':'),
  honorCipherOrder: true,
  ecdhCurve: 'P-384:P-256',
  sessionTimeout: 300
};
```

| Component | Breakdown |
|-----------|-----------|
| `key` | Private key file contents. |
| `cert` | Server certificate (full chain). |
| `minVersion` / `maxVersion` | Acceptable TLS versions. |
| `ciphers` | Colon-separated cipher suite list (TLS 1.2). |
| `honorCipherOrder` | Server's preference takes priority over client's. |
| `ecdhCurve` | Elliptic curve for ECDHE key exchange. |

**Constraints and Limitations:**
- TLS 1.3 cipher suites are negotiated automatically and cannot be disabled via `ciphers`; use `minVersion: 'TLSv1.3'` to restrict to TLS 1.3 only.
- Disabling TLS 1.2 entirely may break older clients; assess your user base before enforcing TLS 1.3 exclusively.
- `tls.getCiphers()` returns cipher names in lowercase, but they must be uppercased when passed to the `ciphers` option.

#### Annotated Code Example

```js
// tls-server.js — Hardened HTTPS server
const https = require('node:https');
const fs = require('node:fs');
const express = require('express');

const app = express();

const httpsOptions = {
  key: fs.readFileSync('/etc/ssl/private/server.key'),
  cert: fs.readFileSync('/etc/ssl/certs/server.crt'),
  minVersion: 'TLSv1.2',
  maxVersion: 'TLSv1.3',
  ciphers: [
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384',
    'ECDHE-ECDSA-CHACHA20-POLY1305',
    'ECDHE-RSA-CHACHA20-POLY1305'
  ].join(':'),
  honorCipherOrder: true,
  ecdhCurve: 'P-384:P-256'
};

const server = https.createServer(httpsOptions, app);

app.get('/', (req, res) => {
  res.json({ secure: true, protocol: req.protocol });
});

server.listen(443, () => {
  console.log('HTTPS server on port 443 (TLS 1.2–1.3)');
});

// HTTP redirect to HTTPS
const http = require('node:http');
http.createServer((req, res) => {
  res.writeHead(301, { Location: `https://${req.headers.host}${req.url}` });
  res.end();
}).listen(80);
```

**Expected Output (for `GET /` over TLS 1.3):**
```
{"secure":true,"protocol":"https"}
```

**Why this output:** The server negotiates TLS 1.3 with the client, encrypts the response, and `req.protocol` returns `"https"`. HTTP requests on port 80 receive a 301 redirect to the HTTPS URL.

#### Real-World Cases

- **Public-facing web applications:** All production traffic should use TLS 1.2+ with strong cipher suites.
- **API endpoints:** Mutual TLS (mTLS) for service-to-service authentication.
- **Compliance (PCI DSS, HIPAA):** TLS 1.2 minimum is mandated; TLS 1.3 is recommended.

---

### Sub-Feature 1.2: HSTS (HTTP Strict Transport Security)

#### Definitions

**Core Definition:** HSTS is a response header that instructs browsers to only connect to the domain over HTTPS for a specified duration, preventing protocol downgrade and cookie-stripping attacks.

**Technical Definition:** The `Strict-Transport-Security` header uses three directives: `max-age` (seconds the policy is cached), `includeSubDomains` (applies to all subdomains), and `preload` (signals eligibility for the browser-shipped HSTS preload list). The browser only honours the header when it is received over HTTPS; sent over plain HTTP, it is silently ignored. Helmet's `hsts` middleware defaults to `max-age=31536000`, `includeSubDomains: true`, and `preload: false`.

**Beginner-Friendly Explanation:** HSTS is like a rule you give your browser: "For the next year, never use the unencrypted version of this website, even if someone sends you a link that says to." This prevents attackers on the same network from tricking the browser into using an insecure connection.

#### Purposes

- To prevent SSL stripping attacks where an attacker downgrades HTTPS to HTTP.
- To ensure that all future connections to the domain use HTTPS without an initial redirect.
- To extend HTTPS enforcement to all subdomains via `includeSubDomains`.
- To close the first-connection gap via the preload list.

#### Syntax Rules and Structure

```js
const helmet = require('helmet');
app.use(helmet.hsts({
  maxAge: 31536000,          // 1 year (seconds)
  includeSubDomains: true,
  preload: true
}));
```

| Directive | Breakdown |
|-----------|-----------|
| `max-age` | Seconds the browser caches the HTTPS-only policy. |
| `includeSubDomains` | Extends policy to all subdomains. |
| `preload` | Signals eligibility for browser preload lists. |

**Constraints and Limitations:**
- HSTS is ignored when received over plain HTTP; the first connection to a new domain is unprotected.
- `includeSubDomains` applies to every subdomain, including staging and legacy services; audit DNS before enabling.
- `preload` is non-standard and inert without submission to hstspreload.org.
- A short `max-age` (e.g., 86400) is advisable for initial rollout; a misconfiguration self-heals within a day.

#### Annotated Code Example

```js
// hsts.js — HSTS with Helmet
const express = require('express');
const helmet = require('helmet');
const app = express();

app.use(helmet.hsts({
  maxAge: 63072000,          // 2 years
  includeSubDomains: true,
  preload: true
}));

app.get('/', (req, res) => {
  res.send('HSTS enforced');
});

app.listen(443, () => console.log('HSTS server on 443'));
```

**Expected Response Header:**
```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

**Why this output:** The header tells the browser to cache the HTTPS-only policy for two years, apply it to all subdomains, and consider the domain for preload list inclusion.

#### Real-World Cases

- **Banking and financial services:** HSTS is mandatory for PCI DSS compliance.
- **Government websites:** Many national security standards require HSTS with preload.
- **Any production HTTPS site:** HSTS closes the downgrade window for repeat visitors.

---

## Core Concept 2: Secure Cookies

### Definitions

**Core Definition:** The `Secure` cookie attribute ensures that a cookie is only transmitted over HTTPS connections, preventing it from being sent over unencrypted HTTP where it could be intercepted.

**Technical Definition:** When the `Secure` attribute is set on a cookie, the browser only includes it in requests made over an encrypted connection (HTTPS). This prevents network attackers from reading the cookie via packet sniffing on plaintext connections. In Express, the `Secure` flag is set via the `secure: true` option in `res.cookie()` or in the `cookie` object of `express-session` configuration.

**Beginner-Friendly Explanation:** A cookie without the Secure flag is like writing your password on a postcard. Anyone who handles the postcard (network intermediary) can read it. The Secure flag is like requiring the message to be sent in a sealed envelope that only the intended recipient can open.

### Purposes

- To prevent session tokens and authentication cookies from being transmitted over unencrypted connections.
- To mitigate protocol stripping attacks where an attacker forces HTTP to intercept cookies.
- To comply with security best practices and standards (OWASP, PCI DSS).

### Sub-Feature 2.1: Setting the Secure Flag

#### Definitions

**Core Definition:** The `secure` option in Express cookie configuration sets the `Secure` attribute on the `Set-Cookie` header.

**Technical Definition:** In `express-session`, the default is `secure: false`. In production, this must be set to `true`, which requires an HTTPS-enabled server. When behind a reverse proxy that terminates TLS, `app.set('trust proxy', 1)` must be set so that `express-session` can detect the secure connection via the `X-Forwarded-Proto` header.

**Beginner-Friendly Explanation:** Setting `secure: true` is like telling the browser "only send this cookie when you're using the secure connection." If you're behind a proxy that handles the encryption, you need to tell Express to trust the proxy's signal.

#### Purposes

- To ensure session cookies are never exposed over HTTP.
- To prevent cookie theft via man-in-the-middle attacks on unencrypted networks.
- To satisfy the requirement that `SameSite=None` cookies must also be `Secure`.

#### Syntax Rules and Structure

```js
// express-session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {
    secure: true,            // HTTPS only
    httpOnly: true,
    sameSite: 'lax',
    maxAge: 3600000
  }
}));
```

| Component | Breakdown |
|-----------|-----------|
| `secure: true` | Cookie only sent over HTTPS. |
| `trust proxy` | Required behind TLS-terminating reverse proxy. |

**Constraints and Limitations:**
- `secure: true` requires HTTPS; setting it without HTTPS will prevent cookies from being sent.
- Behind a reverse proxy, `app.set('trust proxy', 1)` is mandatory for `secure` detection.
- The `Secure` flag does not encrypt the cookie content; it only controls transmission.

#### Annotated Code Example

```js
// secure-cookie.js — Secure cookie with trust proxy
const express = require('express');
const session = require('express-session');
const app = express();

// Trust the first proxy (Nginx, Cloudflare, Heroku router)
app.set('trust proxy', 1);

app.use(session({
  secret: process.env.SESSION_SECRET || 'dev-secret',
  resave: false,
  saveUninitialized: false,
  name: 'app.sid',
  cookie: {
    secure: true,            // HTTPS only
    httpOnly: true,
    sameSite: 'lax',
    maxAge: 3600000
  }
}));

app.get('/', (req, res) => {
  req.session.views = (req.session.views || 0) + 1;
  res.json({ views: req.session.views });
});

app.listen(3000, () => console.log('Secure cookie server on 3000'));
```

**Expected Response Header (over HTTPS):**
```
Set-Cookie: app.sid=s%3A...; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

**Why this output:** The `Secure` attribute ensures the cookie is only sent over HTTPS. `HttpOnly` prevents JavaScript access, and `SameSite=Lax` mitigates CSRF. The `trust proxy` setting allows `express-session` to detect the secure connection behind the proxy.

#### Real-World Cases

- **Banking applications:** Session cookies must be `Secure` to prevent theft on public Wi-Fi.
- **E-commerce:** Protecting customer sessions during checkout.
- **Any production application:** `Secure` is the baseline for session cookies.

---

## Core Concept 3: HTTP-only Cookies

### Definitions

**Core Definition:** The `HttpOnly` cookie attribute prevents client-side JavaScript from accessing the cookie via `document.cookie`, protecting session tokens from theft through cross-site scripting (XSS).

**Technical Definition:** When the `HttpOnly` attribute is set, the browser makes the cookie inaccessible to JavaScript APIs. This means that even if an attacker successfully injects a script into the page, the script cannot read the session cookie. In `express-session`, `httpOnly` defaults to `true`.

**Beginner-Friendly Explanation:** Think of an `HttpOnly` cookie as a key that's locked in a safe that only the browser itself can open. JavaScript (which an attacker might control through an XSS vulnerability) can't reach into the safe. The browser sends the key automatically with requests, but scripts can't read it.

### Purposes

- To prevent XSS attacks from stealing session tokens via `document.cookie`.
- To protect authentication cookies from client-side script access.
- To comply with OWASP session management recommendations.

### Sub-Feature 3.1: Setting the HttpOnly Flag

#### Definitions

**Core Definition:** The `httpOnly` option in Express cookie configuration sets the `HttpOnly` attribute on the `Set-Cookie` header.

**Technical Definition:** `express-session` defaults `cookie.httpOnly` to `true`. For manually set cookies via `res.cookie()`, `httpOnly: true` must be explicitly passed. The flag is a boolean; when truthy, the `HttpOnly` attribute is included in the `Set-Cookie` header.

**Beginner-Friendly Explanation:** Setting `httpOnly: true` is like telling the browser "keep this cookie in a place where scripts can't see it." Only the browser's network layer can access it.

#### Purposes

- To render session tokens invisible to JavaScript, neutralising XSS-based token theft.
- To provide defence-in-depth even if other XSS mitigations fail.
- To meet the requirement that session cookies should not be accessible to client-side code.

#### Syntax Rules and Structure

```js
res.cookie('session', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax'
});
```

| Component | Breakdown |
|-----------|-----------|
| `httpOnly: true` | Cookie not accessible via `document.cookie`. |
| `secure: true` | Cookie only sent over HTTPS. |
| `sameSite: 'lax'` | Cross-site request control. |

**Constraints and Limitations:**
- `HttpOnly` does not prevent XSS; it only limits the impact by preventing cookie theft. The XSS can still perform actions on behalf of the user.
- Some client-side libraries require reading cookies; `HttpOnly` will break them.
- `HttpOnly` must be set on the server; it cannot be added by client-side JavaScript.

#### Annotated Code Example

```js
// httponly-cookie.js
const express = require('express');
const crypto = require('node:crypto');
const app = express();

app.post('/login', (req, res) => {
  const sessionToken = crypto.randomBytes(32).toString('hex');

  res.cookie('session', sessionToken, {
    httpOnly: true,          // Not accessible via document.cookie
    secure: true,            // HTTPS only
    sameSite: 'lax',
    maxAge: 3600000
  });

  res.json({ message: 'Logged in' });
});

app.listen(3000, () => console.log('HttpOnly server on 3000'));
```

**Expected Response Header:**
```
Set-Cookie: session=abc123...; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

**Why this output:** The `HttpOnly` attribute prevents JavaScript from reading the `session` cookie. An XSS payload running `document.cookie` would return an empty string or other cookies, but not the session token.

#### Real-World Cases

- **All session cookies:** `HttpOnly` should be set on every session cookie without exception.
- **CSRF tokens:** If CSRF tokens are stored in cookies for the double-submit pattern, they must **not** be `HttpOnly` because JavaScript needs to read them.
- **Analytics cookies:** Non-sensitive cookies may omit `HttpOnly` if client-side code needs access.

---

## Core Concept 4: SameSite Cookies

### Definitions

**Core Definition:** The `SameSite` cookie attribute controls whether a cookie is sent with cross-site requests, providing defence against Cross-Site Request Forgery (CSRF) and cross-site information leakage.

**Technical Definition:** The `SameSite` attribute accepts three values: `Strict` (cookie sent only for same-site requests), `Lax` (cookie sent for same-site requests and top-level cross-site navigations), and `None` (cookie sent in all contexts, requiring the `Secure` attribute). Modern browsers default to `Lax` for cookies that do not explicitly set the attribute.

**Beginner-Friendly Explanation:** SameSite is like a rule about who can carry your keys. `Strict` means only you can carry them. `Lax` means you can carry them and a trusted friend can carry them if they're walking to your house (top-level navigation). `None` means anyone can carry them, but only in a sealed envelope (`Secure`).

### Purposes

- To mitigate CSRF attacks by preventing cookies from being sent with cross-site requests.
- To balance security with usability for different types of cookies.
- To control third-party cookie behaviour for embedded content and integrations.
- To comply with modern browser defaults that enforce `SameSite=Lax` when unspecified.

### Sub-Feature 4.1: Strict, Lax, and None Configurations

#### Definitions

**Core Definition:** Each `SameSite` value represents a different trade-off between security and cross-site functionality.

**Technical Definition:** `Strict` cookies are only sent in a first-party context — the cookie is not attached to any request initiated by a third-party website. `Lax` cookies are sent for top-level navigations (clicking a link to your site) but not for cross-site subresource requests or background POSTs. `None` cookies are sent in all contexts and must be paired with `Secure`.

**Beginner-Friendly Explanation:** `Strict` is the most secure but can break workflows where users arrive from external links and expect to be logged in. `Lax` is the modern default — it allows top-level navigation but blocks cross-site POSTs, preventing most CSRF attacks. `None` is for legitimate cross-site use cases like embedded widgets or payment iframes, and must always be `Secure`.

#### Purposes

- To prevent CSRF attacks by restricting cookie transmission in cross-site contexts.
- To choose the appropriate security-usability trade-off for each cookie's purpose.
- To support legitimate cross-site integrations (OAuth, embedded content) with `None`.
- To align with browser defaults that enforce `Lax` automatically.

#### Syntax Rules and Structure

```js
// Session cookie: Lax (default recommendation)
res.cookie('session', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax'
});

// Embedded widget cookie: None (requires Secure)
res.cookie('widget_token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'none'
});

// High-security cookie: Strict
res.cookie('admin_session', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict'
});
```

| Value | Cross-Site GET | Cross-Site POST | Requires Secure |
|-------|---------------|-----------------|-----------------|
| Strict | No | No | No |
| Lax | Top-level only | No | No |
| None | Yes | Yes | Yes |

**Constraints and Limitations:**
- `SameSite=None` requires the `Secure` attribute; without it, the browser rejects the cookie.
- `Strict` can break OAuth callback flows and external link navigation.
- Browser defaults have evolved; explicitly setting `SameSite` is recommended for consistency.

#### Annotated Code Example

```js
// samesite-config.js — SameSite for different cookie types
const express = require('express');
const session = require('express-session');
const app = express();

// Session cookie: Lax (balance of security and usability)
app.use(session({
  secret: process.env.SESSION_SECRET || 'dev-secret',
  name: 'app.sid',
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',           // Top-level navigation allowed
    maxAge: 3600000
  }
}));

// Embedded widget cookie: None (for cross-site iframe)
app.get('/widget-login', (req, res) => {
  res.cookie('widget_token', 'token-value', {
    httpOnly: true,
    secure: true,
    sameSite: 'none'           // Required for cross-site iframe
  });
  res.json({ message: 'Widget cookie set' });
});

// Admin cookie: Strict (maximum security)
app.get('/admin-login', (req, res) => {
  res.cookie('admin_session', 'admin-token', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict'         // No cross-site transmission
  });
  res.json({ message: 'Admin cookie set' });
});

app.listen(3000, () => console.log('SameSite server on 3000'));
```

**Expected Response Headers:**
```
Set-Cookie: app.sid=...; HttpOnly; Secure; SameSite=Lax
Set-Cookie: widget_token=...; HttpOnly; Secure; SameSite=None
Set-Cookie: admin_session=...; HttpOnly; Secure; SameSite=Strict
```

**Why this output:** Each cookie has a different `SameSite` value based on its purpose. The session cookie uses `Lax` for balance. The widget token uses `None` because it must work in a cross-site iframe. The admin session uses `Strict` for maximum protection.

#### Real-World Cases

- **OAuth flows:** `SameSite=Lax` allows the callback redirect to carry the session cookie.
- **Embedded payment iframes:** `SameSite=None; Secure` allows the iframe to send cookies.
- **Admin panels:** `SameSite=Strict` prevents any cross-site request from carrying the admin session.
- **Third-party widgets:** `SameSite=None` for cookies that must be sent in embedded contexts.

---

## Core Concept 5: Security Headers

### Definitions

**Core Definition:** Security headers are HTTP response headers that instruct the browser to enforce policies controlling resource loading, referrer information, browser feature access, and violation reporting.

**Technical Definition:** Key security headers include `Permissions-Policy` (controls browser features like camera and geolocation), `Referrer-Policy` (controls how much referrer information is sent with requests), and the Reporting API headers (`Reporting-Endpoints` and `Report-To`) which define endpoints for violation reports. Unlike Helmet's default headers, these require explicit configuration for each application's specific needs.

**Beginner-Friendly Explanation:** Security headers are like rules you give the browser: "Don't let the page use the camera," "Don't send the full URL of this page when someone clicks a link," and "If something violates these rules, send me a report so I know." These rules are tailored to each application.

### Purposes

- To control which browser features (camera, microphone, geolocation) the page and its iframes can use.
- To control how much referrer information is leaked to other sites.
- To receive violation reports for CSP, Permissions-Policy, and other policy headers.
- To comply with security standards that require explicit feature restriction.

### Sub-Feature 5.1: Permissions-Policy

#### Definitions

**Core Definition:** The `Permissions-Policy` header (formerly `Feature-Policy`) controls which browser features and APIs can be used by the document and any embedded iframes.

**Technical Definition:** The header uses a directive-based syntax where each directive maps a feature (e.g., `camera`, `geolocation`, `microphone`) to an allowlist. An empty allowlist `()` disables the feature entirely. `self` allows the feature for the same origin. `*` allows it for all origins. The default allowlist for most directives is `self`.

**Beginner-Friendly Explanation:** Permissions-Policy is like a settings panel for your page's browser features. You can turn off the camera, microphone, and location entirely, or allow them only for your own site.

#### Purposes

- To disable browser features that the application does not need.
- To prevent third-party iframes from accessing sensitive APIs (camera, microphone).
- To reduce the attack surface for malicious scripts that might try to abuse browser APIs.
- To comply with security standards that require explicit feature restriction.

#### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  res.setHeader(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(), payment=(), usb=()'
  );
  next();
});
```

| Directive | Breakdown |
|-----------|-----------|
| `camera=()` | Disables camera for all origins. |
| `geolocation=(self)` | Allows geolocation only for same origin. |
| `microphone=()` | Disables microphone. |
| `payment=()` | Disables Payment Request API. |

**Constraints and Limitations:**
- The header is experimental but widely supported in modern browsers.
- Directives must be comma-separated; invalid syntax is ignored.
- Some features have different default allowlists; specify explicitly for clarity.

#### Annotated Code Example

```js
// permissions-policy.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  res.setHeader(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(), payment=(), usb=(), magnetometer=(), gyroscope=()'
  );
  next();
});

app.get('/', (req, res) => {
  res.send('Permissions-Policy applied');
});

app.listen(3000, () => console.log('Permissions server on 3000'));
```

**Expected Response Header:**
```
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), magnetometer=(), gyroscope=()
```

**Why this output:** Every listed feature is disabled for all origins (`()`). The application explicitly declares that it does not need these browser APIs, reducing the attack surface.

#### Real-World Cases

- **API-only services:** Disable all browser features; the API does not serve HTML.
- **SaaS applications:** Allow only the features the application needs (e.g., geolocation for a delivery app).
- **Embedded widgets:** Restrict iframe access to sensitive APIs.

---

### Sub-Feature 5.2: Referrer-Policy

#### Definitions

**Core Definition:** The `Referrer-Policy` header controls how much referrer information (the `Referer` header) is included with requests made from the page.

**Technical Definition:** Valid values include `no-referrer` (never send referrer), `origin` (send only the origin), `same-origin` (send full referrer for same-origin requests, none for cross-origin), and `strict-origin-when-cross-origin` (send full referrer for same-origin and HTTPS→HTTPS cross-origin, but only origin for HTTPS→HTTP). Helmet's default is `no-referrer`.

**Beginner-Friendly Explanation:** When you click a link from your site to another site, the browser normally tells the other site where you came from. Referrer-Policy controls how much of that information is shared. `no-referrer` shares nothing. `strict-origin-when-cross-origin` shares the full URL for same-site clicks but only the domain for cross-site clicks.

#### Purposes

- To prevent leaking sensitive URL information (tokens, paths) to external sites.
- To comply with privacy regulations (GDPR, CCPA) that restrict data sharing.
- To control referrer behaviour for analytics and partner integrations.
- To balance privacy with legitimate analytics needs.

#### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  next();
});
```

| Value | Behaviour |
|-------|-----------|
| `no-referrer` | Never send referrer. |
| `same-origin` | Full referrer for same-origin; none for cross-origin. |
| `strict-origin-when-cross-origin` | Full for same-origin; origin only for cross-origin. |
| `unsafe-url` | Always send full URL (not recommended). |

**Constraints and Limitations:**
- The header only affects requests made **from** the page; it does not control incoming referrers.
- Older browsers may not support all values; `no-referrer-when-downgrade` is a legacy fallback.

#### Annotated Code Example

```js
// referrer-policy.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  next();
});

app.get('/', (req, res) => {
  res.send('Referrer-Policy applied');
});

app.listen(3000, () => console.log('Referrer server on 3000'));
```

**Expected Response Header:**
```
Referrer-Policy: strict-origin-when-cross-origin
```

**Why this output:** The header tells the browser to send the full referrer URL for same-origin requests but only the origin (domain) for cross-origin requests. This protects sensitive path information when linking to external sites.

#### Real-World Cases

- **Privacy-focused applications:** `no-referrer` prevents any referrer leakage.
- **Analytics-heavy sites:** `strict-origin-when-cross-origin` allows internal analytics while limiting external leakage.
- **SaaS with partner integrations:** Balance between referrer sharing and privacy.

---

### Sub-Feature 5.3: Reporting API (Reporting-Endpoints and report-to)

#### Definitions

**Core Definition:** The Reporting API allows the browser to send violation reports (CSP violations, deprecation warnings, network errors) to a specified endpoint, enabling monitoring and debugging of security policy enforcement.

**Technical Definition:** The `Reporting-Endpoints` header defines named endpoints as a comma-separated list of `name="url"` pairs. Policy headers (CSP, Permissions-Policy, Document-Policy) include a `report-to` directive referencing one of these endpoint names. Deprecation, intervention, and crash reports are sent to the endpoint named `default` without requiring a `report-to` directive. The `Reporting-Endpoints` header supersedes the older `Report-To` header.

**Beginner-Friendly Explanation:** The Reporting API is like a security camera that sends you a notification whenever someone violates one of your rules. You set up an endpoint (a URL on your server), and you tell the browser "if a CSP violation happens, send a report to this URL." You can then analyse these reports to find and fix security policy issues.

#### Purposes

- To receive real-time notifications of CSP violations, deprecation warnings, and network errors.
- To debug and tune security policies (CSP, Permissions-Policy) based on actual violations.
- To monitor for deprecation of browser features used by the application.
- To comply with security monitoring requirements in regulated environments.

#### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  res.setHeader(
    'Reporting-Endpoints',
    'csp-endpoint="https://report.example.com/csp", default="https://report.example.com/default"'
  );
  next();
});

// CSP with report-to directive
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "script-src 'self'; report-to csp-endpoint"
  );
  next();
});
```

| Component | Breakdown |
|-----------|-----------|
| `Reporting-Endpoints` | Defines named endpoints. |
| `csp-endpoint` | Name referenced by `report-to` directive. |
| `default` | Receives deprecation/intervention reports automatically. |
| `report-to` | Directive in policy headers referencing an endpoint. |

**Constraints and Limitations:**
- Reports may be sent with a delay (up to a minute); Chrome's `--short-reporting-delay` flag reduces this during development.
- The `default` endpoint is mandatory for deprecation and intervention reports.
- `Reporting-Endpoints` is replacing `Report-To`; use the newer header for future compatibility.
- Endpoints must be HTTPS; non-secure endpoints are ignored.

#### Annotated Code Example

```js
// reporting-api.js — Reporting API with Express
const express = require('express');
const app = express();
app.use(express.json({ type: ['application/json', 'application/reports+json'] }));

// Set Reporting-Endpoints on all responses
app.use((req, res, next) => {
  res.setHeader(
    'Reporting-Endpoints',
    'csp-endpoint="https://report.example.com/csp", default="https://report.example.com/default"'
  );
  next();
});

// CSP with report-to
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self'; report-to csp-endpoint"
  );
  next();
});

// Report collection endpoint
app.post('/csp', (req, res) => {
  console.log('CSP violation report:', req.body);
  res.status(204).end();
});

app.post('/default', (req, res) => {
  console.log('Default report:', req.body);
  res.status(204).end();
});

app.get('/', (req, res) => {
  res.send('Reporting API configured');
});

app.listen(3000, () => console.log('Reporting server on 3000'));
```

**Expected Response Headers:**
```
Reporting-Endpoints: csp-endpoint="https://report.example.com/csp", default="https://report.example.com/default"
Content-Security-Policy: default-src 'self'; script-src 'self'; report-to csp-endpoint
```

**Expected Output (server console when a CSP violation occurs):**
```
CSP violation report: { "csp-report": { "document-uri": "...", "violated-directive": "script-src", ... } }
```

**Why this output:** The browser sends a POST request to the `/csp` endpoint with a JSON body containing the violation details. The server logs the report for analysis.

#### Real-World Cases

- **Large applications with CSP:** Reporting API helps identify and fix CSP violations without breaking functionality.
- **Compliance monitoring:** Reports provide evidence of policy enforcement for audits.
- **Deprecation tracking:** Receive alerts when browser features used by the application are deprecated.

---

## References

- Node.js HTTPS Documentation — https://nodejs.org/api/https.html
- Node.js TLS Documentation — https://nodejs.org/api/tls.html
- Node.js `tls.getCiphers()` — https://nodejs.org/api/tls.html#tlsgetciphers
- Node.js HTTPS Server Options — https://nodejs.org/api/https.html#httpscreateserveroptions-requestlistener
- Express.js Security Best Practices — https://expressjs.com/en/advanced/best-practice-security/
- Express.js Session Middleware — https://expressjs.com/en/resources/middleware/session.html
- Express.js Behind Proxies — https://expressjs.com/en/guide/behind-proxies.html
- Helmet.js Official Documentation — https://helmet.js.org/
- Helmet HSTS Middleware — https://helmet.js.org/middlewares/strict-transport-security/
- HSTS Preload List Submission — https://hstspreload.org/
- MDN HTTP Strict Transport Security — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
- MDN Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- MDN SameSite Cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- MDN Permissions-Policy — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy
- MDN Referrer-Policy — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
- MDN Reporting-Endpoints — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Reporting-Endpoints
- Chrome Reporting API Guide — https://developer.chrome.com/docs/capabilities/web-apis/reporting-api
- OWASP Secure Headers Project — https://owasp.org/www-project-secure-headers/
- OWASP Session Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP TLS Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html
- RFC 8446 (TLS 1.3) — https://datatracker.ietf.org/doc/html/rfc8446
- RFC 6797 (HSTS) — https://datatracker.ietf.org/doc/html/rfc6797
- CVE-2024-47764 (cookie package vulnerability) — https://nvd.nist.gov/vuln/detail/CVE-2024-47764