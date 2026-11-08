# ASP.NET Core Data Protection: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **ASP.NET Core Data Protection** system — the framework's built‑in cryptographic API for encrypting and tamper‑proofing data that **only your application needs to read back**. It is not a general‑purpose encryption library, and it is **not** a password‑hashing mechanism. The distinction is critical: Data Protection is for **symmetric, reversible** protection of application data (cookies, antiforgery tokens, short‑lived opaque tokens); **hashing** (via `PasswordHasher<T>`) is for **irreversible** password storage. Confusing the two is a security anti‑pattern. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ASP.NET Core Data Protection is a framework that provides a simple, easy‑to‑use cryptographic API for encrypting and tamper‑proofing data using automatically managed keys, purpose‑based isolation, and authenticated encryption algorithms. |
| **Technical Definition** | The Data Protection system is built around two primary interfaces: **`IDataProtectionProvider`** (the root of the system) and **`IDataProtector`** (the component that performs `Protect` and `Unprotect` operations). `IDataProtector` is obtained by calling `IDataProtectionProvider.CreateProtector(purpose)`, where the **purpose string** derives a unique subkey for that consumer, providing cryptographic isolation even when the root keys are shared. The system manages **key generation, rotation, and persistence** automatically (default key lifetime: 90 days). Protected payloads are **encrypted and tamper‑proofed** (authenticated encryption); `Unprotect` throws `CryptographicException` if the payload has been tampered with or was produced by a different protector. |
| **Beginner‑Friendly Explanation** | Data Protection is the **“seal it in an envelope that only my app can open”** system. You take a piece of data — a cookie, a token, a session identifier — and call `Protect`. The system encrypts it and adds a tamper‑proof seal. Only your app can call `Unprotect` to read it back. The system manages the keys for you: it generates them, rotates them every 90 days, and (in production) persists them so all your servers can read the same data. **It is not for passwords** — passwords must be hashed, not encrypted. |

### Key Characteristics

- **Purpose‑based isolation** – Each consumer gets a unique subkey derived from its purpose string; one protector cannot unprotect another's payloads.
- **Authenticated encryption** – Payloads are encrypted **and** tamper‑proofed; tampering throws `CryptographicException`.
- **Automatic key management** – Keys are generated, rotated (default 90 days), and persisted by the system.
- **Key‑ring persistence** – In production, the key ring must be persisted (file system, database, Azure Blob) so all instances share keys.
- **Key encryption at rest** – Keys can be encrypted with DPAPI, X.509 certificates, or Azure Key Vault.
- **Not a password hasher** – Use `PasswordHasher<T>` for passwords; Data Protection is for reversible application data.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **dependency injection**, **middleware**, and **cookies**.
- Familiarity with **symmetric encryption**, **key management**, and **hashing** concepts.
- For production: a persistent key store (file system, database, or Azure Blob) and a key‑encryption mechanism (DPAPI, certificate, or Key Vault).

### Related Programming Areas

- **Cookie authentication** – Data Protection encrypts and signs authentication cookies.
- **Antiforgery** – CSRF tokens are protected with Data Protection.
- **Session state** – Session identifiers are protected.
- **Opaque tokens** – Short‑lived tokens that only the app needs to read.
- **Password hashing** – A separate concern, handled by `PasswordHasher<T>`.
- **Secret management** – Key encryption at rest depends on secure key storage.

### Core Concepts / Features

1. **Data Protection** – The overall system, architecture, and scope.
2. **`IDataProtector`** – The interface for `Protect` and `Unprotect`.
3. **Encryption** – Authenticated encryption, algorithms, and key management.
4. **Hashing** – Why it is different from encryption, and when to use `PasswordHasher<T>`.
5. **Data Protection Best Practices** – Key persistence, encryption at rest, purpose isolation, and multi‑instance deployments.

---

## 1. Data Protection (The System)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ASP.NET Core Data Protection is a framework that provides a simple, easy‑to‑use cryptographic API for encrypting and tamper‑proofing application data using automatically managed keys and purpose‑based isolation. |
| **Technical Definition** | The system is composed of **`IDataProtectionProvider`** (the root service, registered via `AddDataProtection()`) and **`IDataProtector`** (obtained via `CreateProtector(purpose)`). The provider manages the **key ring** — a collection of cryptographic keys with activation and expiration dates. When a key is created, it gets an activation date and a default **90‑day expiration**; expired keys remain usable for decryption but a new active key is generated. The system uses **authenticated encryption** (AES‑CBC with HMAC or AES‑GCM) and derives unique subkeys per purpose string. |
| **Beginner‑Friendly Explanation** | Data Protection is the **“seal and unseal”** system built into ASP.NET Core. It handles the hard parts of cryptography — key generation, rotation, algorithm selection, and tamper detection — so you can focus on calling `Protect` and `Unprotect`. The system is designed for **your app’s own data**: cookies, tokens, and session identifiers that no other system should read. It is not for passwords (hash those) and not for data that other systems need to decrypt (use explicit AES‑GCM for that). |

### Purposes

- **To** provide a **simple, safe** cryptographic API for encrypting and tamper‑proofing application data.
- **To** **manage keys automatically** — generation, rotation (90‑day default), and activation/expiration.
- **To** **isolate consumers** through purpose strings so one component cannot read another's payloads.
- **To** **protect cookies, antiforgery tokens, and opaque tokens** without requiring the developer to implement cryptography.
- **To** **scale across instances** when the key ring is persisted to a shared store.

### Syntax Rules and Structure

#### Service Registration

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDataProtection()
    .SetApplicationName("MyApp")   // Isolates this app from others sharing the key ring
    .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\keys\"))
    .ProtectKeysWithCertificate("{CERTIFICATE_THUMBPRINT}");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddDataProtection()` | Registers the Data Protection services. |
| `SetApplicationName("MyApp")` | Sets a unique application discriminator, isolating this app from others sharing the same key ring. |
| `PersistKeysToFileSystem(...)` | Persists the key ring to a shared directory. |
| `ProtectKeysWithCertificate(...)` | Encrypts keys at rest with an X.509 certificate. |

#### Consuming `IDataProtector`

```csharp
public class TokenService
{
    private readonly IDataProtector _protector;

    public TokenService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("MyApp.TokenService.v1");
    }

    public string Protect(string plaintext) => _protector.Protect(plaintext);
    public string Unprotect(string ciphertext) => _protector.Unprotect(ciphertext);
}
```

**Syntax rules:**

- **`AddDataProtection()` must be called** before resolving `IDataProtectionProvider` — the framework registers it automatically in ASP.NET Core, but explicit registration allows configuration.
- **`CreateProtector(purpose)` is required** — the purpose string is the isolation boundary; two protectors with different purposes cannot read each other's payloads.
- **`IDataProtector` is thread‑safe** — obtain it once and reuse it for multiple `Protect`/`Unprotect` calls.
- **`Unprotect` throws `CryptographicException`** if the payload is tampered with or was produced by a different protector.

**Constraints and limitations:**

- **Data Protection is not for passwords** — use `PasswordHasher<T>` for irreversible password storage.
- **Data Protection is not for cross‑system encryption** — if another system (e.g., a Python job) must decrypt the data, use explicit AES‑GCM with a documented format.
- **Key ring persistence is mandatory in production** — ephemeral keys cause cookie/token failures across restarts and instances.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configure Data Protection with persistent keys
builder.Services.AddDataProtection()
    .SetApplicationName("MyApp")
    .PersistKeysToFileSystem(new DirectoryInfo("/var/keys"))
    .ProtectKeysWithCertificate("{CERT_THUMBPRINT}");

var app = builder.Build();

// Consume the protector
app.MapGet("/protect", (IDataProtectionProvider provider) =>
{
    var protector = provider.CreateProtector("MyApp.Demo.v1");
    var protectedPayload = protector.Protect("Hello, world!");
    return Results.Ok(protectedPayload);
});

app.MapGet("/unprotect", (IDataProtectionProvider provider, string payload) =>
{
    var protector = provider.CreateProtector("MyApp.Demo.v1");
    try
    {
        var plaintext = protector.Unprotect(payload);
        return Results.Ok(plaintext);
    }
    catch (CryptographicException)
    {
        return Results.BadRequest("Invalid payload.");
    }
});
```

**Step‑by‑step explanation:**

1. `AddDataProtection()` registers the system; `SetApplicationName` isolates this app; `PersistKeysToFileSystem` and `ProtectKeysWithCertificate` configure production‑grade key storage and encryption.
2. The `/protect` endpoint creates a protector with the purpose `"MyApp.Demo.v1"` and protects a string.
3. The `/unprotect` endpoint uses the **same purpose** to decrypt; a different purpose would throw `CryptographicException`.

**Expected Output:** The `/protect` endpoint returns an opaque, encrypted, tamper‑proof string; `/unprotect` returns the original `"Hello, world!"` when given the same payload and purpose.

**Why this output?** The purpose string derives the correct subkey; the authenticated encryption ensures integrity.

### Real‑World Cases

- **Cookie authentication:** ASP.NET Core Identity and cookie middleware use Data Protection to encrypt and sign cookies.
- **Antiforgery tokens:** CSRF tokens are protected with Data Protection.
- **Opaque API tokens:** Short‑lived tokens that only the issuing app needs to read.
- **Session identifiers:** Session state identifiers are protected.

### References

- Consumer APIs overview for ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/consumer-apis/overview.md
- Get started with the Data Protection APIs – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/consumer-apis/overview.md
- Configure ASP.NET Core Data Protection – https://learn.microsoft.com/el-gr/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-10.0

---

## 2. `IDataProtector` (The Interface)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IDataProtector` is the interface that performs the actual `Protect` and `Unprotect` operations on data, using the subkey derived from its purpose string. |
| **Technical Definition** | `IDataProtector` extends `IDataProtectionProvider` (so purposes can be chained) and defines `Protect(byte[])`/`Protect(string)` and `Unprotect(byte[])`/`Unprotect(string)` methods. It is obtained by calling `IDataProtectionProvider.CreateProtector(purpose)`. The protector is **thread‑safe** and intended to be created once per consumer and reused. The `Protect` method returns a payload that is **encrypted and tamper‑proofed**; `Unprotect` verifies the integrity and decrypts, throwing `CryptographicException` on any failure. |
| **Beginner‑Friendly Explanation** | `IDataProtector` is the **“seal”** itself. You get one from the provider by giving it a purpose string. Then you call `Protect` to seal data and `Unprotect` to open it. The seal is cryptographically bound to the purpose — a seal created with one purpose cannot be opened with another. The protector is safe to reuse, so you create it once and keep it. |

### Purposes

- **To** perform **`Protect` and `Unprotect`** operations on application data.
- **To** enforce **purpose‑based isolation** — payloads protected by one purpose cannot be unprotected by another.
- **To** provide **thread‑safe** cryptographic operations for concurrent request processing.
- **To** **detect tampering** — `Unprotect` throws if the payload has been modified.
- **To** support **chained purposes** for hierarchical isolation (e.g., per‑tenant protectors).

### Syntax Rules and Structure

#### `Protect` and `Unprotect`

```csharp
var protector = provider.CreateProtector("MyApp.Secrets.v1");

// Protect
string protectedPayload = protector.Protect("sensitive data");

// Unprotect
try
{
    string plaintext = protector.Unprotect(protectedPayload);
}
catch (CryptographicException)
{
    // Tampered or wrong protector
}
```

**Syntax rules:**

- **`CreateProtector(purpose)` must be called** to obtain an `IDataProtector` — the provider itself cannot protect or unprotect data.
- **The purpose string must be unique to the consumer** — using the same purpose across different components destroys isolation.
- **`Unprotect` throws `CryptographicException`** on tampering or mismatched protector — catch it and treat the payload as invalid.
- **`IDataProtector` is thread‑safe** — create it once and reuse it.

**Constraints and limitations:**

- **Do not use user input as the sole purpose string** — an attacker could force a collision with another component's purpose. Combine a fixed prefix with the user identifier.
- **The protected payload is not human‑readable** — it is encrypted and Base64URL‑encoded.
- **`Unprotect` cannot recover data protected by a different purpose** — by design.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class SecureMessageService
{
    private readonly IDataProtector _protector;

    public SecureMessageService(IDataProtectionProvider provider)
    {
        // Purpose includes the component name and version for isolation
        _protector = provider.CreateProtector(
            "Contoso.Messaging.SecureMessage", "v1");
    }

    public string ProtectMessage(string plaintext)
        => _protector.Protect(plaintext);

    public string? UnprotectMessage(string protectedPayload)
    {
        try
        {
            return _protector.Unprotect(protectedPayload);
        }
        catch (CryptographicException)
        {
            return null;   // Tampered or invalid
        }
    }
}
```

**Step-by-step explanation:**

1. The protector is created with a hierarchical purpose: `"Contoso.Messaging.SecureMessage", "v1"`.
2. `ProtectMessage` seals the plaintext.
3. `UnprotectMessage` attempts to open it, returning `null` on failure.

**Expected Output:** A protected payload that only this service (with this purpose) can unprotect; tampered payloads return `null`.

**Why this output?** The purpose string isolates this service; `Unprotect` throws on tampering, which is caught and handled safely.

### Real‑World Cases

- **Per‑tenant isolation:** `provider.CreateProtector("Tenant", tenantId)` gives each tenant a separate cryptographic boundary.
- **Versioned purposes:** `"MyApp.Token.v1"` and `"MyApp.Token.v2"` isolate versions during migration.
- **Component‑specific protection:** Each service uses its own purpose namespace.

### References

- Consumer APIs overview for ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/consumer-apis/overview.md
- Purpose strings in ASP.NET Core – https://learn.microsoft.com/es-es/aspnet/core/security/data-protection/consumer-apis/purpose-strings?view=aspnetcore-9.0
- Purpose hierarchy and multi‑tenancy – https://learn.microsoft.com/es-es/aspnet/core/security/data-protection/consumer-apis/purpose-strings-multitenancy?view=aspnetcore-9.0

---

## 3. Encryption (Authenticated Encryption and Key Management)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Data Protection uses **authenticated encryption** — a combination of confidentiality (encryption) and integrity (tamper‑proofing) — with automatically managed keys to protect application data. |
| **Technical Definition** | The system supports **AES‑CBC with HMAC** (explicit encryption + validation algorithms) and **AES‑GCM** (built‑in authentication). The framework handles **AAD (Additional Authenticated Data)** automatically and verifies the MAC **before** decrypting, exposing only a generic `CryptographicException` to prevent padding‑oracle attacks. The **key ring** is the collection of master keys; each key has an activation date and a default **90‑day expiration**; expired keys remain for decryption while a new active key is generated. |
| **Beginner‑Friendly Explanation** | Encryption in Data Protection is **“scramble the data and attach a tamper‑proof seal.”** The system uses modern authenticated encryption algorithms (AES‑CBC with HMAC, or AES‑GCM) so that even if someone modifies the ciphertext, the seal breaks and `Unprotect` refuses to return data. The keys that drive the encryption are managed by the framework: they are generated, rotated every 90 days, and (in production) persisted so all servers can read the same data. |

### Purposes

- **To** provide **confidentiality** — the protected payload cannot be read without the key.
- **To** provide **integrity** — tampering is detected and rejected before decryption.
- **To** **rotate keys automatically** — the 90‑day lifetime limits the exposure window of any single key.
- **To** **support multiple algorithms** — AES‑CBC+HMAC or AES‑GCM, selected via `UseCryptographicAlgorithms`.
- **To** **encrypt keys at rest** — DPAPI, X.509 certificates, or Azure Key Vault protect the key ring itself.

### Syntax Rules and Structure

#### Configuring Cryptographic Algorithms

```csharp
builder.Services.AddDataProtection()
    .UseCryptographicAlgorithms(new AuthenticatedEncryptorConfiguration
    {
        EncryptionAlgorithm = EncryptionAlgorithm.AES_256_GCM
        // AES-GCM has built-in authentication; ValidationAlgorithm is ignored
    });
```

**Syntax rules:**

- **`UseCryptographicAlgorithms`** selects the encryption algorithm; for AES‑CBC, pair it with `ValidationAlgorithm = ValidationAlgorithm.HMACSHA256`.
- **AES‑GCM does not need a separate validation algorithm** — authentication is built in.
- **The framework handles AAD automatically** — do not attempt to supply it.

#### Key Lifetime

```csharp
builder.Services.AddDataProtection()
    .SetDefaultKeyLifetime(TimeSpan.FromDays(14));   // Default is 90 days
```

**Syntax rules:**

- **Default key lifetime is 90 days** — change it with `SetDefaultKeyLifetime`.
- **Expired keys remain usable for decryption** — the system generates a new active key but keeps retired keys so old data can still be read.

**Constraints and limitations:**

- **Key rotation does not immediately invalidate old data** — retired keys remain on the system.
- **Algorithm changes require careful migration** — data protected with the old algorithm must be re‑protected or read with the old configuration.
- **Custom algorithm implementations must be carefully validated** — public parameterless constructors and minimum key sizes are required.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddDataProtection()
    .SetApplicationName("MyApp")
    .PersistKeysToFileSystem(new DirectoryInfo("/var/keys"))
    .ProtectKeysWithCertificate("{CERT_THUMBPRINT}")
    .SetDefaultKeyLifetime(TimeSpan.FromDays(30))
    .UseCryptographicAlgorithms(new AuthenticatedEncryptorConfiguration
    {
        EncryptionAlgorithm = EncryptionAlgorithm.AES_256_GCM
    });
```

**Step-by-step explanation:**

1. The application name isolates this app's keys from others.
2. The key ring is persisted to a shared directory and encrypted at rest with a certificate.
3. Key lifetime is reduced to 30 days for tighter rotation.
4. AES‑256‑GCM is selected for authenticated encryption with built‑in integrity.

**Expected Output:** Keys are generated, rotated every 30 days, persisted to the shared directory, and encrypted at rest; payloads are protected with AES‑256‑GCM.

**Why this output?** Each configuration call customises a different aspect of key management and encryption.

### Real-World Cases

- **Multi‑instance deployments:** Shared key ring in a database, file share, or Azure Blob Storage.
- **Compliance:** Key encryption at rest with X.509 certificates or Azure Key Vault.
- **High‑security environments:** AES‑GCM for authenticated encryption with a shorter key lifetime.

### References

- Security cards: Configure Authenticated Encryption and Cryptographic Algorithms – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/cryptography.md
- Key management in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/implementation/key-management.md
- Key encryption at rest – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/implementation/key-encryption-at-rest.md

---

## 4. Hashing (The Distinct Concern)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Hashing is a **one‑way** cryptographic operation that transforms data (typically a password) into a fixed‑length digest that cannot be reversed, used for verification rather than retrieval. |
| **Technical Definition** | ASP.NET Core Identity provides **`PasswordHasher<TUser>`** for password storage. It uses **PBKDF2** with HMAC‑SHA256, a 128‑bit salt, a 256‑bit subkey, and 10,000 iterations by default. **`PasswordHasher<T>` does not use Data Protection** — it is a separate, purpose‑built password‑hashing mechanism. `KeyDerivation.Pbkdf2` is also available but is **not recommended for new apps**; new apps should use `PasswordHasher<T>`. |
| **Beginner‑Friendly Explanation** | Hashing is the **“fingerprint”** of a password. You cannot turn a fingerprint back into a person, and you cannot turn a hash back into a password. When a user logs in, you hash the provided password and compare the hash to the stored one. If an attacker steals the database, they get hashes, not passwords — and reversing PBKDF2 hashes is computationally infeasible. **Data Protection is not for this** — it is reversible encryption, which is the wrong tool for passwords. |

### Purposes

- **To** store passwords **irreversibly** — even the application cannot recover the original password.
- **To** use **salted, iterated hashing** (PBKDF2) to resist rainbow‑table and brute‑force attacks.
- **To** provide a **purpose‑built API** (`PasswordHasher<T>`) that handles salt generation and verification.
- **To** **separate** password security from reversible application encryption — the two use different primitives and must not be conflated.

### Syntax Rules and Structure

#### Hashing with `PasswordHasher<T>`

```csharp
var hasher = new PasswordHasher<IdentityUser>();
var user = new IdentityUser { UserName = "alice" };

// Hash
string hash = hasher.HashPassword(user, "P@ssw0rd!");

// Verify
var result = hasher.VerifyHashedPassword(user, hash, "P@ssw0rd!");
if (result == PasswordVerificationResult.Success)
{
    // Authenticated
}
```

**Syntax rules:**

- **Use `PasswordHasher<T>` for password storage** — it is the recommended API.
- **The user object is part of the hash input** — it binds the hash to the user, preventing hash‑swapping attacks.
- **`VerifyHashedPassword` returns a result** — `Success`, `Failed`, or `SuccessRehashNeeded`.
- **Never encrypt passwords** — encryption is reversible; hashing is not.

**Constraints and limitations:**

- **`KeyDerivation.Pbkdf2` is not recommended for new apps** — use `PasswordHasher<T>`.
- **Hashing is intentionally slow** — it is designed to resist brute force, not to be fast.
- **Password resets require a token, not decryption** — because the original password cannot be recovered.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class PasswordService
{
    private readonly PasswordHasher<IdentityUser> _hasher = new();

    public string HashPassword(IdentityUser user, string password)
        => _hasher.HashPassword(user, password);

    public bool VerifyPassword(IdentityUser user, string hash, string password)
        => _hasher.VerifyHashedPassword(user, hash, password)
           == PasswordVerificationResult.Success;
}
```

**Step-by-step explanation:**

1. `PasswordHasher<IdentityUser>` is instantiated.
2. `HashPassword` produces a salted, iterated PBKDF2 hash bound to the user.
3. `VerifyPassword` compares the provided password to the stored hash.

**Expected Output:** A stored hash (not the password) and a boolean verification result.

**Why this output?** Hashing is one‑way; verification recomputes the hash and compares.

### Real‑World Cases

- **User registration and login:** Passwords are hashed on registration and verified on login.
- **Password reset:** Tokens are used for reset; the password itself is never recovered.
- **Compliance:** PBKDF2 with sufficient iterations meets common password‑storage standards.

### References

- Hash passwords in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/consumer-apis/password-hashing.md
- Does `PasswordHasher<T>` use Data Protection? (Stack Overflow) – https://stackoverflow.com/questions/…/does-microsoft-aspnetcore-identity-passwordhashertuser-use-aspnet-core-data-protection
- ASP.NET Core Data Protection vs AES vs Password Hashing – https://www.muhammadshahid.dev/blog/aspnet-core-data-protection-aes

---

## 5. Data Protection Best Practices (The Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Data Protection best practices are the guidelines for configuring, persisting, and consuming the Data Protection system so that it is secure, reliable, and scalable in production. |
| **Technical Definition** | Best practices encompass **key‑ring persistence** (shared storage for multi‑instance deployments), **key encryption at rest** (DPAPI, certificate, or Key Vault), **application isolation** (`SetApplicationName`), **purpose‑string discipline** (unique, versioned, never user‑input‑only), **lifetime tuning** (90‑day default, configurable), **algorithm selection** (AES‑GCM or AES‑CBC+HMAC), and **error handling** (catch `CryptographicException` and treat payload as invalid). |
| **Beginner‑Friendly Explanation** | These are the **“do not ship without these”** rules for Data Protection. The biggest mistakes are: using ephemeral keys in production (cookies break on restart), not isolating applications (one app can read another's cookies), using user input as the only purpose string (attackers can force collisions), and forgetting to encrypt keys at rest (stolen key ring = compromised data). |

### Purposes

- **To** ensure **key‑ring persistence** so data survives restarts and is shared across instances.
- **To** **encrypt keys at rest** so a stolen key ring does not compromise protected data.
- **To** **isolate applications** that share a key ring using `SetApplicationName`.
- **To** **isolate consumers** within an application using purpose strings.
- **To** **handle `CryptographicException`** safely — treat the payload as invalid, not as a server error.
- **To** **rotate keys** according to the security policy (default 90 days, configurable).

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Key persistence** | Persist the key ring to a shared store (file system, database, Azure Blob) in production. | Rely on ephemeral keys — cookies/tokens break on restart or across instances. |
| **Key encryption at rest** | Encrypt keys with DPAPI (Windows), X.509 certificate (multi‑machine), or Azure Key Vault. | Leave keys unencrypted at rest — a stolen key ring compromises all protected data. |
| **Application isolation** | Call `SetApplicationName("MyApp")` when sharing a key ring. | Share a key ring without an application discriminator — apps can read each other's data. |
| **Purpose strings** | Use unique, versioned, component‑scoped purpose strings (`"MyApp.Token.v1"`). | Use the same purpose for everything, or use only user input — it destroys isolation. |
| **Key lifetime** | Use the 90‑day default or a shorter, policy‑driven lifetime. | Set an extremely short lifetime without planning for re‑protection of existing data. |
| **Algorithm** | Use AES‑GCM (built‑in authentication) or AES‑CBC+HMAC via `UseCryptographicAlgorithms`. | Attempt to implement your own encryption — the framework handles AAD and nonces. |
| **Error handling** | Catch `CryptographicException` and treat the payload as invalid. | Let `CryptographicException` propagate as a 500 — it is a client‑data problem, not a server error. |
| **Scope** | Use Data Protection for app‑only data (cookies, antiforgery, opaque tokens). | Use Data Protection for passwords (hash instead) or cross‑system data (use explicit AES‑GCM). |

### Annotated Code Example (Best‑Practice Configuration)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDataProtection()
    .SetApplicationName("MyApp")
    .PersistKeysToAzureBlobStorage(
        new Uri("https://mystorage.blob.core.windows.net/keys"),
        new DefaultAzureCredential())
    .ProtectKeysWithAzureKeyVault(
        new Uri("https://myvault.vault.azure.net/keys/dataprotection"),
        new DefaultAzureCredential())
    .SetDefaultKeyLifetime(TimeSpan.FromDays(90))
    .UseCryptographicAlgorithms(new AuthenticatedEncryptorConfiguration
    {
        EncryptionAlgorithm = EncryptionAlgorithm.AES_256_GCM
    });
```

**Why this follows best practices:**

- **Key ring is persisted to Azure Blob Storage** — shared across instances and durable across restarts.
- **Keys are encrypted at rest with Azure Key Vault** — a stolen blob does not compromise the keys.
- **`SetApplicationName`** isolates this app from others sharing the storage account.
- **Key lifetime is explicitly set** — the default 90 days is documented and intentional.
- **AES‑256‑GCM** provides authenticated encryption with built‑in integrity.

### Real‑World Cases

- **Azure App Service scale‑out:** Key ring in Blob Storage, encrypted with Key Vault, shared across instances.
- **On‑premises web farms:** Key ring on a UNC share, encrypted with an X.509 certificate.
- **Multi‑tenant SaaS:** Per‑tenant purpose strings with a shared application discriminator.

### References

- Configure ASP.NET Core Data Protection – https://learn.microsoft.com/el-gr/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-10.0
- Key storage providers in ASP.NET Core – https://learn.microsoft.com/zh-hk/aspnet/core/security/data-protection/implementation/key-storage-providers?view=aspnetcore-5.0
- Key encryption at rest – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/implementation/key-encryption-at-rest.md
- Purpose strings – https://learn.microsoft.com/es-es/aspnet/core/security/data-protection/consumer-apis/purpose-strings?view=aspnetcore-9.0
- Security cards: Cryptography rules – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/cryptography.md

---

## Summary Table: Data Protection at a Glance

| Concept | Primary Role | Key API / Tool | When to Use |
|---------|--------------|----------------|-------------|
| **Data Protection** | Encrypt and tamper‑proof app data | `AddDataProtection()`, `IDataProtectionProvider` | Cookies, antiforgery, opaque tokens |
| **`IDataProtector`** | Perform `Protect` / `Unprotect` | `CreateProtector(purpose)`, `Protect`, `Unprotect` | Every consumer of the system |
| **Encryption** | Confidentiality + integrity | AES‑GCM or AES‑CBC+HMAC, key ring, 90‑day rotation | Every protected payload |
| **Hashing** | Irreversible password storage | `PasswordHasher<T>`, PBKDF2 | Passwords only — **not** Data Protection |
| **Best practices** | Secure, reliable, scalable protection | Key persistence, encryption at rest, purpose isolation | Every production deployment |

---

## References (Full List)

- Consumer APIs overview for ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/consumer-apis/overview.md
- Get started with the Data Protection APIs – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/consumer-apis/overview.md
- Configure ASP.NET Core Data Protection – https://learn.microsoft.com/el-gr/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-10.0
- Key storage providers in ASP.NET Core – https://learn.microsoft.com/zh-hk/aspnet/core/security/data-protection/implementation/key-storage-providers?view=aspnetcore-5.0
- Key encryption at rest – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/implementation/key-encryption-at-rest.md
- Key management in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/implementation/key-management.md
- Purpose strings in ASP.NET Core – https://learn.microsoft.com/es-es/aspnet/core/security/data-protection/consumer-apis/purpose-strings?view=aspnetcore-9.0
- Purpose hierarchy and multi‑tenancy – https://learn.microsoft.com/es-es/aspnet/core/security/data-protection/consumer-apis/purpose-strings-multitenancy?view=aspnetcore-9.0
- Security cards: Configure Authenticated Encryption and Cryptographic Algorithms – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/cryptography.md
- Hash passwords in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/security/data-protection/consumer-apis/password-hashing.md
- Does `PasswordHasher<T>` use Data Protection? (Stack Overflow) – https://stackoverflow.com/questions/…/does-microsoft-aspnetcore-identity-passwordhashertuser-use-aspnet-core-data-protection
- ASP.NET Core Data Protection vs AES vs Password Hashing – https://www.muhammadshahid.dev/blog/aspnet-core-data-protection-aes
- Persist Data Protection keys across restarts and replicas (GitHub) – https://github.com/AsiBackbone/NetCoreApplicationTemplate/commit/641327f
- Data Protection (OpenIddict components) – https://docs.openiddictcomponents.com/