# Common Web Attacks — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Common web attacks are a class of security vulnerabilities that exploit how web applications handle untrusted input, manage user sessions, or interact with external systems, enabling attackers to execute unauthorised actions, steal data, or disrupt services.

**Technical Definition:** Web attacks exploit vulnerabilities in the application layer (OWASP Top 10:2021 A01–A10), including injection flaws (A03), broken access control (A01), cryptographic failures (A02), insecure design (A04), security misconfiguration (A05), vulnerable components (A06), authentication failures (A07), data integrity failures (A08), logging failures (A09), and server-side request forgery (A10). Each category has distinct attack vectors, exploitation techniques, and mitigation strategies grounded in input validation, output encoding, secure session management, and least-privilege principles.

**Beginner-Friendly Explanation:** Think of a web application as a shop with many doors and windows. Common web attacks are the different ways a thief might try to get in: some try to trick the shopkeeper into handing over the keys (social engineering and CSRF), some climb through a window you forgot to lock (injection), some steal your identity badge (session attacks), and some just keep rattling the doorknob until it opens (brute force). Each type of attack requires a different type of lock, alarm, or guard.

### Key Characteristics

- **Input-driven:** Most attacks begin with untrusted input — URL parameters, query strings, request bodies, headers, or cookies.
- **Defence-in-depth required:** No single mitigation is sufficient; layered defences (validation, encoding, rate limiting, session hardening) are necessary.
- **Framework-agnostic principles:** The same attack classes affect Node.js, Python, Java, PHP, and .NET applications, with technology-specific implementation details.
- **Asymmetric impact:** A small coding mistake can lead to full database compromise, account takeover, or remote code execution.
- **Evolving landscape:** New attack variants (e.g., SSRF in cloud metadata services, ReDoS in regex engines) emerge as architectures evolve.

### Prerequisites

- **Basic web development knowledge:** HTTP methods, headers, cookies, sessions.
- **JavaScript/Node.js fundamentals:** `child_process`, async/await, Express middleware.
- **Understanding of databases:** SQL and NoSQL query concepts.
- **Familiarity with OWASP Top 10:** The canonical list of web application security risks.

### Related Programming Areas

- **Authentication and authorisation:** Password storage, session management, MFA.
- **Cryptography:** Hashing (Argon2id, bcrypt), signing, encryption.
- **Network security:** Firewalls, reverse proxies, TLS termination.
- **Secure coding:** Input validation, output encoding, parameterised queries.
- **Monitoring and logging:** Detecting and responding to attacks.

### Core Concepts

1. **SQL & NoSQL Injection** — parameterised queries, operator injection, dynamic query security.
2. **Cross-Site Scripting (XSS)** — Stored, Reflected, and DOM-based XSS mitigation.
3. **Cross-Site Request Forgery (CSRF)** — double-submit cookies, state-changing GET, SPA considerations.
4. **Brute Force & Denial of Service (DoS)** — rate limiting, credential stuffing, ReDoS.
5. **Credential & Session Attacks** — session fixation, hijacking, password hashing with Argon2id/bcrypt.
6. **SSRF & Command Injection** — SSRF prevention, safe `child_process` usage.

---

## Core Concept 1: SQL & NoSQL Injection

### Definitions

**Core Definition:** Injection attacks occur when untrusted user input is interpreted as part of a query or command, allowing an attacker to alter the query's logic, bypass authentication, or extract data.

**Technical Definition:** SQL injection exploits applications that concatenate user input directly into SQL query strings, enabling attackers to inject SQL keywords, operators, or UNION clauses. NoSQL injection (specifically MongoDB operator injection) exploits the fact that MongoDB queries are JavaScript objects — when user input is passed directly as a query value, an attacker can submit `{$gt: ""}` or `{$ne: null}` to replace an equality check with a comparison operator that matches many documents.

**Beginner-Friendly Explanation:** Imagine a librarian who takes your request for a book and builds a sentence: "Find me the book called [your request]." If you say "Harry Potter," you get Harry Potter. But if you say "Harry Potter OR anything else," the librarian brings you the entire library. Injection attacks are the database equivalent — injecting extra instructions into a query to change what it does.

### Purposes

- To prevent attackers from bypassing authentication and accessing unauthorised data.
- To ensure that user input is always treated as data, never as executable query logic.
- To protect database integrity from malicious modification or deletion.
- To prevent data exfiltration through UNION-based or blind injection techniques.

### Sub-Feature 1.1: SQL Injection — Parameterised Query Bypasses

#### Definitions

**Core Definition:** Parameterised queries (prepared statements) separate SQL code from data, ensuring that user input is always treated as a value, never as executable SQL.

**Technical Definition:** A parameterised query sends the SQL template and the parameter values to the database separately. The database compiles the query plan first and then binds the parameters, so the parameter values cannot alter the query structure. String concatenation and manual escaping are the primary causes of SQL injection; parameterisation is the primary defence.

**Beginner-Friendly Explanation:** Instead of building a sentence by pasting words together ("Find me the book called " + user_input), parameterised queries use a template with blanks: "Find me the book called [BLANK]." You fill in the blank separately, and the database knows the blank is just a value, not a command.

#### Purposes

- To prevent SQL injection by structurally separating code from data.
- To enable database engines to cache query plans for better performance.
- To eliminate the need for manual escaping, which is error-prone.

#### Syntax Rules and Structure

```js
// Parameterised query (SQLite example)
const stmt = db.prepare('SELECT * FROM users WHERE email = ? AND password_hash = ?');
const user = stmt.get(email, passwordHash);

// Parameterised query (MySQL example)
const [rows] = await connection.execute(
  'SELECT * FROM users WHERE email = ? AND password_hash = ?',
  [email, passwordHash]
);
```

| Component | Breakdown |
|-----------|-----------|
| `?` | Placeholder for the parameter value. |
| `[email, passwordHash]` | Values bound to placeholders in order. |
| `prepare()` | Compiles the query plan; input cannot alter structure. |

**Constraints and Limitations:**
- Table names, column names, and `ORDER BY` directions cannot be parameterised; use allowlist validation for these.
- The `LIKE` operator requires careful handling; parameterise the pattern, not the wildcards.
- ORMs (Sequelize, Knex, Mongoose) provide parameterisation automatically for most operations.

#### Annotated Code Example

```js
// sqli-prevention.js — SQLite parameterised query
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const app = express();
app.use(express.json());

const db = new sqlite3.Database(':memory:');

db.serialize(() => {
  db.run('CREATE TABLE users (id INTEGER PRIMARY KEY, email TEXT, password_hash TEXT)');
  db.run("INSERT INTO users (email, password_hash) VALUES ('alice@example.com', 'hash123')");
});

// SAFE: Parameterised query
app.post('/login', (req, res) => {
  const { email, password } = req.body;

  // ❌ VULNERABLE (never do this):
  // db.get(`SELECT * FROM users WHERE email = '${email}' AND password_hash = '${password}'`, ...)

  // ✅ SAFE: Parameterised query with placeholders
  db.get(
    'SELECT * FROM users WHERE email = ? AND password_hash = ?',
    [email, password],
    (err, row) => {
      if (err) return res.status(500).send('Database error');
      if (row) return res.json({ message: 'Login successful', user: row.email });
      res.status(401).json({ error: 'Invalid credentials' });
    }
  );
});

app.listen(3000, () => console.log('SQLi-safe server on 3000'));
```

**Expected Output (for `POST /login` with `{"email":"alice@example.com","password":"hash123"}`):**
```
{"message":"Login successful","user":"alice@example.com"}
```

**Expected Output (for `POST /login` with `{"email":"alice@example.com' OR '1'='1","password":"anything"}`):**
```
{"error":"Invalid credentials"}
```

**Why this output:** The parameterised query treats the entire email string as a literal value. The injection payload `' OR '1'='1` becomes a literal email address that does not match any row, so the query returns no results and the login fails.

#### Real-World Cases

- **Login endpoints:** Parameterised queries prevent authentication bypass via `' OR '1'='1`.
- **Search functionality:** Parameterising the search term prevents UNION-based data extraction.
- **Data access APIs:** Parameterised queries ensure that user IDs cannot be manipulated to access other users' records.

---

### Sub-Feature 1.2: NoSQL Injection — MongoDB Operator Injection

#### Definitions

**Core Definition:** MongoDB operator injection occurs when user-supplied input is passed directly as a query value, allowing an attacker to substitute comparison operators (e.g., `$gt`, `$ne`) for expected scalar values, altering the query's logic.

**Technical Definition:** MongoDB queries are JavaScript objects. When an application does `User.findOne({ email: req.body.email, password: req.body.password })`, an attacker can submit `password[$ne]=null` in a URL-encoded body, which the `qs` parser converts to `{ password: { $ne: null } }`. This changes the password check from "equals" to "not equal to null," matching the first user whose password is not blank — typically every real account. The `$where` operator is even more dangerous because it executes arbitrary JavaScript on the MongoDB server.

**Beginner-Friendly Explanation:** Imagine a security guard who checks IDs by reading a form that says "Name: _____". Normally you write your name. But if you write "Name: anyone except 'nobody'", the guard will let in almost anyone. MongoDB operator injection works the same way — you replace the expected value with a condition that is broadly true.

#### Purposes

- To prevent attackers from bypassing authentication through operator substitution.
- To ensure that query filters are always constructed from typed, validated values.
- To protect against the more severe `$where` operator, which enables arbitrary JavaScript execution.

#### Syntax Rules and Structure

```js
// SAFE: Validate types before querying
const { username, password } = req.body;
if (typeof username !== 'string' || typeof password !== 'string') {
  return res.status(400).json({ error: 'Invalid input' });
}

// SAFE: Use Mongoose schema validation (casts to declared types)
const userSchema = new mongoose.Schema({
  username: { type: String, required: true },
  password: { type: String, required: true }
});
```

| Technique | Prevention |
|-----------|-----------|
| Type validation | Reject objects where strings are expected. |
| Schema validation | Mongoose casts values to declared types. |
| `mongo-sanitize` | Strips `$`-prefixed keys from `req.body` and `req.query`. |
| Disable `$where` | Disable server-side JavaScript in MongoDB config. |

**Constraints and Limitations:**
- Schema validation is not a complete defence if the application bypasses Mongoose and uses the raw driver.
- The `qs` library is the default query parser in Express; it can be configured to prevent nested objects.
- CVE-2024-53900 and CVE-2025-23061 affected Mongoose's `populate()` method; ensure Mongoose is updated.

#### Annotated Code Example

```js
// nosql-prevention.js — MongoDB operator injection prevention
const express = require('express');
const mongoose = require('mongoose');
const mongoSanitize = require('express-mongo-sanitize');
const app = express();
app.use(express.json());

// ✅ Sanitize $ and . keys from req.body, req.query, req.params
app.use(mongoSanitize());

// ✅ Define a strict schema — Mongoose casts input to declared types
const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  passwordHash: { type: String, required: true }
});
const User = mongoose.model('User', userSchema);

app.post('/login', async (req, res) => {
  const { email, passwordHash } = req.body;

  // ✅ Type validation — reject non-string values
  if (typeof email !== 'string' || typeof passwordHash !== 'string') {
    return res.status(400).json({ error: 'Invalid input type' });
  }

  // ✅ Mongoose will cast email to String, preventing operator injection
  const user = await User.findOne({ email, passwordHash });

  if (!user) return res.status(401).json({ error: 'Invalid credentials' });
  res.json({ message: 'Login successful', email: user.email });
});

// Connect to MongoDB (in-memory example)
mongoose.connect('mongodb://localhost:27017/test')
  .then(() => app.listen(3000, () => console.log('NoSQL-safe server on 3000')))
  .catch(err => console.error(err));
```

**Expected Output (for `POST /login` with `{"email":{"$ne":null},"passwordHash":{"$ne":null}}`):**
```
{"error":"Invalid input type"}
```

**Why this output:** The type validation rejects the request because `email` is an object, not a string. Even without type validation, Mongoose schema casting would convert the object to a string (e.g., `[object Object]`), which would not match any user.

#### Real-World Cases

- **Login bypass:** Attackers submit `password[$ne]=null` to bypass authentication.
- **Data extraction:** `role[$gt]=` is used to find users with roles greater than an empty string, potentially returning admin users.
- **`$where` injection:** Attackers inject `$where: "sleep(5000)"` to cause a time-based blind injection.

---

### Sub-Feature 1.3: Securing Dynamic Queries

#### Definitions

**Core Definition:** Dynamic queries are queries whose structure (table names, column names, sort directions) is determined at runtime based on user input, requiring allowlist validation rather than parameterisation.

**Technical Definition:** While parameterisation handles values safely, SQL identifiers (table names, column names) and query keywords (ASC/DESC) cannot be parameterised in most database drivers. Dynamic queries must validate these elements against an allowlist of known-safe values before constructing the query string.

**Beginner-Friendly Explanation:** Some parts of a query are like the chapter titles in a book — you can't just substitute any words, they must be from a fixed list. If a user wants to sort by "name" or "price," you check that "name" and "price" are in your allowed list before building the query.

#### Purposes

- To allow user-driven sorting and filtering without opening injection vectors.
- To ensure that only pre-approved identifiers are used in query construction.
- To prevent attackers from using `ORDER BY` or `GROUP BY` clauses for injection.

#### Syntax Rules and Structure

```js
const ALLOWED_SORT_COLUMNS = ['name', 'price', 'created_at'];
const ALLOWED_SORT_ORDERS = ['ASC', 'DESC'];

const sortBy = ALLOWED_SORT_COLUMNS.includes(req.query.sortBy)
  ? req.query.sortBy : 'created_at';
const sortOrder = ALLOWED_SORT_ORDERS.includes(req.query.sortOrder?.toUpperCase())
  ? req.query.sortOrder.toUpperCase() : 'DESC';

const query = `SELECT * FROM products ORDER BY ${sortBy} ${sortOrder}`;
```

| Component | Breakdown |
|-----------|-----------|
| `ALLOWED_SORT_COLUMNS` | Frozen array of permitted column names. |
| `includes()` | Returns `true` only if the input is in the allowlist. |
| Fallback | Defaults to a safe column if input is invalid. |

**Constraints and Limitations:**
- Allowlists must be maintained as the schema evolves.
- Never interpolate user input directly into the query string, even for allowlisted values — validate first.

#### Annotated Code Example

```js
// dynamic-query.js — Allowlist validation for dynamic SQL
const express = require('express');
const app = express();

// Frozen allowlists — cannot be modified at runtime
const ALLOWED_SORT = Object.freeze(['name', 'price', 'created_at']);
const ALLOWED_ORDER = Object.freeze(['ASC', 'DESC']);

app.get('/products', (req, res) => {
  const sortBy = ALLOWED_SORT.includes(req.query.sortBy)
    ? req.query.sortBy : 'name';
  const sortOrder = ALLOWED_ORDER.includes(req.query.sortOrder?.toUpperCase())
    ? req.query.sortOrder.toUpperCase() : 'ASC';

  // Safe: both values validated against allowlists
  const query = `SELECT id, name, price FROM products ORDER BY ${sortBy} ${sortOrder}`;
  console.log('Query:', query);

  res.json({ query, sortBy, sortOrder });
});

app.listen(3000, () => console.log('Dynamic query server on 3000'));
```

**Expected Output (for `GET /products?sortBy=price&sortOrder=DESC`):**
```
{"query":"SELECT id, name, price FROM products ORDER BY price DESC","sortBy":"price","sortOrder":"DESC"}
```

**Expected Output (for `GET /products?sortBy=id;DROP TABLE products--`):**
```
{"query":"SELECT id, name, price FROM products ORDER BY name ASC","sortBy":"name","sortOrder":"ASC"}
```

**Why this output:** The injection payload `id;DROP TABLE products--` is not in the `ALLOWED_SORT` array, so it falls back to the safe default `name`. The attacker cannot control the query structure.

#### Real-World Cases

- **E-commerce sorting:** Letting users sort products by name, price, or date without injection risk.
- **Admin dashboards:** Allowing administrators to filter data by predefined columns.
- **Reporting tools:** Dynamically grouping by validated dimensions.

---

## Core Concept 2: Cross-Site Scripting (XSS)

### Definitions

**Core Definition:** Cross-Site Scripting (XSS) is a vulnerability that allows an attacker to inject malicious JavaScript into a web page viewed by other users, enabling session hijacking, credential theft, or defacement.

**Technical Definition:** XSS occurs when an application includes untrusted data in its output without proper context-aware encoding. Three main types exist: **Stored XSS** (malicious script is persisted in the database and served to all users), **Reflected XSS** (malicious script is included in the immediate response from a crafted request), and **DOM-based XSS** (malicious script is executed by client-side JavaScript that reads untrusted data and writes it to the DOM). Context-aware output encoding — applying the correct encoding (HTML entity, HTML attribute, JavaScript, URL) based on where the data appears in the document — is the primary defence.

**Beginner-Friendly Explanation:** Imagine a guestbook at a hotel where anyone can write a message. If someone writes "Check out this great deal!" but also secretly writes "By the way, give your room key to the next person who reads this," that hidden instruction is like an XSS payload. When other guests read the guestbook, they follow the hidden instruction. XSS prevention ensures that any message written in the guestbook is treated as plain text, not as instructions.

### Purposes

- To prevent attackers from executing arbitrary JavaScript in victims' browsers.
- To protect session cookies, credentials, and sensitive data from theft.
- To maintain the integrity and trustworthiness of the application's user interface.
- To comply with OWASP Top 10 requirements for injection prevention.

### Sub-Feature 2.1: Context-Aware Output Encoding

#### Definitions

**Core Definition:** Context-aware output encoding is the process of transforming untrusted data into a safe representation based on the specific context (HTML body, HTML attribute, JavaScript, URL) in which it will be inserted.

**Technical Definition:** Different contexts require different encoding rules. In an HTML body context, `<`, `>`, `&`, `"`, and `'` must be HTML-entity encoded. In an HTML attribute context, the same characters must be encoded, plus consideration for attribute-breakout. In a JavaScript context, characters that can break out of a string literal or introduce new statements must be Unicode-escaped. In a URL context, all characters except unreserved characters must be percent-encoded.

**Beginner-Friendly Explanation:** If you're writing a message on a whiteboard, you use a marker. If you're writing on a glass window, you use a different type of pen. Context-aware encoding means using the right "pen" for the right surface — HTML encoding for HTML, JavaScript encoding for JavaScript, and URL encoding for URLs.

#### Purposes

- To neutralise malicious payloads by ensuring they are rendered as inert text, not executable code.
- To prevent XSS in all rendering contexts (HTML body, attributes, script blocks, URLs).
- To provide a consistent, framework-independent defence against injection.

#### Syntax Rules and Structure

```js
// HTML entity encoding (for HTML body context)
function htmlEncode(str) {
  return str.replace(/[&<>"']/g, (char) => ({
    '&': '&amp;', '<': '&lt;', '>': '&gt;',
    '"': '&quot;', "'": '&#x27;'
  }[char]));
}

// JavaScript encoding (for inline script context)
function jsEncode(str) {
  return str.replace(/[^a-zA-Z0-9]/g, (char) =>
    '\\x' + char.charCodeAt(0).toString(16).padStart(2, '0')
  );
}

// URL encoding
const urlEncoded = encodeURIComponent(userInput);
```

| Context | Encoding Type | Characters Encoded |
|---------|--------------|-------------------|
| HTML body | HTML entity | `< > & " '` |
| HTML attribute | HTML entity | `< > & " '` plus attribute-specific |
| JavaScript string | JavaScript escape | Non-alphanumeric |
| URL parameter | Percent encoding | All non-unreserved |

**Constraints and Limitations:**
- Encoding must be applied **at the point of output**, not at input, because the same data may be used in multiple contexts.
- Never use blacklist filtering alone; encoding is the primary defence.
- Rich-text content (e.g., from WYSIWYG editors) requires HTML sanitisation (DOMPurify) rather than simple encoding.

#### Annotated Code Example

```js
// xss-prevention.js — Context-aware encoding
const express = require('express');
const app = express();

// HTML entity encoding function
function escapeHtml(str) {
  const map = { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#x27;' };
  return String(str).replace(/[&<>"']/g, (ch) => map[ch]);
}

app.get('/search', (req, res) => {
  const query = req.query.q || '';

  // ✅ SAFE: HTML-encode before inserting into HTML body
  const safeQuery = escapeHtml(query);

  res.send(`
    <!DOCTYPE html>
    <html>
      <body>
        <h1>Search results for: ${safeQuery}</h1>
        <p>You searched for "${safeQuery}"</p>
      </body>
    </html>
  `);
});

app.listen(3000, () => console.log('XSS-safe server on 3000'));
```

**Expected Output (for `GET /search?q=<script>alert(1)</script>`):**
```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Search results for: &lt;script&gt;alert(1)&lt;/script&gt;</h1>
    <p>You searched for "&lt;script&gt;alert(1)&lt;/script&gt;"</p>
  </body>
</html>
```

**Why this output:** The `<` and `>` characters in the script tag are HTML-entity encoded to `&lt;` and `&gt;`. The browser renders these as literal text rather than interpreting them as HTML tags. The script never executes.

#### Real-World Cases

- **Comment sections:** Stored XSS in blog comments can affect every visitor.
- **Search results:** Reflected XSS in search pages can steal session cookies via crafted links.
- **Profile pages:** User-supplied names, bios, or avatars are common XSS vectors.

---

### Sub-Feature 2.2: Mitigating Stored, Reflected, and DOM-based XSS

#### Definitions

**Core Definition:** The three types of XSS differ in how the malicious payload reaches the victim: Stored XSS persists in the database, Reflected XSS is embedded in the request, and DOM-based XSS is executed entirely in the browser by client-side JavaScript.

**Technical Definition:** Stored XSS requires encoding **at output** for all users who view the stored data. Reflected XSS requires encoding **at output** for the immediate response. DOM-based XSS requires safe JavaScript APIs (e.g., `textContent` instead of `innerHTML`) or sanitisation libraries when dynamic DOM manipulation is necessary.

**Beginner-Friendly Explanation:** Stored XSS is like writing graffiti on a wall that everyone sees. Reflected XSS is like writing graffiti on a mirror that only the person looking at it sees. DOM-based XSS is like giving someone a set of instructions that they follow in their own home — the dangerous code never touches the server.

#### Purposes

- To prevent stored payloads from executing for any user who views the data.
- To prevent reflected payloads from executing when a crafted link is clicked.
- To prevent DOM-based payloads from executing via unsafe client-side JavaScript.

#### Syntax Rules and Structure

```js
// ❌ UNSAFE: innerHTML with untrusted data
element.innerHTML = userInput;

// ✅ SAFE: textContent (renders as text, not HTML)
element.textContent = userInput;

// ✅ SAFE: DOMPurify sanitisation for rich text
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

| XSS Type | Location of Payload | Defence |
|----------|-------------------|---------|
| Stored | Database | Output encoding on every render |
| Reflected | Request URL/body | Output encoding on immediate response |
| DOM-based | Client-side JS | Safe DOM APIs (textContent, createElement) |

**Constraints and Limitations:**
- `innerHTML` is never safe with untrusted data, even after client-side encoding.
- DOMPurify must be configured with an allowlist of permitted tags and attributes.
- Frameworks like React and Vue automatically escape text interpolation, but `dangerouslySetInnerHTML` and `v-html` bypass this protection.

#### Annotated Code Example

```js
// xss-dom-prevention.js — Safe DOM manipulation (client-side)
// This would run in the browser

// ❌ VULNERABLE: innerHTML executes scripts
// document.getElementById('output').innerHTML = location.hash.slice(1);

// ✅ SAFE: textContent renders as text
const userInput = new URLSearchParams(window.location.search).get('name') || '';
document.getElementById('output').textContent = userInput;

// ✅ SAFE: createElement + textContent for dynamic DOM
const div = document.createElement('div');
div.textContent = userInput;
document.getElementById('container').appendChild(div);

// ✅ SAFE: DOMPurify for rich text (if HTML must be rendered)
// import DOMPurify from 'dompurify';
// element.innerHTML = DOMPurify.sanitize(userInput);
```

**Expected Output (when URL contains `?name=<img src=x onerror=alert(1)>`):**
```
The page displays the literal text: <img src=x onerror=alert(1)>
No alert is triggered.
```

**Why this output:** `textContent` treats the entire string as text. The `<img>` tag is not parsed as HTML, so the `onerror` event never fires.

#### Real-World Cases

- **Stored XSS in CMS:** A blog post containing a malicious script executes for every reader.
- **Reflected XSS in error messages:** An error page that reflects the invalid input without encoding.
- **DOM-based XSS in SPAs:** Client-side routers that read URL fragments and inject them into the DOM.

---

## Core Concept 3: Cross-Site Request Forgery (CSRF)

### Definitions

**Core Definition:** Cross-Site Request Forgery (CSRF) is an attack that tricks an authenticated user's browser into performing an unwanted action on a trusted site where the user is currently authenticated.

**Technical Definition:** CSRF exploits the fact that browsers automatically include cookies (including session cookies) with every request to a domain. If a state-changing endpoint relies solely on cookie-based authentication and does not verify the request's origin or intent, an attacker can craft a malicious page that submits a form or sends a fetch request to that endpoint on behalf of the victim. Defences include the synchroniser token pattern, double-submit cookies, SameSite cookie attributes, and Fetch Metadata headers.

**Beginner-Friendly Explanation:** Imagine you're logged into your bank's website. An attacker sends you an email with a link to "free kittens." When you click it, the page secretly submits a form to your bank's website — transferring money to the attacker — using the session cookie your browser automatically includes. Because you're already logged in, the bank thinks you authorised the transfer.

### Purposes

- To prevent attackers from executing state-changing actions on behalf of authenticated users.
- To ensure that every state-changing request includes proof of user intent.
- To protect against both traditional form-based CSRF and modern fetch-based CSRF.

### Sub-Feature 3.1: Double-Submit Cookie Pattern

#### Definitions

**Core Definition:** The double-submit cookie pattern generates a random token, stores it in a cookie, and requires the client to echo the same token in a request header or body; the server validates that both values match.

**Technical Definition:** In the signed double-submit cookie pattern, the server generates a cryptographically random token, signs it with an HMAC using a server-side secret, and sets it as a cookie (with `HttpOnly=false` so JavaScript can read it). The client reads the cookie and includes the token in a custom header (e.g., `X-CSRF-Token`) for every state-changing request. The server validates the signature and checks that the header value matches the cookie value. Because an attacker on a different origin cannot read the cookie (due to Same-Origin Policy) nor set custom headers via a simple form submission, the attack fails.

**Beginner-Friendly Explanation:** Think of a bank that gives you a special stamp when you open an account. To make a withdrawal, you must bring both your account card (the cookie) and the stamp (the token in the header). An attacker can steal your card by tricking you into visiting their website, but they can't get the stamp because it's locked in your browser's same-origin vault.

#### Purposes

- To provide CSRF protection without server-side token storage.
- To work effectively with single-page applications (SPAs) that use fetch/axios.
- To tie tokens to the user's authenticated session via HMAC signing.

#### Syntax Rules and Structure

```js
// Server: generate and set CSRF token cookie
const crypto = require('node:crypto');

function generateCsrfToken(secret) {
  const token = crypto.randomBytes(32).toString('hex');
  const signature = crypto.createHmac('sha256', secret).update(token).digest('hex');
  return `${token}.${signature}`;
}

// Set cookie
res.cookie('csrf_token', token, {
  httpOnly: false,   // Must be readable by JavaScript
  sameSite: 'lax',
  secure: true
});
```

| Component | Breakdown |
|-----------|-----------|
| `crypto.randomBytes(32)` | Cryptographically random token. |
| HMAC signature | Ties token to server secret; prevents forgery. |
| `httpOnly: false` | Allows client JavaScript to read the cookie. |
| `sameSite: 'lax'` | Limits cross-origin cookie sending. |

**Constraints and Limitations:**
- Requires JavaScript to read the cookie and set the header; for non-JS forms, use a synchroniser token instead.
- XSS can defeat double-submit cookies because the attacker can read the cookie.
- Subdomain cookie injection can defeat the pattern if not carefully configured.

#### Annotated Code Example

```js
// csrf-double-submit.js
const express = require('express');
const crypto = require('node:crypto');
const cookieParser = require('cookie-parser');
const app = express();
app.use(cookieParser());
app.use(express.json());

const CSRF_SECRET = process.env.CSRF_SECRET || 'change-me-in-production';

// Endpoint to issue CSRF token
app.get('/csrf-token', (req, res) => {
  const token = crypto.randomBytes(32).toString('hex');
  const signature = crypto.createHmac('sha256', CSRF_SECRET)
    .update(token).digest('hex');
  const signedToken = `${token}.${signature}`;

  res.cookie('csrf_token', signedToken, {
    httpOnly: false,   // JS must read it
    sameSite: 'lax',
    secure: process.env.NODE_ENV === 'production'
  });
  res.json({ csrfToken: signedToken });
});

// Middleware to validate CSRF on state-changing requests
function validateCsrf(req, res, next) {
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) return next();

  const cookieToken = req.cookies.csrf_token;
  const headerToken = req.headers['x-csrf-token'];

  if (!cookieToken || !headerToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: 'CSRF token mismatch' });
  }

  // Verify HMAC signature
  const [token, signature] = cookieToken.split('.');
  const expected = crypto.createHmac('sha256', CSRF_SECRET)
    .update(token).digest('hex');
  if (!crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected))) {
    return res.status(403).json({ error: 'Invalid CSRF signature' });
  }

  next();
}

app.post('/transfer', validateCsrf, (req, res) => {
  res.json({ message: 'Transfer completed', amount: req.body.amount });
});

app.listen(3000, () => console.log('CSRF-safe server on 3000'));
```

**Expected Output (for `POST /transfer` without CSRF header):**
```
{"error":"CSRF token mismatch"}
```

**Expected Output (for `POST /transfer` with valid CSRF cookie and header):**
```
{"message":"Transfer completed","amount":100}
```

**Why this output:** The attacker's page cannot read the CSRF cookie (same-origin policy prevents it) and cannot set a custom `X-CSRF-Token` header on a simple form submission. Therefore, the request fails validation.

#### Real-World Cases

- **Banking transfers:** CSRF protection ensures that a malicious page cannot initiate a transfer.
- **Email settings:** CSRF can change a victim's email forwarding rules to exfiltrate messages.
- **Password changes:** CSRF can change a victim's password, locking them out of their account.

---

### Sub-Feature 3.2: State-Changing GET Requests and SPA Considerations

#### Definitions

**Core Definition:** State-changing GET requests are endpoints that modify server state despite using the GET method (which is intended to be safe/idempotent); they are particularly vulnerable to CSRF because they can be triggered by simple `<img>` or `<script>` tags. SPAs require special CSRF consideration because they use fetch/axios rather than traditional form submissions.

**Technical Definition:** According to HTTP semantics (RFC 9110), GET, HEAD, OPTIONS, and TRACE are "safe" methods that should not change server state. Applications that violate this (e.g., `GET /delete-user?id=123`) are trivially exploitable via CSRF. For SPAs, the double-submit cookie pattern works well because the client can read the cookie via JavaScript and include it in a custom header, which the browser's same-origin policy prevents attackers from forging.

**Beginner-Friendly Explanation:** A state-changing GET is like a door that opens when you knock — anyone who knows the knock can open it, even if they're not supposed to. An SPA is like a modern office with electronic locks; the CSRF protection is the badge reader that checks both your ID card (cookie) and your fingerprint (custom header).

#### Purposes

- To ensure that only state-changing HTTP methods (POST, PUT, PATCH, DELETE) are used for modifications.
- To protect SPAs using the double-submit cookie pattern with custom headers.
- To leverage Fetch Metadata headers (`Sec-Fetch-Site`) as an additional layer for modern browsers.

#### Syntax Rules and Structure

```js
// ❌ BAD: State-changing GET
app.get('/delete-user', (req, res) => { /* deletes user */ });

// ✅ GOOD: State-changing POST with CSRF protection
app.post('/delete-user', validateCsrf, (req, res) => { /* deletes user */ });

// SPA: Read CSRF token from cookie and send in header
// const csrfToken = document.cookie.match(/csrf_token=([^;]+)/)?.[1];
// fetch('/api/transfer', {
//   method: 'POST',
//   headers: { 'X-CSRF-Token': csrfToken, 'Content-Type': 'application/json' },
//   body: JSON.stringify({ amount: 100 })
// });
```

| Method | Safe? | CSRF Risk | Mitigation |
|--------|-------|-----------|-----------|
| GET | Yes | Low (if no state change) | Never change state on GET |
| POST | No | High | CSRF token or SameSite cookie |
| PUT/PATCH | No | High | CSRF token or SameSite cookie |
| DELETE | No | High | CSRF token or SameSite cookie |

**Constraints and Limitations:**
- Some legacy applications use GET for state changes; these must be refactored or protected.
- `SameSite=Lax` is the default in modern browsers and blocks most CSRF for cross-site POST requests, but it is not a complete defence.
- Fetch Metadata headers are not supported by all browsers; use them as a supplementary defence.

#### Annotated Code Example

```js
// csrf-spa.js — SPA-compatible CSRF protection
const express = require('express');
const crypto = require('node:crypto');
const app = express();
app.use(express.json());

// In-memory store for demonstration
const users = [{ id: 1, name: 'Alice', email: 'alice@example.com' }];

// Issue CSRF token
app.get('/api/csrf', (req, res) => {
  const token = crypto.randomBytes(32).toString('hex');
  res.cookie('csrf', token, { httpOnly: false, sameSite: 'strict' });
  res.json({ csrfToken: token });
});

// CSRF validation for state-changing methods only
app.use((req, res, next) => {
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) return next();
  const cookieToken = req.cookies?.csrf;
  const headerToken = req.headers['x-csrf-token'];
  if (!cookieToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }
  next();
});

// State-changing route uses POST, not GET
app.post('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'User not found' });
  user.name = req.body.name || user.name;
  res.json(user);
});

app.listen(3000, () => console.log('SPA CSRF server on 3000'));
```

**Expected Output (for `POST /api/users/1` without CSRF header):**
```
{"error":"CSRF validation failed"}
```

**Why this output:** The SPA must first call `GET /api/csrf` to obtain the token, then include it in the `X-CSRF-Token` header of the state-changing request. An attacker cannot read the cookie or set the header from a different origin.

#### Real-World Cases

- **REST APIs for SPAs:** React, Vue, and Angular applications use fetch/axios with CSRF headers.
- **Mobile app backends:** Token-based authentication (JWT) avoids CSRF entirely because the token is not sent automatically.
- **Legacy form-based apps:** Synchroniser tokens are more appropriate for server-rendered forms.

---

## Core Concept 4: Brute Force & Denial of Service (DoS)

### Definitions

**Core Definition:** Brute force attacks systematically try many credential combinations to gain unauthorised access, while Denial of Service (DoS) attacks aim to exhaust server resources (CPU, memory, connections) to make the application unavailable.

**Technical Definition:** Credential stuffing uses leaked username/password pairs from other breaches to attempt login on a target site. Distributed brute force attacks use botnets or proxy chains to distribute attempts across many IP addresses, defeating simple IP-based rate limiting. Algorithmic DoS (ReDoS) exploits catastrophic backtracking in regular expressions, causing exponential CPU consumption with a crafted input. Heavy payload attacks send large request bodies or deeply nested JSON to exhaust memory or parsing CPU.

**Beginner-Friendly Explanation:** Brute force is like a thief trying every key on a keyring until one opens your door. Credential stuffing is like using keys stolen from other houses — they might not fit, but some will. DoS is like a crowd of people all trying to enter your shop at once, blocking legitimate customers from getting in.

### Purposes

- To prevent automated attacks that attempt to guess credentials or OTP codes.
- To protect server resources from exhaustion by malicious or accidental traffic spikes.
- To detect and block distributed attacks that span many IP addresses.
- To ensure that login and registration endpoints remain available for legitimate users.

### Sub-Feature 4.1: Rate Limiting and Credential Stuffing Prevention

#### Definitions

**Core Definition:** Rate limiting restricts the number of requests a client can make within a time window, mitigating brute force and credential stuffing attacks.

**Technical Definition:** Effective rate limiting combines per-IP, per-account, and global limits. Per-IP limits alone are insufficient against distributed attacks; per-account limits (e.g., 5 failed login attempts per account per minute) prevent targeted brute force even when the attacker rotates IPs. Progressive delays (exponential backoff) and CAPTCHA challenges further raise the cost of automated attacks. The `express-rate-limit` package provides a standard implementation for Node.js.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer at a club who lets in only a certain number of people per minute. If someone keeps trying to get in after being rejected, the bouncer makes them wait longer each time. This stops someone from trying a thousand different fake IDs in one night.

#### Purposes

- To slow down or block automated credential guessing.
- To prevent resource exhaustion from high-volume requests.
- To protect authentication, registration, password reset, and OTP endpoints.
- To raise the economic cost of credential stuffing campaigns.

#### Syntax Rules and Structure

```js
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 60 * 1000,       // 1 minute
  max: 5,                     // 5 requests per window per IP
  standardHeaders: true,      // Return rate limit info in headers
  legacyHeaders: false,       // Disable X-RateLimit-* headers
  message: { error: 'Too many attempts. Try again later.' },
  keyGenerator: (req) => req.ip  // Default: IP address
});
```

| Component | Breakdown |
|-----------|-----------|
| `windowMs` | Time window in milliseconds. |
| `max` | Maximum requests allowed per window. |
| `keyGenerator` | Function returning the key to rate-limit by (IP, account, etc.). |
| `standardHeaders` | Includes `RateLimit-*` headers in the response. |

**Constraints and Limitations:**
- IP-based limiting can be bypassed by distributed attacks; combine with per-account limits.
- Legitimate users behind NAT may share an IP and be incorrectly limited.
- In-memory rate limiting does not work across multiple server instances; use Redis for distributed deployments.

#### Annotated Code Example

```js
// rate-limit.js — Multi-layer rate limiting
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();
app.use(express.json());

// Per-IP rate limiter for login
const loginLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: { error: 'Too many login attempts from this IP' },
  standardHeaders: true,
  legacyHeaders: false
});

// Per-account rate limiter (in-memory for demonstration)
const accountAttempts = new Map();
function accountLimiter(req, res, next) {
  const email = req.body.email;
  if (!email) return next();

  const now = Date.now();
  const record = accountAttempts.get(email) || { count: 0, resetAt: now + 60000 };

  if (now > record.resetAt) {
    record.count = 0;
    record.resetAt = now + 60000;
  }

  record.count++;
  accountAttempts.set(email, record);

  if (record.count > 5) {
    return res.status(429).json({ error: 'Account temporarily locked' });
  }
  next();
}

app.post('/login', loginLimiter, accountLimiter, (req, res) => {
  // Simulate login logic
  res.json({ message: 'Login endpoint' });
});

app.listen(3000, () => console.log('Rate-limited server on 3000'));
```

**Expected Output (for 6 rapid `POST /login` requests with the same email):**
```
{"error":"Account temporarily locked"}
```

**Why this output:** The per-account limiter increments a counter for the email. After 5 attempts within the 60-second window, the 6th request is rejected with a 429 status.

#### Real-World Cases

- **Login endpoints:** Preventing credential stuffing by limiting attempts per account and per IP.
- **Password reset:** Rate limiting password reset requests to prevent email flooding.
- **OTP verification:** Limiting OTP attempts to prevent brute force of 6-digit codes.

---

### Sub-Feature 4.2: Algorithmic DoS — ReDoS and Heavy Payloads

#### Definitions

**Core Definition:** Algorithmic Denial of Service (ReDoS) exploits regular expressions with catastrophic backtracking, causing exponential CPU consumption for certain inputs. Heavy payload attacks send large or deeply nested request bodies to exhaust memory or parsing CPU.

**Technical Definition:** ReDoS occurs when a regex pattern contains nested quantifiers (e.g., `(a+)+$`) that, for certain non-matching inputs, cause the regex engine to explore an exponential number of backtracking paths. Node.js's regex engine (V8's Irregexp) is vulnerable to this. Heavy payload attacks exploit body parsers that accept unbounded JSON; deeply nested objects (e.g., `{"a":{"a":{"a":...}}}`) can cause stack overflow or excessive CPU. Express's `express.json()` accepts a `limit` option to bound body size.

**Beginner-Friendly Explanation:** ReDoS is like asking someone to solve a puzzle that seems simple but has millions of possible combinations — they'll be stuck for a very long time. Heavy payload attacks are like sending a letter so heavy that the mailroom collapses under its weight.

#### Purposes

- To prevent regex-based endpoints from being taken down by crafted input.
- To bound the resources consumed by request parsing.
- To protect against memory exhaustion from large or deeply nested payloads.

#### Syntax Rules and Structure

```js
// Bound body size
app.use(express.json({ limit: '100kb' }));

// Avoid catastrophic backtracking patterns
// ❌ BAD: /(a+)+$/
// ✅ GOOD: /^a+$/

// Use regex timeout libraries or safe-regex
const safeRegex = require('safe-regex');
if (!safeRegex(userPattern)) {
  return res.status(400).json({ error: 'Invalid pattern' });
}
```

| Mitigation | Implementation |
|------------|---------------|
| Body size limit | `express.json({ limit: '100kb' })` |
| Safe regex | `safe-regex` package or manual review |
| Input length limit | Reject inputs exceeding a maximum length |
| Timeout | Use `AbortController` for long-running operations |

**Constraints and Limitations:**
- `safe-regex` may produce false positives/negatives; manual review is recommended.
- Node.js does not support regex timeouts natively; consider using a worker thread with a timeout.
- Deeply nested JSON can cause stack overflow during parsing; limit depth.

#### Annotated Code Example

```js
// redos-prevention.js — Bound payload size and safe regex
const express = require('express');
const safeRegex = require('safe-regex');
const app = express();

// ✅ Bound body size to prevent heavy payload DoS
app.use(express.json({ limit: '50kb' }));

// ✅ Validate regex patterns before using them
app.get('/validate', (req, res) => {
  const pattern = req.query.pattern;
  if (!pattern || typeof pattern !== 'string') {
    return res.status(400).json({ error: 'Pattern required' });
  }

  // Reject patterns with catastrophic backtracking potential
  if (!safeRegex(pattern)) {
    return res.status(400).json({ error: 'Unsafe regex pattern detected' });
  }

  // ✅ Limit input length before applying regex
  const input = (req.query.input || '').slice(0, 1000);
  try {
    const regex = new RegExp(pattern);
    const result = regex.test(input);
    res.json({ pattern, input: input.slice(0, 50), matches: result });
  } catch (err) {
    res.status(400).json({ error: 'Invalid regex' });
  }
});

app.listen(3000, () => console.log('ReDoS-safe server on 3000'));
```

**Expected Output (for `GET /validate?pattern=(a+)+$&input=aaaaaaaaaaaaaaaaaaaaaaaaaaaa!`):**
```
{"error":"Unsafe regex pattern detected"}
```

**Why this output:** `safe-regex` detects the nested quantifier `(a+)+` as potentially catastrophic and rejects it before it can be compiled and executed.

#### Real-World Cases

- **Search endpoints:** User-supplied regex patterns can cause ReDoS.
- **Input validation:** Complex email or URL validation regexes are common ReDoS vectors.
- **API gateways:** Large JSON payloads can exhaust memory in body parsers.

---

## Core Concept 5: Credential & Session Attacks

### Definitions

**Core Definition:** Credential and session attacks target the mechanisms by which users prove their identity (credentials) and maintain that identity across requests (sessions), enabling attackers to impersonate legitimate users.

**Technical Definition:** Session fixation occurs when an attacker sets a victim's session ID before authentication, then hijacks the authenticated session. Session hijacking involves stealing a valid session ID (via XSS, network sniffing, or malware) and using it to impersonate the user. Credential harvesting captures login credentials through phishing, malware, or compromised databases. Password hashing with modern algorithms (Argon2id, bcrypt) ensures that even if the credential database is compromised, attackers cannot easily recover plaintext passwords.

**Beginner-Friendly Explanation:** A session is like a coat-check ticket. If an attacker can copy your ticket number, they can claim your coat. Session fixation is like an attacker giving you a ticket they already copied before you check in. Password hashing is like storing a fingerprint of your password instead of the password itself — even if someone steals the fingerprint, they can't reverse-engineer your password.

### Purposes

- To prevent attackers from hijacking authenticated sessions.
- To ensure that passwords are stored in a format that resists offline cracking.
- To detect and mitigate credential theft through breach monitoring.
- To enforce secure session lifecycle management (creation, rotation, destruction).

### Sub-Feature 5.1: Session Fixation and Hijacking Prevention

#### Definitions

**Core Definition:** Session fixation prevention requires regenerating the session ID after successful authentication, destroying the old session. Session hijacking prevention requires binding sessions to client characteristics (IP, User-Agent), using secure cookie flags, and protecting against XSS.

**Technical Definition:** In Express with `express-session`, `req.session.regenerate()` creates a new session ID and destroys the old session record. This one-line fix prevents session fixation because the attacker's pre-set session ID becomes invalid after login. Additional defences include `SameSite=Lax` or `Strict` cookie attributes, `Secure` flag (HTTPS only), `HttpOnly` flag (no JavaScript access), and session fingerprinting (binding to User-Agent + Accept-Language).

**Beginner-Friendly Explanation:** Imagine a hotel where the room key you're given at check-in is the same key you had before you checked in. An attacker could give you a key, wait for you to check in, then use a copy of that key to enter your room. Session regeneration is like the hotel giving you a brand-new key at check-in and invalidating any old keys.

#### Purposes

- To prevent session fixation attacks by invalidating pre-authentication session IDs.
- To limit the window of opportunity for session hijacking.
- To ensure that stolen session IDs become useless after a short period.
- To bind sessions to client characteristics, making theft less useful.

#### Syntax Rules and Structure

```js
app.post('/login', async (req, res) => {
  // Authenticate user...
  req.session.regenerate((err) => {   // ✅ Regenerate session ID
    if (err) return res.status(500).send('Session error');
    req.session.userId = user.id;     // Store user ID in new session
    res.json({ message: 'Logged in' });
  });
});
```

| Method | Purpose |
|--------|---------|
| `req.session.regenerate()` | Creates new session ID, destroys old. |
| `cookie.httpOnly: true` | Prevents JavaScript access to session cookie. |
| `cookie.secure: true` | Cookie only sent over HTTPS. |
| `cookie.sameSite: 'lax'` | Limits cross-origin cookie sending. |

**Constraints and Limitations:**
- `regenerate()` is asynchronous; errors must be handled.
- Fingerprinting can cause false positives for users who change networks or browsers.
- `SameSite=Strict` may break OAuth flows; `Lax` is a common compromise.

#### Annotated Code Example

```js
// session-security.js — Session fixation prevention
const express = require('express');
const session = require('express-session');
const app = express();
app.use(express.json());

app.use(session({
  secret: process.env.SESSION_SECRET || 'dev-secret',
  resave: false,
  saveUninitialized: false,
  name: 'app.sid',                     // Custom cookie name
  cookie: {
    httpOnly: true,                    // No JavaScript access
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 3600000                    // 1 hour
  }
}));

// Simulated login
app.post('/login', (req, res) => {
  const { username } = req.body;
  if (!username) return res.status(400).json({ error: 'Username required' });

  // ✅ Regenerate session ID after authentication
  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: 'Session error' });

    req.session.userId = username;
    req.session.createdAt = Date.now();

    res.json({
      message: 'Logged in',
      sessionId: req.sessionID  // For demonstration only; never expose in production
    });
  });
});

// Protected route
app.get('/profile', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  res.json({ userId: req.session.userId });
});

app.listen(3000, () => console.log('Session-secure server on 3000'));
```

**Expected Output (for `POST /login` with `{"username":"alice"}`):**
```
{"message":"Logged in","sessionId":"new-session-id-here"}
```

**Why this output:** `req.session.regenerate()` creates a new session ID and invalidates the old one. The attacker's pre-set session ID (if any) is no longer valid, preventing session fixation.

#### Real-World Cases

- **Banking applications:** Session regeneration after login prevents account takeover.
- **E-commerce:** Protecting customer sessions during checkout.
- **Healthcare portals:** Compliance requirements (HIPAA) mandate secure session management.

---

### Sub-Feature 5.2: Password Hashing with Argon2id and bcrypt

#### Definitions

**Core Definition:** Password hashing is the process of transforming a plaintext password into a fixed-length digest using a one-way cryptographic function, making it computationally infeasible to reverse.

**Technical Definition:** Modern password hashing algorithms (Argon2id, bcrypt, scrypt) are deliberately slow and memory-hard. Argon2id is the winner of the Password Hashing Competition and is recommended by OWASP as the first choice for new applications. bcrypt is a well-established alternative with a cost factor of 12+ (approximately 400ms on modern hardware). Both algorithms incorporate a per-user random salt to prevent rainbow table attacks. A pepper (application-level secret) can be added for additional protection if the database is compromised.

**Beginner-Friendly Explanation:** A password hash is like a fingerprint of your password. You can't turn a fingerprint back into a person, and two different passwords produce completely different fingerprints. Argon2id and bcrypt are like fingerprinting machines that are deliberately slow — they take 400 milliseconds per fingerprint, so an attacker trying billions of guesses would need years.

#### Purposes

- To prevent attackers from recovering plaintext passwords if the database is compromised.
- To make brute-force and rainbow-table attacks computationally infeasible.
- To comply with NIST SP 800-63B and OWASP recommendations.
- To enable password migration from weaker to stronger algorithms over time.

#### Syntax Rules and Structure

```js
// Argon2id (recommended)
const argon2 = require('argon2');
const hash = await argon2.hash(password, {
  type: argon2.argon2id,
  memoryCost: 65536,   // 64 MiB
  timeCost: 3,         // 3 iterations
  parallelism: 4       // 4 threads
});

// bcrypt (established alternative)
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12); // Cost factor 12
```

| Algorithm | Type | Memory-Hard | Recommended |
|-----------|------|-------------|-------------|
| MD5 | Fast hash | No | Never |
| SHA-256 | Fast hash | No | Never |
| bcrypt | Adaptive | No | Yes (cost 12+) |
| scrypt | Adaptive | Yes | Yes |
| Argon2id | Adaptive | Yes | Preferred |

**Constraints and Limitations:**
- Argon2id requires more memory; tune parameters based on server resources.
- bcrypt has a 72-byte input limit; longer passwords are truncated.
- Hash migration requires re-hashing on next login (you cannot re-hash existing hashes).

#### Annotated Code Example

```js
// password-hashing.js — Argon2id and bcrypt
const argon2 = require('argon2');
const bcrypt = require('bcrypt');
const crypto = require('node:crypto');

const PEPPER = process.env.PASSWORD_PEPPER || 'app-level-secret';

// ✅ Argon2id hashing (OWASP first choice)
async function hashPasswordArgon2(password) {
  const peppered = crypto.createHmac('sha256', PEPPER)
    .update(password).digest('hex');
  return await argon2.hash(peppered, {
    type: argon2.argon2id,
    memoryCost: 65536,
    timeCost: 3,
    parallelism: 4
  });
}

// ✅ bcrypt hashing (established alternative)
async function hashPasswordBcrypt(password) {
  return await bcrypt.hash(password, 12);
}

// ✅ Verify with constant-time comparison
async function verifyPassword(hash, password, algorithm) {
  if (algorithm === 'argon2') {
    const peppered = crypto.createHmac('sha256', PEPPER)
      .update(password).digest('hex');
    return await argon2.verify(hash, peppered);
  }
  return await bcrypt.compare(password, hash);
}

// Demonstration
(async () => {
  const password = 'SecureP@ss123!';

  const argonHash = await hashPasswordArgon2(password);
  console.log('Argon2id hash:', argonHash.slice(0, 50) + '...');

  const bcryptHash = await hashPasswordBcrypt(password);
  console.log('bcrypt hash:', bcryptHash.slice(0, 29) + '...');

  console.log('Argon2 verify (correct):', await verifyPassword(argonHash, password, 'argon2'));
  console.log('Argon2 verify (wrong):', await verifyPassword(argonHash, 'wrong', 'argon2'));
  console.log('bcrypt verify (correct):', await verifyPassword(bcryptHash, password, 'bcrypt'));
})();
```

**Expected Output:**
```
Argon2id hash: $argon2id$v=19$m=65536,t=3,p=4$...
bcrypt hash: $2b$12$...
Argon2 verify (correct): true
Argon2 verify (wrong): false
bcrypt verify (correct): true
```

**Why this output:** The hash strings include the algorithm identifier, parameters, salt, and hash. Verification returns `true` for the correct password and `false` for the wrong one. The pepper is applied before hashing, so an attacker without the pepper cannot verify guesses even if they have the hash.

#### Real-World Cases

- **User registration:** Hashing passwords before storing them in the database.
- **Login verification:** Comparing a submitted password against the stored hash.
- **Hash migration:** Re-hashing passwords from bcrypt to Argon2id on next login.

---

## Core Concept 6: SSRF & Command Injection

### Definitions

**Core Definition:** Server-Side Request Forgery (SSRF) tricks the server into making HTTP requests to unintended destinations, while command injection executes arbitrary operating system commands through unsanitised input passed to shell commands.

**Technical Definition:** SSRF occurs when an application fetches a URL provided by the user without validating the destination, allowing attackers to access internal services, cloud metadata endpoints (e.g., `169.254.169.254`), or localhost. Command injection occurs when user input is concatenated into a shell command string and passed to `child_process.exec()`, `execSync()`, or `spawn()` with `shell: true`, allowing attackers to inject shell metacharacters (`;`, `|`, `&&`) to execute arbitrary commands.

**Beginner-Friendly Explanation:** SSRF is like asking a friend to fetch a book from the library, but they end up breaking into the librarian's office because you gave them a fake address. Command injection is like giving someone instructions in a language where the same word can mean "fetch the file" or "delete everything" — if you don't check what they heard, they might do the wrong thing.

### Purposes

- To prevent attackers from using the server as a proxy to access internal resources.
- To ensure that user input is never passed unsanitised to shell commands.
- To protect cloud environments from metadata service credential theft.
- To prevent remote code execution through OS command injection.

### Sub-Feature 6.1: SSRF Prevention — Allowlist and DNS Validation

#### Definitions

**Core Definition:** SSRF prevention requires validating that URLs requested by the server point to safe, external destinations, using allowlists of permitted hosts, validating resolved IP addresses, and disabling automatic redirects.

**Technical Definition:** Effective SSRF defence in Node.js involves normalising the URL (converting Unicode, replacing backslashes, removing credentials), restricting protocols to `http` and `https`, parsing with the WHATWG URL API, resolving the hostname to an IP address, and blocking private (RFC1918), loopback, link-local, and cloud metadata IP ranges. Redirects must be validated individually, and short timeouts must be enforced.

**Beginner-Friendly Explanation:** SSRF prevention is like giving a delivery driver a list of approved addresses and checking each address before they leave — if the address is the company's private office (internal network) or the security guard's desk (cloud metadata), the delivery is rejected.

#### Purposes

- To prevent access to internal services and cloud metadata endpoints.
- To protect against DNS rebinding attacks that resolve to internal IPs after initial validation.
- To ensure that redirects do not bypass initial URL validation.
- To enforce protocol restrictions (only HTTP/HTTPS).

#### Syntax Rules and Structure

```js
const { URL } = require('node:url');
const dns = require('node:dns').promises;
const net = require('node:net');

function isPrivateIP(ip) {
  return /^(127\.|10\.|172\.(1[6-9]|2\d|3[01])\.|192\.168\.|169\.254\.|::1|fc|fd|fe80)/.test(ip);
}

async function validateUrl(urlString) {
  const url = new URL(urlString);

  // Restrict protocols
  if (!['http:', 'https:'].includes(url.protocol)) {
    throw new Error('Protocol not allowed');
  }

  // Resolve DNS and check IP
  const addresses = await dns.resolve(url.hostname);
  for (const addr of addresses) {
    if (isPrivateIP(addr)) {
      throw new Error('SSRF attempt: private IP detected');
    }
  }

  return url;
}
```

| Defence | Implementation |
|---------|---------------|
| Protocol allowlist | Only `http:` and `https:` |
| IP validation | Block private, loopback, link-local, metadata ranges |
| Redirect control | `redirect: 'manual'` or validate each redirect |
| Timeout | Short timeouts (e.g., 5 seconds) |

**Constraints and Limitations:**
- DNS rebinding can bypass initial IP validation if the hostname is resolved again during the request.
- Cloud metadata IPs vary by provider (AWS: `169.254.169.254`, GCP: `metadata.google.internal`).
- URL parsers may differ in how they handle edge cases; use the WHATWG URL API consistently.

#### Annotated Code Example

```js
// ssrf-prevention.js — Safe outbound request with validation
const express = require('express');
const { URL } = require('node:url');
const dns = require('node:dns').promises;
const app = express();
app.use(express.json());

const ALLOWED_HOSTS = ['api.github.com', 'jsonplaceholder.typicode.com'];

function isPrivateIP(ip) {
  const privateRanges = [
    /^127\./, /^10\./, /^172\.(1[6-9]|2\d|3[01])\./,
    /^192\.168\./, /^169\.254\./, /^::1$/, /^fc/, /^fd/, /^fe80/
  ];
  return privateRanges.some(re => re.test(ip));
}

async function validateDestination(urlString) {
  const url = new URL(urlString);

  if (!['http:', 'https:'].includes(url.protocol)) {
    throw new Error('Only HTTP/HTTPS allowed');
  }

  if (!ALLOWED_HOSTS.includes(url.hostname)) {
    throw new Error('Host not in allowlist');
  }

  const addresses = await dns.resolve(url.hostname);
  for (const addr of addresses) {
    if (isPrivateIP(addr)) {
      throw new Error('Private IP detected');
    }
  }
  return url;
}

app.post('/fetch', async (req, res) => {
  try {
    const url = await validateDestination(req.body.url);
    const response = await fetch(url, { redirect: 'manual', signal: AbortSignal.timeout(5000) });
    const data = await response.text();
    res.json({ status: response.status, data: data.slice(0, 200) });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

app.listen(3000, () => console.log('SSRF-safe server on 3000'));
```

**Expected Output (for `POST /fetch` with `{"url":"http://169.254.169.254/latest/meta-data/"}`):**
```
{"error":"Host not in allowlist"}
```

**Why this output:** The metadata IP `169.254.169.254` is not in the `ALLOWED_HOSTS` array, so validation fails before any request is made.

#### Real-World Cases

- **Image proxies:** Fetching user-provided image URLs can lead to SSRF if not validated.
- **Webhook testers:** Testing webhook URLs can be exploited to probe internal services.
- **PDF generators:** Rendering PDFs from user-supplied URLs can trigger SSRF.

---

### Sub-Feature 6.2: Command Injection Prevention — Safe `child_process` Usage

#### Definitions

**Core Definition:** Command injection prevention requires using `child_process.execFile()` or `spawn()` with argument arrays instead of `exec()` with string concatenation, and validating all input passed to child processes.

**Technical Definition:** `child_process.exec()` invokes a shell (`/bin/sh` on Unix, `cmd.exe` on Windows) and passes the entire command as a string, allowing shell metacharacters (`;`, `|`, `&&`, `$()`) to inject additional commands. `child_process.execFile()` and `spawn()` with `shell: false` (the default) execute the command directly without shell interpretation, and arguments are passed as an array, preventing injection. If `shell: true` is used with `execFile` or `spawn`, arguments are not escaped (Node.js DEP0190), reintroducing the vulnerability.

**Beginner-Friendly Explanation:** `exec()` is like telling a chef "Make me a sandwich with ham and cheese" — if someone sneaks in "and also burn down the kitchen," the chef might do it. `execFile()` is like handing the chef a recipe card with separate ingredient slots — "Make me a sandwich with [ham] and [cheese]" — the chef only uses the ingredients you specify.

#### Purposes

- To prevent attackers from executing arbitrary OS commands through user input.
- To ensure that command arguments are treated as data, not shell syntax.
- To avoid the performance overhead of spawning a shell when not needed.
- To provide clear separation between the command and its arguments.

#### Syntax Rules and Structure

```js
// ❌ VULNERABLE: exec with string concatenation
const { exec } = require('child_process');
exec(`ls ${userInput}`, (err, stdout) => { ... });

// ✅ SAFE: execFile with argument array (no shell)
const { execFile } = require('child_process');
execFile('ls', [userInput], (err, stdout) => { ... });

// ✅ SAFE: spawn with argument array and shell: false
const { spawn } = require('child_process');
spawn('ls', [userInput], { shell: false });

// ❌ DANGEROUS: shell: true with arguments (DEP0190)
execFile('ls', [userInput], { shell: true }); // Arguments not escaped!
```

| Method | Shell Invocation | Safe? |
|--------|-----------------|-------|
| `exec(cmd)` | Yes | No (string injection) |
| `execFile(cmd, args)` | No (default) | Yes |
| `spawn(cmd, args)` | No (default) | Yes |
| `execFile(cmd, args, { shell: true })` | Yes | No (DEP0190) |
| `spawn(cmd, args, { shell: true })` | Yes | No (DEP0190) |

**Constraints and Limitations:**
- `execFile` requires the executable name to be a literal; arguments can be user-controlled if validated.
- Allowlist validation should still be applied to arguments where possible.
- On Windows, `execFile` cannot execute `.bat` or `.cmd` files directly; use `cmd.exe /c` with caution.

#### Annotated Code Example

```js
// command-injection-prevention.js — Safe child_process usage
const express = require('express');
const { execFile } = require('node:child_process');
const { promisify } = require('node:util');
const app = express();
app.use(express.json());

const execFileAsync = promisify(execFile);

// ✅ SAFE: execFile with argument array, no shell
app.post('/ping', async (req, res) => {
  const host = req.body.host;

  // Validate input: only allow alphanumeric, dots, and hyphens
  if (!/^[a-zA-Z0-9.-]+$/.test(host)) {
    return res.status(400).json({ error: 'Invalid host format' });
  }

  try {
    // ✅ SAFE: Arguments passed as array, no shell interpretation
    const { stdout } = await execFileAsync('ping', ['-c', '1', host], {
      timeout: 5000
    });
    res.json({ output: stdout });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// ❌ VULNERABLE comparison (do NOT use):
// app.post('/ping-unsafe', (req, res) => {
//   exec(`ping -c 1 ${req.body.host}`, (err, stdout) => { ... });
// });

app.listen(3000, () => console.log('Command-safe server on 3000'));
```

**Expected Output (for `POST /ping` with `{"host":"example.com"}`):**
```
{"output":"PING example.com ..."}
```

**Expected Output (for `POST /ping` with `{"host":"example.com; rm -rf /"}`):**
```
{"error":"Invalid host format"}
```

**Why this output:** The validation regex `/^[a-zA-Z0-9.-]+$/` rejects the semicolon and space, preventing the injection. Even without validation, `execFile` would pass the entire string as a single argument to `ping`, which would fail to resolve the hostname rather than executing `rm -rf /`.

#### Real-World Cases

- **Image processing:** Passing user-supplied filenames to ImageMagick or FFmpeg.
- **Network diagnostics:** Ping or traceroute endpoints that accept user-provided hosts.
- **File conversion:** Calling `ffmpeg` or `libreoffice` with user-supplied paths.

---

## References

- OWASP SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP NoSQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/NoSQL_Injection_Prevention_Cheat_Sheet.html
- OWASP XSS Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP DOM-based XSS Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
- OWASP CSRF Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP Authentication Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Password Storage Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP Session Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP SSRF Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP Node.js SSRF Prevention — https://owasp.org/www-community/pages/controls/SSRF_Prevention_in_Nodejs.html
- OWASP Command Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- OWASP Top 10:2021 — https://owasp.org/www-project-top-ten/
- NIST SP 800-63B Digital Identity Guidelines — https://pages.nist.gov/800-63-3/sp800-63b.html
- CWE-307: Improper Restriction of Excessive Authentication Attempts — https://cwe.mitre.org/data/definitions/307.html
- CWE-78: OS Command Injection — https://cwe.mitre.org/data/definitions/78.html
- Node.js `child_process` Documentation — https://nodejs.org/api/child_process.html
- Node.js DEP0190: `child_process` with shell option — https://nodejs.org/api/deprecations.html#dep0190
- Express.js `express-rate-limit` — https://expressjs.com/en/resources/middleware/rate-limits.html
- `express-mongo-sanitize` npm Package — https://www.npmjs.com/package/express-mongo-sanitize
- `argon2` npm Package — https://www.npmjs.com/package/argon2
- `bcrypt` npm Package — https://www.npmjs.com/package/bcrypt
- `safe-regex` npm Package — https://www.npmjs.com/package/safe-regex
- CVE-2024-53900 (Mongoose `populate()` injection) — https://nvd.nist.gov/vuln/detail/CVE-2024-53900
- CVE-2025-23061 (Mongoose `$where` bypass) — https://nvd.nist.gov/vuln/detail/CVE-2025-23061