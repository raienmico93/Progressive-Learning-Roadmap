# Authentication Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Authentication is the process of verifying that an individual, entity, or website is who it claims to be — establishing a trusted identity that the system can use to make authorisation decisions and associate actions with the correct user.

**Technical Definition:** Authentication (AuthN) is distinct from authorisation (AuthZ): AuthN verifies *who you are*, while AuthZ determines *what you are allowed to do*. Authentication establishes a verifiable identity claim by validating one or more authentication factors — something the user knows (password), something the user has (TOTP device, hardware key), or something the user is (biometric). The result is a session or token that the client presents on subsequent requests, allowing the server to associate each request with a previously authenticated identity.

**Beginner-Friendly Explanation:** Authentication is like showing your ID at the airport. The security officer checks that your face matches the photo on your ID and that the ID is genuine. Once verified, you get a boarding pass (session or token) that proves you've passed security. On subsequent checkpoints, you show the boarding pass instead of your ID again. Authentication is the process of proving who you are; the boarding pass is the credential that maintains that proof.

### Key Characteristics

- **Identity-first:** Every authenticated request is associated with a unique, immutable user identifier.
- **Multi-factor capable:** Authentication strength increases with additional independent factors.
- **Stateful or stateless:** Sessions (server-tracked) and tokens (client-side) are the two primary architectural models.
- **Phishing resistance:** Passkeys/WebAuthn represent the strongest form of authentication, resistant to credential theft.
- **Session lifecycle:** Login establishes an authentication state; logout explicitly destroys it on both client and server.
- **Defence in depth:** Rate limiting, account lockout, and secure cookie flags are essential supporting controls.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A password hashing library:** `npm install argon2` or `npm install bcrypt`.
- **A session or token library:** `npm install express-session` or `npm install jsonwebtoken`.
- **For MFA:** `npm install otplib qrcode`.
- **For passkeys:** `npm install @simplewebauthn/server @simplewebauthn/browser`.

### Related Programming Areas

- **Authorisation:** Role-based access control (RBAC) and permission checks that follow authentication.
- **Session Management:** Cookie security, session stores, and session lifecycle.
- **Cryptography:** Password hashing, token signing, and key pair generation.
- **Error Handling:** Authentication failures (401) and their proper response formatting.
- **Security Headers:** Helmet, CORS, and cookie flags that protect authentication state.

### Core Concepts

1. **Identity** — establishing distinct user identity claims and maintaining immutable identifiers.
2. **Login & Logout** — session initiation, credential validation, and authentication state destruction.
3. **Sessions vs. Tokens** — stateful server-tracked auth vs. stateless client-side tokens.
4. **Multi-Factor Authentication (MFA)** — TOTP-based second factors (Google Authenticator, Authy).
5. **Passkeys & WebAuthn** — phishing-resistant cryptographic key pairs using biometrics or hardware keys.

---

## Core Concept 1: Identity

### Definitions

**Core Definition:** Identity in authentication is the set of claims about a user — including a unique, immutable identifier — that the system uses to distinguish one user from another and to associate actions, data, and permissions with the correct individual.

**Technical Definition:** A user identity comprises: (1) an **immutable user identifier** — a UUID or similar stable token that never changes and is never reused, even if the user changes their email or username; (2) **authentication claims** — the credentials (password hash, TOTP secret, WebAuthn public key) that verify the identity; and (3) **attribute claims** — roles, permissions, and profile data. The immutable identifier is critical: it is the foreign key that links the user to their data across the entire system lifecycle. If you store the user ID on the client and send it with requests, anyone can spoof being logged in by sending an ID with their requests. Therefore, identity must always be established server-side, either through a session ID or a signed token.

**Beginner-Friendly Explanation:** A user's identity is like their national ID number. It never changes — not when they get married and change their surname, not when they move house. The system uses this number to say "this is the same person as before." If you relied on the person's name instead, you'd lose track of them every time their name changed. In authentication, the immutable user ID is the anchor that keeps everything linked to the right person.

### Purposes

- To establish distinct user identity claims that persist across the system lifecycle.
- To maintain immutable user identifiers that never change, even when user attributes change.
- To provide a stable foreign key for linking user data, permissions, and audit logs.
- To prevent identity spoofing by ensuring identity is always established server-side.

### Syntax Rules and Structure

#### Identity Schema (Mongoose Example)

```javascript
// models/User.js
const mongoose = require('mongoose');
const { v4: uuidv4 } = require('uuid');

const userSchema = new mongoose.Schema({
  _id: {
    type: String,
    default: () => uuidv4(),     // Immutable UUID identifier
    immutable: true               // Never changes
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
  },
  passwordHash: {
    type: String,
    required: true
  },
  roles: {
    type: [String],
    default: ['user']
  },
  createdAt: {
    type: Date,
    default: Date.now,
    immutable: true
  }
});
```

| Component | Breakdown |
|-----------|-----------|
| `_id` | Immutable UUID — the primary identity anchor. |
| `email` | Mutable attribute — can change without breaking identity. |
| `passwordHash` | Authentication claim — never the plain password. |
| `immutable: true` | Mongoose option preventing updates. |

**Rules:**
- The user identifier must be **immutable** — it never changes, even if email or username changes.
- Use UUIDs or similar non-sequential identifiers — never auto-incrementing integers (they leak user count and enable enumeration).
- Never accept a user ID from the client as proof of identity — always derive it from the session or token.
- Store password hashes, not passwords. Use Argon2id or bcrypt with appropriate cost parameters.

**Constraints:**
- Email addresses are often used as login identifiers but should never be the immutable primary key (users change emails).
- Auto-incrementing IDs are an information disclosure risk — prefer UUIDs.

### Annotated Code Example

```javascript
// identity.js
const { v4: uuidv4 } = require('uuid');
const argon2 = require('argon2');

async function createUser(email, password) {
  // 1. Generate immutable identifier
  const userId = uuidv4();

  // 2. Hash password with Argon2id
  const passwordHash = await argon2.hash(password, {
    type: argon2.argon2id,
    memoryCost: 19456,    // 19 MiB
    timeCost: 2,
    parallelism: 1
  });

  // 3. Create user document
  const user = {
    id: userId,
    email: email.toLowerCase().trim(),
    passwordHash,
    roles: ['user'],
    createdAt: new Date()
  };

  // 4. Persist to database
  await db.collection('users').insertOne(user);

  return { id: userId, email: user.email };
}
```

**Expected Output:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "alice@example.com"
}
```

**Why this output:** The `uuidv4()` function generates a 128-bit random identifier that is statistically unique and impossible to guess. The password is hashed with Argon2id — the current OWASP-recommended algorithm — so the plain password is never stored. The returned object contains only the immutable ID and email, not the hash.

### Real-World Cases

- **User accounts:** Every user gets a UUID at registration that never changes.
- **Audit logs:** Actions are recorded against the immutable user ID, not the email.
- **Data migration:** When a user changes their email, all foreign key references to the user ID remain valid.
- **Multi-tenant SaaS:** The user ID is combined with a tenant ID for cross-tenant isolation.

---

## Core Concept 2: Login & Logout

### Definitions

**Core Definition:** Login is the process of validating credentials and establishing an authenticated session or token; logout is the explicit destruction of that authentication state on both the client and the server.

**Technical Definition:** The login flow receives credentials (typically email and password), looks up the user by a lookup identifier, verifies the password against the stored hash using a constant-time comparison, and then establishes an authenticated context. In session-based authentication, this means regenerating the session ID (to prevent session fixation) and storing the user ID in the session store. In token-based authentication, this means issuing a short-lived access token and a long-lived refresh token. Logout destroys the session on the server (removing it from the store) and clears the cookie on the client, or for tokens, revokes the refresh token and (optionally) blacklists the access token.

**Beginner-Friendly Explanation:** Login is like checking into a hotel — you show your ID (credentials), and the receptionist gives you a room key (session/token). Logout is like checking out — you return the key, the hotel removes your name from the active guest list, and your access ends. If you just walked away with the key (client-side logout only), the hotel would still think you're checked in.

### Purposes

- To manage session initiation by validating credentials against stored hashes.
- To explicitly destroy active authentication states across both client and server.
- To prevent session fixation attacks by regenerating the session ID on login.
- To issue access and refresh tokens in token-based architectures.

### Syntax Rules and Structure

#### Session-Based Login

```javascript
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  // 1. Lookup user
  const user = await db.findUserByEmail(email);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  // 2. Verify password (constant-time)
  const valid = await argon2.verify(user.passwordHash, password);
  if (!valid) return res.status(401).json({ error: 'Invalid credentials' });

  // 3. Regenerate session to prevent fixation
  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: 'Login failed' });

    // 4. Store user ID in session
    req.session.userId = user.id;

    res.json({ success: true, user: { id: user.id, email: user.email } });
  });
});
```

#### Session-Based Logout

```javascript
app.post('/auth/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) return res.status(500).json({ error: 'Logout failed' });
    res.clearCookie('connect.sid'); // Clear the session cookie
    res.json({ success: true });
  });
});
```

#### Token-Based Login (JWT)

```javascript
const jwt = require('jsonwebtoken');

app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await db.findUserByEmail(email);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  const valid = await argon2.verify(user.passwordHash, password);
  if (!valid) return res.status(401).json({ error: 'Invalid credentials' });

  // Short-lived access token (15 min)
  const accessToken = jwt.sign(
    { userId: user.id, type: 'access' },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );

  // Long-lived refresh token (7 days)
  const refreshToken = jwt.sign(
    { userId: user.id, type: 'refresh' },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  // Store refresh token hash in database for revocation
  await db.storeRefreshToken(user.id, refreshToken);

  res.json({ accessToken, refreshToken });
});
```

**Rules:**
- Always use **constant-time** password comparison (`argon2.verify`, `bcrypt.compare`).
- **Regenerate** the session ID on login to prevent session fixation.
- Use **short-lived access tokens** (5–15 minutes) and **long-lived refresh tokens** (7–30 days) for token-based auth.
- On logout, **destroy the session server-side** — do not rely on client-side deletion alone.
- For tokens, **revoke the refresh token** in the database on logout.

**Constraints:**
- JWTs cannot be revoked without a denylist — logout does not immediately invalidate access tokens.
- Session stores (Redis, database) must be used in production — the default in-memory store leaks memory and does not scale.

### Annotated Code Example

```javascript
// auth-routes.js
const express = require('express');
const argon2 = require('argon2');
const jwt = require('jsonwebtoken');
const router = express.Router();

// Login
router.post('/login', async (req, res) => {
  const { email, password } = req.body;

  const user = await db.findUserByEmail(email);
  if (!user) {
    return res.status(401).json({ error: 'Invalid email or password' });
  }

  const valid = await argon2.verify(user.passwordHash, password);
  if (!valid) {
    return res.status(401).json({ error: 'Invalid email or password' });
  }

  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: 'Login failed' });
    req.session.userId = user.id;
    res.json({ success: true, user: { id: user.id, email: user.email } });
  });
});

// Logout
router.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) return res.status(500).json({ error: 'Logout failed' });
    res.clearCookie('connect.sid');
    res.json({ success: true });
  });
});

module.exports = router;
```

**Expected Output (for `POST /auth/login` with valid credentials):**
```json
{
  "success": true,
  "user": { "id": "550e8400-e29b-41d4-a716-446655440000", "email": "alice@example.com" }
}
```

**Expected Output (for `POST /auth/logout`):**
```json
{ "success": true }
```

**Why this output:** The login handler verifies the password against the Argon2id hash, regenerates the session ID (preventing fixation), and stores the immutable user ID in the session. The logout handler destroys the session server-side and clears the cookie client-side.

### Real-World Cases

- **Web applications:** Session-based login with HTTP-only cookies.
- **Mobile apps:** Token-based login with access and refresh tokens.
- **SPAs:** Token-based auth with refresh token rotation.
- **Admin panels:** Session-based auth with additional MFA.

---

## Core Concept 3: Sessions vs. Tokens

### Definitions

**Core Definition:** Session-based authentication stores authentication state on the server and gives the client an opaque session ID; token-based authentication stores authentication state in a self-contained token held by the client.

**Technical Definition:** In **stateful authentication**, a unique session ID is generated at login. This session ID is an opaque reference to user information stored on the server — it contains no user data itself. In **stateless authentication**, all user identity information is stored in a token held by the client. This token can be transmitted to any server or microservice, eliminating the need to maintain session state on the server. Stateless verification is often delegated to an authorisation server that generates, issues, and optionally encrypts tokens at login.

**Beginner-Friendly Explanation:** Session-based auth is like a coat check — you hand over your coat, get a ticket (session ID), and the coat (user data) stays at the counter (server). Token-based auth is like a passport — everything the border officer needs to know about you is in the passport (token), and any border (server) can read it without calling your home country (central session store).

### Purposes

- To compare stateful server-tracked authentication against stateless client-side token architectures.
- To choose the appropriate model based on application architecture (monolith vs. microservices).
- To understand the security trade-offs of each approach.
- To implement session stores (Redis, database) for scalable session-based auth.

### Syntax Rules and Structure

#### Comparison Table

| Dimension | Session-Based | Token-Based (JWT) |
|-----------|---------------|-------------------|
| **State** | Server-side (session store). | Client-side (token). |
| **Credential** | Opaque session ID in cookie. | Signed JWT in header/cookie. |
| **Scalability** | Requires shared session store. | Horizontally scalable. |
| **Revocation** | Immediate (delete session). | Hard (denylist required). |
| **Logout** | Easy (destroy session). | Harder (token invalidation). |
| **Security** | httpOnly cookies (more secure). | Risk if token is stolen. |
| **Token rotation** | Not needed. | Refresh token rotation needed. |
| **Best for** | Monolithic apps. | Microservices, SPAs, mobile. |

#### Session Middleware

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis').default;

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,          // Prevent JS access
    secure: true,            // HTTPS only
    sameSite: 'strict',      // CSRF protection
    maxAge: 86400000         // 24 hours
  }
}));
```

#### Token Middleware

```javascript
function authenticateToken(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Authentication required' });
  }

  const token = authHeader.split(' ')[1];
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = { id: decoded.userId };
    next();
  } catch (err) {
    const message = err.name === 'TokenExpiredError'
      ? 'Token expired'
      : 'Invalid token';
    res.status(401).json({ error: message });
  }
}
```

**Rules:**
- Use sessions for monolithic applications where server-side rendering is used.
- Use tokens for microservices, SPAs, and mobile apps where the server should be stateless.
- Always use a shared session store (Redis) in production — never the default in-memory store.
- Set `httpOnly`, `secure`, and `sameSite` on session cookies.
- Use short-lived access tokens and long-lived refresh tokens for token-based auth.

**Constraints:**
- Sessions require a shared store for horizontal scaling — this is an additional infrastructure dependency.
- JWTs cannot be revoked — logout does not invalidate access tokens immediately.
- Token storage on the client is a security consideration — httpOnly cookies are safer than localStorage.

### Annotated Code Example

```javascript
// sessions-vs-tokens.js
// SESSION-BASED: Server remembers who you are
app.post('/auth/session-login', async (req, res) => {
  const user = await verifyCredentials(req.body);
  req.session.userId = user.id;   // Server stores the identity
  res.json({ success: true });     // Client gets only a session ID cookie
});

app.get('/api/session-profile', (req, res) => {
  // Server looks up the user from the session store
  const user = await db.findUserById(req.session.userId);
  res.json({ user });
});

// TOKEN-BASED: Client carries the identity
app.post('/auth/token-login', async (req, res) => {
  const user = await verifyCredentials(req.body);
  const token = jwt.sign({ userId: user.id }, JWT_SECRET, { expiresIn: '15m' });
  res.json({ token });             // Client stores and sends the token
});

app.get('/api/token-profile', authenticateToken, (req, res) => {
  // Server reads the identity from the token — no session store needed
  const user = await db.findUserById(req.user.id);
  res.json({ user });
});
```

**Expected Output (session login):**
```
Set-Cookie: connect.sid=s%3Aabc123...; HttpOnly; Secure; SameSite=Strict
{ "success": true }
```

**Expected Output (token login):**
```json
{ "token": "eyJhbGciOiJIUzI1NiIs..." }
```

**Why this output:** The session-based flow stores the user ID server-side and sends only an opaque session ID cookie. The token-based flow sends a self-contained JWT that the client stores and sends back on every request. The server verifies the session by looking it up; it verifies the token by checking its signature.

### Real-World Cases

- **Monolithic web apps:** Session-based auth with Redis session store.
- **Microservices:** Token-based auth with JWTs passed between services.
- **SPAs:** Token-based auth with httpOnly cookies for refresh tokens.
- **Mobile apps:** Token-based auth with secure storage for refresh tokens.

---

## Core Concept 4: Multi-Factor Authentication (MFA)

### Definitions

**Core Definition:** Multi-factor authentication (MFA) requires two or more independent authentication factors — something you know, something you have, or something you are — to verify identity.

**Technical Definition:** TOTP (Time-based One-Time Password) is defined in RFC 6238 and builds on the HMAC-based one-time password (HOTP) algorithm from RFC 4226. HOTP derives a code from a shared secret and an incrementing counter. TOTP replaces that counter with the current time divided into fixed windows (typically 30 seconds). Both the server and the authenticator app hold the same secret, so they independently compute the same 6-digit code for the same 30-second window. Microsoft's research found that MFA reduces the risk of account compromise by 99.22% across the entire user population, and by 98.56% even when the attacker already holds leaked credentials.

**Beginner-Friendly Explanation:** MFA is like needing both a key and a fingerprint to open a vault. If someone steals your key (password), they still can't open the vault without your fingerprint (the second factor). TOTP apps like Google Authenticator generate a new 6-digit code every 30 seconds — even if a hacker steals your password, they can't guess the current code.

### Purposes

- To implement an extra layer of identity verification beyond passwords.
- To use Time-based One-Time Passwords (TOTP) via apps like Google Authenticator.
- To provide recovery codes for account recovery when the primary device is lost.
- To prevent credential-stuffing and phishing attacks from succeeding.

### Syntax Rules and Structure

#### TOTP Setup (Registration)

```javascript
const { authenticator } = require('otplib');
const QRCode = require('qrcode');

// 1. Generate a secret for the user
const secret = authenticator.generateSecret();

// 2. Generate a QR code for Google Authenticator
const otpauth = authenticator.keyuri(user.email, 'MyApp', secret);
const qrCodeUrl = await QRCode.toDataURL(otpauth);

// 3. Store the secret (encrypted) in the user's record
await db.updateUser(user.id, { totpSecret: encrypt(secret) });

// 4. Return the QR code to the client for scanning
res.json({ qrCodeUrl });
```

#### TOTP Verification (Login)

```javascript
const { authenticator } = require('otplib');

async function verifyTOTP(userId, token) {
  const user = await db.findUserById(userId);
  const secret = decrypt(user.totpSecret);

  // Verify with a ±1 window tolerance (±30 seconds)
  const isValid = authenticator.verify({ token, secret });

  if (!isValid) {
    throw new Error('Invalid MFA code');
  }

  return true;
}
```

| Component | Breakdown |
|-----------|-----------|
| `authenticator.generateSecret()` | Generates a base32-encoded secret. |
| `authenticator.keyuri()` | Builds the `otpauth://` URI for QR codes. |
| `authenticator.verify()` | Verifies a TOTP token against the secret. |
| `window` | Tolerance for clock drift (default ±1 window). |

**Rules:**
- Store the TOTP secret **encrypted** — it is as sensitive as a password.
- Provide **recovery codes** (single-use backup codes) at setup.
- Rate-limit MFA verification attempts — brute-forcing 6 digits is feasible without limits.
- Use `otplib` (actively maintained) rather than `speakeasy` (not maintained).
- SMS-based MFA is vulnerable to SIM-swap attacks — prefer app-based TOTP.

**Constraints:**
- TOTP codes are valid for 30 seconds — clock drift can cause failures.
- Recovery codes must be hashed and single-use.
- MFA should be required for sensitive actions (password change, payment) as well as login.

### Annotated Code Example

```javascript
// mfa-routes.js
const { authenticator } = require('otplib');
const QRCode = require('qrcode');
const crypto = require('crypto');

// Setup: generate secret and QR code
app.post('/auth/mfa/setup', requireAuth, async (req, res) => {
  const secret = authenticator.generateSecret();
  const otpauth = authenticator.keyuri(req.user.email, 'MyApp', secret);
  const qrCodeUrl = await QRCode.toDataURL(otpauth);

  // Encrypt secret before storing
  const encrypted = encrypt(secret, process.env.ENCRYPTION_KEY);
  await db.updateUser(req.user.id, { totpSecret: encrypted, mfaEnabled: false });

  res.json({ qrCodeUrl, secret }); // Secret shown once for manual entry
});

// Verify: enable MFA after first successful code
app.post('/auth/mfa/verify', requireAuth, async (req, res) => {
  const { token } = req.body;
  const user = await db.findUserById(req.user.id);
  const secret = decrypt(user.totpSecret, process.env.ENCRYPTION_KEY);

  const isValid = authenticator.verify({ token, secret });
  if (!isValid) {
    return res.status(400).json({ error: 'Invalid code' });
  }

  // Generate recovery codes
  const recoveryCodes = Array.from({ length: 8 }, () =>
    crypto.randomBytes(4).toString('hex')
  );
  const hashedCodes = recoveryCodes.map(c => hash(c));
  await db.updateUser(req.user.id, {
    mfaEnabled: true,
    recoveryCodes: hashedCodes
  });

  res.json({ success: true, recoveryCodes });
});
```

**Expected Output (for `/auth/mfa/setup`):**
```json
{
  "qrCodeUrl": "data:image/png;base64,iVBORw0KGgo...",
  "secret": "JBSWY3DPEHPK3PXP"
}
```

**Expected Output (for `/auth/mfa/verify` with a valid code):**
```json
{
  "success": true,
  "recoveryCodes": ["a1b2c3d4", "e5f6a7b8", "..."]
}
```

**Why this output:** The setup endpoint generates a TOTP secret, creates a QR code for Google Authenticator, and stores the encrypted secret. The verify endpoint checks the user's 6-digit code against the secret. If valid, MFA is enabled and single-use recovery codes are generated and hashed.

### Real-World Cases

- **Banking apps:** MFA required for login and high-value transactions.
- **Corporate SSO:** TOTP as a second factor for enterprise accounts.
- **Developer platforms:** GitHub, npm, and AWS all support TOTP-based MFA.
- **SaaS platforms:** MFA as an optional (or mandatory) security setting.

---

## Core Concept 5: Passkeys & WebAuthn

### Definitions

**Core Definition:** Passkeys are phishing-resistant authentication credentials based on WebAuthn (Web Authentication) that replace passwords with cryptographic key pairs stored on the user's device and protected by biometrics or a hardware security key.

**Technical Definition:** WebAuthn is a W3C standard that enables public-key cryptography for authentication. During registration, the authenticator (device) generates a key pair: the private key is stored securely on the device (never leaves it), and the public key is sent to the server. During authentication, the server issues a challenge, the authenticator signs it with the private key, and the server verifies the signature with the stored public key. Passkeys are phishing-resistant because the private key never leaves the device and is bound to the origin (domain). The `@simplewebauthn/server` and `@simplewebauthn/browser` packages provide the server and client implementations.

**Beginner-Friendly Explanation:** A passkey is like a physical key that never leaves your pocket. When a website asks you to prove who you are, your device uses your fingerprint (or face) to unlock the key and sign a message. The website verifies the signature with the matching lock. Because the key never leaves your device and only works for the specific website, phishing attacks and credential theft are impossible — there's no password to steal.

### Purposes

- To understand the shifting paradigm from passwords to secure, phishing-resistant cryptographic key pairs.
- To implement WebAuthn registration and authentication using biometrics or hardware keys.
- To provide a fallback recovery mechanism when the primary device is lost.
- To eliminate password-related attack vectors (phishing, credential stuffing, brute force).

### Syntax Rules and Structure

#### Registration Flow

```javascript
const { generateRegistrationOptions, verifyRegistrationResponse } = require('@simplewebauthn/server');

// 1. Generate registration options
app.post('/auth/passkey/register/options', requireAuth, async (req, res) => {
  const options = await generateRegistrationOptions({
    rpName: 'MyApp',
    rpID: 'myapp.example.com',
    userID: req.user.id,
    userName: req.user.email,
    attestationType: 'none',
    authenticatorSelection: {
      residentKey: 'required',      // Discoverable credential
      userVerification: 'required'  // Biometric/PIN required
    }
  });

  // Store challenge in session for verification
  req.session.currentChallenge = options.challenge;
  res.json(options);
});

// 2. Verify registration response
app.post('/auth/passkey/register/verify', requireAuth, async (req, res) => {
  const verification = await verifyRegistrationResponse({
    response: req.body,
    expectedChallenge: req.session.currentChallenge,
    expectedOrigin: 'https://myapp.example.com',
    expectedRPID: 'myapp.example.com'
  });

  if (!verification.verified) {
    return res.status(400).json({ error: 'Registration failed' });
  }

  // Store the credential
  await db.storePasskey(req.user.id, {
    id: verification.registrationInfo.credentialID,
    publicKey: verification.registrationInfo.credentialPublicKey,
    counter: verification.registrationInfo.counter
  });

  res.json({ verified: true });
});
```

#### Authentication Flow

```javascript
const { generateAuthenticationOptions, verifyAuthenticationResponse } = require('@simplewebauthn/server');

app.post('/auth/passkey/login/options', async (req, res) => {
  const user = await db.findUserByEmail(req.body.email);
  if (!user) return res.status(404).json({ error: 'User not found' });

  const options = await generateAuthenticationOptions({
    rpID: 'myapp.example.com',
    allowCredentials: user.passkeys.map(pk => ({
      id: pk.id,
      type: 'public-key'
    })),
    userVerification: 'required'
  });

  req.session.currentChallenge = options.challenge;
  req.session.pendingUserId = user.id;
  res.json(options);
});

app.post('/auth/passkey/login/verify', async (req, res) => {
  const user = await db.findUserById(req.session.pendingUserId);
  const credential = user.passkeys.find(pk => pk.id === req.body.id);

  const verification = await verifyAuthenticationResponse({
    response: req.body,
    expectedChallenge: req.session.currentChallenge,
    expectedOrigin: 'https://myapp.example.com',
    expectedRPID: 'myapp.example.com',
    credential: {
      id: credential.id,
      publicKey: credential.publicKey,
      counter: credential.counter
    },
    requireUserVerification: true
  });

  if (!verification.verified) {
    return res.status(400).json({ verified: false });
  }

  // Update counter to detect cloned authenticators
  await db.updatePasskeyCounter(credential.id, verification.authenticationInfo.newCounter);

  // Create session
  req.session.userId = user.id;
  res.json({ verified: true });
});
```

| Component | Breakdown |
|-----------|-----------|
| `generateRegistrationOptions()` | Creates the challenge and options for registration. |
| `verifyRegistrationResponse()` | Verifies the attestation and extracts the public key. |
| `generateAuthenticationOptions()` | Creates the challenge for login. |
| `verifyAuthenticationResponse()` | Verifies the signed challenge. |
| `newCounter` | Must overwrite stored counter — detects cloned authenticators. |

**Rules:**
- **Store the challenge** in the session between the options and verify calls.
- **Update the counter** after each successful authentication — this detects cloned authenticators.
- Set `requireUserVerification: true` for stronger ceremonies.
- Provide **recovery mechanisms** — register an additional passkey or backup code during setup.
- Use **short sessions** after WebAuthn verification — don't issue a long-lived JWT.

**Constraints:**
- WebAuthn requires HTTPS (except localhost).
- Not all browsers and devices support passkeys — provide fallback authentication.
- Counter updates are critical for security — a counter that does not increase indicates a cloned authenticator.

### Annotated Code Example

```javascript
// passkey-service.js
const { generateRegistrationOptions, verifyRegistrationResponse,
        generateAuthenticationOptions, verifyAuthenticationResponse } = require('@simplewebauthn/server');

class PasskeyService {
  constructor(rpID, rpName, origin) {
    this.rpID = rpID;
    this.rpName = rpName;
    this.origin = origin;
  }

  async getRegistrationOptions(user) {
    return generateRegistrationOptions({
      rpName: this.rpName,
      rpID: this.rpID,
      userID: user.id,
      userName: user.email,
      attestationType: 'none',
      authenticatorSelection: {
        residentKey: 'required',
        userVerification: 'required'
      }
    });
  }

  async verifyRegistration(userId, response, expectedChallenge) {
    const verification = await verifyRegistrationResponse({
      response,
      expectedChallenge,
      expectedOrigin: this.origin,
      expectedRPID: this.rpID
    });

    if (verification.verified) {
      await db.storePasskey(userId, {
        id: verification.registrationInfo.credentialID,
        publicKey: verification.registrationInfo.credentialPublicKey,
        counter: verification.registrationInfo.counter
      });
    }

    return verification.verified;
  }
}
```

**Expected Output (for registration options):**
```json
{
  "challenge": "base64url-encoded-challenge",
  "rp": { "name": "MyApp", "id": "myapp.example.com" },
  "user": { "id": "user-id", "name": "alice@example.com" },
  "pubKeyCredParams": [{ "type": "public-key", "alg": -7 }],
  "authenticatorSelection": {
    "residentKey": "required",
    "userVerification": "required"
  }
}
```

**Expected Output (for verification):**
```json
{ "verified": true }
```

**Why this output:** The registration options include a challenge, the relying party ID, and the user information. The browser's WebAuthn API uses these to create a credential. The server verifies the attestation and stores the public key, credential ID, and counter.

### Real-World Cases

- **Consumer apps:** Google, Apple, and Microsoft all support passkeys.
- **Enterprise SSO:** Passkeys as a phishing-resistant MFA method.
- **Banking:** Passkeys for high-assurance authentication.
- **Developer platforms:** GitHub and npm support passkeys for account security.

---

## References

- OWASP Authentication Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Password Storage Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- NIST SP 800-63B-4 — Digital Identity Guidelines: Authentication and Authenticator Management — https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b-4.pdf
- RFC 6238 — TOTP: Time-Based One-Time Password Algorithm — https://datatracker.ietf.org/doc/html/rfc6238
- RFC 4226 — HOTP: An HMAC-Based One-Time Password Algorithm — https://datatracker.ietf.org/doc/html/rfc4226
- W3C WebAuthn Specification — https://www.w3.org/TR/webauthn-3/
- Express.js — Session Middleware — https://expressjs.com/en/resources/middleware/session.html
- express-session — npm — https://www.npmjs.com/package/express-session
- jsonwebtoken — npm — https://www.npmjs.com/package/jsonwebtoken
- argon2 — npm — https://www.npmjs.com/package/argon2
- otplib — npm — https://www.npmjs.com/package/otplib
- @simplewebauthn/server — npm — https://www.npmjs.com/package/@simplewebauthn/server
- @simplewebauthn/browser — npm — https://www.npmjs.com/package/@simplewebauthn/browser
- Shattered.io — Two-Factor Authentication in Node.js: 11 Steps [2026] — https://shattered.io/two-factor-authentication-nodejs/
- Shattered.io — JWT Authentication in Node.js: 12 Steps [2026] — https://shattered.io/jwt-authentication-nodejs/
- FreeCodeCamp — How to Set Up WebAuthn in Node.js for Passwordless Biometric Login — https://www.freecodecamp.org/news/set-up-webauthn-in-node-js-for-passwordless-biometric-login
- Security Boulevard — Add Passkeys to Node and Express in 30 Minutes — https://securityboulevard.com/add-passkeys-to-node-and-express-in-30-minutes/
- Shattered.io — WebAuthn em Node.js: Implementar Passkeys em 12 Passos [2026] — https://shattered.io/webauthn-nodejs/
- Microsoft — MFA Research (99.22% compromise reduction) — https://www.microsoft.com/en-us/security/blog/2019/08/20/one-simple-action-you-can-take-to-prevent-99-9-percent-of-account-attacks/