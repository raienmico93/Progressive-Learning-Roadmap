# Secure Client Architecture in React: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Secure Client Architecture in React is the set of architectural patterns, browser security mechanisms, and server-side controls that protect a React single-page application from client-side attacks—principally XSS, CSRF, and token theft—while ensuring that sensitive credentials never reach a place where JavaScript can read them.

**Technical Definition:** Secure Client Architecture encompasses four interlocking domains: (1) **XSS mitigation**, which relies on React's auto-escaping, disciplined avoidance of dangerous DOM sinks (e.g., `dangerouslySetInnerHTML`, `innerHTML`), and the storage of authentication tokens in `HttpOnly` cookies rather than `localStorage`; (2) **CSRF defence**, implemented through a combination of `SameSite` cookie attributes, synchronizer tokens or double-submit cookies, and custom request headers that force CORS preflight; (3) **secure communications**, enforced via HTTPS, Content Security Policy (CSP) with nonces and `strict-dynamic`, and correctly configured CORS with strict origin allowlists; and (4) the **Backend-for-Frontend (BFF) pattern**, in which a thin server layer acts as the OAuth/OIDC client, holds tokens server-side in encrypted `HttpOnly` cookies, and reverse-proxies API calls with the access token attached server-side. The unifying principle is that the browser is a hostile environment: any script running on the page can read `localStorage`, `sessionStorage`, and non-`HttpOnly` cookies, so credentials must be stored where JavaScript cannot reach them, and every state-changing request must be proven to originate from the legitimate application.

**Beginner-Friendly Explanation:** Your React app runs in a browser, and browsers are not safe places to keep secrets. If a hacker manages to run even one line of JavaScript on your page, they can read anything you've stored in `localStorage`—including login tokens. Secure Client Architecture is about designing your app so that even if an attacker gets a script onto the page, they can't steal your users' credentials. You do this by storing tokens in `HttpOnly` cookies (which JavaScript can't read), by adding anti-CSRF protections so other websites can't make requests on your users' behalf, by using HTTPS and CSP to block malicious scripts, and—most robustly—by keeping tokens entirely on a server layer (the BFF pattern) so the browser never sees them at all.

### Key Characteristics

- **Defence in Depth:** No single mechanism is sufficient. XSS mitigation, CSRF defence, CSP, CORS, and the BFF pattern work together. A CSRF token can be defeated by an XSS vulnerability, so XSS must be prevented first. 
- **Storage Determines Blast Radius:** The choice between `localStorage` and `HttpOnly` cookies determines the ceiling of an XSS bug's impact. A token in `localStorage` is unbounded until revoked; a token in an `HttpOnly` cookie is bounded to the browser tab and session lifetime. 
- **Prove Intent, Not Just Identity:** CSRF defences (tokens, custom headers, `SameSite`) exist because the browser automatically attaches cookies to cross-site requests. The server must be able to distinguish a legitimate request from a forged one. 
- **The Browser Is Hostile by Default:** Any script—whether from an XSS bug, a compromised npm dependency, or a third-party tag—can read JavaScript-accessible storage and send data anywhere. 
- **Server-Side Enforcement Is Authoritative:** Client-side controls (CSP, CORS, token storage) are necessary but not sufficient. The BFF pattern and server-side token handling provide the strongest boundary by ensuring tokens never reach the browser at all. 

### Prerequisites

- Solid understanding of React components, Hooks, and the render lifecycle.
- Familiarity with HTTP fundamentals: headers, cookies, status codes, and the Same-Origin Policy.
- Basic knowledge of OAuth 2.0 / OpenID Connect flows (Authorization Code + PKCE).
- Awareness of browser security mechanisms: `SameSite`, `HttpOnly`, `Secure`, and CORS.
- Understanding of server-side rendering (Next.js, Express) for BFF and CSP nonce generation.

### Related Programming Areas

- **Web Security:** XSS, CSRF, clickjacking, and injection attacks.
- **Identity & Access Management:** OAuth 2.0, OIDC, and token lifecycle management.
- **HTTP Protocol:** Cookies, CORS, CSP headers, and Fetch Metadata.
- **Server-Side Rendering:** Next.js middleware, Server Components, and data access layers.
- **Observability:** CSP violation reporting and security event logging.

### Core Concepts / Features

1. XSS (Cross-Site Scripting) Mitigation
2. CSRF (Cross-Site Request Forgery) Defence
3. Secure Communications (HTTPS, CSP, CORS)
4. BFF (Backend-for-Frontend) Pattern

---

## Core Concept 1: XSS (Cross-Site Scripting) Mitigation

### Definitions

**Core Definition:** XSS mitigation is the set of practices that prevent an attacker from injecting and executing malicious scripts in a React application, and that limit the damage if an injection does occur—principally by storing authentication tokens where JavaScript cannot read them.

**Technical Definition:** Cross-Site Scripting (XSS) occurs when an attacker injects malicious content into a webpage that is then executed by the victim's browser in the context of the trusted origin. React provides built-in output escaping: values rendered in JSX curly braces (`{value}`) are automatically escaped, preventing most injection attacks. However, React's auto-escaping stops at deliberate escape hatches: `dangerouslySetInnerHTML`, direct DOM manipulation via `innerHTML`, and `javascript:` or `data:` URLs are all sinks that re-open the vulnerability.  The storage of authentication tokens is the second critical dimension: `localStorage`, `sessionStorage`, and any cookie without the `HttpOnly` flag are fully readable by JavaScript. Because XSS is JavaScript execution in your origin, a single injected script can call `localStorage.getItem('token')` and POST it to an attacker's server in one line.  OWASP's guidance is unambiguous: session tokens should not be stored anywhere JavaScript can reach.  The preferred pattern for SPAs is to store the refresh token in an `HttpOnly`, `Secure`, `SameSite` cookie and keep the short-lived access token in memory only. 

**Beginner-Friendly Explanation:** XSS is when a hacker tricks your app into running their JavaScript code. React protects you from most of this automatically: when you put `{userInput}` in your JSX, React escapes it so it can't be executed as code. But if you use `dangerouslySetInnerHTML` or manipulate the DOM directly, you're bypassing React's protection. More importantly, if you store your login token in `localStorage`, any hacker who gets a script running on your page can read it and steal it. The fix is to store tokens in `HttpOnly` cookies, which JavaScript can't read, or—even better—to keep tokens on the server using the BFF pattern.

### Purposes

- To prevent attackers from injecting and executing malicious scripts through user-controlled data.
- To ensure React's auto-escaping is not bypassed by dangerous DOM sinks.
- To store authentication tokens where XSS cannot exfiltrate them.
- To limit the blast radius of an XSS vulnerability if one occurs.
- To sanitise any HTML that must be rendered dynamically (e.g., rich text from a CMS).
- To deploy CSP as a defence-in-depth layer that blocks inline script execution.

### Syntax Rules and Structure

**General Syntax for Safe Rendering (React Auto-Escaping):**
```jsx
function UserComment({ comment }) {
  // ✅ SAFE: React auto-escapes the value
  return <p>{comment}</p>;
}
```

**Component Breakdown:**
- `{comment}`: React escapes the string, so `<script>alert('XSS')</script>` renders as text, not executable code.
- This is the default and safest way to render user-controlled data.

**General Syntax for Sanitised HTML (When You Must Render HTML):**
```jsx
import DOMPurify from 'dompurify';

function RichText({ html }) {
  // Sanitise the HTML before rendering
  const sanitizedHtml = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href'],
  });

  return <div dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />;
}
```

**Component Breakdown:**
- `DOMPurify.sanitize(html)`: Removes dangerous tags and attributes.
- `ALLOWED_TAGS` / `ALLOWED_ATTR`: Restrict what can be rendered.
- `dangerouslySetInnerHTML={{ __html: sanitizedHtml }}`: Only used with sanitised content.

**General Syntax for Token Storage (In-Memory + HttpOnly Cookie):**
```javascript
// authStore.js — access token in memory only
import { create } from 'zustand';

export const useAuthStore = create((set) => ({
  accessToken: null, // In-memory only; NOT persisted
  user: null,
  setAuth: (accessToken, user) => set({ accessToken, user }),
  clearAuth: () => set({ accessToken: null, user: null }),
}));

// Server sets HttpOnly refresh cookie
res.cookie('refresh_token', refreshToken, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000,
  path: '/api/auth/refresh',
});
```

**Component Breakdown:**
- `accessToken: null`: Stored in memory (Zustand store), never written to `localStorage`.
- `httpOnly: true`: The cookie is invisible to JavaScript.
- `secure: true`: Only sent over HTTPS.
- `sameSite: 'strict'`: Not sent on cross-site requests.
- `path: '/api/auth/refresh'`: Restricts the cookie to the refresh endpoint only.

**Syntax Rules:**
- Rely on React's auto-escaping for all text content rendered via `{}`.
- Never use `dangerouslySetInnerHTML` with unsanitised content.
- Sanitise HTML with DOMPurify or a similar library before rendering.
- Never store authentication tokens in `localStorage` or `sessionStorage`.
- Store the refresh token in an `HttpOnly`, `Secure`, `SameSite=Strict` cookie.
- Store the access token in memory only (a JavaScript variable or state store).
- Validate and sanitise URLs: reject `javascript:` and `data:` URLs. 

**Constraints and Limitations:**
- React's auto-escaping does not protect against `dangerouslySetInnerHTML` or direct DOM manipulation.
- `HttpOnly` cookies prevent token theft but do not prevent in-session abuse: an XSS payload can still fire authenticated `fetch()` calls that ride along on the victim's session. 
- `localStorage` is synchronous and has a size limit, but the primary concern is security, not capacity.
- Sanitising HTML adds bundle size and runtime cost; consider whether rich HTML rendering is truly necessary.
- CSP is a defence-in-depth layer, not a substitute for proper escaping and sanitisation.

### Annotated Code Examples

**Example 1: Safe Rendering vs. Dangerous Rendering**

```jsx
import React from 'react';
import DOMPurify from 'dompurify';

function CommentSection({ comments }) {
  return (
    <div>
      <h2>Comments</h2>
      {comments.map((comment) => (
        <div key={comment.id} style={{ border: '1px solid #ccc', padding: '8px', marginBottom: '8px' }}>
          {/* ✅ SAFE: React auto-escapes */}
          <p>{comment.text}</p>

          {/* ❌ DANGEROUS: Never do this with user input */}
          {/* <div dangerouslySetInnerHTML={{ __html: comment.text }} /> */}

          {/* ✅ SAFE: Sanitised HTML if rich text is required */}
          {comment.richText && (
            <div
              dangerouslySetInnerHTML={{
                __html: DOMPurify.sanitize(comment.richText, {
                  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
                  ALLOWED_ATTR: ['href'],
                }),
              }}
            />
          )}
        </div>
      ))}
    </div>
  );
}

export default CommentSection;
```

**Expected Output:** A user-submitted comment containing `<script>alert('XSS')</script>` renders as the literal text `<script>alert('XSS')</script>`, not as an executable script. Rich text with allowed tags (bold, italic, links) renders correctly, while disallowed tags (scripts, event handlers) are stripped.

**Why This Output Occurs:** React's JSX auto-escaping converts `<` to `&lt;` and `>` to `&gt;`, so the browser displays the text rather than executing it. For rich text, DOMPurify removes all tags and attributes not in the allowlist, including `<script>`, `onclick`, and `javascript:` URLs. The combination of auto-escaping for plain text and sanitisation for rich text provides comprehensive XSS protection.

**Example 2: Token Storage Comparison**

```jsx
// ❌ INSECURE: Token in localStorage — vulnerable to XSS
function insecureLogin(token) {
  localStorage.setItem('accessToken', token);
  // Any XSS payload can do: localStorage.getItem('accessToken')
}
```
```jsx
// ✅ SECURE: Access token in memory, refresh token in HttpOnly cookie
import { create } from 'zustand';

const useAuthStore = create((set) => ({
  accessToken: null, // Memory only — not accessible after page reload
  setAccessToken: (token) => set({ accessToken: token }),
  clearAccessToken: () => set({ accessToken: null }),
}));

// Server sets the refresh token as an HttpOnly cookie
app.post('/api/auth/login', (req, res) => {
  const { accessToken, refreshToken } = authenticate(req.body);

  res.cookie('refresh_token', refreshToken, {
    httpOnly: true,     // JavaScript cannot read this
    secure: true,       // HTTPS only
    sameSite: 'strict', // Not sent on cross-site requests
    path: '/api/auth/refresh',
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });

  // Access token returned in JSON — client stores it in memory
  res.json({ accessToken });
});
```

**Expected Output:** After login, the access token is stored in a Zustand store (memory only) and is lost on page reload. The refresh token is stored in an `HttpOnly` cookie. An XSS payload attempting `localStorage.getItem('accessToken')` returns `null` because the token was never written to `localStorage`. Attempting `document.cookie` does not reveal the refresh token because of the `HttpOnly` flag.

**Why This Output Occurs:** The access token in memory is not persisted to any JavaScript-accessible storage, so it cannot be exfiltrated by a script. The refresh token in the `HttpOnly` cookie is invisible to `document.cookie` and any JavaScript read attempt. The `path: '/api/auth/refresh'` restriction ensures the refresh cookie is only sent to the refresh endpoint, minimising exposure. This is the pattern recommended by OWASP, Auth0, and the IETF OAuth Security Best Current Practice. 

### Real-World Cases

- **Content platforms:** Rendering user comments with auto-escaping and sanitising rich text from CMS editors.
- **Social media:** Preventing profile-bio XSS by escaping all user-controlled fields.
- **E-commerce:** Storing session tokens in `HttpOnly` cookies to prevent theft if a third-party widget is compromised.
- **Financial applications:** Using in-memory access tokens with `HttpOnly` refresh cookies and strict CSP to protect against token theft.
- **Enterprise SaaS:** Deploying CSP with nonces to block inline script injection, even if a template escaping bug is introduced.

---

## Core Concept 2: CSRF (Cross-Site Request Forgery) Defence

### Definitions

**Core Definition:** CSRF defence is the set of mechanisms that ensure state-changing requests (POST, PUT, DELETE) originate from the legitimate React application and not from a malicious third-party site that exploits the browser's automatic cookie attachment.

**Technical Definition:** A CSRF attack occurs when a malicious website tricks an authenticated user's browser into performing an unwanted action on a trusted site. Because browsers automatically include cookies (including session cookies) with every request to the target origin, the server cannot distinguish between a legitimate request and a forged one unless an additional proof of intent is required.  OWASP recommends three complementary defences: (1) **`SameSite` cookie attributes**—`SameSite=Lax` or `SameSite=Strict` prevents the browser from sending cookies on cross-site requests, which is sufficient for most modern browsers;  (2) **Synchronizer token pattern**—a per-session or per-request token is issued to the client and must be sent in a custom header (e.g., `X-CSRF-Token`) that the server validates against its stored copy;  and (3) **Fetch Metadata headers** (`Sec-Fetch-Site`, `Sec-Fetch-Mode`) for modern browsers.  For stateless APIs, the **double-submit cookie** pattern is used: the token is stored in a cookie and also sent in a header, and the server checks that they match.  Critically, XSS defeats all CSRF mitigations, so XSS prevention must be in place first. 

**Beginner-Friendly Explanation:** CSRF is when a bad website tricks your browser into making a request to your bank (or your app) using your logged-in session. Because your browser automatically sends your session cookie with the request, the server thinks it's you. CSRF defences make the request include something extra—like a secret token or a special header—that a malicious site can't provide. The easiest defence is to set your session cookie to `SameSite=Strict`, which tells the browser "never send this cookie on requests from other websites." For extra security, you can also use CSRF tokens that the server checks on every state-changing request.

### Purposes

- To prevent malicious sites from forging state-changing requests using the victim's authenticated session.
- To prove that a request originated from the legitimate React application.
- To complement cookie-based authentication without relying solely on the browser's Same-Origin Policy.
- To protect sensitive operations: password changes, fund transfers, account modifications.
- To provide defence in depth alongside `SameSite` cookie attributes.
- To support both stateful (synchronizer token) and stateless (double-submit) architectures.

### Syntax Rules and Structure

**General Syntax for Server-Side CSRF Token Issuance (Express):**
```javascript
import express from 'express';
import crypto from 'crypto';

const app = express();
app.use(express.json());

// In-memory store (use Redis/DB in production)
const csrfTokens = new Map();

// Issue CSRF token
app.get('/api/csrf-token', (req, res) => {
  const sessionId = req.cookies.session_id || crypto.randomUUID();
  const token = crypto.randomBytes(32).toString('hex');
  csrfTokens.set(sessionId, token);

  // Set session cookie (HttpOnly, Secure, SameSite)
  res.cookie('session_id', sessionId, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    path: '/',
  });

  res.json({ csrfToken: token });
});

// Validate CSRF token on state-changing requests
app.post('/api/transfer', (req, res) => {
  const sessionId = req.cookies.session_id;
  const headerToken = req.headers['x-csrf-token'];

  if (!sessionId || csrfTokens.get(sessionId) !== headerToken) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }

  // Process the transfer
  res.json({ ok: true });
});
```

**Component Breakdown:**
- `crypto.randomBytes(32)`: Generates a cryptographically strong token.
- `csrfTokens.set(sessionId, token)`: Stores the token server-side, associated with the session.
- `res.cookie('session_id', ...)`: Sets the session cookie with `HttpOnly`, `Secure`, and `SameSite=Strict`.
- `req.headers['x-csrf-token']`: Reads the CSRF token from the custom header.
- `csrfTokens.get(sessionId) !== headerToken`: Validates that the header token matches the stored token.

**General Syntax for React Client (Axios Interceptor):**
```javascript
import axios from 'axios';

const api = axios.create({ withCredentials: true });

let csrfToken = null;

// Fetch CSRF token once on app initialisation
export async function initCsrf() {
  const { data } = await api.get('/api/csrf-token');
  csrfToken = data.csrfToken;
}

// Attach CSRF token to every state-changing request
api.interceptors.request.use((config) => {
  if (config.method !== 'get' && csrfToken) {
    config.headers['X-CSRF-Token'] = csrfToken;
  }
  return config;
});

export default api;
```

**Component Breakdown:**
- `withCredentials: true`: Ensures cookies are sent with cross-origin requests.
- `initCsrf()`: Fetches the CSRF token once and stores it in memory.
- `api.interceptors.request.use(...)`: Attaches the CSRF token to all non-GET requests.
- `config.headers['X-CSRF-Token']`: The custom header that the server validates.

**Syntax Rules:**
- Set `SameSite=Lax` or `SameSite=Strict` on all session cookies. 
- Use a custom header (e.g., `X-CSRF-Token`) that is not a CORS-safelisted header; this forces a preflight request for cross-origin attempts. 
- For stateless APIs, use the double-submit cookie pattern: store the token in a cookie and send it in a header, then validate that they match. 
- Always validate CSRF tokens on the server; never rely solely on client-side checks.
- Use `Secure` and `HttpOnly` on all cookies that carry session or CSRF tokens.
- For modern browsers, consider Fetch Metadata headers (`Sec-Fetch-Site`) as an additional layer. 

**Constraints and Limitations:**
- XSS defeats all CSRF mitigations: if an attacker can run JavaScript on your origin, they can read the CSRF token. XSS prevention must come first. 
- `SameSite=Lax` allows top-level GET navigations; `SameSite=Strict` may break legitimate cross-site navigation flows (e.g., OAuth redirects).
- The synchronizer token pattern requires server-side storage, which adds state.
- The double-submit cookie pattern is vulnerable if the attacker can set cookies via a subdomain (cookie tossing).
- CORS preflight adds latency to cross-origin requests; for same-origin requests (BFF pattern), this overhead is eliminated.

### Annotated Code Examples

**Example 1: Synchronizer Token Pattern (Express + React)**

```javascript
// server.js
import express from 'express';
import cookieParser from 'cookie-parser';
import crypto from 'crypto';

const app = express();
app.use(express.json());
app.use(cookieParser());

const csrfStore = new Map();

app.get('/api/csrf-token', (req, res) => {
  const sessionId = req.cookies.session_id || crypto.randomUUID();
  const csrfToken = crypto.randomBytes(32).toString('hex');
  csrfStore.set(sessionId, csrfToken);

  res.cookie('session_id', sessionId, {
    httpOnly: true,
    secure: false, // true in production
    sameSite: 'strict',
    path: '/',
  });

  res.json({ csrfToken });
});

app.post('/api/transfer', (req, res) => {
  const sessionId = req.cookies.session_id;
  const headerToken = req.headers['x-csrf-token'];

  if (!sessionId || csrfStore.get(sessionId) !== headerToken) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }

  res.json({ ok: true, message: 'Transfer processed' });
});

app.listen(3001, () => console.log('Server on port 3001'));
```

```javascript
// apiClient.js (React)
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3001',
  withCredentials: true,
});

let csrfToken = null;

export async function initCsrf() {
  const { data } = await api.get('/api/csrf-token');
  csrfToken = data.csrfToken;
}

api.interceptors.request.use((config) => {
  if (config.method !== 'get' && csrfToken) {
    config.headers['X-CSRF-Token'] = csrfToken;
  }
  return config;
});

export default api;
```

**Expected Output:** On app initialisation, `initCsrf()` fetches the CSRF token and stores it in memory. When the React app sends a POST to `/api/transfer`, the interceptor attaches the `X-CSRF-Token` header. The server validates that the header token matches the stored token for the session. A request from a malicious site (which cannot read the CSRF token or set the custom header) is rejected with 403.

**Why This Output Occurs:** The CSRF token is stored server-side and associated with the session ID. The React client fetches it once and attaches it to all non-GET requests via an interceptor. A cross-site attacker cannot read the token (it's not in a cookie the attacker can access) and cannot set the custom `X-CSRF-Token` header on a cross-origin request without triggering a CORS preflight that the server rejects. The `SameSite=Strict` cookie attribute provides an additional layer by preventing the session cookie from being sent on cross-site requests altogether.

**Example 2: Double-Submit Cookie Pattern (Stateless)**

```javascript
// server.js — no server-side token store
import express from 'express';
import cookieParser from 'cookie-parser';
import crypto from 'crypto';

const app = express();
app.use(express.json());
app.use(cookieParser());

app.get('/api/csrf-token', (req, res) => {
  const token = crypto.randomBytes(32).toString('hex');

  // Set CSRF token in a non-HttpOnly cookie
  res.cookie('csrf_token', token, {
    httpOnly: false, // Must be readable by JS for double-submit
    secure: false,
    sameSite: 'strict',
    path: '/',
  });

  res.json({ csrfToken: token });
});

app.post('/api/transfer', (req, res) => {
  const cookieToken = req.cookies.csrf_token;
  const headerToken = req.headers['x-csrf-token'];

  if (!cookieToken || !headerToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }

  res.json({ ok: true, message: 'Transfer processed' });
});
```

**Expected Output:** The server issues a CSRF token in a non-`HttpOnly` cookie and also returns it in the JSON response. The React client reads the token from the response and sends it in the `X-CSRF-Token` header. The server checks that the cookie token and the header token match. A cross-site attacker cannot read the cookie (due to `SameSite=Strict`) and cannot set the header without a CORS preflight.

**Why This Output Occurs:** The double-submit pattern requires no server-side storage: the token is stored in a cookie and echoed in a header. The server only checks that the two match. The `SameSite=Strict` attribute prevents the cookie from being sent on cross-site requests, so an attacker on a different origin cannot obtain the token. The custom header forces a CORS preflight, which the server rejects for unauthorized origins.

### Real-World Cases

- **Banking applications:** Using synchronizer tokens with `SameSite=Strict` cookies for fund transfers and password changes.
- **E-commerce:** Protecting cart modifications and checkout requests with CSRF tokens.
- **SaaS applications:** Using the double-submit cookie pattern for stateless APIs with JWT authentication.
- **Social media:** Protecting profile updates and post creation from CSRF attacks via `SameSite` cookies and custom headers.
- **Healthcare:** Using CSRF tokens alongside `SameSite=Strict` for HIPAA-compliant session management.

---

## Core Concept 3: Secure Communications (HTTPS, CSP, CORS)

### Definitions

**Core Definition:** Secure Communications encompasses the transport-layer (HTTPS), content-restriction (CSP), and cross-origin permission (CORS) mechanisms that ensure data is transmitted securely, scripts are restricted to trusted sources, and cross-origin requests are explicitly authorised.

**Technical Definition:** **HTTPS** encrypts all traffic between the browser and the server, preventing man-in-the-middle attacks and ensuring that cookies marked `Secure` are only transmitted over encrypted connections. **Content Security Policy (CSP)** is an HTTP response header that tells the browser which sources of scripts, styles, images, and other resources are allowed to load. For React applications, CSP must account for inline scripts injected by the framework (e.g., hydration scripts in Next.js). The modern approach is nonce-based CSP with `'strict-dynamic'`: a per-request nonce is generated and included in both the CSP header and the `nonce` attribute of legitimate inline scripts.  `'strict-dynamic'` instructs the browser to trust scripts loaded by already-trusted scripts, which is necessary for React's dynamic imports.  **CORS (Cross-Origin Resource Sharing)** is the browser mechanism that controls which origins can read responses from a given server. The browser sends a preflight `OPTIONS` request for non-simple requests (e.g., those with custom headers or non-GET methods), and the server must respond with the appropriate `Access-Control-Allow-*` headers.  Secure CORS configuration uses strict origin allowlists and never combines wildcard origins (`*`) with credentials. 

**Beginner-Friendly Explanation:** HTTPS is like sending a letter in a sealed, tamper-proof envelope instead of a postcard—no one can read or change it in transit. CSP is like giving the browser a guest list of which scripts are allowed to run on your page; if a script isn't on the list, the browser blocks it. CORS is the browser's way of asking "can this website talk to my API?" and the server answering with a specific yes or no. Together, these three mechanisms ensure that your app communicates securely, runs only trusted code, and only accepts requests from authorised origins.

### Purposes

- To encrypt all traffic between the browser and server, preventing eavesdropping and tampering.
- To restrict which scripts can execute on the page, mitigating XSS even if an injection occurs.
- To allow legitimate cross-origin API requests while blocking malicious ones.
- To generate per-request CSP nonces that allow React's inline hydration scripts while blocking injected scripts.
- To configure CORS with strict origin allowlists that prevent unauthorised cross-origin access.
- To provide defence in depth alongside XSS and CSRF mitigations.

### Syntax Rules and Structure

**General Syntax for CSP with Nonce and `strict-dynamic` (Next.js Middleware):**
```javascript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64');

  const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    connect-src 'self' https:;
    font-src 'self' https://fonts.gstatic.com;
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    upgrade-insecure-requests;
  `;

  const response = NextResponse.next();
  response.headers.set('Content-Security-Policy', cspHeader.replace(/\s+/g, ' ').trim());
  response.headers.set('x-nonce', nonce);

  return response;
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

**Component Breakdown:**
- `crypto.randomUUID()`: Generates a unique nonce per request.
- `'nonce-${nonce}'`: Allows the inline script with this nonce to execute.
- `'strict-dynamic'`: Trusts scripts loaded by the nonced script (e.g., dynamic imports).
- `'unsafe-inline'` in `style-src`: Required for Next.js inline styles; CSS injection is not an XSS vector. 
- `object-src 'none'`: Blocks plugins like Flash.
- `upgrade-insecure-requests`: Rewrites HTTP URLs to HTTPS.

**General Syntax for CORS Configuration (Express):**
```javascript
import cors from 'cors';

const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com',
];

app.use(cors({
  origin: (origin, callback) => {
    // Allow requests with no origin (e.g., same-origin, curl)
    if (!origin) return callback(null, true);
    if (allowedOrigins.includes(origin)) {
      return callback(null, true);
    }
    callback(new Error('Not allowed by CORS'));
  },
  credentials: true, // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token'],
  maxAge: 86400, // Cache preflight for 24 hours
}));
```

**Component Breakdown:**
- `origin`: A function that validates the request origin against an allowlist.
- `credentials: true`: Allows cookies to be sent with cross-origin requests.
- `methods`: The HTTP methods allowed for cross-origin requests.
- `allowedHeaders`: The headers the client is allowed to send.
- `maxAge`: How long the browser should cache the preflight response.

**General Syntax for CSP Nonce Injection in React (Server-Side):**
```jsx
// For React with renderToPipeableStream
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App nonce={nonce} />, {
  nonce: nonce,
  bootstrapScripts: [{ src: '/main.js', nonce: nonce }],
});
```

**Component Breakdown:**
- `nonce={nonce}`: Passes the nonce to the React tree.
- `bootstrapScripts: [{ src: '/main.js', nonce: nonce }]`: Adds the nonce to the client bundle script tag.
- `renderToPipeableStream` accepts a `nonce` option that applies to all scripts it generates. 

**Syntax Rules:**
- Always enforce HTTPS in production; use `Strict-Transport-Security` (HSTS) headers.
- Use nonce-based CSP with `'strict-dynamic'` for React applications that inject inline scripts.
- `style-src` may require `'unsafe-inline'` for CSS-in-JS libraries; this is acceptable because CSS injection is not an XSS vector. 
- Set `object-src 'none'` and `base-uri 'self'` to block plugin-based and base-tag attacks.
- Configure CORS with a strict origin allowlist; never use `*` with `credentials: true`.
- Include `X-CSRF-Token` in `allowedHeaders` if using CSRF tokens.
- Use `maxAge` to reduce preflight overhead for legitimate cross-origin requests.

**Constraints and Limitations:**
- Nonce-based CSP requires dynamic rendering; static pages cannot use per-request nonces. 
- `'strict-dynamic'` disables host-based allowlisting in CSP3 browsers; only nonces matter. 
- CSP violation reports can be noisy; filter and monitor them.
- CORS preflight adds latency to cross-origin requests; same-origin requests (BFF pattern) avoid this.
- Wildcard CORS with credentials is blocked by browsers; a specific origin must be returned.
- Some third-party scripts may require additional CSP directives; audit and update the policy.

### Annotated Code Examples

**Example 1: Nonce-Based CSP for a React SPA (Next.js Middleware)**

```javascript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

function generateNonce(): string {
  const array = new Uint8Array(16);
  crypto.getRandomValues(array);
  return Array.from(array, (byte) => byte.toString(16).padStart(2, '0')).join('');
}

export function middleware(request: NextRequest) {
  const nonce = generateNonce();

  const csp = [
    "default-src 'self'",
    `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`,
    "style-src 'self' 'unsafe-inline'",
    "img-src 'self' data: https:",
    "connect-src 'self' https:",
    "font-src 'self' https://fonts.gstatic.com",
    "object-src 'none'",
    "base-uri 'self'",
    "form-action 'self'",
    "upgrade-insecure-requests",
  ].join('; ');

  const response = NextResponse.next();
  response.headers.set('Content-Security-Policy', csp);
  response.headers.set('x-nonce', nonce);

  return response;
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

**Expected Output:** Every page response includes a `Content-Security-Policy` header with a unique nonce. React's inline hydration scripts (which include the same nonce) execute normally. An injected `<script>` tag without the nonce is blocked by the browser, and a CSP violation is reported to the console.

**Why This Output Occurs:** The middleware generates a fresh nonce for each request and injects it into both the CSP header and the response headers (so Next.js can apply it to script tags). The browser only executes inline scripts whose `nonce` attribute matches the value in the CSP header. `'strict-dynamic'` allows those trusted scripts to load additional scripts (e.g., dynamic imports), which is essential for React's code-splitting. Injected scripts lack the nonce and are blocked.

**Example 2: Secure CORS Configuration**

```javascript
import express from 'express';
import cors from 'cors';

const app = express();

const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com',
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin) return callback(null, true);
    if (allowedOrigins.includes(origin)) {
      return callback(null, true);
    }
    callback(new Error(`Origin ${origin} not allowed by CORS`));
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token'],
  maxAge: 86400,
}));

app.post('/api/data', (req, res) => {
  res.json({ message: 'Data received' });
});

app.listen(3001);
```

**Expected Output:** A request from `https://app.example.com` is allowed and receives the response. A request from `https://evil.com` is rejected by the browser with a CORS error. The preflight `OPTIONS` request returns the appropriate `Access-Control-Allow-*` headers, and the browser caches the preflight for 24 hours.

**Why This Output Occurs:** The `origin` function validates the request origin against an explicit allowlist. `credentials: true` allows cookies to be sent, which requires a specific origin (not `*`). The `allowedHeaders` list includes `X-CSRF-Token`, so the browser permits that header in cross-origin requests. The `maxAge` directive reduces preflight overhead by caching the permission for 24 hours.

### Real-World Cases

- **Next.js applications:** Using middleware-generated nonces for CSP with `'strict-dynamic'` to allow React hydration while blocking injected scripts.
- **Microservices architectures:** Configuring CORS with strict origin allowlists for each service.
- **Third-party integrations:** Allowing specific partner origins to access APIs while blocking all others.
- **Financial applications:** Deploying HSTS with a long `max-age` to enforce HTTPS.
- **Content platforms:** Using CSP to restrict image and font sources to trusted CDNs.

---

## Core Concept 4: BFF (Backend-for-Frontend) Pattern

### Definitions

**Core Definition:** The Backend-for-Frontend (BFF) pattern is an architectural approach in which a thin server layer sits between the React SPA and the backend APIs, acting as the OAuth/OIDC client and holding all tokens server-side so the browser never sees them.

**Technical Definition:** In the BFF pattern, a lightweight server component (the BFF) performs the OAuth 2.0 Authorization Code flow with PKCE, exchanges the authorization code for tokens, and stores the tokens in an encrypted, `HttpOnly`, `SameSite=Strict` cookie that the SPA cannot read. The SPA makes API requests to the BFF, which reverse-proxies them to the resource server with the access token attached server-side.  The BFF is a confidential client (it can keep a client secret) and is defined as the recommended architecture for browser-based applications by the IETF in OAuth 2.0 for Browser-Based Applications, section 6.1.  The BFF pattern solves the fundamental problem that SPAs are public clients: they cannot securely store client secrets, and any token they hold is accessible to JavaScript running on the page. By moving token handling to the server, the browser holds only a session cookie, achieving a security level on par with a traditional OAuth-secured website.  The pattern also eliminates the need for CSRF tokens in some implementations, because the session cookie can be set with `SameSite=Strict` and the BFF can validate `Origin` and `Sec-Fetch-Site` headers. 

**Beginner-Friendly Explanation:** Normally, when your React app logs a user in, it gets a token and stores it in the browser. But browsers are unsafe—a hacker who runs a script on your page can steal that token. The BFF pattern solves this by adding a small server layer between your React app and your APIs. This server handles the login, keeps the tokens safely on the server, and only gives the browser a session cookie. When your React app needs data, it asks the BFF, which adds the token server-side and forwards the request. Your React app never sees a token, so there's nothing for a hacker to steal.

### Purposes

- To eliminate token exposure to the browser by keeping all tokens server-side.
- To act as a confidential OAuth/OIDC client that can securely hold a client secret.
- To provide a session-cookie-based authentication model for SPAs that is as secure as traditional server-rendered applications.
- To reverse-proxy API requests with the access token attached server-side, shielding the SPA from raw tokens.
- To handle token refresh transparently on the server without involving the browser.
- To simplify SPA code by removing token management responsibilities from the client.

### Syntax Rules and Structure

**General Syntax for BFF OAuth Flow (Node.js/Express):**
```javascript
// BFF server (Express)
import express from 'express';
import crypto from 'crypto';

const app = express();

// Step 1: Initiate login — redirect to IdP
app.get('/bff/login', (req, res) => {
  const state = crypto.randomUUID();
  const codeVerifier = crypto.randomBytes(32).toString('base64url');
  const codeChallenge = crypto
    .createHash('sha256')
    .update(codeVerifier)
    .digest('base64url');

  // Store PKCE verifier and state in HttpOnly cookies
  res.cookie('pkce_verifier', codeVerifier, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 10 * 60 * 1000,
  });
  res.cookie('oauth_state', state, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 10 * 60 * 1000,
  });

  const authUrl = new URL('https://idp.example.com/authorize');
  authUrl.searchParams.set('response_type', 'code');
  authUrl.searchParams.set('client_id', process.env.CLIENT_ID);
  authUrl.searchParams.set('redirect_uri', 'https://api.example.com/bff/callback');
  authUrl.searchParams.set('scope', 'openid profile email offline_access');
  authUrl.searchParams.set('state', state);
  authUrl.searchParams.set('code_challenge', codeChallenge);
  authUrl.searchParams.set('code_challenge_method', 'S256');

  res.redirect(authUrl.toString());
});

// Step 2: Handle callback — exchange code for tokens
app.get('/bff/callback', async (req, res) => {
  const { code, state } = req.query;
  const storedState = req.cookies.oauth_state;
  const codeVerifier = req.cookies.pkce_verifier;

  if (state !== storedState) {
    return res.status(400).send('Invalid state');
  }

  const tokenResponse = await fetch('https://idp.example.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'https://api.example.com/bff/callback',
      client_id: process.env.CLIENT_ID,
      client_secret: process.env.CLIENT_SECRET, // Confidential client
      code_verifier: codeVerifier,
    }),
  });

  const tokens = await tokenResponse.json();

  // Store tokens in encrypted HttpOnly cookie
  res.cookie('session', encrypt(JSON.stringify(tokens)), {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000,
    path: '/',
  });

  res.redirect('/dashboard');
});

// Step 3: Proxy API requests with token attached server-side
app.use('/bff/api', async (req, res) => {
  const session = decrypt(req.cookies.session);
  if (!session?.access_token) {
    return res.status(401).json({ error: 'Not authenticated' });
  }

  const response = await fetch(`https://api.example.com${req.path}`, {
    method: req.method,
    headers: {
      Authorization: `Bearer ${session.access_token}`,
      'Content-Type': 'application/json',
    },
    body: req.method !== 'GET' ? JSON.stringify(req.body) : undefined,
  });

  const data = await response.json();
  res.status(response.status).json(data);
});

// Step 4: Get current user
app.get('/bff/user', (req, res) => {
  const session = decrypt(req.cookies.session);
  if (!session?.user) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  res.json({ user: session.user });
});
```

**Component Breakdown:**
- `/bff/login`: Generates PKCE verifier, stores it in an `HttpOnly` cookie, and redirects to the IdP.
- `/bff/callback`: Exchanges the authorization code for tokens using the client secret (confidential client), then stores the tokens in an encrypted `HttpOnly` cookie.
- `/bff/api/*`: Reverse-proxies API requests, attaching the access token from the server-side session.
- `/bff/user`: Returns the current user's session claims.
- `client_secret`: Never exposed to the browser; stored in server environment variables.

**General Syntax for React Client (BFF-Aware):**
```jsx
// authClient.js
const BFF_BASE = 'https://api.example.com/bff';

export async function login() {
  window.location.href = `${BFF_BASE}/login`;
}

export async function getUser() {
  const res = await fetch(`${BFF_BASE}/user`, {
    credentials: 'include',
  });
  if (!res.ok) return null;
  return res.json();
}

export async function apiFetch(path, options = {}) {
  const res = await fetch(`${BFF_BASE}/api${path}`, {
    ...options,
    credentials: 'include',
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}
```

**Component Breakdown:**
- `credentials: 'include'`: Sends the `HttpOnly` session cookie with every request.
- `login()`: Redirects the browser to the BFF login endpoint.
- `apiFetch()`: Calls the BFF proxy, which attaches the access token server-side.
- The React app never sees or stores a token.

**Syntax Rules:**
- The BFF must be a confidential client: it holds the client secret and never exposes it to the browser. 
- Tokens must be stored in an encrypted, `HttpOnly`, `SameSite=Strict` cookie. 
- The SPA should only receive a session cookie, never a raw access or refresh token.
- The BFF must run on the same parent domain as the SPA for cookies to be first-party and reliable. 
- Use `Origin` and `Sec-Fetch-Site` header validation on state-changing requests as a CSRF defence. 
- Implement token refresh on the server side; the browser should never handle refresh tokens.
- For Next.js, use libraries like NextAuth or Duende BFF that implement the pattern out of the box. 

**Constraints and Limitations:**
- The BFF adds a server layer, increasing infrastructure complexity and maintenance.
- The BFF must be hosted on the same parent domain as the SPA for cookies to work reliably.
- The BFF becomes a single point of failure; it must be deployed with high availability.
- For organisations with limited resources, the BFF may require more effort than alternative patterns (e.g., in-memory access token + `HttpOnly` refresh cookie). 
- The BFF pattern does not eliminate the need for CSP and XSS prevention; it reduces the impact of XSS but does not prevent it.
- Token forwarding adds latency to each API call; consider connection pooling and caching.

### Annotated Code Examples

**Example 1: Complete BFF Login and API Proxy (Express)**

```javascript
// bff-server.js
import express from 'express';
import cookieParser from 'cookie-parser';
import crypto from 'crypto';

const app = express();
app.use(express.json());
app.use(cookieParser());

const ENCRYPTION_KEY = crypto.randomBytes(32); // Use env var in production
const IV_LENGTH = 16;

function encrypt(text) {
  const iv = crypto.randomBytes(IV_LENGTH);
  const cipher = crypto.createCipheriv('aes-256-gcm', ENCRYPTION_KEY, iv);
  const encrypted = Buffer.concat([cipher.update(text, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  return Buffer.concat([iv, tag, encrypted]).toString('base64');
}

function decrypt(encryptedBase64) {
  try {
    const data = Buffer.from(encryptedBase64, 'base64');
    const iv = data.subarray(0, IV_LENGTH);
    const tag = data.subarray(IV_LENGTH, IV_LENGTH + 16);
    const encrypted = data.subarray(IV_LENGTH + 16);
    const decipher = crypto.createDecipheriv('aes-256-gcm', ENCRYPTION_KEY, iv);
    decipher.setAuthTag(tag);
    return decipher.update(encrypted) + decipher.final('utf8');
  } catch {
    return null;
  }
}

// Login
app.get('/bff/login', (req, res) => {
  const state = crypto.randomUUID();
  const verifier = crypto.randomBytes(32).toString('base64url');
  const challenge = crypto.createHash('sha256').update(verifier).digest('base64url');

  res.cookie('pkce_verifier', verifier, { httpOnly: true, secure: true, sameSite: 'strict' });
  res.cookie('oauth_state', state, { httpOnly: true, secure: true, sameSite: 'strict' });

  const url = new URL('https://idp.example.com/authorize');
  url.searchParams.set('response_type', 'code');
  url.searchParams.set('client_id', process.env.CLIENT_ID);
  url.searchParams.set('redirect_uri', 'https://api.example.com/bff/callback');
  url.searchParams.set('scope', 'openid profile email offline_access');
  url.searchParams.set('state', state);
  url.searchParams.set('code_challenge', challenge);
  url.searchParams.set('code_challenge_method', 'S256');

  res.redirect(url.toString());
});

// Callback
app.get('/bff/callback', async (req, res) => {
  const { code, state } = req.query;
  if (state !== req.cookies.oauth_state) {
    return res.status(400).send('Invalid state');
  }

  const tokenRes = await fetch('https://idp.example.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'https://api.example.com/bff/callback',
      client_id: process.env.CLIENT_ID,
      client_secret: process.env.CLIENT_SECRET,
      code_verifier: req.cookies.pkce_verifier,
    }),
  });

  const tokens = await tokenRes.json();

  // Store encrypted session cookie
  res.cookie('session', encrypt(JSON.stringify(tokens)), {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000,
    path: '/',
  });

  res.redirect('/');
});

// User endpoint
app.get('/bff/user', (req, res) => {
  const session = decrypt(req.cookies.session);
  if (!session?.id_token) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  // In production, verify the ID token and extract user claims
  res.json({ user: { name: 'Alice', email: 'alice@example.com' } });
});

// API proxy
app.use('/bff/api', async (req, res) => {
  const session = decrypt(req.cookies.session);
  if (!session?.access_token) {
    return res.status(401).json({ error: 'Not authenticated' });
  }

  const response = await fetch(`https://api.example.com${req.path}`, {
    method: req.method,
    headers: {
      Authorization: `Bearer ${session.access_token}`,
      'Content-Type': 'application/json',
    },
    body: req.method !== 'GET' ? JSON.stringify(req.body) : undefined,
  });

  const data = await response.json();
  res.status(response.status).json(data);
});

app.listen(3002, () => console.log('BFF on port 3002'));
```

**Expected Output:** The React SPA redirects to `/bff/login`, which redirects to the IdP. After authentication, the IdP redirects back to `/bff/callback`, which exchanges the code for tokens and stores them in an encrypted `HttpOnly` cookie. The SPA calls `/bff/user` to get the current user and `/bff/api/*` to access protected APIs. The browser never sees an access or refresh token.

**Why This Output Occurs:** The BFF acts as a confidential OAuth client, performing the Authorization Code flow with PKCE and a client secret. Tokens are encrypted with AES-256-GCM and stored in an `HttpOnly` cookie. The SPA sends the session cookie with every request (via `credentials: 'include'`), and the BFF decrypts the session, extracts the access token, and attaches it to the upstream API call. The `sameSite: 'strict'` attribute prevents the session cookie from being sent on cross-site requests, providing CSRF protection. 

**Example 2: React SPA with BFF Session**

```jsx
import React, { useState, useEffect, createContext, useContext } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchUser() {
      try {
        const res = await fetch('/bff/user', { credentials: 'include' });
        if (res.ok) {
          const data = await res.json();
          setUser(data.user);
        }
      } catch {
        // Not authenticated
      } finally {
        setLoading(false);
      }
    }
    fetchUser();
  }, []);

  function login() {
    window.location.href = '/bff/login';
  }

  function logout() {
    window.location.href = '/bff/logout';
  }

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}

// Protected API call
export async function fetchData(path) {
  const res = await fetch(`/bff/api${path}`, {
    credentials: 'include',
    headers: { 'Content-Type': 'application/json' },
  });
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}
```

**Expected Output:** The app renders a loading state while fetching the user session. If authenticated, the user's name is displayed. If not, a "Log in" button redirects to the BFF login endpoint. API calls go through `/bff/api`, and the browser never handles a token.

**Why This Output Occurs:** The `AuthProvider` fetches the user session from the BFF on mount. The `credentials: 'include'` option ensures the session cookie is sent. The `login()` function redirects the browser to the BFF login endpoint, which handles the OAuth flow. All API calls go through the BFF proxy, which attaches the access token server-side. The SPA code contains no token management logic whatsoever.

### Real-World Cases

- **Next.js applications:** Using NextAuth or Duende BFF to implement the pattern with minimal custom code.
- **Enterprise SaaS:** Using a BFF to integrate with corporate IdPs (Okta, Entra ID) without exposing tokens to the browser.
- **Financial services:** Using a BFF with encrypted `HttpOnly` cookies to meet regulatory requirements for token handling.
- **Healthcare:** Using a BFF to keep PHI-protected API tokens server-side.
- **Multi-tenant platforms:** Using a single BFF to serve multiple SPAs, with tenant-specific token handling.

---

## References

- Cross Site Scripting Prevention Cheat Sheet – OWASP: https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- Cross-Site Request Forgery Prevention Cheat Sheet – OWASP: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- Secure Token Storage for SPAs: Avoiding XSS Theft – Safeguard: https://safeguard.sh/resources/blog/single-page-application-token-storage-security
- CSRF Protection in React: Tokens, Cookies & Best Practices – DEV Community: https://dev.to/pentest_testing_corp/csrf-protection-in-react-tokens-cookies-best-practices-45d2
- Securing a React SPA with the BFF Pattern and Abblix OIDC Server – Abblix: https://www.abblix.com/ru/docs/react-spa-bff-guide
- Token Handler Design Overview – Curity: https://curity.io/resources/learn/token-handler-overview/
- Token Handler Installation – Curity: https://curity.io/resources/learn/token-handler-getting-started/
- Nonce-based CSP migration in Next.js 15 middleware – GitHub: https://raw.githubusercontent.com/jikig-ai/soleur/196ddea35a7772224a6266371e718594fa0e49ad/knowledge-base/project/learnings/2026-03-20-nonce-based-csp-nextjs-middleware.md
- Master CORS: Fix Errors Fast & Securely – Strapi: https://strapi.io/blog/what-is-cors-configuration-guide
- Backend For Frontend (BFF) Samples – Duende Software: https://docs.duendesoftware.com/bff/samples/
- Should I use the BFF pattern or store access token in memory – Stack Overflow: https://stackoverflow.com/questions/79830891/should-i-use-the-bff-pattern-or-store-access-token-in-memory-and-a-refresh-token
- How to ensure React security for cookie – GitHub Discussion: https://github.com/orgs/community/discussions/160281
- React Security Best Practices Checklist (2026) – Safeguard: https://safeguard.sh/resources/blog/react-security-best-practices-checklist
- OAuth 2.0 for Browser-Based Applications – IETF: https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps
- MDN Web Docs – Content Security Policy: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- MDN Web Docs – Cross-Origin Resource Sharing (CORS): https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- MDN Web Docs – Set-Cookie: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- web.dev – SameSite Cookies Explained: https://web.dev/articles/samesite-cookies-explained
- web.dev – Content Security Policy: https://web.dev/articles/csp
- Next.js – Content Security Policy: https://nextjs.org/docs/app/guides/content-security-policy
- DOMPurify – GitHub: https://github.com/cure53/DOMPurify
- Duende BFF – Documentation: https://docs.duendesoftware.com/bff/