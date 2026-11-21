# jQuery Security Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Security in a jQuery‑based application is the discipline of protecting the application's data, users, and functionality from unauthorised access, modification, or disruption by applying established principles — such as threat modelling, defence in depth, least privilege, and secure defaults — throughout the software development lifecycle, from design through deployment and maintenance.

**Technical Definition**
Application security is formally defined through the **CIA triad**: **Confidentiality** (preserving authorised restrictions on information access and disclosure, including means for protecting personal privacy and proprietary information), **Integrity** (guarding against improper information modification or destruction and ensuring information non‑repudiation and authenticity), and **Availability** (ensuring timely and reliable access to and use of information). In jQuery‑specific contexts, security concerns arise from client‑side code that processes untrusted input, manipulates the DOM, communicates with servers via AJAX, and handles data attributes — all of which can introduce injection vulnerabilities, access control gaps, and data exposure if not carefully managed. The four foundational principles applied in this cheat sheet are: **Threat Modelling** (systematically identifying threats and vulnerabilities during design), **Defence in Depth** (layering independent security controls so that no single failure compromises the system), **Least Privilege** (granting users, processes, and programs only the minimum access necessary to perform their intended function), and **Secure Defaults** (configuring systems so that the out‑of‑the‑box experience is secure, with users opting out of security rather than opting in).

**Beginner‑Friendly Explanation**
Security is about keeping the bad guys out — and keeping the good guys' data safe. When you build a web page with jQuery, you are connecting a user's browser to a server and to data. Security fundamentals are the habits that prevent someone from stealing that data, impersonating a user, or breaking the application. Think of it like securing a house: you lock the doors (authentication), decide who gets a key (authorisation), put valuables in a safe (encryption), install an alarm (monitoring), and make sure the default state of the locks is "locked" rather than "open" (secure defaults). No single lock is perfect, so you use several layers (defence in depth) and give each person only the keys they need (least privilege). Before building, you think about how someone might try to break in (threat modelling). These principles apply just as much to a jQuery application as to a physical building.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **CIA triad as the security objective** | Confidentiality, Integrity, and Availability define what security is trying to achieve. |
| **Threat modelling precedes code** | Threats are identified during design, before vulnerabilities are written into production. |
| **Defence in depth assumes failure** | Every security component can be bypassed; multiple independent layers increase attacker effort and enable detection. |
| **Least privilege limits blast radius** | Minimal permissions reduce the attack surface and contain the damage if a component is compromised. |
| **Secure defaults protect the unaware** | The default configuration is the secure one; users must actively opt out of protection. |
| **Client‑side validation is not security** | jQuery validation improves UX but must be backed by server‑side enforcement; client‑side hiding of buttons is not access control. |
| **Injection is the persistent threat** | SQL injection, XSS, and command injection remain top OWASP risks because untrusted input is interpreted as code. |

---

### Prerequisites

- Basic understanding of how web applications work (client, server, HTTP, databases).
- Familiarity with jQuery selection, DOM manipulation, AJAX, and event handling.
- Awareness of HTML forms, cookies, and browser storage (localStorage, sessionStorage).
- No prior security expertise is required; the cheat sheet builds from first principles.

---

### Related Programming Areas

- **Authentication and Authorisation** — verifying identity and enforcing access rules.
- **Input Validation and Output Encoding** — preventing injection and XSS.
- **Session Management** — protecting user sessions and cookies.
- **Cryptography** — protecting data at rest and in transit.
- **Secure Configuration** — eliminating misconfigurations that expose systems.
- **Supply Chain Security** — managing dependencies and third‑party libraries.

---

### Core Concepts / Features (Index)

1. Security (Core Concept)
2. Threat Modelling
3. Attack Surface
4. Defence in Depth
5. Least Privilege
6. Secure Defaults
7. Security Best Practices


## Core Concept 1: Security (Core Definition and CIA Triad)

### Definitions

**Core Definition**
Security is the state of being protected against threats to the confidentiality, integrity, and availability of information and systems, achieved through the application of controls that prevent, detect, and respond to malicious or accidental harm.

**Technical Definition**
Security is formally characterised by the **CIA triad**. **Confidentiality** preserves authorised restrictions on information access and disclosure, including means for protecting personal privacy and proprietary information. **Integrity** guards against improper information modification or destruction and ensures information non‑repudiation and authenticity. **Availability** ensures timely and reliable access to and use of information. In web applications, these objectives are pursued through authentication (verifying identity), authorisation (enforcing access rules), auditing (tracking access and modifications), and management (session security, exception handling, and secure configuration).

**Beginner‑Friendly Explanation**
Security means three things: that private information stays private (confidentiality), that information is not tampered with (integrity), and that the application is there when you need it (availability). Every security decision you make — encrypting a password, validating a form, writing logs — serves one or more of these three goals.

---

### Purposes

- **To protect sensitive data** from unauthorised disclosure (confidentiality).
- **To prevent data from being altered or corrupted** by malicious or accidental actions (integrity).
- **To ensure the application remains accessible** to legitimate users (availability).
- **To provide a foundation for all other security principles** — threat modelling, defence in depth, least privilege, and secure defaults all serve the CIA triad.
- **To guide risk decisions** by identifying which of the three objectives a given control protects.

---

### Syntax Rules and Structure

Security is not expressed in a single syntax; it is a set of principles applied through code, configuration, and process. The CIA triad provides the evaluative framework:

| Objective | Question | jQuery‑Relevant Example |
|---|---|---|
| **Confidentiality** | Can unauthorised parties read the data? | Avoiding exposure of session tokens in client‑side code. |
| **Integrity** | Can unauthorised parties modify the data? | Validating and encoding data before DOM insertion. |
| **Availability** | Can legitimate users access the application? | Preventing client‑side denial‑of‑service through unbounded loops or requests. |

---

### Constraints and Limitations

- **Perfect security does not exist**; the goal is risk reduction, not absolute protection.
- **Security must be balanced against usability**; overly restrictive controls can make the application unusable.
- **Client‑side jQuery code runs in an untrusted environment**; users can modify it, so security enforcement must occur on the server.

---

### Annotated Code Example

```javascript
// ❌ INSECURE: exposing sensitive data in client-side code
var apiKey = "sk_live_123456789";          // exposed in the browser
$.ajax({
  url: "https://api.example.com/data",
  headers: { "Authorization": "Bearer " + apiKey }
});

// ✅ SECURE: server-side proxy holds the secret
// Client calls a server endpoint that authenticates the user
$.ajax({
  url: "/api/data",                         // server proxies the request
  type: "GET",
  dataType: "json"
  // No secret in client-side code
});
```

**Expected Output**
Both versions fetch data. The secure version prevents the API key from being visible in the browser's source code, DevTools, or network tab, protecting **confidentiality**.

**Why this output occurs**: Client‑side JavaScript is fully visible to users. Any secret embedded in it can be extracted. Server‑side proxying keeps secrets out of the client, consistent with the confidentiality objective of the CIA triad.

---

### Real‑World Cases

- **Login forms**: Passwords are sent over HTTPS (confidentiality) and hashed on the server (integrity of stored credentials).
- **Shopping carts**: Prices are validated on the server to prevent tampering (integrity).
- **Content delivery**: CDNs and caching improve availability of static assets.

---

## Core Concept 2: Threat Modelling

### Definitions

**Core Definition**
Threat modelling is a systematic and structured security technique used to identify the security objectives, threats, and vulnerabilities of an application — typically during the design phase — in order to make informed design and engineering decisions and to prioritise security efforts.

**Technical Definition**
Threat modelling, as described by OWASP, is a structured technique for identifying security objectives, threats, and vulnerabilities to help make design and engineering decisions and determine where to prioritise efforts in designing, developing, and deploying secure applications. The process is iterative and includes steps such as identifying security objectives, profiling the application, decomposing the application (generating context and scenarios), identifying threats, and determining countermeasures. Frameworks such as **STRIDE** (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) provide a mnemonic for systematically categorising threats during the modelling process.

**Beginner‑Friendly Explanation**
Threat modelling is like planning a bank robbery — from the defender's side. Before you build the bank, you ask: Who might try to rob it? How would they get in? What are they after? What defences would stop them? In software, you ask the same questions about your application: Who are the potential attackers? What are the valuable assets (user data, payment information)? How could they attack? What controls will stop them? Doing this before writing code is far cheaper than fixing vulnerabilities after launch.

---

### Purposes

- **To identify threats and vulnerabilities before code is written**, reducing the cost of remediation.
- **To prioritise security efforts** based on risk and impact.
- **To align security controls with business objectives** and data sensitivity.
- **To document design decisions** and their security rationale.
- **To provide a repeatable process** for ongoing security review as the application evolves.

---

### Syntax Rules and Structure

#### Threat Modelling Process (OWASP‑Aligned)

| Step | Description |
|---|---|
| **1. Identify Security Objectives** | Define what needs protection (CIA triad) and why. |
| **2. Profile the Application** | Understand architecture, data flows, and trust boundaries. |
| **3. Decompose the Application** | Break down components, entry points, and assets. |
| **4. Identify Threats** | Use frameworks such as STRIDE to categorise threats. |
| **5. Determine Countermeasures** | Select controls that mitigate the identified threats. |

#### STRIDE Categories

| Category | Threat |
|---|---|
| **Spoofing** | Impersonating a user or system. |
| **Tampering** | Unauthorised modification of data or code. |
| **Repudiation** | Denying having performed an action. |
| **Information Disclosure** | Exposing data to unauthorised parties. |
| **Denial of Service** | Making the system unavailable. |
| **Elevation of Privilege** | Gaining permissions beyond what is authorised. |

---

### Constraints and Limitations

- **Threat modelling is not a one‑time activity**; it should be repeated as the application changes.
- **Models are only as good as the assumptions** behind them; unstated assumptions are a common source of blind spots.
- **Threat modelling requires diverse input** (development, security, operations, business) to be effective.

---

### Annotated Code Example (Conceptual Threat Model in Code)

```javascript
// Step 1: Identify assets and trust boundaries
// Asset: user credentials. Trust boundary: client → server.

// Step 2: Apply STRIDE — Spoofing threat on login
// Threat: an attacker submits another user's credentials

// Step 3: Countermeasure — server-side authentication + rate limiting
$.ajax({
  url: "/api/login",
  type: "POST",
  data: { username: u, password: p },
  dataType: "json",
  success: function (response) {
    // Server has already validated credentials and set an HttpOnly cookie
    console.log("Authenticated");
  },
  error: function () {
    // Generic error to avoid leaking whether the username exists
    console.log("Login failed");
  }
});
```

**Expected Output**
The client sends credentials over HTTPS. The server verifies them, sets a secure session cookie, and returns a generic error on failure. The generic error prevents **information disclosure** (revealing whether a username exists), and rate limiting mitigates **denial of service** through brute force.

**Why this output occurs**: The threat model identified spoofing and information disclosure as key threats on the login endpoint. The countermeasures — server‑side authentication, secure cookies, generic errors, and rate limiting — directly address those threats.

---

### Real‑World Cases

- **Payment flows**: Threat modelling identifies tampering with transaction amounts; countermeasures include server‑side price validation and HTTPS.
- **File uploads**: Threat modelling identifies malicious file uploads; countermeasures include extension allowlists and storing files outside the webroot.
- **API design**: Threat modelling identifies broken object‑level authorisation (IDOR); countermeasures include per‑object access checks.

---

## Core Concept 3: Attack Surface

### Definitions

**Core Definition**
The attack surface of a system is the set of all points — code, interfaces, services, protocols, and practices — through which an attacker could attempt to enter, extract data from, or otherwise compromise the system.

**Technical Definition**
Attack surface is defined as "the set of ways that a system might be susceptible to an attack" or "the exposure to malicious activity". It encompasses **network attack surface** (open ports, services, protocols), **software attack surface** (code that processes incoming data, email, XML, office documents, web forms), and **human attack surface** (employees with access to sensitive information who may be socially engineered). **Attack surface reduction** is the practice of minimising the total reachable and exploitable vulnerabilities by disabling unused services, restricting permissions, and removing unnecessary code paths.

**Beginner‑Friendly Explanation**
The attack surface is every door, window, and vent in your house that a burglar could try. The more openings you have, the more chances there are to get in. In a web application, the attack surface includes every form, every API endpoint, every file upload field, and every library you include. Reducing the attack surface means closing the openings you do not need — turning off unused services, removing unnecessary features, and limiting who can access what.

---

### Purposes

- **To identify all entry points** that an attacker could exploit.
- **To prioritise security testing and hardening** on the most exposed surfaces.
- **To reduce the number of exploitable vulnerabilities** by eliminating unnecessary code and services.
- **To limit lateral movement** within a system if one component is compromised.
- **To inform threat modelling** by clarifying where threats are most likely to originate.

---

### Syntax Rules and Structure

| Attack Surface Category | Examples |
|---|---|
| **Network** | Open ports, outward‑facing web servers, internal services |
| **Software** | Code processing input, web forms, SQL interfaces, custom data formats |
| **Human** | Employees with sensitive access, social engineering targets |
| **Client‑side (jQuery‑specific)** | DOM‑based XSS sinks, exposed API keys, client‑side validation logic |

---

### Constraints and Limitations

- **Some attack surface is unavoidable**; the goal is to minimise unnecessary exposure, not eliminate all entry points.
- **New features add attack surface**; every new form, API, or integration must be evaluated for security impact.
- **Third‑party libraries and dependencies expand the attack surface** and must be monitored and updated.

---

### Annotated Code Example

```javascript
// ❌ EXCESSIVE ATTACK SURFACE: exposing debug endpoints and verbose errors
app.get("/api/debug", function (req, res) {
  res.json({ config: process.env });       // leaks secrets
});

app.use(function (err, req, res, next) {
  res.status(500).send(err.stack);         // leaks internal paths
});

// ✅ REDUCED ATTACK SURFACE: disable debug endpoints, generic errors
app.use(function (err, req, res, next) {
  console.error(err.stack);                // log details server-side
  res.status(500).send("An error occurred"); // generic client message
});
```

**Expected Output**
The insecure version exposes environment variables and stack traces to attackers, expanding the software attack surface through information disclosure. The secure version logs details server‑side and returns a generic message, reducing the surface.

**Why this output occurs**: Debug endpoints and verbose errors are entry points that provide attackers with useful information. Disabling them and returning generic errors reduces the exploitable software surface, consistent with attack surface reduction principles.

---

### Real‑World Cases

- **API gateways**: Restricting which endpoints are publicly accessible reduces network attack surface.
- **File uploads**: Limiting accepted file types and sizes reduces the software attack surface.
- **Client‑side code**: Removing unused jQuery plugins and exposing fewer global functions reduces the client‑side attack surface.

---

## Core Concept 4: Defence in Depth

### Definitions

**Core Definition**
Defence in depth is a security design principle based on the assumption that any single security component or measure can be bypassed, and that security is therefore achieved through a coordinated combination of multiple, independent layers of protection aligned with attack scenarios.

**Technical Definition**
Defence in depth relies on "a coordinated combination of technical, organisational, and physical measures aligned with attack scenarios, in order to increase the effort and resources required for attackers to achieve their ends, to facilitate detection, and to limit the operational impact of attacks". It is defined by "the use of multiple robust and independent security layers, designed to ensure that no single attack can bypass several safeguards simultaneously". The principle is derived from military strategy: it seeks to delay rather than prevent the advance of an attacker, relying on the tendency of an attack to lose momentum over time or as it covers a larger area. OWASP describes it as a design principle to guide the selection of controls to ensure resilience against different forms of attack.

**Beginner‑Friendly Explanation**
Defence in depth is like a castle with a moat, a wall, a gatehouse, and armed guards. If an attacker gets past the moat, the wall stops them. If they get past the wall, the gatehouse stops them. No single defence is perfect, but together they make the castle very hard to take. In a web application, defence in depth means using several independent security controls — input validation, output encoding, authentication, authorisation, logging, and encryption — so that if one fails, another still protects the system.

---

### Purposes

- **To increase the effort required by an attacker** to succeed.
- **To ensure that a single control failure does not compromise the entire system**.
- **To facilitate detection** of attacks through multiple monitoring points.
- **To limit the operational impact** of a successful attack.
- **To provide resilience** against different forms of attack by combining complementary controls.

---

### Syntax Rules and Structure

#### Layers of Defence (Web Application Context)

| Layer | Control Examples |
|---|---|
| **Input** | Validation, allowlists, length limits |
| **Processing** | Parameterised queries, safe APIs, sandboxing |
| **Output** | Contextual encoding, sanitisation |
| **Authentication** | Strong password policies, multi‑factor authentication |
| **Authorisation** | Least privilege, role‑based access control |
| **Session** | Secure cookies, timeouts, rotation |
| **Network** | HTTPS, firewalls, segmentation |
| **Monitoring** | Logging, intrusion detection, alerting |

---

### Constraints and Limitations

- **Multiple layers add complexity**; controls must be coordinated to avoid gaps or conflicting behaviour.
- **Defence in depth is not a substitute for fixing root causes**; it reduces impact but does not eliminate vulnerabilities.
- **Layers must be independent**; if the same flaw affects multiple layers, the depth is illusory.

---

### Annotated Code Example

```javascript
// ❌ SINGLE LAYER: client-side validation only
$("#email").on("submit", function (e) {
  if (!isValidEmail($("#email").val())) {
    e.preventDefault();                    // client-side only
  }
});

// ✅ DEFENCE IN DEPTH: client-side + server-side + output encoding
// Client-side (UX layer)
$("#email").on("submit", function (e) {
  if (!isValidEmail($("#email").val())) {
    e.preventDefault();
    showError("Invalid email");
  }
});

// Server-side (enforcement layer)
app.post("/subscribe", function (req, res) {
  if (!isValidEmail(req.body.email)) {
    return res.status(400).send("Invalid email");   // server enforces
  }
  db.query("INSERT INTO subscribers (email) VALUES (?)", [req.body.email]); // parameterised
  res.send("Subscribed");
});

// Output encoding (presentation layer)
// Template engine automatically escapes HTML
```

**Expected Output**
A malicious user who bypasses the client‑side validation is stopped by the server‑side check. If the email is stored and later displayed, the template engine encodes it, preventing XSS. Three independent layers — client, server, and output — provide defence in depth.

**Why this output occurs**: Client‑side validation is bypassable. Server‑side validation enforces the rule. Parameterised queries prevent SQL injection. Output encoding prevents XSS. Each layer addresses a different failure mode.

---

### Real‑World Cases

- **Login systems**: Password hashing, rate limiting, account lockout, and multi‑factor authentication operate as independent layers.
- **Payment processing**: Tokenisation, HTTPS, server‑side price validation, and fraud detection work together.
- **File uploads**: Extension allowlisting, content‑type checking, size limits, and storing files outside the webroot.

---

## Core Concept 5: Least Privilege

### Definitions

**Core Definition**
The principle of least privilege dictates that a user, process, or program should be given only the minimum level of access or permissions necessary to perform its intended function, and nothing more.

**Technical Definition**
The principle of least privilege (PoLP) is a fundamental concept in information security. It requires that "a user, process, or program should be given only the minimum level of access or permissions necessary to perform its intended function, and nothing more". Its primary goals are to reduce the **attack surface** (the sum of all possible entry points an unauthorised user could use to gain access) and to limit the **blast radius** (the extent of damage an attacker can inflict if they successfully compromise part of the system). PoLP applies to operating systems, databases, cloud IAM roles, and software engineering (e.g., microservices with minimal permissions).

**Beginner‑Friendly Explanation**
Least privilege means giving people only the keys they need to do their job — nothing more. A hotel cleaner needs a key to the rooms they clean, not a key to the safe. In a web application, the database user that reads product information should only have read access to the product table, not the ability to delete customer records. If an attacker compromises that database user, they can only do what that user could do — which is very little. This limits the damage.

---

### Purposes

- **To minimise the attack surface** by reducing the number of permissions an attacker can exploit.
- **To limit the blast radius** of a compromise by containing the damage to the minimum necessary scope.
- **To enforce the principle of separation of duties** by ensuring no single account has excessive control.
- **To simplify auditing and compliance** by making permissions explicit and minimal.
- **To reduce the risk of accidental damage** from human error.

---

### Syntax Rules and Structure

| Context | Least‑Privilege Example |
|---|---|
| **Database** | Application user has `SELECT` on required tables only; no `DROP`, `DELETE`, or admin rights |
| **Operating system** | Web server runs as a low‑privilege user, not `root` or `Administrator` |
| **Cloud IAM** | Service account can read from one bucket and write to another, but cannot delete buckets or access other resources |
| **Microservices** | Each service has its own credentials and only the permissions needed for its specific task |

---

### Constraints and Limitations

- **Least privilege requires ongoing management**; permissions must be reviewed and revoked as roles change.
- **Overly restrictive permissions can break functionality**; the goal is the minimum *necessary*, not the minimum possible.
- **Shared accounts violate least privilege**; each user and service should have unique credentials.

---

### Annotated Code Example

```javascript
// ❌ VIOLATION: database connection with root privileges
var connection = mysql.createConnection({
  user: "root",                           // full database access
  password: process.env.DB_ROOT_PASSWORD,
  database: "app"
});
connection.query("SELECT * FROM products", function (err, rows) { … });

// ✅ LEAST PRIVILEGE: dedicated user with read-only access
var connection = mysql.createConnection({
  user: "app_reader",                     // only SELECT on products
  password: process.env.DB_READER_PASSWORD,
  database: "app"
});
connection.query("SELECT * FROM products", function (err, rows) { … });
```

**Expected Output**
Both queries return the product list. The least‑privilege version limits the damage an attacker can do if the `app_reader` credentials are compromised: they cannot modify or delete data, and they cannot access other tables.

**Why this output occurs**: The `app_reader` user has only the permissions required to execute `SELECT` on the `products` table. Any attempt to run `DELETE` or `DROP` would be rejected by the database, containing the blast radius.

---

### Real‑World Cases

- **Web servers**: Running as `www-data` or `nginx` rather than `root`.
- **CI/CD pipelines**: Deploy keys with write access only to the specific repository and branch needed.
- **Cloud functions**: IAM roles scoped to the exact resources the function needs.

---

## Core Concept 6: Secure Defaults

### Definitions

**Core Definition**
Secure defaults is the principle that a system's out‑of‑the‑box configuration should be secure, and that users should have to explicitly opt out of security protections rather than explicitly opt in to them.

**Technical Definition**
OWASP describes secure defaults as follows: "by default, the experience should be secure, and it should be up to the user to reduce their security – if they are allowed". It is imperative for the software environment to have default secure settings which may be opted out of by the user or other options which may be opted into (commonly known as opt‑in and opt‑out). For example, password aging and complexity should be enabled by default, and users may be allowed to turn these features off, simplifying their use of the application while increasing their risk. Secure defaults do not mean turning off all network services; they mean ensuring the "least number of possible loopholes and fewer drawbacks". The principle traces to Saltzer and Schroeder's **fail‑safe defaults**: base access decisions on permission rather than exclusion.

**Beginner‑Friendly Explanation**
Secure defaults mean that when you install or configure something, the safe settings are already turned on. You do not have to remember to enable security — you have to actively decide to turn it off. Think of a new phone that comes with a passcode required by default; you can disable it, but you have to make that choice deliberately. In web applications, secure defaults include enabling HTTPS, requiring strong passwords, setting secure cookie flags, and denying access unless explicitly granted.

---

### Purposes

- **To protect users who do not change default settings**.
- **To reduce the risk of misconfiguration** by making the safe configuration the path of least resistance.
- **To align with fail‑safe defaults** — access is denied unless explicitly permitted.
- **To create a secure baseline** that users can consciously weaken if they choose.
- **To satisfy compliance and regulatory requirements** that mandate secure configurations.

---

### Syntax Rules and Structure

| Setting | Secure Default | Insecure Default (Avoid) |
|---|---|---|
| **HTTPS** | Redirect HTTP to HTTPS; HSTS enabled | Allow plain HTTP |
| **Cookies** | `Secure`, `HttpOnly`, `SameSite` flags set | No flags |
| **Password policy** | Minimum length and complexity enabled | No requirements |
| **Access control** | Deny by default; allow explicitly | Allow by default; deny explicitly |
| **Error messages** | Generic client message; detailed server log | Stack traces exposed to client |
| **Debug mode** | Disabled in production | Enabled in production |

---

### Constraints and Limitations

- **Secure defaults can conflict with usability**; a balance must be struck, but the default must remain secure.
- **Users may disable security controls**; the system should warn them of the risk.
- **Defaults must be reviewed as threats evolve**; "today's secure default becomes tomorrow's vulnerability".

---

### Annotated Code Example

```javascript
// ❌ INSECURE DEFAULT: cookies without security flags
res.cookie("session", sessionId);         // no Secure, HttpOnly, SameSite

// ✅ SECURE DEFAULT: cookies with security flags
res.cookie("session", sessionId, {
  secure: true,                           // HTTPS only
  httpOnly: true,                         // not accessible via JavaScript
  sameSite: "strict"                      // not sent on cross-site requests
});
```

**Expected Output**
Both versions set a session cookie. The secure version protects the cookie from interception over HTTP (`Secure`), from theft via XSS (`HttpOnly`), and from cross‑site request forgery (`SameSite`).

**Why this output occurs**: The secure flags are enabled by default, so the developer does not need to remember to add them. If the developer wants to weaken the configuration, they must explicitly set `secure: false`, which serves as a conscious decision.

---

### Real‑World Cases

- **Frameworks**: Django's `CSRF_COOKIE_SECURE` and `SESSION_COOKIE_SECURE` default to `False` in development but should be `True` in production templates.
- **Cloud services**: S3 buckets default to private access, not public read.
- **Operating systems**: Windows User Account Control prompts for elevation by default.

---

## Core Concept 7: Security Best Practices

### Definitions

**Core Definition**
Security best practices are the consolidated, actionable conventions — derived from threat modelling, defence in depth, least privilege, and secure defaults — that guide the design, implementation, and maintenance of secure jQuery‑based web applications.

**Technical Definition**
Security best practices synthesise the foundational principles into concrete controls: (1) **validate input at trust boundaries** using allowlists, and **encode output contextually** at the sink; (2) **enforce access control on the server** with deny‑by‑default and per‑object checks (preventing IDOR); (3) **use parameterised queries** to prevent SQL injection; (4) **apply secure defaults** to cookies, HTTPS, error handling, and debug modes; (5) **apply least privilege** to database users, service accounts, and IAM roles; (6) **manage dependencies and the supply chain** with lockfiles, SCA scanning, and SBOMs; (7) **keep secrets out of client‑side code** and inject them at runtime from a manager; and (8) **log security‑relevant events** and return generic errors to clients while retaining detail server‑side.

**Beginner‑Friendly Explanation**
Best practices are the habits that keep your application safe in everyday development. Validate what users type, encode what you display, check permissions on the server, use parameterised queries, turn on secure defaults, give each service only the access it needs, keep secrets out of the browser, and log what happens. None of these is exotic or difficult — they are the straightforward application of the principles covered in this cheat sheet. Following them consistently prevents the vast majority of common web vulnerabilities.

---

### Purposes

- **To provide a single actionable checklist** for secure jQuery development.
- **To prevent the most common vulnerabilities** identified by OWASP (broken access control, injection, misconfiguration).
- **To balance security with development velocity** through automated controls and clear patterns.
- **To establish a baseline for code reviews and security audits**.
- **To ensure that client‑side jQuery code does not become the weakest link** in an otherwise secure system.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Input validation** | Allowlist at trust boundaries | Trusting client‑side validation |
| **Output encoding** | Contextual encoding at the sink | Raw/unescaped output (`innerHTML`, `dangerouslySetInnerHTML`) |
| **Access control** | Server‑side deny‑by‑default; per‑object checks | Client‑side hiding of buttons |
| **SQL** | Parameterised queries | String concatenation |
| **Secrets** | Runtime injection from a manager | Secrets in source code or client bundles |
| **Cookies** | `Secure`, `HttpOnly`, `SameSite` | No security flags |
| **Errors** | Generic client message; detailed server log | Stack traces to clients |
| **Dependencies** | Lockfiles, SCA, SBOM | Unpinned versions |

---

### Constraints and Limitations

- **Best practices must be adapted to context**; a static marketing site has different needs from a banking application.
- **Automation is essential**; manual checks decay over time.
- **Security is a continuous process**, not a one‑time checklist.

---

### Annotated Code Example (Consolidated)

```javascript
// ✅ SECURITY BEST PRACTICES — CONSOLIDATED

// 1. Input validation at the trust boundary (client + server)
function isValidUsername(u) {
  return /^[a-zA-Z0-9_]{3,20}$/.test(u);  // allowlist
}
$("#username").on("input", function () {
  var ok = isValidUsername($(this).val());
  $("#hint").text(ok ? "" : "3–20 letters, digits, underscores");
});

// 2. Server-side enforcement (not shown) — the client check is UX only

// 3. Output encoding (template engine auto-escapes)
// <div><%= username %></div>  — escaped by default

// 4. Parameterised query (server-side)
// db.query("SELECT * FROM users WHERE username = ?", [username]);

// 5. Secure cookie defaults
// res.cookie("session", id, { secure: true, httpOnly: true, sameSite: "strict" });

// 6. Generic error handling
$.ajax({
  url: "/api/profile",
  type: "GET",
  dataType: "json",
  error: function () {
    $("#status").text("Unable to load profile");  // generic client message
  }
});

// 7. No secrets in client code — server proxies API calls
```

**Expected Output**
The client validates the username for UX, but the server enforces the same rule (or stricter). Output is encoded by the template engine. Database queries are parameterised. Cookies are secure. Errors are generic. No secrets are exposed. Each practice corresponds to a principle from the cheat sheet.

**Why this output occurs**: The code demonstrates input validation (allowlist), server‑side enforcement (conceptual), output encoding (template engine), parameterised queries (conceptual), secure defaults (cookies), generic errors, and secret isolation — the concrete expression of threat modelling, defence in depth, least privilege, and secure defaults.

---

### Real‑World Cases

- **Code reviews**: A checklist ensures that new jQuery code follows established security patterns.
- **Legacy refactoring**: Identifying client‑only validation and string‑concatenated SQL as candidates for remediation.
- **CI/CD pipelines**: Automated secret scanning and SCA scanning enforce best practices at scale.

---

## References

- OWASP Threat Modeling — https://wiki.owasp.org/index.php/Threat_Modeling
- OWASP Threat Modeling Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
- OWASP: Defence in Depth — https://wiki.owasp.org/index.php/Defense_in_depth
- ANSSI: Defence‑in‑Depth: A Methodology — https://messervices.cyber.gouv.fr/documents-guides/anssi_back_to_basics_defence_in_depth_methodology_1.0.pdf
- OWASP: Least Privilege Principle — https://community.owasp.org/controls/Least_Privilege_Principle
- OWASP: Least Privilege (Wiki) — https://wiki.owasp.org/index.php/Least_privilege
- OWASP: Establish Secure Defaults — https://wiki.owasp.org/index.php/Establish_secure_defaults
- NIST: Confidentiality, Integrity, Availability (CIA Triad) — https://csrc.nist.gov/glossary/term/confidentiality_integrity_availability
- NIST SP 800‑12: An Introduction to Computer Security — https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-12.pdf
- Microsoft: STRIDE Threat Model — https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats
- OWASP Top 10:2025 — https://owasp.org/Top10/2025/
- OWASP Application Security Verification Standard (ASVS) — https://owasp.org/www-project-application-security-verification-standard/
- OWASP Secure Coding Practices Quick Reference Guide — https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/
- NIST: Attack Surface Reduction — https://csrc.nist.rip/CSRC/media/Presentations/FISSEA-2016-Conference-Presentation-Attack-Surfa/images-media/FISSEA%20_Fishman-Attack_Surface_Reduction-Tues400.pdf
- CWE/SANS Top 25 — https://cwe.mitre.org/top25/
- MDN Web Security — https://developer.mozilla.org/en-US/docs/Web/Security
- Saltzer and Schroeder: The Protection of Information in Computer Systems — https://web.mit.edu/Saltzer/www/publications/protection/
- OWASP: Input Validation Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP: Cross Site Scripting Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP: SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP: Access Control Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html
- OWASP: Secrets Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html