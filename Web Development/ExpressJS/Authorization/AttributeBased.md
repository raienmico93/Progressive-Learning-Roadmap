# Attribute-Based Access Control (ABAC) & Policy Engines — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Attribute-Based Access Control (ABAC) is an authorization model that evaluates attributes of the subject (user), the resource (object), the action, and the environment (context) against a policy to make dynamic access decisions — rather than relying solely on static roles.

**Technical Definition:** ABAC is defined by the NIST Special Publication 800-162 as an access control method where subject and object attributes, environmental conditions, and a set of policies are evaluated to make access decisions. Unlike RBAC, which grants access based on role membership, ABAC evaluates fine-grained attributes at request time: the requesting user's department, clearance level, and location; the target resource's classification, owner, and creation date; the action being performed; and contextual factors like time of day, IP subnet, and request rate. **Policy-Based Access Control (PBAC)** centralises these rules into a logical business engine — either embedded in the application (CASL, Casbin) or decoupled as an external service (Cerbos, Open Policy Agent). The policy engine evaluates the request context against the policy and returns an allow/deny decision.

**Beginner-Friendly Explanation:** RBAC is like a nightclub with coloured wristbands — blue for regular, gold for VIP. ABAC is like a smart door that considers many factors: "Is this person a manager? Is it during business hours? Is the document marked 'confidential'? Is the request coming from the office network?" The door makes a decision based on all these factors, not just the wristband colour. A policy engine is the rulebook the door follows — a central place where all the rules are written and evaluated.

### Key Characteristics

- **Fine-grained:** Decisions are based on attributes, not just role membership.
- **Dynamic:** Access decisions can change based on time, location, and request context.
- **Multi-dimensional:** Subject, resource, action, and environment attributes are all evaluated.
- **Policy-driven:** Rules are expressed declaratively in a policy language or engine.
- **Centralised (PBAC):** Policies live in one place, evaluated consistently across services.
- **Auditable:** Every decision can be logged with the attributes that led to it.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A policy engine library:** `npm install @casl/ability` or `npm install casbin` (embedded), or a running Cerbos/OPA instance (external).
- **A working authentication system:** JWT or session-based, with `req.user` populated with attributes.
- **Basic JavaScript knowledge:** Functions, objects, and middleware concepts.

### Related Programming Areas

- **Role-Based Access Control (RBAC):** ABAC is a superset — RBAC can be implemented as a special case of ABAC.
- **Policy-Based Access Control (PBAC):** The architectural pattern of centralising policies in engines.
- **Open Policy Agent (OPA):** A general-purpose policy engine using the Rego language.
- **Cerbos:** A stateless authorization service with YAML-based policies.
- **CASL:** An isomorphic JavaScript authorization library (works on client and server).
- **Zero Trust Architecture:** ABAC is a foundational element of zero-trust security models.

### Core Concepts

1. **User Attributes** — evaluating claims linked to the subject.
2. **Resource Attributes** — evaluating metadata attached to the object.
3. **Contextual Authorization (Environment)** — evaluating execution realities.
4. **Policy-Based Access Control (PBAC)** — centralising rules into policy engines (CASL, Cerbos, OPA).

---

## Core Concept 1: User Attributes

### Definitions

**Core Definition:** User attributes (subject attributes) are the properties and claims associated with the requesting user — such as their department, role, security clearance level, employment status, and location — that are evaluated during an access decision.

**Technical Definition:** In ABAC, the subject is the entity requesting access. Subject attributes are typically drawn from the identity provider (IdP), the application's user database, or the authentication token (JWT claims). Common subject attributes include: `department` (e.g., `engineering`, `finance`), `clearanceLevel` (e.g., `public`, `confidential`, `secret`), `jobTitle`, `employmentType` (e.g., `full-time`, `contractor`), `location`, and `roles`. These attributes are evaluated against the policy to determine whether the subject can perform the requested action on the requested resource. The key advantage over RBAC is that ABAC can express rules like "users in the finance department with a clearance level of 'confidential' or higher can approve expenses over $10,000" — a rule that would require many roles to express in pure RBAC.

**Beginner-Friendly Explanation:** User attributes are like the information on your employee ID card. Instead of just a colour (role), the card shows your department, your security clearance, your job title, and whether you're a full-time employee or a contractor. A smart door can say "only full-time employees from the finance department with 'confidential' clearance can enter the vault." That's ABAC — the door reads all the attributes on your card, not just the colour.

### Purposes

- To evaluate claims linked to the subject (e.g., clearance level, department, security clearance).
- To enable fine-grained access rules that cannot be expressed with roles alone.
- To enforce the principle of least privilege based on multiple dimensions.
- To support dynamic rules that consider employment type, tenure, and location.

### Syntax Rules and Structure

#### User Attribute Schema

```javascript
const userAttributes = {
  id: 'usr_123',
  department: 'finance',
  clearanceLevel: 'confidential',        // public | internal | confidential | secret
  jobTitle: 'Senior Accountant',
  employmentType: 'full-time',           // full-time | contractor | intern
  location: 'US-NY',
  roles: ['accountant', 'manager'],
  yearsOfService: 5
};
```

| Attribute | Type | Example Values |
|-----------|------|----------------|
| `department` | String | `engineering`, `finance`, `hr` |
| `clearanceLevel` | Enum | `public`, `internal`, `confidential`, `secret` |
| `employmentType` | Enum | `full-time`, `contractor`, `intern` |
| `location` | String | `US-NY`, `EU-DE`, `APAC-SG` |
| `yearsOfService` | Number | 0–40 |
| `roles` | Array | `['accountant', 'manager']` |

#### Attribute Loading

```javascript
// Load attributes from database and attach to req.user
async function loadUserAttributes(req, res, next) {
  const user = await User.findById(req.user.id).select(
    'department clearanceLevel jobTitle employmentType location roles'
  );
  req.user = { ...req.user, ...user.toObject() };
  next();
}
```

**Rules:**
- Load attributes from a trusted source (database or IdP) — never from client-supplied input.
- Include attributes in the JWT for stateless evaluation, or load them from the database for real-time accuracy.
- Use a consistent naming convention for attributes.
- Validate attribute values against an allow-list (e.g., `clearanceLevel` must be one of the defined levels).
- Refresh attributes regularly — stale attributes can grant unintended access.

**Constraints:**
- Attributes in JWTs become stale — use short-lived tokens for attribute changes.
- Too many attributes bloat the token — consider selective inclusion or database lookups.
- Attribute sources must be authoritative — a user must not be able to modify their own clearance level.

### Annotated Code Example

```javascript
// user-attributes.js
const express = require('express');
const app = express();

// Simulated user database with attributes
const users = {
  usr_alice: {
    id: 'usr_alice',
    department: 'finance',
    clearanceLevel: 'confidential',
    employmentType: 'full-time',
    roles: ['accountant']
  },
  usr_bob: {
    id: 'usr_bob',
    department: 'engineering',
    clearanceLevel: 'internal',
    employmentType: 'contractor',
    roles: ['developer']
  }
};

// Middleware — attach user attributes to req.user
app.use((req, res, next) => {
  const userId = req.headers['x-user-id'];
  req.user = users[userId] || null;
  next();
});

// ABAC rule: only full-time finance staff with confidential clearance
app.get('/api/expenses/approve', (req, res) => {
  if (!req.user) {
    return res.status(401).json({ error: 'Authentication required' });
  }

  const { department, clearanceLevel, employmentType } = req.user;

  if (department !== 'finance') {
    return res.status(403).json({ error: 'Only finance can approve expenses' });
  }

  if (clearanceLevel !== 'confidential' && clearanceLevel !== 'secret') {
    return res.status(403).json({ error: 'Insufficient clearance' });
  }

  if (employmentType !== 'full-time') {
    return res.status(403).json({ error: 'Full-time employees only' });
  }

  res.json({ message: 'Expense approved' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/expenses/approve` with `X-User-Id: usr_alice`):**
```json
{ "message": "Expense approved" }
```

**Expected Output (for `GET /api/expenses/approve` with `X-User-Id: usr_bob`):**
```json
{ "error": "Only finance can approve expenses" }
```

**Why this output:** Alice is in finance, has confidential clearance, and is full-time — all three attribute checks pass. Bob is in engineering — the first check fails. This rule requires three attributes; in pure RBAC, you would need to create a "FinanceConfidentialFullTime" role and assign it to Alice — which does not scale when attributes change.

### Real-World Cases

- **Financial services:** Only licensed advisors with Series 7 certification can approve trades.
- **Healthcare:** Only doctors with specific specialties can prescribe certain medications.
- **Government:** Only staff with "secret" clearance and US citizenship can access classified documents.
- **SaaS:** Only workspace owners can delete the workspace, regardless of their role.

---

## Core Concept 2: Resource Attributes

### Definitions

**Core Definition:** Resource attributes (object attributes) are the properties and metadata attached to the resource being accessed — such as its classification level, owner, department, creation date, and sensitivity — that are evaluated during an access decision.

**Technical Definition:** In ABAC, the object is the resource being accessed (a file, record, API endpoint, or document). Object attributes are drawn from the resource itself or from a metadata store. Common object attributes include: `classification` (e.g., `public`, `internal`, `confidential`, `secret`), `ownerId`, `department`, `createdAt`, `sensitivity`, `status` (e.g., `draft`, `published`, `archived`), and `tags`. The policy evaluates these attributes alongside subject attributes. For example: "A user can edit a document if they are the owner OR if they are in the same department AND the document status is 'draft' AND the user has 'write' permission." Resource attributes are typically loaded alongside the resource in the repository layer.

**Beginner-Friendly Explanation:** Resource attributes are like the labels on a file folder. One folder says "CONFIDENTIAL — Finance — Created 2024." Another says "PUBLIC — Marketing — Created 2026." A smart filing system can say "you can only open this folder if your clearance level matches the folder's classification label AND you're in the same department." The labels on the folder (resource attributes) are just as important as the information on your ID card (user attributes).

### Purposes

- To evaluate metadata attached to the object being accessed (e.g., file classification status, record creation date, department ownership).
- To enforce rules based on the resource's sensitivity and ownership.
- To enable dynamic access decisions that change as the resource's state changes.
- To support classification-based access control (CBAC) models.

### Syntax Rules and Structure

#### Resource Attribute Schema

```javascript
const documentAttributes = {
  id: 'doc_456',
  classification: 'confidential',         // public | internal | confidential | secret
  ownerId: 'usr_alice',
  department: 'finance',
  status: 'draft',                        // draft | review | published | archived
  createdAt: '2026-01-15T10:30:00Z',
  sensitivity: 'high',
  tags: ['budget', '2026']
};
```

| Attribute | Type | Example Values |
|-----------|------|----------------|
| `classification` | Enum | `public`, `internal`, `confidential`, `secret` |
| `ownerId` | String | User ID of the resource owner. |
| `department` | String | Department that owns the resource. |
| `status` | Enum | `draft`, `review`, `published`, `archived` |
| `createdAt` | Date | When the resource was created. |
| `sensitivity` | Enum | `low`, `medium`, `high` |

#### Loading Resource Attributes

```javascript
async function loadDocument(req, res, next) {
  const doc = await Document.findById(req.params.id)
    .select('classification ownerId department status createdAt');
  if (!doc) return res.status(404).json({ error: 'Document not found' });
  req.document = doc;
  next();
}
```

**Rules:**
- Load resource attributes **before** the authorization check — the policy needs them.
- Never trust client-supplied resource attributes — load them from the database.
- Ensure resource attributes are complete — missing attributes should fail closed (deny).
- Cache resource attributes if they are expensive to load, but invalidate on change.

**Constraints:**
- Resource attribute loading adds a database query — combine with the main resource query.
- Stale resource attributes (e.g., a document that was reclassified) can grant unintended access.
- Classification levels must be defined consistently across users and resources.

### Annotated Code Example

```javascript
// resource-attributes.js
const express = require('express');
const app = express();

// Simulated documents with attributes
const documents = {
  doc_1: {
    id: 'doc_1',
    classification: 'public',
    ownerId: 'usr_alice',
    department: 'finance',
    status: 'published'
  },
  doc_2: {
    id: 'doc_2',
    classification: 'confidential',
    ownerId: 'usr_bob',
    department: 'engineering',
    status: 'draft'
  }
};

// Clearance hierarchy: public < internal < confidential < secret
const CLEARANCE_LEVELS = { public: 0, internal: 1, confidential: 2, secret: 3 };

app.use((req, res, next) => {
  req.user = {
    id: req.headers['x-user-id'],
    clearanceLevel: req.headers['x-clearance'] || 'public',
    department: req.headers['x-department']
  };
  next();
});

// ABAC: clearance level must meet or exceed document classification
app.get('/api/documents/:id', (req, res) => {
  const doc = documents[req.params.id];
  if (!doc) return res.status(404).json({ error: 'Document not found' });

  const userLevel = CLEARANCE_LEVELS[req.user.clearanceLevel] ?? -1;
  const docLevel = CLEARANCE_LEVELS[doc.classification] ?? 99;

  if (userLevel < docLevel) {
    return res.status(403).json({
      error: 'Insufficient clearance',
      required: doc.classification,
      granted: req.user.clearanceLevel
    });
  }

  res.json({ data: doc });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/documents/doc_1` with `X-Clearance: public`):**
```json
{ "data": { "id": "doc_1", "classification": "public", ... } }
```

**Expected Output (for `GET /api/documents/doc_2` with `X-Clearance: public`):**
```json
{
  "error": "Insufficient clearance",
  "required": "confidential",
  "granted": "public"
}
```

**Why this output:** The document's classification level is evaluated against the user's clearance level. Doc 1 is public — a public clearance is sufficient. Doc 2 is confidential — a public clearance is insufficient. The clearance hierarchy (public < internal < confidential < secret) determines the comparison.

### Real-World Cases

- **Document management:** Classification-based access (public, internal, confidential, secret).
- **Healthcare:** Patient records accessible only to staff in the same department.
- **Legal:** Case files accessible only to the assigned legal team.
- **Financial services:** Trade records accessible only to the trading desk that created them.

---

## Core Concept 3: Contextual Authorization (Environment)

### Definitions

**Core Definition:** Contextual authorization evaluates environment attributes — the circumstances surrounding the access request, such as time of day, geographic location, network subnet, and request rate — to make access decisions that depend on the current context rather than just the user and resource.

**Technical Definition:** Environmental attributes (also called context attributes) are dynamic properties of the request that are not attached to the subject or the object. They include: `time` (current time, day of week, business hours), `location` (IP address, geolocation, country), `network` (subnet, VPN status, internal vs. external), `device` (user agent, device type, managed vs. unmanaged), `requestRate` (requests per minute, anomaly detection), and `riskScore` (computed from behavioural signals). Policies can express rules like "access is allowed only during business hours (09:00–18:00) from the corporate network" or "high-risk requests require additional authentication." Contextual attributes are evaluated at request time and can change between requests from the same user.

**Beginner-Friendly Explanation:** Contextual authorization is like a smart door that considers the circumstances: "Is it during business hours? Is the request coming from the office network? Is this the user's usual device? Has the user made an unusual number of requests?" The same user might be allowed access at 10 AM from the office network but denied at 2 AM from a foreign IP address. The user hasn't changed — the context has.

### Purposes

- To evaluate execution realities (e.g., time of day, geographic IP location, network subnet boundaries, current request rate limit).
- To implement zero-trust security models where every request is evaluated in context.
- To detect and respond to anomalous access patterns.
- To enforce location-based and time-based access policies.

### Syntax Rules and Structure

#### Context Attributes

```javascript
const contextAttributes = {
  time: new Date(),
  hour: new Date().getHours(),
  dayOfWeek: new Date().getDay(),
  ip: req.ip,
  country: geoip.lookup(req.ip)?.country,
  isInternalNetwork: isPrivateIP(req.ip),
  userAgent: req.get('User-Agent'),
  isManagedDevice: req.get('X-Managed-Device') === 'true',
  requestRate: await getRateLimit(req.user.id),
  riskScore: await computeRiskScore(req)
};
```

| Attribute | Type | Example Values |
|-----------|------|----------------|
| `hour` | Number | 0–23 |
| `dayOfWeek` | Number | 0 (Sunday) – 6 (Saturday) |
| `country` | String | `US`, `DE`, `SG` |
| `isInternalNetwork` | Boolean | `true` / `false` |
| `isManagedDevice` | Boolean | `true` / `false` |
| `requestRate` | Number | Requests per minute |
| `riskScore` | Number | 0–100 |

#### Policy with Context

```javascript
function canAccessResource(user, resource, context) {
  // Rule: business hours only for confidential resources
  if (resource.classification === 'confidential') {
    const isBusinessHours = context.hour >= 9 && context.hour < 18;
    const isWeekday = context.dayOfWeek >= 1 && context.dayOfWeek <= 5;
    if (!isBusinessHours || !isWeekday) return false;
  }

  // Rule: internal network required for secret resources
  if (resource.classification === 'secret' && !context.isInternalNetwork) {
    return false;
  }

  // Rule: high-risk requests require managed device
  if (context.riskScore > 70 && !context.isManagedDevice) {
    return false;
  }

  return true;
}
```

**Rules:**
- Evaluate context attributes **at request time** — they change between requests.
- Use context to **augment** (not replace) subject and resource attributes.
- Log context attributes alongside the access decision for auditability.
- Handle missing context attributes by **failing closed** (deny access).
- Rate-limit context evaluation — computing a risk score on every request adds latency.

**Constraints:**
- Geolocation data can be inaccurate — IP-based geolocation has error margins.
- Time-based rules can lock out legitimate users in different time zones.
- Risk scores require behavioural baselines — initial deployment may have high false positives.

### Annotated Code Example

```javascript
// contextual-auth.js
const express = require('express');
const app = express();

// Simulated resource
const resources = {
  res_1: { id: 'res_1', classification: 'public' },
  res_2: { id: 'res_2', classification: 'confidential' },
  res_3: { id: 'res_3', classification: 'secret' }
};

function isPrivateIP(ip) {
  return ip.startsWith('10.') || ip.startsWith('192.168.') || ip === '::1';
}

app.use((req, res, next) => {
  req.context = {
    hour: new Date().getHours(),
    dayOfWeek: new Date().getDay(),
    ip: req.ip,
    isInternalNetwork: isPrivateIP(req.ip),
    requestRate: parseInt(req.headers['x-request-rate'] || '0')
  };
  next();
});

app.get('/api/resources/:id', (req, res) => {
  const resource = resources[req.params.id];
  if (!resource) return res.status(404).json({ error: 'Not found' });

  const ctx = req.context;

  // Rule: confidential resources only during business hours
  if (resource.classification === 'confidential') {
    const isBusinessHours = ctx.hour >= 9 && ctx.hour < 18;
    const isWeekday = ctx.dayOfWeek >= 1 && ctx.dayOfWeek <= 5;
    if (!isBusinessHours || !isWeekday) {
      return res.status(403).json({
        error: 'Confidential resources accessible only during business hours'
      });
    }
  }

  // Rule: secret resources only from internal network
  if (resource.classification === 'secret' && !ctx.isInternalNetwork) {
    return res.status(403).json({
      error: 'Secret resources accessible only from internal network'
    });
  }

  // Rule: rate limit
  if (ctx.requestRate > 100) {
    return res.status(429).json({ error: 'Rate limit exceeded' });
  }

  res.json({ data: resource, context: ctx });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/resources/res_2` during business hours):**
```json
{ "data": { "id": "res_2", "classification": "confidential" }, "context": { ... } }
```

**Expected Output (for `GET /api/resources/res_2` at 2 AM):**
```json
{ "error": "Confidential resources accessible only during business hours" }
```

**Expected Output (for `GET /api/resources/res_3` from external IP):**
```json
{ "error": "Secret resources accessible only from internal network" }
```

**Why this output:** The same user accessing the same resource gets different results depending on the context. Confidential resources require business hours; secret resources require the internal network; rate limits apply to all requests. The context attributes are evaluated dynamically at request time.

### Real-World Cases

- **Banking:** High-value transfers allowed only during business hours from known locations.
- **Healthcare:** Patient records accessible only from hospital network during shifts.
- **Government:** Classified documents accessible only from secure facilities.
- **SaaS:** Admin actions allowed only from the corporate VPN.

---

## Core Concept 4: Policy-Based Access Control (PBAC) & Policy Engines

### Definitions

**Core Definition:** Policy-Based Access Control (PBAC) is the architectural pattern of centralising complex authorization rules into a logical business engine — either embedded in the application or decoupled as an external service — that evaluates requests against policies and returns allow/deny decisions.

**Technical Definition:** A policy engine is a component that evaluates a policy against a request context (subject, resource, action, environment) and returns a decision. Engines can be **embedded** (running in the same process as the application, e.g., CASL, Casbin) or **external** (running as a separate service, e.g., Cerbos, Open Policy Agent). Embedded engines are faster (no network hop) but tie the policy to the application's language and deployment. External engines provide language-agnostic, centralised policy management but add latency and an operational dependency. **CASL** is an isomorphic JavaScript authorization library — the same ability definitions work on both client and server. **Cerbos** is a stateless authorization service with YAML-based policies, designed for decentralised decision-making. **Open Policy Agent (OPA)** is a general-purpose policy engine using the Rego language, widely adopted in cloud-native environments.

**Beginner-Friendly Explanation:** A policy engine is like a company's HR department. Instead of every manager making their own decisions about who can do what, there's one central rulebook (the policy) and one department (the engine) that evaluates every request against the rulebook. The manager asks "Can Alice approve this expense?" and the HR department checks the rules and says yes or no. This ensures consistency — every manager follows the same rules.

### Purposes

- To centralise complex authorization rules into logical business engines or decoupled services.
- To decouple policy from code — policies can be updated without redeploying the application.
- To provide consistent authorization across microservices and languages.
- To enable policy testing, versioning, and auditing as first-class concerns.

### Sub-Feature 4.1: CASL (Embedded, Isomorphic)

#### Syntax Rules and Structure

```javascript
const { AbilityBuilder, Ability } = require('@casl/ability');

function defineAbilityFor(user) {
  const { can, cannot, build } = new AbilityBuilder(Ability);

  if (user.roles.includes('admin')) {
    can('manage', 'all');                  // Admin can do anything
  } else {
    can('read', 'Post');
    can('update', 'Post', { authorId: user.id });  // Only own posts
    can('delete', 'Post', { authorId: user.id });
    cannot('delete', 'Post', { published: true }); // Cannot delete published
  }

  return build();
}

// Check ability
const ability = defineAbilityFor(req.user);
if (ability.can('update', post)) {
  // Allowed
}
```

| Component | Breakdown |
|-----------|-----------|
| `can(action, subject, conditions)` | Grants permission with optional conditions. |
| `cannot(action, subject, conditions)` | Explicitly denies permission. |
| `ability.can(action, subject)` | Checks if the action is allowed. |

**Rules:**
- Use CASL for JavaScript/TypeScript applications where policy logic is simple enough to embed.
- Define abilities once and share between client and server (isomorphic).
- Use conditions (`{ authorId: user.id }`) for ownership checks.
- `cannot` rules override `can` rules — use for exceptions.

---

### Sub-Feature 4.2: Cerbos (External, Stateless)

#### Syntax Rules and Structure

```yaml
# cerbos/policies/resource_policies/document.yaml
apiVersion: api.cerbos.dev/v1
resourcePolicy:
  version: "default"
  resource: "document"
  rules:
    - actions: ["view"]
      effect: EFFECT_ALLOW
      roles: ["user"]
      condition:
        match:
          expr: request.resource.attr.classification == "public"

    - actions: ["edit"]
      effect: EFFECT_ALLOW
      roles: ["user"]
      condition:
        match:
          expr: request.resource.attr.ownerId == request.principal.id

    - actions: ["delete"]
      effect: EFFECT_ALLOW
      roles: ["admin"]
```

```javascript
// Check permission with Cerbos SDK
const { Cerbos } = require('@cerbos/http');
const cerbos = new Cerbos('http://localhost:3592');

const decision = await cerbos.checkResource({
  principal: {
    id: req.user.id,
    roles: req.user.roles,
    attr: { department: req.user.department }
  },
  resource: {
    kind: 'document',
    id: req.params.id,
    attr: { ownerId: doc.ownerId, classification: doc.classification }
  },
  actions: ['edit']
});

if (decision.isAllowed('edit')) {
  // Allowed
}
```

| Component | Breakdown |
|-----------|-----------|
| `resourcePolicy` | YAML policy defining rules for a resource type. |
| `actions` | The actions the rule applies to. |
| `roles` | The roles the rule applies to. |
| `condition` | CEL expression evaluating attributes. |

**Rules:**
- Use Cerbos when policies need to be shared across multiple services or languages.
- Policies are versioned and deployed independently of application code.
- The `principal` and `resource` attributes are passed with each check.
- Cerbos is stateless — it evaluates each request independently.

---

### Sub-Feature 4.3: Open Policy Agent (External, General-Purpose)

#### Syntax Rules and Structure

```rego
# policy/document.rego
package document

default allow = false

# Allow if user is the owner
allow {
    input.user.id == input.resource.ownerId
}

# Allow if user is admin
allow {
    input.user.roles[_] == "admin"
}

# Allow if user has clearance >= resource classification
allow {
    clearance_levels := {"public": 0, "internal": 1, "confidential": 2, "secret": 3}
    clearance_levels[input.user.clearanceLevel] >= clearance_levels[input.resource.classification]
}
```

```javascript
// Query OPA
const response = await fetch('http://localhost:8181/v1/data/document/allow', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    input: {
      user: { id: req.user.id, roles: req.user.roles, clearanceLevel: req.user.clearanceLevel },
      resource: { ownerId: doc.ownerId, classification: doc.classification },
      action: 'read'
    }
  })
});

const { result } = await response.json();
if (result === true) {
  // Allowed
}
```

| Component | Breakdown |
|-----------|-----------|
| `package document` | Namespace for the policy. |
| `default allow = false` | Fail-closed default. |
| `allow { ... }` | Rule that evaluates to true or false. |
| `input` | The request context (user, resource, action). |

**Rules:**
- Use OPA when you need a general-purpose policy engine across diverse systems.
- Rego is a declarative language — policies are data, not code.
- OPA can be deployed as a sidecar, a central service, or a library.
- Policies are versioned and distributed via bundles.

### Annotated Code Example (CASL)

```javascript
// casl-policy.js
const express = require('express');
const { AbilityBuilder, Ability } = require('@casl/ability');
const app = express();

// Define abilities for a user
function defineAbilityFor(user) {
  const { can, cannot, build } = new AbilityBuilder(Ability);

  if (user.roles.includes('admin')) {
    can('manage', 'all');
  } else {
    can('read', 'Document');
    can('update', 'Document', { ownerId: user.id });
    can('delete', 'Document', { ownerId: user.id });
    cannot('delete', 'Document', { status: 'published' });
  }

  return build();
}

// Middleware — attach ability to request
app.use((req, res, next) => {
  req.user = {
    id: req.headers['x-user-id'],
    roles: (req.headers['x-roles'] || 'user').split(',')
  };
  req.ability = defineAbilityFor(req.user);
  next();
});

// Route with CASL check
app.put('/api/documents/:id', (req, res) => {
  const document = {
    id: req.params.id,
    ownerId: req.headers['x-doc-owner'],
    status: req.headers['x-doc-status'] || 'draft'
  };

  if (req.ability.can('update', document)) {
    return res.json({ message: 'Document updated' });
  }

  res.status(403).json({ error: 'Forbidden' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `PUT /api/documents/1` with `X-User-Id: usr_alice`, `X-Doc-Owner: usr_alice`):**
```json
{ "message": "Document updated" }
```

**Expected Output (for `PUT /api/documents/1` with `X-User-Id: usr_bob`, `X-Doc-Owner: usr_alice`):**
```json
{ "error": "Forbidden" }
```

**Why this output:** The CASL ability is defined per user. Alice can update documents she owns (`{ ownerId: user.id }`). Bob cannot update Alice's document because he is not the owner and does not have the `admin` role.

### Real-World Cases

- **CASL:** Full-stack JavaScript apps where the same policy runs on client and server.
- **Cerbos:** Microservices architectures with policies managed by a central team.
- **OPA:** Kubernetes admission control, API gateways, and multi-language environments.
- **Hybrid:** CASL for simple in-app rules; Cerbos or OPA for complex, cross-service policies.

---

## References

- NIST SP 800-162 — Guide to Attribute Based Access Control (ABAC) Definition and Considerations — https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-162.pdf
- NIST — Attribute-Based Access Control (ABAC) Overview — https://csrc.nist.gov/projects/attribute-based-access-control
- OWASP Authorization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- Open Policy Agent (OPA) Documentation — https://www.openpolicyagent.org/docs/latest/
- Cerbos Documentation — https://docs.cerbos.dev/
- CASL Documentation — https://casl.js.org/v6/en/
- @casl/ability — npm — https://www.npmjs.com/package/@casl/ability
- Casbin Documentation — https://casbin.org/
- casbin — npm — https://www.npmjs.com/package/casbin
- Permit.io — ABAC vs RBAC: A Comprehensive Guide — https://www.permit.io/blog/rbac-vs-abac
- Permit.io — What is Attribute-Based Access Control (ABAC)? — https://www.permit.io/blog/what-is-attribute-based-access-control
- Oso — Attribute-Based Access Control (ABAC) — https://www.osohq.com/academy/attribute-based-access-control-abac
- Auth0 — Attribute-Based Access Control (ABAC) — https://auth0.com/docs/manage-users/access-control/abac
- Cerbos — ABAC Explained — https://cerbos.dev/blog/attribute-based-access-control-abac
- Open Policy Agent — Rego Policy Language — https://www.openpolicyagent.org/docs/latest/policy-language/
- CEL (Common Expression Language) — https://github.com/google/cel-spec
- Zero Trust Architecture — NIST SP 800-207 — https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-207.pdf
- OWASP Top 10 (2021) — A01: Broken Access Control — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- Stack Overflow — ABAC vs RBAC in Node.js — https://stackoverflow.com/questions/31141060/
- GitHub — Cerbos Node.js SDK — https://github.com/cerbos/cerbos-sdk-javascript