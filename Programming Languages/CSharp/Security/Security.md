# Security Fundamentals in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Security in C# is the discipline of designing, building, and operating .NET applications so that they resist adversarial attacks, protect sensitive data, and enforce correct access to resources across every phase of the software development lifecycle.

**Technical Definition**
Software security in the .NET ecosystem is a multi-layered engineering concern that spans **secure design** (attack surface reduction, defense in depth, least privilege, secure defaults, threat modeling), **secure coding** (input validation, safe APIs, cryptography, deserialization safety, memory safety in unsafe code), **security testing** (SAST, DAST, dependency auditing, penetration testing), and **secure operations** (secrets management, HTTPS enforcement, CI/CD hardening, runtime monitoring). Microsoft's Security Development Lifecycle (SDL) formalises these concerns as mandatory training, threat modeling, secure coding standards, and security testing gates. The .NET platform provides strong primitives — the managed runtime eliminates whole classes of memory-safety bugs, `System.Security.Cryptography` provides vetted cryptographic algorithms, ASP.NET Core provides authentication and authorization middleware, and modern deserialisers like `System.Text.Json` avoid type-resolution vulnerabilities — but the developer is responsible for using them correctly. Most .NET breaches do not come from CLR zero-days; they come from unaudited NuGet dependencies, insecure deserialisation, secrets committed to source control, and debug settings left on in production.

**Beginner-Friendly Explanation**
Security is "keeping bad things from happening to your application and its users." In C#, that means: validating everything that comes from the outside, never trusting user input, running with the fewest permissions possible, keeping secrets out of your code, using safe libraries instead of writing your own crypto, and thinking like an attacker before you ship. The .NET platform gives you good tools, but the defaults are tuned for developer convenience, not adversarial production traffic — so you have to actively harden them.

---

### Key Characteristics

- **Lifecycle-wide** — Security must be considered during design, implementation, testing, deployment, and operations, not bolted on at the end.
- **Adversarial mindset** — Security engineering requires thinking like an attacker, not just like a developer.
- **Layered** — No single control is sufficient; defense in depth places independent controls at every layer.
- **Principle-driven** — Least privilege, secure defaults, fail-safe, and don't-trust-input are durable principles that outlive specific technologies.
- **Threat-model-driven** — Systematic threat identification (STRIDE) converts vague "security" into concrete, prioritised risks and mitigations.
- **Dependency-sensitive** — The NuGet dependency tree is often the largest attack surface in a modern .NET application.
- **Measurable** — Analyzers, NuGet audit, SAST tools, and penetration testing provide objective evidence of security posture.

---

### Prerequisites

- Working knowledge of C#, ASP.NET Core, and the .NET CLI.
- Familiarity with HTTP, authentication, and authorization concepts.
- Basic understanding of cryptography (hashing, encryption, signing) and why not to roll your own.
- Awareness of SQL, XSS, CSRF, and insecure deserialisation as common vulnerability classes.
- Understanding of dependency management (NuGet, lock files, SBOMs).
- Familiarity with Git and CI/CD pipelines.

---

### Related Programming Areas

- **Threat modeling** — Structured adversarial analysis of system design.
- **Attack surface reduction** — Minimising exposed entry points and data.
- **Defense in depth** — Layered, independent security controls.
- **Least privilege** — Running code and users with minimal permissions.
- **Secure defaults** — Choosing the safest configuration out of the box.
- **Secure coding** — Input validation, safe APIs, cryptography, deserialisation.
- **Supply chain security** — NuGet auditing, SBOMs, package provenance.
- **Security testing** — SAST, DAST, dependency scanning, penetration testing.

---

### Core Concepts / Features

1. Security (overarching discipline)
2. Threat Modeling
3. Attack Surface
4. Defense in Depth
5. Least Privilege
6. Secure Defaults
7. Security Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with "To"), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Security (Overarching Discipline)

### Definitions

**Core Definition**
Security is the practice of protecting an application and its data from unauthorised access, modification, disclosure, and disruption across the entire software lifecycle.

**Technical Definition**
Security in C# and .NET is the composition of secure design principles, secure coding practices, security testing, and secure deployment and operations. Microsoft's SDL defines security training, threat modeling, secure coding standards, security testing, and response processes as mandatory activities. The .NET platform provides memory safety through the managed runtime, vetted cryptography through `System.Security.Cryptography`, authentication and authorization middleware in ASP.NET Core, data protection APIs for encryption, and modern serialisation with `System.Text.Json` that avoids type-resolution vulnerabilities. Security is not a product or a library; it is an emergent property of disciplined engineering across the lifecycle.

**Beginner-Friendly Explanation**
Security is "making sure your app cannot be tricked into doing something harmful." In C#, that means validating input, keeping secrets secret, using safe libraries, enforcing authentication and authorization, and staying current on dependency vulnerabilities. The platform helps, but you are responsible for using it correctly.

---

### Purposes

- To protect sensitive data from unauthorised disclosure.
- To prevent unauthorised modification of data and code.
- To ensure that only authorised users and services can perform privileged operations.
- To maintain availability under adversarial conditions.
- To satisfy regulatory and contractual obligations.
- To reduce the blast radius of a compromise.
- To build user trust and protect organisational reputation.

---

### Syntax Rules and Structure

#### General Security Lifecycle

```
1. Training and awareness (SDL requirement).
2. Secure design: attack surface reduction, defense in depth, least privilege, secure defaults.
3. Threat modeling: identify, rank, and mitigate threats.
4. Secure coding: input validation, safe APIs, cryptography, deserialisation safety.
5. Security testing: SAST, DAST, dependency audit, penetration testing.
6. Secure deployment: secrets management, HTTPS, hardening, least-privilege identities.
7. Monitoring and response: logging, alerting, incident response.
```

**Component breakdown**

- **Design** — Principles applied before code exists.
- **Threat model** — Structured adversarial analysis.
- **Coding** — Safe APIs and defensive programming.
- **Testing** — Automated and manual verification.
- **Deployment** — Configuration hardening and secret management.
- **Operations** — Monitoring and response.

#### Syntax Rules

- Security must be considered at every phase; it cannot be retrofitted.
- Use platform-provided security primitives; do not roll your own crypto.
- Treat all external input as untrusted until validated.
- Apply least privilege to users, services, and data access.
- Keep dependencies audited and up to date.

#### Constraints and Limitations

- Security is never "done"; new threats and vulnerabilities emerge continuously.
- No single control is sufficient; layered defenses are required.
- Security controls can conflict with usability and performance; trade-offs must be explicit.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Security-Relevant C# Recommendations

```csharp
// ✅ Use the most restrictive data type
public void Resize(uint newSize) { /* unsigned prevents negative sizes */ }

// ✅ Validate input at the boundary
public IActionResult Create(string email)
{
    if (!Regex.IsMatch(email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$"))
        return BadRequest("Invalid email");
    // ...
}

// ✅ Use platform cryptography, not custom algorithms
using var aes = Aes.Create();

// ✅ Never hardcode secrets
var connectionString = builder.Configuration.GetConnectionString("Default");
```

**Expected Output (conceptual)**

```
Restrictive types, boundary validation, vetted cryptography, and externalised secrets
reduce the attack surface and eliminate common vulnerability classes.
```

**Why this output occurs**

- Unsigned types prevent negative-size attacks.
- Boundary validation blocks malformed input before it reaches business logic.
- Platform crypto avoids implementation errors.
- Externalised secrets keep credentials out of source control.

---

### Real-World Cases

- **Financial services** — Mandatory SDL, threat modeling, and secure coding for PCI-DSS compliance.
- **Healthcare** — HIPAA-driven encryption, access controls, and audit logging.
- **SaaS platforms** — Multi-tenant isolation, least-privilege identities, and supply-chain auditing.
- **Open-source libraries** — NuGet audit, SBOM generation, and signed packages.

---

### References

- Pre-SDL Requirements: Security Training — Microsoft Learn - https://learn.microsoft.com/sv-se/previous-versions/windows/desktop/cc307407(v=msdn.10)
- Security (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/pl-pl/previous-versions/ms173195(v=vs.85)
- Security Principles — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices

---

## 2. Threat Modeling

### Definitions

**Core Definition**
Threat modeling is a structured, repeatable process for identifying potential security threats against a system's design, prioritising risks, and defining mitigations before code is written.

**Technical Definition**
Threat modeling is an adversarial design analysis technique that answers four questions: **What are we building?** (data flow diagram with trust boundaries), **What can go wrong?** (threat identification using a methodology such as STRIDE), **What are we going to do about it?** (mitigations prioritised by risk), and **Did we do a good enough job?** (validation that mitigations are effective and no significant threats were missed). Microsoft's SDL uses the **STRIDE** mnemonic — Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege — to systematically categorise threats per component and data flow. Threat modeling is iterative: architectural changes, new integrations, and feature additions require updated threat models.

**Beginner-Friendly Explanation**
Threat modeling is "thinking like an attacker before you write the code." You draw a diagram of your system, mark where trust changes (the internet vs. your internal network), and ask: "How could someone pretend to be someone else? How could they change my data? How could they deny doing something? How could they see data they shouldn't? How could they take my service down? How could they get more permissions than they should?" For each answer, you decide how to stop it. Doing this early is far cheaper than fixing vulnerabilities after launch.

---

### Purposes

- To identify security risks during the design phase, when mitigation is cheapest.
- To systematically cover all six STRIDE threat categories rather than focusing on familiar ones.
- To prioritise mitigations by risk, aligning security effort with business impact.
- To build security "into" the system rather than "bolting it on" later.
- To create a shared understanding of the system's trust boundaries and data flows.
- To satisfy SDL and compliance requirements for structured security analysis.
- To keep the threat model current as the system evolves.

---

### Syntax Rules and Structure

#### The Four-Step Process

```
1. Application decomposition (system modeling)
   - Data flow diagram (DFD)
   - External entities, processes, data stores, data flows, trust boundaries

2. Threat identification and ranking
   - STRIDE per element
   - Risk ranking (likelihood × impact)

3. Mitigations
   - Define controls for each identified threat
   - Prioritise by risk

4. Review and validation
   - Verify mitigations are effective
   - Confirm no significant threats were missed
```

**Component breakdown**

- **DFD** — Maps what the system is and where data flows.
- **Trust boundaries** — Points where the level of trust changes; controls must exist here.
- **STRIDE** — Six threat categories applied systematically.
- **Risk ranking** — Likelihood × impact matrix.
- **Mitigations** — Specific controls, not vague intentions.
- **Validation** — Confirms the model and mitigations are adequate.

#### STRIDE Categories

| Category | Threat | Example |
|---|---|---|
| **Spoofing** | Pretending to be someone/something else | Forged authentication tokens |
| **Tampering** | Unauthorised modification of data or code | Modifying API payloads in transit |
| **Repudiation** | Denying an action without proof | Missing audit logs for an order |
| **Information disclosure** | Exposing data to unauthorised parties | API returning extra fields |
| **Denial of service** | Making the system unavailable | Flooding a public API |
| **Elevation of privilege** | Gaining higher access than authorised | Exploiting a misconfigured role |

#### Syntax Rules

- Create a DFD before identifying threats; threats without a model are guesswork.
- Apply STRIDE to every component, data store, and data flow — not just the obvious ones.
- Rank threats by risk (likelihood × impact) to prioritise mitigations.
- Revisit the threat model whenever architecture, integrations, or features change.
- Use tools such as the Microsoft Threat Modeling Tool to automate DFD-based STRIDE enumeration.

#### Constraints and Limitations

- Threat modeling is a team activity; a single developer's model may miss systemic risks.
- STRIDE is not exhaustive; other methodologies (PASTA, OCTAVE, LINDDUN) may be appropriate for specific domains.
- Threat models can become stale if not maintained alongside the system.
- Risk ranking is subjective and depends on accurate likelihood and impact estimates.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: STRIDE Analysis for an API Endpoint

```
Component: POST /api/orders
Data flows: Client → API → Database
Trust boundary: Internet → DMZ

STRIDE analysis:
  Spoofing:         Require authenticated JWT; validate issuer and audience.
  Tampering:        Use HTTPS; validate request body against schema; sign critical fields.
  Repudiation:      Write immutable audit log entries for every order.
  Information disclosure: Return only fields the caller is authorised to see.
  Denial of service: Rate-limit per client; enforce request size limits.
  Elevation of privilege: Authorise against the order's owner; do not trust client-supplied role claims.
```

**Expected Output (conceptual)**

```
Six concrete threats identified; six concrete mitigations defined.
Risk-ranked: Elevation of privilege (High), Spoofing (High), Tampering (Medium), ...
```

**Why this output occurs**

- STRIDE systematically covers all six threat categories for the component and its data flows.
- Each threat maps to a specific, implementable control.

---

#### Example 2: Trust Boundary Identification in a DFD

```
[Client] --(HTTPS)--> [API Gateway] --(mTLS)--> [Order Service] --(SQL)--> [Database]

Trust boundaries:
  - Internet ↔ API Gateway (TLS termination, WAF)
  - API Gateway ↔ Order Service (mTLS, authorization)
  - Order Service ↔ Database (least-privilege DB user, TLS)
```

**Expected Output (conceptual)**

```
Three trust boundaries identified; each requires independent security controls.
```

**Why this output occurs**

- Every point where trust changes is a place where controls must exist to prevent unauthorised access.

---

### Real-World Cases

- **E-commerce checkout** — STRIDE analysis of payment flows identifies spoofing, tampering, and repudiation risks.
- **Microservices architectures** — Trust boundaries between services require mTLS and authorization.
- **Cloud migrations** — New trust boundaries between on-premises and cloud require updated threat models.

---

### References

- Evaluate application threats with threat modeling — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling
- Threat Modeling Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/9c87d0a0e9411de028b9d7eb853872082d6d3382/cheatsheets/Threat_Modeling_Cheat_Sheet.md
- Threat Modeling Tool — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling
- Security Principles (threat modeling) — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles

---

## 3. Attack Surface

### Definitions

**Core Definition**
Attack surface is the sum of all points where an attacker can interact with, input data into, or extract data from a system.

**Technical Definition**
Attack surface encompasses every exposed interface, input channel, protocol, service, dependency, and data store that an adversary can reach. In .NET applications, this includes HTTP endpoints, gRPC services, message queue consumers, file upload handlers, deserialisation entry points, database connections, third-party NuGet dependencies, and configuration surfaces. Attack surface reduction is the practice of eliminating, minimising, or hardening these entry points so that fewer paths exist for exploitation. It is a core secure design principle in Microsoft's SDL, alongside defense in depth, least privilege, and secure defaults.

**Beginner-Friendly Explanation**
Attack surface is "all the doors and windows an attacker can try to open." Every endpoint you expose, every library you depend on, and every input you accept is a potential door. Attack surface reduction means closing the doors you do not need, locking the ones you do, and checking who comes through.

---

### Purposes

- To minimise the number of exploitable entry points.
- To reduce the amount of code that must be secured and tested.
- To limit the blast radius of a compromise.
- To simplify security review and auditing.
- To eliminate unnecessary features that create hidden vulnerabilities.
- To improve performance by removing unused services and dependencies.

---

### Syntax Rules and Structure

#### General Reduction Techniques

```
1. Remove unused endpoints, services, and features.
2. Disable unused protocols and ports.
3. Validate and constrain all input at the boundary.
4. Use the most restrictive data types and schemas.
5. Minimise dependencies; audit transitive NuGet packages.
6. Avoid exposing detailed error information.
7. Apply rate limiting and request size limits.
8. Use a reverse proxy / WAF in front of application servers.
```

**Component breakdown**

- **Endpoints** — Every exposed API route is a door.
- **Protocols** — Unused protocols (e.g., legacy SOAP, unencrypted HTTP) increase surface.
- **Input validation** — Constrains what can come through the door.
- **Dependencies** — Each NuGet package is code you did not write but must trust.
- **Error messages** — Detailed errors can reveal internal structure to attackers.
- **Rate limiting** — Reduces denial-of-service exposure.

#### Syntax Rules (C# / ASP.NET Core)

- Remove or disable unused controllers, minimal API endpoints, and middleware.
- Use `[Authorize]` by default and `[AllowAnonymous]` sparingly.
- Disable detailed exception pages in production (`UseExceptionHandler` instead of `UseDeveloperExceptionPage`).
- Enforce HTTPS and HSTS.
- Use request size limits and rate limiting middleware.
- Audit NuGet dependencies with `dotnet list package --vulnerable`.

#### Constraints and Limitations

- Reducing attack surface may remove functionality that users need; trade-offs must be explicit.
- Dependencies are necessary but must be audited and updated.
- Some attack surface is inherent to the application's purpose.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Reducing ASP.NET Core Attack Surface

```csharp
// ✅ Remove unused middleware and endpoints
var builder = WebApplication.CreateSlimBuilder(args); // fewer defaults

// ✅ Authorize by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// ✅ Disable developer exception page in production
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

// ✅ Enforce HTTPS
app.UseHttpsRedirection();

// ✅ Rate limiting
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", o =>
    {
        o.PermitLimit = 100;
        o.Window = TimeSpan.FromMinutes(1);
    });
});
```

**Expected Output (conceptual)**

```
Fewer exposed defaults, authorization required by default,
detailed errors suppressed, HTTPS enforced, rate limiting active.
```

**Why this output occurs**

- `CreateSlimBuilder` reduces the default feature surface.
- Fallback authorization policy closes anonymous access by default.
- Production exception handling hides internal details.
- HTTPS and rate limiting harden the exposed surface.

---

#### Example 2: Dependency Surface Reduction

```bash
# Audit dependencies for known vulnerabilities
dotnet list package --vulnerable --include-transitive

# Remove unused packages
dotnet remove package Unused.Package
```

**Expected Output (conceptual)**

```
Vulnerable transitive dependency identified and removed or updated.
```

**Why this output occurs**

- The dependency tree is part of the attack surface; vulnerable packages are exploitable entry points.

---

### Real-World Cases

- **API gateways** — Expose only required routes; block everything else.
- **Container images** — Use minimal base images (e.g., Chainguard) to reduce the package attack surface.
- **Legacy applications** — Disable unused SOAP endpoints and unencrypted protocols.
- **Serverless functions** — Each function is a separate surface; minimise permissions and inputs.

---

### References

- Security Principles (reduce surface area) — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles
- Pre-SDL Requirements: Attack surface reduction — Microsoft Learn - https://learn.microsoft.com/sv-se/previous-versions/windows/desktop/cc307407(v=msdn.10)
- Security And Identity (least privilege and authorization) — autoskills - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md

---

## 4. Defense in Depth

### Definitions

**Core Definition**
Defense in depth is a security strategy that places multiple independent layers of controls between an attacker and the assets being protected.

**Technical Definition**
Defense in depth assumes that any single control can fail and therefore places overlapping, independent controls at every layer — physical, identity and access, network, compute, application, and data. Microsoft recommends a layered approach across seven layers in its Azure security architecture. In application security, defense in depth means that authentication, authorization, input validation, output encoding, encryption, logging, and monitoring are all implemented independently, so that a bypass at one layer does not grant unrestricted access to the next. The principle is explicitly listed as a core secure design concept in Microsoft's SDL.

**Beginner-Friendly Explanation**
Defense in depth is "not putting all your security eggs in one basket." If your login page is the only thing stopping an attacker, one bypass means full access. Defense in depth adds more locks: authorization checks on every endpoint, input validation, output encoding, encryption, logging, and monitoring. If one lock fails, the others still hold.

---

### Purposes

- To ensure that failure of a single control does not lead to full compromise.
- To provide overlapping coverage against different attack vectors.
- To slow down and detect attackers at multiple points.
- To satisfy defense-in-depth requirements in compliance frameworks.
- To create independent verification of security decisions.
- To increase the cost and complexity of a successful attack.

---

### Syntax Rules and Structure

#### Layered Controls in a .NET Web Application

```
Layer 1: Network       — TLS, reverse proxy, WAF, rate limiting
Layer 2: Identity      — Authentication (OIDC, JWT, cookies)
Layer 3: Authorization  — Policies, roles, resource-based checks
Layer 4: Input         — Validation, schema enforcement, size limits
Layer 5: Application   — Safe APIs, deserialisation safety, error handling
Layer 6: Data          — Encryption at rest and in transit, least-privilege DB users
Layer 7: Monitoring    — Audit logs, anomaly detection, alerting
```

**Component breakdown**

- **Network** — Stops volumetric and protocol-level attacks.
- **Identity** — Confirms who the caller is.
- **Authorization** — Confirms what the caller can do.
- **Input** — Rejects malformed or malicious data.
- **Application** — Uses safe APIs and fails securely.
- **Data** — Protects data even if other layers are bypassed.
- **Monitoring** — Detects and responds to attacks that succeed.

#### Syntax Rules

- Each layer should be independently sufficient to stop the threats it is responsible for.
- Do not rely on a single control for multiple purposes.
- Validate at the boundary and re-validate at trust boundaries.
- Log security-relevant events at every layer.
- Fail securely: on error, deny access rather than grant it.

#### Constraints and Limitations

- More layers add operational complexity and latency.
- Defense in depth is not a substitute for fixing root causes.
- Layers must be independently maintained; a forgotten layer becomes a false sense of security.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Layered ASP.NET Core Controls

```csharp
// Layer 2: Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* validate issuer, audience, signature */ });

// Layer 3: Authorization
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanEditOrders", policy =>
        policy.RequireClaim("permission", "orders:write"));
});

// Layer 4: Input validation
app.MapPost("/orders", (CreateOrderDto dto) =>
{
    if (!Validator.TryValidateObject(dto, new ValidationContext(dto), null, true))
        return Results.BadRequest();
    // ...
}).RequireAuthorization("CanEditOrders");

// Layer 7: Audit logging
app.Use(async (context, next) =>
{
    await next();
    logger.LogInformation("Request {Path} by {User}", context.Request.Path,
        context.User.Identity?.Name);
});
```

**Expected Output (conceptual)**

```
Unauthenticated request: 401.
Authenticated but unauthorized: 403.
Malformed payload: 400.
Authorized valid request: 200, with audit log entry.
```

**Why this output occurs**

- Each layer independently enforces its responsibility.
- A failure at one layer does not bypass the others.

---

### Real-World Cases

- **Banking applications** — Network segmentation, MFA, authorization, encryption, and fraud monitoring.
- **Healthcare systems** — Access controls, audit trails, encryption, and anomaly detection.
- **Cloud-native microservices** — mTLS between services, authorization at the gateway and service, and centralized logging.

---

### References

- Security Principles (defense in depth) — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles
- Describe defense in depth — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/describe-defense-in-depth/
- Pre-SDL Requirements: Defense in depth — Microsoft Learn - https://learn.microsoft.com/sv-se/previous-versions/windows/desktop/cc307407(v=msdn.10)

---

## 5. Least Privilege

### Definitions

**Core Definition**
The principle of least privilege states that every user, process, service, and component should operate with only the minimum permissions necessary to perform its function, and no more.

**Technical Definition**
Least privilege in .NET applies at multiple levels: **user accounts** should have only the roles and claims required for their tasks; **application pool identities** should run as low-privilege virtual accounts; **database users** should have only the CRUD permissions needed on specific objects; **managed identities** and service principals should have narrowly scoped RBAC roles; and **code** should request only the permissions it actually uses. Microsoft's SDL lists least privilege as a core secure design principle, and IIS guidance explicitly recommends running worker processes as low-privilege identities unique per site to limit the damage of a compromise. The principle is also enforced through ASP.NET Core's `[Authorize]` attribute and policy-based authorization, which should be applied by default.

**Beginner-Friendly Explanation**
Least privilege means "give everyone and everything the smallest key that still opens the doors they need." Your web app should not run as an administrator. Your database user should not be able to drop tables if it only needs to read and write rows. Your users should not be admins unless their job requires it. If an attacker compromises a low-privilege component, they can do far less damage than if it were running as root.

---

### Purposes

- To limit the damage an attacker can do after compromising a component.
- To reduce the blast radius of a vulnerability.
- To satisfy compliance requirements (PCI-DSS, HIPAA, SOC 2).
- To prevent accidental misuse of privileges.
- To simplify auditing and access reviews.
- To enforce separation of duties.

---

### Syntax Rules and Structure

#### General Least-Privilege Checklist

```
✅ Run application pools under low-privilege virtual accounts.
✅ Use a dedicated database user with only required CRUD permissions.
✅ Apply [Authorize] by default; use [AllowAnonymous] explicitly and sparingly.
✅ Scope managed identities and service principals to specific resources.
✅ Avoid running containers as root; use non-root users.
✅ Do not grant `db_owner` or `sysadmin` to application accounts.
✅ Use resource-based authorization for per-object access.
```

**Component breakdown**

- **Application pool identity** — The OS account under which the app runs.
- **Database user** — The account used for data access.
- **Authorization policy** — The application-level permission model.
- **Managed identity / service principal** — Cloud resource access identity.
- **Container user** — The UID inside the container.

#### Syntax Rules (C# / ASP.NET Core)

```csharp
// ✅ Authorize by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// ✅ Apply [Authorize] to controllers/endpoints
[Authorize(Policy = "CanEditOrders")]
public IActionResult Edit(int id) { ... }

// ✅ Use [AllowAnonymous] only where truly needed
[AllowAnonymous]
public IActionResult Health() => Ok();

// ✅ Resource-based authorization
var result = await _authorizationService.AuthorizeAsync(
    User, order, "OrderOwnerPolicy");
if (!result.Succeeded) return Forbid();
```

#### Constraints and Limitations

- Least privilege can increase administrative overhead; automation and policy-as-code help.
- Overly restrictive permissions can break functionality; permissions must be tested.
- Privilege creep occurs over time; regular access reviews are required.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Default Authorization with Explicit Anonymous Exceptions

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

var app = builder.Build();

app.MapGet("/orders", [Authorize] () => GetOrders());
app.MapGet("/health", [AllowAnonymous] () => "OK");

app.Run();
```

**Expected Output (conceptual)**

```
GET /orders without authentication: 401.
GET /health without authentication: 200.
GET /orders with valid token: 200.
```

**Why this output occurs**

- The fallback policy requires authentication for all endpoints by default.
- `[AllowAnonymous]` explicitly opts the health endpoint out.
- Least privilege is enforced at the authorization layer.

---

#### Example 2: Low-Privilege Database User

```sql
-- ✅ Least-privilege application user
CREATE USER app_user WITH PASSWORD '...';
GRANT SELECT, INSERT, UPDATE ON Orders TO app_user;
GRANT SELECT ON Products TO app_user;
-- No DELETE, no DDL, no db_owner
```

**Expected Output (conceptual)**

```
The application can read and write orders and read products.
It cannot delete orders or alter the schema.
```

**Why this output occurs**

- The database user has only the permissions required for the application's function.

---

### Real-World Cases

- **Web applications** — Application pool identity running as `ApplicationPoolIdentity` rather than `NetworkService` or `LocalSystem`.
- **Cloud services** — Managed identities scoped to a single Key Vault or Storage Account.
- **Containers** — Non-root user with read-only filesystem where possible.
- **Multi-tenant SaaS** — Per-tenant database users or row-level security.

---

### References

- Security Principles (least privilege) — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles
- Planning Step 4: Plan Application Security (least privilege) — Microsoft Learn - https://learn.microsoft.com/en-us/iis/application-frameworks/scenario-build-an-aspnet-website-on-iis/planning-step-4-plan-application-security
- Security And Identity (authorization by default) — autoskills - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md

---

## 6. Secure Defaults

### Definitions

**Core Definition**
Secure defaults are the configuration and behavioral choices that are safest out of the box, requiring deliberate, informed action to weaken them.

**Technical Definition**
Secure defaults in .NET and ASP.NET Core mean that the platform and the application choose the most secure option when a developer does not explicitly configure otherwise. This includes HTTPS enforcement, HSTS, secure cookie flags (`HttpOnly`, `Secure`, `SameSite`), antiforgery protection for cookie-based apps, `[Authorize]` by default (or a fallback policy), safe deserialisation (`System.Text.Json` without type-name handling), validated JWT issuer/audience/signature, and secrets kept out of source code. Microsoft's SDL lists secure defaults as a core secure design principle: "If your application demands features that force you to reduce or change default security settings, test the effects and understand the implications before making the change." .NET 8+ enables NuGet Audit by default, and modern templates enable HTTPS redirection and HSTS outside development.

**Beginner-Friendly Explanation**
Secure defaults mean "the safest setting is the one you get if you don't change anything." If the default is insecure, developers who don't read the docs will ship vulnerabilities. Secure defaults flip that: you have to actively opt into less secure behaviour, which forces a conscious decision. For example, ASP.NET Core templates enable HTTPS redirection by default; if you disable it, you are making an explicit choice.

---

### Purposes

- To protect developers who do not configure security explicitly.
- To make the safe choice the path of least resistance.
- To reduce the prevalence of common misconfigurations.
- To force conscious decisions when weakening security.
- To align with compliance and framework requirements.
- To provide a baseline that security reviews can verify.

---

### Syntax Rules and Structure

#### Secure Defaults Checklist

```
✅ HTTPS redirection and HSTS enabled outside development.
✅ Antiforgery protection enabled for cookie-based apps and form posts.
✅ [Authorize] by default or a fallback authorization policy.
✅ Secure cookie flags: HttpOnly, Secure, SameSite.
✅ JWT validation: issuer, audience, signature, lifetime.
✅ System.Text.Json without TypeNameHandling.
✅ NuGet Audit enabled (default in .NET 8+; transitive in .NET 9+).
✅ No secrets in source code or appsettings.json.
✅ Detailed exception pages disabled in production.
```

**Component breakdown**

- **HTTPS/HSTS** — Encrypts traffic and prevents downgrade.
- **Antiforgery** — Prevents CSRF on cookie-based apps.
- **Authorization** — Denies by default.
- **Cookies** — Prevents client-side script access and cross-site sending.
- **JWT validation** — Prevents token forgery and replay.
- **Deserialisation** — Avoids type-resolution RCE.
- **NuGet Audit** — Catches vulnerable dependencies.
- **Secrets** — Keeps credentials out of Git.
- **Error handling** — Avoids information disclosure.

#### Syntax Rules

- Do not disable secure defaults without understanding the implications and documenting the decision.
- Use `CreateSlimBuilder` for Native AOT and minimal surface area.
- Configure `FallbackPolicy` to require authentication by default.
- Enable `UseHttpsRedirection()` and `UseHsts()` outside development.
- Use `AddAntiforgery()` and validate tokens on state-changing requests.
- Keep `TypeNameHandling` off in JSON serialisation.

#### Constraints and Limitations

- Secure defaults can break compatibility with legacy clients or workflows.
- Some defaults are environment-specific (e.g., developer exception page in Development only).
- Secure defaults are a baseline, not a complete security strategy.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Secure ASP.NET Core Defaults

```csharp
var builder = WebApplication.CreateBuilder(args);

// Secure defaults
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

builder.Services.AddAntiforgery();
builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubDomains = true;
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
```

**Expected Output (conceptual)**

```
HTTP requests redirected to HTTPS.
HSTS header sent outside development.
Detailed errors suppressed in production.
Authentication required by default.
```

**Why this output occurs**

- Each secure default is enabled explicitly (or is the template default).
- Weakening any of them requires a deliberate code change.

---

#### Example 2: Safe Deserialisation Default

```csharp
// ✅ System.Text.Json does not resolve arbitrary types by name
var order = JsonSerializer.Deserialize<Order>(json);

// ❌ Newtonsoft.Json with TypeNameHandling.All can instantiate arbitrary types
// var settings = new JsonSerializerSettings { TypeNameHandling = TypeNameHandling.All };
```

**Expected Output (conceptual)**

```
System.Text.Json: deserialises only the specified type; no type-resolution RCE.
Newtonsoft with TypeNameHandling.All: attacker-controlled type names may execute code.
```

**Why this output occurs**

- `System.Text.Json` uses compile-time type information, not attacker-supplied type names.

---

### Real-World Cases

- **ASP.NET Core templates** — HTTPS redirection and HSTS enabled by default.
- **.NET 8+ NuGet Audit** — Vulnerability warnings during `dotnet restore`.
- **JWT authentication** — Issuer, audience, and signature validation are required by default.
- **Cookie authentication** — `HttpOnly`, `Secure`, and `SameSite` flags set by default.

---

### References

- Security Principles (secure defaults) — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles
- Security And Identity (security defaults) — autoskills - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices

---

## 7. Security Best Practices

### Definitions

**Core Definition**
Security best practices are empirically grounded guidelines for designing, coding, configuring, and operating .NET applications to minimise vulnerability to attack.

**Technical Definition**
Best practices in .NET security span **input handling** (validate at boundaries, use parameterised queries, encode output), **deserialisation** (avoid `BinaryFormatter`, avoid `TypeNameHandling` in Newtonsoft.Json, prefer `System.Text.Json`), **secrets management** (keep secrets out of source, use Secret Manager in development and a managed vault in production), **cryptography** (use `System.Security.Cryptography`, never roll your own), **dependency management** (NuGet Audit, SBOMs, package source mapping), **authentication and authorization** (secure flows, `[Authorize]` by default, MFA), **transport security** (HTTPS, HSTS, forwarded headers), **error handling** (no detailed errors in production, fail securely), **container hardening** (non-root, minimal images, `allowPrivilegeEscalation: false`), and **CI/CD security** (SAST, `TreatWarningsAsErrors`, `dotnet list package --vulnerable`).

**Beginner-Friendly Explanation**
Security best practices are "the checklist that keeps your app safe." They include: validate everything from the outside, never trust user input, keep secrets out of Git, use safe deserialisers, audit your NuGet packages, require authentication by default, use HTTPS, and don't show detailed errors to users. Following these prevents the most common .NET vulnerabilities.

---

### Purposes

- To prevent injection, XSS, CSRF, deserialisation RCE, and other common vulnerability classes.
- To keep secrets out of source control and logs.
- To ensure cryptographic operations use vetted algorithms and APIs.
- To detect and remediate vulnerable dependencies.
- To enforce authentication and authorization consistently.
- To harden containers and deployment configurations.
- To integrate security into CI/CD pipelines.

---

### Guidelines and Structure

#### Input and Output

```
✅ Validate input at the boundary (length, range, format, type).
✅ Use parameterised queries or an ORM; never concatenate user input into SQL.
✅ Encode output for the correct context (HTML, JS, URL, CSS).
✅ Use the most restrictive data types.
❌ Do not trust filenames; do not make security decisions based on them.
❌ Do not display detailed exception information in production.
```

#### Deserialisation

```
✅ Prefer System.Text.Json for untrusted data.
✅ Use [JsonDerivedType] with an explicit allow-list for polymorphism.
❌ Avoid BinaryFormatter (obsolete, removed in .NET 9).
❌ Avoid Newtonsoft.Json TypeNameHandling.All/Auto/Objects.
```

#### Secrets

```
✅ Keep secrets out of source code and appsettings.json.
✅ Use Secret Manager in development.
✅ Use a managed vault (Azure Key Vault, AWS Secrets Manager) in production.
✅ Persist data-protection keys outside ephemeral storage in multi-instance deployments.
```

#### Cryptography

```
✅ Use System.Security.Cryptography classes.
✅ Use vetted algorithms (AES-GCM, RSA-OAEP, SHA-256+).
❌ Do not implement your own encryption algorithms.
```

#### Authentication and Authorization

```
✅ Use the most secure authentication flow available.
✅ Apply [Authorize] by default; use [AllowAnonymous] sparingly.
✅ Enable MFA for sensitive operations.
✅ Validate JWT issuer, audience, signature, and lifetime.
✅ Use policies and claims, not hard-coded role checks.
```

#### Transport and Headers

```
✅ Enforce HTTPS and HSTS outside development.
✅ Configure forwarded headers correctly behind proxies.
✅ Set secure cookie flags: HttpOnly, Secure, SameSite.
✅ Do not combine AllowAnyOrigin with credentials in CORS.
```

#### Dependencies and Supply Chain

```
✅ Enable NuGet Audit (default in .NET 8+; transitive in .NET 9+).
✅ Treat audit warnings as errors in CI.
✅ Generate an SBOM (CycloneDX or SPDX) on every build.
✅ Use package source mapping.
✅ Run dotnet list package --vulnerable --include-transitive in CI.
```

#### Containers

```
✅ Run as a non-root user.
✅ Set allowPrivilegeEscalation: false.
✅ Use minimal base images.
✅ Do not store secrets in image layers.
```

#### CI/CD

```
✅ Run SAST (Roslyn security analyzers, CodeQL, SonarQube).
✅ Fail the build on security warnings.
✅ Scan dependencies and containers.
✅ Sign and verify artifacts where possible.
```

#### Error Handling and Logging

```
✅ Fail securely; deny access on error.
✅ Log security-relevant events (authentication, authorization failures, input rejection).
✅ Do not log secrets or sensitive data.
✅ Do not expose stack traces or internal details to users.
```

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe SQL Access

```csharp
// ❌ SQL injection risk
var sql = $"SELECT * FROM Users WHERE Name = '{userInput}'";

// ✅ Parameterised query (Dapper)
var user = await connection.QuerySingleOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Name = @Name", new { Name = userInput });

// ✅ Parameterised EF Core
var user = await dbContext.Users
    .FirstOrDefaultAsync(u => u.Name == userInput);
```

**Expected Output (conceptual)**

```
Injection attempt treated as literal data, not executable SQL.
```

**Why this output occurs**

- Parameters are sent separately from the SQL text, so the database never interprets user input as code.

---

#### Example 2: Secrets Management

```csharp
// ✅ Development: Secret Manager
dotnet user-secrets set "ConnectionStrings:Default" "..."

// ✅ Production: environment-specific configuration or managed vault
builder.Configuration.AddAzureKeyVault(new Uri(vaultUri), new DefaultAzureCredential());

// ❌ Never commit secrets
// appsettings.json: "ConnectionStrings": { "Default": "Server=...;Password=..." }
```

**Expected Output (conceptual)**

```
Secrets loaded at runtime from a secure store; none present in source control.
```

**Why this output occurs**

- Secret Manager keeps development secrets out of the repository.
- Key Vault (or an equivalent) provides audited, access-controlled secret storage in production.

---

#### Example 3: Dependency Auditing in CI

```xml
<PropertyGroup>
  <NuGetAudit>true</NuGetAudit>
  <NuGetAuditMode>all</NuGetAuditMode>
  <NuGetAuditLevel>low</NuGetAuditLevel>
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
</PropertyGroup>
```

```bash
dotnet list package --vulnerable --include-transitive
```

**Expected Output (conceptual)**

```
Build fails if any dependency (including transitive) has a known vulnerability at or above Low severity.
```

**Why this output occurs**

- NuGet Audit checks against the GitHub Advisory Database.
- `TreatWarningsAsErrors` converts audit warnings into build failures.

---

### Real-World Cases

- **ASP.NET Core APIs** — Parameterised EF Core queries, `[Authorize]` by default, HTTPS/HSTS, and safe JSON.
- **Microservices** — mTLS, scoped managed identities, and NuGet audit in every pipeline.
- **Containers** — Non-root user, minimal base image, and no secrets in layers.
- **Desktop applications** — DPAPI or platform key stores for local secrets; no hard-coded credentials.

---

### References

- Security Principles — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles
- Security (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/pl-pl/previous-versions/ms173195(v=vs.85)
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- Security And Identity — autoskills - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- Unsafe code best practices — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/unsafe-code
- Planning Step 4: Plan Application Security — Microsoft Learn - https://learn.microsoft.com/en-us/iis/application-frameworks/scenario-build-an-aspnet-website-on-iis/planning-step-4-plan-application-security
- Threat Modeling Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/9c87d0a0e9411de028b9d7eb853872082d6d3382/cheatsheets/Threat_Modeling_Cheat_Sheet.md
- Evaluate application threats with threat modeling — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling
- Pre-SDL Requirements: Security Training — Microsoft Learn - https://learn.microsoft.com/sv-se/previous-versions/windows/desktop/cc307407(v=msdn.10)

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`BinaryFormatter`** | Obsolete (SYSLIB0011); disabled by default; removed in .NET 9. Do not use for untrusted data. |
| **Newtonsoft.Json `TypeNameHandling.All/Auto/Objects`** | Functionally equivalent to `BinaryFormatter` RCE risk; avoid with untrusted input. |
| **`System.Runtime.Caching.MemoryCache`** | Legacy; `IMemoryCache` is recommended for ASP.NET Core. |
| **`async void` outside event handlers** | Exceptions are unobservable; use `async Task`. |
| **Sync-over-async (`.Result`, `.Wait()`)** | Deadlock and thread-pool starvation risk; use `await`. |
| **`[AllowAnonymous]` overuse** | Increases attack surface; use only where truly necessary. |
| **`AllowAnyOrigin` + credentials in CORS** | Insecure combination; use specific origins with credentials. |
| **Detailed exception pages in production** | Information disclosure; use `UseExceptionHandler`. |
| **`appsettings.json` for production secrets** | Secrets end up in Git; use Secret Manager (dev) and a managed vault (prod). |
| **Running containers as root** | Privilege escalation risk; use non-root users and `allowPrivilegeEscalation: false`. |
| **NuGet Audit** | Default in .NET 8+; transitive auditing in .NET 9+. Treat warnings as errors in CI. |
| **`System.Text.Json` source generation** | Required for Native AOT and recommended for security-sensitive serialisation. |

---

## Summary Table: Security Principles at a Glance

| Principle | Question It Answers | .NET / ASP.NET Core Implementation |
|---|---|---|
| **Threat modeling** | What can go wrong, and what will we do about it? | STRIDE, DFD, trust boundaries, Microsoft Threat Modeling Tool |
| **Attack surface** | What can an attacker reach? | Minimal endpoints, `CreateSlimBuilder`, dependency audit, rate limiting |
| **Defense in depth** | What if one control fails? | Layered auth, validation, encryption, logging, monitoring |
| **Least privilege** | What is the minimum permission needed? | Low-privilege identities, scoped RBAC, `[Authorize]` by default |
| **Secure defaults** | What happens if we configure nothing? | HTTPS/HSTS, antiforgery, safe JSON, NuGet Audit, secure cookies |
| **Best practices** | How do we avoid known vulnerability classes? | Parameterised queries, safe deserialisation, secrets vault, SAST, SBOM |

---

## References (Consolidated)

- Pre-SDL Requirements: Security Training — Microsoft Learn - https://learn.microsoft.com/sv-se/previous-versions/windows/desktop/cc307407(v=msdn.10)
- Security (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/pl-pl/previous-versions/ms173195(v=vs.85)
- Security Principles — Microsoft Learn (archived) - https://learn.microsoft.com/en-gb/archive/blogs/jmeier/security-principles
- Evaluate application threats with threat modeling — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling
- Threat Modeling Cheat Sheet — OWASP - https://github.com/owasp/cheatsheetseries/blob/9c87d0a0e9411de028b9d7eb853872082d6d3382/cheatsheets/Threat_Modeling_Cheat_Sheet.md
- Threat Modeling Tool — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling
- Describe defense in depth — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/describe-defense-in-depth/
- Planning Step 4: Plan Application Security — Microsoft Learn - https://learn.microsoft.com/en-us/iis/application-frameworks/scenario-build-an-aspnet-website-on-iis/planning-step-4-plan-application-security
- Security And Identity — autoskills - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- Unsafe code best practices — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/unsafe-code