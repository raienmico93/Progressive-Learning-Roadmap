# Password Security — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Password security is the discipline of storing, verifying, and managing user passwords in a way that protects them from theft, cracking, and misuse — even if the underlying database is compromised.

**Technical Definition:** Password security encompasses six core practices: (1) **hashing** — transforming plaintext passwords into irreversible fixed-length digests using one-way cryptographic functions; (2) **salting** — appending or prepending a unique, cryptographically random value to each password before hashing to defeat rainbow tables and identical-hash detection; (3) **secure storage** — using memory-hard, computationally expensive algorithms (Argon2id, bcrypt, scrypt, PBKDF2) to slow down brute-force attacks; (4) **password reset flows** — generating cryptographically secure, short-lived, single-use tokens for account recovery; (5) **credential validation** — enforcing minimum entropy and checking against breached-password databases; and (6) **work factor tuning** — periodically increasing the computational cost of hashing as hardware advances.

**Beginner-Friendly Explanation:** When you create a password, the website should never store it as-is. Instead, it should run it through a "one-way blender" (hash function) that turns `hunter2` into something like `$2b$12$...` — and there's no way to blend it back. Every password gets a unique random "pinch of salt" mixed in before blending, so two users with the same password get completely different hashes. Even if a hacker steals the database, they can't easily reverse the hashes because the blending is intentionally slow and memory-hungry. And if you forget your password, the site sends you a special one-time link that expires quickly — because reset links are just as valuable as passwords.

### Key Characteristics

- **One-way by design:** Hashing is irreversible; there is no "decrypt" for a hash.
- **Unique salt per password:** Every stored hash includes a unique random salt, defeating precomputed tables.
- **Intentionally slow:** Modern algorithms are memory-hard and CPU-intensive, making brute-force economically infeasible.
- **Tunable cost:** Work factors (iterations, memory, parallelism) can be increased as hardware improves.
- **Defense against breaches:** Even if the database leaks, attackers face months or years of cracking effort per password.
- **Complete flow:** Password security includes registration, login, reset, validation, and rotation — not just hashing.
- **Standardised algorithms:** Argon2id (RFC 9106), bcrypt, scrypt (RFC 7914), and PBKDF2 (RFC 8018) are the accepted options.

### Prerequisites

- **Cryptographic hash fundamentals:** One-way functions, digests, collision resistance.
- **Node.js `crypto` module:** `scrypt`, `randomBytes`, `timingSafeEqual`.
- **Password hashing libraries:** `bcrypt`, `argon2`, `@node-rs/argon2`, `scrypt-js`.
- **Database fundamentals:** Unique constraints, indexes, transactions.
- **HTTP fundamentals:** POST forms, cookies, redirects, email delivery.
- **Web security concepts:** Rate limiting, CSRF, XSS, HTTPS, timing attacks.
- **Breach-detection APIs:** HaveIBeenPwned k-anonymity API.

### Related Programming Areas

- **Authentication:** Password hashing is the first step in credential verification.
- **Session management:** Post-login sessions depend on successful password verification.
- **Multi-factor authentication:** MFA complements passwords but does not replace them.
- **Passwordless authentication:** Passkeys eliminate passwords but coexist with them during migration.
- **Identity providers:** Auth0, Okta, Keycloak handle password security internally.
- **Compliance:** PCI DSS, HIPAA, SOC 2, GDPR all mandate password security controls.

### Core Concepts

1. **Password Hashing** — transforming plaintext inputs into secure representations.
2. **Salt** — appending unique, random bytes to every password prior to hashing.
3. **Secure Password Storage** — leveraging modern, resource-heavy hashing functions like Argon2id and bcrypt.
4. **Password Reset Flows** — generating secure, short-lived, single-use cryptographic reset tokens.
5. **Credential Validation** — enforcing minimum entropy, complexity rules, and checking against leaked password databases.
6. **Work Factor Tuning** — adjusting memory and time costs of hashing functions as hardware advances.

---

## Core Concept 1: Password Hashing

### Definitions

**Core Definition:** Password hashing is the one-way cryptographic transformation of a plaintext password into a fixed-length digest that cannot be reversed, used to verify passwords without storing them in plaintext.

**Technical Definition:** A password hash function takes an arbitrary-length password and produces a fixed-length output (digest) such that: (1) the same input always produces the same output; (2) different inputs produce different outputs (collision resistance); (3) the transformation is computationally infeasible to reverse. For password storage, **key derivation functions (KDFs)** like Argon2id, bcrypt, scrypt, and PBKDF2 are used — they are deliberately slow and memory-hard, unlike general-purpose hashes (SHA-256, MD5) which are designed to be fast. The output is typically encoded in a modular crypt format (MCF) string that includes the algorithm, parameters, salt, and digest.

**Beginner-Friendly Explanation:** Hashing is like putting a document through a paper shredder that turns it into a unique pattern of confetti. You can't reassemble the document from the confetti, but if you shred the same document again, you get the same pattern. So when you log in, the site shreds your password and compares the confetti pattern with the stored one — if they match, you're in. Because the shredding is one-way, even the site's own developers can't recover your password.

### Purposes

- To store passwords in a form that cannot be reversed to plaintext.
- To verify user credentials during login by comparing hash outputs.
- To protect users in the event of a database breach.
- To prevent insider access to plaintext passwords.
- To enable password-based authentication without ever storing the password itself.

### Syntax Rules and Structure

#### General Syntax (bcrypt)

```typescript
import bcrypt from 'bcrypt';

const hash = await bcrypt.hash(plaintextPassword, costFactor);
const isValid = await bcrypt.compare(plaintextPassword, storedHash);
```

| Component | Breakdown |
|-----------|-----------|
| `bcrypt.hash(password, cost)` | Returns a hash string in MCF format. |
| `cost` | Work factor (log₂ iterations); 10–12 recommended. |
| `bcrypt.compare(password, hash)` | Constant-time comparison; returns `true`/`false`. |
| Hash format | `$2b$12$saltsaltsaltsalthashhashhashhashhashhashhashhashhash` |

#### General Syntax (Argon2id)

```typescript
import argon2 from 'argon2';

const hash = await argon2.hash(password, {
  type: argon2.argon2id,
  memoryCost: 65536,   // 64 MiB
  timeCost: 3,         // iterations
  parallelism: 1,      // threads
});

const isValid = await argon2.verify(hash, password);
```

| Component | Breakdown |
|-----------|-----------|
| `type` | `argon2id` (recommended), `argon2i`, or `argon2d`. |
| `memoryCost` | Memory in KiB (65536 = 64 MiB). |
| `timeCost` | Number of passes over memory. |
| `parallelism` | Number of parallel threads. |
| `argon2.verify()` | Constant-time verification. |

#### Hash Format (Modular Crypt Format)

| Algorithm | Format | Example |
|-----------|--------|---------|
| **bcrypt** | `$2b$<cost>$<22-char salt><31-char hash>` | `$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewY5GyYqVr/if1G6` |
| **Argon2id** | `$argon2id$v=19$m=<mem>,t=<time>,p=<par>$<salt>$<hash>` | `$argon2id$v=19$m=65536,t=3,p=1$c29tZXNhbHQ$...` |
| **scrypt** | `$scrypt$ln=<N>,r=<r>,p=<p>$<salt>$<hash>` | `$scrypt$ln=17,r=8,p=1$...` |
| **PBKDF2** | `$pbkdf2-sha256$<iterations>$<salt>$<hash>` | `$pbkdf2-sha256$600000$...` |

#### Syntax Rules

- **Never use general-purpose hashes** (MD5, SHA-1, SHA-256) for passwords — they are too fast.
- **Always use a KDF** designed for passwords: Argon2id (preferred), scrypt, bcrypt, or PBKDF2.
- **Always hash with a unique salt** — bcrypt, Argon2, scrypt, and PBKDF2 generate salts automatically.
- **Use constant-time comparison** for verification — bcrypt/Argon2 libraries handle this internally.
- **Encode the hash** using the library's MCF format — never store components separately unless required.
- **Never truncate passwords** — allow the full length; bcrypt's 72-byte limit requires pre-hashing with SHA-256 or Base64 encoding.

#### Constraints and Limitations

- **bcrypt truncates at 72 bytes** — passwords longer than 72 bytes are silently truncated unless pre-hashed.
- **Argon2 requires more memory** — high memory costs may be a DoS vector on constrained servers.
- **Password hashing is CPU-intensive** — a single hash takes ~100ms; this must be rate-limited.
- **Hashing is not encryption** — there is no key to decrypt; if the hash is lost, the password is lost.
- **Hash upgrades require re-hashing** — when algorithm parameters change, passwords must be re-hashed on next login.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Hashing and Verifying Passwords with Argon2id (Node.js)

```typescript
// password.service.ts
import argon2 from 'argon2';

export class PasswordService {
  private readonly options: argon2.Options = {
    type: argon2.argon2id,
    memoryCost: 65536,   // 64 MiB
    timeCost: 3,         // 3 iterations
    parallelism: 1,
  };

  async hash(password: string): Promise<string> {
    return argon2.hash(password, this.options);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    try {
      return await argon2.verify(hash, password);
    } catch {
      return false;
    }
  }

  // Detect if a stored hash needs re-hashing (parameters changed)
  needsRehash(hash: string): boolean {
    return argon2.needsRehash(hash, this.options);
  }
}
```

```typescript
// auth.service.ts — register and login
@Injectable()
export class AuthService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly passwordService: PasswordService,
  ) {}

  async register(email: string, password: string): Promise<User> {
    const hash = await this.passwordService.hash(password);
    return this.prisma.user.create({
      data: { email: email.toLowerCase().trim(), passwordHash: hash },
    });
  }

  async login(email: string, password: string): Promise<User> {
    const user = await this.prisma.user.findUnique({
      where: { email: email.toLowerCase().trim() },
    });

    // Always perform a hash comparison to prevent user enumeration
    const hash = user?.passwordHash ?? '$argon2id$v=19$m=65536,t=3,p=1$invalid$invalid';
    const valid = await this.passwordService.verify(password, hash);

    if (!user || !valid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Re-hash if parameters have changed
    if (this.passwordService.needsRehash(user.passwordHash)) {
      const newHash = await this.passwordService.hash(password);
      await this.prisma.user.update({
        where: { id: user.id },
        data: { passwordHash: newHash },
      });
    }

    return user;
  }
}
```

**Expected behaviour:**
- `register()` hashes the password with Argon2id (64 MiB, 3 iterations) and stores the MCF string.
- `login()` verifies the password using `argon2.verify()`, which uses constant-time comparison.
- If the user does not exist, a dummy hash is still compared to prevent timing-based enumeration.
- If the stored hash was created with weaker parameters, it is re-hashed transparently.

**Why this works:** Argon2id is memory-hard, making GPU/ASIC attacks expensive. The MCF format stores algorithm, parameters, and salt alongside the hash, enabling transparent upgrades. Constant-time comparison prevents timing attacks.

### Real-World Cases

- **User registration:** Hashing passwords with Argon2id before storing them.
- **User login:** Verifying submitted passwords against stored hashes.
- **Password change:** Re-hashing the new password after verifying the old one.
- **Hash migration:** Upgrading from bcrypt to Argon2id transparently on next login.

---

## Core Concept 2: Salt

### Definitions

**Core Definition:** A salt is a unique, cryptographically random value generated per password and combined with the password before hashing, ensuring that identical passwords produce different hashes.

**Technical Definition:** A salt is a random value (typically 16–32 bytes) that is concatenated with the password before applying the hash function. Salts defeat **rainbow table attacks** (precomputed hash-to-password mappings) and **identical-hash detection** (where two users with the same password produce the same hash). Salts are stored alongside the hash (in the MCF string) and do not need to be secret — their value lies in being unique and unpredictable. Modern password hashing algorithms (bcrypt, Argon2, scrypt, PBKDF2) generate and store salts automatically; developers should never generate salts manually.

**Beginner-Friendly Explanation:** Imagine two users both choose the password "password123". Without salts, both users would have the exact same hash in the database. An attacker could build a giant lookup table (rainbow table) mapping common hashes back to passwords. With salts, each user gets a unique random string mixed in before hashing — so the same password produces completely different hashes. The salt is stored next to the hash, but it's useless to an attacker because they'd need to crack each password individually.

### Purposes

- To defeat rainbow table attacks (precomputed hash databases).
- To ensure that identical passwords produce different hashes across users.
- To prevent attackers from identifying which users share the same password.
- To make each password crack attempt unique, forcing attackers to attack each user individually.
- To enable safe password hashing without requiring a global secret (unlike pepper).

### Syntax Rules and Structure

#### Salt Generation (Handled Automatically by Libraries)

```typescript
import { randomBytes } from 'node:crypto';

// Manual salt generation (only for custom implementations)
const salt = randomBytes(16); // 16 bytes = 128 bits of entropy

// ✅ Libraries generate salts automatically
const hash = await bcrypt.hash(password, 12);          // Salt embedded
const hash2 = await argon2.hash(password, options);    // Salt embedded
```

| Component | Breakdown |
|-----------|-----------|
| `randomBytes(16)` | Cryptographically secure random bytes (128-bit salt). |
| Salt size | 16–32 bytes (128–256 bits) recommended. |
| Storage | Embedded in the MCF hash string. |
| Secret? | No — salts are public, stored alongside the hash. |

#### Salt vs. Pepper

| Feature | Salt | Pepper |
|---------|------|--------|
| **Uniqueness** | Unique per password | Shared across all passwords |
| **Storage** | Stored in the hash string | Stored separately (env var, HSM) |
| **Secret?** | No | Yes |
| **Purpose** | Defeat rainbow tables | Add defense-in-depth if DB leaks |
| **Algorithm support** | Built into bcrypt/Argon2/scrypt | Requires manual pre-hashing |

#### Syntax Rules

- **Never reuse a salt** — each password must have a unique salt.
- **Never use a predictable salt** — use `crypto.randomBytes()` or let the library handle it.
- **Never store salts separately** — MCF format embeds them in the hash string.
- **Salt length must be sufficient** — at least 16 bytes (128 bits) to prevent collisions.
- **Do not use a global salt** — that's a pepper, and it must be kept secret.
- **Let the library handle salts** — manual salt management is error-prone.

#### Constraints and Limitations

- **Salts do not slow down brute-force attacks** — they only prevent precomputation and deduplication.
- **Salts do not protect against weak passwords** — a weak password with a salt is still weak.
- **Salts must be stored** — if lost, the hash cannot be verified.
- **Salt generation must be cryptographically secure** — `Math.random()` is not acceptable.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Demonstrating Salt Uniqueness

```typescript
// salt-demo.ts
import bcrypt from 'bcrypt';
import argon2 from 'argon2';

async function demoSalts() {
  const password = 'password123';

  // bcrypt generates a unique salt for each hash
  const hash1 = await bcrypt.hash(password, 12);
  const hash2 = await bcrypt.hash(password, 12);

  console.log('Hash 1:', hash1);
  console.log('Hash 2:', hash2);
  console.log('Same?', hash1 === hash2); // false

  // Extract salt from the hash (bcrypt MCF format: $2b$12$<22-char-salt><31-char-hash>)
  const salt1 = hash1.slice(7, 29);
  const salt2 = hash2.slice(7, 29);
  console.log('Salt 1:', salt1);
  console.log('Salt 2:', salt2);
  console.log('Salts equal?', salt1 === salt2); // false

  // Both hashes verify the same password
  console.log('Verify 1:', await bcrypt.compare(password, hash1)); // true
  console.log('Verify 2:', await bcrypt.compare(password, hash2)); // true

  // Argon2 also generates unique salts
  const argon1 = await argon2.hash(password);
  const argon2Hash = await argon2.hash(password);
  console.log('Argon2 same?', argon1 === argon2Hash); // false
}

demoSalts();
```

**Expected Output:**
```
Hash 1: $2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewY5GyYqVr/if1G6
Hash 2: $2b$12$Xy9z8w7v6u5t4s3r2q1p0OYz6TtxMQJqhN8/LewY5GyYqVr/if1G6
Same? false
Salt 1: LQv3c1yqBWVHxkd0LHAkCO
Salt 2: Xy9z8w7v6u5t4s3r2q1p0O
Salts equal? false
Verify 1: true
Verify 2: true
Argon2 same? false
```

**Why this output:** Both bcrypt and Argon2 generate a unique random salt for each hash. The same password produces different hashes with different salts. Both hashes still verify the same password because the salt is embedded in the hash and used during verification.

### Real-World Cases

- **Every password hash** uses a unique salt automatically.
- **Rainbow table defense:** Attackers cannot precompute hashes because each salt produces a unique hash.
- **Deduplication prevention:** Two users with the same password have different hashes.
- **Pepper for defense-in-depth:** A secret pepper (HMAC with a secret key) can be added before hashing for extra protection.

---

## Core Concept 3: Secure Password Storage

### Definitions

**Core Definition:** Secure password storage is the practice of using modern, memory-hard, computationally expensive key derivation functions (Argon2id, bcrypt, scrypt, PBKDF2) to store passwords in a way that resists brute-force, GPU, and ASIC attacks.

**Technical Definition:** The four recommended password hashing algorithms are Argon2id, scrypt, bcrypt, and PBKDF2. **Argon2id** (RFC 9106) is the winner of the Password Hashing Competition and is recommended for new applications; it combines data-independent (Argon2i) and data-dependent (Argon2d) memory access patterns to resist side-channel and GPU attacks. **scrypt** (RFC 7914) is memory-hard and available in Node.js's `crypto` module. **bcrypt** is widely supported but limited to 72-byte passwords and lacks memory-hardness. **PBKDF2** (RFC 8018) is FIPS-approved and recommended for compliance contexts. OWASP recommends Argon2id with a minimum of 19 MiB memory, 2 iterations, and 1 degree of parallelism, or bcrypt with a cost factor of 10 or more.

**Beginner-Friendly Explanation:** Think of password hashing algorithms as different types of locks. A cheap lock (SHA-256) can be picked in seconds. A better lock (bcrypt) takes longer. An even better lock (Argon2id) doesn't just take longer — it also requires a lot of memory, which makes it much harder to attack with specialized hardware (GPUs and ASICs). The goal is to make cracking so expensive that it's not worth the attacker's time, even if they steal the database.

### Purposes

- To make brute-force attacks computationally and economically infeasible.
- To resist GPU and ASIC-accelerated cracking.
- To comply with OWASP, NIST, and industry standards.
- To provide a tunable cost that can be increased as hardware improves.
- To protect passwords even if the database is compromised.

### Syntax Rules and Structure

#### OWASP Recommended Parameters (2024)

| Algorithm | Recommended Parameters |
|-----------|------------------------|
| **Argon2id** | 19 MiB memory, 2 iterations, 1 parallelism (minimum) |
| **scrypt** | N=2^17 (131072), r=8, p=1 (minimum) |
| **bcrypt** | Cost factor 10 or more (12 recommended) |
| **PBKDF2-HMAC-SHA256** | 600,000 iterations |
| **PBKDF2-HMAC-SHA512** | 210,000 iterations |

#### Node.js Argon2id Implementation

```typescript
import argon2 from 'argon2';

const HASH_OPTIONS: argon2.Options = {
  type: argon2.argon2id,
  memoryCost: 19456,  // 19 MiB (OWASP minimum)
  timeCost: 2,        // 2 iterations
  parallelism: 1,
};

export async function hashPassword(password: string): Promise<string> {
  return argon2.hash(password, HASH_OPTIONS);
}

export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  try {
    return await argon2.verify(hash, password);
  } catch {
    return false;
  }
}
```

#### Node.js scrypt Implementation (Built-In)

```typescript
import { scrypt, randomBytes, timingSafeEqual } from 'node:crypto';
import { promisify } from 'node:util';

const scryptAsync = promisify(scrypt);

const SCRYPT_PARAMS = {
  N: 2 ** 17,   // 131072 (CPU/memory cost)
  r: 8,         // block size
  p: 1,         // parallelization
  keylen: 64,   // derived key length
};

export async function hashPassword(password: string): Promise<string> {
  const salt = randomBytes(16);
  const derivedKey = (await scryptAsync(password, salt, SCRYPT_PARAMS.keylen, {
    N: SCRYPT_PARAMS.N,
    r: SCRYPT_PARAMS.r,
    p: SCRYPT_PARAMS.p,
  })) as Buffer;

  return `$scrypt$ln=${Math.log2(SCRYPT_PARAMS.N)},r=${SCRYPT_PARAMS.r},p=${SCRYPT_PARAMS.p}$${salt.toString('base64')}$${derivedKey.toString('base64')}`;
}

export async function verifyPassword(password: string, stored: string): Promise<boolean> {
  const parts = stored.split('$');
  const params = Object.fromEntries(
    parts[2].split(',').map((p) => p.split('=')),
  );
  const salt = Buffer.from(parts[3], 'base64');
  const expected = Buffer.from(parts[4], 'base64');

  const derivedKey = (await scryptAsync(password, salt, expected.length, {
    N: 2 ** Number(params.ln),
    r: Number(params.r),
    p: Number(params.p),
  })) as Buffer;

  return timingSafeEqual(derivedKey, expected);
}
```

#### Comparison Table

| Algorithm | Memory-Hard | GPU-Resistant | Max Password | FIPS | Recommendation |
|-----------|-------------|---------------|--------------|------|----------------|
| **Argon2id** | ✅ Yes | ✅ Excellent | Unlimited | ❌ No | **First choice** for new applications |
| **scrypt** | ✅ Yes | ✅ Good | Unlimited | ❌ No | Good alternative; built into Node.js |
| **bcrypt** | ❌ No | ⚠️ Moderate | 72 bytes | ❌ No | Legacy; use cost ≥ 12 |
| **PBKDF2** | ❌ No | ❌ Weak | Unlimited | ✅ Yes | Only for FIPS compliance |
| **SHA-256/MD5** | ❌ No | ❌ None | — | — | ❌ Never use for passwords |

#### Syntax Rules

- **Prefer Argon2id** for new applications — it's the OWASP first choice.
- **Use scrypt** if Argon2id is unavailable — it's built into Node.js.
- **Use bcrypt with cost ≥ 12** if Argon2 and scrypt are unavailable.
- **Use PBKDF2 only for FIPS compliance** — with ≥ 600,000 iterations (SHA-256).
- **Never use MD5, SHA-1, or SHA-256 alone** for password storage.
- **Store hashes in MCF format** to enable algorithm migration.
- **Benchmark on your production hardware** — parameters should result in ~250ms–1s hash time.
- **Upgrade hashes transparently on login** when parameters change.

#### Constraints and Limitations

- **Argon2 memory cost must fit within server memory** — high costs can cause OOM under load.
- **bcrypt's 72-byte limit** requires pre-hashing longer passwords (SHA-256 then Base64).
- **PBKDF2 is not memory-hard** — it's vulnerable to GPU acceleration.
- **Hash upgrades require user login** — passwords cannot be re-hashed without the plaintext.
- **CPU/memory cost is a DoS vector** — rate-limit hashing endpoints to prevent exhaustion.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Argon2id with Transparent Hash Upgrade

```typescript
// password.service.ts
import argon2 from 'argon2';

const CURRENT_PARAMS: argon2.Options = {
  type: argon2.argon2id,
  memoryCost: 65536,  // 64 MiB
  timeCost: 3,
  parallelism: 1,
};

export class PasswordService {
  async hash(password: string): Promise<string> {
    return argon2.hash(password, CURRENT_PARAMS);
  }

  async verify(password: string, hash: string): Promise<{
    valid: boolean;
    needsRehash: boolean;
  }> {
    try {
      const valid = await argon2.verify(hash, password);
      const needsRehash = valid && argon2.needsRehash(hash, CURRENT_PARAMS);
      return { valid, needsRehash };
    } catch {
      return { valid: false, needsRehash: false };
    }
  }
}
```

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly passwordService: PasswordService,
  ) {}

  async login(email: string, password: string): Promise<User> {
    const user = await this.prisma.user.findUnique({ where: { email } });
    const hash = user?.passwordHash ?? '$argon2id$v=19$m=65536,t=3,p=1$invalid$invalid';

    const { valid, needsRehash } = await this.passwordService.verify(password, hash);

    if (!user || !valid) throw new UnauthorizedException('Invalid credentials');

    // Transparently upgrade the hash if parameters changed
    if (needsRehash) {
      const newHash = await this.passwordService.hash(password);
      await this.prisma.user.update({
        where: { id: user.id },
        data: { passwordHash: newHash },
      });
    }

    return user;
  }
}
```

**Expected behaviour:** When a user logs in with a password hashed using older parameters, `argon2.needsRehash()` returns `true`, and the password is re-hashed with current parameters and saved. The user is unaware of the upgrade.

**Why this works:** `argon2.needsRehash()` compares the stored hash's parameters with the current configuration. If they differ, the password is re-hashed. This enables transparent upgrades without requiring users to reset their passwords.

### Real-World Cases

- **Consumer applications:** Argon2id with 64 MiB memory and 3 iterations.
- **High-traffic APIs:** scrypt with tuned parameters to balance security and throughput.
- **FIPS-compliant systems:** PBKDF2-HMAC-SHA256 with 600,000 iterations.
- **Legacy systems:** bcrypt with cost 12 and a migration plan to Argon2id.

---

## Core Concept 4: Password Reset Flows

### Definitions

**Core Definition:** A password reset flow is a secure, self-service process by which a user who has forgotten their password can regain access to their account by proving ownership of their email address (or another verified identifier) and setting a new password.

**Technical Definition:** A password reset flow involves: (1) the user requests a reset by providing their email address; (2) the server generates a cryptographically random, single-use, time-limited reset token; (3) the token is stored as a hash (never plaintext) with an expiration (typically 15–60 minutes); (4) the token is sent to the user's verified email as a link; (5) when the user clicks the link, the server validates the token, marks it as used, and allows the user to set a new password; (6) all active sessions are invalidated. Reset tokens must be generated with `crypto.randomBytes()`, hashed before storage, single-use, and short-lived. The API response must not reveal whether the email exists (to prevent enumeration).

**Beginner-Friendly Explanation:** When you forget your password, the site sends a special one-time link to your email. The link contains a long random code that only you should have. When you click it, the site checks the code, lets you set a new password, and then burns the code so it can't be used again. The code expires after a short time, and any other devices that were logged in get kicked out — so if someone else had access, they lose it.

### Purposes

- To allow users to regain access to their accounts without support intervention.
- To prove ownership of the email address (or another verified identifier).
- To prevent account takeover via predictable or reusable reset tokens.
- To invalidate sessions after a password change, preventing persistent access by attackers.
- To provide a secure alternative to security questions (which are weak).
- To comply with security standards (OWASP, NIST) that mandate secure reset flows.

### Syntax Rules and Structure

#### Reset Token Database Schema

```prisma
model PasswordResetToken {
  id        String   @id @default(uuid())
  userId    String
  tokenHash String   @unique   // SHA-256 hash of the token
  expiresAt DateTime
  usedAt    DateTime?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([expiresAt])
}
```

| Field | Purpose |
|-------|---------|
| `tokenHash` | SHA-256 hash of the token (never store plaintext). |
| `expiresAt` | Short TTL (15–60 minutes). |
| `usedAt` | Marks the token as consumed after use. |

#### Reset Token Generation

```typescript
import { randomBytes, createHash } from 'node:crypto';

export class PasswordResetService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly passwordService: PasswordService,
    private readonly emailService: EmailService,
    private readonly sessionService: SessionService,
  ) {}

  async requestReset(email: string): Promise<void> {
    const user = await this.prisma.user.findUnique({
      where: { email: email.toLowerCase().trim() },
    });

    // Always return success to prevent email enumeration
    if (!user) return;

    // Invalidate any existing unused tokens for this user
    await this.prisma.passwordResetToken.updateMany({
      where: { userId: user.id, usedAt: null },
      data: { usedAt: new Date() },
    });

    // Generate a cryptographically secure token
    const token = randomBytes(32).toString('base64url'); // 256 bits
    const tokenHash = createHash('sha256').update(token).digest('hex');

    await this.prisma.passwordResetToken.create({
      data: {
        userId: user.id,
        tokenHash,
        expiresAt: new Date(Date.now() + 30 * 60 * 1000), // 30 minutes
      },
    });

    // Send email with the plaintext token (never store it)
    await this.emailService.sendPasswordReset(user.email, token);
  }

  async resetPassword(token: string, newPassword: string): Promise<void> {
    // Validate password policy
    const result = passwordSchema.safeParse(newPassword);
    if (!result.success) throw new BadRequestException(result.error.issues);

    // Hash the incoming token and look it up
    const tokenHash = createHash('sha256').update(token).digest('hex');
    const record = await this.prisma.passwordResetToken.findUnique({
      where: { tokenHash },
      include: { user: true },
    });

    if (!record || record.usedAt || record.expiresAt < new Date()) {
      throw new BadRequestException('Invalid or expired reset token');
    }

    // Hash the new password
    const passwordHash = await this.passwordService.hash(newPassword);

    // Use a transaction: update password, mark token used, revoke sessions
    await this.prisma.$transaction([
      this.prisma.user.update({
        where: { id: record.userId },
        data: { passwordHash },
      }),
      this.prisma.passwordResetToken.update({
        where: { id: record.id },
        data: { usedAt: new Date() },
      }),
      this.prisma.refreshToken.updateMany({
        where: { userId: record.userId },
        data: { revokedAt: new Date() },
      }),
      this.prisma.session.deleteMany({
        where: { userId: record.userId },
      }),
    ]);

    // Notify the user that their password was changed
    await this.emailService.sendPasswordChangedNotification(record.user.email);
  }
}
```

#### Syntax Rules

- **Always use `crypto.randomBytes()`** for token generation — never `Math.random()`.
- **Use at least 256 bits of entropy** (32 bytes) for reset tokens.
- **Store only the hash of the token** — never the plaintext token.
- **Set a short expiration** — 15–60 minutes is standard.
- **Single-use only** — mark the token as used after consumption.
- **Invalidate all existing tokens** when a new one is requested.
- **Invalidate all active sessions** after a password change.
- **Return the same response** whether the email exists or not (prevent enumeration).
- **Rate-limit reset requests** to prevent abuse.
- **Use HTTPS** for all reset links.
- **Notify the user** when their password is changed (both success and failure attempts).
- **Never include the token in logs** — redact it.

#### Constraints and Limitations

- **Email delivery is not guaranteed** — reset emails may be delayed or blocked.
- **Email accounts can be compromised** — if the email is compromised, so is the reset flow.
- **Token leakage via referrer headers** — reset links should not include tokens in URLs that are shared or logged.
- **Shared email addresses** — family or team email accounts can be a risk.
- **Users may reuse old passwords** — consider preventing reuse of the last N passwords.
- **Rate limiting is essential** — without it, attackers can enumerate emails or spam users.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Password Reset Flow (NestJS)

```typescript
// auth/password-reset.controller.ts
@Controller('auth/password-reset')
export class PasswordResetController {
  constructor(private readonly passwordResetService: PasswordResetService) {}

  @Post('request')
  @HttpCode(200)
  async request(@Body() dto: RequestResetDto): Promise<{ message: string }> {
    await this.passwordResetService.requestReset(dto.email);
    // Always return the same message to prevent enumeration
    return { message: 'If the email exists, a reset link has been sent.' };
  }

  @Post('confirm')
  @HttpCode(200)
  async confirm(@Body() dto: ConfirmResetDto): Promise<{ message: string }> {
    await this.passwordResetService.resetPassword(dto.token, dto.newPassword);
    return { message: 'Password has been reset. Please log in again.' };
  }
}
```

```typescript
// auth/password-reset.service.ts (continued)
async requestReset(email: string): Promise<void> {
  const normalizedEmail = email.toLowerCase().trim();

  // Rate-limit by email and IP
  await this.rateLimiter.consume(`reset:${normalizedEmail}`, 3, 15 * 60_000);

  const user = await this.prisma.user.findUnique({
    where: { email: normalizedEmail },
  });

  // Prevent enumeration: always return success
  if (!user) return;

  // Invalidate existing tokens
  await this.prisma.passwordResetToken.updateMany({
    where: { userId: user.id, usedAt: null },
    data: { usedAt: new Date() },
  });

  const token = randomBytes(32).toString('base64url');
  const tokenHash = createHash('sha256').update(token).digest('hex');

  await this.prisma.passwordResetToken.create({
    data: {
      userId: user.id,
      tokenHash,
      expiresAt: new Date(Date.now() + 30 * 60 * 1000),
    },
  });

  const resetUrl = `https://app.example.com/reset?token=${token}`;
  await this.emailService.sendPasswordReset(user.email, resetUrl);
}
```

**Expected behaviour:**
- `POST /auth/password-reset/request` with any email returns the same message.
- If the email exists, a reset link is sent with a 256-bit token valid for 30 minutes.
- `POST /auth/password-reset/confirm` validates the token, hashes the new password, marks the token as used, and revokes all sessions.
- Rate limiting prevents abuse (3 requests per 15 minutes per email).

**Why this works:** Tokens are cryptographically random (256 bits), hashed before storage (SHA-256), single-use, and short-lived. Email enumeration is prevented by returning the same response. All sessions are revoked after a password change, preventing persistent access by attackers.

### Real-World Cases

- **Consumer applications:** Self-service password reset via email link.
- **Enterprise:** Password reset via admin or SSO provider.
- **Banking:** Password reset requires additional verification (MFA, security questions, or in-branch).
- **Healthcare:** Password reset requires identity verification per HIPAA.

---

## Core Concept 5: Credential Validation

### Definitions

**Core Definition:** Credential validation is the process of enforcing password strength requirements and checking passwords against known-breached databases before accepting them during registration or password change.

**Technical Definition:** Credential validation combines several techniques: (1) **entropy requirements** — minimum length (12+ characters) and maximum length (64+ characters); (2) **complexity rules** — optional requirements for character classes (though NIST 800-63B recommends against complexity rules); (3) **blocklist checks** — rejecting common passwords, dictionary words, and breached passwords via the HaveIBeenPwned k-anonymity API; (4) **context-specific checks** — rejecting passwords containing the user's email, name, or the application name; (5) **repetition checks** — rejecting passwords with repeated or sequential characters. OWASP recommends checking every new password against a list of the top 100,000 passwords and requiring a minimum length of 12 characters (or 8 with MFA).

**Beginner-Friendly Explanation:** Credential validation is like a bouncer at the door checking IDs. Before accepting your password, the system checks: "Is it long enough? Is it on the list of known-breached passwords? Does it contain your email or your name? Is it just '12345678'?" If the answer is yes to any of these, the password is rejected, and you're asked to pick a stronger one. This prevents users from choosing passwords that attackers already know.

### Purposes

- To prevent users from choosing weak, predictable, or already-breached passwords.
- To reduce the risk of credential stuffing attacks (where attackers reuse leaked passwords).
- To enforce a minimum level of password entropy.
- To comply with OWASP and NIST recommendations.
- To protect the application and its users from account takeover.

### Syntax Rules and Structure

#### Password Policy (OWASP-Aligned)

```typescript
import { z } from 'zod';
import { createHash } from 'node:crypto';

export const passwordSchema = z
  .string()
  .min(12, 'Password must be at least 12 characters')
  .max(128, 'Password must be at most 128 characters')
  .refine(
    (pwd) => !containsSequentialChars(pwd),
    'Password must not contain sequential characters (e.g., 1234, abcd)',
  )
  .refine(
    (pwd) => !containsRepeatedChars(pwd),
    'Password must not contain repeated characters (e.g., aaaa)',
  );

function containsSequentialChars(pwd: string): boolean {
  const sequences = ['abcdefghijklmnopqrstuvwxyz', '0123456789', 'qwertyuiop'];
  for (const seq of sequences) {
    for (let i = 0; i < seq.length - 3; i++) {
      if (pwd.toLowerCase().includes(seq.slice(i, i + 4))) return true;
    }
  }
  return false;
}

function containsRepeatedChars(pwd: string): boolean {
  return /(.)\1{3,}/.test(pwd); // 4+ repeated characters
}
```

#### Breach Detection via HaveIBeenPwned

```typescript
import { createHash } from 'node:crypto';

async function isBreached(password: string): Promise<boolean> {
  const sha1 = createHash('sha1').update(password).digest('hex').toUpperCase();
  const prefix = sha1.slice(0, 5);
  const suffix = sha1.slice(5);

  const response = await fetch(
    `https://api.pwnedpasswords.com/range/${prefix}`,
    {
      headers: {
        'Add-Padding': 'true', // Enable padding to prevent response size leakage
        'User-Agent': 'MyApp-PasswordValidator',
      },
    },
  );

  if (!response.ok) {
    // Fail open (do not block registration if the API is down)
    return false;
  }

  const body = await response.text();
  return body.split('\n').some((line) => line.startsWith(suffix));
}
```

| Component | Breakdown |
|-----------|-----------|
| SHA-1 hash | Computed locally; only the first 5 chars are sent. |
| `prefix` | 5-character prefix sent to the API. |
| `suffix` | Remaining 35 characters, matched locally. |
| `Add-Padding` | Prevents response size from leaking whether the prefix exists. |
| Fail open | If the API is down, allow the password (configurable). |

#### Context-Specific Checks

```typescript
function containsContext(pwd: string, user: { email: string; name?: string }): boolean {
  const lower = pwd.toLowerCase();
  const emailLocal = user.email.split('@')[0].toLowerCase();

  if (emailLocal.length >= 3 && lower.includes(emailLocal)) return true;
  if (user.name && user.name.length >= 3 && lower.includes(user.name.toLowerCase())) return true;
  if (lower.includes('myapp')) return true;

  return false;
}
```

#### Syntax Rules

- **Enforce a minimum length of 12 characters** (8 if MFA is required).
- **Allow a maximum length of at least 64 characters** — do not truncate.
- **Do not require complexity rules** (NIST 800-63B recommends against them).
- **Check against the top 100,000 passwords** (SecLists, HaveIBeenPwned).
- **Use the HaveIBeenPwned k-anonymity API** for breach detection.
- **Reject passwords containing the user's email, name, or app name.**
- **Reject passwords with sequential or repeated characters.**
- **Provide clear, constructive error messages** — do not just say "weak password".
- **Do not block legitimate passphrases** — allow all Unicode, spaces, and long passwords.
- **Check on both registration and password change.**

#### Constraints and Limitations

- **Breach detection requires an external API** — network latency and availability are concerns.
- **K-anonymity is not perfect** — the API learns the first 5 characters of the SHA-1 hash.
- **Complexity rules frustrate users** — they often lead to predictable patterns.
- **Password meters are easily gamed** — they encourage predictable substitutions (`P@ssw0rd`).
- **Context-specific checks can be overly aggressive** — a password containing "app" might be rejected for a user named "Appleton".
- **Failing open vs. failing closed** — if the breach API is down, do you allow or reject? (Failing open is usually the right call for UX.)

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comprehensive Credential Validation (NestJS)

```typescript
// auth/password-validator.service.ts
@Injectable()
export class PasswordValidatorService {
  private readonly commonPasswords = new Set<string>(); // Loaded from a file

  constructor() {
    // Load the top 10,000 passwords at startup
    const data = readFileSync('./data/common-passwords.txt', 'utf8');
    data.split('\n').forEach((pwd) => this.commonPasswords.add(pwd.trim().toLowerCase()));
  }

  async validate(
    password: string,
    user: { email: string; name?: string },
  ): Promise<{ valid: boolean; errors: string[] }> {
    const errors: string[] = [];

    // 1. Length
    if (password.length < 12) errors.push('Password must be at least 12 characters');
    if (password.length > 128) errors.push('Password must be at most 128 characters');

    // 2. Common passwords
    if (this.commonPasswords.has(password.toLowerCase())) {
      errors.push('Password is too common');
    }

    // 3. Sequential and repeated characters
    if (containsSequentialChars(password)) {
      errors.push('Password must not contain sequential characters');
    }
    if (containsRepeatedChars(password)) {
      errors.push('Password must not contain repeated characters');
    }

    // 4. Context-specific
    if (containsContext(password, user)) {
      errors.push('Password must not contain your email, name, or the application name');
    }

    // 5. Breach detection
    if (errors.length === 0 && (await isBreached(password))) {
      errors.push('This password has appeared in a data breach and cannot be used');
    }

    return { valid: errors.length === 0, errors };
  }
}
```

```typescript
// auth/auth.service.ts — registration
async register(dto: RegisterDto): Promise<User> {
  const user = { email: dto.email.toLowerCase().trim(), name: dto.name };

  // Validate the password
  const validation = await this.passwordValidator.validate(dto.password, user);
  if (!validation.valid) {
    throw new BadRequestException({
      error: 'PasswordValidationError',
      details: validation.errors,
    });
  }

  const passwordHash = await this.passwordService.hash(dto.password);
  return this.prisma.user.create({
    data: { email: user.email, name: user.name, passwordHash },
  });
}
```

**Expected behaviour:**
- `register()` with `password123` → rejected (breached, sequential).
- `register()` with `alice@example.com` and password `alice123456` → rejected (contains email).
- `register()` with `correct horse battery staple` → accepted (long passphrase, not breached).
- `register()` with `qwertyuiop12345` → rejected (sequential).

**Why this works:** Multiple validation layers (length, common passwords, sequential characters, context, breach detection) ensure that weak or compromised passwords are rejected before hashing. Clear error messages guide the user to choose a stronger password.

### Real-World Cases

- **Consumer registration:** Blocking common passwords and breached passwords at signup.
- **Enterprise policy:** Enforcing minimum length and blocking context-specific passwords.
- **Financial services:** Combining password validation with MFA for high-security accounts.
- **Compliance:** Meeting PCI DSS, HIPAA, and SOC 2 password requirements.

---

## Core Concept 6: Work Factor Tuning

### Definitions

**Core Definition:** Work factor tuning is the practice of adjusting the computational cost (iterations, memory, parallelism) of password hashing algorithms to keep pace with hardware advances, ensuring that brute-force attacks remain economically infeasible over time.

**Technical Definition:** Work factors are the tunable parameters of password hashing algorithms: **bcrypt** uses a cost factor (log₂ iterations), **Argon2id** uses memory cost (KiB), time cost (iterations), and parallelism, **scrypt** uses N (CPU/memory cost), r (block size), and p (parallelization), and **PBKDF2** uses an iteration count. As hardware improves (Moore's Law, GPU/ASIC advancements), the cost of brute-force attacks decreases; work factors must be increased periodically to maintain a constant cost. OWASP recommends targeting a hash time of ~250ms–1s on production hardware. Parameters should be benchmarked on the actual deployment hardware and reviewed annually.

**Beginner-Friendly Explanation:** Think of work factor tuning like adjusting the difficulty of a lock. Ten years ago, a simple lock was enough because lock-picking tools were slow. Today, with faster tools, you need a more complex lock. Similarly, password hashing algorithms have adjustable "difficulty" settings. As computers get faster, you increase the difficulty so that cracking a password still takes an impractically long time. The goal is to make each hash take about 250ms to 1 second — slow enough to deter attackers, fast enough to not slow down legitimate logins.

### Purposes

- To maintain a constant cost of brute-force attacks as hardware improves.
- To balance security (slow hashing) with usability (fast logins).
- To comply with OWASP and NIST recommendations for hash parameters.
- To enable transparent upgrades of existing hashes without requiring password resets.
- To protect against GPU and ASIC-accelerated cracking.

### Syntax Rules and Structure

#### Benchmarking Hash Parameters

```typescript
import argon2 from 'argon2';
import bcrypt from 'bcrypt';
import { performance } from 'node:perf_hooks';

async function benchmark(
  name: string,
  hashFn: () => Promise<string>,
  iterations: number = 10,
): Promise<void> {
  const times: number[] = [];

  for (let i = 0; i < iterations; i++) {
    const start = performance.now();
    await hashFn();
    const end = performance.now();
    times.push(end - start);
  }

  const avg = times.reduce((a, b) => a + b, 0) / times.length;
  const min = Math.min(...times);
  const max = Math.max(...times);

  console.log(`${name}: avg=${avg.toFixed(1)}ms min=${min.toFixed(1)}ms max=${max.toFixed(1)}ms`);
}

async function tuneParameters() {
  // Benchmark Argon2id with different memory costs
  for (const memoryCost of [19456, 32768, 65536, 131072]) {
    await benchmark(`argon2id m=${memoryCost}KiB`, () =>
      argon2.hash('test-password', {
        type: argon2.argon2id,
        memoryCost,
        timeCost: 3,
        parallelism: 1,
      }),
    );
  }

  // Benchmark bcrypt with different cost factors
  for (const cost of [10, 11, 12, 13, 14]) {
    await benchmark(`bcrypt cost=${cost}`, () =>
      bcrypt.hash('test-password', cost),
    );
  }
}

tuneParameters();
```

**Expected Output (varies by hardware):**
```
argon2id m=19456KiB: avg=45.2ms min=42.1ms max=51.3ms
argon2id m=32768KiB: avg=72.1ms min=68.5ms max=78.9ms
argon2id m=65536KiB: avg=142.3ms min=138.1ms max=151.2ms
argon2id m=131072KiB: avg=287.5ms min=281.3ms max=295.1ms
bcrypt cost=10: avg=62.3ms min=59.1ms max=68.2ms
bcrypt cost=11: avg=124.1ms min=118.5ms max=132.7ms
bcrypt cost=12: avg=248.7ms min=241.2ms max=259.3ms
bcrypt cost=13: avg=497.1ms min=485.3ms max=512.4ms
bcrypt cost=14: avg=994.2ms min=978.1ms max=1012.7ms
```

**Why this output:** The benchmark measures the time for each configuration. The goal is to choose parameters that result in ~250ms–1s per hash. On this hardware, `argon2id` with 131072 KiB (128 MiB) memory and `bcrypt` with cost 12 both fall within the target range.

#### Recommended Parameters (OWASP 2024)

| Algorithm | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| **Argon2id** | 19 MiB, t=2, p=1 | 64 MiB, t=3, p=1 | OWASP first choice |
| **scrypt** | N=2^17, r=8, p=1 | N=2^17, r=8, p=1 | Built into Node.js |
| **bcrypt** | cost=10 | cost=12 | Legacy; 72-byte limit |
| **PBKDF2-HMAC-SHA256** | 600,000 | 600,000 | FIPS compliance |
| **PBKDF2-HMAC-SHA512** | 210,000 | 210,000 | FIPS compliance |

#### Transparent Hash Upgrade on Login

```typescript
@Injectable()
export class AuthService {
  async login(email: string, password: string): Promise<User> {
    const user = await this.prisma.user.findUnique({ where: { email } });
    const hash = user?.passwordHash ?? DUMMY_HASH;

    const valid = await this.passwordService.verify(password, hash);
    if (!user || !valid) throw new UnauthorizedException('Invalid credentials');

    // Check if the hash needs upgrading
    if (this.passwordService.needsRehash(hash)) {
      const newHash = await this.passwordService.hash(password);
      await this.prisma.user.update({
        where: { id: user.id },
        data: { passwordHash: newHash },
      });
    }

    return user;
  }
}
```

#### Syntax Rules

- **Benchmark on production hardware** — parameters that work on a developer laptop may be too slow on a constrained server.
- **Target 250ms–1s per hash** — slow enough to deter attackers, fast enough for UX.
- **Review parameters annually** — hardware improves; parameters should increase.
- **Increase one parameter at a time** — change memory cost first, then time cost.
- **Use `needsRehash()`** to detect when a stored hash uses outdated parameters.
- **Upgrade hashes transparently on login** — users should not be forced to reset passwords.
- **Document parameter choices** — record the rationale and the benchmark results.
- **Consider the DoS risk** — high memory costs can be exploited to exhaust server memory.
- **Rate-limit login and registration** — hashing is CPU-intensive.

#### Constraints and Limitations

- **Benchmarking requires production-like hardware** — cloud instances vary in CPU performance.
- **Memory cost is limited by server memory** — high memory costs can cause OOM under concurrent load.
- **Increasing work factor increases login latency** — a 1-second hash on every login is noticeable.
- **Older hashes cannot be upgraded without the plaintext** — upgrades only happen on login.
- **PBKDF2 and bcrypt are not memory-hard** — they are more vulnerable to GPU attacks than Argon2id and scrypt.
- **Compliance requirements may mandate specific parameters** — e.g., FIPS requires PBKDF2.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Benchmarking and Selecting Argon2id Parameters

```typescript
// scripts/benchmark-argon2.ts
import argon2 from 'argon2';
import { performance } from 'node:perf_hooks';

const TARGET_MS = 500; // Target hash time

async function measure(
  memoryCost: number,
  timeCost: number,
  parallelism: number,
): Promise<number> {
  const iterations = 5;
  const times: number[] = [];

  for (let i = 0; i < iterations; i++) {
    const start = performance.now();
    await argon2.hash('benchmark-password', {
      type: argon2.argon2id,
      memoryCost,
      timeCost,
      parallelism,
    });
    times.push(performance.now() - start);
  }

  return times.reduce((a, b) => a + b, 0) / times.length;
}

async function findOptimalParameters(): Promise<void> {
  const configs = [
    { memoryCost: 19456, timeCost: 2, parallelism: 1 },
    { memoryCost: 32768, timeCost: 3, parallelism: 1 },
    { memoryCost: 65536, timeCost: 3, parallelism: 1 },
    { memoryCost: 131072, timeCost: 4, parallelism: 1 },
    { memoryCost: 262144, timeCost: 4, parallelism: 1 },
  ];

  console.log('Benchmarking Argon2id parameters:');
  console.log('Memory (KiB) | Time | Parallelism | Avg (ms)');

  for (const config of configs) {
    const avg = await measure(config.memoryCost, config.timeCost, config.parallelism);
    console.log(
      `${config.memoryCost.toString().padStart(12)} | ${config.timeCost.toString().padStart(4)} | ${config.parallelism.toString().padStart(11)} | ${avg.toFixed(1).padStart(8)}`,
    );
  }

  console.log(`\nTarget: ~${TARGET_MS}ms per hash`);
  console.log('Choose the highest-cost configuration that stays within the target.');
}

findOptimalParameters();
```

**Expected Output (varies by hardware):**
```
Benchmarking Argon2id parameters:
Memory (KiB) | Time | Parallelism | Avg (ms)
       19456 |    2 |           1 |     42.3
       32768 |    3 |           1 |     89.7
       65536 |    3 |           1 |    142.5
      131072 |    4 |           1 |    312.8
      262144 |    4 |           1 |    612.4

Target: ~500ms per hash
Choose the highest-cost configuration that stays within the target.
```

**Why this output:** The benchmark measures the average hash time for each configuration. On this hardware, 131072 KiB with time=4 gives ~312ms, and 262144 KiB with time=4 gives ~612ms. The optimal choice depends on the target (500ms) and the server's memory capacity.

#### Example 2: Transparent Hash Upgrade on Login

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  private readonly CURRENT_PARAMS = {
    type: argon2.argon2id,
    memoryCost: 65536,
    timeCost: 3,
    parallelism: 1,
  } as const;

  async login(email: string, password: string): Promise<User> {
    const user = await this.prisma.user.findUnique({
      where: { email: email.toLowerCase().trim() },
    });

    // Dummy hash to prevent timing-based enumeration
    const hash = user?.passwordHash ?? DUMMY_ARGON2_HASH;

    const valid = await argon2.verify(hash, password).catch(() => false);
    if (!user || !valid) throw new UnauthorizedException('Invalid credentials');

    // Check if the hash was created with older parameters
    if (argon2.needsRehash(hash, this.CURRENT_PARAMS)) {
      const newHash = await argon2.hash(password, this.CURRENT_PARAMS);
      await this.prisma.user.update({
        where: { id: user.id },
        data: { passwordHash: newHash },
      });
      this.logger.log(`Upgraded password hash for user ${user.id}`);
    }

    return user;
  }
}
```

**Expected behaviour:** When a user logs in with a password hashed using older parameters (e.g., 19 MiB memory), `argon2.needsRehash()` returns `true`, the password is re-hashed with the current parameters (64 MiB), and the new hash is saved. The user experiences a slightly longer login (~150ms extra) but is unaware of the upgrade.

**Why this works:** `argon2.needsRehash()` compares the stored hash's parameters with the current configuration. If they differ, the password is re-hashed. This enables transparent upgrades without requiring password resets, ensuring that all hashes eventually use the strongest parameters.

### Real-World Cases

- **Annual security reviews:** Re-benchmarking hash parameters on new hardware.
- **Compliance audits:** Documenting parameter choices and upgrade history.
- **Cloud migration:** Re-benchmarking when moving from on-premises to cloud (or between cloud providers).
- **Post-breach response:** Increasing work factors after a breach to make future cracking more expensive.
- **Algorithm migration:** Upgrading from bcrypt to Argon2id transparently on login.

---

## References

- OWASP Cheat Sheet Series — Password Storage — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Authentication — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Forgot Password — https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html
- NIST SP 800-63B — Digital Identity Guidelines: Authentication and Lifecycle Management — https://pages.nist.gov/800-63-3/sp800-63b.html
- RFC 9106 — Argon2 Memory-Hard Function for Password Hashing and Proof-of-Work Applications — https://www.rfc-editor.org/rfc/rfc9106
- RFC 7914 — The scrypt Password-Based Key Derivation Function — https://www.rfc-editor.org/rfc/rfc7914
- RFC 8018 — PKCS #5: Password-Based Cryptography Specification Version 2.1 — https://www.rfc-editor.org/rfc/rfc8018
- NIST SP 800-132 — Recommendation for Password-Based Key Derivation — https://csrc.nist.gov/publications/detail/sp/800-132/final
- HaveIBeenPwned — Pwned Passwords API — https://haveibeenpwned.com/API/v3#PwnedPasswords
- Argon2 — Official Website — https://www.argon2.com/
- bcrypt — npm package — https://www.npmjs.com/package/bcrypt
- argon2 — npm package — https://www.npmjs.com/package/argon2
- Node.js Documentation — `crypto.scrypt()` — https://nodejs.org/api/crypto.html#cryptoscryptpassword-salt-keylen-options-callback
- Node.js Documentation — `crypto.timingSafeEqual()` — https://nodejs.org/api/crypto.html#cryptotimingsafeequala-b
- Node.js Documentation — `crypto.randomBytes()` — https://nodejs.org/api/crypto.html#cryptorandombytessize-callback
- OWASP — Credential Stuffing Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html
- SecLists — Top 10,000 Passwords — https://github.com/danielmiessler/SecLists/tree/master/Passwords
- Password Hashing Competition — https://www.password-hashing.net/
- Latacora — Cryptographic Right Answers — https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html
- Auth0 — Password Security Best Practices — https://auth0.com/blog/password-security-best-practices/