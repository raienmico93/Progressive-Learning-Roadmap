# Express.js Authentication Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Express.js authentication fundamentals are the collection of concepts, middleware patterns, and security controls used to verify the identity of a client making a request to an Express application and to maintain that verified identity across subsequent requests. Express itself does not include a built-in authentication system; authentication is intentionally delegated to application code or to middleware libraries such as Passport.js.

**Technical Definition**
In an Express.js context, authentication is implemented through middleware functions that inspect request credentials — passwords, session identifiers, bearer tokens, or federated assertions — and attach an authenticated principal to the request object, conventionally as `req.user`. Persistent authentication is achieved either through stateful session management using a server-side session store and an HTTP cookie containing a session identifier, or through stateless token-based mechanisms such as JSON Web Tokens (JWTs) that are signed and verified on each request.

**Beginner-Friendly Explanation**
Authentication is how your app answers the question, “Who is making this request?” In Express, you write or plug in small helper functions called middleware that check a user’s credentials — such as an email and password, a session cookie, or a token — before allowing access to protected routes. Express does not do this for you out of the box; it gives you the flexibility to choose the approach that fits your application.

---

### Key Characteristics

- **Middleware-centric** — Authentication logic in Express is implemented as middleware that runs before route handlers and can accept, reject, or modify the request.
- **Pluggable and unopinionated** — Express does not prescribe an authentication scheme; developers may use sessions, tokens, OAuth, WebAuthn, or any combination.
- **Stateful or stateless** — Authentication state can be stored server-side (sessions) or encoded into cryptographically signed tokens (JWTs).
- **Identity-centric** — Every authentication approach ultimately produces an identity representation, typically attached to `req.user`, which downstream authorization logic consumes.
- **Security-sensitive by default** — Poorly configured sessions, weak password storage, or missing token validation can lead to session hijacking, credential theft, or privilege escalation.

---

### Prerequisites

- **Node.js and npm** — Express is a Node.js framework; a current LTS release of Node.js is assumed.
- **Basic HTTP knowledge** — Understanding of request/response cycles, headers, cookies, and status codes.
- **Express routing and middleware basics** — Familiarity with `app.use()`, route handlers, and the `(req, res, next)` signature.
- **JavaScript asynchronous patterns** — Promises and `async/await` are used heavily in password hashing and token operations.
- **Basic database access** — Most authentication flows require persisting users and credentials; examples in this sheet use an in-memory store for clarity, but production systems require a database.

---

### Related Programming Areas

- **Authorization** — Determining what an authenticated identity is allowed to do.
- **Session management** — Server-side state maintenance and session identifier security.
- **Cryptography** — Password hashing, token signing, and key management.
- **Web security** — CSRF, XSS, clickjacking, and transport security.
- **Identity federation** — OAuth 2.0, OpenID Connect, SAML, and social login.
- **API design** — Stateless authentication for REST and GraphQL APIs.

---

### Core Concepts / Features

1. **Authentication** — Verifying who a client claims to be.
2. **Identity** — The representation of the authenticated subject within the application.
3. **Credentials** — The secrets or assertions used to prove identity.
4. **Sessions** — Stateful, server-managed authentication persistence.
5. **Tokens** — Stateless, cryptographically signed authentication artifacts.
6. **Authentication Best Practices** — Defensive patterns for password storage, session hardening, token validation, and abuse prevention.

---

## Core Concept 1: Authentication

### Definitions

**Core Definition**
Authentication is the process of verifying that an individual, entity, or website is who or what it claims to be by determining the validity of one or more authenticators, such as passwords, fingerprints, or security tokens.

**Technical Definition**
In Express.js, authentication is performed by middleware that extracts credentials from an incoming HTTP request, validates them against an identity store or a cryptographic assertion, and, on success, populates `req.user` with the authenticated principal before calling `next()`. On failure, the middleware terminates the request with a `401 Unauthorized` response or redirects to a login page.

**Beginner-Friendly Explanation**
Authentication is the login step. When a user submits a username and password, or presents a token or cookie, your Express app checks whether those credentials are genuine. If they are, the app remembers who the user is for the rest of the request — and, in session-based systems, for subsequent requests.

---

### Purposes

- To verify that a request originates from a claimed identity before granting access to protected resources.
- To attach a trustworthy identity object to the request so downstream middleware and route handlers can make authorization decisions.
- To reject unauthenticated requests consistently, preventing accidental exposure of private data.
- To establish the foundation for session creation or token issuance.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express Authentication Middleware

```javascript
// Generic authentication middleware signature
function authenticate(req, res, next) {
  // 1. Extract credentials from req
  // 2. Validate credentials
  // 3. On success: attach identity, call next()
  // 4. On failure: respond 401 or redirect
}

app.use('/protected', authenticate);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `req` | Incoming request object containing headers, cookies, body, and query parameters. |
| `res` | Response object used to send `401`, redirect, or other error responses. |
| `next` | Callback that passes control to the next middleware or route handler. |
| `req.user` | Conventional property where the authenticated identity is stored. |

#### Complete General Syntax: Passport.js Authentication Middleware

```javascript
app.post(
  '/login',
  passport.authenticate('local', {
    failureRedirect: '/login',
    failureMessage: true
  }),
  function (req, res) {
    res.redirect('/dashboard');
  }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `passport.authenticate('local', options)` | Passport middleware that invokes the configured `local` strategy. |
| `failureRedirect` | URL to redirect to when authentication fails. |
| `failureMessage` | Enables a failure message to be stored for display. |
| Final handler | Runs only after successful authentication; `req.user` is populated. |

#### Syntax Rules

- Authentication middleware must either call `next()` or send a response; otherwise the request hangs.
- `req.user` should be set only after credentials have been fully validated.
- Authentication middleware should be mounted before any route handler that requires authentication.
- Passport strategies must be configured and registered before `passport.authenticate()` is used.

#### Constraints and Limitations

- Express does not provide built-in authentication; every application must implement or integrate it.
- Passport’s default failure response is `401 Unauthorized`; for browser-based flows, `failureRedirect` must be set explicitly.
- Middleware order is critical: authentication must run before authorization and route logic.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Custom Authentication Middleware with bcrypt

**Setup**

```bash
mkdir express-auth-demo && cd express-auth-demo
npm init -y
npm install express bcrypt
```

**`server.js`**

```javascript
// server.js
const express = require('express');
const bcrypt = require('bcrypt');

const app = express();
app.use(express.json());

// In-memory user store for demonstration only.
// Production systems must use a persistent database.
const users = [];

// --- Registration route: hash password before storing ---
app.post('/register', async (req, res) => {
  const { username, password } = req.body;

  // Validate input
  if (!username || !password) {
    return res.status(400).json({ error: 'Username and password required' });
  }

  // Hash the password with bcrypt (cost factor 12)
  const passwordHash = await bcrypt.hash(password, 12);

  users.push({ username, passwordHash });
  res.status(201).json({ message: 'User registered' });
});

// --- Custom authentication middleware ---
async function authenticate(req, res, next) {
  // Extract credentials from the Authorization header (Basic scheme example)
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Basic ')) {
    return res.status(401).json({ error: 'Missing credentials' });
  }

  // Decode Base64 username:password
  const decoded = Buffer.from(authHeader.slice(6), 'base64').toString('utf8');
  const [username, password] = decoded.split(':');

  // Find user
  const user = users.find((u) => u.username === username);
  if (!user) {
    // Use a generic error to avoid account enumeration
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Compare supplied password with stored hash
  const match = await bcrypt.compare(password, user.passwordHash);
  if (!match) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Attach identity to the request
  req.user = { username: user.username };
  next();
}

// --- Protected route ---
app.get('/profile', authenticate, (req, res) => {
  res.json({ message: `Hello, ${req.user.username}` });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

**Step-by-Step Explanation**

1. **Registration** — The client sends `{ "username": "alice", "password": "secret" }` to `POST /register`. The password is hashed with bcrypt using cost factor 12 before storage. bcrypt automatically generates and embeds a unique salt.
2. **Authentication middleware** — The client sends an `Authorization: Basic base64(alice:secret)` header. The middleware decodes the credentials, finds the user, and calls `bcrypt.compare()` to verify the password against the stored hash.
3. **Identity attachment** — On success, `req.user` is set to `{ username: 'alice' }`.
4. **Protected route** — The `/profile` route runs only after `next()` is called, and it can safely read `req.user`.

**Expected Output**

```
# Registration
POST /register  { "username": "alice", "password": "secret" }
→ 201 { "message": "User registered" }

# Authenticated request
GET /profile
Authorization: Basic YWxpY2U6c2VjcmV0
→ 200 { "message": "Hello, alice" }

# Failed request (wrong password)
GET /profile
Authorization: Basic YWxpY2U6d3Jvbmc=
→ 401 { "error": "Invalid credentials" }
```

**Why This Output Occurs**

- The registration route returns `201` because a new resource (the user record) is created.
- The protected route returns `200` only when `bcrypt.compare()` resolves to `true`; otherwise the middleware short-circuits with `401`.
- Using a generic `"Invalid credentials"` message for both unknown users and wrong passwords mitigates account enumeration.

---

#### Example 2: Passport.js Local Strategy

**Setup**

```bash
npm install express passport passport-local express-session bcrypt
```

**`server.js`**

```javascript
const express = require('express');
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const session = require('express-session');
const bcrypt = require('bcrypt');

const app = express();
app.use(express.urlencoded({ extended: false }));

// Session middleware (required for Passport session support)
app.use(
  session({
    secret: process.env.SESSION_SECRET || 'dev-only-secret',
    resave: false,
    saveUninitialized: false,
    cookie: { httpOnly: true, sameSite: 'lax' }
  })
);

app.use(passport.initialize());
app.use(passport.session());

// Demo user store
const users = [
  {
    id: 1,
    username: 'bob',
    passwordHash: bcrypt.hashSync('password123', 12)
  }
];

// Configure Passport local strategy
passport.use(
  new LocalStrategy(async (username, password, done) => {
    try {
      const user = users.find((u) => u.username === username);
      if (!user) {
        return done(null, false, { message: 'Invalid credentials' });
      }
      const match = await bcrypt.compare(password, user.passwordHash);
      if (!match) {
        return done(null, false, { message: 'Invalid credentials' });
      }
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  })
);

// Serialize user into session
passport.serializeUser((user, done) => {
  done(null, user.id);
});

// Deserialize user from session
passport.deserializeUser((id, done) => {
  const user = users.find((u) => u.id === id);
  done(null, user);
});

// Login route using Passport middleware
app.post(
  '/login',
  passport.authenticate('local', {
    failureRedirect: '/login-failed',
    failureMessage: true
  }),
  (req, res) => {
    res.json({ message: `Welcome, ${req.user.username}` });
  }
);

app.get('/login-failed', (req, res) => {
  res.status(401).json({ error: 'Login failed' });
});

app.listen(3000, () => console.log('Passport demo on port 3000'));
```

**Step-by-Step Explanation**

1. **Session setup** — `express-session` is mounted before Passport so that Passport can persist authentication state across requests.
2. **Strategy configuration** — `LocalStrategy` receives `username` and `password` from the request body and calls `done(null, user)` on success.
3. **Serialization** — On successful login, Passport stores `user.id` in the session.
4. **Deserialization** — On subsequent requests, Passport retrieves the user by ID and populates `req.user`.
5. **Route protection** — Any route that checks `req.isAuthenticated()` benefits from this session-backed identity.

**Expected Output**

```
POST /login  username=bob&password=password123
→ 200 { "message": "Welcome, bob" }

POST /login  username=bob&password=wrong
→ 302 Location: /login-failed
→ GET /login-failed → 401 { "error": "Login failed" }
```

**Why This Output Occurs**

- On success, Passport’s middleware calls the final handler and `req.user` is populated from the deserialized session user.
- On failure with `failureRedirect`, Passport issues a `302` redirect instead of a `401`, which is appropriate for browser-based login forms.

---

### Real-World Cases

- **Traditional web application login** — Passport Local Strategy with session cookies is widely used for server-rendered applications where users submit HTML forms.
- **API Basic Authentication** — Custom middleware validating `Authorization: Basic` headers is common for internal service-to-service APIs or simple protected endpoints.
- **Multi-factor authentication entry point** — The first authentication step (password verification) is followed by a second middleware that checks a one-time password or WebAuthn assertion.

---

## Core Concept 2: Identity

### Definitions

**Core Definition**
Identity is the unique representation of a subject engaged in an online transaction. A digital identity is always unique in the context of a digital service, but it does not necessarily need to be traceable back to a specific real-life subject.

**Technical Definition**
In Express.js, identity is materialized as an object attached to `req.user` after successful authentication. This object typically contains at least a stable user identifier (`id`, `sub`, or `username`) and may include roles, permissions, and other claims. Downstream authorization middleware reads `req.user` to make access-control decisions.

**Beginner-Friendly Explanation**
Identity is “who the user is” inside your application. After login, Express needs a way to remember that this request belongs to Alice, not Bob. That remembered identity is usually stored in `req.user` and is used by the rest of your code to decide what Alice is allowed to see or do.

---

### Purposes

- To provide a stable, trustworthy representation of the authenticated subject for the duration of a request.
- To serve as the input to authorization decisions, such as role checks or ownership checks.
- To avoid re-verifying credentials on every request when session or token state is available.
- To support auditing and logging by associating actions with a known principal.

---

### Syntax Rules and Structure

#### Complete General Syntax: Attaching Identity to the Request

```javascript
// After successful credential validation:
req.user = {
  id: user.id,
  username: user.username,
  roles: user.roles
};
next();
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `req.user` | Conventional Express property for the authenticated principal. |
| `id` | Stable, unique identifier for the user. |
| `username` | Human-readable identifier. |
| `roles` | Optional authorization claims. |

#### Complete General Syntax: Accessing Identity in a Route Handler

```javascript
app.get('/dashboard', (req, res) => {
  if (!req.user) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  res.json({ userId: req.user.id });
});
```

#### Syntax Rules

- `req.user` should never be trusted if it has not been set by authentication middleware.
- Identity objects should contain only the minimum information needed for authorization; sensitive fields must be excluded.
- In token-based systems, identity claims are often carried in the JWT `sub` (subject) claim and mapped to `req.user` after verification.

#### Constraints and Limitations

- Express does not enforce a shape for `req.user`; the application must define and document it.
- If `req.user` is populated from a client-controlled source without verification, the application is vulnerable to impersonation.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Populating Identity from a Verified JWT

```javascript
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
  const authHeader = req.headers.authorization;
  const token = authHeader && authHeader.split(' ')[1]; // Bearer <token>

  if (!token) {
    return res.status(401).json({ error: 'Token missing' });
  }

  jwt.verify(token, process.env.JWT_SECRET, (err, payload) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid or expired token' });
    }

    // Map verified claims to req.user
    req.user = {
      id: payload.sub,
      username: payload.username,
      roles: payload.roles || []
    };
    next();
  });
}

app.get('/api/me', authenticateToken, (req, res) => {
  res.json({ identity: req.user });
});
```

**Expected Output**

```
GET /api/me
Authorization: Bearer <valid-jwt>
→ 200 {
  "identity": {
    "id": "user_42",
    "username": "carol",
    "roles": ["editor"]
  }
}
```

**Why This Output Occurs**

- `jwt.verify()` cryptographically verifies the token signature and expiration.
- Only after verification is `req.user` populated; unverified tokens never reach the route handler.

---

### Real-World Cases

- **Role-based dashboards** — `req.user.roles` determines which UI components and API endpoints are accessible.
- **Audit trails** — Logging `req.user.id` alongside each action creates accountability.
- **Multi-tenant SaaS** — `req.user.tenantId` scopes data access to the correct organization.

---

## Core Concept 3: Credentials

### Definitions

**Core Definition**
Credentials are the authenticators presented by a client to prove a claimed identity. They include passwords, passphrases, one-time codes, cryptographic keys, certificates, and biometric assertions.

**Technical Definition**
In Express.js authentication flows, credentials are extracted from HTTP headers, request bodies, cookies, or query parameters. Password credentials must be stored only as slow, salted hashes; token credentials must be verified cryptographically; and all credential handling must occur over TLS to prevent interception.

**Beginner-Friendly Explanation**
Credentials are the “proof” a user provides — usually a password, but sometimes a token or a fingerprint. Your Express app must check that proof without ever storing the original password in a readable form.

---

### Purposes

- To allow a client to prove ownership of an identity without exposing long-term secrets.
- To enable password-based login while protecting stored passwords with adaptive hashing.
- To support machine-to-machine authentication through API keys or signed tokens.
- To provide a recoverable authentication mechanism when passwords are forgotten, using short-lived reset credentials.

---

### Syntax Rules and Structure

#### Complete General Syntax: Password Hashing with bcrypt

```javascript
const bcrypt = require('bcrypt');

// Hashing (registration or password change)
const hash = await bcrypt.hash(plaintextPassword, 12);

// Verification (login)
const match = await bcrypt.compare(plaintextPassword, storedHash);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `bcrypt.hash(password, cost)` | Produces a salted hash; `cost` controls computational expense. |
| `bcrypt.compare(password, hash)` | Re-hashes the candidate password with the stored salt and compares. |
| `cost = 12` | A work factor consistent with OWASP guidance of 10 or more. |

#### Syntax Rules

- Never store plaintext passwords or fast hashes such as SHA-256 for password storage.
- bcrypt has a **72-byte password limit**; applications must enforce this or use Argon2id for longer passphrases.
- Password comparison must use a constant-time comparison function, which `bcrypt.compare()` provides.
- All credential transmission must occur over TLS.

#### Constraints and Limitations

- bcrypt is a legacy-but-acceptable algorithm; OWASP recommends Argon2id as the first choice for new systems.
- Fast hashing algorithms are unsuitable for passwords because they enable rapid offline guessing.
- Passwords must never be logged or included in error messages.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Secure Registration and Login with bcrypt

```javascript
const express = require('express');
const bcrypt = require('bcrypt');
const app = express();
app.use(express.json());

const users = [];

app.post('/register', async (req, res) => {
  const { username, password } = req.body;

  // Enforce minimum password length (OWASP: 15 chars without MFA)
  if (!password || password.length < 15) {
    return res.status(400).json({ error: 'Password too short' });
  }

  // Reject passwords exceeding bcrypt's 72-byte limit
  if (Buffer.byteLength(password, 'utf8') > 72) {
    return res.status(400).json({ error: 'Password exceeds 72 bytes' });
  }

  const passwordHash = await bcrypt.hash(password, 12);
  users.push({ username, passwordHash });
  res.status(201).json({ message: 'Registered' });
});

app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.find((u) => u.username === username);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const match = await bcrypt.compare(password, user.passwordHash);
  if (!match) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  res.json({ message: 'Authenticated' });
});

app.listen(3000);
```

**Expected Output**

```
POST /register { "username": "dave", "password": "correct-horse-battery-staple" }
→ 201 { "message": "Registered" }

POST /login { "username": "dave", "password": "correct-horse-battery-staple" }
→ 200 { "message": "Authenticated" }

POST /login { "username": "dave", "password": "wrong" }
→ 401 { "error": "Invalid credentials" }
```

**Why This Output Occurs**

- The registration route enforces a minimum length of 15 characters, consistent with OWASP guidance for systems without MFA.
- The 72-byte check prevents silent truncation by bcrypt.
- `bcrypt.compare()` returns `false` for a wrong password, producing a generic `401`.

---

### Real-World Cases

- **E-commerce checkout** — Password verification is required before accessing stored payment methods.
- **Password reset flows** — A short-lived, single-use token acts as a temporary credential for resetting a forgotten password.
- **API key authentication** — A long random string acts as a credential for service-to-service calls and must be hashed at rest.

---

## Core Concept 4: Sessions

### Definitions

**Core Definition**
Session management is the process by which a server maintains the state of an entity interacting with it, so that it can remember how to react to subsequent requests throughout a transaction.

**Technical Definition**
In Express.js, sessions are implemented with middleware such as `express-session`, which creates a server-side session record and sends the client an HTTP cookie containing only a session identifier. The session data itself remains on the server and is retrieved by that identifier on each request. The default in-memory `MemoryStore` is explicitly unsuitable for production.

**Beginner-Friendly Explanation**
A session is how your app remembers a user between requests. After login, the server gives the browser a cookie containing a random ID. On later requests, the browser sends that cookie back, and the server looks up the stored session data to remember who the user is. The actual user data stays on the server, not in the cookie.

---

### Purposes

- To maintain authentication state across multiple HTTP requests in a stateless protocol.
- To avoid sending credentials on every request after initial login.
- To support server-side invalidation of authentication, such as immediate logout or forced sign-out of all devices.
- To store per-user session data such as preferences and access rights.

---

### Syntax Rules and Structure

#### Complete General Syntax: express-session Configuration

```javascript
const session = require('express-session');

app.use(
  session({
    name: 'sid',                    // Custom cookie name
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      httpOnly: true,
      secure: true,                 // HTTPS only in production
      sameSite: 'lax',
      maxAge: 1000 * 60 * 60        // 1 hour
    },
    store: new RedisStore({ client: redisClient }) // Production store
  })
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `name` | Cookie name; avoid default fingerprintable names. |
| `secret` | Signs the session ID cookie to prevent tampering. |
| `resave` | Avoids unnecessary session store writes when `false`. |
| `saveUninitialized` | Prevents creating sessions for unauthenticated requests when `false`. |
| `cookie.httpOnly` | Prevents client-side JavaScript access, mitigating XSS-based session theft. |
| `cookie.secure` | Restricts cookie transmission to HTTPS. |
| `cookie.sameSite` | Mitigates CSRF by controlling cross-site cookie sending. |
| `store` | Replaces `MemoryStore` with a scalable, production-grade store. |

#### Syntax Rules

- `express-session` must be mounted before any route or middleware that reads `req.session`.
- Session identifiers must be long, random, and computationally unpredictable.
- The `secure` cookie flag must be enabled in production.
- A production session store (Redis, database, etc.) must replace `MemoryStore`.

#### Constraints and Limitations

- `MemoryStore` leaks memory and does not scale beyond a single process; it is for development only.
- Session data is not saved in the cookie itself, only the session ID.
- Session cookies are subject to browser policies and can be blocked if `SameSite=None` is used without `Secure`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Session-Based Login and Protected Route

```javascript
const express = require('express');
const session = require('express-session');
const app = express();

app.use(express.json());
app.use(
  session({
    secret: 'dev-secret-change-in-production',
    resave: false,
    saveUninitialized: false,
    cookie: {
      httpOnly: true,
      sameSite: 'lax',
      maxAge: 1000 * 60 * 30 // 30 minutes
    }
  })
);

// Fake user verification (use bcrypt and a database in production)
function verifyUser(username, password) {
  return username === 'erin' && password === 'secret';
}

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  if (!verifyUser(username, password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Regenerate session ID to prevent session fixation
  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: 'Session error' });

    req.session.userId = username;
    res.json({ message: 'Logged in' });
  });
});

function requireSession(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  next();
}

app.get('/account', requireSession, (req, res) => {
  res.json({ user: req.session.userId });
});

app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) return res.status(500).json({ error: 'Logout failed' });
    res.clearCookie('connect.sid');
    res.json({ message: 'Logged out' });
  });
});

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Session middleware** — Creates a session cookie with `httpOnly` and `sameSite` protections.
2. **Login** — After credential verification, `req.session.regenerate()` issues a new session ID to prevent session fixation, then stores `userId` in the session.
3. **Protected route** — `requireSession` checks for `req.session.userId`; the account route is accessible only when it exists.
4. **Logout** — `req.session.destroy()` removes the server-side session record, and the cookie is cleared.

**Expected Output**

```
POST /login { "username": "erin", "password": "secret" }
→ 200 { "message": "Logged in" }
Set-Cookie: connect.sid=s%3A...; HttpOnly; SameSite=Lax

GET /account
Cookie: connect.sid=s%3A...
→ 200 { "user": "erin" }

POST /logout
→ 200 { "message": "Logged out" }

GET /account
→ 401 { "error": "Not authenticated" }
```

**Why This Output Occurs**

- The session cookie is issued only after successful login.
- The protected route reads the session from the cookie and retrieves the stored `userId`.
- After logout, the session is destroyed server-side, so the old cookie no longer maps to a valid session.

---

### Real-World Cases

- **Banking applications** — Short-lived sessions with strict `httpOnly`, `Secure`, and `SameSite` flags protect against session hijacking.
- **Administrative dashboards** — Server-side session invalidation allows immediate revocation of access when an employee leaves.
- **Shopping carts** — Sessions store cart contents for both anonymous and authenticated users.

---

## Core Concept 5: Tokens

### Definitions

**Core Definition**
A token is a compact, self-contained credential that encodes authentication or authorization information and can be verified cryptographically without necessarily consulting a server-side session store.

**Technical Definition**
In Express.js, token-based authentication most commonly uses JSON Web Tokens (JWTs), which consist of three Base64URL-encoded parts: a header, a payload containing claims, and a signature. The `jsonwebtoken` library provides `jwt.sign()` to issue tokens and `jwt.verify()` to validate signatures, expiration, audience, and issuer. Tokens are typically transmitted in the `Authorization: Bearer <token>` header.

**Beginner-Friendly Explanation**
A token is like a stamped ticket. When you log in, the server gives you a ticket that says who you are and when it expires. On later requests, you show the ticket instead of your password. The server checks the stamp (signature) to make sure the ticket is real and has not been tampered with.

---

### Purposes

- To enable stateless authentication that scales horizontally without shared session storage.
- To carry identity claims that APIs can verify independently.
- To support short-lived access credentials with separate refresh mechanisms.
- To allow cross-domain and mobile clients to authenticate without relying on cookies.

---

### Syntax Rules and Structure

#### Complete General Syntax: JWT Signing and Verification

```javascript
const jwt = require('jsonwebtoken');

// Sign
const token = jwt.sign(
  { sub: user.id, username: user.username },
  process.env.JWT_SECRET,
  { expiresIn: '15m', algorithm: 'HS256' }
);

// Verify
jwt.verify(token, process.env.JWT_SECRET, { algorithms: ['HS256'] }, (err, payload) => {
  if (err) {
    // reject
  }
  // payload contains sub, username, iat, exp
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `payload` | Claims about the subject; `sub` is the standard subject claim. |
| `secretOrPrivateKey` | HMAC secret or private key used to sign the token. |
| `expiresIn` | Token lifetime; short values (5–15 minutes) are recommended for access tokens. |
| `algorithm` | Signing algorithm; `HS256` for symmetric, `RS256`/`ES256` for asymmetric. |
| `jwt.verify()` | Validates signature, expiration, and optionally `audience` and `issuer`. |

#### Syntax Rules

- Tokens must be transmitted over TLS.
- The `algorithms` option in `jwt.verify()` must be explicitly set to prevent algorithm-confusion attacks.
- Access tokens should be short-lived; refresh tokens should be rotated and stored securely.
- Sensitive data must never be placed in the JWT payload, because JWTs are signed, not encrypted.

#### Constraints and Limitations

- JWTs cannot be revoked before expiration without a server-side denylist or short expiration plus refresh rotation.
- Storing tokens in `localStorage` exposes them to XSS; `HttpOnly` cookies are safer for browser-based applications when CSRF protections are also in place.
- The `none` algorithm must never be accepted.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: JWT Issuance and Protected Route

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();
app.use(express.json());

const JWT_SECRET = process.env.JWT_SECRET || 'dev-only-secret';

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  // Replace with real credential verification
  if (username !== 'frank' || password !== 'secret') {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const token = jwt.sign(
    { sub: 'user_99', username: 'frank' },
    JWT_SECRET,
    { expiresIn: '15m', algorithm: 'HS256' }
  );

  res.json({ accessToken: token });
});

function authenticateToken(req, res, next) {
  const authHeader = req.headers.authorization;
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) return res.status(401).json({ error: 'Token required' });

  jwt.verify(token, JWT_SECRET, { algorithms: ['HS256'] }, (err, payload) => {
    if (err) return res.status(403).json({ error: 'Invalid or expired token' });
    req.user = { id: payload.sub, username: payload.username };
    next();
  });
}

app.get('/api/protected', authenticateToken, (req, res) => {
  res.json({ message: `Hello, ${req.user.username}` });
});

app.listen(3000);
```

**Expected Output**

```
POST /login { "username": "frank", "password": "secret" }
→ 200 { "accessToken": "eyJhbGciOiJIUzI1NiIs..." }

GET /api/protected
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
→ 200 { "message": "Hello, frank" }

GET /api/protected
Authorization: Bearer invalid.token.here
→ 403 { "error": "Invalid or expired token" }
```

**Why This Output Occurs**

- `jwt.sign()` creates a token whose payload contains `sub` and `username`, with an expiration 15 minutes in the future.
- `jwt.verify()` rejects tokens with invalid signatures or expired `exp` claims.
- `req.user` is populated only after successful verification.

---

### Real-World Cases

- **Mobile APIs** — Mobile clients store short-lived access tokens and use refresh tokens to obtain new access tokens without re-entering passwords.
- **Microservices** — Services verify JWTs issued by a central identity provider without calling back to a session store.
- **Single-page applications** — SPAs use bearer tokens for API calls, often with refresh-token rotation for longer sessions.

---

## Core Concept 6: Authentication Best Practices

### Definitions

**Core Definition**
Authentication best practices are the defensive design, implementation, and operational patterns that reduce the risk of credential compromise, session hijacking, token abuse, and authentication bypass.

**Technical Definition**
In Express.js, authentication best practices encompass password storage with slow adaptive hashing, session hardening with secure cookie flags and production stores, token validation with explicit algorithms and short lifetimes, rate limiting, CSRF protection, account-enumeration defenses, and immediate invalidation on credential change.

**Beginner-Friendly Explanation**
Best practices are the “rules of the road” that keep your login system safe. They include hashing passwords so thieves cannot read them, locking cookies so JavaScript cannot steal them, making tokens expire quickly, and slowing down attackers who try thousands of passwords.

---

### Purposes

- To protect stored credentials even if the database is compromised.
- To prevent session hijacking through cookie theft, fixation, or cross-site request forgery.
- To limit the damage of stolen tokens through short expiration and rotation.
- To resist brute-force and credential-stuffing attacks.
- To avoid leaking information that helps attackers enumerate valid accounts.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Session Cookie

```javascript
app.use(
  session({
    name: '__Host-sid',
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      path: '/',
      maxAge: 1000 * 60 * 30
    },
    store: productionStore
  })
);
```

**Component Breakdown**

| Directive | Security Effect |
|-----------|-----------------|
| `__Host-` prefix | Enforces `Secure`, `Path=/`, and no `Domain` attribute. |
| `httpOnly: true` | Blocks JavaScript access to the cookie, mitigating XSS-based theft. |
| `secure: true` | Cookie is sent only over HTTPS. |
| `sameSite: 'strict'` | Strong CSRF mitigation for same-site flows. |
| `maxAge` | Limits the window of session validity. |
| `productionStore` | Avoids `MemoryStore` memory leaks and scaling limits. |

#### Syntax Rules

- Password hashing must use Argon2id, bcrypt, or scrypt; fast hashes are prohibited.
- Session IDs must be regenerated on login to prevent fixation.
- Access tokens must be short-lived (5–15 minutes) and refresh tokens rotated.
- Login and password-reset endpoints must be rate-limited.
- Authentication errors must be generic to prevent account enumeration.
- Deprecated or vulnerable Express versions must not be used.

#### Constraints and Limitations

- `SameSite=None` requires `Secure` and is blocked by some browser configurations.
- bcrypt’s 72-byte limit constrains password length; applications must enforce this explicitly.
- JWT revocation requires additional state; short expiration is the primary mitigation.
- `MemoryStore` is not a production session store.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Rate-Limited Login with Generic Errors

```javascript
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10,                  // 10 attempts per window per IP
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many login attempts. Try again later.' }
});

app.post('/login', loginLimiter, async (req, res) => {
  const { username, password } = req.body;

  const user = await findUserByUsername(username);

  // Always perform a hash comparison, even if user is not found,
  // to equalize timing and reduce account enumeration.
  const hash = user ? user.passwordHash : DUMMY_HASH;
  const match = await bcrypt.compare(password, hash);

  if (!user || !match) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: 'Session error' });
    req.session.userId = user.id;
    res.json({ message: 'Authenticated' });
  });
});
```

**Expected Output**

```
# First 10 attempts with wrong password:
POST /login → 401 { "error": "Invalid credentials" }

# 11th attempt within 15 minutes:
POST /login → 429 { "error": "Too many login attempts. Try again later." }
```

**Why This Output Occurs**

- The rate limiter tracks requests per IP and returns `429 Too Many Requests` after the threshold.
- The generic `"Invalid credentials"` message does not reveal whether the username exists.
- Comparing against a dummy hash when the user is not found helps equalize response timing.

---

#### Example 2: Secure JWT Verification with Algorithm Pinning

```javascript
function authenticateToken(req, res, next) {
  const authHeader = req.headers.authorization;
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) return res.status(401).json({ error: 'Token required' });

  jwt.verify(
    token,
    process.env.JWT_PUBLIC_KEY, // Use public key for RS256
    { algorithms: ['RS256'], issuer: 'https://auth.example.com', audience: 'api.example.com' },
    (err, payload) => {
      if (err) return res.status(403).json({ error: 'Invalid token' });
      req.user = { id: payload.sub };
      next();
    }
  );
}
```

**Expected Output**

```
GET /api/protected
Authorization: Bearer <RS256-token-signed-by-correct-key>
→ 200 { "message": "Access granted" }

GET /api/protected
Authorization: Bearer <HS256-token-or-wrong-key>
→ 403 { "error": "Invalid token" }
```

**Why This Output Occurs**

- The `algorithms: ['RS256']` option rejects tokens signed with a different algorithm, preventing algorithm-confusion attacks.
- `issuer` and `audience` validation ensures the token was intended for this API and issued by the trusted authority.

---

### Real-World Cases

- **Financial services** — Mandatory short session timeouts, `HttpOnly` cookies, and rate limiting on login endpoints are common regulatory expectations.
- **Healthcare portals** — Immediate session invalidation on password change and audit logging of authentication events are standard.
- **Public APIs** — Short-lived JWTs with refresh-token rotation and algorithm pinning protect against token replay and confusion attacks.

---

## References

- OWASP Authentication Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Session Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Password Storage Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- Express.js Production Best Practices: Security — https://expressjs.com/en/advanced/best-practice-security.html
- Passport.js Authentication Concepts — https://www.passportjs.org/concepts/authentication/downloads/html/
- express-session Documentation — https://expressjs.com/en/resources/middleware/session
- jsonwebtoken (npm) — https://www.npmjs.com/package/jsonwebtoken
- Express.js FAQ: How can I authenticate users? — https://expressjs.com/en/starter/faq.html