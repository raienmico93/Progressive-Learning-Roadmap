# Authentication Security in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Authentication security is the discipline of verifying the identity of users, services, and devices in a way that resists credential theft, replay, forgery, and impersonation, while protecting the secrets and tokens that underpin that verification.

**Technical Definition**
Authentication security in .NET encompasses the cryptographic storage and verification of passwords (via adaptive, salted hashing), the implementation of multi-factor authentication (TOTP, FIDO2/passkeys, or push notifications), the issuance, validation, and lifecycle management of bearer tokens (JWT access tokens, opaque refresh tokens, and session cookies), and the enforcement of authentication policies across the ASP.NET Core pipeline. The .NET platform provides first-party primitives — `PasswordHasher<TUser>` for PBKDF2-based password hashing, ASP.NET Core Identity for TOTP and passkey support, `Microsoft.AspNetCore.Authentication.JwtBearer` for token validation, and `Microsoft.IdentityModel.JsonWebTokens` for modern JWT handling — but the developer is responsible for configuring them according to current guidance (OWASP, NIST SP 800-63B) and for avoiding deprecated or insecure defaults such as `BinaryFormatter`, `TypeNameHandling.All`, and low-iteration PBKDF2.

**Beginner-Friendly Explanation**
Authentication security is "making sure the person logging in is really who they say they are — and that an attacker who steals the password database or a token still cannot get in." In C#, that means hashing passwords with a slow algorithm and a unique salt, requiring a second factor for sensitive operations, signing tokens with strong keys and validating every claim, and never storing secrets in plaintext or in source code. The framework gives you good building blocks, but the defaults are tuned for convenience, not adversarial production traffic — you have to configure them deliberately.

---

### Key Characteristics

- **Credential-protection-first** — Passwords must be stored using adaptive, salted hashing (Argon2id, bcrypt, or PBKDF2 with high iteration counts); fast hashes such as MD5 and SHA-1 are prohibited.
- **Defence against credential stuffing** — Rate limiting, lockout, breached-password screening, and consistent response timing are as important as the hash algorithm itself.
- **Phishing-resistant MFA** — TOTP is a baseline; FIDO2/passkeys are the most secure form of MFA and protect against phishing.
- **Token minimalism** — JWTs are signed, not encrypted; they must not contain passwords, secrets, or high-risk PII.
- **Short-lived access tokens + rotating refresh tokens** — Access tokens should be short-lived; refresh tokens should be rotated on every use with reuse detection.
- **Framework-supported, not framework-assumed** — `PasswordHasher<TUser>` uses PBKDF2; iteration counts and algorithms must be reviewed against current OWASP guidance, not left at defaults blindly.
- **Lifecycle-wide** — Authentication security spans registration, login, password reset, MFA enrolment, token issuance, token revocation, and account recovery.

---

### Prerequisites

- Working knowledge of C#, ASP.NET Core, and dependency injection.
- Familiarity with HTTP, cookies, and bearer tokens.
- Basic understanding of hashing, salting, and symmetric vs. asymmetric cryptography.
- Awareness of OWASP Top 10 and NIST SP 800-63B password guidance.
- A test environment with HTTPS enabled (tokens and cookies must never transit plaintext).
- Familiarity with `Microsoft.AspNetCore.Identity` and `Microsoft.AspNetCore.Authentication.JwtBearer`.

---

### Related Programming Areas

- **Password hashing** — Adaptive, salted algorithms and work-factor tuning.
- **Multi-factor authentication** — TOTP, FIDO2/passkeys, and step-up authentication.
- **Token security** — JWT validation, refresh-token rotation, and revocation.
- **Session and cookie security** — `HttpOnly`, `Secure`, `SameSite`, and security stamps.
- **Authorization** — Policies, claims, and resource-based checks that follow authentication.
- **Security fundamentals** — Threat modeling, least privilege, and secure defaults.

---

### Core Concepts / Features

1. Authentication Security (Overarching Discipline)
2. Password Hashing
3. BCrypt
4. `PasswordHasher<T>`
5. Multi-Factor Authentication (MFA)
6. Token Security
7. Authentication Security Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with "To"), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Authentication Security (Overarching Discipline)

### Definitions

**Core Definition**
Authentication security is the set of engineering practices that ensure only the rightful owner of an identity can prove that ownership, and that the proof cannot be forged, replayed, or stolen without detection.

**Technical Definition**
Authentication security in .NET is implemented through three pillars: **credential protection** (adaptive salted hashing of passwords, breached-password screening, lockout, and rate limiting), **factor diversity** (password + TOTP, FIDO2/passkeys, or push-based MFA), and **token integrity** (signed JWTs with short lifetimes, opaque refresh tokens with rotation and reuse detection, and secure cookies with `HttpOnly`/`Secure`/`SameSite`). Each pillar must be independently strong; a compromise of one must not collapse the others. The OWASP .NET Security Cheat Sheet explicitly recommends using ASP.NET Core Identity or a supported identity provider, never implementing password recovery or session management from scratch, and configuring `PasswordHasherOptions.IterationCount` to meet current PBKDF2 guidance rather than assuming the framework default is sufficient.

**Beginner-Friendly Explanation**
Authentication security is "the chain of proof that you are you." It starts with a password that is stored so that even a database thief cannot easily recover it, adds a second factor so that a stolen password is not enough, and issues tokens that are signed, short-lived, and hard to replay. The framework provides the links, but you must connect them correctly.

---

### Purposes

- To prevent credential theft from yielding account takeover.
- To resist phishing, credential stuffing, and replay attacks.
- To limit the damage of a database breach by making offline cracking infeasible.
- To satisfy NIST SP 800-63B, OWASP ASVS, and regulatory requirements.
- To provide auditable, revocable proof of identity across sessions and services.
- To support step-up authentication for sensitive operations.
- To protect users who reuse passwords across services.

---

### Syntax Rules and Structure

#### General Authentication Security Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Registration                                              │
│  - Password hashed with adaptive algorithm + unique salt    │
│  - Breached-password screening                             │
│  - MFA enrolment (TOTP or passkey)                         │
├─────────────────────────────────────────────────────────────┤
│  Login                                                      │
│  - Password verification (constant-time comparison)         │
│  - Lockout + rate limiting                                  │
│  - Second factor challenge (TOTP / passkey)                 │
├─────────────────────────────────────────────────────────────┤
│  Token issuance                                             │
│  - Short-lived access token (JWT, signed)                   │
│  - Opaque refresh token (hashed at rest, rotating)          │
│  - Secure cookie (HttpOnly, Secure, SameSite)               │
├─────────────────────────────────────────────────────────────┤
│  Token validation                                           │
│  - Signature, issuer, audience, lifetime, type             │
│  - Revocation checks where required                         │
└─────────────────────────────────────────────────────────────┘
```

**Component breakdown**

- **Registration** — Hash the password, screen for breaches, and enrol MFA.
- **Login** — Verify the hash, enforce lockout, and challenge the second factor.
- **Token issuance** — Issue short-lived access tokens and rotating refresh tokens.
- **Token validation** — Validate every claim and reject anything that fails.

#### Syntax Rules

- Never store passwords in plaintext or reversible encryption.
- Never use MD5 or SHA-1 for password hashing; use Argon2id, bcrypt, scrypt, or PBKDF2 with high iterations.
- Never implement password reset or session management from scratch.
- Always validate JWT signature, issuer, audience, and lifetime; set `RequireHttpsMetadata = true` outside development.
- Never place passwords, secrets, or high-risk PII in a JWT payload.

#### Constraints and Limitations

- Password hashing is deliberately slow; it must be tuned to the deployment hardware (target ~100–500 ms per hash).
- MFA via SMS is no longer recommended due to known attack vectors.
- JWTs are not encrypted; confidentiality requires JWE or an opaque token.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: End-to-End Authentication Flow (Conceptual)

```csharp
// Registration
var hasher = new PasswordHasher<ApplicationUser>();
user.PasswordHash = hasher.HashPassword(user, plainPassword);

// Login
var result = hasher.VerifyHashedPassword(user, user.PasswordHash, providedPassword);
if (result == PasswordVerificationResult.Success)
{
    // Issue short-lived access token + rotating refresh token
    var accessToken = GenerateJwt(user);
    var refreshToken = GenerateOpaqueRefreshToken();
    await StoreRefreshTokenAsync(user.Id, Hash(refreshToken));
}

// Validation
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = configuration["Jwt:Issuer"],
            ValidAudience = configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(configuration["Jwt:Key"]))
        };
    });
```

**Expected Output (conceptual)**

```
Registration stores an adaptive hash, not the password.
Login verifies the hash and issues a short-lived JWT plus a rotating refresh token.
Token validation rejects tokens with invalid signature, issuer, audience, or lifetime.
```

**Why this output occurs**

- Each stage uses the appropriate primitive: adaptive hashing for credentials, signed tokens for sessions, and strict validation for every request.

---

### Real-World Cases

- **SaaS platforms** — Password + TOTP MFA, short-lived JWTs, and rotating refresh tokens.
- **Financial services** — FIDO2/passkeys for high-value transactions and step-up authentication.
- **Healthcare** — PBKDF2 with OWASP-recommended iterations and strict session revocation.
- **Consumer apps** — Breached-password screening at registration and lockout on repeated failures.

---

### References

- DotNet Security Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Password Hashing: bcrypt vs. Argon2 vs. PBKDF2 — Encryption Consulting - https://www.encryptionconsulting.com/password-storage-done-right-hashing-salting-and-bcrypt-vs-argon2-vs-pbkdf2/
- How to handle JWT in .NET — WorkOS - https://webflow.workos.com/blog/how-to-handle-jwt-in-net
- Multi-factor authentication in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/57fea1b959863e300f7d7926113bb9385c39909e/aspnetcore/security/authentication/mfa.md

---

## 2. Password Hashing

### Definitions

**Core Definition**
Password hashing is the one-way transformation of a password into a fixed-length value using a deliberately slow, salted algorithm, so that the original password cannot be recovered from the stored value.

**Technical Definition**
Password hashing in .NET uses adaptive algorithms that are intentionally expensive in CPU and/or memory: **Argon2id** (OWASP first choice), **scrypt** (second), **bcrypt** (legacy but acceptable with cost ≥ 12), and **PBKDF2** (required only when FIPS validation is mandated). Each password must be combined with a unique, random salt so that identical passwords produce different hashes and precomputed rainbow tables are useless. Verification recomputes the hash with the stored salt and parameters and compares the results in constant time. The work factor (iterations, cost, or memory) must be tuned to the deployment hardware so that a single hash takes roughly 100–500 ms, making offline brute-force attacks economically infeasible while keeping login latency acceptable.

**Beginner-Friendly Explanation**
Password hashing is "turning a password into a fingerprint that cannot be reversed." You never store the password itself; you store the fingerprint. When the user logs in, you fingerprint what they typed and compare the fingerprints. Because the fingerprinting process is deliberately slow and uses a unique random ingredient (the salt), an attacker who steals the database cannot easily guess the original passwords.

---

### Purposes

- To prevent plaintext password exposure in a database breach.
- To make offline brute-force and rainbow-table attacks computationally infeasible.
- To ensure identical passwords produce different hashes via unique salts.
- To allow work factors to be increased over time as hardware improves.
- To satisfy NIST SP 800-63B and OWASP password storage requirements.
- To protect users who reuse passwords across services.

---

### Syntax Rules and Structure

#### Algorithm Selection (OWASP 2026)

| Priority | Algorithm | Parameters | When to Use |
|---|---|---|---|
| **1** | Argon2id | OWASP current parameters (e.g., 46 MiB memory) | New systems |
| **2** | scrypt | Appropriate cost parameters | When Argon2id is unavailable |
| **3** | bcrypt | Cost factor ≥ 12 | Legacy systems |
| **4** | PBKDF2 | HMAC-SHA-256 or SHA-512 with high iterations | FIPS-140 compliance only |
| **Never** | MD5, SHA-1, plain SHA-256 | — | Prohibited |

#### General Syntax (PBKDF2 via KeyDerivation — conceptual)

```csharp
using System.Security.Cryptography;
using Microsoft.AspNetCore.Cryptography.KeyDerivation;

byte[] salt = RandomNumberGenerator.GetBytes(16);
string hash = Convert.ToBase64String(
    KeyDerivation.Pbkdf2(
        password: password,
        salt: salt,
        prf: KeyDerivationPrf.HMACSHA256,
        iterationCount: 210_000,
        numBytesRequested: 32));
```

**Component breakdown**

- `salt` — Unique per password, stored alongside the hash.
- `prf` — HMAC-SHA256 or HMAC-SHA512 (not HMAC-SHA1 for new systems).
- `iterationCount` — Must meet current OWASP guidance (210,000 for PBKDF2-HMAC-SHA512).
- `numBytesRequested` — Output length (32 bytes is common).

#### Syntax Rules

- Use a unique, cryptographically random salt of at least 128 bits per password.
- Store the algorithm identifier, parameters, and salt alongside the hash.
- Verify in constant time to avoid timing side channels.
- Increase work factors over time and rehash on successful login when parameters are outdated.
- Never log passwords or hashes.

#### Constraints and Limitations

- Password length limits: bcrypt truncates at 72 bytes; use EnhancedHashPassword or pre-hashing if longer passwords must be supported.
- PBKDF2 with HMAC-SHA1 is legacy; prefer SHA-256 or SHA-512.
- Very high work factors can cause login latency and denial-of-service if not rate-limited.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: PBKDF2 with Configurable Iterations

```csharp
public static string HashPassword(string password, int iterations = 210_000)
{
    byte[] salt = RandomNumberGenerator.GetBytes(16);
    byte[] subkey = KeyDerivation.Pbkdf2(
        password, salt, KeyDerivationPrf.HMACSHA512,
        iterations, 32);

    return $"{iterations}.{Convert.ToBase64String(salt)}.{Convert.ToBase64String(subkey)}";
}

public static bool Verify(string password, string stored)
{
    var parts = stored.Split('.');
    var iterations = int.Parse(parts[0]);
    var salt = Convert.FromBase64String(parts[1]);
    var expected = Convert.FromBase64String(parts[2]);
    var actual = KeyDerivation.Pbkdf2(
        password, salt, KeyDerivationPrf.HMACSHA512,
        iterations, expected.Length);
    return CryptographicOperations.FixedTimeEquals(actual, expected);
}
```

**Expected Output (conceptual)**

```
Hash format: iterations.salt.subkey
Verification recomputes the subkey and compares in constant time.
```

**Why this output occurs**

- The stored string includes all parameters needed to verify, enabling future rehashing when parameters change.
- `FixedTimeEquals` prevents timing attacks.

---

#### Example 2: Breached-Password Screening

```csharp
// Check against a breached-password corpus (e.g., Have I Been Pwned k-anonymity API)
var isBreached = await breachedPasswordService.IsBreachedAsync(password);
if (isBreached)
    return BadRequest("This password has appeared in a data breach.");
```

**Expected Output (conceptual)**

```
Registration rejects passwords that appear in known breach corpora.
```

**Why this output occurs**

- Length and composition rules alone do not prevent users from choosing known-compromised passwords.

---

### Real-World Cases

- **Consumer registration** — Argon2id or bcrypt cost 12 with breached-password screening.
- **Enterprise identity** — PBKDF2 with FIPS-validated parameters.
- **Legacy migration** — bcrypt cost 12 with `PasswordNeedsRehash` checks to upgrade over time.

---

### References

- Password Hashing: bcrypt vs. Argon2 vs. PBKDF2 — Encryption Consulting - https://www.encryptionconsulting.com/password-storage-done-right-hashing-salting-and-bcrypt-vs-argon2-vs-pbkdf2/
- DotNet Security Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- bcrypt in C# — MojoAuth - https://mojoauth.com/security-guides/bcrypt-in-c
- ASP.NET Core Identity PasswordHasherOptions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.passwordhasheroptions

---

## 3. BCrypt

### Definitions

**Core Definition**
bcrypt is an adaptive password-hashing function that incorporates a configurable cost factor, a built-in random salt, and a 60-character output format, making it deliberately slow and resistant to GPU-based cracking.

**Technical Definition**
bcrypt uses the Blowfish cipher's key schedule with an expensive key-expansion routine. The `BCrypt.Net-Next` package is the maintained .NET implementation. `HashPassword(password, workFactor: 12)` produces a 60-character string of the form `$2a$12$[22-char salt][31-char hash]`, where `12` is the cost (2¹² key-expansion rounds). Verification reads the salt and cost from the stored hash and recomputes the hash with the same parameters. The library generates the salt automatically and includes it in the output, so no separate salt column is required. Cost factors should be chosen so that a single hash takes roughly 100–500 ms on the deployment hardware; cost 12 is the current minimum recommended for legacy systems, while Argon2id is preferred for new systems.

**Beginner-Friendly Explanation**
bcrypt is "a password hashing function that is slow on purpose." It automatically adds a random salt and mixes the password into a hash that takes a controllable amount of time to compute. The "cost factor" (work factor) controls how slow it is: cost 12 means 2¹² rounds. The slower it is, the harder it is for an attacker to guess passwords, but the slower your login becomes. You tune it so that a hash takes about a tenth of a second to half a second.

---

### Purposes

- To provide a simple, battle-tested password hashing option in .NET.
- To automatically generate and embed unique salts.
- To allow the cost factor to be increased over time.
- To resist GPU and ASIC-based brute-force attacks through memory-hard-ish key expansion.
- To produce hashes compatible with bcrypt implementations in other languages.
- To support rehashing when the cost factor becomes outdated.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using BCrypt.Net;

// Hash (cost 12 recommended minimum; library default is 11)
string hash = BCrypt.HashPassword(password, workFactor: 12);

// Verify
bool valid = BCrypt.Verify(password, hash);

// Check whether a stored hash needs rehashing at a higher cost
if (BCrypt.PasswordNeedsRehash(hash, 13))
{
    var newHash = BCrypt.HashPassword(password, workFactor: 13);
    // store newHash
}

// Enhanced variants for passwords longer than 72 bytes
string enhancedHash = BCrypt.EnhancedHashPassword(password);
bool enhancedValid = BCrypt.EnhancedVerify(password, enhancedHash);
```

**Component breakdown**

- `workFactor` — Cost exponent (2^workFactor rounds). Minimum 12 for legacy; tune to 100–500 ms.
- `hash` — 60-character output containing version, cost, salt, and hash.
- `Verify` — Returns `true`/`false`; reads parameters from the stored hash.
- `PasswordNeedsRehash` — Indicates whether the stored hash uses a lower cost than the desired target.
- `EnhancedHashPassword` / `EnhancedVerify` — Pre-hash with SHA-384 to support passwords longer than 72 bytes.

#### Syntax Rules

- Store the full 60-character hash in a column of at least 60 characters (preferably 100+ for future-proofing).
- Never truncate the hash.
- Use cost ≥ 12 for legacy systems; benchmark on deployment hardware to choose the highest cost that stays within the latency budget.
- Catch `SaltParseException` when verifying malformed hashes and treat the login as failed.
- Do not hash on every request; bcrypt is deliberately expensive.

#### Constraints and Limitations

- bcrypt truncates passwords at 72 bytes; passwords longer than 72 bytes that share the first 72 bytes verify as equal.
- The library default cost is 11, which may be below current guidance for new systems.
- bcrypt is legacy relative to Argon2id; new systems should prefer Argon2id.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Hash and Verify

```csharp
string password = "correct horse battery staple";
string hash = BCrypt.HashPassword(password, workFactor: 12);
Console.WriteLine(hash);
Console.WriteLine(hash.Length);
Console.WriteLine(BCrypt.Verify(password, hash));
Console.WriteLine(BCrypt.Verify("wrong", hash));
```

**Expected Output**

```
$2a$12$7ZsOMDMnSjET3bnJZuNNSusZzfhPuu6NaPd26zLC0o25APR48qJUa
60
True
False
```

**Why this output occurs**

- The hash encodes version (`$2a$`), cost (`12`), a 22-character salt, and a 31-character hash.
- `Verify` recomputes the hash with the stored salt and cost and compares the results.

---

#### Example 2: Rehash on Login When Cost Is Outdated

```csharp
if (BCrypt.Verify(password, storedHash))
{
    if (BCrypt.PasswordNeedsRehash(storedHash, 13))
    {
        var newHash = BCrypt.HashPassword(password, workFactor: 13);
        await userManager.UpdatePasswordHashAsync(user, newHash);
    }
    // sign in
}
```

**Expected Output (conceptual)**

```
Successful login triggers a transparent upgrade to cost 13.
The user is not prompted; the new hash replaces the old one.
```

**Why this output occurs**

- `PasswordNeedsRehash` detects outdated cost; the password is available at login to rehash.

---

### Real-World Cases

- **Legacy ASP.NET applications** — bcrypt cost 12 with rehashing to Argon2id over time.
- **Multi-language systems** — bcrypt hashes are compatible across .NET, Python, Node.js, and Java.
- **Consumer apps** — bcrypt with breached-password screening and rate limiting.

---

### References

- bcrypt in C# — MojoAuth Security Guides - https://mojoauth.com/security-guides/bcrypt-in-c
- BCrypt.Net-Next — GitHub - https://github.com/BcryptNet/bcrypt.net
- How to Implement Secure Password Hashing Using Bcrypt in .NET — C# Corner - https://www.c-sharpcorner.com/article/how-to-implement-secure-password-hashing-using-bcrypt-in-net/
- Password Hashing: bcrypt vs. Argon2 vs. PBKDF2 — Encryption Consulting - https://www.encryptionconsulting.com/password-storage-done-right-hashing-salting-and-bcrypt-vs-argon2-vs-pbkdf2/

---

## 4. `PasswordHasher<T>`

### Definitions

**Core Definition**
`PasswordHasher<TUser>` is ASP.NET Core Identity's built-in implementation of `IPasswordHasher<TUser>`, providing PBKDF2-based password hashing with a random salt, configurable iteration count, and a self-describing hash format.

**Technical Definition**
`Microsoft.AspNetCore.Identity.PasswordHasher<TUser>` implements the standard Identity password hashing. It uses PBKDF2 with HMAC-SHA512 (in .NET 8+ Identity v3), a 128-bit random salt, and a configurable iteration count. The hash format includes a format marker, iteration count, salt, and subkey, so the hasher can verify hashes produced with older parameters and report whether rehashing is needed via `PasswordVerificationResult.SuccessRehashNeeded`. The `HashPassword` method produces a base64-encoded string; `VerifyHashedPassword` returns `Failed`, `Success`, or `SuccessRehashNeeded`. The OWASP .NET Security Cheat Sheet explicitly recommends using `PasswordHasher<TUser>` rather than calling `KeyDerivation.Pbkdf2` directly, and configuring `PasswordHasherOptions.IterationCount` to meet current guidance.

**Beginner-Friendly Explanation**
`PasswordHasher<T>` is "the password hasher that comes with ASP.NET Core Identity." You do not have to write any cryptography yourself: you call `HashPassword` when a user registers, and `VerifyHashedPassword` when they log in. It automatically generates a salt, uses PBKDF2, and can tell you when a stored hash needs to be upgraded because the iteration count has increased.

---

### Purposes

- To provide a first-party, audited password hashing implementation.
- To avoid the risks of hand-rolled PBKDF2 or custom cryptography.
- To support transparent rehashing when iteration counts increase.
- To integrate seamlessly with ASP.NET Core Identity's user store.
- To satisfy OWASP guidance for using Identity's hasher rather than direct `KeyDerivation` calls.
- To allow iteration counts to be configured for the deployment hardware.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using Microsoft.AspNetCore.Identity;

var hasher = new PasswordHasher<ApplicationUser>();

// Hash
string hash = hasher.HashPassword(user, plainPassword);

// Verify
PasswordVerificationResult result = hasher.VerifyHashedPassword(
    user, hash, providedPassword);

if (result == PasswordVerificationResult.SuccessRehashNeeded)
{
    // Rehash with the current iteration count
    user.PasswordHash = hasher.HashPassword(user, providedPassword);
}
```

**Component breakdown**

- `HashPassword(user, password)` — Returns a base64-encoded hash string.
- `VerifyHashedPassword(user, hash, password)` — Returns `Failed`, `Success`, or `SuccessRehashNeeded`.
- `PasswordHasherOptions.IterationCount` — Configures PBKDF2 iterations.
- Default iteration count: 100,000 (Identity v3 / .NET 8+); OWASP recommends 210,000 for PBKDF2-HMAC-SHA512 in .NET 10+.

#### Configuration

```csharp
builder.Services.Configure<PasswordHasherOptions>(options =>
{
    options.IterationCount = 210_000; // OWASP guidance for PBKDF2-HMAC-SHA512
});
```

#### Syntax Rules

- Use `PasswordHasher<TUser>` instead of calling `KeyDerivation.Pbkdf2` directly.
- Configure `IterationCount` based on the algorithm and current OWASP guidance, and benchmark on deployment hardware.
- Handle `SuccessRehashNeeded` by rehashing and persisting the new hash.
- Do not lower the iteration count without a documented risk acceptance.

#### Constraints and Limitations

- The default iteration count may lag current OWASP guidance; explicitly configure it.
- PBKDF2 is not memory-hard; Argon2id is preferred for new systems without FIPS constraints.
- The hasher requires a `TUser` instance for API compatibility, even when the user object is not used in the hash.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Hash, Verify, and Rehash

```csharp
var hasher = new PasswordHasher<ApplicationUser>();
var user = new ApplicationUser { UserName = "alice" };

string hash = hasher.HashPassword(user, "S3cret!");
var result = hasher.VerifyHashedPassword(user, hash, "S3cret!");
Console.WriteLine(result); // Success

if (hasher.VerifyHashedPassword(user, hash, "S3cret!")
    == PasswordVerificationResult.SuccessRehashNeeded)
{
    user.PasswordHash = hasher.HashPassword(user, "S3cret!");
}
```

**Expected Output (conceptual)**

```
Success
```

**Why this output occurs**

- The hash is verified against the stored salt and iteration count.
- If the iteration count is outdated, `SuccessRehashNeeded` triggers a rehash.

---

#### Example 2: Configuring Iteration Count

```csharp
builder.Services.Configure<PasswordHasherOptions>(options =>
{
    options.IterationCount = 210_000;
});
```

**Expected Output (conceptual)**

```
All new hashes use 210,000 PBKDF2 iterations.
Existing hashes with lower counts verify successfully and report SuccessRehashNeeded.
```

**Why this output occurs**

- The hasher reads the iteration count from the stored hash during verification, so old hashes remain valid.

---

### Real-World Cases

- **ASP.NET Core Identity applications** — Default password hashing with explicit iteration configuration.
- **Migration from legacy systems** — Verify old hashes and rehash to the current iteration count on login.
- **Compliance environments** — Document the chosen iteration count against OWASP or NIST guidance.

---

### References

- PasswordHasher<TUser> Class — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.passwordhasher-1
- PasswordHasherOptions Class — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.passwordhasheroptions
- DotNet Security Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Raise Identity password-hash work factor — GitHub PR - https://github.com/Abblix/Oidc.Server.GettingStarted/pull/22

---

## 5. Multi-Factor Authentication (MFA)

### Definitions

**Core Definition**
Multi-factor authentication is the requirement that a user present two or more independent proofs of identity — something they know, something they possess, or something they are — before access is granted.

**Technical Definition**
MFA in ASP.NET Core Identity supports **TOTP** (Time-based One-Time Password) by default, compatible with Microsoft Authenticator, Google Authenticator, and other compliant apps. TOTP generates a 6-digit code every 30 seconds from a shared secret. **FIDO2/passkeys** are supported in ASP.NET Core Identity in .NET 10+ and are the most secure MFA method because they are phishing-resistant. SMS-based MFA is no longer recommended due to known attack vectors including SIM swapping and interception. MFA can be enforced per user (`TwoFactorEnabled`) or per policy (e.g., an `amr` claim requiring `mfa` for administrative pages). Step-up authentication allows a user to start with a password and be challenged for a second factor only when accessing sensitive operations.

**Beginner-Friendly Explanation**
MFA is "a second lock on the door." Even if an attacker steals your password, they still need your phone (TOTP) or your fingerprint/security key (passkey) to get in. TOTP is the code from an authenticator app; passkeys are the modern, phishing-resistant option. SMS is no longer considered safe enough. ASP.NET Core Identity gives you TOTP out of the box and passkeys in .NET 10+.

---

### Purposes

- To prevent account takeover when a password is stolen.
- To resist phishing through FIDO2/passkeys.
- To provide step-up authentication for sensitive operations.
- To satisfy regulatory requirements for strong customer authentication.
- To reduce the impact of credential stuffing and password reuse.
- To offer a phishing-resistant alternative to passwords entirely.

---

### Syntax Rules and Structure

#### MFA Support in ASP.NET Core Identity

| Method | Support | Phishing-Resistant | Recommendation |
|---|---|---|---|
| **TOTP** | Default in Identity | No | Baseline MFA |
| **FIDO2/passkeys** | .NET 10+ | Yes | Preferred MFA/passwordless |
| **SMS** | Possible | No | Not recommended |

#### General Syntax (TOTP)

```csharp
// Enable 2FA for a user
await userManager.SetTwoFactorEnabledAsync(user, true);

// Generate the shared key and QR code during enrolment
var key = await userManager.GetAuthenticatorKeyAsync(user);
var qrCodeUri = $"otpauth://totp/{issuer}:{user.Email}?secret={key}&issuer={issuer}";

// Verify a TOTP code
var isValid = await userManager.VerifyTwoFactorTokenAsync(
    user, userManager.Options.Tokens.AuthenticatorTokenProvider, code);
```

#### General Syntax (Passkeys — .NET 10+)

```csharp
// Registration
var options = await signInManager.MakePasskeyCreationOptionsAsync(
    new PasskeyUserEntity { Name = user.Email });

// Authentication
var assertionOptions = await signInManager.MakePasskeyRequestOptionsAsync(user);
```

#### Step-Up Authentication (AMR Claim)

```csharp
public class AdditionalUserClaimsPrincipalFactory
    : UserClaimsPrincipalFactory<IdentityUser, IdentityRole>
{
    public override async Task<ClaimsPrincipal> CreateAsync(IdentityUser user)
    {
        var principal = await base.CreateAsync(user);
        var identity = (ClaimsIdentity)principal.Identity;
        identity.AddClaim(new Claim("amr", user.TwoFactorEnabled ? "mfa" : "pwd"));
        return principal;
    }
}

// Policy requiring MFA
options.AddPolicy("TwoFactorEnabled",
    x => x.RequireClaim("amr", "mfa"));
```

**Component breakdown**

- `SetTwoFactorEnabledAsync` — Enables MFA for a user.
- `GetAuthenticatorKeyAsync` — Returns the shared TOTP secret for QR-code enrolment.
- `VerifyTwoFactorTokenAsync` — Validates a TOTP code.
- `amr` claim — Records the authentication methods used; `mfa` indicates a second factor was presented.
- Passkey APIs — `MakePasskeyCreationOptionsAsync` and `MakePasskeyRequestOptionsAsync` for WebAuthn flows.

#### Syntax Rules

- TOTP is enabled by default with `AddDefaultIdentity`; use `AddIdentity` if you need to control token providers.
- Rate-limit TOTP verification to prevent brute-force of the 6-digit code.
- Prefer passkeys over TOTP for phishing resistance.
- Do not use SMS as a second factor.
- Store TOTP secrets encrypted (e.g., with ASP.NET Core Data Protection).

#### Constraints and Limitations

- TOTP is phishable (a real-time phishing proxy can relay the code).
- Passkeys require a platform authenticator or FIDO2 security key; headless VMs cannot complete the flow.
- SMS is vulnerable to SIM swapping and interception.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: TOTP Enrolment and Verification

```csharp
// Enrolment
var key = await userManager.GetAuthenticatorKeyAsync(user);
if (key is null)
{
    await userManager.ResetAuthenticatorKeyAsync(user);
    key = await userManager.GetAuthenticatorKeyAsync(user);
}
var uri = $"otpauth://totp/MyApp:{user.Email}?secret={key}&issuer=MyApp";

// Verification
var code = "123456";
var valid = await userManager.VerifyTwoFactorTokenAsync(
    user, TokenOptions.DefaultAuthenticatorProvider, code);
```

**Expected Output (conceptual)**

```
The user scans the QR code, enters the 6-digit code, and MFA is enabled.
Subsequent logins require the code.
```

**Why this output occurs**

- The shared secret is generated and stored encrypted.
- The code is validated against the time-based algorithm.

---

#### Example 2: Step-Up MFA Policy

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireMfa", policy =>
        policy.RequireClaim("amr", "mfa"));
});
```

```csharp
[Authorize(Policy = "RequireMfa")]
public IActionResult AdminDashboard() => View();
```

**Expected Output (conceptual)**

```
Users authenticated with only a password are denied access to admin pages.
Users who completed MFA have the amr=mfa claim and are allowed.
```

**Why this output occurs**

- The policy checks for the `amr` claim with value `mfa`.

---

### Real-World Cases

- **Administrative consoles** — Step-up MFA for admin pages; password-only for regular pages.
- **Financial transactions** — Passkeys or TOTP for high-value operations.
- **Enterprise identity** — FIDO2 security keys for phishing-resistant MFA.
- **Consumer apps** — TOTP with authenticator apps and breached-password screening.

---

### References

- Multi-factor authentication in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/57fea1b959863e300f7d7926113bb9385c39909e/aspnetcore/security/authentication/mfa.md
- Enable Web Authentication API (WebAuthn) passkeys — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authentication/passkeys/
- ASP.NET Core 中的多重身份验证 — Microsoft Learn - https://learn.microsoft.com/zh-cn/aspnet/core/security/authentication/mfa
- Security Considerations — Duende Software - https://docs.duendesoftware.com/security-considerations/

---

## 6. Token Security

### Definitions

**Core Definition**
Token security is the practice of issuing, transmitting, validating, and revoking bearer tokens in a way that prevents forgery, replay, theft, and privilege escalation.

**Technical Definition**
Token security in .NET centres on **JWT access tokens** and **opaque refresh tokens**. JWTs are signed (JWS) using asymmetric algorithms (RS256, ES256) or symmetric algorithms (HS256); the signature provides integrity and authenticity, not confidentiality. Validation must check the signature, issuer (`iss`), audience (`aud`), lifetime (`exp`, `nbf`), and token type. Access tokens should be short-lived (minutes) and must not contain passwords, secrets, or high-risk PII. Refresh tokens should be opaque, stored as hashes at rest, rotated on every use, and bound to a token family so that reuse of a rotated token triggers family-wide revocation. The modern .NET library is `Microsoft.IdentityModel.JsonWebTokens` (`JsonWebTokenHandler`), which is async-native, AOT-compatible, and ~30% faster than the deprecated `JwtSecurityTokenHandler`. Keys should be distributed via JWKS endpoints, and `RequireHttpsMetadata = true` must be set outside development.

**Beginner-Friendly Explanation**
Token security is "making sure the badge you are given cannot be forged, copied, or used forever." A JWT is a signed badge that says who you are and what you can do. It is not encrypted — anyone can read it — so you never put secrets in it. Access badges expire quickly; refresh badges let you get a new access badge, but each refresh badge can be used only once and is rotated. If an old refresh badge is reused, the system assumes it was stolen and revokes the whole session.

---

### Purposes

- To prevent token forgery through signature validation.
- To prevent replay through short lifetimes and refresh-token rotation.
- To limit the impact of token theft through family-wide revocation.
- To avoid information disclosure by keeping secrets out of JWT payloads.
- To enable stateless authentication for APIs while retaining revocation capability for refresh tokens.
- To support key rotation through JWKS.

---

### Syntax Rules and Structure

#### JWT Validation Parameters

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://issuer.example.com";
        options.Audience = "my-api";
        options.RequireHttpsMetadata = true; // non-development
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ClockSkew = TimeSpan.FromMinutes(2)
        };
    });
```

**Component breakdown**

- `Authority` — The issuer metadata endpoint; used for JWKS discovery.
- `Audience` — The intended recipient of the token.
- `RequireHttpsMetadata` — Must be `true` outside development.
- `ValidateIssuer`, `ValidateAudience`, `ValidateLifetime`, `ValidateIssuerSigningKey` — All must be `true`.
- `ClockSkew` — Small tolerance for clock drift.

#### Refresh-Token Rotation Pattern

```
1. Client presents refresh token.
2. Server looks up the token hash.
3. If the token is already used/revoked → revoke the entire token family.
4. If valid → mark it used, issue a new access token + new refresh token.
5. Store the new refresh token hash; return both tokens to the client.
```

**Component breakdown**

- **Opaque token** — Random bytes, not a JWT.
- **Hashed at rest** — Store SHA-256 of the token, not the token itself.
- **Rotation** — Each refresh token is single-use.
- **Reuse detection** — A reused token indicates theft; revoke the family.

#### Syntax Rules

- Use asymmetric keys (RS256/ES256) for distributed systems; distribute public keys via JWKS.
- Never create an access token directly from a username/password request without MFA where required.
- Do not store access tokens server-side if they are self-contained JWTs; validate the signature and claims.
- Store refresh tokens hashed; rotate on every use; detect reuse.
- Never place passwords, secrets, or high-risk PII in a JWT payload.
- Keep `Microsoft.IdentityModel` packages patched to avoid known CVEs.

#### Constraints and Limitations

- JWTs are not encrypted; confidentiality requires JWE or an opaque token.
- JWT revocation is difficult for stateless access tokens; short lifetimes and refresh-token revocation are the practical mitigation.
- Symmetric keys (HS256) require secure key distribution to every validator; asymmetric keys are preferred for scale.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: JWT Issuance and Validation

```csharp
// Issuance (conceptual)
var claims = new[]
{
    new Claim(JwtRegisteredClaimNames.Sub, user.Id),
    new Claim(JwtRegisteredClaimNames.Email, user.Email),
    new Claim("permission", "orders:read")
};

var token = new JwtSecurityToken(
    issuer: configuration["Jwt:Issuer"],
    audience: configuration["Jwt:Audience"],
    claims: claims,
    expires: DateTime.UtcNow.AddMinutes(15),
    signingCredentials: new SigningCredentials(
        new SymmetricSecurityKey(Encoding.UTF8.GetBytes(configuration["Jwt:Key"])),
        SecurityAlgorithms.HmacSha256));

// Validation is configured in AddJwtBearer (see above).
```

**Expected Output (conceptual)**

```
A short-lived JWT is issued with sub, email, and permission claims.
Requests with invalid signature, issuer, audience, or expired lifetime are rejected with 401.
```

**Why this output occurs**

- The token is signed and includes all required claims.
- Validation parameters reject anything that does not match.

---

#### Example 2: Refresh-Token Rotation with Reuse Detection

```csharp
public async Task<TokenPair> RefreshAsync(string refreshToken)
{
    var hash = Hash(refreshToken);
    var stored = await _store.FindByHashAsync(hash);

    if (stored is null || stored.IsRevoked)
        throw new SecurityTokenException("Invalid refresh token");

    if (stored.IsUsed)
    {
        // Reuse detected — revoke the entire family
        await _store.RevokeFamilyAsync(stored.FamilyId);
        throw new SecurityTokenException("Refresh token reuse detected");
    }

    stored.IsUsed = true;
    var newAccess = GenerateJwt(stored.UserId);
    var newRefresh = GenerateOpaqueToken();
    await _store.AddAsync(new RefreshToken
    {
        UserId = stored.UserId,
        FamilyId = stored.FamilyId,
        Hash = Hash(newRefresh),
        ExpiresAt = DateTime.UtcNow.AddDays(7)
    });

    return new TokenPair(newAccess, newRefresh);
}
```

**Expected Output (conceptual)**

```
Valid unused token: new access + refresh token issued; old token marked used.
Reused token: entire family revoked; request rejected.
```

**Why this output occurs**

- Rotation makes each refresh token single-use.
- Reuse detection assumes theft and revokes the session.

---

### Real-World Cases

- **SPA + API architectures** — Short-lived JWTs for API calls; opaque refresh tokens in `HttpOnly` cookies.
- **Microservices** — Asymmetric JWTs verified by each service using JWKS.
- **Mobile apps** — Refresh tokens stored in platform secure storage with rotation.
- **OpenID Connect** — `JsonWebTokenHandler` for async validation and AOT compatibility.

---

### References

- How to handle JWT in .NET — WorkOS - https://webflow.workos.com/blog/how-to-handle-jwt-in-net
- DotNet Security Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Refresh tokens in a .NET 10 Minimal API — DEV Community - https://dev.to/…/refresh-tokens-in-a-net-10-minimal-api-rotation-reuse-detection-and-revocation
- JWT access + refresh tokens in ASP.NET Core (.NET 10) — DEV Community - https://dev.to/…/jwt-access-refresh-tokens-in-aspnet-core-net-10-rotation-revocation-and-the-bugs-to-avoid
- Konfigurowanie JWT uwierzytelniania elementu nośnego w ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/pl-pl/aspnet/core/security/authentication/configure-jwt-bearer-authentication

---

## 7. Authentication Security Best Practices

### Definitions

**Core Definition**
Authentication security best practices are empirically grounded guidelines for implementing password storage, MFA, token handling, session management, and account protection in a way that resists real-world attacks and satisfies current standards.

**Technical Definition**
Best practices in .NET authentication security span **password storage** (adaptive hashing, unique salts, breached-password screening, OWASP-aligned parameters), **account protection** (lockout, rate limiting, consistent timing, MFA), **session and cookie security** (`HttpOnly`, `Secure`, `SameSite`, security stamps, short lifetimes), **token security** (signature validation, issuer/audience/lifetime checks, short-lived access tokens, rotating refresh tokens), **redirect safety** (`LocalRedirect`, allowlists), and **input handling** (antiforgery, output encoding, safe deserialisation). They are codified in the OWASP .NET Security Cheat Sheet, NIST SP 800-63B, and Microsoft's ASP.NET Core security guidance.

**Beginner-Friendly Explanation**
Authentication best practices are "the checklist that keeps login secure." They include: hash passwords with a slow algorithm, use a unique salt, screen for breached passwords, lock accounts after repeated failures, require a second factor, use secure cookies and short-lived tokens, validate every JWT claim, never put secrets in a token, and never implement password reset or session management yourself. Following these prevents the most common authentication vulnerabilities.

---

### Purposes

- To prevent credential theft and account takeover.
- To resist credential stuffing, phishing, and replay attacks.
- To satisfy NIST SP 800-63B, OWASP ASVS, and regulatory requirements.
- To limit the blast radius of a database breach.
- To provide secure, revocable sessions across services.
- To avoid common implementation mistakes in password reset, session management, and token handling.

---

### Guidelines and Structure

#### Password Storage

```
✅ Use Argon2id (preferred), scrypt, bcrypt cost ≥ 12, or PBKDF2 with OWASP-aligned iterations.
✅ Use a unique random salt per password.
✅ Screen for breached passwords at registration and password change.
✅ Configure PasswordHasherOptions.IterationCount; do not assume the default is sufficient.
✅ Rehash on successful login when parameters are outdated.
❌ Never use MD5, SHA-1, or plain SHA-256 for passwords.
❌ Never store passwords in plaintext or reversible encryption.
```

**Sources: password hashing guidance; OWASP .NET cheat sheet**.

#### Account Protection

```
✅ Enable lockout with lockoutOnFailure: true.
✅ Rate-limit login, registration, and password recovery.
✅ Return consistent responses and comparable timing to avoid account enumeration.
✅ Protect registration and password recovery with the same protections as login.
```

**Source: OWASP .NET Security Cheat Sheet**.

#### MFA

```
✅ Enable TOTP by default with Identity.
✅ Prefer FIDO2/passkeys for phishing resistance.
✅ Use step-up MFA for sensitive operations via the amr claim.
❌ Do not use SMS as a second factor.
```

**Sources: MFA guidance**.

#### Cookies and Sessions

```
✅ HttpOnly = true.
✅ SecurePolicy = Always.
✅ Choose SameSite to fit the authentication flow; test federated sign-in.
✅ Set ExpireTimeSpan and understand sliding vs. absolute expiration.
✅ Update the security stamp on security-sensitive changes.
```

**Source: OWASP .NET Security Cheat Sheet**.

#### Tokens

```
✅ Validate signature, issuer, audience, and lifetime.
✅ Use short-lived access tokens.
✅ Rotate refresh tokens on every use; detect reuse and revoke the family.
✅ Store refresh tokens hashed at rest.
✅ Use asymmetric keys and JWKS for distributed systems.
✅ Keep Microsoft.IdentityModel packages patched.
❌ Never put passwords, secrets, or high-risk PII in a JWT.
```

**Sources: JWT guidance; refresh-token rotation**.

#### Redirects and Input

```
✅ Use LocalRedirect or Url.IsLocalUrl for return URLs.
✅ Use server-controlled allowlists for external redirects.
✅ Enable antiforgery protection for state-changing operations.
✅ Encode output; do not use Html.Raw for untrusted content.
✅ Use System.Text.Json, not BinaryFormatter.
```

**Source: OWASP .NET Security Cheat Sheet**.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Secure Cookie Configuration

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Lax; // adjust for federated flows
    options.ExpireTimeSpan = TimeSpan.FromMinutes(30);
    options.SlidingExpiration = false; // absolute session limit
});
```

**Expected Output (conceptual)**

```
Cookies are not accessible to JavaScript (HttpOnly).
Cookies are only sent over HTTPS (Secure).
Cookies are not sent on cross-site subrequests unless appropriate (SameSite).
Sessions expire after 30 minutes regardless of activity.
```

**Why this output occurs**

- Each flag closes a specific attack vector: XSS cookie theft, plaintext transmission, and CSRF.

---

#### Example 2: Lockout and Rate Limiting

```csharp
builder.Services.Configure<IdentityOptions>(options =>
{
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
    options.Lockout.AllowedForNewUsers = true;
});

var result = await signInManager.PasswordSignInAsync(
    user, password, isPersistent: false, lockoutOnFailure: true);
```

**Expected Output (conceptual)**

```
After 5 failed attempts, the account is locked for 15 minutes.
Successful login before lockout resets the counter.
```

**Why this output occurs**

- `lockoutOnFailure: true` counts failures and triggers lockout.

---

#### Example 3: Security Stamp Validation

```csharp
builder.Services.Configure<SecurityStampValidatorOptions>(options =>
{
    options.ValidationInterval = TimeSpan.FromMinutes(5);
});

// On password change or role change:
await userManager.UpdateSecurityStampAsync(user);
```

**Expected Output (conceptual)**

```
Existing cookies are rejected at the next stamp validation after a security-sensitive change.
Revocation is not necessarily immediate; it occurs within the validation interval.
```

**Why this output occurs**

- The security stamp is embedded in the cookie; changing it invalidates existing cookies.

---

### Real-World Cases

- **ASP.NET Core Identity applications** — Lockout, MFA, secure cookies, and iteration-count configuration.
- **APIs with JWTs** — Strict validation, short lifetimes, rotating refresh tokens, and JWKS.
- **Multi-tenant SaaS** — Per-tenant password policies and MFA enforcement.
- **Regulated industries** — Documented alignment with NIST SP 800-63B and OWASP ASVS.

---

### References

- DotNet Security Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Password Hashing: bcrypt vs. Argon2 vs. PBKDF2 — Encryption Consulting - https://www.encryptionconsulting.com/password-storage-done-right-hashing-salting-and-bcrypt-vs-argon2-vs-pbkdf2/
- Multi-factor authentication in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/57fea1b959863e300f7d7926113bb9385c39909e/aspnetcore/security/authentication/mfa.md
- How to handle JWT in .NET — WorkOS - https://webflow.workos.com/blog/how-to-handle-jwt-in-net
- Refresh tokens in a .NET 10 Minimal API — DEV Community - https://dev.to/…/refresh-tokens-in-a-net-10-minimal-api-rotation-reuse-detection-and-revocation
- Enable Web Authentication API (WebAuthn) passkeys — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authentication/passkeys/
- bcrypt in C# — MojoAuth - https://mojoauth.com/security-guides/bcrypt-in-c
- PasswordHasherOptions Class — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.passwordhasheroptions

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`BinaryFormatter`** | Obsolete and insecure; do not use for untrusted data |
| **`JwtSecurityTokenHandler`** | Legacy; `JsonWebTokenHandler` in `Microsoft.IdentityModel.JsonWebTokens` is recommended |
| **PBKDF2-HMAC-SHA1** | Legacy; prefer HMAC-SHA256/512 for new systems |
| **bcrypt passwords > 72 bytes** | Truncated; use EnhancedHashPassword or Argon2id |
| **SMS MFA** | Not recommended; known attack vectors |
| **`TypeNameHandling.All` in Newtonsoft.Json** | Insecure deserialisation risk; avoid with untrusted input |
| **`PasswordHasherOptions.IterationCount` default** | 100,000; may lag OWASP guidance; configure explicitly |
| **Passkeys in ASP.NET Core Identity** | .NET 10+; requires platform authenticator or FIDO2 key |
| **Refresh-token rotation** | Required for OAuth best practice; reuse detection must revoke the family |
| **`RequireHttpsMetadata = false`** | Only acceptable in development; production must be `true` |

---

## Summary Table: Authentication Security at a Glance

| Concern | Primitive / Practice | Key Configuration |
|---|---|---|
| **Password storage** | Argon2id, bcrypt (≥ 12), PBKDF2 (OWASP iterations) | Unique salt, breached-password screening, rehash on login |
| **ASP.NET Core hashing** | `PasswordHasher<TUser>` | `PasswordHasherOptions.IterationCount` |
| **MFA** | TOTP (default), FIDO2/passkeys (.NET 10+) | `TwoFactorEnabled`, `amr` claim, step-up policies |
| **JWT access tokens** | `JsonWebTokenHandler` / `AddJwtBearer` | Validate issuer, audience, lifetime, signing key |
| **Refresh tokens** | Opaque, hashed at rest, rotating | Reuse detection → family revocation |
| **Cookies** | `HttpOnly`, `Secure`, `SameSite` | `ConfigureApplicationCookie`, security stamp |
| **Lockout** | `IdentityOptions.Lockout` | `lockoutOnFailure: true`, rate limiting |
| **Redirects** | `LocalRedirect`, `Url.IsLocalUrl` | Allowlist external destinations |
| **Input** | Antiforgery, output encoding, safe JSON | `AutoValidateAntiforgeryToken`, `System.Text.Json` |

---

## References (Consolidated)

- DotNet Security Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Password Hashing: bcrypt vs. Argon2 vs. PBKDF2 — Encryption Consulting - https://www.encryptionconsulting.com/password-storage-done-right-hashing-salting-and-bcrypt-vs-argon2-vs-pbkdf2/
- bcrypt in C# — MojoAuth Security Guides - https://mojoauth.com/security-guides/bcrypt-in-c
- BCrypt.Net-Next — GitHub - https://github.com/BcryptNet/bcrypt.net
- PasswordHasher<TUser> Class — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.passwordhasher-1
- PasswordHasherOptions Class — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.passwordhasheroptions
- Multi-factor authentication in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/57fea1b959863e300f7d7926113bb9385c39909e/aspnetcore/security/authentication/mfa.md
- Enable Web Authentication API (WebAuthn) passkeys — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authentication/passkeys/
- How to handle JWT in .NET — WorkOS - https://webflow.workos.com/blog/how-to-handle-jwt-in-net
- Refresh tokens in a .NET 10 Minimal API — DEV Community - https://dev.to/…/refresh-tokens-in-a-net-10-minimal-api-rotation-reuse-detection-and-revocation
- JWT access + refresh tokens in ASP.NET Core (.NET 10) — DEV Community - https://dev.to/…/jwt-access-refresh-tokens-in-aspnet-core-net-10-rotation-revocation-and-the-bugs-to-avoid
- Raise Identity password-hash work factor — GitHub PR - https://github.com/Abblix/Oidc.Server.GettingStarted/pull/22
- Konfigurowanie JWT uwierzytelniania elementu nośnego w ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/pl-pl/aspnet/core/security/authentication/configure-jwt-bearer-authentication
- Security Considerations — Duende Software - https://docs.duendesoftware.com/security-considerations/
- ASP.NET Core 中的多重身份验证 — Microsoft Learn - https://learn.microsoft.com/zh-cn/aspnet/core/security/authentication/mfa