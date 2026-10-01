# Password Authentication — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Password authentication is the process of verifying a user's identity by comparing a submitted password against a stored, cryptographically hashed representation of the user's original password — without ever storing or transmitting the plaintext password itself.

**Technical Definition:** Password authentication relies on one-way cryptographic hash functions that transform a password into a fixed-length digest from which the original password cannot be feasibly recovered. Modern implementations use memory-hard algorithms (Argon2id, scrypt) or CPU-bound algorithms (bcrypt) with per-user random salts to defeat precomputation attacks (rainbow tables), optional application-level peppers for defence-in-depth against database breaches, constant-time comparison to eliminate timing side-channels, and breach-database screening to reject known-compromised credentials. Password reset flows must issue cryptographically random, single-use, time-limited tokens stored only as hashes, while preventing account enumeration through uniform responses.

**Beginner-Friendly Explanation:** Password authentication is like storing a fingerprint instead of the actual key. When you register, the system takes your password and runs it through a one-way blender (hashing). What comes out is a unique "fingerprint" that can't be reversed back into the password. When you log in later, the system blends your input the same way and compares the two fingerprints. If they match, you're in. The beauty is that even if a hacker steals the database, they only get fingerprints — not the actual keys.

### Key Characteristics

- **One-way hashing:** Password hashes are irreversible; the original password cannot be recovered from the hash.
- **Memory-hard algorithms:** Argon2id consumes configurable memory, making GPU/ASIC brute-force attacks expensive.
- **Automatic salting:** Modern algorithms generate a unique random salt per password, embedded in the hash string.
- **Pepper as defence-in-depth:** A server-side secret adds a layer that database-only breaches cannot bypass.
- **Constant-time comparison:** Verification uses timing-safe functions to prevent side-channel attacks.
- **Breach screening:** Passwords are checked against known-breached databases using k-anonymity.
- **Single-use reset tokens:** Reset tokens are random, hashed in storage, time-limited, and consumed on use.

### Prerequisites

- **Node.js runtime** (v18 or higher; `crypto.timingSafeEqual` is available since v6.6.0).
- **Express.js installed:** `npm install express`.
- **A password hashing library:** `npm install argon2` (recommended) or `npm install bcrypt`.
- **A breach-check library:** `npm install hibp` (optional but recommended).
- **Basic JavaScript knowledge:** Async/await, Promises, and environment variables.

### Related Programming Areas

- **Authentication Fundamentals:** Password authentication is one factor in the broader authentication landscape.
- **Session Management:** Authenticated sessions are established after successful password verification.
- **Cryptography:** Hashing, salts, peppers, and constant-time comparison.
- **Error Handling:** Authentication failures must return uniform 401 responses without information leakage.
- **Security Headers:** HTTPS, HSTS, and secure cookie flags protect password transmission.

### Core Concepts

1. **Password Hashing** — utilising memory-hard algorithms (Argon2id, bcrypt) instead of SHA-based algorithms.
2. **Salt & Pepper** — per-user salts and application-level peppers.
3. **Secure Password Verification** — constant-time comparison to prevent timing attacks.
4. **Password Policies** — NIST guidelines: length-focused, breach-screened.
5. **Password Reset Flows** — secure, time-limited, single-use tokens and enumeration prevention.

---

## Core Concept 1: Password Hashing

### Definitions

**Core Definition:** Password hashing is the process of transforming a plaintext password into a fixed-length, irreversible cryptographic digest using a deliberately slow, computationally expensive algorithm, so that stolen hashes are impractical to reverse into passwords.

**Technical Definition:** Password hashing algorithms are designed to be one-way (irreversible) and deliberately slow to compute. Argon2id is the winner of the Password Hashing Competition (PHC) and is the OWASP-recommended first choice for new applications as of 2026. It is memory-hard — it consumes a configurable amount of RAM — which blunts GPU and ASIC attacks far more effectively than bcrypt's CPU-bound design. bcrypt remains acceptable with a work factor of 10 or more, but is limited to 72-byte passwords. General-purpose hashes (SHA-256, SHA-512, MD5) are unacceptable for password storage because they are fast — modern GPUs can calculate billions of SHA-256 hashes per second. The hash string is self-describing: it includes the algorithm, version, cost parameters, salt, and digest in a single string (e.g., `$argon2id$v=19$m=65536,t=3,p=4$...`).

**Beginner-Friendly Explanation:** A password hash is like a meat grinder that turns a steak into mincemeat. You can't turn mincemeat back into a steak. But some grinders are fast (SHA-256) — a thief can grind millions of steaks per second. Argon2id is a slow, heavy grinder that requires you to physically carry 64 MB of meat to the grinder before it will run. That memory requirement makes it incredibly expensive for a thief to grind millions of passwords.

### Purposes

- To store passwords in a form that is computationally infeasible to reverse.
- To use memory-hard algorithms (Argon2id) that resist GPU and ASIC attacks.
- To include algorithm and cost parameters in the hash string for self-describing storage.
- To calibrate hashing time to 250–500ms on production hardware.

### Syntax Rules and Structure

#### Argon2id (Recommended)

```javascript
const argon2 = require('argon2');

// Hashing — Argon2id with OWASP-recommended parameters
const hashedPassword = await argon2.hash(password, {
  type: argon2.argon2id,
  memoryCost: 65536,    // 64 MB
  timeCost: 3,          // 3 iterations
  parallelism: 4        // 4 threads
});

// Verification — timing-safe by design
const isValid = await argon2.verify(hashedPassword, inputPassword);
```

#### bcrypt (Acceptable Alternative)

```javascript
const bcrypt = require('bcrypt');

// Hashing — cost factor 12 (2^12 iterations)
const hashedPassword = await bcrypt.hash(password, 12);

// Verification
const isValid = await bcrypt.compare(inputPassword, hashedPassword);
```

| Component | Breakdown |
|-----------|-----------|
| `argon2.hash(password, options)` | Returns a Promise resolving to the hash string. |
| `type: argon2.argon2id` | Hybrid mode — best resistance to side-channel and GPU attacks. |
| `memoryCost: 65536` | 64 MB of RAM per hash — the key to GPU resistance. |
| `timeCost: 3` | Number of passes over the memory. |
| `parallelism: 4` | Number of threads (lanes). |

**Rules:**
- **Use Argon2id for new applications** — OWASP's first choice as of 2026.
- Never use MD5, SHA-1, or SHA-256 alone for passwords — they are too fast.
- Tune hashing time to 250–500ms on production hardware — unnoticeable to users, extremely expensive for attackers.
- Store the **entire hash string** — it includes the algorithm, parameters, salt, and digest.
- Use `argon2.verify()` or `bcrypt.compare()` — they are timing-safe by design.

**Constraints:**
- bcrypt truncates input at 72 bytes — validate password length before hashing.
- Argon2 requires more memory — ensure your server has sufficient RAM for concurrent hash operations.
- `check_needs_rehash()` allows upgrading parameters without forcing password resets.

### Annotated Code Example

```javascript
// password-hashing.js
const argon2 = require('argon2');

async function registerUser(password) {
  // Argon2id with OWASP-recommended parameters
  const hash = await argon2.hash(password, {
    type: argon2.argon2id,
    memoryCost: 65536,    // 64 MB
    timeCost: 3,
    parallelism: 4
  });

  console.log('Hash string:', hash);
  // Output: $argon2id$v=19$m=65536,t=3,p=4$c29tZXNhbHQ$...

  await db.users.create({ password: hash });
}

async function loginUser(inputPassword, storedHash) {
  const isValid = await argon2.verify(storedHash, inputPassword);

  if (!isValid) {
    throw new Error('Invalid credentials');
  }

  // Check if the hash needs upgrading (e.g., parameters increased)
  if (await argon2.needsRehash(storedHash, { memoryCost: 131072, timeCost: 4 })) {
    const newHash = await argon2.hash(inputPassword, {
      type: argon2.argon2id,
      memoryCost: 131072,
      timeCost: 4,
      parallelism: 4
    });
    await db.users.update({ password: newHash });
  }

  return true;
}
```

**Expected Output (hash string):**
```
$argon2id$v=19$m=65536,t=3,p=4$c29tZXNhbHQ$RdescudvJCsgt3ub+b+dWRWJTmaaJObG
```

**Expected Output (verification):**
```
true
```

**Why this output:** The hash string encodes the algorithm (`argon2id`), version (`v=19`), parameters (`m=65536,t=3,p=4`), salt (`c29tZXNhbHQ`), and digest. `argon2.verify()` recomputes the hash with the extracted parameters and salt, then compares the digest. `needsRehash()` detects when the stored parameters are below current recommendations, allowing seamless upgrades on the next login.

### Real-World Cases

- **User registration:** Every new account stores an Argon2id hash.
- **Login verification:** `argon2.verify()` checks the submitted password against the stored hash.
- **Migration from bcrypt:** Detect bcrypt hashes on login, verify with bcrypt, re-hash with Argon2id, and update the database.
- **Parameter upgrades:** Raise `memoryCost` and `timeCost` as hardware improves, using `needsRehash()`.

---

## Core Concept 2: Salt & Pepper

### Definitions

**Core Definition:** A salt is a unique, random value generated per password and stored alongside the hash to ensure identical passwords produce different hashes; a pepper is a secret value stored separately from the database (typically in environment configuration) and mixed into the password before hashing for an additional layer of defence.

**Technical Definition:** Salting defeats rainbow-table attacks: without a salt, two users with the same password have the same hash, and a precomputed table of hash→password mappings can crack them instantly. With a salt, the attacker must compute a unique rainbow table for every possible salt — computationally prohibitive. Modern algorithms (Argon2, bcrypt) generate and embed the salt automatically in the hash string. A pepper is an application-level secret that is **not** stored in the database. If the database is breached, the attacker has the hashes and salts but not the pepper — they cannot verify guesses offline without also compromising the application server or environment configuration.

**Beginner-Friendly Explanation:** A salt is like adding a unique pinch of spice to every dish before grinding it. Even if two steaks are identical, one gets paprika and the other gets cumin — the mincemeat comes out different every time. A pepper is like a secret ingredient that only the head chef knows, kept in a locked safe, not in the kitchen with the other spices. If a thief steals the spice rack (the database), they still can't replicate the dish without the secret ingredient.

### Purposes

- To implement automatic per-user salts that defeat rainbow tables.
- To implement application-level peppers stored separately from the database.
- To ensure that identical passwords produce different hashes.
- To add a defence layer that database-only breaches cannot bypass.

### Syntax Rules and Structure

#### Automatic Salting (Argon2/bcrypt)

```javascript
const argon2 = require('argon2');

// Salt is generated automatically and embedded in the hash
const hash = await argon2.hash(password);
// $argon2id$v=19$m=65536,t=3,p=4$<salt>$<digest>
```

#### Pepper Implementation

```javascript
const crypto = require('crypto');

const PEPPER = process.env.PASSWORD_PEPPER; // 32+ char hex secret

async function hashWithPepper(password) {
  // Option A: HMAC the password with the pepper before hashing
  const peppered = crypto
    .createHmac('sha256', PEPPER)
    .update(password)
    .digest('hex');

  // Now hash the peppered value with Argon2id
  return argon2.hash(peppered, {
    type: argon2.argon2id,
    memoryCost: 65536,
    timeCost: 3,
    parallelism: 4
  });
}

async function verifyWithPepper(inputPassword, storedHash) {
  const peppered = crypto
    .createHmac('sha256', PEPPER)
    .update(inputPassword)
    .digest('hex');

  return argon2.verify(storedHash, peppered);
}
```

| Component | Breakdown |
|-----------|-----------|
| `argon2.hash(password)` | Salt generated automatically, embedded in hash. |
| `PEPPER` | Environment variable — 32+ hex characters, high entropy. |
| `createHmac('sha256', PEPPER)` | HMAC pre-hash with the pepper. |
| `argon2.verify(hash, peppered)` | Verify the peppered password. |

**Rules:**
- **Salt is automatic** with Argon2 and bcrypt — never manage salts manually.
- **Pepper must be stored separately** — environment variable or secrets manager, never in the database.
- Pepper should be at least 16 characters (ideally 32+) of high entropy, generated with `crypto.randomBytes()`.
- If the pepper is lost, all passwords become unverifiable — back it up securely.
- Use HMAC (not plain concatenation) to combine pepper with password before hashing.

**Constraints:**
- Pepper adds an operational dependency — if the pepper changes, all existing hashes become invalid.
- Some security experts debate the marginal benefit of pepper; it is defence-in-depth, not a replacement for strong hashing.
- The pepper must be rotated carefully — support dual-pepper verification during rotation.

### Annotated Code Example

```javascript
// salt-pepper.js
const argon2 = require('argon2');
const crypto = require('crypto');

const PEPPER = process.env.PASSWORD_PEPPER; // 32-byte hex from env

// Generate a secure pepper (one-time setup)
// node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

async function hashPassword(password) {
  // Step 1: Apply pepper via HMAC
  const peppered = crypto
    .createHmac('sha256', PEPPER)
    .update(password)
    .digest('hex');

  // Step 2: Argon2id with automatic salt
  return argon2.hash(peppered, {
    type: argon2.argon2id,
    memoryCost: 65536,
    timeCost: 3,
    parallelism: 4
  });
}

async function verifyPassword(inputPassword, storedHash) {
  const peppered = crypto
    .createHmac('sha256', PEPPER)
    .update(inputPassword)
    .digest('hex');

  return argon2.verify(storedHash, peppered);
}

// Usage
(async () => {
  const hash = await hashPassword('MySecurePassword123!');
  console.log('Hash:', hash.substring(0, 50) + '...');

  const valid = await verifyPassword('MySecurePassword123!', hash);
  console.log('Valid:', valid); // true

  const invalid = await verifyPassword('WrongPassword', hash);
  console.log('Invalid:', invalid); // false
})();
```

**Expected Output:**
```
Hash: $argon2id$v=19$m=65536,t=3,p=4$c29tZXNhbHQ$...
Valid: true
Invalid: false
```

**Why this output:** The password is first HMAC'd with the pepper, producing a deterministic but secret-dependent value. That value is then hashed with Argon2id, which adds a unique random salt. The stored hash contains the salt and parameters but **not** the pepper. An attacker who steals the database has the hash and salt but cannot verify guesses without the pepper.

### Real-World Cases

- **Database breach defence:** Even if the entire `users` table is exfiltrated, the pepper (stored in environment configuration) prevents offline cracking.
- **Compliance:** Some regulations require defence-in-depth measures beyond standard hashing.
- **High-value targets:** Banking and healthcare applications where breach impact is severe.

---

## Core Concept 3: Secure Password Verification

### Definitions

**Core Definition:** Secure password verification is the process of comparing a submitted password against a stored hash using a constant-time algorithm that does not leak information through execution timing differences.

**Technical Definition:** Standard string comparison operators (`===`, `==`) short-circuit at the first mismatched byte, returning faster when fewer leading bytes match. An attacker can measure response times to determine how many prefix bytes of their guess match the secret — a **timing side-channel attack**. Node.js provides `crypto.timingSafeEqual(a, b)` which performs a bitwise XOR across all bytes regardless of match position, ensuring constant execution time. Modern password hashing libraries (Argon2, bcrypt) use timing-safe comparison internally in their `verify()` methods, so developers should use `argon2.verify()` and `bcrypt.compare()` rather than comparing hashes manually.

**Beginner-Friendly Explanation:** Timing attacks are like a game of "hangman" where the judge's response time tells you how many letters you got right. If the judge answers instantly, you know your first letter was wrong. If they pause, you know you got the first letter right and should keep guessing. `timingSafeEqual` makes the judge always pause for the same amount of time — you learn nothing from timing.

### Purposes

- To use constant-time string comparison algorithms to prevent side-channel timing attacks.
- To ensure that verification time does not depend on how many characters match.
- To protect password hashes, tokens, and other secrets from byte-by-byte reconstruction.
- To use library-provided `verify()` methods that are timing-safe by design.

### Syntax Rules and Structure

```javascript
const crypto = require('crypto');

// ❌ VULNERABLE: Standard comparison short-circuits
function verifyToken(provided, expected) {
  return provided === expected; // Timing leak!
}

// ✅ SECURE: Constant-time comparison
function verifyToken(provided, expected) {
  const a = Buffer.from(provided);
  const b = Buffer.from(expected);

  // Buffers must be the same length
  if (a.length !== b.length) return false;

  return crypto.timingSafeEqual(a, b);
}

// ✅ BEST: Use library verify methods (timing-safe internally)
const isValid = await argon2.verify(storedHash, inputPassword);
// OR
const isValid = await bcrypt.compare(inputPassword, storedHash);
```

| Component | Breakdown |
|-----------|-----------|
| `crypto.timingSafeEqual(a, b)` | Compares two Buffers in constant time. |
| `a.length !== b.length` | Early return for length mismatch (leaks length only, not content). |
| `argon2.verify()` | Timing-safe verification built into the library. |
| `bcrypt.compare()` | Timing-safe verification built into the library. |

**Rules:**
- **Never** compare password hashes with `===`, `==`, or `.equals()`.
- Use `crypto.timingSafeEqual()` for custom token/signature comparison.
- Use `argon2.verify()` or `bcrypt.compare()` for password verification — they handle timing safety internally.
- Buffers passed to `timingSafeEqual` must be the same length — check length first.

**Constraints:**
- `timingSafeEqual` throws if Buffers are different lengths — always check length first.
- Timing attacks are amplified in Node.js due to the single-threaded event loop — microsecond differences become measurable.
- Length mismatch leaks the length of the secret, but not its content.

### Annotated Code Example

```javascript
// secure-verification.js
const argon2 = require('argon2');
const crypto = require('crypto');

// ❌ VULNERABLE: Manual comparison
async function insecureVerify(inputPassword, storedHash) {
  const inputHash = await argon2.hash(inputPassword);
  return inputHash === storedHash; // Timing leak + useless (different salts)
}

// ✅ SECURE: Library verification (timing-safe)
async function secureVerify(inputPassword, storedHash) {
  return argon2.verify(storedHash, inputPassword);
}

// ✅ SECURE: Custom token comparison with timingSafeEqual
function verifyResetToken(providedToken, storedTokenHash) {
  const provided = Buffer.from(providedToken, 'hex');
  const stored = Buffer.from(storedTokenHash, 'hex');

  if (provided.length !== stored.length) return false;

  return crypto.timingSafeEqual(provided, stored);
}
```

**Expected Output (secure verification):**
```javascript
const hash = await argon2.hash('correct-password');

console.log(await secureVerify('correct-password', hash)); // true
console.log(await secureVerify('wrong-password', hash));   // false
```

**Why this output:** `argon2.verify()` internally extracts the salt and parameters from the hash string, recomputes the hash of the input password, and compares the digests using a timing-safe comparison. The output is `true` for the correct password and `false` for the wrong one, with no timing difference between the two cases.

### Real-World Cases

- **Password login:** Every login uses `argon2.verify()` or `bcrypt.compare()`.
- **Password reset tokens:** Comparing the submitted token against the stored token hash.
- **API keys:** Comparing provided API keys against stored hashes.
- **Webhook signatures:** Verifying HMAC signatures with `timingSafeEqual`.

---

## Core Concept 4: Password Policies

### Definitions

**Core Definition:** A password policy is the set of rules that govern acceptable passwords at creation and change — defining minimum and maximum lengths, character restrictions, and screening requirements — with modern guidance favouring length over complexity and mandatory breach-database checks.

**Technical Definition:** NIST Special Publication 800-63B (current revision: 800-63B-4) fundamentally rewrote password policy guidance. Key requirements: allow passwords between 8 and at least 64 characters (15 recommended when the password is the sole authenticator); accept all printable ASCII and Unicode, including spaces and emoji; **do not** impose composition rules (forced symbols, mixed case, digits); **do not** require periodic rotation unless there is evidence of compromise; and **screen** new passwords against a list of known-breached and common values. Blocklist screening is mandatory under NIST — every new or changed password must be checked against known-breached credentials. The practical implementation uses the Have I Been Pwned Pwned Passwords API with k-anonymity: hash the password with SHA-1, send only the first 5 hex characters, and compare suffixes locally.

**Beginner-Friendly Explanation:** The old password rules ("one uppercase, one number, one symbol, change every 90 days") are like forcing everyone to wear a red hat, blue shirt, and green shoes. Everyone ends up looking the same — and attackers know exactly what to expect. NIST says: wear whatever you want, but make it long. And before you pick it, we'll check if it's on a list of known-stolen outfits. A 40-character passphrase that happens to be a leaked song lyric is weaker than a random 12-character string, and only a breach check catches it.

### Purposes

- To enforce modern NIST guidelines focusing on length and breach screening.
- To check passwords against breached credential databases via APIs like HaveIBeenPwned.
- To reject common passwords (e.g., "password123", "qwerty") without imposing arbitrary complexity rules.
- To support long passphrases up to 64+ characters.

### Syntax Rules and Structure

#### NIST-Compliant Validation

```javascript
const { pwnedPassword } = require('hibp');

async function validatePassword(password) {
  // 1. Length: 8 minimum, 64 maximum
  if (password.length < 8) {
    throw new Error('Password must be at least 8 characters');
  }
  if (password.length > 64) {
    throw new Error('Password must not exceed 64 characters');
  }

  // 2. No composition rules — accept any characters
  // 3. Breach screening via HIBP k-anonymity
  const breachCount = await pwnedPassword(password);

  if (breachCount > 0) {
    throw new Error(
      `This password has appeared in ${breachCount.toLocaleString()} data breaches. Please choose a different password.`
    );
  }

  return true;
}
```

| Rule | NIST Guidance | Implementation |
|------|--------------|----------------|
| Minimum length | 8 characters (15 if sole factor) | `password.length < 8` |
| Maximum length | At least 64 characters | `password.length > 64` |
| Composition rules | **None** — no forced symbols/case/digits | Do not check character classes. |
| Rotation | **None** — only on compromise | No expiry enforcement. |
| Breach screening | **Mandatory** | HIBP `pwnedPassword()` API. |
| Character support | All printable ASCII + Unicode | Accept spaces, emoji, etc. |

**Rules:**
- Minimum 8 characters; 15 recommended when password is the only factor.
- Maximum at least 64 characters — never truncate or silently strip.
- **Do not** force character composition rules.
- **Do not** require periodic rotation unless compromise is suspected.
- **Always** screen new passwords against breached databases.
- Allow all printable ASCII and Unicode characters.

**Constraints:**
- HIBP API rate limits require handling (the `hibp` library throws a `RateLimitError` with `retryAfterSeconds`).
- Breach check adds latency to registration — consider caching or asynchronous verification.
- Some legacy systems require composition rules for compliance — document deviations.

### Annotated Code Example

```javascript
// password-policy.js
const { pwnedPassword } = require('hibp');

async function validateNewPassword(password) {
  const errors = [];

  // Length checks (NIST: 8 min, 64 max)
  if (password.length < 8) {
    errors.push('Password must be at least 8 characters');
  }
  if (password.length > 64) {
    errors.push('Password must not exceed 64 characters');
  }

  // No composition rules (NIST recommends against them)

  // Breach screening (mandatory under NIST)
  try {
    const breachCount = await pwnedPassword(password);
    if (breachCount > 0) {
      errors.push(
        `This password has been found in ${breachCount.toLocaleString()} data breach(es). Please choose a different password.`
      );
    }
  } catch (err) {
    // Log the error but don't block registration if HIBP is unavailable
    console.error('Breach check failed:', err.message);
  }

  if (errors.length > 0) {
    throw new Error(errors.join('. '));
  }

  return { valid: true };
}
```

**Expected Output (for a valid, unbreached password):**
```json
{ "valid": true }
```

**Expected Output (for a breached password):**
```json
{
  "error": "This password has been found in 3,861,493 data breach(es). Please choose a different password."
}
```

**Expected Output (for a short password):**
```json
{
  "error": "Password must be at least 8 characters"
}
```

**Why this output:** The function checks length first, then performs breach screening via the HIBP k-anonymity API. A password that appears in known breaches is rejected with a descriptive count. The full password never leaves the server — only the first 5 characters of its SHA-1 hash are sent to the API.

### Real-World Cases

- **User registration:** Every new password is length-checked and breach-screened.
- **Password change:** Same validation applies when users change their password.
- **Enterprise compliance:** NIST SP 800-63B is the standard for US federal systems.
- **Consumer apps:** Breach screening protects users from credential-stuffing attacks.

---

## Core Concept 5: Password Reset Flows

### Definitions

**Core Definition:** A password reset flow is the process by which a user who has forgotten their password can securely set a new one, typically via a time-limited, single-use token delivered to their registered email address.

**Technical Definition:** The secure password reset flow has three phases: (1) **Request** — the user submits their email; the server generates a cryptographically random token (32 bytes), stores only its hash in the database with a short expiry (1 hour), and emails the plaintext token to the user. (2) **Verify** — the user clicks the link containing the token; the server hashes the submitted token and compares it against the stored hash using constant-time comparison. (3) **Reset** — if the token is valid and unexpired, the user sets a new password; the token is immediately deleted (single-use). **Account enumeration** must be prevented: the request endpoint always returns the same message ("If that email exists, a reset link has been sent") regardless of whether the account exists, and performs comparable work on both paths (hashing a dummy token if the user is not found) to prevent timing leaks.

**Beginner-Friendly Explanation:** A password reset flow is like a hotel giving you a temporary key when you've lost your room key. The temporary key only works once, only for one hour, and only for your specific room. The front desk doesn't tell strangers whether your room is occupied — they just say "if that room exists, we'll send a key to the registered guest." Even if someone tries 1,000 email addresses, they can't tell which ones have accounts.

### Purposes

- To design secure, time-limited, single-use, cryptographically random reset tokens.
- To prevent account enumeration vulnerabilities on reset forms.
- To store reset tokens as hashes, not plaintext.
- To invalidate existing reset tokens when a new one is requested.

### Syntax Rules and Structure

#### Token Schema (Mongoose)

```javascript
const mongoose = require('mongoose');

const resetTokenSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    index: true
  },
  tokenHash: {
    type: String,
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now,
    expires: '1h'  // TTL index — auto-delete after 1 hour
  }
});

module.exports = mongoose.model('PasswordResetToken', resetTokenSchema);
```

#### Request Reset (Enumeration-Resistant)

```javascript
const crypto = require('crypto');
const bcrypt = require('bcrypt');

async function requestPasswordReset(email) {
  const user = await User.findOne({ email });

  // Always return the same message — do not reveal if account exists
  if (!user) {
    // Perform dummy work to prevent timing-based enumeration
    await bcrypt.hash(crypto.randomBytes(32).toString('hex'), 10);
    return { message: 'If that email exists, a reset link has been sent.' };
  }

  // Delete existing tokens (single-use enforcement)
  await PasswordResetToken.deleteMany({ userId: user._id });

  // Generate cryptographically secure token (32 bytes = 256 bits)
  const plainToken = crypto.randomBytes(32).toString('hex');
  const tokenHash = await bcrypt.hash(plainToken, 10);

  await PasswordResetToken.create({ userId: user._id, tokenHash });

  const resetLink = `${process.env.BASE_URL}/auth/reset-password?token=${plainToken}&userId=${user._id}`;

  await sendEmail({
    to: email,
    subject: 'Reset your password',
    text: `Reset your password here: ${resetLink}\n\nThis link expires in 1 hour.`
  });

  return { message: 'If that email exists, a reset link has been sent.' };
}
```

#### Verify Token and Reset Password

```javascript
async function resetPassword(userId, plainToken, newPassword) {
  // 1. Find the token record
  const tokenRecord = await PasswordResetToken.findOne({ userId });

  if (!tokenRecord) {
    throw new Error('Invalid or expired reset token');
  }

  // 2. Compare the plain token against the stored hash (timing-safe)
  const isValid = await bcrypt.compare(plainToken, tokenRecord.tokenHash);

  if (!isValid) {
    throw new Error('Invalid or expired reset token');
  }

  // 3. Hash the new password
  const passwordHash = await bcrypt.hash(newPassword, 12);

  // 4. Update password and delete token (single-use)
  await Promise.all([
    User.findByIdAndUpdate(userId, { password: passwordHash }),
    PasswordResetToken.deleteOne({ userId })
  ]);

  return { message: 'Password updated successfully' };
}
```

| Component | Breakdown |
|-----------|-----------|
| `crypto.randomBytes(32)` | 256-bit cryptographically secure random token. |
| `bcrypt.hash(plainToken, 10)` | Store only the hash, never the plain token. |
| `expires: '1h'` | MongoDB TTL index — auto-deletes after 1 hour. |
| `deleteMany({ userId })` | Invalidate existing tokens on new request. |
| Uniform message | "If that email exists..." — prevents enumeration. |

**Rules:**
- Tokens must be **cryptographically random** (minimum 32 bytes).
- Store **only the token hash** — never the plaintext token.
- Tokens must be **single-use** — delete after successful reset.
- Tokens must be **time-limited** — 1 hour maximum.
- The request endpoint must return the **same message** regardless of account existence.
- Perform comparable work on both paths to prevent **timing-based enumeration**.
- Use **constant-time comparison** (`bcrypt.compare`) for token verification.

**Constraints:**
- Email delivery is asynchronous — tokens may arrive out of order or be delayed.
- Multiple reset requests should invalidate previous tokens.
- Rate-limit the request endpoint to prevent email bombing.

### Annotated Code Example

```javascript
// password-reset-routes.js
const express = require('express');
const crypto = require('crypto');
const bcrypt = require('bcrypt');
const router = express.Router();

// POST /auth/forgot-password
router.post('/forgot-password', async (req, res) => {
  const { email } = req.body;

  const user = await User.findOne({ email });

  if (user) {
    // Delete existing tokens
    await PasswordResetToken.deleteMany({ userId: user._id });

    // Generate 32-byte token
    const plainToken = crypto.randomBytes(32).toString('hex');
    const tokenHash = await bcrypt.hash(plainToken, 10);

    await PasswordResetToken.create({ userId: user._id, tokenHash });

    const resetLink = `${process.env.BASE_URL}/auth/reset-password?token=${plainToken}&userId=${user._id}`;
    await sendEmail(email, 'Password Reset', resetLink);
  } else {
    // Dummy work to equalise timing
    await bcrypt.hash(crypto.randomBytes(32).toString('hex'), 10);
  }

  // ALWAYS same response — prevents enumeration
  res.json({
    message: 'If that email exists, a reset link has been sent.'
  });
});

// POST /auth/reset-password
router.post('/reset-password', async (req, res) => {
  try {
    const { userId, token, newPassword } = req.body;

    if (!newPassword || newPassword.length < 8) {
      return res.status(400).json({ error: 'Password must be at least 8 characters' });
    }

    const tokenRecord = await PasswordResetToken.findOne({ userId });
    if (!tokenRecord) {
      return res.status(400).json({ error: 'Invalid or expired reset token' });
    }

    const isValid = await bcrypt.compare(token, tokenRecord.tokenHash);
    if (!isValid) {
      return res.status(400).json({ error: 'Invalid or expired reset token' });
    }

    const passwordHash = await bcrypt.hash(newPassword, 12);

    await Promise.all([
      User.findByIdAndUpdate(userId, { password: passwordHash }),
      PasswordResetToken.deleteOne({ userId })
    ]);

    res.json({ message: 'Password updated successfully' });
  } catch (err) {
    res.status(500).json({ error: 'Reset failed' });
  }
});

module.exports = router;
```

**Expected Output (for `POST /auth/forgot-password`):**
```json
{ "message": "If that email exists, a reset link has been sent." }
```

**Expected Output (for `POST /auth/reset-password` with a valid token):**
```json
{ "message": "Password updated successfully" }
```

**Expected Output (for `POST /auth/reset-password` with an invalid token):**
```json
{ "error": "Invalid or expired reset token" }
```

**Why this output:** The forgot-password endpoint never reveals whether the email exists — it returns the same message in both cases. The reset-password endpoint verifies the token hash with `bcrypt.compare` (timing-safe), updates the password, and deletes the token to enforce single-use.

### Real-World Cases

- **Consumer apps:** Standard forgot-password flow for web and mobile.
- **Enterprise systems:** Additional verification steps for high-security accounts.
- **Multi-tenant SaaS:** Reset links scoped to the tenant context.
- **Compliance:** Audit logging of all reset requests and completions.

---

## References

- OWASP Authentication Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Password Storage Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- NIST SP 800-63B-4 — Digital Identity Guidelines: Authentication and Authenticator Management — https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b-4.pdf
- Node.js — `crypto.timingSafeEqual()` — https://nodejs.org/api/crypto.html#cryptotimingsafeequala-b
- Node.js — `crypto.randomBytes()` — https://nodejs.org/api/crypto.html#cryptorandombytessize-callback
- argon2 — npm — https://www.npmjs.com/package/argon2
- bcrypt — npm — https://www.npmjs.com/package/bcrypt
- bcryptjs — npm — https://www.npmjs.com/package/bcryptjs
- hibp — npm (Have I Been Pwned SDK) — https://www.npmjs.com/package/hibp
- Have I Been Pwned — Pwned Passwords API — https://haveibeenpwned.com/API/v3#PwnedPasswords
- Shattered.io — Argon2id: Hacher un Mot de Passe en 10 Étapes [2026] — https://shattered.io/argon2-password-hashing-nodejs/
- Shattered.io — bcrypt Password Hashing in Node.js: 11 Steps [2026] — https://shattered.io/bcrypt-password-hashing-nodejs/
- Security Architecture — Password Hashing with Argon2id (Node.js) — https://mintlify.wiki/anandstays-creator/KnowledgeBase/architecture/security-practices
- TRAE-Skills — Password Hashing Best Practices — https://github.com/HighMark-31/TRAE-Skills/blob/main/security/Password_Hashing_Best_Practices.md
- OC-291: Non-Constant-Time Comparison — https://raw.githubusercontent.com/MetalLegBob/solana-vibes-kit/refs/heads/main/dinhs-bulwark/knowledge-base/patterns/crypto/OC-291-non-constant-time-comparison.md
- Password Validation: Secure Credential Checks — safeguard.sh — https://safeguard.sh/resources/blog/password-validation
- How to Implement Password Reset Flow with MongoDB — OneUptime — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mongodb-password-reset-flow/README.md
- Keep Auth Endpoints Silent About Account Existence — https://raw.githubusercontent.com/pproenca/dot-skills/refs/heads/master/skills/.experimental/adversarial-tanstack/references/sec-no-account-enumeration.md
- CVE-2026-32943: Parse Server Password Reset Single-Use Bypass — https://security.glexia.com
- RFC 9106 — Argon2 Memory-Hard Function — https://datatracker.ietf.org/doc/html/rfc9106
- RFC 2898 — PKCS #5: Password-Based Cryptography Specification (PBKDF2) — https://datatracker.ietf.org/doc/html/rfc2898