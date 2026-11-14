# Cryptography in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Cryptography is the practice of protecting information through mathematical techniques that provide confidentiality (keeping data secret), integrity (detecting tampering), authenticity (verifying identity), and non-repudiation (proving an action occurred).

**Technical Definition**
Cryptography in .NET is implemented through the `System.Security.Cryptography` namespace, which provides vetted implementations of symmetric encryption (AES), asymmetric encryption (RSA, ECDsa, ECDH), cryptographic hashing (SHA-2 family), message authentication (HMAC, AES-GCM), password-based key derivation (PBKDF2), digital signatures (RSA, ECDSA), and cryptographically secure random number generation. The namespace is backed by the operating system's cryptographic primitives — OpenSSL on Linux/macOS and CNG on Windows — and is FIPS 140-2/140-3 validated on certified platforms. .NET 10 introduces first-class support for NIST post-quantum algorithms (ML-KEM, ML-DSA, SLH-DSA), addressing the future threat posed by quantum computing to RSA and ECC. Cryptography is unforgiving: code that compiles and passes tests can still be catastrophically insecure due to a reused nonce, a fast hash used for passwords, or a predictable random number source.

**Beginner-Friendly Explanation**
Cryptography is “the mathematics of secrecy and trust.” It lets you lock data so only authorised parties can read it (encryption), detect whether data has been tampered with (hashing and authentication), prove who created or approved data (digital signatures), and generate values that attackers cannot guess (secure randomness). .NET gives you strong, well-tested building blocks in `System.Security.Cryptography`, but the rules are strict: never invent your own algorithm, never reuse a nonce, never hash passwords with a fast hash, and never use `System.Random` for anything secret. One small mistake can undo the entire scheme.

---

### Key Characteristics

- **Never roll your own crypto** — Use vetted, high-level primitives from `System.Security.Cryptography`; do not implement algorithms yourself.
- **Confidentiality alone is insufficient** — Encryption without authentication is vulnerable to tampering; use authenticated encryption (AES-GCM).
- **Passwords require slow, salted KDFs** — SHA-256 and MD5 are fast hashes and must never be used for password storage; use PBKDF2, bcrypt, or Argon2id.
- **Cryptographic randomness is mandatory** — `System.Random` is predictable; use `RandomNumberGenerator` for keys, salts, nonces, and tokens.
- **Key management is the hardest part** — Generating keys is easy; storing, rotating, and protecting them is where breaches happen.
- **Post-quantum transition is beginning** — .NET 10 adds ML-KEM, ML-DSA, and SLH-DSA for future-proofing against quantum attacks.
- **FIPS compliance may constrain algorithm choice** — Some environments mandate FIPS-validated algorithms; PBKDF2 is the typical choice for passwords in those cases.

---

### Prerequisites

- Working knowledge of C# and the .NET CLI.
- Basic understanding of byte arrays, `Span<T>`, and `using` statements.
- Familiarity with `System.Security.Cryptography` namespace overview (hashing, symmetric/asymmetric encryption, random generation).
- Awareness of OWASP cryptographic storage guidance and NIST standards.
- Understanding of when to use the Data Protection API versus raw cryptographic primitives.

---

### Related Programming Areas

- **Authentication security** — Password hashing, token signing, and MFA.
- **Secure coding** — Secret management, input validation, and error handling.
- **Common vulnerabilities** — Cryptographic failures (OWASP A04:2025).
- **Key management** — Azure Key Vault, Data Protection API, and key rotation.
- **Digital signatures** — Code signing, document signing, and JWT signatures.

---

### Core Concepts / Features

1. Cryptography (Overarching Discipline)
2. `System.Security.Cryptography`
3. Hashing
4. Encryption
5. Digital Signatures
6. Key Management
7. Cryptography Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with “To”), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Cryptography (Overarching Discipline)

### Definitions

**Core Definition**
Cryptography is the science of securing information using mathematical algorithms that provide confidentiality, integrity, authentication, and non-repudiation.

**Technical Definition**
Cryptography in .NET encompasses four families of algorithms: **symmetric encryption** (AES) for fast bulk confidentiality; **asymmetric cryptography** (RSA, ECDsa, ECDH) for key exchange, signatures, and small-scale encryption; **cryptographic hashing** (SHA-2, SHA-3) for integrity and fingerprinting; and **key derivation functions** (PBKDF2) for turning passwords into keys. Each family has precise usage rules: AES-GCM requires a unique nonce per message; RSA encryption requires OAEP padding; password hashing requires a slow KDF with a unique salt; and digital signatures require the private key to remain secret. The .NET runtime delegates to the OS crypto library, ensuring hardware acceleration and FIPS validation where required. .NET 10 adds post-quantum algorithms (ML-KEM, ML-DSA, SLH-DSA) for defence against future quantum attacks.

**Beginner-Friendly Explanation**
Cryptography is “using math to keep secrets and prove trust.” It includes encryption (locking data so only the right key opens it), hashing (creating a fingerprint to detect changes), digital signatures (proving who sent something), and secure random numbers (generating values attackers cannot guess). .NET has all of these built in, but each has strict rules — break one rule and the security collapses.

---

### Purposes

- To protect data confidentiality at rest and in transit.
- To detect unauthorised modification through integrity checks.
- To verify the authenticity of data and identities.
- To provide non-repudiation through digital signatures.
- To generate unpredictable values for keys, salts, and tokens.
- To satisfy regulatory and compliance requirements (PCI-DSS, HIPAA, FIPS).
- To future-proof against quantum computing threats.

---

### Syntax Rules and Structure

#### General Cryptographic Decision Flow

```
1. What is the goal?
   - Confidentiality → Symmetric encryption (AES-GCM)
   - Integrity → Hashing (SHA-256) or HMAC
   - Authenticity → Digital signatures (RSA/ECDSA)
   - Password storage → Slow KDF (PBKDF2/bcrypt/Argon2)
   - Random values → RandomNumberGenerator

2. What are the constraints?
   - FIPS compliance? → Use FIPS-validated algorithms
   - Key storage? → Managed vault (Key Vault, HSM)
   - Legacy compatibility? → Avoid unless migration is planned

3. Apply the correct primitive with the correct parameters.
```

**Component breakdown**

- **Confidentiality** — AES-GCM with a unique nonce and a random key.
- **Integrity** — SHA-256 or HMAC-SHA256.
- **Authenticity** — RSA or ECDSA signatures.
- **Passwords** — PBKDF2 with ≥ 600,000 iterations (SHA-256) or a vetted library for Argon2/bcrypt.
- **Randomness** — `RandomNumberGenerator.GetBytes`.

#### Syntax Rules

- Use high-level, misuse-resistant primitives; never implement cryptographic algorithms manually.
- Never use `System.Random` for secrets; use `RandomNumberGenerator`.
- Never reuse a nonce with the same key in AES-GCM.
- Never use MD5 or SHA-1 for security purposes.
- Never use SHA-256 alone for password hashing; use a slow KDF.
- Store keys separately from the data they protect.

#### Constraints and Limitations

- Cryptography is easy to get subtly wrong; a single nonce reuse or weak key can break the scheme.
- Performance costs: PBKDF2 with high iterations takes ~0.4 s per hash; AES-GCM is fast but requires careful nonce management.
- FIPS compliance may restrict algorithm choice (e.g., PBKDF2 is FIPS-approved; Argon2 may not be).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Cryptographic Randomness

```csharp
using System.Security.Cryptography;

// ❌ Anti-pattern: predictable
var weakToken = new Random().Next().ToString();

// ✅ Pattern: cryptographically secure
byte[] bytes = RandomNumberGenerator.GetBytes(32);
string token = Convert.ToBase64String(bytes);
```

**Expected Output (conceptual)**

```
Weak token: predictable, guessable.
Secure token: 256 bits of entropy from the OS CSPRNG.
```

**Why this output occurs**

- `System.Random` is seeded from the clock and is deterministic; `RandomNumberGenerator` draws from the OS cryptographic random source.

---

#### Example 2: Choosing the Right Primitive

```csharp
// Confidentiality: AES-GCM
using var aes = new AesGcm(key, tagSizeInBytes: 16);

// Integrity: SHA-256
byte[] hash = SHA256.HashData(data);

// Password: PBKDF2
byte[] derived = Rfc2898DeriveBytes.Pbkdf2(password, salt, 600_000, HashAlgorithmName.SHA256, 32);

// Signature: ECDSA
using var ecdsa = ECDsa.Create(ECCurve.NamedCurves.nistP256);
byte[] signature = ecdsa.SignData(data, HashAlgorithmName.SHA256);
```

**Expected Output (conceptual)**

```
Each primitive is used for its intended purpose with appropriate parameters.
```

**Why this output occurs**

- Each family solves a different problem; using the wrong primitive (e.g., SHA-256 for passwords) creates a vulnerability.

---

### Real-World Cases

- **Data at rest** — AES-GCM encryption of sensitive database columns or files.
- **Data in transit** — TLS (which uses AES-GCM and ECDHE internally).
- **Password storage** — PBKDF2 or Argon2id with unique salts.
- **API tokens** — Random 256-bit tokens from `RandomNumberGenerator`.
- **Document signing** — RSA or ECDSA signatures over document hashes.

---

### References

- C# Cryptography Best Practices (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-cryptography-best-practices
- System.Security.Cryptography Namespace — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography
- Post-Quantum Cryptography in .NET 10: A Practical Guide — Duende Software - https://duendesoftware.com/blog/20260514-post-quantum-cryptography-in-dotnet-10

---

## 2. `System.Security.Cryptography`

### Definitions

**Core Definition**
`System.Security.Cryptography` is the .NET namespace that provides cryptographic services, including secure encoding and decoding, hashing, random number generation, message authentication, symmetric and asymmetric encryption, and digital signatures.

**Technical Definition**
The namespace contains abstract base classes (`HashAlgorithm`, `SymmetricAlgorithm`, `AsymmetricAlgorithm`) and concrete implementations (`SHA256`, `AesGcm`, `RSA`, `ECDsa`, `RandomNumberGenerator`, `Rfc2898DeriveBytes`). Modern APIs favor static convenience methods (`SHA256.HashData`, `RandomNumberGenerator.GetBytes`) and `Span<T>`-based overloads to minimize allocations. The namespace delegates to the OS cryptographic library: OpenSSL on Linux and macOS, CNG on Windows. .NET 10 introduces `MLKem`, `MLDsa`, and `SlhDsa` classes for post-quantum cryptography, available when the underlying OS supports the required algorithms (OpenSSL 3.5+ or Windows CNG).

**Beginner-Friendly Explanation**
`System.Security.Cryptography` is “the toolbox of cryptographic tools that ships with .NET.” It contains everything you need: tools to hash data, encrypt data, sign data, generate random numbers, and derive keys from passwords. You do not need third-party libraries for most tasks — the built-in toolbox is well-tested and backed by the operating system.

---

### Purposes

- To provide vetted, OS-backed cryptographic primitives.
- To avoid the risks of hand-rolled or third-party cryptography.
- To ensure FIPS compliance on certified platforms.
- To offer modern, allocation-efficient APIs (`Span<T>` overloads, static methods).
- To support post-quantum algorithms as they become standardized.
- To centralize cryptographic operations under a single, auditable namespace.

---

### Syntax Rules and Structure

#### General Syntax (Key Classes)

| Class | Purpose | Modern API |
|---|---|---|
| `SHA256` | Hashing | `SHA256.HashData(data)` |
| `HMACSHA256` | Message authentication | `HMACSHA256.HashData(key, data)` |
| `AesGcm` | Authenticated symmetric encryption | `new AesGcm(key, tagSize)` |
| `RSA` | Asymmetric encryption/signatures | `RSA.Create(2048)` |
| `ECDsa` | Elliptic curve signatures | `ECDsa.Create(ECCurve.NamedCurves.nistP256)` |
| `RandomNumberGenerator` | Secure random bytes | `RandomNumberGenerator.GetBytes(32)` |
| `Rfc2898DeriveBytes` | Password-based KDF | `Rfc2898DeriveBytes.Pbkdf2(...)` |
| `MLKem` (.NET 10) | Post-quantum key encapsulation | `MLKem.GenerateKey(...)` |
| `MLDsa` (.NET 10) | Post-quantum signatures | `MLDsa.GenerateKey(...)` |

#### Syntax Rules

- Use static methods (`HashData`, `GetBytes`) where available to reduce allocations.
- Use `using` statements for disposable algorithm instances (`AesGcm`, `RSA`, `ECDsa`).
- Prefer `Span<T>` overloads for high-performance scenarios.
- Do not use obsolete constructors (`new Rfc2898DeriveBytes(...)` defaults to SHA-1 and is obsolete in .NET 10).
- For FIPS compliance, use the `*Cng` classes on Windows or ensure the OS crypto library is FIPS-validated.

#### Constraints and Limitations

- Post-quantum algorithms require OS support (OpenSSL 3.5+ or Windows CNG with PQC support).
- Some algorithms (e.g., `AesGcm`) are not available on all platforms or require specific runtime versions.
- `BinaryFormatter`-related classes are obsolete and insecure; do not use them.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Static Hashing API

```csharp
using System.Security.Cryptography;

byte[] data = Encoding.UTF8.GetBytes("hello");
byte[] hash = SHA256.HashData(data);
Console.WriteLine(Convert.ToHexString(hash));
```

**Expected Output**

```
2CF24DBA5FB0A30E26E83B2AC5B9E29E1B161E5C1FA7425E73043362938B9824
```

**Why this output occurs**

- `SHA256.HashData` is the modern, allocation-friendly static API for one-shot hashing.

---

#### Example 2: Post-Quantum Signatures (.NET 10)

```csharp
using System.Security.Cryptography;

using var mldsa = MLDsa.GenerateKey(MLDsaAlgorithm.MLDsa65);
byte[] signature = mldsa.SignData(data);
bool valid = mldsa.VerifyData(data, signature);
```

**Expected Output (conceptual)**

```
A post-quantum signature is generated and verified.
Available only when the OS supports the algorithm.
```

**Why this output occurs**

- .NET 10 exposes `MLDsa` for FIPS 204 post-quantum signatures.

---

### Real-World Cases

- **All .NET cryptographic operations** — This namespace is the foundation.
- **TLS and HTTPS** — Uses the same OS primitives under the hood.
- **JWT signing and validation** — RSA and ECDSA from this namespace.
- **File encryption** — AES-GCM or AES-CBC+HMAC.

---

### References

- System.Security.Cryptography Namespace — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography
- Post-Quantum Cryptography in .NET 10 — Microsoft .NET Blog - https://devblogs.microsoft.com/dotnet/post-quantum-cryptography-in-dotnet/
- What’s New in .NET 10 Libraries and Runtime — GitHub - https://github.com/dotnet/core/blob/main/release-notes/10.0/whats-new.md

---

## 3. Hashing

### Definitions

**Core Definition**
Hashing is the one-way transformation of data of arbitrary length into a fixed-length value (the hash) that serves as a unique fingerprint, used for integrity verification, password storage (via KDFs), and digital signatures.

**Technical Definition**
A cryptographic hash function takes an input and produces a fixed-size output (e.g., 256 bits for SHA-256) such that any change to the input produces an unpredictable, completely different output, and it is computationally infeasible to find two inputs with the same hash (collision resistance) or to reverse the hash. .NET provides SHA-256, SHA-384, and SHA-512 (recommended), and SHA-3 for newer applications. **Password hashing is a special case**: fast hashes like SHA-256 are unsuitable because attackers can try billions of guesses per second. Passwords require a slow, salted key-derivation function (KDF): PBKDF2 (built-in, FIPS-approved), bcrypt, or Argon2id (memory-hard, preferred by OWASP).

**Beginner-Friendly Explanation**
Hashing is “creating a fingerprint for data.” If you hash a file and later hash it again, the fingerprints match only if the file has not changed. For passwords, you never store the password itself — you store its fingerprint. But because fast hashes can be guessed quickly, you use a deliberately slow fingerprinting process (PBKDF2, bcrypt, Argon2) with a unique random ingredient (salt) for each password.

---

### Purposes

- To verify data integrity (detect tampering or corruption).
- To store passwords securely using a slow, salted KDF.
- To create message authentication codes (HMAC) for integrity and authenticity.
- To generate deterministic identifiers from data (e.g., content hashes).
- To support digital signatures (signing the hash of a document).
- To satisfy OWASP password storage requirements.

---

### Syntax Rules and Structure

#### General Syntax (General Hashing)

```csharp
byte[] hash = SHA256.HashData(data);
```

#### General Syntax (Password Hashing with PBKDF2)

```csharp
byte[] salt = RandomNumberGenerator.GetBytes(16);
byte[] hash = Rfc2898DeriveBytes.Pbkdf2(
    password: Encoding.UTF8.GetBytes(password),
    salt: salt,
    iterations: 600_000,
    hashAlgorithm: HashAlgorithmName.SHA256,
    outputLength: 32);
```

#### General Syntax (Verification with Constant-Time Comparison)

```csharp
byte[] expected = Convert.FromBase64String(storedHash);
byte[] actual = Rfc2898DeriveBytes.Pbkdf2(
    password, salt, iterations, HashAlgorithmName.SHA256, expected.Length);
bool valid = CryptographicOperations.FixedTimeEquals(actual, expected);
```

#### Algorithm Guidance

| Use Case | Algorithm | Parameters |
|---|---|---|
| **General integrity** | SHA-256 / SHA-512 | — |
| **Message authentication** | HMAC-SHA256 | Random key ≥ 256 bits |
| **Password storage (FIPS)** | PBKDF2-HMAC-SHA256 | ≥ 600,000 iterations, 16-byte salt |
| **Password storage (non-FIPS, preferred)** | Argon2id | OWASP parameters (e.g., 19 MiB, 2 iterations) |
| **Password storage (legacy)** | bcrypt | Cost ≥ 12 |
| **Never** | MD5, SHA-1, plain SHA-256 for passwords | — |

#### Syntax Rules

- Use SHA-256 or stronger for general-purpose hashing; never MD5 or SHA-1.
- Never use a fast hash (SHA-256) for password storage; use PBKDF2, bcrypt, or Argon2id.
- Use a unique, random salt per password (≥ 16 bytes).
- Store the salt and iteration count alongside the hash so parameters can be upgraded.
- Verify password hashes with `CryptographicOperations.FixedTimeEquals`, not `==` or `SequenceEqual`.
- Use `PasswordHasher<TUser>` from ASP.NET Core Identity instead of rolling your own PBKDF2.

#### Constraints and Limitations

- PBKDF2 is not memory-hard; Argon2id is preferred for new systems when FIPS compliance is not required.
- bcrypt truncates passwords at 72 bytes.
- High iteration counts add login latency (~0.4 s for 600,000 iterations); tune to deployment hardware.
- Hashing is deterministic; identical inputs produce identical hashes unless a salt is used.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: SHA-256 Integrity Check

```csharp
byte[] original = Encoding.UTF8.GetBytes("important data");
byte[] hash = SHA256.HashData(original);
Console.WriteLine(Convert.ToHexString(hash));
```

**Expected Output**

```
B94D27B9934D3E08A52E52D7DA7DABFAC484EFE37A5380EE9088F7ACE2EFCDE9
```

**Why this output occurs**

- SHA-256 produces a deterministic 256-bit fingerprint; any change to the input produces a completely different hash.

---

#### Example 2: PBKDF2 Password Hashing and Verification

```csharp
const int Iterations = 600_000;
byte[] salt = RandomNumberGenerator.GetBytes(16);
byte[] hash = Rfc2898DeriveBytes.Pbkdf2("password", salt, Iterations, HashAlgorithmName.SHA256, 32);

string stored = $"{Iterations}.{Convert.ToBase64String(salt)}.{Convert.ToBase64String(hash)}";

// Verify
var parts = stored.Split('.');
var verifyHash = Rfc2898DeriveBytes.Pbkdf2("password",
    Convert.FromBase64String(parts[1]), int.Parse(parts[0]), HashAlgorithmName.SHA256, 32);
bool valid = CryptographicOperations.FixedTimeEquals(verifyHash, Convert.FromBase64String(parts[2]));
Console.WriteLine(valid); // True
```

**Expected Output**

```
True
```

**Why this output occurs**

- The stored string includes the iteration count and salt, so verification recomputes the hash with the same parameters and compares in constant time.

---

### Real-World Cases

- **Password storage** — PBKDF2 or Argon2id with unique salts.
- **File integrity** — SHA-256 checksums for downloads.
- **API request signing** — HMAC-SHA256 over request bodies.
- **Content addressing** — Git uses SHA-1/SHA-256 for object IDs.
- **Blockchain** — SHA-256 for proof-of-work and block integrity.

---

### References

- C# Cryptography Best Practices (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-cryptography-best-practices
- PBKDF2 in C#: Rfc2898DeriveBytes.Pbkdf2 Example — MojoAuth - https://mojoauth.com/security-guides/pbkdf2-in-c
- OWASP Password Storage Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html

---

## 4. Encryption

### Definitions

**Core Definition**
Encryption is the reversible transformation of plaintext into ciphertext using a cryptographic algorithm and a key, such that only someone possessing the correct key can recover the original plaintext.

**Technical Definition**
Encryption in .NET is either **symmetric** (one shared key for both encryption and decryption; fast, suitable for bulk data) or **asymmetric** (a public key for encryption, a private key for decryption; slow, used for key exchange and small payloads). The modern symmetric standard is **AES-GCM** (Galois/Counter Mode), which provides both confidentiality and integrity (authenticated encryption). AES-CBC without a MAC is vulnerable to padding-oracle attacks and must not be used for new designs. RSA encryption requires OAEP padding (never PKCS#1 v1.5 for new designs). .NET 10 adds ML-KEM for post-quantum key encapsulation.

**Beginner-Friendly Explanation**
Encryption is “locking data so only the right key can open it.” Symmetric encryption uses the same key to lock and unlock — it is fast and used for most data. Asymmetric encryption uses two keys: a public key to lock and a private key to unlock — it is slower and used for key exchange and signatures. For symmetric encryption, always use AES-GCM, which also detects tampering. Never use AES-CBC alone, because an attacker can modify the ciphertext without detection.

---

### Purposes

- To protect data confidentiality at rest (databases, files, backups).
- To protect data in transit (TLS uses AES-GCM).
- To encrypt small payloads with asymmetric keys (key wrapping, hybrid encryption).
- To provide authenticated encryption that detects tampering.
- To support envelope encryption (data key encrypted by a master key).
- To enable post-quantum key encapsulation (ML-KEM).

---

### Syntax Rules and Structure

#### General Syntax (AES-GCM Encryption)

```csharp
byte[] key = RandomNumberGenerator.GetBytes(32);      // 256-bit key
byte[] nonce = RandomNumberGenerator.GetBytes(12);     // 96-bit nonce, unique per message
byte[] ciphertext = new byte[plaintext.Length];
byte[] tag = new byte[16];

using var aes = new AesGcm(key, tag.Length);
aes.Encrypt(nonce, plaintext, ciphertext, tag);

// Store/transmit: nonce + ciphertext + tag
```

#### General Syntax (AES-GCM Decryption)

```csharp
using var aes = new AesGcm(key, tag.Length);
byte[] decrypted = new byte[ciphertext.Length];
aes.Decrypt(nonce, ciphertext, tag, decrypted);
// Throws AuthenticationTagMismatchException if tampered with.
```

#### General Syntax (RSA Encryption with OAEP)

```csharp
using var rsa = RSA.Create(2048);
byte[] encrypted = rsa.Encrypt(data, RSAEncryptionPadding.OaepSHA256);
byte[] decrypted = rsa.Decrypt(encrypted, RSAEncryptionPadding.OaepSHA256);
```

#### Algorithm Selection

| Use Case | Algorithm | Notes |
|---|---|---|
| **Bulk data at rest** | AES-GCM (256-bit key) | Authenticated; unique nonce per message |
| **Legacy compatibility** | AES-CBC + HMAC | Must authenticate; avoid for new designs |
| **Key exchange** | RSA-OAEP or ECDH | OAEP padding only for RSA encryption |
| **Post-quantum key exchange** | ML-KEM (.NET 10) | FIPS 203; OS support required |
| **Never** | ECB mode, AES-CBC without MAC | Insecure |

#### Syntax Rules

- Use AES-GCM for authenticated symmetric encryption.
- Generate a unique 12-byte nonce per message with `RandomNumberGenerator.GetBytes(12)`; never reuse a nonce with the same key.
- Use a 256-bit key generated from `RandomNumberGenerator`.
- Store the nonce, ciphertext, and tag together; the nonce is not secret.
- Use RSA-OAEP (never PKCS#1 v1.5) for RSA encryption.
- Never hard-code keys or nonces; never derive keys from passwords without a KDF.

#### Constraints and Limitations

- AES-GCM nonce reuse with the same key completely breaks confidentiality and integrity; limit each key to ~2^32 messages with random nonces.
- RSA can encrypt only small payloads (bounded by key size minus padding).
- AES-CBC without authentication is vulnerable to padding-oracle attacks.
- Post-quantum algorithms require OS support and are not yet universally available.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: AES-GCM Encrypt and Decrypt

```csharp
byte[] key = RandomNumberGenerator.GetBytes(32);
byte[] nonce = RandomNumberGenerator.GetBytes(12);
byte[] plaintext = Encoding.UTF8.GetBytes("Sensitive Information");
byte[] ciphertext = new byte[plaintext.Length];
byte[] tag = new byte[16];

using (var aes = new AesGcm(key, 16))
{
    aes.Encrypt(nonce, plaintext, ciphertext, tag);
}

byte[] decrypted = new byte[ciphertext.Length];
using (var aes = new AesGcm(key, 16))
{
    aes.Decrypt(nonce, ciphertext, tag, decrypted);
}
Console.WriteLine(Encoding.UTF8.GetString(decrypted));
```

**Expected Output**

```
Sensitive Information
```

**Why this output occurs**

- AES-GCM encrypts and authenticates; decryption succeeds only if the key, nonce, ciphertext, and tag are unchanged.

---

#### Example 2: Tampering Detection

```csharp
ciphertext[0] ^= 0xFF; // flip a bit
try
{
    using var aes = new AesGcm(key, 16);
    aes.Decrypt(nonce, ciphertext, tag, decrypted);
}
catch (AuthenticationTagMismatchException)
{
    Console.WriteLine("Tampering detected");
}
```

**Expected Output**

```
Tampering detected
```

**Why this output occurs**

- AES-GCM verifies the authentication tag; any modification causes decryption to throw.

---

### Real-World Cases

- **Database column encryption** — AES-GCM for sensitive fields.
- **File encryption** — AES-GCM with per-file nonces.
- **TLS** — AES-GCM is the dominant TLS cipher.
- **Envelope encryption** — Data encrypted with a DEK; DEK encrypted with a KEK.
- **Post-quantum key exchange** — ML-KEM for future-proofed TLS.

---

### References

- C# Cryptography Best Practices (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-cryptography-best-practices
- AES-128 Encryption and Decryption in C# — MojoAuth - https://mojoauth.com/security-guides/aes-128-encryption--c
- Doing AES Encryption Correct in Your .NET Application — PropertyGuru - https://propertyguru.tech/doing-aes-encryption-correct-in-your-net-application
- Post-Quantum Cryptography in .NET 10 — Duende Software - https://duendesoftware.com/blog/20260514-post-quantum-cryptography-in-dotnet-10

---

## 5. Digital Signatures

### Definitions

**Core Definition**
A digital signature is a cryptographic value that proves the authenticity and integrity of data and provides non-repudiation — the signer cannot later deny having signed it.

**Technical Definition**
A digital signature is produced by hashing the data and then encrypting the hash (or applying a signature algorithm) with the signer’s **private key**. Anyone with the corresponding **public key** can verify the signature by recomputing the hash and checking the signature. .NET provides **RSA** signatures (`RSA.SignData` / `VerifyData`) and **ECDSA** signatures (`ECDsa.SignData` / `VerifyData`). RSA is widely supported and used in enterprise environments; ECDSA offers equivalent security with smaller keys and faster computation. .NET 10 adds **ML-DSA** (FIPS 204) for post-quantum signatures. Signatures are used in JWT signing (RS256, ES256), code signing, document signing, and API request authentication.

**Beginner-Friendly Explanation**
A digital signature is “a cryptographic seal that proves who created or approved something.” It is like a handwritten signature, but far stronger: it cannot be forged without the private key, and it proves that the data has not been changed. In .NET, you sign with a private key and verify with a public key. JWT tokens use digital signatures so the server can verify that the token was issued by a trusted party.

---

### Purposes

- To prove the authenticity of data (who created or approved it).
- To guarantee data integrity (the data has not been altered).
- To provide non-repudiation (the signer cannot deny signing).
- To sign JWTs, documents, code, and API requests.
- To support certificate-based authentication and PKI.
- To enable post-quantum signatures with ML-DSA (.NET 10).

---

### Syntax Rules and Structure

#### General Syntax (RSA Signature)

```csharp
using var rsa = RSA.Create(2048);
byte[] signature = rsa.SignData(data, HashAlgorithmName.SHA256, RSASignaturePadding.Pss);
bool valid = rsa.VerifyData(data, signature, HashAlgorithmName.SHA256, RSASignaturePadding.Pss);
```

#### General Syntax (ECDSA Signature)

```csharp
using var ecdsa = ECDsa.Create(ECCurve.NamedCurves.nistP256);
byte[] signature = ecdsa.SignData(data, HashAlgorithmName.SHA256);
bool valid = ecdsa.VerifyData(data, signature, HashAlgorithmName.SHA256);
```

#### General Syntax (ML-DSA Post-Quantum — .NET 10)

```csharp
using var mldsa = MLDsa.GenerateKey(MLDsaAlgorithm.MLDsa65);
byte[] signature = mldsa.SignData(data);
bool valid = mldsa.VerifyData(data, signature);
```

#### Algorithm Selection

| Algorithm | Key Size | Signature Size | Use Case |
|---|---|---|---|
| **RSA-PSS (2048+)** | 2048+ bits | 256 bytes | Enterprise, JWT (RS256), wide compatibility |
| **ECDSA (P-256/P-384)** | 256/384 bits | ~64/96 bytes | Modern APIs, JWT (ES256), smaller keys |
| **ML-DSA (.NET 10)** | Post-quantum | Larger | Future-proofing against quantum attacks |

#### Syntax Rules

- Use RSA with PSS padding (`RSASignaturePadding.Pss`) for new designs; PKCS#1 v1.5 is legacy.
- Use ECDSA with NIST P-256 or P-384 curves for modern applications.
- Hash the data before signing; never sign raw data without a hash.
- Protect the private key rigorously; compromise of the private key allows forgery.
- Verify signatures with the signer’s public key, obtained through a trusted channel (certificate, JWKS).
- Use ML-DSA only when OS support is available and interoperability is confirmed.

#### Constraints and Limitations

- RSA signatures are larger and slower than ECDSA.
- ECDSA requires careful nonce management; reusing a nonce in ECDSA (the “nonce reuse” problem) leaks the private key.
- ML-DSA is new and may not be supported by all platforms or libraries.
- Signature verification does not prove the identity of the signer unless the public key is bound to an identity through a certificate or trusted directory.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: RSA Signature and Verification

```csharp
using var rsa = RSA.Create(2048);
byte[] data = Encoding.UTF8.GetBytes("contract terms");
byte[] signature = rsa.SignData(data, HashAlgorithmName.SHA256, RSASignaturePadding.Pss);
bool valid = rsa.VerifyData(data, signature, HashAlgorithmName.SHA256, RSASignaturePadding.Pss);
Console.WriteLine(valid); // True
```

**Expected Output**

```
True
```

**Why this output occurs**

- The private key signs the hash; the public key verifies it; any change to the data invalidates the signature.

---

#### Example 2: ECDSA Signature

```csharp
using var ecdsa = ECDsa.Create(ECCurve.NamedCurves.nistP256);
byte[] signature = ecdsa.SignData(data, HashAlgorithmName.SHA256);
bool valid = ecdsa.VerifyData(data, signature, HashAlgorithmName.SHA256);
```

**Expected Output (conceptual)**

```
ECDSA produces a compact signature that verifies with the public key.
```

**Why this output occurs**

- ECDSA uses elliptic curve mathematics to provide equivalent security with smaller keys.

---

### Real-World Cases

- **JWT signing** — RS256 (RSA) or ES256 (ECDSA) for API tokens.
- **Code signing** — Authenticode signatures on executables and packages.
- **Document signing** — PDF and XML signatures for legal documents.
- **API request authentication** — AWS Signature V4 and similar schemes.
- **Certificate chains** — X.509 certificates are signed by a CA’s private key.

---

### References

- Compile-N-Run — .NET Digital Signatures - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/13-net-security/7-net-digital-signatures.mdx
- C# 使用应用RSA和ECC进行数字签名和签名验证 — CSDN - https://blog.csdn.net/…
- Post-Quantum Cryptography in .NET 10: A Practical Guide — Duende Software - https://duendesoftware.com/blog/20260514-post-quantum-cryptography-in-dotnet-10

---

## 6. Key Management

### Definitions

**Core Definition**
Key management is the practice of securely generating, storing, distributing, rotating, and retiring cryptographic keys throughout their lifecycle.

**Technical Definition**
Key management in .NET spans the entire key lifecycle: **generation** (using `RandomNumberGenerator` or hardware security modules), **storage** (never in source code; use Azure Key Vault, AWS Secrets Manager, or the Data Protection API with persistent key storage), **distribution** (asymmetric public keys can be shared openly; symmetric keys must be transmitted through secure channels), **rotation** (periodic replacement to limit the impact of compromise), and **retirement** (secure deletion after the key is no longer needed). The **ASP.NET Core Data Protection API** provides a key-management-aware framework for encrypting sensitive data in web applications, with pluggable key storage (file system, Azure Blob Storage, Azure Key Vault) and key encryption (DPAPI, Azure Key Vault, certificate). Keys should be stored in HSMs for high-security scenarios, and key operations should be restricted to the minimum required permissions.

**Beginner-Friendly Explanation**
Key management is “keeping your cryptographic keys safe throughout their life.” Generating a key is easy; the hard part is storing it where attackers cannot reach it, rotating it before it is compromised, and making sure it is not accidentally leaked. In .NET, you should never hard-code keys in source code. Use Azure Key Vault or the Data Protection API to store keys securely, and rotate them regularly.

---

### Purposes

- To protect cryptographic keys from theft and misuse.
- To enable key rotation without re-encrypting all data.
- To separate key material from the data it protects.
- To satisfy compliance requirements (PCI-DSS, FIPS, SOC 2).
- To support envelope encryption and HSM-backed keys.
- To recover from key compromise through rotation and revocation.

---

### Syntax Rules and Structure

#### General Syntax (Data Protection API with Key Vault)

```csharp
builder.Services.AddDataProtection()
    .SetApplicationName("MyApp")
    .PersistKeysToAzureBlobStorage(blobUri, credential)
    .ProtectKeysWithAzureKeyVault(keyIdentifier, credential);
```

#### General Syntax (Key Vault Client)

```csharp
var client = new KeyClient(new Uri(vaultUri), new DefaultAzureCredential());
KeyVaultKey key = await client.CreateKeyAsync("my-key", KeyType.Rsa);
```

#### General Syntax (Data Protection Usage)

```csharp
public class TokenService
{
    private readonly IDataProtector _protector;
    public TokenService(IDataProtectionProvider provider)
        => _protector = provider.CreateProtector("Tokens");

    public string Protect(string plaintext) => _protector.Protect(plaintext);
    public string Unprotect(string ciphertext) => _protector.Unprotect(ciphertext);
}
```

#### Key Management Checklist

```
✅ Never hard-code keys in source code or configuration files.
✅ Use a managed vault (Azure Key Vault, AWS Secrets Manager) for key storage.
✅ Use HSM-backed keys for high-security scenarios.
✅ Implement automatic key rotation (e.g., Key Vault rotation policies).
✅ Restrict key operations to the minimum required (Get, Wrap, Unwrap).
✅ Use versionless key identifiers when auto-rotation is enabled.
✅ Back up keys securely; test restoration.
✅ Audit key access and monitor for anomalies.
```

#### Syntax Rules

- The Data Protection API is the recommended way to encrypt sensitive data in ASP.NET Core applications.
- Store Data Protection keys in a persistent, shared location (Azure Blob Storage, file share) for multi-instance deployments.
- Protect keys at rest with Azure Key Vault or DPAPI (Windows).
- Use `SetApplicationName` consistently across all instances.
- Rotate keys periodically; the Data Protection API supports key rotation with overlapping validity.
- For raw cryptographic keys, use `KeyClient` from `Azure.Security.KeyVault.Keys`.

#### Constraints and Limitations

- Key Vault access requires network connectivity and appropriate RBAC permissions.
- The Data Protection API is designed for ASP.NET Core applications; other app types may need a different key store.
- HSM-backed keys are more expensive and may have latency implications.
- Key rotation requires coordination with applications that cache keys.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Data Protection with Azure Key Vault

```csharp
builder.Services.AddDataProtection()
    .SetApplicationName("MyApp")
    .PersistKeysToAzureBlobStorage(new Uri(blobUri), new DefaultAzureCredential())
    .ProtectKeysWithAzureKeyVault(new Uri(keyIdentifier), new DefaultAzureCredential());
```

**Expected Output (conceptual)**

```
Data Protection keys are stored in Azure Blob Storage and encrypted with a Key Vault key.
All application instances share the same key ring.
```

**Why this output occurs**

- The key ring is persisted to shared storage; the master key is protected by Key Vault.

---

#### Example 2: Encrypting and Decrypting with IDataProtector

```csharp
var protector = provider.CreateProtector("Tokens");
string ciphertext = protector.Protect("sensitive data");
string plaintext = protector.Unprotect(ciphertext);
Console.WriteLine(plaintext); // "sensitive data"
```

**Expected Output**

```
sensitive data
```

**Why this output occurs**

- The Data Protection API manages key selection and rotation transparently; `Protect` and `Unprotect` use the current key.

---

### Real-World Cases

- **ASP.NET Core cookie encryption** — Data Protection API encrypts authentication cookies.
- **CSRF tokens** — Data Protection protects antiforgery tokens.
- **Multi-tenant SaaS** — Per-tenant key isolation with Key Vault.
- **Database column encryption** — Envelope encryption with a KEK in Key Vault.
- **Code signing** — Private keys stored in HSMs or Key Vault.

---

### References

- Configure ASP.NET Core Data Protection — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview
- Secure an ASP.NET Core Blazor Web App with Microsoft Entra ID — Azure Key Vault key encryption - https://learn.microsoft.com/en-us/aspnet/core/blazor/security/blazor-web-app-with-entra
- C# Cryptography Best Practices (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-cryptography-best-practices
- Azure Key Vault key management best practices — GitHub - https://github.com/aiskillstore/marketplace/blob/main/skills/microsoft/azure-compliance/references/sdk/azure-security-keyvault-keys-dotnet.md

---

## 7. Cryptography Best Practices

### Definitions

**Core Definition**
Cryptography best practices are empirically grounded guidelines for using cryptographic primitives correctly, safely, and maintainably.

**Technical Definition**
Best practices span **algorithm selection** (AES-GCM for encryption, SHA-256+ for hashing, PBKDF2/bcrypt/Argon2 for passwords, RSA-PSS/ECDSA for signatures), **parameter management** (unique salts, unique nonces, sufficient key sizes, high iteration counts), **randomness** (`RandomNumberGenerator` only), **constant-time comparison** (`CryptographicOperations.FixedTimeEquals`), **key lifecycle** (generation, storage, rotation, retirement), **legacy API avoidance** (no `BinaryFormatter`, no `System.Random`, no SHA-1/MD5, no AES-CBC without MAC), and **post-quantum readiness** (ML-KEM, ML-DSA, SLH-DSA in .NET 10).

**Beginner-Friendly Explanation**
Cryptography best practices are “the rules that keep your crypto from being broken.” They include: use AES-GCM, never reuse a nonce, hash passwords with PBKDF2 or Argon2, use `RandomNumberGenerator` for secrets, compare hashes in constant time, store keys in a vault, and never use MD5 or SHA-1. Following these prevents the subtle mistakes that make otherwise-correct code catastrophically insecure.

---

### Purposes

- To prevent catastrophic cryptographic failures.
- To satisfy OWASP, NIST, and regulatory requirements.
- To ensure interoperability across platforms and libraries.
- To future-proof against quantum computing threats.
- To make cryptographic code auditable and maintainable.

---

### Guidelines and Structure

#### Randomness

```
✅ Use RandomNumberGenerator.GetBytes for keys, salts, nonces, and tokens.
❌ Never use System.Random for anything secret.
```

#### Hashing

```
✅ Use SHA-256 or SHA-512 for integrity.
✅ Use HMAC-SHA256 for message authentication.
✅ Use PBKDF2 (≥ 600,000 iterations, SHA-256) or Argon2id for passwords.
❌ Never use MD5 or SHA-1 for security.
❌ Never use SHA-256 alone for passwords.
```

#### Encryption

```
✅ Use AES-GCM with a 256-bit key and a unique 12-byte nonce per message.
✅ Use RSA-OAEP for RSA encryption.
✅ Use ECDH for key agreement.
❌ Never use AES-CBC without a MAC.
❌ Never reuse a nonce with the same key.
❌ Never hard-code keys or nonces.
```

#### Digital Signatures

```
✅ Use RSA-PSS (2048+ bits) or ECDSA (P-256/P-384).
✅ Hash before signing.
✅ Protect private keys in HSMs or Key Vault.
❌ Never reuse an ECDSA nonce.
```

#### Key Management

```
✅ Store keys in Azure Key Vault or a managed HSM.
✅ Use the Data Protection API for ASP.NET Core data encryption.
✅ Rotate keys on a schedule.
✅ Restrict key permissions to the minimum required.
❌ Never commit keys to source control.
```

#### Constant-Time Operations

```
✅ Use CryptographicOperations.FixedTimeEquals for secret comparison.
❌ Never use == or SequenceEqual on secrets.
```

#### Post-Quantum Readiness

```
✅ Evaluate ML-KEM, ML-DSA, and SLH-DSA in .NET 10 for long-term confidentiality.
✅ Plan hybrid deployments (classical + post-quantum) during transition.
❌ Do not assume current RSA/ECC will remain secure indefinitely.
```

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Complete Secure Encryption Workflow

```csharp
byte[] key = RandomNumberGenerator.GetBytes(32);
byte[] nonce = RandomNumberGenerator.GetBytes(12);
byte[] plaintext = Encoding.UTF8.GetBytes("secret");
byte[] ciphertext = new byte[plaintext.Length];
byte[] tag = new byte[16];

using var aes = new AesGcm(key, 16);
aes.Encrypt(nonce, plaintext, ciphertext, tag);
// Store nonce + ciphertext + tag
```

**Expected Output (conceptual)**

```
Each message uses a fresh random nonce; the key is 256 bits from the CSPRNG.
```

**Why this output occurs**

- All parameters follow best practices: secure randomness, unique nonce, authenticated encryption.

---

#### Example 2: Constant-Time Password Verification

```csharp
byte[] expected = storedHash;
byte[] actual = ComputeHash(password, salt);
bool valid = CryptographicOperations.FixedTimeEquals(actual, expected);
```

**Expected Output (conceptual)**

```
Comparison takes the same time regardless of where the first mismatch occurs.
```

**Why this output occurs**

- `FixedTimeEquals` prevents timing side channels that could leak password information.

---

### Real-World Cases

- **All cryptographic code** — Following best practices prevents the most common failures.
- **Security audits** — OWASP ASVS and NIST guidelines reference these practices.
- **Post-quantum migration planning** — .NET 10 provides the primitives; best practices guide adoption.

---

### References

- C# Cryptography Best Practices (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-cryptography-best-practices
- SOTA-skills .NET security rules — GitHub - https://github.com/martinholovsky/SOTA-skills/blob/main/skills/sota-dotnet/rules/04-security.md
- OWASP Cryptographic Storage Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
- PBKDF2 in C# — MojoAuth - https://mojoauth.com/security-guides/pbkdf2-in-c
- Post-Quantum Cryptography in .NET 10 — Microsoft .NET Blog - https://devblogs.microsoft.com/dotnet/post-quantum-cryptography-in-dotnet/

---

## Summary Table: Cryptographic Primitives and Rules

| Goal | Primitive | Key Rule |
|---|---|---|
| **Confidentiality (bulk)** | AES-GCM (256-bit key) | Unique nonce per message |
| **Integrity** | SHA-256 / SHA-512 | Never MD5/SHA-1 |
| **Message authentication** | HMAC-SHA256 | Random key ≥ 256 bits |
| **Password storage** | PBKDF2 (≥ 600k iterations) or Argon2id | Unique salt; never SHA-256 |
| **Digital signatures** | RSA-PSS or ECDSA | Protect private key; hash before signing |
| **Secure randomness** | `RandomNumberGenerator` | Never `System.Random` |
| **Secret comparison** | `CryptographicOperations.FixedTimeEquals` | Never `==` or `SequenceEqual` |
| **Key storage** | Azure Key Vault / Data Protection API | Never hard-code |
| **Post-quantum** | ML-KEM, ML-DSA, SLH-DSA (.NET 10) | OS support required |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`BinaryFormatter`** | Removed in .NET 9; insecure; do not use |
| **`System.Random` for secrets** | Predictable; use `RandomNumberGenerator` |
| **MD5 / SHA-1** | Broken for security; do not use |
| **SHA-256 for passwords** | Too fast; use PBKDF2/bcrypt/Argon2 |
| **AES-CBC without MAC** | Vulnerable to padding-oracle attacks; use AES-GCM |
| **AES-GCM nonce reuse** | Completely breaks security; never reuse |
| **`Rfc2898DeriveBytes` constructors** | Default to SHA-1; obsolete in .NET 10; use static `Pbkdf2` |
| **`==` for hash comparison** | Timing side channel; use `FixedTimeEquals` |
| **Hard-coded keys/nonces** | Catastrophic; use a vault and random generation |
| **Post-quantum algorithms** | .NET 10+ only; OS support required |

---

## References (Consolidated)

- C# Cryptography Best Practices (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-cryptography-best-practices
- System.Security.Cryptography Namespace — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography
- PBKDF2 in C#: Rfc2898DeriveBytes.Pbkdf2 Example — MojoAuth - https://mojoauth.com/security-guides/pbkdf2-in-c
- AES-128 Encryption and Decryption in C# — MojoAuth - https://mojoauth.com/security-guides/aes-128-encryption--c
- Doing AES Encryption Correct in Your .NET Application — PropertyGuru - https://propertyguru.tech/doing-aes-encryption-correct-in-your-net-application
- Compile-N-Run — .NET Digital Signatures - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/13-net-security/7-net-digital-signatures.mdx
- Configure ASP.NET Core Data Protection — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview
- Secure an ASP.NET Core Blazor Web App with Microsoft Entra ID — Azure Key Vault key encryption - https://learn.microsoft.com/en-us/aspnet/core/blazor/security/blazor-web-app-with-entra
- Post-Quantum Cryptography in .NET 10 — Microsoft .NET Blog - https://devblogs.microsoft.com/dotnet/post-quantum-cryptography-in-dotnet/
- Post-Quantum Cryptography in .NET 10: A Practical Guide — Duende Software - https://duendesoftware.com/blog/20260514-post-quantum-cryptography-in-dotnet-10
- What’s New in .NET 10 Libraries and Runtime — GitHub - https://github.com/dotnet/core/blob/main/release-notes/10.0/whats-new.md
- SOTA-skills .NET security rules — GitHub - https://github.com/martinholovsky/SOTA-skills/blob/main/skills/sota-dotnet/rules/04-security.md
- OWASP Cryptographic Storage Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
- OWASP Password Storage Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html