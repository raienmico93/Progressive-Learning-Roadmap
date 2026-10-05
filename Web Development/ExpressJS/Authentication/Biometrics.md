# Biometrics & Next-Gen Auth — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Biometric and next-generation authentication encompasses passwordless mechanisms that verify user identity through possession of a device (WebAuthn/passkeys), control of a communication channel (magic links, OTP), or inherent biological traits (fingerprint, Face ID), eliminating the need for shared secrets like passwords.

**Technical Definition:** This domain covers FIDO2/WebAuthn (a W3C standard for public-key cryptography-based authentication using platform or roaming authenticators), passkeys (FIDO2 credentials synced across devices via cloud keychains), and passwordless flows using one-time passwords (OTPs) or magic links delivered via email or SMS. These methods replace the shared-secret model of passwords with either asymmetric cryptography (WebAuthn) or short-lived, single-use tokens delivered through channels the user controls.

**Beginner-Friendly Explanation:** Instead of typing a password, you can log in using your fingerprint, your face, a code sent to your phone, or a link emailed to you. Passkeys are like digital keys stored in your device's secure hardware — they never leave your device and are tied to the specific website, making them phishing-resistant. Magic links and OTPs are like temporary passwords that expire quickly, delivered through channels only you can access.

### Key Characteristics

- **Phishing-resistant:** WebAuthn/passkeys are cryptographically bound to the origin, so they cannot be used on fake sites.
- **No shared secrets:** Private keys never leave the user's device; OTPs and magic links are short-lived and single-use.
- **Hardware-backed security:** Passkeys leverage secure enclaves, TPMs, or hardware security keys.
- **Cross-device support:** Synced passkeys (iCloud Keychain, Google Password Manager) work across the user's devices.
- **Reduced friction:** Users authenticate with biometrics or a PIN instead of remembering complex passwords.
- **Phishing risk for OTP/magic links:** Unlike passkeys, OTP and magic link flows are vulnerable to real-time phishing relays and email compromise.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express`.
- **HTTPS or localhost:** WebAuthn requires a secure context.
- **Basic understanding of public-key cryptography:** Key pairs, signing, and verification.
- **A registered relying party domain:** For WebAuthn (e.g., `example.com`).
- **Email/SMS delivery service:** For OTP and magic link flows (e.g., Nodemailer, Twilio, Telnyx).

### Related Programming Areas

- **FIDO2/WebAuthn:** The underlying standard for passkeys.
- **OAuth 2.1/OIDC:** Complementary protocols for federated identity.
- **Session management:** Passkeys and OTPs ultimately establish sessions.
- **Rate limiting:** Essential for OTP and magic link endpoints to prevent abuse.
- **Cryptography:** CBOR/COSE encoding, SHA-256 hashing, and ECDSA/RSA signatures.

### Core Concepts

1. **WebAuthn & Passkeys** — FIDO2 credentials, registration, and authentication flows using Express backends.
2. **Passwordless Email/SMS** — Magic links, OTP generation, and verification timing windows.

---

## Core Concept 1: WebAuthn & Passkeys

### Definitions

**Core Definition:** WebAuthn (Web Authentication) is a W3C standard that defines an API for creating and using public-key-based credentials (passkeys) for strong, phishing-resistant authentication.

**Technical Definition:** WebAuthn (officially "Web Authentication: An API for accessing Public Key Credentials") is a W3C Recommendation (Level 2, April 2021; Level 3 in active development) that enables web applications to authenticate users using public-key cryptography. A **passkey** is a FIDO2/WebAuthn credential where the private key stays in the device's secure hardware and never reaches the server. The server stores only the credential's public key, ID, and signature counter. WebAuthn defines two ceremonies: **registration** (attestation) and **authentication** (assertion).

**Beginner-Friendly Explanation:** WebAuthn lets you log in using your fingerprint, face, or a hardware security key instead of a password. When you register, your device creates a unique key pair: a private key that stays on your device (protected by biometrics or a PIN) and a public key that is sent to the website. When you log in, the website sends a challenge, your device signs it with the private key, and the website verifies the signature with the public key. Because the private key never leaves your device and the signature is tied to the website's domain, phishing attacks are impossible.

### Purposes

- To eliminate password-based authentication and its associated risks (phishing, credential stuffing, reuse).
- To provide phishing-resistant authentication by cryptographically binding credentials to the relying party's origin.
- To enable passwordless login using biometrics, device PINs, or hardware security keys.
- To support cross-device authentication through synced passkeys.

### Sub-Feature 1.1: FIDO2 Credentials

#### Definitions

**Core Definition:** A FIDO2 credential is a public-key credential created by an authenticator (device or security key) and registered with a relying party (the server).

**Technical Definition:** A FIDO2 credential consists of a credential ID (a unique identifier for the credential), a public key (stored on the server), and a signature counter (incremented with each authentication to detect cloning). The private key is stored in the authenticator's secure hardware and is never exposed. The credential is scoped to the relying party ID (e.g., `example.com`), meaning it can only be used for that domain.

**Beginner-Friendly Explanation:** A FIDO2 credential is like a digital key that lives in your device's secure vault. The key is unique to a specific website — the key you use for `example.com` won't work for `evil.com`. When you log in, your device uses the key to sign a challenge, proving you possess it without ever revealing the key itself.

#### Syntax Rules and Structure

**Credential Data Structure (stored on server):**
```json
{
  "credentialID": "base64url-encoded-id",
  "credentialPublicKey": "base64url-encoded-public-key",
  "counter": 0,
  "transports": ["internal", "hybrid", "usb", "nfc", "ble"],
  "createdAt": "2026-01-15T10:30:00Z"
}
```

| Field | Description |
|-------|-------------|
| `credentialID` | Unique identifier for the credential. |
| `credentialPublicKey` | The public key (COSE format, base64url-encoded). |
| `counter` | Signature counter for replay protection (0 for synced passkeys). |
| `transports` | Supported transport methods. |

#### Constraints and Limitations

- The relying party ID (RP ID) must be a registrable domain, never a URL or port, and cannot be changed later without invalidating all credentials.
- WebAuthn requires a secure context (HTTPS or localhost).
- Synced passkeys report a signature counter of 0; a non-incrementing counter is not necessarily a clone.
- As of SimpleWebAuthn v10, `userID` must be a `Uint8Array`, not a string.

### Sub-Feature 1.2: Registration Flow (Attestation)

#### Definitions

**Core Definition:** Registration is the ceremony where the authenticator creates a new credential and the server stores the associated public key.

**Technical Definition:** The registration ceremony begins with the server generating a challenge and registration options (including `rp`, `user`, `pubKeyCredParams`, `excludeCredentials`). The client passes these to `navigator.credentials.create()`, which prompts the user to create a credential using their authenticator. The authenticator returns an attestation response, which the client sends to the server for verification. The server verifies the challenge, origin, and attestation, then stores the credential's ID, public key, and counter.

**Beginner-Friendly Explanation:** Registration is like signing up for a new security key at the front desk. The server gives you a unique challenge, your device creates a new key pair, and you give the server the public key (like giving them a copy of your lock so they can verify your signature later). The private key stays on your device.

#### Syntax Rules and Structure

**Server-side Registration Options:**
```js
const { generateRegistrationOptions } = require('@simplewebauthn/server');

const options = await generateRegistrationOptions({
  rpName: 'My App',
  rpID: 'example.com',
  userID: new Uint8Array([1, 2, 3, 4]),
  userName: 'alice@example.com',
  attestationType: 'none',
  excludeCredentials: userCredentials.map(cred => ({
    id: cred.credentialID,
    type: 'public-key'
  })),
  authenticatorSelection: {
    residentKey: 'preferred',
    userVerification: 'preferred'
  }
});
```

| Parameter | Description |
|-----------|-------------|
| `rpName` | Human-readable relying party name. |
| `rpID` | The relying party's domain (e.g., `example.com`). |
| `userID` | Unique user identifier (Uint8Array). |
| `userName` | Human-readable username (email or username). |
| `attestationType` | `'none'` for most use cases. |
| `excludeCredentials` | Credentials already registered to prevent duplicates. |

**Server-side Verification:**
```js
const { verifyRegistrationResponse } = require('@simplewebauthn/server');

const verification = await verifyRegistrationResponse({
  response: req.body.attestationResponse,
  expectedChallenge: storedChallenge,
  expectedOrigin: 'https://example.com',
  expectedRPID: 'example.com'
});

if (verification.verified) {
  const { credentialID, credentialPublicKey, counter } = verification.registrationInfo;
  // Store credentialID, credentialPublicKey, counter in database
}
```

#### Annotated Code Example

```js
// webauthn-registration.js
const express = require('express');
const session = require('express-session');
const {
  generateRegistrationOptions,
  verifyRegistrationResponse
} = require('@simplewebauthn/server');
const app = express();

app.use(express.json());
app.use(session({ secret: 'secret', resave: false, saveUninitialized: false }));

// Simulated user and credential store
const users = new Map();
const credentials = new Map();

const RP_ID = 'localhost';
const RP_NAME = 'My WebAuthn App';
const ORIGIN = 'http://localhost:3000';

// Step 1: Generate registration options
app.post('/register/options', async (req, res) => {
  const { username } = req.body;

  // Find or create user
  let user = users.get(username);
  if (!user) {
    user = { id: crypto.randomUUID(), username };
    users.set(username, user);
  }

  // Get existing credentials for this user
  const userCredentials = [...credentials.values()]
    .filter(c => c.userId === user.id);

  const options = await generateRegistrationOptions({
    rpName: RP_NAME,
    rpID: RP_ID,
    userID: new TextEncoder().encode(user.id),
    userName: username,
    attestationType: 'none',
    excludeCredentials: userCredentials.map(c => ({
      id: c.credentialID,
      type: 'public-key'
    })),
    authenticatorSelection: {
      residentKey: 'preferred',
      userVerification: 'preferred'
    }
  });

  // Store challenge in session
  req.session.challenge = options.challenge;
  req.session.username = username;

  res.json(options);
});

// Step 2: Verify registration response
app.post('/register/verify', async (req, res) => {
  const { body } = req;

  try {
    const verification = await verifyRegistrationResponse({
      response: body,
      expectedChallenge: req.session.challenge,
      expectedOrigin: ORIGIN,
      expectedRPID: RP_ID
    });

    if (!verification.verified) {
      return res.status(400).json({ error: 'Verification failed' });
    }

    const { credentialID, credentialPublicKey, counter } = verification.registrationInfo;
    const user = users.get(req.session.username);

    // Store credential in database
    credentials.set(credentialID, {
      credentialID,
      credentialPublicKey,
      counter,
      userId: user.id,
      createdAt: new Date()
    });

    // Clear challenge
    delete req.session.challenge;

    res.json({ verified: true, message: 'Registration successful' });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

app.listen(3000, () => console.log('WebAuthn registration on 3000'));
```

**Expected Output (for `POST /register/options`):**
```json
{
  "challenge": "base64url-challenge",
  "rp": { "name": "My WebAuthn App", "id": "localhost" },
  "user": { "id": "base64url-user-id", "name": "alice", "displayName": "alice" },
  "pubKeyCredParams": [
    { "alg": -7, "type": "public-key" },
    { "alg": -257, "type": "public-key" }
  ],
  "timeout": 60000,
  "attestation": "none"
}
```

**Expected Output (for `POST /register/verify` with valid response):**
```json
{
  "verified": true,
  "message": "Registration successful"
}
```

**Why this output:** The server generates registration options with a random challenge, stores it in the session, and sends them to the client. The client calls `navigator.credentials.create()` with these options, prompts the user to create a passkey (via biometrics or PIN), and returns the attestation response. The server verifies the challenge, origin, and attestation, then stores the credential's public key and ID. The challenge is single-use and cleared after verification.

### Sub-Feature 1.3: Authentication Flow (Assertion)

#### Definitions

**Core Definition:** Authentication (assertion) is the ceremony where the client proves possession of a registered credential by signing a challenge with the private key.

**Technical Definition:** The authentication ceremony begins with the server generating a challenge and authentication options (including `allowCredentials`). The client calls `navigator.credentials.get()`, which prompts the user to authenticate. The authenticator signs the challenge with the private key and returns an assertion response. The server verifies the signature using the stored public key, checks the challenge, origin, and counter, and then establishes a session.

**Beginner-Friendly Explanation:** Authentication is like proving you have the key to a lock. The server sends a random challenge, your device signs it with your private key, and the server verifies the signature using the public key it stored during registration. If the signature is valid, you're logged in.

#### Syntax Rules and Structure

**Server-side Authentication Options:**
```js
const { generateAuthenticationOptions } = require('@simplewebauthn/server');

const options = await generateAuthenticationOptions({
  rpID: 'example.com',
  allowCredentials: userCredentials.map(cred => ({
    id: cred.credentialID,
    type: 'public-key',
    transports: cred.transports
  })),
  userVerification: 'preferred'
});
```

**Server-side Verification:**
```js
const { verifyAuthenticationResponse } = require('@simplewebauthn/server');

const verification = await verifyAuthenticationResponse({
  response: req.body.assertionResponse,
  expectedChallenge: storedChallenge,
  expectedOrigin: 'https://example.com',
  expectedRPID: 'example.com',
  credential: {
    id: storedCredential.credentialID,
    publicKey: storedCredential.credentialPublicKey,
    counter: storedCredential.counter
  }
});

if (verification.verified) {
  // Update counter for replay protection
  await updateCounter(
    storedCredential.credentialID,
    verification.authenticationInfo.newCounter
  );
  // Establish session
}
```

#### Annotated Code Example

```js
// webauthn-authentication.js
const express = require('express');
const session = require('express-session');
const {
  generateAuthenticationOptions,
  verifyAuthenticationResponse
} = require('@simplewebauthn/server');
const app = express();

app.use(express.json());
app.use(session({ secret: 'secret', resave: false, saveUninitialized: false }));

const RP_ID = 'localhost';
const ORIGIN = 'http://localhost:3000';

// Simulated credential store
const credentials = new Map();

// Step 1: Generate authentication options
app.post('/login/options', async (req, res) => {
  const { username } = req.body;

  // Get user's credentials
  const userCredentials = [...credentials.values()]
    .filter(c => c.username === username);

  if (userCredentials.length === 0) {
    return res.status(400).json({ error: 'No credentials found' });
  }

  const options = await generateAuthenticationOptions({
    rpID: RP_ID,
    allowCredentials: userCredentials.map(c => ({
      id: c.credentialID,
      type: 'public-key',
      transports: c.transports
    })),
    userVerification: 'preferred'
  });

  req.session.challenge = options.challenge;
  req.session.username = username;

  res.json(options);
});

// Step 2: Verify authentication response
app.post('/login/verify', async (req, res) => {
  const { body } = req;

  try {
    const credential = credentials.get(body.id);
    if (!credential) {
      return res.status(400).json({ error: 'Credential not found' });
    }

    const verification = await verifyAuthenticationResponse({
      response: body,
      expectedChallenge: req.session.challenge,
      expectedOrigin: ORIGIN,
      expectedRPID: RP_ID,
      credential: {
        id: credential.credentialID,
        publicKey: credential.credentialPublicKey,
        counter: credential.counter
      }
    });

    if (!verification.verified) {
      return res.status(400).json({ error: 'Verification failed' });
    }

    // Update counter for replay protection
    credential.counter = verification.authenticationInfo.newCounter;

    // Clear challenge
    delete req.session.challenge;

    // Establish session
    req.session.userId = credential.userId;

    res.json({
      verified: true,
      message: 'Authentication successful',
      userId: credential.userId
    });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

app.listen(3000, () => console.log('WebAuthn authentication on 3000'));
```

**Expected Output (for `POST /login/options`):**
```json
{
  "challenge": "base64url-challenge",
  "rpId": "localhost",
  "allowCredentials": [
    { "id": "base64url-credential-id", "type": "public-key", "transports": ["internal"] }
  ],
  "userVerification": "preferred",
  "timeout": 60000
}
```

**Expected Output (for `POST /login/verify` with valid response):**
```json
{
  "verified": true,
  "message": "Authentication successful",
  "userId": "user-uuid"
}
```

**Why this output:** The server generates authentication options with the user's credential IDs and a new challenge. The client calls `navigator.credentials.get()`, the authenticator signs the challenge, and the client returns the assertion. The server verifies the signature using the stored public key, checks the challenge and origin, updates the counter, and establishes a session.

### Real-World Cases

- **Google, Microsoft, Apple:** All support passkeys for passwordless login.
- **GitHub:** Supports passkeys for account security.
- **Shopify:** Offers passkey login for merchants.
- **Enterprise SSO:** Passkeys replace passwords for workforce identity.
- **Banking apps:** Use WebAuthn for high-assurance authentication.

---

## Core Concept 2: Passwordless Email/SMS

### Definitions

**Core Definition:** Passwordless email/SMS authentication uses one-time passwords (OTPs) or magic links delivered through email or SMS to verify user identity without requiring a password.

**Technical Definition:** Magic links are cryptographically signed, single-use URLs sent to the user's email that authenticate the session when clicked. OTPs are short numeric codes (typically 4–8 digits) generated using a cryptographically secure pseudo-random number generator (CSPRNG), stored server-side with a short time-to-live (TTL), and delivered via SMS or email. Both methods prove possession of the email account or phone number, delegating authentication to the security of that channel.

**Beginner-Friendly Explanation:** Instead of typing a password, you enter your email or phone number and receive either a link (magic link) or a short code (OTP). Clicking the link or entering the code proves you control that email or phone, and you're logged in. It's like a temporary key that self-destructs after use.

### Purposes

- To eliminate password-related risks (reuse, credential stuffing, phishing for passwords).
- To provide a low-friction login experience (no password to remember).
- To serve as a fallback or recovery method for passkey users.
- To enable mobile-first authentication without requiring app installation.

### Sub-Feature 2.1: Magic Links

#### Definitions

**Core Definition:** A magic link is a single-use, time-limited URL sent to a user's email that authenticates the user when clicked.

**Technical Definition:** The server generates a cryptographically signed token (JWT or opaque token) and embeds it in a URL. The URL is sent to the user's email. When the user clicks it, the server validates the token (signature, expiry, single-use flag) and issues a session. The token is then invalidated to prevent replay.

**Beginner-Friendly Explanation:** A magic link is like a "click here to log in" button sent to your email. It's a one-time-use link that expires quickly. You don't need to remember anything — just click the link and you're in.

#### Syntax Rules and Structure

**Token Generation:**
```js
const jwt = require('jsonwebtoken');

const token = jwt.sign(
  { email: user.email, purpose: 'magic-link' },
  process.env.MAGIC_LINK_SECRET,
  { expiresIn: '15m' }
);

const magicLink = `https://example.com/auth/callback?token=${token}`;
```

**Token Verification:**
```js
app.get('/auth/callback', (req, res) => {
  const { token } = req.query;

  try {
    const payload = jwt.verify(token, process.env.MAGIC_LINK_SECRET);

    // Check if token has already been used
    if (usedTokens.has(token)) {
      return res.status(400).json({ error: 'Token already used' });
    }

    usedTokens.add(token); // Mark as used

    // Find or create user
    const user = getOrCreateUser(payload.email);

    // Establish session
    req.session.userId = user.id;

    res.json({ message: 'Logged in', user });
  } catch (err) {
    res.status(400).json({ error: 'Invalid or expired token' });
  }
});
```

**Rules:**
- Tokens must be single-use; mark them as used after first validation.
- Tokens should expire quickly (15–30 minutes recommended).
- The email should display a short code (e.g., the first 6 characters) that the user can verify matches the one shown on the login page to prevent phishing.
- Rate-limit magic link requests to prevent abuse.

#### Annotated Code Example

```js
// magic-link.js
const express = require('express');
const jwt = require('jsonwebtoken');
const nodemailer = require('nodemailer');
const session = require('express-session');
const app = express();

app.use(express.json());
app.use(session({ secret: 'secret', resave: false, saveUninitialized: false }));

const usedTokens = new Set();

// Simulated email transporter
const transporter = nodemailer.createTransport({
  host: 'smtp.example.com',
  port: 587,
  auth: { user: 'noreply@example.com', pass: 'password' }
});

// Request magic link
app.post('/auth/magic-link', async (req, res) => {
  const { email } = req.body;

  const token = jwt.sign(
    { email, purpose: 'magic-link' },
    process.env.MAGIC_LINK_SECRET,
    { expiresIn: '15m' }
  );

  const magicLink = `http://localhost:3000/auth/callback?token=${token}`;
  const shortCode = token.slice(0, 6).toUpperCase();

  // Send email
  await transporter.sendMail({
    to: email,
    subject: 'Your Magic Link',
    html: `<p>Click <a href="${magicLink}">here</a> to log in.</p>
           <p>Or enter this code: <strong>${shortCode}</strong></p>`
  });

  res.json({
    message: 'Magic link sent',
    shortCode // For display in UI to prevent phishing
  });
});

// Verify magic link
app.get('/auth/callback', (req, res) => {
  const { token } = req.query;

  try {
    const payload = jwt.verify(token, process.env.MAGIC_LINK_SECRET);

    if (usedTokens.has(token)) {
      return res.status(400).json({ error: 'Token already used' });
    }
    usedTokens.add(token);

    const user = { id: 'user-1', email: payload.email };
    req.session.userId = user.id;

    res.json({ message: 'Logged in', user });
  } catch (err) {
    res.status(400).json({ error: 'Invalid or expired token' });
  }
});

app.listen(3000, () => console.log('Magic link auth on 3000'));
```

**Expected Output (for `POST /auth/magic-link`):**
```json
{
  "message": "Magic link sent",
  "shortCode": "EYJHBG"
}
```

**Expected Output (for clicking the magic link):**
```json
{
  "message": "Logged in",
  "user": { "id": "user-1", "email": "alice@example.com" }
}
```

**Why this output:** The server generates a signed JWT with the user's email and a 15-minute expiry, embeds it in a URL, and emails it to the user. A short code (first 6 characters) is also displayed in the UI so the user can verify the email matches their login attempt. When the user clicks the link, the server verifies the token, checks it hasn't been used, marks it as used, and establishes a session.

### Sub-Feature 2.2: OTP Generation and Verification

#### Definitions

**Core Definition:** An OTP (One-Time Password) is a short numeric code generated server-side, delivered via SMS or email, and verified within a limited time window.

**Technical Definition:** The server generates a random numeric code (typically 4–8 digits) using a CSPRNG (e.g., Node's `crypto.randomInt()` or `crypto.randomBytes()`), stores it with an expiration timestamp, and delivers it to the user via SMS or email. The user submits the code, and the server verifies it matches and hasn't expired. Attempt limits and rate limiting prevent brute-force attacks.

**Beginner-Friendly Explanation:** An OTP is like a temporary password sent to your phone or email. You enter it within a few minutes, and it expires after that. If someone tries to guess it, the server locks them out after a few failed attempts.

#### Syntax Rules and Structure

**OTP Generation:**
```js
const crypto = require('crypto');

function generateOTP(length = 6) {
  const digits = '0123456789';
  let otp = '';
  for (let i = 0; i < length; i++) {
    otp += digits[crypto.randomInt(0, 10)];
  }
  return otp;
}
```

**OTP Storage (Redis with TTL):**
```js
await redisClient.setEx(
  `otp:${phoneNumber}`,
  300, // 5 minutes TTL
  JSON.stringify({ code: otp, attempts: 0 })
);
```

**OTP Verification:**
```js
app.post('/verify-otp', async (req, res) => {
  const { phoneNumber, code } = req.body;
  const stored = await redisClient.get(`otp:${phoneNumber}`);

  if (!stored) {
    return res.status(400).json({ error: 'OTP expired or not found' });
  }

  const data = JSON.parse(stored);

  if (data.attempts >= 3) {
    await redisClient.del(`otp:${phoneNumber}`);
    return res.status(429).json({ error: 'Too many attempts' });
  }

  if (data.code !== code) {
    data.attempts += 1;
    await redisClient.setEx(`otp:${phoneNumber}`, 300, JSON.stringify(data));
    return res.status(400).json({ error: 'Invalid code', attemptsLeft: 3 - data.attempts });
  }

  await redisClient.del(`otp:${phoneNumber}`);
  // Establish session
  res.json({ verified: true });
});
```

**Rules:**
- Use `crypto.randomInt()` or `crypto.randomBytes()` — never `Math.random()`.
- Store OTPs with a TTL (5–10 minutes recommended).
- Limit failed attempts (3–5) and invalidate the OTP after exceeding the limit.
- Rate-limit OTP requests (e.g., 5 per phone number per 15 minutes).
- Only the most recent OTP should be valid; issuing a new one invalidates the previous.
- OTPs should be invalidated immediately after successful use.

#### Annotated Code Example

```js
// otp-auth.js
const express = require('express');
const crypto = require('crypto');
const redis = require('redis');
const app = express();

app.use(express.json());

const redisClient = redis.createClient();
redisClient.connect();

// Generate secure OTP
function generateOTP(length = 6) {
  let otp = '';
  for (let i = 0; i < length; i++) {
    otp += crypto.randomInt(0, 10);
  }
  return otp;
}

// Request OTP
app.post('/auth/otp/request', async (req, res) => {
  const { phoneNumber } = req.body;

  // Rate limiting check
  const rateKey = `otp-rate:${phoneNumber}`;
  const rateCount = await redisClient.incr(rateKey);
  if (rateCount === 1) {
    await redisClient.expire(rateKey, 900); // 15 minutes
  }
  if (rateCount > 5) {
    return res.status(429).json({ error: 'Too many OTP requests. Try again later.' });
  }

  const otp = generateOTP(6);
  const expiresAt = Date.now() + 5 * 60 * 1000; // 5 minutes

  // Store OTP with TTL
  await redisClient.setEx(
    `otp:${phoneNumber}`,
    300,
    JSON.stringify({ code: otp, attempts: 0, createdAt: Date.now() })
  );

  // Send OTP via SMS (simulated)
  console.log(`OTP for ${phoneNumber}: ${otp}`);

  res.json({
    message: 'OTP sent',
    expiresIn: 300
  });
});

// Verify OTP
app.post('/auth/otp/verify', async (req, res) => {
  const { phoneNumber, code } = req.body;

  const stored = await redisClient.get(`otp:${phoneNumber}`);
  if (!stored) {
    return res.status(400).json({ error: 'OTP expired or not found' });
  }

  const data = JSON.parse(stored);

  if (data.attempts >= 3) {
    await redisClient.del(`otp:${phoneNumber}`);
    return res.status(429).json({ error: 'Too many attempts. Request a new OTP.' });
  }

  if (data.code !== code) {
    data.attempts += 1;
    const ttl = await redisClient.ttl(`otp:${phoneNumber}`);
    await redisClient.setEx(`otp:${phoneNumber}`, ttl, JSON.stringify(data));
    return res.status(400).json({
      error: 'Invalid code',
      attemptsLeft: 3 - data.attempts
    });
  }

  // Success — invalidate OTP
  await redisClient.del(`otp:${phoneNumber}`);

  // Establish session
  res.json({ verified: true, message: 'OTP verified successfully' });
});

app.listen(3000, () => console.log('OTP auth on 3000'));
```

**Expected Output (for `POST /auth/otp/request`):**
```json
{"message":"OTP sent","expiresIn":300}
```

**Expected Output (for `POST /auth/otp/verify` with valid code):**
```json
{"verified":true,"message":"OTP verified successfully"}
```

**Expected Output (for invalid code after 3 attempts):**
```json
{"error":"Too many attempts. Request a new OTP."}
```

**Why this output:** The server generates a 6-digit OTP using `crypto.randomInt()`, stores it in Redis with a 300-second TTL, and sends it via SMS (simulated). The verification endpoint checks the code against the stored value, tracks failed attempts, and invalidates the OTP after 3 failed attempts or successful verification. Rate limiting prevents OTP spam.

### Sub-Feature 2.3: Verification Timing Windows

#### Definitions

**Core Definition:** The verification timing window is the period during which an OTP or magic link is valid before it expires.

**Technical Definition:** OTPs and magic links must have a short TTL to limit the window of opportunity for attackers. Industry best practices recommend 5–10 minutes for OTPs and 15–30 minutes for magic links. The Auth0 default for one-time-use codes is 3 minutes. After expiry, the code or link must be rejected and a new one requested.

**Beginner-Friendly Explanation:** The timing window is like an expiration date on a coupon. After a few minutes, the code or link stops working, so even if someone intercepts it, they can't use it later.

#### Syntax Rules and Structure

| Method | Recommended TTL | Rationale |
|--------|----------------|-----------|
| SMS OTP | 5 minutes | Balance between security and usability. |
| Email OTP | 10 minutes | Email delivery may be slower. |
| Magic link | 15–30 minutes | Allows time to check email. |
| Auth0 default | 3 minutes | High-security default. |

**Rules:**
- Shorter TTLs increase security but may cause usability issues.
- Auth0 uses 3 minutes by default and allows only 3 failed attempts.
- Only the most recent OTP/link should be valid; issuing a new one invalidates the previous.
- OTPs should be invalidated immediately after successful use.
- Rate-limit OTP requests to prevent abuse (e.g., 5 per phone number per 15 minutes).

#### Constraints and Limitations

- Too short a window frustrates users (especially with SMS delivery delays).
- Too long a window increases the risk of interception and replay.
- Clock skew between server and client can affect TOTP-based verification.

### Real-World Cases

- **Auth0 Passwordless:** Supports magic links and OTPs with configurable TTL and attempt limits.
- **Slack:** Uses magic links for workspace sign-in.
- **WhatsApp:** Uses SMS OTP for account verification.
- **Banking apps:** Use OTP for transaction verification and login.
- **Discord:** Uses magic links for account verification.

---

## References

- Add Passkeys to Node and Express in 30 Minutes — https://mojoauth.com/blog/add-passkeys-to-node-and-express-in-30-minutes
- SimpleWebAuthn Documentation — https://simplewebauthn.dev/
- SimpleWebAuthn Example Project — https://simplewebauthn.dev/docs/advanced/example-project
- W3C Web Authentication: An API for accessing Public Key Credentials Level 2 — https://www.w3.org/TR/webauthn-2/
- W3C Web Authentication Level 3 (Draft) — https://www.w3.org/TR/webauthn-3/
- passkeys-codelab by GoogleChromeLabs — https://github.com/GoogleChromeLabs/passkeys-codelab
- Passport-Magic-Login — https://www.npmjs.com/package/passport-magic-login
- How to Implement OTP Authentication in Node.js — https://dev.to/raza_engage/how-to-implement-otp-authentication-in-nodejs-a-complete-guide-for-secure-user-verification-3l40
- Auth0 Passwordless Best Practices — https://auth0.com/docs/authenticate/passwordless/best-practices
- The Developer's Practical Guide to Passwordless Authentication in 2026 — https://mojoauth.com/blog/the-developers-practical-guide-to-passwordless-authentication-in-2026
- Magic Links, Passkeys, OTP, and Social Login — https://mojoauth.com/blog/magic-links-passkeys-otp-and-social-login-which-passwordless-method-fits-your-application
- OTP 2FA with Node.js and Express (Telnyx) — https://github.com/team-telnyx/telnyx-code-examples/blob/master/sms-two-factor-auth-nodejs/README.md
- passport-simple-webauthn2 — https://www.npmjs.com/package/passport-simple-webauthn2
- @forwardemail/passport-fido2-webauthn — https://www.npmjs.com/package/@forwardemail/passport-fido2-webauthn
- WebAuthn Demo with Node.js, Express.js and MongoDB — https://github.com/josephden16/webauthn-demo
- FIDO2/WebAuthn Implementation Guide — https://developers.yubico.com/WebAuthn/
- OWASP Authentication Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html