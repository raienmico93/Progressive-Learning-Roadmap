# CORS — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Cross-Origin Resource Sharing (CORS)** is a **browser-enforced security mechanism** defined by the WHATWG Fetch Standard that allows a server to **declare which origins** — combinations of scheme, host, and port — are permitted to read its responses from a web browser. CORS relaxes the **same-origin policy (SOP)** in a controlled, header-based manner, enabling legitimate cross-origin data sharing while preventing unauthorized scripts from reading sensitive responses .

**Technical Definition.** CORS is an **HTTP-header-based protocol** in which the browser and server negotiate cross-origin access. For **non-simple requests**, the browser first sends a **preflight `OPTIONS` request** carrying `Access-Control-Request-Method` and `Access-Control-Request-Headers`; the server responds with `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, and optionally `Access-Control-Allow-Credentials` and `Access-Control-Max-Age`. The browser evaluates these headers against the actual request and either permits or blocks the response from being read by JavaScript. **CORS is enforced entirely by the browser**; non-browser clients (curl, Postman, servers) ignore CORS headers entirely .

**Beginner-Friendly Explanation.** CORS is **the browser asking your server for permission before letting one website read data from another**. By default, a webpage at `https://app.com` cannot read a response from `https://api.com` — that is the **same-origin policy**. CORS is the mechanism that lets `api.com` say “yes, `app.com` is allowed to read my data.” If the server does not say so, the browser blocks the response. **CORS protects users, not servers**: it stops a malicious site from silently reading your bank’s API using your logged-in session.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Browser-enforced** | CORS is checked by the browser, not by the server; the server only supplies headers, and non-browser clients ignore them. |
| **Origin-based** | An **origin** is the triple (scheme, host, port); `https://app.com` and `http://app.com` are different origins. |
| **Header-negotiated** | Access is granted through `Access-Control-Allow-*` response headers, not through server-side blocking. |
| **Preflight for non-simple requests** | Requests that use non-simple methods or non-safelisted headers trigger an `OPTIONS` preflight before the actual request. |
| **Credentials require explicit opt-in** | Cookies, TLS client certificates, and `Authorization` headers are sent only when both client and server explicitly enable credentials. |
| **Wildcard has limits** | `Access-Control-Allow-Origin: *` is valid only when credentials are **not** used. |

---

### Prerequisites

- **HTTP fundamentals**: methods, status codes, headers, and the request/response model.
- **The same-origin policy**: understanding why browsers restrict cross-origin reads.
- **A browser and a server**: to observe preflight behavior in DevTools.
- **Node.js and Express** (for the `cors` package examples).
- **`curl` or Postman**: to inspect response headers directly.

---

### Related Programming Areas

- **Browser security model**: CORS is a relaxation of the same-origin policy.
- **API security**: CORS misconfiguration is a recurring OWASP Top 10 issue (Security Misconfiguration).
- **Authentication**: credentialed CORS interacts directly with cookies and `Authorization` headers.
- **Web application architecture**: SPAs on separate domains from their APIs depend on CORS for every API call.

---

## 1. CORS

### Definitions

**Core Definition.** **CORS** is the HTTP-header-based mechanism that lets a server authorize browsers to expose cross-origin responses to JavaScript.

**Technical Definition.** The CORS protocol distinguishes **simple requests** (which proceed directly, with the browser checking the response headers) from **preflighted requests** (which require a prior `OPTIONS` exchange). The browser attaches an **`Origin`** header to every cross-origin request; the server responds with `Access-Control-Allow-Origin` and related headers; the browser compares them and either exposes or blocks the response.

**Beginner-Friendly Explanation.** CORS is **the permission slip system for cross-origin requests**. The browser always sends the `Origin` header so the server knows who is asking. The server replies with headers saying who is allowed. The browser reads those headers and decides whether your JavaScript code gets to see the response.

### Purposes

- **To relax the same-origin policy safely** for legitimate cross-origin data sharing.
- **To protect users from malicious cross-origin reads** by requiring explicit server authorization.
- **To enable modern web architectures** where frontends and APIs live on different origins.
- **To standardize cross-origin access** across browsers and server frameworks.

### Syntax Rules and Structure

| Request Header (Browser → Server) | Meaning |
|---|---|
| `Origin` | The origin of the page making the request (always sent for CORS requests). |
| `Access-Control-Request-Method` | (Preflight only) The HTTP method the actual request will use. |
| `Access-Control-Request-Headers` | (Preflight only) The non-safelisted headers the actual request will send. |

| Response Header (Server → Browser) | Meaning |
|---|---|
| `Access-Control-Allow-Origin` | The allowed origin, or `*` for any origin (incompatible with credentials). |
| `Access-Control-Allow-Methods` | The allowed HTTP methods (preflight response). |
| `Access-Control-Allow-Headers` | The allowed request headers (preflight response). |
| `Access-Control-Allow-Credentials` | `true` if credentials are allowed. |
| `Access-Control-Max-Age` | Seconds the preflight result may be cached. |
| `Access-Control-Expose-Headers` | Response headers exposed to JavaScript beyond the safelist. |

**Constraints and limitations.**

- **CORS does not block requests**; it blocks the browser from **reading the response**. The request may still reach the server .
- **`Access-Control-Allow-Origin: *` cannot be combined with credentials**; the browser rejects such responses .
- **The `Origin` header can be spoofed by non-browser clients**; CORS is not a server-side access control mechanism .

### Annotated Complete Step by Step Code Examples

#### Example 1: A simple cross-origin request

```javascript
// Step 1: The browser sends a simple GET with an Origin header.
fetch("https://api.example.com/data")
  .then((res) => res.json())
  .then((data) => console.log(data));
```

**Expected Request:**
```
GET /data HTTP/1.1
Origin: https://app.example.com
```

**Expected Response (allowed):**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Content-Type: application/json
```

**Expected Response (blocked):**
```
HTTP/1.1 200 OK
Content-Type: application/json
(no Access-Control-Allow-Origin header)
→ Browser blocks JavaScript from reading the response.
```

**Why this output occurs.** The browser sends `Origin` automatically. If the server echoes the origin in `Access-Control-Allow-Origin`, the browser exposes the response. If the header is missing or does not match, the response is still delivered but **invisible to JavaScript**.

---

### Real-World Cases

- **SPAs calling APIs on a different domain**: the API must allow the SPA’s origin.
- **Third-party widgets**: embedded widgets call the host API with the widget origin.
- **Public APIs**: use `Access-Control-Allow-Origin: *` for unauthenticated endpoints.

---

## 2. The `cors` Package

### Definitions

**Core Definition.** The **`cors`** npm package is a **Connect/Express middleware** that sets CORS response headers according to configurable options, simplifying CORS setup in Node.js applications .

**Technical Definition.** `cors(options)` returns middleware that inspects the request’s `Origin` header and sets the appropriate `Access-Control-Allow-*` headers. It supports static origins, dynamic origin validation via a callback, preflight handling, credential support, and per-route or global mounting.

**Beginner-Friendly Explanation.** The `cors` package is **the standard way to add CORS headers in Express**. Instead of writing header logic manually, you call `app.use(cors())` and it handles the headers for you. You can customize which origins, methods, and headers are allowed.

### Purposes

- **To add CORS headers with minimal code** in Express/Connect applications.
- **To support dynamic origin validation** through a callback function .
- **To handle preflight requests automatically** when mounted globally.
- **To configure credentials, methods, headers, and max-age** per route or globally.

### Syntax Rules and Structure

**General syntax.**

```javascript
const cors = require("cors");
app.use(cors(options));
```

**Key options.**

| Option | Type | Description |
|---|---|---|
| `origin` | `Boolean \| String \| RegExp \| Array \| Function` | Configures `Access-Control-Allow-Origin`. `true` reflects the request origin; `false` disables CORS . |
| `methods` | `String \| Array` | Configures `Access-Control-Allow-Methods` (e.g., `"GET,HEAD,PUT,PATCH,POST,DELETE"`). |
| `allowedHeaders` | `String \| Array` | Configures `Access-Control-Allow-Headers`. |
| `exposedHeaders` | `String \| Array` | Configures `Access-Control-Expose-Headers`. |
| `credentials` | `Boolean` | Configures `Access-Control-Allow-Credentials`. |
| `maxAge` | `Number` | Configures `Access-Control-Max-Age` (seconds). |
| `preflightContinue` | `Boolean` | Passes the preflight response to the next handler instead of ending it. |
| `optionsSuccessStatus` | `Number` | Status for successful `OPTIONS` (default `204`; some legacy clients need `200`) . |

**Constraints and limitations.**

- **The package sets headers; it does not block requests** — the browser enforces CORS .
- **Credentials with `origin: "*"` or `true`** may produce an invalid combination; use an explicit origin list with `credentials: true`.
- **Dynamic origin functions must call the callback**; failing to do so hangs the request.

### Annotated Complete Step by Step Code Examples

#### Example 1: Global CORS with a specific origin

```javascript
// Step 1: Import and configure.
const express = require("express");
const cors = require("cors");
const app = express();

const corsOptions = {
  origin: "https://app.example.com",
  credentials: true,
  optionsSuccessStatus: 200,
};

// Step 2: Mount globally.
app.use(cors(corsOptions));

app.get("/api/data", (req, res) => {
  res.json({ message: "Hello" });
});

app.listen(3000);
```

**Expected Response:**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Vary: Origin
```

**Why this output occurs.** The middleware matches the request origin against the configured origin and sets the allow-origin and credentials headers. `Vary: Origin` is added so caches do not serve the wrong origin’s response.

---

#### Example 2: Dynamic origin validation

```javascript
// Step 1: Define a dynamic origin function.
const corsOptions = {
  origin: (origin, callback) => {
    const allowed = ["https://app.example.com", "https://admin.example.com"];
    if (!origin || allowed.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS"));
    }
  },
  credentials: true,
};

// Step 2: Mount the middleware.
app.use(cors(corsOptions));
```

**Expected Output (allowed):**
```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```

**Expected Output (disallowed):**
```
→ The callback receives an error; the request is rejected or the header is not set.
```

**Why this output occurs.** The function receives the request’s origin and calls `callback(null, true)` for allowed origins or `callback(error)` for disallowed ones. This pattern supports loading allowed origins from a database or configuration .

---

### Real-World Cases

- **Express APIs**: the standard middleware for CORS in Node.js services.
- **Multi-tenant SaaS**: dynamic origin validation against a tenant registry.
- **Development vs. production**: permissive CORS in development, strict allow-lists in production.

---

## 3. Origin

### Definitions

**Core Definition.** An **origin** is the **combination of scheme, host, and port** that identifies where a resource came from. Two URLs share an origin only if all three components match.

**Technical Definition.** Origins are compared using the **ASCII serialization** of the tuple `(scheme, host, port)`. For example, `https://example.com` and `https://example.com:443` are the same origin (the default port is implicit), while `http://example.com` and `https://example.com` are different origins.

**Beginner-Friendly Explanation.** An origin is **the “address” of a website** — its protocol, domain, and port. `https://app.com` and `https://api.com` are different origins. `https://app.com` and `http://app.com` are **also** different, because the protocol differs. CORS decisions are made origin by origin.

### Purposes

- **To define the security boundary** for the same-origin policy and CORS.
- **To identify the caller** in the `Origin` request header.
- **To scope permissions** to precise combinations of scheme, host, and port.

### Syntax Rules and Structure

| Component | Example | Note |
|---|---|---|
| **Scheme** | `https` | `http` and `https` are different origins. |
| **Host** | `example.com` | Subdomains are different origins (`app.example.com` ≠ `example.com`). |
| **Port** | `443` | Default ports are implicit but still part of the origin. |

**Origin serialization.**

```
https://app.example.com:443
```

**Constraints and limitations.**

- **`Origin: null`** is sent by sandboxed iframes, `file://` pages, and some redirects; treat it as an untrusted origin.
- **The `Origin` header can be spoofed** by non-browser clients; never use it as server-side authorization .

### Annotated Complete Step by Step Code Examples

#### Example 1: Comparing origins

```javascript
// Same origin:
https://app.example.com/page1
https://app.example.com/page2
// Different origins:
https://app.example.com
http://app.example.com        // scheme differs
https://api.example.com       // host differs
https://app.example.com:8443  // port differs
```

**Expected Output (conceptual):**
```
The first pair is same-origin (no CORS needed).
The second group each requires CORS because one component differs.
```

**Why this output occurs.** The origin is the full triple. Changing any one component creates a new origin, and the browser treats the request as cross-origin.

---

### Real-World Cases

- **Microfrontends**: each microfrontend may be served from a different origin, requiring CORS.
- **Local development**: `http://localhost:3000` calling `http://localhost:4000` is cross-origin (different port).
- **Custom domains**: `app.example.com` calling `api.example.com` is cross-origin (different subdomain).

---

## 4. Methods

### Definitions

**Core Definition.** In CORS, **methods** refer to the HTTP methods that a cross-origin request may use, and the `Access-Control-Allow-Methods` response header declares which methods the server permits.

**Technical Definition.** The Fetch Standard defines a **CORS-safelisted method set** (`GET`, `HEAD`, `POST`) that does not by itself trigger a preflight. Other methods (`PUT`, `PATCH`, `DELETE`, etc.) force a preflight, and the server must list them in `Access-Control-Allow-Methods` for the actual request to proceed .

**Beginner-Friendly Explanation.** Methods are **the HTTP verbs** your cross-origin request uses. `GET`, `HEAD`, and `POST` are considered “simple” and may proceed without a preflight. `PUT`, `PATCH`, and `DELETE` trigger a preflight, and the server must explicitly allow them.

### Purposes

- **To restrict which operations** cross-origin clients can perform.
- **To satisfy preflight requirements** for non-simple methods.
- **To reduce preflight overhead** by aligning allowed methods with actual API usage.

### Syntax Rules and Structure

**General syntax (response header).**

```
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

**Safelisted methods (no preflight for method alone).**

```
GET, HEAD, POST
```

**Constraints and limitations.**

- **Safelisted methods still trigger a preflight** if non-safelisted headers are sent.
- **`Access-Control-Allow-Methods` is only meaningful in preflight responses**; it is not required on the actual response.

### Annotated Complete Step by Step Code Examples

#### Example 1: Preflight for a `PUT` request

```http
OPTIONS /api/item/1 HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PUT
```

**Expected Preflight Response:**
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: PUT, DELETE
Access-Control-Max-Age: 3600
```

**Expected Actual Response:**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
```

**Why this output occurs.** `PUT` is not safelisted, so the browser sends a preflight. The server responds with `Access-Control-Allow-Methods` including `PUT`. The browser then sends the actual `PUT`, and the server includes `Access-Control-Allow-Origin` on the actual response.

---

### Real-World Cases

- **REST APIs**: `GET`, `POST`, `PUT`, `PATCH`, `DELETE` are common; all non-`GET`/`POST` methods require preflight.
- **Read-only public APIs**: allow only `GET` and `HEAD`.
- **Admin APIs**: allow `GET`, `POST`, `PUT`, `DELETE` with credentials.

---

## 5. Headers

### Definitions

**Core Definition.** In CORS, **headers** refers to the request headers that may be sent cross-origin and the response headers that the server allows (`Access-Control-Allow-Headers`) or exposes (`Access-Control-Expose-Headers`).

**Technical Definition.** The Fetch Standard defines a **CORS-safelisted request-header set** (`Accept`, `Accept-Language`, `Content-Language`, `Content-Type` with limited values, `Range`). Sending any non-safelisted header forces a preflight carrying `Access-Control-Request-Headers`; the server must list those headers in `Access-Control-Allow-Headers` .

**Beginner-Friendly Explanation.** Headers are **the extra information attached to a request** — things like `Content-Type`, `Authorization`, or `X-Request-ID`. The browser automatically allows a few simple headers. If you send anything else, the browser asks the server in a preflight: “is this header okay?” The server must say yes in `Access-Control-Allow-Headers`.

### Purposes

- **To permit non-safelisted headers** such as `Authorization` and custom `X-` headers.
- **To expose non-safelisted response headers** to JavaScript via `Access-Control-Expose-Headers`.
- **To control header-based attack surface** by limiting which headers cross-origin clients may send.

### Syntax Rules and Structure

**Preflight request headers.**

```
Access-Control-Request-Headers: content-type, authorization, x-request-id
```

**Preflight response headers.**

```
Access-Control-Allow-Headers: Content-Type, Authorization, X-Request-ID
```

**Exposing response headers.**

```
Access-Control-Expose-Headers: X-Total-Count, X-Page-Size
```

**Constraints and limitations.**

- **Header name matching is case-insensitive**, but the server should echo canonical casing.
- **`Access-Control-Allow-Headers: *`** is a wildcard but is **not compatible with credentialed requests** in all browsers; prefer explicit lists.
- **Only safelisted response headers are readable by JavaScript** unless exposed .

### Annotated Complete Step by Step Code Examples

#### Example 1: Allowing `Authorization` and a custom header

```javascript
const corsOptions = {
  origin: "https://app.example.com",
  allowedHeaders: ["Content-Type", "Authorization", "X-Request-ID"],
  exposedHeaders: ["X-Total-Count"],
  credentials: true,
};
app.use(cors(corsOptions));
```

**Expected Preflight Response:**
```
Access-Control-Allow-Headers: Content-Type, Authorization, X-Request-ID
Access-Control-Expose-Headers: X-Total-Count
Access-Control-Allow-Credentials: true
```

**Why this output occurs.** The preflight lists the allowed request headers, and the actual response exposes `X-Total-Count` so the client can read it. `credentials: true` enables cookies and `Authorization`.

---

### Real-World Cases

- **JWT APIs**: `Authorization` is a non-safelisted header and always triggers preflight.
- **Pagination APIs**: `X-Total-Count` must be exposed for clients to read total counts.
- **CSRF-protected APIs**: custom `X-CSRF-Token` headers must be allowed.

---

## 6. Credentials

### Definitions

**Core Definition.** **Credentials** in CORS refers to **cookies, TLS client certificates, and `Authorization` headers** (including username/password) that may be included in a cross-origin request. The server must explicitly allow them with `Access-Control-Allow-Credentials: true` .

**Technical Definition.** By default, cross-origin requests made with `fetch()` do **not** include credentials. The client must set `credentials: "include"` (or `XMLHttpRequest.withCredentials = true`), and the server must respond with `Access-Control-Allow-Credentials: true`. **The wildcard `Access-Control-Allow-Origin: *` is invalid when credentials are used**; the server must echo a specific origin .

**Beginner-Friendly Explanation.** Credentials are **the things that prove who you are** — cookies, login headers, client certificates. By default, cross-origin requests do not send them, because that would let a malicious site act as you. If you want to send them, **both sides must agree**: the client says “include credentials,” and the server says “credentials are allowed.” And you cannot use the `*` wildcard with credentials.

### Purposes

- **To enable authenticated cross-origin API calls** (e.g., a SPA sending a session cookie to its API).
- **To prevent credential leakage** to untrusted origins by requiring explicit opt-in.
- **To support OAuth and JWT flows** where the `Authorization` header must cross origins.

### Syntax Rules and Structure

**Client side (Fetch).**

```javascript
fetch("https://api.example.com/me", {
  credentials: "include",
});
```

**Server side (Express `cors`).**

```javascript
app.use(cors({
  origin: "https://app.example.com",
  credentials: true,
}));
```

**Expected Response.**

```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```

**Constraints and limitations.**

- **`Access-Control-Allow-Origin: *` cannot be combined with credentials** .
- **Preflight requests never include credentials**; the preflight response must still indicate that credentials are allowed.
- **Cookies require `SameSite` compatibility**; `SameSite=None` must be paired with `Secure`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Credentialed cross-origin request

```javascript
// Step 1: Client sends credentials.
fetch("https://api.example.com/me", {
  credentials: "include",
  headers: { Authorization: "Bearer token" },
})
  .then((res) => res.json())
  .then((data) => console.log(data));
```

**Expected Response:**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Set-Cookie: session=abc123; Secure; HttpOnly
```

**Why this output occurs.** The client opts in with `credentials: "include"`. The server echoes the specific origin (not `*`) and sets `Access-Control-Allow-Credentials: true`. The browser accepts the response and processes the `Set-Cookie` header.

---

#### Example 2: The invalid wildcard + credentials combination

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

**Expected Browser Behavior:**
```
→ The browser blocks the response; credentials are not exposed.
```

**Why this output occurs.** The CORS specification forbids the combination of `*` and credentials. The browser rejects the response because it cannot determine which specific origin is trusted .

---

### Real-World Cases

- **Cookie-based sessions**: SPAs on `app.example.com` calling `api.example.com` with session cookies.
- **OAuth token APIs**: `Authorization` headers in cross-origin requests.
- **Single sign-on (SSO)**: cross-origin authentication flows that rely on cookies.

---

## 7. Preflight

### Definitions

**Core Definition.** A **preflight request** is an automatic `OPTIONS` request that the browser sends **before** a cross-origin request that is **not simple**, to ask the server whether the actual request is permitted .

**Technical Definition.** A request requires a preflight if it uses a method outside `GET`, `HEAD`, `POST`, or sends a request header outside the CORS-safelisted set. The preflight carries `Access-Control-Request-Method` and `Access-Control-Request-Headers`; the server responds with `Access-Control-Allow-Methods` and `Access-Control-Allow-Headers`. A successful preflight may be cached for `Access-Control-Max-Age` seconds .

**Beginner-Friendly Explanation.** A preflight is **the browser asking permission before sending the real request**. If your request is “simple” (a basic `GET` or `POST` with simple headers), it goes straight through. If it uses `PUT`, `DELETE`, or a custom header like `Authorization`, the browser first sends an `OPTIONS` request to the server: “can I send this?” If the server says yes, the real request follows.

### Purposes

- **To protect servers from unexpected cross-origin requests** with side effects.
- **To allow the server to declare permitted methods and headers** before the actual request is sent.
- **To reduce repeated preflight overhead** through `Access-Control-Max-Age` caching .

### Syntax Rules and Structure

**Preflight request.**

```
OPTIONS /api/resource HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization, Content-Type
```

**Preflight response.**

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 3600
```

**Simple request conditions (no preflight).**

| Condition | Requirement |
|---|---|
| **Method** | `GET`, `HEAD`, or `POST` |
| **Headers** | Only CORS-safelisted headers |
| **`Content-Type`** | `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` |

**Constraints and limitations.**

- **Preflight requests never include credentials** .
- **A failed preflight blocks the actual request**; the browser returns a network error to JavaScript.
- **`Access-Control-Max-Age` is capped by browsers** (e.g., Chromium caps at 2 hours in modern versions) .

### Annotated Complete Step by Step Code Examples

#### Example 1: Full preflight flow

```javascript
// Step 1: Client sends a non-simple request.
fetch("https://api.example.com/items/1", {
  method: "PUT",
  headers: {
    "Content-Type": "application/json",
    Authorization: "Bearer token",
  },
  body: JSON.stringify({ name: "Updated" }),
});
```

**Expected Preflight Request:**
```
OPTIONS /items/1 HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: content-type, authorization
```

**Expected Preflight Response:**
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, PUT
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 3600
```

**Expected Actual Request:**
```
PUT /items/1 HTTP/1.1
Origin: https://app.example.com
Authorization: Bearer token
Content-Type: application/json
```

**Expected Actual Response:**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
```

**Why this output occurs.** `PUT` and `Authorization` are both non-simple, so the browser sends a preflight. The server allows both, and the browser proceeds with the actual `PUT`. The actual response also includes `Access-Control-Allow-Origin` so the client can read it.

---

### Real-World Cases

- **REST APIs with `PUT`/`DELETE`**: every such request triggers a preflight.
- **Authenticated APIs**: `Authorization` headers always trigger preflight.
- **High-traffic APIs**: `Access-Control-Max-Age` reduces preflight overhead by caching the result.

---

## 8. CORS Best Practices

### Definitions

**Core Definition.** **CORS best practices** are guidelines for configuring CORS so that it **enables legitimate cross-origin access without opening the application to data theft or misconfiguration vulnerabilities**.

**Technical Definition.** Best practices center on **strict origin allow-lists**, **explicit method and header allow-lists**, **careful credential handling**, **preflight caching**, **environment-aware configuration**, and **avoiding reflection of untrusted origins** .

**Beginner-Friendly Explanation.** Best practices mean **being specific, not permissive**. Do not use `*` unless the API is truly public. List exactly which origins, methods, and headers you allow. Never reflect arbitrary origins back. If you use credentials, use explicit origins. And test your configuration with real browser requests.

### Purposes

- **To prevent data exposure** to unauthorized origins.
- **To avoid OWASP Top 10 Security Misconfiguration** findings .
- **To reduce preflight overhead** through caching and appropriate allow-lists.
- **To keep development and production configurations separate**.

### Syntax Rules and Structure

| Area | Best Practice |
|---|---|
| **Origin** | Use an explicit allow-list; avoid `*` for authenticated APIs . |
| **Dynamic origin** | Validate against a trusted source; never reflect `req.headers.origin` without checks. |
| **Credentials** | Use `credentials: true` only with explicit origins; never combine with `*` . |
| **Methods** | Allow only the methods the API actually uses. |
| **Headers** | Allow only the headers the client actually sends; expose only necessary response headers. |
| **Preflight cache** | Set `Access-Control-Max-Age` to reduce preflight overhead . |
| **Environment** | Use permissive CORS in development and strict allow-lists in production. |
| **Testing** | Test with real browser requests, not just `curl` (which ignores CORS). |

**Constraints and limitations.**

- **CORS is not server-side authorization**; it does not prevent non-browser clients from calling the API .
- **`Access-Control-Allow-Origin: *` is acceptable only for genuinely public, unauthenticated APIs** .

### Annotated Complete Step by Step Code Examples

#### Example 1: Production-grade CORS configuration

```javascript
const corsOptions = {
  // Step 1: Strict origin allow-list.
  origin: ["https://app.example.com", "https://admin.example.com"],

  // Step 2: Only the methods the API uses.
  methods: ["GET", "POST", "PUT", "DELETE"],

  // Step 3: Only the headers the client sends.
  allowedHeaders: ["Content-Type", "Authorization"],

  // Step 4: Expose only necessary response headers.
  exposedHeaders: ["X-Total-Count"],

  // Step 5: Credentials with explicit origins.
  credentials: true,

  // Step 6: Cache preflight for 1 hour.
  maxAge: 3600,
};

app.use(cors(corsOptions));
```

**Expected Response (allowed origin):**
```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Expose-Headers: X-Total-Count
Access-Control-Max-Age: 3600
Vary: Origin
```

**Expected Response (disallowed origin):**
```
→ No CORS headers are set; the browser blocks the response.
```

**Why this output occurs.** The explicit allow-list prevents arbitrary origins from receiving the CORS headers. The methods and headers lists are minimal. Credentials are paired with specific origins, and `Vary: Origin` ensures caches do not serve the wrong origin’s response.

---

#### Example 2: Development vs. production configuration

```javascript
const isProduction = process.env.NODE_ENV === "production";

const corsOptions = isProduction
  ? {
      origin: ["https://app.example.com"],
      credentials: true,
      maxAge: 3600,
    }
  : {
      origin: true, // reflect any origin in development
      credentials: true,
    };

app.use(cors(corsOptions));
```

**Expected Output (development):**
```
Access-Control-Allow-Origin: http://localhost:3000
```

**Expected Output (production):**
```
Access-Control-Allow-Origin: https://app.example.com
```

**Why this output occurs.** Development uses a permissive configuration to support local tools and varying ports, while production enforces a strict allow-list. The same codebase serves both environments safely.

---

### Real-World Cases

- **OWASP recommendations**: explicitly warn against `Access-Control-Allow-Origin: *` with credentials and against reflecting untrusted origins .
- **Public APIs**: may use `*` for unauthenticated, read-only endpoints, but should document the limitation.
- **Multi-environment deployments**: use environment variables or configuration files to vary CORS per environment.

---

## References

- MDN — Cross-Origin Resource Sharing (CORS) - https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN — Access-Control-Allow-Credentials - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials
- MDN — Preflight request - https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request
- WHATWG Fetch Standard — CORS Protocol - https://fetch.spec.whatwg.org/#http-cors-protocol
- WHATWG Fetch Standard — CORS-preflight fetch - https://fetch.spec.whatwg.org/#cors-preflight-fetch
- npm — cors Package - https://www.npmjs.com/package/cors
- expressjs/cors — GitHub Repository - https://github.com/expressjs/cors
- OWASP WSTG — Testing Cross Origin Resource Sharing - https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing
- OWASP — CORS Misconfiguration Prevention Guide - https://www.securecodinghub.com
- W3C — Cross-Origin Resource Sharing (Historical Specification) - https://www.w3.org/TR/cors/
- MDN — Access-Control-Max-Age - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age
- W3C — Preflight Result Cache - https://www.w3.org/TR/cors/#preflight-result-cache