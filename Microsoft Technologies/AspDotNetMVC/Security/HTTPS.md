# ASP.NET Core HTTPS: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **HTTPS/TLS** stack: transport security, certificate management, HTTP Strict Transport Security (HSTS), and production deployment patterns. HTTPS is not a single setting—it is the combination of **TLS termination** (at Kestrel or a reverse proxy), **certificate trust**, **HSTS enforcement**, and **correct scheme detection behind proxies**. A critical framing note: **Kestrel can terminate TLS directly**, but the preferred production layout is often to **bind Kestrel to loopback and terminate TLS in a reverse proxy** (Caddy, Nginx, Cloudflare) . Each requested topic is treated independently—no sections are merged.


## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | HTTPS is HTTP transmitted over **TLS (Transport Layer Security)** , providing **confidentiality** (encryption), **integrity** (tamper detection), and **authenticity** (server identity via certificates) for data in transit. |
| **Technical Definition** | HTTPS combines HTTP with TLS. The client and server perform a **TLS handshake** to negotiate protocol version and cipher suite, the server presents an **X.509 certificate** proving its identity, and a symmetric session key is established for encrypted communication. ASP.NET Core supports HTTPS through **Kestrel** (direct TLS termination) or a **reverse proxy** (TLS terminated at the edge, with **Forwarded Headers** restoring the correct scheme to the app) . **HSTS** is an opt‑in response header that instructs browsers to only use HTTPS for future visits. |
| **Beginner‑Friendly Explanation** | HTTPS is the **“sealed envelope”** version of HTTP. Without it, anyone on the network can read or modify the data between the user and your app. With it, the data is encrypted, tamper‑proof, and the user can verify they are talking to your server (not an impostor). The **certificate** is the server’s ID card; **TLS** is the protocol that does the encryption; **HSTS** tells the browser “always use the sealed envelope, never the postcard.” |


### Key Characteristics

- **Confidentiality** — TLS encrypts the payload so eavesdroppers see only ciphertext.
- **Integrity** — TLS detects tampering in transit.
- **Authenticity** — the server presents an X.509 certificate; the client verifies it against a trusted root CA.
- **Protocol versions** — TLS 1.2 and TLS 1.3 are the modern standards; older versions (SSL 3.0, TLS 1.0/1.1) are deprecated .
- **Certificate types** — development (self‑signed, trusted locally), CA‑issued (production), wildcard (`*.example.com`), and SAN (multiple hostnames) .
- **HSTS** — `Strict-Transport-Security` header tells browsers to refuse HTTP for a configurable period (`max-age`), optionally including subdomains (`includeSubDomains`) and preload lists .
- **Reverse‑proxy awareness** — when TLS terminates at a proxy, **Forwarded Headers Middleware** must restore the original scheme so `UseHttpsRedirection` and `UseHsts` behave correctly .


### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware**, **Kestrel**, **dependency injection**, and **configuration**.
- Familiarity with **certificates** (X.509, PFX/PEM), **TLS handshake**, and **reverse proxies**.
- For development: the ASP.NET Core HTTPS development certificate (`dotnet dev-certs https --trust`) .


### Related Programming Areas

- **Kestrel** — the web server that can terminate TLS directly .
- **Reverse proxies** — Nginx, Caddy, Cloudflare, Azure App Service, and YARP terminate TLS at the edge .
- **HSTS** — browser‑side enforcement of HTTPS .
- **Forwarded Headers** — scheme and client‑IP restoration behind proxies .
- **Certificate authentication (mTLS)** — client certificates for mutual TLS .
- **Security headers** — CSP, `X-Content-Type-Options`, and other browser defenses that complement HTTPS .


### Core Concepts / Features

1. **HTTPS** — the overall transport security model and how ASP.NET Core enforces it.
2. **TLS** — protocol versions, configuration, and cipher suites.
3. **Certificates** — development certificates, CA‑issued certificates, and Kestrel configuration.
4. **HSTS** — Strict‑Transport‑Security header, configuration, and rollout strategy.
5. **HTTPS Best Practices** — production deployment patterns, reverse proxies, and operational hygiene.


## 1. HTTPS (The Transport Security Model)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | HTTPS is HTTP over TLS, providing encrypted, authenticated, and tamper‑proof communication between the client and the server. |
| **Technical Definition** | ASP.NET Core enforces HTTPS through **`UseHttpsRedirection()`** (redirects HTTP requests to HTTPS with a 307/308 status) and **`UseHsts()`** (adds the `Strict-Transport-Security` header). Redirection requires knowing the HTTPS port—configured via `ASPNETCORE_HTTPS_PORT`, `options.HttpsPort`, or Kestrel endpoint configuration . When TLS terminates at a reverse proxy, **`UseForwardedHeaders()`** must run **before** redirection so the app sees `Request.IsHttps = true` and does not create an infinite redirect loop . |
| **Beginner‑Friendly Explanation** | HTTPS enforcement is the **“redirect and remember”** system. If a user types `http://`, the app redirects them to `https://`. If the app is behind a proxy that already handles HTTPS, the app needs to be told “the request is actually secure” via forwarded headers—otherwise it keeps redirecting in a loop. **HSTS** takes it further: the browser remembers the instruction and refuses HTTP for future visits, even before contacting the server. |


### Purposes

- **To** **encrypt data in transit** — prevent eavesdropping on credentials, tokens, and personal data.
- **To** **prevent downgrade attacks** — HSTS stops attackers from forcing the browser back to HTTP.
- **To** **satisfy compliance** — PCI DSS, HIPAA, and modern browser requirements demand HTTPS.
- **To** **enable HTTP/2 and HTTP/3** — both protocols require TLS in browsers.
- **To** **protect authentication** — cookies and tokens must travel over encrypted channels.


### Syntax Rules and Structure

#### General Syntax (HTTPS Redirection)

```csharp
builder.Services.AddHttpsRedirection(options =>
{
    options.RedirectStatusCode = StatusCodes.Status307TemporaryRedirect;
    options.HttpsPort = 5001;   // Or configure ASPNETCORE_HTTPS_PORT
});

var app = builder.Build();
app.UseHttpsRedirection();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddHttpsRedirection()` | Configures the redirection middleware. |
| `RedirectStatusCode` | 307 (temporary) or 308 (permanent). |
| `HttpsPort` | The port HTTPS listens on; required if it cannot be inferred. |

#### General Syntax (Behind a Reverse Proxy)

```csharp
app.UseForwardedHeaders();   // Must run before UseHttpsRedirection
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
app.UseHttpsRedirection();
```

**Syntax rules:**

- **`UseForwardedHeaders()` must precede `UseHttpsRedirection()`** — otherwise the app sees HTTP and redirects endlessly .
- **`UseHsts()` must only run outside Development** — HSTS is cached by browsers and difficult to undo .
- **The HTTPS port must be configured** — `ASPNETCORE_HTTPS_PORT`, `options.HttpsPort`, or Kestrel endpoints .

**Constraints and limitations:**

- **Redirection does not protect the first request** — the initial HTTP request is still sent in cleartext before the redirect; HSTS closes this window after the first visit .
- **Proxy misconfiguration causes redirect loops** — if forwarded headers are missing, `Request.IsHttps` is false and the app redirects the already‑HTTPS request back to HTTPS .


### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHttpsRedirection(options =>
{
    options.RedirectStatusCode = StatusCodes.Status308PermanentRedirect;
    options.HttpsPort = 443;
});

var app = builder.Build();

app.UseForwardedHeaders();   // Restores scheme behind a proxy
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Step‑by‑step explanation:**

1. `AddHttpsRedirection` configures a permanent (308) redirect to port 443.
2. `UseForwardedHeaders` restores the original scheme before redirection runs.
3. `UseHsts` is gated to non‑Development environments.
4. `UseHttpsRedirection` redirects HTTP requests to HTTPS.

**Expected Output:** HTTP requests receive a 308 redirect to the HTTPS URL; requests arriving via a proxy that terminated TLS are recognized as already secure and are not redirected.

**Why this output?** The forwarded headers middleware sets `Request.Scheme` to `https`, so the redirection middleware correctly sees the request as secure.

### Real‑World Cases

- **Public web apps** — redirect all HTTP traffic to HTTPS and enable HSTS.
- **API projects** — configure API projects to **only listen on HTTPS** rather than redirecting, as recommended for backend services .
- **Behind Azure App Service or Nginx** — use forwarded headers to avoid redirect loops .

### References

- Enforce HTTPS in ASP.NET Core – https://learn.microsoft.com/lb-lu/aspnet/core/security/enforcing-ssl?view=aspnetcore-11.0#4
- Configure ASP.NET Core to work with proxy servers and load balancers – https://learn.microsoft.com/vi-vn/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-10.0&viewFallbackFrom=aspnetcore-1.0#2
- ASP.NET Core Security Checklist (2026 Production Guide) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist


## 2. TLS (The Protocol)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | TLS (Transport Layer Security) is the cryptographic protocol that provides encrypted, authenticated communication over a network, and is the “S” in HTTPS. |
| **Technical Definition** | TLS negotiates a **protocol version** (TLS 1.2, TLS 1.3), a **cipher suite** (key exchange, bulk encryption, and MAC algorithms), and authenticates the server via its **X.509 certificate**. Kestrel supports TLS 1.2 and TLS 1.3 directly, including SNI and mutual TLS (mTLS) . The default `SslProtocols.None` delegates protocol selection to the **operating system**, which typically means TLS 1.2 and TLS 1.3 on modern systems . |
| **Beginner‑Friendly Explanation** | TLS is the **“encryption engine”** behind HTTPS. It is the set of rules that the client and server follow to agree on how to encrypt the connection. Modern TLS (1.2 and 1.3) is secure; old versions (SSL 3.0, TLS 1.0/1.1) are broken and must be disabled. Kestrel can do TLS itself, or you can let a reverse proxy handle it and forward plain HTTP internally. |


### Purposes

- **To** provide **strong encryption** using modern cipher suites.
- **To** **authenticate the server** through its certificate.
- **To** **support forward secrecy** (TLS 1.3) and modern key exchange.
- **To** **allow the OS to negotiate the best protocol** when `SslProtocols.None` is used.
- **To** **enable mutual TLS (mTLS)** for client authentication in zero‑trust environments .


### Syntax Rules and Structure

#### General Syntax (Explicit Protocol Configuration)

```csharp
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.SslProtocols = SslProtocols.Tls12 | SslProtocols.Tls13;
    });
});
```

#### General Syntax (appsettings.json)

```json
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://*:443",
        "SslProtocols": ["Tls12", "Tls13"],
        "Certificate": {
          "Path": "/etc/ssl/private/server.pfx",
          "Password": "FROM_ENVIRONMENT"
        }
      }
    }
  }
}
```

**Syntax rules:**

- **`SslProtocols.None` is the default** — the OS selects the best protocol; this is usually correct .
- **Explicitly enabling `Tls12 | Tls13`** is acceptable when you need to guarantee a minimum version .
- **Never enable SSL 3.0, TLS 1.0, or TLS 1.1** — they are deprecated and vulnerable .
- **Certificate passwords must come from environment variables or a secret store** — never plaintext in `appsettings.json` .

**Constraints and limitations:**

- **The OS must support TLS 1.3** — older operating systems may not; `SslProtocols.None` gracefully negotiates the best available version.
- **Cipher suite configuration is OS‑level** — Kestrel does not expose per‑cipher configuration; use OS policy or a reverse proxy for fine‑grained control.


### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        // Explicitly allow only TLS 1.2 and 1.3
        listenOptions.SslProtocols = SslProtocols.Tls12 | SslProtocols.Tls13;
    });
});

var app = builder.Build();
app.UseHttpsRedirection();
app.Run();
```

**Step‑by‑step explanation:**

1. `ConfigureKestrel` customises the HTTPS defaults.
2. `SslProtocols.Tls12 | SslProtocols.Tls13` disables older protocol versions.
3. The app only accepts TLS 1.2 and 1.3 connections.

**Expected Output:** Clients attempting SSL 3.0, TLS 1.0, or TLS 1.1 are rejected; TLS 1.2 and 1.3 connections succeed.

**Why this output?** The allowed protocol list is restricted at the Kestrel level.

### Real‑World Cases

- **PCI DSS compliance** — requires TLS 1.2+ and disallows older protocols.
- **Modern browser compatibility** — TLS 1.2 and 1.3 are universally supported.
- **Zero‑trust networks** — mTLS with client certificates for service‑to‑service authentication .

### References

- Kestrel security considerations (TLS 1.2, TLS 1.3, SNI, mTLS) – https://learn.microsoft.com/…/kestrel/security
- Kestrel: Default supported TLS protocol versions changed – https://github.com/dotnet/aspnetcore/…
- Configure endpoints for Kestrel (SslProtocols configuration) – https://learn.microsoft.com/ka-ge/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-9.0#2


## 3. Certificates (Server Identity)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An X.509 certificate is a digitally signed document that binds a public key to a server’s identity (hostname), enabling clients to verify they are communicating with the legitimate server. |
| **Technical Definition** | ASP.NET Core uses certificates in **PFX** (PKCS#12, with private key) or **PEM** (certificate + key files) format. Kestrel can load certificates from a file, the Windows certificate store, or a `X509Certificate2` instance . **Development certificates** are self‑signed and trusted locally via `dotnet dev-certs https --trust`. **Production certificates** are issued by a trusted CA and must cover every hostname the app serves, using **Subject Alternative Names (SANs)** or a **wildcard** (`*.example.com`) . |
| **Beginner‑Friendly Explanation** | A certificate is the **server’s ID card**. It says “this public key belongs to `example.com`” and is signed by a trusted authority. The browser checks the signature and the hostname before trusting the connection. In development, a self‑signed certificate is fine because you trust it manually. In production, you need a certificate from a real CA—either a single‑domain cert, a SAN cert covering multiple names, or a wildcard cert covering all subdomains. |


### Purposes

- **To** **prove server identity** — prevent man‑in‑the‑middle attacks.
- **To** **enable TLS** — the certificate’s public key is used in the handshake.
- **To** **cover all hostnames** — SANs and wildcards ensure every name the app serves is valid.
- **To** **support certificate rotation** — certificates expire and must be renewed.
- **To** **enable mTLS** — client certificates authenticate the client to the server .


### Syntax Rules and Structure

#### Development Certificate (Trust Locally)

```bash
dotnet dev-certs https --trust
```

#### General Syntax (Kestrel with PFX)

```csharp
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
    {
        listenOptions.UseHttps("server.pfx", "FROM_ENVIRONMENT");
    });
});
```

#### General Syntax (appsettings.json)

```json
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://*:443",
        "Certificate": {
          "Path": "/etc/ssl/private/server.pfx",
          "Password": "FROM_ENVIRONMENT"
        }
      }
    }
  }
}
```

**Syntax rules:**

- **The certificate password must never be committed to source control** — use environment variables, Azure Key Vault, or another secret store .
- **PFX files contain the private key** — protect them with file permissions and encryption.
- **SAN certificates must include every hostname** — wildcards cover only one level (`*.example.com` covers `api.example.com` but not `api.eu.example.com`) .
- **Development certificates must not be used in production** — they are self‑signed and not trusted by clients .

**Constraints and limitations:**

- **Certificate expiry** — expired certificates cause browser errors and API failures; automate renewal.
- **Wildcard limitations** — wildcards do not cover the apex domain or multiple levels; SANs are required for those .
- **Certificate store access** — on Windows, Kestrel can load from the certificate store; on Linux, file‑based loading is typical.


### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 443, listenOptions =>
    {
        listenOptions.UseHttps(
            "/etc/ssl/private/server.pfx",
            builder.Configuration["Certificates:Password"]);
    });
});

var app = builder.Build();
app.UseHttpsRedirection();
app.Run();
```

**Step‑by‑step explanation:**

1. Kestrel listens on port 443 on all interfaces.
2. `UseHttps` loads the PFX certificate and its password from configuration (backed by a vault).
3. The app serves HTTPS on 443.

**Expected Output:** Clients connecting to `https://example.com` receive a valid certificate; the password is never in source code.

**Why this output?** The certificate is loaded from a protected file and the password comes from a secret store.

### Real‑World Cases

- **Development** — `dotnet dev-certs https --trust` for localhost.
- **Production behind a reverse proxy** — the proxy holds the CA‑issued certificate; Kestrel binds to loopback without TLS .
- **Multi‑tenant SaaS** — SAN or wildcard certificates covering tenant subdomains.

### References

- Configure endpoints for Kestrel (certificate configuration) – https://learn.microsoft.com/ka-ge/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-9.0#2
- Enforce HTTPS in ASP.NET Core (development certificate) – https://learn.microsoft.com/lb-lu/aspnet/core/security/enforcing-ssl?view=aspnetcore-11.0#4
- secureconfig/dotnet.md (Kestrel TLS and certificate guidance) – https://github.com/jposluns/secureconfig/blob/main/dotnet.md#1


## 4. HSTS (Strict‑Transport‑Security)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | HSTS (HTTP Strict Transport Security) is a response header that instructs browsers to only access the site over HTTPS for a specified period, preventing protocol downgrade and cookie hijacking attacks. |
| **Technical Definition** | ASP.NET Core implements HSTS via **`UseHsts()`**, which adds the `Strict-Transport-Security` header with configurable `max-age` (default 30 days), `includeSubDomains`, and `preload` directives . HSTS is **opt‑in** and **highly cacheable**—it must not be enabled in Development, and initial production `max-age` should be small (hours to one day) until HTTPS is proven stable . The middleware excludes loopback hosts by default (`localhost`, `127.0.0.1`, `[::1]`) . |
| **Beginner‑Friendly Explanation** | HSTS is the **“remember to always use HTTPS”** instruction. The first time a browser visits over HTTPS, the server says “for the next N days, never use HTTP for this site.” The browser remembers and refuses HTTP—even if the user types `http://` or an attacker tries to downgrade. Because it is hard to undo, you start with a short `max-age` and increase it once you are confident HTTPS is stable. |


### Purposes

- **To** **prevent protocol downgrade attacks** — attackers cannot force the browser back to HTTP.
- **To** **prevent cookie hijacking** — HSTS ensures cookies are only sent over HTTPS.
- **To** **protect the first request after the policy is set** — the browser skips the HTTP request entirely.
- **To** **enable preload lists** — browsers can ship with HSTS preloaded for the domain.
- **To** **cover subdomains** — `includeSubDomains` applies the policy to all subdomains .


### Syntax Rules and Structure

#### General Syntax (Basic HSTS)

```csharp
builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubDomains = true;
    options.Preload = true;
});

var app = builder.Build();
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
```

**Component breakdown:**

| Option | Description |
|--------|-------------|
| `MaxAge` | How long the browser remembers the policy (default 30 days). |
| `IncludeSubDomains` | Applies the policy to all subdomains. |
| `Preload` | Requests inclusion in browser preload lists. |
| `ExcludedHosts` | Hosts excluded from HSTS (loopback excluded by default). |

#### General Syntax (Initial Rollout with Short MaxAge)

```csharp
builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromHours(12);   // Start small
});
```

**Syntax rules:**

- **`UseHsts()` must only run outside Development** — HSTS is cached and difficult to undo .
- **Start with a short `max-age`** — hours to one day — then increase to 6–12 months once HTTPS is stable .
- **`includeSubDomains` requires every subdomain to support HTTPS** — otherwise users of HTTP subdomains will be locked out .
- **`preload` requires submission to `hstspreload.org`** and a long `max-age` .

**Constraints and limitations:**

- **HSTS is not a replacement for HTTPS** — it only works after the first successful HTTPS request.
- **Reversing HSTS is slow** — browsers cache the policy for `max-age`; use a short initial value.
- **The `preload` directive is not part of the RFC** — it is a browser‑supported extension .


### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(60);
    options.IncludeSubDomains = true;
    options.ExcludedHosts.Add("example.com");
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
app.UseHttpsRedirection();
app.Run();
```

**Step‑by‑step explanation:**

1. HSTS is configured with a 60‑day `max-age` and subdomain coverage.
2. `example.com` is explicitly excluded (loopback hosts are excluded automatically).
3. HSTS is enabled only outside Development.
4. HTTPS redirection runs after HSTS.

**Expected Output:** Production HTTPS responses include `Strict-Transport-Security: max-age=5184000; includeSubDomains`. Browsers will refuse HTTP for 60 days.

**Why this output?** The middleware adds the header on HTTPS responses; the browser caches and enforces it.

### Real‑World Cases

- **Public web apps** — HSTS with `max-age` of 6–12 months and `includeSubDomains`.
- **Initial HTTPS migration** — short `max-age` (hours) until the infrastructure is proven.
- **Preload submission** — long `max-age` (2 years) with `includeSubDomains` and `preload` .

### References

- Enforce HTTPS in ASP.NET Core (HSTS configuration) – https://learn.microsoft.com/lb-lu/aspnet/core/security/enforcing-ssl?view=aspnetcore-11.0#4
- OWASP HSTS guidance – https://learn.microsoft.com/lb-lu/aspnet/core/security/enforcing-ssl?view=aspnetcore-11.0#4
- ASP.NET Core Security Checklist (HSTS and redirection) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist


## 5. HTTPS Best Practices (Production Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | HTTPS best practices are the guidelines for deploying ASP.NET Core applications with transport security that is secure, reliable, and operationally maintainable in production. |
| **Technical Definition** | Best practices span **TLS termination location** (Kestrel direct vs. reverse proxy), **forwarded headers** for scheme detection, **certificate management** (CA‑issued, SANs, rotation), **protocol version policy** (TLS 1.2+), **HSTS rollout strategy** (short to long `max-age`), **redirection configuration**, **secret management** for certificate passwords, and **monitoring** for certificate expiry and TLS errors. |
| **Beginner‑Friendly Explanation** | These are the **“do not ship without these”** rules for HTTPS. The biggest mistakes are: terminating TLS at a proxy but forgetting forwarded headers (redirect loops), using self‑signed certificates in production (browser warnings), enabling HSTS with a long `max-age` before HTTPS is stable (locking users out), and hard‑coding certificate passwords in configuration files. |


### Purposes

- **To** choose the **right TLS termination architecture** for the deployment.
- **To** ensure the app **correctly detects HTTPS** behind a proxy.
- **To** manage **certificate lifecycle** — issuance, renewal, and rotation.
- **To** enforce **modern TLS versions** and disable deprecated protocols.
- **To** roll out HSTS **safely** without locking users out.
- **To** keep **certificate passwords out of source code**.


### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **TLS termination** | Terminate TLS at a reverse proxy (Caddy, Nginx, Cloudflare) and bind Kestrel to loopback . | Expose Kestrel directly to the internet without TLS. |
| **Forwarded headers** | `UseForwardedHeaders()` **before** `UseHttpsRedirection()` when behind a proxy . | Forget forwarded headers — redirect loops occur. |
| **Protocol versions** | Use `SslProtocols.None` (OS default) or explicitly `Tls12 | Tls13` . | Enable SSL 3.0, TLS 1.0, or TLS 1.1. |
| **Certificates** | Use CA‑issued certificates with SANs covering all hostnames; automate renewal . | Use self‑signed certificates in production. |
| **HSTS** | Start with a short `max-age`; increase after HTTPS is stable; include subdomains only when they all support HTTPS . | Enable HSTS with a long `max-age` on day one. |
| **Redirection** | Configure `HttpsPort` explicitly; use `UseHttpsRedirection()` outside Development . | Leave the HTTPS port unconfigured — redirection fails. |
| **Secrets** | Load certificate passwords from environment variables or a vault . | Commit certificate passwords to `appsettings.json`. |
| **Monitoring** | Monitor certificate expiry and TLS handshake errors. | Discover an expired certificate from user complaints. |


### Annotated Code Example (Best‑Practice Production Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Kestrel bound to loopback; TLS terminated at the reverse proxy
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Loopback, 5000);
});

builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubDomains = true;
});

var app = builder.Build();

app.UseForwardedHeaders();   // Restores scheme from proxy
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Why this follows best practices:**

- **Kestrel is bound to loopback** — the reverse proxy handles TLS and public exposure.
- **Forwarded headers run first** — the app sees the correct scheme and does not redirect in a loop.
- **HSTS is gated to non‑Development** with a one‑year `max-age` and subdomain coverage.
- **HTTPS redirection is enabled** for any HTTP requests that reach the app.

### Real‑World Cases

- **Azure App Service** — TLS terminated at the App Service front end; forwarded headers restore the scheme .
- **Nginx / Caddy** — the proxy holds the certificate and forwards HTTP to Kestrel on loopback .
- **Cloudflare** — TLS terminated at the edge; `CF-Visitor` header and forwarded headers restore the scheme.

### References

- Configure ASP.NET Core to work with proxy servers and load balancers – https://learn.microsoft.com/vi-vn/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-10.0&viewFallbackFrom=aspnetcore-1.0#2
- ASP.NET Core Security Checklist (2026 Production Guide) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- secureconfig/dotnet.md (preferred production layout) – https://github.com/jposluns/secureconfig/blob/main/dotnet.md#1
- Enforce HTTPS in ASP.NET Core (HSTS and redirection) – https://learn.microsoft.com/lb-lu/aspnet/core/security/enforcing-ssl?view=aspnetcore-11.0#4


## Summary Table: HTTPS at a Glance

| Concept | Primary Role | Key API / Configuration | When to Use |
|---------|--------------|-------------------------|-------------|
| **HTTPS** | Encrypted transport | `UseHttpsRedirection()`, `UseHsts()` | Every production app |
| **TLS** | Encryption protocol | `SslProtocols.Tls12 | Tls13`, `SslProtocols.None` | Every HTTPS endpoint |
| **Certificates** | Server identity | `UseHttps("cert.pfx", password)`, `dotnet dev-certs` | Development and production |
| **HSTS** | Browser HTTPS enforcement | `AddHsts(options => ...)`, `UseHsts()` | After HTTPS is stable |
| **Best practices** | Production deployment | Forwarded headers, reverse proxy, secret management | Every deployment |


## References (Full List)

- Enforce HTTPS in ASP.NET Core – https://learn.microsoft.com/lb-lu/aspnet/core/security/enforcing-ssl?view=aspnetcore-11.0#4
- Configure endpoints for Kestrel web server – https://learn.microsoft.com/ka-ge/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-9.0#2
- Configure ASP.NET Core to work with proxy servers and load balancers – https://learn.microsoft.com/vi-vn/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-10.0&viewFallbackFrom=aspnetcore-1.0#2
- Kestrel security considerations (TLS 1.2, TLS 1.3, SNI, mTLS) – https://learn.microsoft.com/…/kestrel/security
- Kestrel: Default supported TLS protocol versions changed – https://github.com/dotnet/aspnetcore/…
- secureconfig/dotnet.md (Kestrel TLS, certificates, production layout) – https://github.com/jposluns/secureconfig/blob/main/dotnet.md#1
- ASP.NET Core Security Checklist (2026 Production Guide) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- OWASP HSTS guidance (referenced in Microsoft docs) – https://learn.microsoft.com/lb-lu/aspnet/core/security/enforcing-ssl?view=aspnetcore-11.0#4
- HSTS preload list – https://hstspreload.org/