# Common Web Vulnerabilities — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Common web vulnerabilities are the recurring classes of security flaws in web applications that allow attackers to steal data, execute code, escalate privileges, or disrupt services — catalogued primarily by the OWASP Top 10 and CWE/SANS Top 25.

**Technical Definition:** Web vulnerabilities arise when untrusted input reaches a sensitive sink without adequate validation, sanitisation, or encoding. The eight most critical classes are: SQL Injection (CWE-89), Cross-Site Scripting (CWE-79), Cross-Site Request Forgery (CWE-352), Server-Side Request Forgery (CWE-918), Path Traversal (CWE-22), Broken Access Control (CWE-284), Insecure Deserialization (CWE-502), and XML External Entity Injection (CWE-611). Each vulnerability class has well-established prevention strategies rooted in input validation, output encoding, parameterisation, allow-listing, and least privilege.

**Beginner-Friendly Explanation:** Imagine your web application is a house. **SQL injection** is like a guest slipping a note to your butler that says "also open the safe" — and the butler obeys. **XSS** is like a guest writing a fake note on your fridge that convinces other guests to open the front door. **CSRF** is like a stranger calling your bank pretending to be you, and the bank believes them because the call came from your phone. **SSRF** is like asking your butler to fetch a package from a room you're not allowed to enter. **Path traversal** is like asking for "the file next door" and the butler walks out of the building. **Broken access control** is like a hotel where every room key opens every door. **Insecure deserialization** is like receiving a package that, when opened, releases a robot that does whatever its sender wanted. **XXE** is like a letter that, when opened, reads itself and any other letters nearby. This cheat sheet teaches you how to seal each of these holes.

### Key Characteristics

- **Input-driven:** Every vulnerability class originates from untrusted input reaching a sensitive sink.
- **Defense in depth:** Multiple layers of protection (validation, encoding, parameterisation, least privilege).
- **Framework-aware:** Modern frameworks provide built-in protections, but they must be used correctly.
- **Catalogued:** OWASP Top 10, CWE Top 25, and SANS 25 provide canonical lists.
- **Testable:** Automated scanners (SAST, DAST, IAST) and manual testing (PortSwigger Academy) can detect most vulnerabilities.
- **Preventable:** Every class has well-established prevention strategies.

### Prerequisites

- **HTTP fundamentals:** Methods, headers, cookies, redirects, and URLs.
- **HTML, JavaScript, and the DOM:** How browsers parse and execute content.
- **SQL fundamentals:** Queries, parameters, and prepared statements.
- **Web frameworks:** Express, NestJS, and their security features.
- **Serialization formats:** JSON, XML, YAML, and their parsers.
- **Browser security model:** Same-origin policy, CORS, CSP, and cookies.
- **OWASP Top 10 and CWE:** The canonical vulnerability catalogues.

### Related Programming Areas

- **Authentication and authorization:** Broken access control is the #1 OWASP risk.
- **Input validation and output encoding:** The primary defences against injection.
- **Session management:** CSRF and session fixation are related.
- **Network security:** SSRF and path traversal involve server-side resources.
- **Dependency management:** Vulnerable libraries (Log4Shell, Spring4Shell) are a major risk.
- **DevSecOps:** SAST, DAST, dependency scanning, and secret scanning.

### Core Concepts

1. **SQL Injection** — preventing malicious query modification via parameterized statements and ORM usage.
2. **Cross-Site Scripting (XSS)** — neutralizing malicious script executions inside client browsers.
3. **Cross-Site Request Forgery (CSRF)** — blocking unauthorized request origin actions using anti-forgery tokens.
4. **SSRF (Server-Side Request Forgery)** — preventing backend application servers from querying internal network infrastructure.
5. **Path Traversal** — stopping attackers from accessing private local files using sanitized inputs like `path.resolve`.
6. **Broken Access Control** — resolving horizontal and vertical privilege escalation flaws.
7. **Insecure Deserialization** — preventing arbitrary code execution from improperly formatted object streams.
8. **XML External Entity (XXE) Injection** — disabling external entity resolution within server-side XML parsers.

---

## Core Concept 1: SQL Injection

### Definitions

**Core Definition:** SQL injection (SQLi) occurs when untrusted input is concatenated into a SQL query, allowing an attacker to modify the query's logic and read, modify, or delete arbitrary data.

**Technical Definition:** SQL injection (CWE-89) is a code injection technique that exploits applications that construct SQL queries by string concatenation with untrusted input. Attackers can inject SQL metacharacters (`'`, `--`, `;`, `UNION`, `OR 1=1`) to alter the query's structure. Variants include in-band (error-based, UNION-based), blind (boolean-based, time-based), and out-of-band injection. Prevention relies on parameterised queries (prepared statements), stored procedures, ORMs/query builders, input validation (allow-lists), and least-privilege database users.

**Beginner-Friendly Explanation:** Imagine a librarian who takes your request and writes it directly onto a form: "Fetch the book named [your request]." If you say "Harry Potter; also, throw away all other books," the librarian writes that too — and obeys. SQL injection is the same: the application takes your input and pastes it into a database query. If you include SQL commands, the database executes them. The fix: use a form with a blank line — the librarian only writes your book name, and the form's structure (the SQL query) is fixed.

### Purposes

- To prevent attackers from reading, modifying, or deleting arbitrary data.
- To prevent authentication bypass (`' OR '1'='1`).
- To prevent data exfiltration via `UNION SELECT`.
- To prevent denial of service via destructive queries.
- To comply with OWASP Top 10 (A03:2021 — Injection).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE — string concatenation
const query = `SELECT * FROM users WHERE email = '${email}' AND password = '${password}'`;
const result = await db.query(query);
```

**Attack:** `email = "' OR '1'='1' --"` bypasses authentication.

#### Parameterised Query (SAFE)

```typescript
// ✅ SAFE — parameterised query
const result = await db.query(
  'SELECT * FROM users WHERE email = $1 AND password = $2',
  [email, password],
);
```

#### ORM / Query Builder (SAFE)

```typescript
// ✅ SAFE — Prisma (parameterised internally)
const user = await prisma.user.findUnique({ where: { email } });

// ✅ SAFE — Knex (parameterised)
const user = await knex('users').where({ email }).first();

// ✅ SAFE — TypeORM query builder (parameterised)
const user = await dataSource
  .getRepository(User)
  .createQueryBuilder('user')
  .where('user.email = :email', { email })
  .getOne();
```

#### Syntax Rules

- **Always use parameterised queries** — never concatenate user input into SQL.
- **Use ORMs or query builders** — they parameterise by default.
- **Never trust input** — validate and sanitise even with parameterisation.
- **Use least-privilege database users** — the app should not have DDL privileges.
- **Avoid dynamic table/column names** — use allow-lists if unavoidable.
- **Disable detailed SQL errors in production** — prevent error-based injection.
- **Use stored procedures carefully** — they can also be vulnerable to injection.
- **Log and monitor** — detect injection attempts.

#### Constraints and Limitations

- **Parameterisation does not work for table/column names** — use allow-lists.
- **ORMs are not immune** — raw query methods (`$queryRaw`, `.raw()`) can be vulnerable.
- **Second-order injection** — data stored in the database and later used unsafely.
- **NoSQL injection** — MongoDB operators (`$ne`, `$gt`) can be injected.
- **Legacy code** — older applications may have many vulnerable queries.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preventing SQL Injection (Node.js + PostgreSQL)

```typescript
// ❌ VULNERABLE — DO NOT USE
app.get('/users', async (req, res) => {
  const { role } = req.query;
  // Attacker sends: ?role=' OR '1'='1
  const result = await db.query(`SELECT * FROM users WHERE role = '${role}'`);
  res.json(result.rows);
});

// ✅ SAFE — parameterised query
app.get('/users', async (req, res) => {
  const { role } = req.query;

  // Validate input (allow-list)
  const allowedRoles = ['admin', 'editor', 'user', 'guest'];
  if (!allowedRoles.includes(role as string)) {
    return res.status(400).json({ error: 'Invalid role' });
  }

  const result = await db.query(
    'SELECT id, email, name, role FROM users WHERE role = $1',
    [role],
  );
  res.json(result.rows);
});

// ✅ SAFE — Prisma
app.get('/users', async (req, res) => {
  const { role } = req.query;
  const users = await prisma.user.findMany({
    where: { role: role as string },
    select: { id: true, email: true, name: true, role: true },
  });
  res.json(users);
});
```

**Expected behaviour:**
- Attacker sends `?role=' OR '1'='1` — the parameterised query treats it as a literal string, finds no matching role, returns an empty array.
- Attacker sends `?role=admin` — returns admin users.
- Attacker sends `?role=hacker` — returns `400 Bad Request` (invalid role).

**Why this works:** Parameterised queries send the SQL structure and the data separately. The database treats the data as data, never as SQL code. The allow-list validation adds defense in depth.

### Real-World Cases

- **Authentication bypass:** `' OR '1'='1' --` in login forms.
- **Data exfiltration:** `UNION SELECT` to retrieve arbitrary data.
- **Data destruction:** `'; DROP TABLE users; --`.
- **Blind injection:** Time-based (`WAITFOR DELAY '0:0:5'`) to infer data.
- **Notable breaches:** Sony Pictures (2011), TalkTalk (2015), Fortnite (2019).

---

## Core Concept 2: Cross-Site Scripting (XSS)

### Definitions

**Core Definition:** XSS occurs when an application includes untrusted data in a web page without proper validation or encoding, allowing attackers to inject malicious scripts that execute in victims' browsers.

**Technical Definition:** XSS (CWE-79) is a code injection vulnerability where attacker-controlled JavaScript executes in the context of a trusted website. Three types: **Stored** (persistent — the payload is saved in the database and served to all users), **Reflected** (the payload is in the request and immediately reflected in the response), and **DOM-based** (the payload is processed by client-side JavaScript without server involvement). Prevention requires context-aware output encoding, HTML sanitisation (DOMPurify), Content Security Policy (CSP), and secure cookie flags (`HttpOnly`).

**Beginner-Friendly Explanation:** Imagine a guestbook where visitors write messages. If the guestbook displays messages as raw HTML, a visitor could write `<script>stealCookies()</script>`, and every subsequent visitor's browser would run that script. XSS is the same: the application takes user input and includes it in the HTML page. If the input contains a script, the browser runs it. The fix: always encode user input before inserting it into HTML — so `<script>` becomes `&lt;script&gt;` and is displayed as text, not executed.

### Purposes

- To prevent attackers from stealing cookies and session tokens.
- To prevent keylogging and form hijacking.
- To prevent defacement of web pages.
- To prevent redirection to malicious sites.
- To comply with OWASP Top 10 (A03:2021 — Injection).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```html
<!-- ❌ VULNERABLE -->
<div>Welcome, <%= userName %></div>
<!-- Attacker sets userName to <img src=x onerror=alert(1)> -->
```

```typescript
// ❌ VULNERABLE — innerHTML
element.innerHTML = userInput;
```

#### Safe Encoding

```typescript
// ✅ SAFE — escape HTML entities
function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

// ✅ SAFE — textContent (auto-escapes)
element.textContent = userInput;
```

#### Framework Auto-Escaping

```tsx
// ✅ SAFE — React escapes by default
<div>Welcome, {userName}</div>

// ❌ VULNERABLE — dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ SAFE — Vue escapes by default
<div>Welcome, {{ userName }}</div>

// ❌ VULNERABLE — v-html
<div v-html="userInput"></div>
```

#### HTML Sanitisation (DOMPurify)

```typescript
import DOMPurify from 'dompurify';

// ✅ SAFE — sanitise before inserting HTML
const clean = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
  ALLOWED_ATTR: ['href'],
});
element.innerHTML = clean;
```

#### Content Security Policy (CSP)

```typescript
// ✅ SAFE — CSP header blocks inline scripts
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data: https:; object-src 'none'; base-uri 'self'; frame-ancestors 'none';",
  );
  next();
});
```

#### Syntax Rules

- **Encode output contextually** — HTML, JavaScript, URL, CSS each require different encoding.
- **Use framework auto-escaping** — React, Vue, Angular escape by default.
- **Avoid `innerHTML`, `dangerouslySetInnerHTML`, `v-html`** — use safe alternatives.
- **Sanitise HTML with DOMPurify** — if rich text is required.
- **Set `HttpOnly` on session cookies** — prevents XSS cookie theft.
- **Implement CSP** — blocks inline scripts and untrusted sources.
- **Use `textContent` instead of `innerHTML`** — auto-escapes.
- **Validate input on the server** — never trust the client.
- **Use `SameSite` cookies** — mitigates some XSS consequences.

#### Constraints and Limitations

- **CSP can break legitimate functionality** — requires careful configuration.
- **DOM-based XSS bypasses server-side encoding** — requires client-side care.
- **Rich text editors require sanitisation** — HTML cannot simply be escaped.
- **`HttpOnly` does not prevent XSS** — it only prevents cookie theft.
- **Browser XSS filters are deprecated** — do not rely on them.
- **CSP `unsafe-inline`** undermines the policy — avoid it.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preventing XSS in Express (Server-Side)

```typescript
import express from 'express';
import helmet from 'helmet';
import DOMPurify from 'isomorphic-dompurify';

const app = express();
app.use(express.json());

// Helmet: sets CSP and other security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      objectSrc: ["'none'"],
      baseUri: ["'self'"],
      frameAncestors: ["'none'"],
    },
  },
}));

// ❌ VULNERABLE — echoing user input without encoding
app.get('/vulnerable', (req, res) => {
  const name = req.query.name as string;
  res.send(`<h1>Hello, ${name}!</h1>`);
  // Attacker sends ?name=<script>alert(1)</script>
});

// ✅ SAFE — escape HTML entities
function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

app.get('/safe', (req, res) => {
  const name = escapeHtml(req.query.name as string);
  res.send(`<h1>Hello, ${name}!</h1>`);
  // Attacker sends ?name=<script>alert(1)</script>
  // Output: <h1>Hello, &lt;script&gt;alert(1)&lt;/script&gt;!</h1>
});

// ✅ SAFE — sanitise rich text with DOMPurify
app.post('/comments', async (req, res) => {
  const { body } = req.body;
  const clean = DOMPurify.sanitize(body, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href'],
  });
  const comment = await prisma.comment.create({ data: { body: clean } });
  res.status(201).json(comment);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `/vulnerable?name=<script>alert(1)</script>` executes the script (vulnerable).
- `/safe?name=<script>alert(1)</script>` displays the text `<script>alert(1)</script>` (safe).
- Comments are sanitised; only allowed tags are kept.
- CSP blocks inline scripts even if an attacker bypasses encoding.

**Why this works:** Multiple layers — output encoding, HTML sanitisation, and CSP. Each layer alone provides some protection; together, they provide defense in depth.

#### Example 2: Preventing DOM-Based XSS (Client-Side)

```typescript
// ❌ VULNERABLE — DOM-based XSS
const userInput = new URLSearchParams(window.location.search).get('name');
document.getElementById('greeting').innerHTML = `Hello, ${userInput}!`;

// ✅ SAFE — textContent
const userInput = new URLSearchParams(window.location.search).get('name');
document.getElementById('greeting').textContent = `Hello, ${userInput}!`;

// ✅ SAFE — DOMPurify for rich text
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput, { ALLOWED_TAGS: ['b', 'i'] });
document.getElementById('greeting').innerHTML = clean;
```

**Expected behaviour:** The vulnerable version executes `<img src=x onerror=alert(1)>`. The safe version displays it as text.

**Why this works:** `textContent` auto-escapes. DOMPurify removes dangerous tags and attributes.

### Real-World Cases

- **Stored XSS:** Comment fields, forum posts, user profiles.
- **Reflected XSS:** Search results, error messages, URL parameters.
- **DOM-based XSS:** Client-side routing, `innerHTML` assignments.
- **Notable breaches:** British Airways (2018), MySpace (Samy worm, 2005), Twitter (2010).

---

## Core Concept 3: Cross-Site Request Forgery (CSRF)

### Definitions

**Core Definition:** CSRF occurs when an attacker tricks a victim's browser into sending an authenticated request to a trusted site without the victim's knowledge, exploiting the browser's automatic cookie-sending behaviour.

**Technical Definition:** CSRF (CWE-352) exploits the fact that browsers automatically attach cookies (including session cookies) to requests sent to a domain, regardless of the request's origin. An attacker hosts a page that submits a form or sends a request to the target site; the victim's browser includes their session cookie, and the server performs the action. Prevention requires anti-CSRF tokens (synchronizer token pattern or double-submit cookie), `SameSite` cookies, Origin/Referer header validation, and custom headers for AJAX requests.

**Beginner-Friendly Explanation:** Imagine you're logged into your bank. You visit a malicious website that has a hidden form that says "Transfer $1,000 to attacker." The form auto-submits to your bank. Because your browser automatically sends your bank's session cookie with the request, the bank thinks it's you and processes the transfer. CSRF is the same: the attacker uses your authenticated session without your knowledge. The fix: include a secret token in every state-changing request that the attacker cannot predict or obtain.

### Purposes

- To prevent unauthorized state-changing actions on behalf of authenticated users.
- To prevent account takeover via password or email changes.
- To prevent unauthorized financial transactions.
- To comply with OWASP Top 10 (A01:2021 — Broken Access Control).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE — no CSRF protection
app.post('/transfer', (req, res) => {
  const { to, amount } = req.body;
  // No CSRF check — attacker can submit this from any site
  bankService.transfer(req.session.userId, to, amount);
  res.json({ success: true });
});
```

#### Synchronizer Token Pattern

```typescript
import csrf from 'csurf';

const csrfProtection = csrf({ cookie: { httpOnly: true, secure: true, sameSite: 'lax' } });

// Provide the token to the client
app.get('/csrf-token', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Validate the token on state-changing requests
app.post('/transfer', csrfProtection, (req, res) => {
  const { to, amount } = req.body;
  bankService.transfer(req.session.userId, to, amount);
  res.json({ success: true });
});

// CSRF error handler
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  next();
});
```

#### Double-Submit Cookie Pattern

```typescript
import { randomBytes } from 'node:crypto';

// Set the CSRF cookie
app.use((req, res, next) => {
  if (!req.cookies['__Host-csrf']) {
    const token = randomBytes(32).toString('base64url');
    res.cookie('__Host-csrf', token, {
      httpOnly: true,
      secure: true,
      sameSite: 'lax',
      path: '/',
    });
  }
  next();
});

// Validate: the header must match the cookie
function validateCsrf(req, res, next) {
  const cookieToken = req.cookies['__Host-csrf'];
  const headerToken = req.headers['x-csrf-token'];

  if (!cookieToken || !headerToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  next();
}

app.post('/transfer', validateCsrf, (req, res) => {
  // ...
});
```

#### SameSite Cookies

```typescript
res.cookie('__Host-sid', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict', // or 'lax'
  path: '/',
});
```

#### Syntax Rules

- **Use anti-CSRF tokens** — synchronizer or double-submit.
- **Set `SameSite=Lax` or `Strict`** — first line of defense.
- **Validate Origin/Referer headers** — for extra protection.
- **Require custom headers for AJAX** — `X-Requested-With` or `X-CSRF-Token`.
- **Never use GET for state-changing operations** — use POST/PUT/DELETE.
- **Regenerate tokens after login** — prevent fixation.
- **Use `__Host-` prefix** — strongest cookie protection.
- **Reject requests without tokens** — fail securely.

#### Constraints and Limitations

- **`SameSite=Lax` does not protect against GET-based CSRF** — use `Strict` for sensitive operations.
- **Token management adds complexity** — must be included in every form/AJAX request.
- **XSS can bypass CSRF** — if an attacker can execute JavaScript, they can read the token.
- **API clients** — mobile apps and CLI tools do not use cookies, so CSRF does not apply (but they need other protections).
- **Subdomain attacks** — a compromised subdomain can set cookies for the parent domain (mitigated by `__Host-`).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full CSRF Protection (Express)

```typescript
import express from 'express';
import cookieParser from 'cookie-parser';
import csrf from 'csurf';
import session from 'express-session';

const app = express();
app.use(express.json());
app.use(cookieParser());
app.use(session({
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, secure: true, sameSite: 'lax', path: '/' },
}));

const csrfProtection = csrf({ cookie: false }); // Store token in session

// Endpoint to get the CSRF token (called by the SPA on load)
app.get('/api/csrf-token', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// All state-changing routes are protected
app.post('/api/transfer', csrfProtection, async (req, res) => {
  const { to, amount } = req.body;
  await bankService.transfer(req.session.userId, to, amount);
  res.json({ success: true });
});

// CSRF error handler
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  next(err);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

```typescript
// SPA client — fetch token and include in requests
async function transfer(to: string, amount: number) {
  // Fetch the CSRF token
  const { csrfToken } = await fetch('/api/csrf-token').then((r) => r.json());

  // Include it in the request
  return fetch('/api/transfer', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken,
    },
    body: JSON.stringify({ to, amount }),
  });
}
```

**Expected behaviour:**
- The SPA fetches the CSRF token on load.
- State-changing requests include the token in the `X-CSRF-Token` header.
- Requests without a valid token return `403 Forbidden`.
- A malicious site cannot obtain the token (same-origin policy).

**Why this works:** The synchronizer token pattern stores a secret in the session and requires it in every state-changing request. The attacker cannot read the token due to the same-origin policy. `SameSite=Lax` provides additional protection.

### Real-World Cases

- **Banking:** Transfer, password change, and email change require CSRF tokens.
- **E-commerce:** Checkout and payment methods require CSRF protection.
- **Social media:** Post creation, profile changes, and password resets.
- **Notable breaches:** Netflix (2006), YouTube (2008), ING Direct (2008).

---

## Core Concept 4: SSRF (Server-Side Request Forgery)

### Definitions

**Core Definition:** SSRF occurs when an attacker can make the server send requests to arbitrary destinations, potentially accessing internal services, cloud metadata endpoints, or the local file system.

**Technical Definition:** SSRF (CWE-918) exploits applications that fetch remote resources based on user-supplied URLs (e.g., webhooks, image proxies, PDF generators, URL previews). Attackers supply URLs pointing to internal services (`http://169.254.169.254/` for AWS metadata, `http://localhost:6379/` for Redis, `http://internal-admin/`). Prevention requires allow-listing URLs, blocking private IP ranges, disabling redirects, using a dedicated egress proxy, and validating the resolved IP before connecting.

**Beginner-Friendly Explanation:** Imagine you ask the concierge to fetch a package from "the front desk." The concierge walks out of the building, through the parking lot, into the basement, and opens a locked door — because "the front desk" was actually a room inside the building. SSRF is the same: the attacker asks your server to fetch a URL, and your server makes requests to internal services that the attacker cannot reach directly. The fix: restrict which URLs the server can fetch, block internal IPs, and use a proxy for outbound requests.

### Purposes

- To prevent attackers from accessing internal services.
- To prevent cloud metadata credential theft (AWS, GCP, Azure).
- To prevent port scanning of internal networks.
- To prevent access to the local file system via `file://`.
- To comply with OWASP Top 10 (A10:2021 — SSRF).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE — fetches any URL
app.post('/fetch', async (req, res) => {
  const { url } = req.body;
  const response = await fetch(url); // Attacker can fetch internal URLs
  res.json(await response.json());
});
```

#### Allow-List Validation

```typescript
// ✅ SAFE — allow-list of domains
const ALLOWED_HOSTS = ['api.example.com', 'cdn.example.com', 'images.example.com'];

async function safeFetch(url: string) {
  const parsed = new URL(url);

  // 1. Enforce HTTPS
  if (parsed.protocol !== 'https:') {
    throw new Error('Only HTTPS is allowed');
  }

  // 2. Allow-list hosts
  if (!ALLOWED_HOSTS.includes(parsed.hostname)) {
    throw new Error('Host not allowed');
  }

  // 3. Block private IPs (in case DNS resolves to one)
  const addresses = await dns.promises.resolve4(parsed.hostname);
  for (const addr of addresses) {
    if (isPrivateIP(addr)) {
      throw new Error('Private IP not allowed');
    }
  }

  return fetch(url, { redirect: 'error' }); // Disable redirects
}

function isPrivateIP(ip: string): boolean {
  return /^(10\.|172\.(1[6-9]|2\d|3[01])\.|192\.168\.|127\.|169\.254\.|0\.)/.test(ip);
}
```

#### Syntax Rules

- **Allow-list URLs** — never allow arbitrary URLs.
- **Enforce HTTPS** — reject `http:`, `file:`, `gopher:`, etc.
- **Block private IP ranges** — `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.0/8`, `169.254.0.0/16`.
- **Resolve DNS before connecting** — prevent DNS rebinding.
- **Disable redirects** — or validate each redirect target.
- **Use a dedicated egress proxy** — with allow-listed destinations.
- **Do not return raw responses** — prevent information leakage.
- **Rate-limit fetch endpoints** — prevent abuse.
- **Log all outbound requests** — for audit and monitoring.

#### Constraints and Limitations

- **DNS rebinding** — an attacker-controlled DNS can resolve to a private IP after validation.
- **IPv6** — private IPv6 ranges (`fc00::/7`, `::1`) must also be blocked.
- **Cloud metadata** — `169.254.169.254` requires IMDSv2 on AWS.
- **Redirects** — an allow-listed URL can redirect to an internal service.
- **URL parsers differ** — `http://example.com@internal/` may parse differently.
- **Legitimate use cases** — webhooks, image proxies, and URL previews require careful design.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: SSRF Protection with Allow-List and DNS Rebinding Prevention (Node.js)

```typescript
import dns from 'node:dns';
import ipaddr from 'ipaddr.js';

const ALLOWED_HOSTS = ['api.example.com', 'cdn.example.com'];

function isPrivateIP(ip: string): boolean {
  const addr = ipaddr.parse(ip);
  const range = addr.range();

  const privateRanges = [
    'private',        // 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
    'loopback',       // 127.0.0.0/8, ::1
    'linkLocal',      // 169.254.0.0/16
    'uniqueLocal',    // fc00::/7
    'unspecified',    // 0.0.0.0
    'broadcast',
    'carrierGradeNat',
  ];

  return privateRanges.includes(range);
}

async function safeFetch(url: string): Promise<Response> {
  const parsed = new URL(url);

  // 1. Enforce HTTPS
  if (parsed.protocol !== 'https:') {
    throw new Error('Only HTTPS is allowed');
  }

  // 2. Allow-list hosts
  if (!ALLOWED_HOSTS.includes(parsed.hostname)) {
    throw new Error('Host not allowed');
  }

  // 3. Resolve DNS and validate IPs
  const addresses = await dns.promises.resolve4(parsed.hostname);
  for (const addr of addresses) {
    if (isPrivateIP(addr)) {
      throw new Error('Private IP not allowed');
    }
  }

  // 4. Fetch with redirects disabled
  const response = await fetch(url, {
    redirect: 'error',
    signal: AbortSignal.timeout(5000), // 5-second timeout
  });

  // 5. Validate the response size
  const contentLength = parseInt(response.headers.get('content-length') ?? '0');
  if (contentLength > 10 * 1024 * 1024) {
    throw new Error('Response too large');
  }

  return response;
}

app.post('/fetch', async (req, res) => {
  try {
    const response = await safeFetch(req.body.url);
    const data = await response.text();
    res.json({ data: data.slice(0, 10000) }); // Limit response size
  } catch (err) {
    res.status(400).json({ error: (err as Error).message });
  }
});
```

**Expected behaviour:**
- `https://api.example.com/data` — allowed.
- `https://evil.com/data` — rejected (host not allowed).
- `http://api.example.com/data` — rejected (not HTTPS).
- `https://api.example.com@169.254.169.254/` — rejected (host mismatch).
- `https://api.example.com/data` with DNS resolving to a private IP — rejected.

**Why this works:** Multiple layers — protocol allow-list, host allow-list, DNS resolution and IP validation, redirect disabling, timeout, and response size limits. This prevents SSRF even with DNS rebinding attempts.

### Real-World Cases

- **Cloud metadata theft:** Accessing `http://169.254.169.254/latest/meta-data/iam/security-credentials/` on AWS.
- **Internal service access:** Fetching `http://internal-admin:8080/` from the application server.
- **Port scanning:** Using the server to probe internal ports.
- **Notable breaches:** Capital One (2019), Shopify (2020), GitLab (2021).

---

## Core Concept 5: Path Traversal

### Definitions

**Core Definition:** Path traversal (directory traversal) occurs when an attacker manipulates file paths to access files outside the intended directory, using sequences like `../` or absolute paths.

**Technical Definition:** Path traversal (CWE-22) exploits applications that construct file paths from user input without proper validation. Attackers use `../` (or `..\` on Windows), URL-encoded variants (`%2e%2e%2f`), or absolute paths (`/etc/passwd`, `C:\Windows\win.ini`) to escape the intended directory. Prevention requires normalising paths, resolving them to absolute paths, verifying they remain within a base directory, and never passing user input directly to file system APIs.

**Beginner-Friendly Explanation:** Imagine a hotel where guests can request their own room service menu by room number. If a guest asks for "room 101/../../kitchen/master-key-file," the staff walks up a level and into the kitchen. Path traversal is the same: the application takes a file name from the user and appends it to a base directory. If the user includes `../`, the path escapes the base directory. The fix: normalise the path, resolve it to an absolute path, and verify it's still inside the base directory.

### Purposes

- To prevent attackers from reading arbitrary files (`/etc/passwd`, `.env`, private keys).
- To prevent writing arbitrary files (web shells, cron jobs).
- To prevent deletion or modification of critical files.
- To comply with OWASP Top 10 (A01:2021 — Broken Access Control).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE
app.get('/files/:name', (req, res) => {
  const filePath = path.join('/uploads', req.params.name);
  res.sendFile(filePath);
  // Attacker requests: /files/../../etc/passwd
});
```

#### Safe Path Validation

```typescript
import path from 'node:path';
import fs from 'node:fs';

const UPLOAD_DIR = '/uploads';

app.get('/files/:name', (req, res) => {
  // 1. Resolve the absolute path
  const requestedPath = path.resolve(UPLOAD_DIR, req.params.name);

  // 2. Verify the path is inside the upload directory
  if (!requestedPath.startsWith(UPLOAD_DIR + path.sep)) {
    return res.status(403).json({ error: 'Access denied' });
  }

  // 3. Verify the file exists and is a regular file
  const stats = fs.statSync(requestedPath);
  if (!stats.isFile()) {
    return res.status(404).json({ error: 'Not found' });
  }

  res.sendFile(requestedPath);
});
```

#### Syntax Rules

- **Use `path.resolve()`** — normalise and resolve to an absolute path.
- **Verify the resolved path starts with the base directory** — plus `path.sep`.
- **Never use `path.join()` alone** — it does not prevent traversal.
- **Reject encoded traversal sequences** — `%2e%2e%2f`, `%252e%252e%252f`.
- **Use allow-lists for file names** — if possible.
- **Never pass user input directly to `fs` APIs** — always validate first.
- **Reject absolute paths** — if the input should be relative.
- **Use a random file name** — when storing uploads (e.g., UUID).
- **Restrict file types** — validate MIME types and extensions.
- **Run the application as a non-root user** — limit the damage.

#### Constraints and Limitations

- **Symlinks** — a symlink inside the base directory can point outside it.
- **Case-insensitive file systems** — Windows and macOS may behave differently.
- **URL encoding** — multiple encoding layers can bypass naive checks.
- **Unicode normalisation** — some characters may bypass checks.
- **Race conditions (TOCTOU)** — the file may change between validation and use.
- **Cloud storage** — object storage (S3) uses keys, not paths — traversal does not apply.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe File Serving (Express)

```typescript
import express from 'express';
import path from 'node:path';
import fs from 'node:fs/promises';

const app = express();
const UPLOAD_DIR = path.resolve('/var/app/uploads');

app.get('/files/:name', async (req, res) => {
  const requestedName = req.params.name;

  // 1. Reject obvious traversal attempts early
  if (requestedName.includes('..') || requestedName.includes('/') || requestedName.includes('\\')) {
    return res.status(400).json({ error: 'Invalid file name' });
  }

  // 2. Resolve the absolute path
  const requestedPath = path.resolve(UPLOAD_DIR, requestedName);

  // 3. Verify the path is inside the upload directory
  if (!requestedPath.startsWith(UPLOAD_DIR + path.sep)) {
    return res.status(403).json({ error: 'Access denied' });
  }

  // 4. Verify the file exists and is a regular file
  try {
    const stats = await fs.stat(requestedPath);
    if (!stats.isFile()) {
      return res.status(404).json({ error: 'Not found' });
    }

    // 5. Resolve symlinks and re-check
    const realPath = await fs.realpath(requestedPath);
    if (!realPath.startsWith(UPLOAD_DIR + path.sep)) {
      return res.status(403).json({ error: 'Access denied' });
    }

    res.sendFile(realPath);
  } catch (err) {
    if ((err as NodeJS.ErrnoException).code === 'ENOENT') {
      return res.status(404).json({ error: 'Not found' });
    }
    res.status(500).json({ error: 'Internal error' });
  }
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `/files/report.pdf` — serves the file if it exists.
- `/files/../../etc/passwd` — rejected (contains `..`).
- `/files/%2e%2e%2f%2e%2e%2fetc%2fpasswd` — rejected (contains `..` after URL decoding).
- `/files/symlink-to-etc-passwd` — rejected (realpath check fails).

**Why this works:** Multiple layers — early rejection of traversal characters, `path.resolve()` normalisation, prefix check, `stat()` verification, and `realpath()` symlink resolution. This prevents path traversal even with encoding and symlink attacks.

### Real-World Cases

- **File download endpoints:** Serving user-uploaded files by name.
- **Template rendering:** Loading templates by name.
- **Log viewing:** Displaying log files by date.
- **Notable breaches:** Apache Struts (CVE-2017-5638), Jenkins (CVE-2018-1000861), various WordPress plugins.

---

## Core Concept 6: Broken Access Control

### Definitions

**Core Definition:** Broken access control occurs when an application fails to properly enforce restrictions on what authenticated users are allowed to do, leading to horizontal (same-level users) or vertical (lower-to-higher privilege) escalation.

**Technical Definition:** Broken Access Control (CWE-284) is the #1 OWASP Top 10 risk (A01:2021). It includes: **horizontal privilege escalation** (user A accesses user B's resources), **vertical privilege escalation** (user accesses admin functions), **IDOR** (Insecure Direct Object Reference — using predictable IDs to access other users' data), **missing function-level access control** (admin endpoints accessible to regular users), **forced browsing** (accessing pages not linked in the UI), and **metadata manipulation** (JWT claims, cookies, hidden fields). Prevention requires deny-by-default, server-side enforcement, resource-level checks, tenant isolation, and audit logging.

**Beginner-Friendly Explanation:** Imagine a hotel where every room key opens every door. Broken access control is the same: the application does not check whether the user is allowed to access a resource. If you know the room number (ID), you can walk in. The fix: every request must be checked against the user's permissions — both "can they use this function?" (vertical) and "can they access this specific resource?" (horizontal).

### Purposes

- To prevent unauthorized access to resources and functions.
- To prevent horizontal privilege escalation (accessing other users' data).
- To prevent vertical privilege escalation (accessing admin functions).
- To prevent IDOR vulnerabilities.
- To comply with OWASP Top 10 (A01:2021).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE — no authorization check
app.get('/api/orders/:id', async (req, res) => {
  const order = await prisma.order.findUnique({ where: { id: req.params.id } });
  res.json(order); // Any authenticated user can access any order
});

// ❌ VULNERABLE — client-side role check
app.get('/api/admin/users', async (req, res) => {
  if (req.query.role === 'admin') { // Attacker can set this
    return res.json(await prisma.user.findMany());
  }
  res.status(403).json({ error: 'Forbidden' });
});
```

#### Safe Authorization

```typescript
// ✅ SAFE — resource-level authorization
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await prisma.order.findFirst({
    where: { id: req.params.id, userId: req.user.id }, // Ownership check
  });
  if (!order) return res.status(404).json({ error: 'Not found' });
  res.json(order);
});

// ✅ SAFE — server-side role check
app.get('/api/admin/users', requireAuth, requireRole('admin'), async (req, res) => {
  res.json(await prisma.user.findMany());
});

// ✅ SAFE — tenant isolation
app.get('/api/posts', requireAuth, async (req, res) => {
  const posts = await prisma.post.findMany({
    where: { tenantId: req.user.tenantId }, // Tenant filter
  });
  res.json(posts);
});
```

#### Syntax Rules

- **Deny by default** — if no rule grants access, deny.
- **Enforce on the server** — never trust client-side checks.
- **Check resource ownership** — for every resource access.
- **Filter by tenant** — in multi-tenant applications.
- **Use centralized authorization** — guards, middleware, or policy engines.
- **Return 404, not 403** — to hide resource existence (or return 403 if appropriate).
- **Log authorization failures** — for security monitoring.
- **Test negative cases** — verify that unauthorized access is denied.
- **Audit permissions regularly** — remove unused grants.
- **Never expose internal IDs** — use UUIDs or hashids.

#### Constraints and Limitations

- **Authorization is application-specific** — no one-size-fits-all solution.
- **IDOR is easy to miss** — every endpoint that accepts an ID needs a check.
- **Role-based checks are insufficient** — resource-level checks are also needed.
- **Admin endpoints are often overlooked** — they must be protected too.
- **API versions can diverge** — v1 may be protected, v2 may not.
- **Mobile and CLI clients** — may bypass web-only protections.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preventing Horizontal and Vertical Privilege Escalation (NestJS)

```typescript
// orders.controller.ts
@Controller('orders')
@UseGuards(AuthGuard)
export class OrdersController {
  constructor(private readonly ordersService: OrdersService) {}

  @Get()
  async findAll(@CurrentUser() user: AuthUser) {
    // Horizontal: only the user's own orders
    return this.ordersService.findByUser(user.id);
  }

  @Get(':id')
  async findById(@Param('id') id: string, @CurrentUser() user: AuthUser) {
    // Horizontal: ownership check
    return this.ordersService.findByIdForUser(id, user.id);
  }

  @Get('admin/all')
  @UseGuards(RolesGuard)
  @Roles('admin')
  async findAllAdmin() {
    // Vertical: admin-only
    return this.ordersService.findAll();
  }

  @Delete(':id')
  async remove(@Param('id') id: string, @CurrentUser() user: AuthUser) {
    // Horizontal + vertical
    return this.ordersService.remove(id, user);
  }
}

// orders.service.ts
@Injectable()
export class OrdersService {
  constructor(private readonly prisma: PrismaService) {}

  async findByUser(userId: string) {
    return this.prisma.order.findMany({ where: { userId } });
  }

  async findByIdForUser(id: string, userId: string) {
    const order = await this.prisma.order.findFirst({
      where: { id, userId }, // Ownership check
    });
    if (!order) throw new NotFoundException('Order not found');
    return order;
  }

  async findAll() {
    return this.prisma.order.findMany();
  }

  async remove(id: string, user: AuthUser) {
    // Admins can delete any order; users can delete only their own
    const isAdmin = user.roles.includes('admin');

    const order = await this.prisma.order.findFirst({
      where: isAdmin ? { id } : { id, userId: user.id },
    });

    if (!order) throw new NotFoundException('Order not found');
    await this.prisma.order.delete({ where: { id } });
  }
}
```

**Expected behaviour:**
- `GET /orders` returns only the user's orders.
- `GET /orders/:id` returns the order only if it belongs to the user.
- `GET /orders/admin/all` returns all orders (admin only).
- `DELETE /orders/:id` allows admins to delete any order, users only their own.

**Why this works:** Every endpoint enforces authorization. Horizontal checks (ownership) prevent IDOR. Vertical checks (roles) prevent privilege escalation. Deny by default.

### Real-World Cases

- **IDOR:** Accessing `/api/users/123/orders` for another user.
- **Forced browsing:** Accessing `/admin/users` directly.
- **JWT claim manipulation:** Changing `"role": "user"` to `"role": "admin"`.
- **Notable breaches:** Facebook (2019), Peloton (2021), Parler (2021), USPS (2018).

---

## Core Concept 7: Insecure Deserialization

### Definitions

**Core Definition:** Insecure deserialization occurs when untrusted data is deserialized without validation, allowing attackers to manipulate object state, execute arbitrary code, or escalate privileges.

**Technical Definition:** Insecure Deserialization (CWE-502) is the exploitation of deserialization processes that reconstruct objects from untrusted data. Vulnerable languages include Java (Java serialization, `ObjectInputStream`), PHP (`unserialize()`), Python (`pickle`), Ruby (`Marshal`), and .NET (`BinaryFormatter`). Attackers craft serialized payloads that, when deserialized, trigger gadgets in the application's classpath (gadget chains) to execute code, read files, or perform SSRF. Prevention requires avoiding native serialization formats, using safe formats (JSON), validating and signing serialized data, and isolating deserialization in sandboxes.

**Beginner-Friendly Explanation:** Imagine you receive a package in the mail. You open it, and inside is a robot that immediately starts doing whatever its sender programmed it to do. Insecure deserialization is the same: the application receives serialized data, reconstructs it into objects, and the reconstruction process triggers code execution. The fix: never deserialize untrusted data with native serialization formats — use JSON with strict schemas, or sign/encrypt the serialized data.

### Purposes

- To prevent arbitrary code execution from deserialized payloads.
- To prevent privilege escalation via manipulated object state.
- To prevent denial of service via malicious payloads.
- To comply with OWASP Top 10 (A08:2021 — Software and Data Integrity Failures).

### Syntax Rules and Structure

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE — Node.js does not have native deserialization,
// but eval, Function, and vm can be equally dangerous
app.post('/deserialize', (req, res) => {
  const data = eval(req.body.data); // NEVER DO THIS
  res.json(data);
});

// ❌ VULNERABLE — YAML with custom types
import yaml from 'js-yaml';
const data = yaml.load(req.body.yaml); // Can execute arbitrary code with some schemas
```

#### Safe Deserialization

```typescript
// ✅ SAFE — JSON with schema validation (Zod)
import { z } from 'zod';

const UserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional(),
});

app.post('/users', (req, res) => {
  const result = UserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ error: result.error.issues });
  }
  // result.data is validated and typed
  res.json(result.data);
});

// ✅ SAFE — YAML with safe schema
import yaml from 'js-yaml';
const data = yaml.load(req.body.yaml, { schema: yaml.JSON_SCHEMA });
```

#### Signed Serialized Data

```typescript
import { createHmac, timingSafeEqual } from 'node:crypto';

// Sign the data with HMAC
function sign(data: string, secret: string): string {
  const signature = createHmac('sha256', secret).update(data).digest('base64url');
  return `${data}.${signature}`;
}

// Verify the signature before deserializing
function verifyAndParse(signed: string, secret: string): unknown {
  const [data, signature] = signed.split('.');
  const expected = createHmac('sha256', secret).update(data).digest('base64url');

  if (!timingSafeEqual(Buffer.from(signature), Buffer.from(expected))) {
    throw new Error('Invalid signature');
  }

  return JSON.parse(data);
}
```

#### Syntax Rules

- **Never deserialize untrusted data** with native serialization formats.
- **Use JSON with strict schemas** — Zod, Joi, class-validator.
- **Avoid `eval`, `Function`, and `vm`** — they execute arbitrary code.
- **Use safe YAML schemas** — `JSON_SCHEMA` or `CORE_SCHEMA`.
- **Sign serialized data** — with HMAC or digital signatures.
- **Validate after deserialization** — schema validation catches malicious payloads.
- **Isolate deserialization** — in a sandbox or separate process.
- **Keep dependencies updated** — gadget chains often rely on known libraries.
- **Monitor for deserialization attacks** — log suspicious payloads.

#### Constraints and Limitations

- **Node.js has fewer native deserialization risks** — but `eval`, `Function`, and `vm` are dangerous.
- **`node-serialize`** is a known vulnerable package — never use it.
- **Some libraries deserialize internally** — Redis, Memcached, and message queues may use native formats.
- **Gadget chains are hard to detect** — they rely on the application's dependencies.
- **Signed data requires key management** — HMAC secrets must be protected.
- **JSON is not immune** — prototype pollution can occur with `__proto__`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe Deserialization with Schema Validation (Node.js)

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json({ limit: '100kb' }));

// ❌ VULNERABLE — eval
app.post('/unsafe', (req, res) => {
  try {
    const data = eval(req.body.expression); // NEVER DO THIS
    res.json({ result: data });
  } catch (err) {
    res.status(400).json({ error: 'Invalid expression' });
  }
});

// ✅ SAFE — schema validation
const UserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional(),
  address: z.object({
    street: z.string().max(200),
    city: z.string().max(100),
    zip: z.string().regex(/^\d{5}$/),
  }).optional(),
}).strict(); // Reject unknown properties

app.post('/safe', (req, res) => {
  const result = UserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ error: result.error.issues });
  }
  res.json(result.data);
});

// ✅ SAFE — signed payloads (for data that must be round-tripped)
import { createHmac, timingSafeEqual } from 'node:crypto';

const SECRET = process.env.HMAC_SECRET!;

function sign(data: unknown): string {
  const json = JSON.stringify(data);
  const signature = createHmac('sha256', SECRET).update(json).digest('base64url');
  return Buffer.from(json).toString('base64url') + '.' + signature;
}

function verify<T>(signed: string): T {
  const [encoded, signature] = signed.split('.');
  const json = Buffer.from(encoded, 'base64url').toString();
  const expected = createHmac('sha256', SECRET).update(json).digest('base64url');

  if (signature.length !== expected.length || !timingSafeEqual(Buffer.from(signature), Buffer.from(expected))) {
    throw new Error('Invalid signature');
  }

  return JSON.parse(json);
}

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `/unsafe` with `{"expression": "process.exit()"}` would crash the server (vulnerable).
- `/safe` with valid data returns the validated object.
- `/safe` with extra properties returns `400 Bad Request` (strict schema).
- Signed payloads are verified before parsing.

**Why this works:** No `eval`, no native deserialization. Schema validation ensures only expected properties are accepted. Signed payloads ensure integrity and authenticity.

### Real-World Cases

- **Java deserialization:** Apache Commons Collections, WebLogic, Jenkins.
- **PHP deserialization:** Laravel, WordPress plugins.
- **Python pickle:** ML models, cached data.
- **.NET BinaryFormatter:** Legacy applications.
- **Notable breaches:** Equifax (2017, Apache Struts), JBoss (CVE-2015-7501), Jenkins (CVE-2016-0792).

---

## Core Concept 8: XML External Entity (XXE) Injection

### Definitions

**Core Definition:** XXE occurs when an XML parser processes external entity references in untrusted XML input, allowing attackers to read files, perform SSRF, or cause denial of service.

**Technical Definition:** XXE (CWE-611) exploits XML parsers that resolve external entities. Attackers inject a DOCTYPE declaration with an external entity pointing to a local file (`file:///etc/passwd`) or an internal service (`http://169.254.169.254/`). When the parser processes the XML, it reads the file or makes the request, and the result may be reflected in the response. Prevention requires disabling DTD processing, disabling external entity resolution, and using safe parser configurations.

**Beginner-Friendly Explanation:** Imagine you receive a letter that says "Before reading this letter, fetch the letter in the locked drawer and read it aloud." XXE is the same: the XML parser reads a "DOCTYPE" instruction that says "include the contents of `/etc/passwd`" — and the parser obeys. The fix: disable DTD processing and external entity resolution in the XML parser.

### Purposes

- To prevent attackers from reading local files via XML parsers.
- To prevent SSRF via XML parsers.
- To prevent denial of service (billion laughs attack).
- To comply with OWASP Top 10 (A05:2021 — Security Misconfiguration).

### Syntax Rules and Structure

#### Vulnerable XML (DO NOT PARSE)

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<user>
  <name>&xxe;</name>
</user>
```

#### Vulnerable Code (DO NOT USE)

```typescript
// ❌ VULNERABLE — default parser settings
import libxmljs from 'libxmljs';
const xml = libxmljs.parseXml(req.body.xml); // Resolves external entities
```

#### Safe Parser Configuration

```typescript
// ✅ SAFE — libxmljs with safe options
import libxmljs from 'libxmljs2';

const xml = libxmljs.parseXml(req.body.xml, {
  noent: false,      // Do not substitute entities
  dtdload: false,    // Do not load external DTDs
  dtdattr: false,    // Do not load DTD attributes
  dtdvalid: false,   // Do not validate against DTD
  nonet: true,       // Do not fetch external resources
});

// ✅ SAFE — fast-xml-parser (does not resolve external entities)
import { XMLParser } from 'fast-xml-parser';
const parser = new XMLParser({
  processEntities: false, // Do not process entities
  allowBooleanAttributes: false,
  ignoreAttributes: false,
});
const result = parser.parse(req.body.xml);
```

#### Syntax Rules

- **Disable DTD processing** — `dtdload: false`, `noent: false`.
- **Disable external entity resolution** — `nonet: true`.
- **Use safe parsers** — `fast-xml-parser` does not resolve external entities by default.
- **Validate XML against a schema** — XSD with strict rules.
- **Limit XML size** — prevent billion laughs and XML bombs.
- **Reject DOCTYPE declarations** — if not required.
- **Keep parsers updated** — vulnerabilities are regularly patched.
- **Use JSON instead of XML** — where possible.

#### Constraints and Limitations

- **Some parsers have XXE enabled by default** — must be explicitly disabled.
- **Different parsers have different options** — check the documentation.
- **SOAP and SAML** — use XML heavily; require careful configuration.
- **Legacy systems** — may have hardcoded vulnerable parsers.
- **Billion laughs** — even with entities disabled, large XML can cause DoS.
- **XInclude** — another external resource mechanism to disable.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe XML Parsing (Node.js)

```typescript
import express from 'express';
import { XMLParser } from 'fast-xml-parser';
import libxmljs from 'libxmljs2';

const app = express();
app.use(express.text({ type: 'application/xml', limit: '1mb' }));

// ❌ VULNERABLE — libxmljs with default options
app.post('/unsafe-xml', (req, res) => {
  try {
    const xml = libxmljs.parseXml(req.body); // Resolves external entities
    res.json({ parsed: xml.toString() });
  } catch (err) {
    res.status(400).json({ error: 'Invalid XML' });
  }
});

// ✅ SAFE — libxmljs with safe options
app.post('/safe-xml', (req, res) => {
  try {
    const xml = libxmljs.parseXml(req.body, {
      noent: false,
      dtdload: false,
      dtdattr: false,
      dtdvalid: false,
      nonet: true,
    });
    res.json({ parsed: xml.toString() });
  } catch (err) {
    res.status(400).json({ error: 'Invalid XML' });
  }
});

// ✅ SAFE — fast-xml-parser (does not resolve external entities)
const parser = new XMLParser({
  processEntities: false,
  allowBooleanAttributes: false,
  ignoreAttributes: false,
  parseAttributeValue: true,
  parseTagValue: true,
});

app.post('/safe-xml-2', (req, res) => {
  try {
    const result = parser.parse(req.body);
    res.json(result);
  } catch (err) {
    res.status(400).json({ error: 'Invalid XML' });
  }
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `/unsafe-xml` with an XXE payload reads `/etc/passwd` and includes it in the response (vulnerable).
- `/safe-xml` with the same payload does not resolve the external entity.
- `/safe-xml-2` with the same payload does not resolve the external entity.

**Why this works:** `libxmljs` with `noent: false` and `nonet: true` disables external entity resolution. `fast-xml-parser` does not resolve external entities by default. Both prevent XXE.

### Real-World Cases

- **SOAP APIs:** XML-based web services are frequent targets.
- **SAML authentication:** XML-based SSO assertions.
- **RSS/Atom feeds:** XML parsing of external feeds.
- **Office documents:** DOCX, XLSX, and PPTX are ZIP files containing XML.
- **Notable breaches:** Facebook (2014), PayPal (2013), various enterprise SOAP APIs.

---

## References

- OWASP Top 10 — https://owasp.org/www-project-top-ten/
- OWASP Top 10 — A01:2021 Broken Access Control — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- OWASP Top 10 — A03:2021 Injection — https://owasp.org/Top10/A03_2021-Injection/
- OWASP Top 10 — A05:2021 Security Misconfiguration — https://owasp.org/Top10/A05_2021-Security_Misconfiguration/
- OWASP Top 10 — A08:2021 Software and Data Integrity Failures — https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/
- OWASP Top 10 — A10:2021 Server-Side Request Forgery — https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/
- OWASP Cheat Sheet Series — SQL Injection Prevention — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cross Site Scripting Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cross-Site Request Forgery Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Server-Side Request Forgery Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Path Traversal — https://cheatsheetseries.owasp.org/cheatsheets/Path_Traversal_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Deserialization — https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
- OWASP Cheat Sheet Series — XML External Entity Prevention — https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Access Control — https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Input Validation — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- CWE Top 25 — https://cwe.mitre.org/top25/
- CWE-89 — SQL Injection — https://cwe.mitre.org/data/definitions/89.html
- CWE-79 — Cross-Site Scripting — https://cwe.mitre.org/data/definitions/79.html
- CWE-352 — Cross-Site Request Forgery — https://cwe.mitre.org/data/definitions/352.html
- CWE-918 — Server-Side Request Forgery — https://cwe.mitre.org/data/definitions/918.html
- CWE-22 — Path Traversal — https://cwe.mitre.org/data/definitions/22.html
- CWE-284 — Improper Access Control — https://cwe.mitre.org/data/definitions/284.html
- CWE-502 — Deserialization of Untrusted Data — https://cwe.mitre.org/data/definitions/502.html
- CWE-611 — Improper Restriction of XML External Entity Reference — https://cwe.mitre.org/data/definitions/611.html
- PortSwigger Web Security Academy — https://portswigger.net/web-security
- PortSwigger — SQL Injection — https://portswigger.net/web-security/sql-injection
- PortSwigger — Cross-Site Scripting — https://portswigger.net/web-security/cross-site-scripting
- PortSwigger — CSRF — https://portswigger.net/web-security/csrf
- PortSwigger — SSRF — https://portswigger.net/web-security/ssrf
- PortSwigger — Path Traversal — https://portswigger.net/web-security/file-path-traversal
- PortSwigger — Access Control — https://portswigger.net/web-security/access-control
- PortSwigger — Insecure Deserialization — https://portswigger.net/web-security/deserialization
- PortSwigger — XXE — https://portswigger.net/web-security/xxe
- DOMPurify — npm package — https://www.npmjs.com/package/dompurify
- Helmet — npm package — https://www.npmjs.com/package/helmet
- csurf — npm package — https://www.npmjs.com/package/csurf
- ipaddr.js — npm package — https://www.npmjs.com/package/ipaddr.js
- fast-xml-parser — npm package — https://www.npmjs.com/package/fast-xml-parser
- libxmljs2 — npm package — https://www.npmjs.com/package/libxmljs2
- Zod — npm package — https://www.npmjs.com/package/zod