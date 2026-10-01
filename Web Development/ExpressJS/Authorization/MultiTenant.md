# Multi-Tenant Authorization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Multi-tenant authorization is the practice of enforcing access control in a shared application instance that serves multiple independent customers (tenants), ensuring that every request, query, and action is scoped to the correct tenant so that no tenant can access another tenant's data.

**Technical Definition:** Multi-tenancy requires three coordinated layers: (1) **tenant identification** — safely determining which tenant a request belongs to, from subdomains, custom headers, or JWT claims; (2) **tenant-scoped data access** — automatically appending tenant filters to every database query so that cross-tenant reads and writes are structurally impossible; and (3) **tenant-aware authorization** — evaluating roles and permissions in the context of the specific tenant, so that a user who is an Admin in Tenant A is not automatically an Admin in Tenant B. Data isolation is enforced through one of several paradigms: **logical isolation** (shared database with a `tenant_id` foreign key), **schema isolation** (separate schemas within the same database), or **physical isolation** (entirely separate database instances per tenant). Global platform administrators must be distinguished from tenant-local administrators to prevent permission pollution.

**Beginner-Friendly Explanation:** Multi-tenant authorization is like an apartment building. Each tenant has their own apartment (data), their own key (credentials), and their own rules (roles). The building manager (the application) ensures that Tenant A's key doesn't open Tenant B's door. Some buildings have separate apartments on the same floor (logical isolation), some have separate floors (schema isolation), and some are entirely separate buildings (physical isolation). And the building owner (global admin) has a master key — but they must not accidentally use it to enter a tenant's apartment without cause.

### Key Characteristics

- **Tenant identification first:** Every request must be associated with exactly one tenant before any data access occurs.
- **Automatic query scoping:** Tenant filters are applied at the data layer (repository/middleware), not left to individual developers to remember.
- **Tenant-aware roles:** A user's role is always evaluated in the context of a specific tenant.
- **Isolation paradigm choice:** Logical, schema, or physical isolation — each with different trade-offs.
- **Global vs. local distinction:** Platform administrators are explicitly separated from tenant administrators.
- **Fail-closed:** If the tenant cannot be determined, the request is denied.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A working authentication system:** JWT or session-based, with tenant context included.
- **A database** (PostgreSQL, MongoDB, etc.) with tenant-aware schemas.
- **An ORM or query builder** that supports global scopes or query interception (Prisma, Sequelize, Knex).

### Related Programming Areas

- **Authorization Fundamentals:** Multi-tenant authorization is a specialised application of authorization.
- **Role-Based Access Control (RBAC):** Roles are scoped to tenants.
- **Attribute-Based Access Control (ABAC):** Tenant ID is a critical attribute in every policy.
- **Database Design:** Isolation paradigm choice affects schema design and migration strategy.
- **Broken Object Level Authorization (BOLA):** Cross-tenant access is a form of BOLA.

### Core Concepts

1. **Tenant Identification** — extracting the tenant identifier safely from subdomains, headers, or JWT claims.
2. **Tenant-Scoped Queries** — appending strict tenant filters automatically to all database operations.
3. **Data Isolation** — logical vs. schema vs. physical isolation paradigms.
4. **Tenant-Level Roles** — managing overlapping roles across tenants.
5. **Global vs. Local Scope** — separating platform administrators from tenant administrators.

---

## Core Concept 1: Tenant Identification

### Definitions

**Core Definition:** Tenant identification is the process of reliably and securely determining which tenant a request belongs to, using a trusted source that the client cannot arbitrarily manipulate.

**Technical Definition:** Tenant identification can be derived from three primary sources: **subdomain** (`tenant-a.example.com`), **custom header** (`X-Tenant-ID: tenant-a`), or **JWT claim** (`{ "tenantId": "tenant-a" }`). The subdomain approach provides natural URL separation and is easy to reason about. The header approach is simpler but requires the client to send the header on every request and is vulnerable to manipulation if not validated. The JWT claim approach is the most secure because the tenant ID is cryptographically bound to the authenticated user — the client cannot change it without invalidating the token. Regardless of the source, the identified tenant must be **validated against the user's authorised tenants** before any data access occurs.

**Beginner-Friendly Explanation:** Tenant identification is like checking which apartment a visitor is here to see. The visitor might say "I'm here for Apartment 3B" (header), or their key might already indicate which apartment it opens (JWT claim), or they might have entered through a door labelled "3B" (subdomain). The building manager must verify that the visitor is actually authorised to visit that apartment — not just take their word for it.

### Purposes

- To extract the tenant identifier safely from subdomains, custom headers (e.g., `X-Tenant-ID`), or JWT user claims.
- To ensure the tenant is determined before any data access occurs.
- To validate that the authenticated user is authorised for the identified tenant.
- To prevent tenant spoofing attacks where a client claims a different tenant.

### Syntax Rules and Structure

#### JWT Claim (Recommended)

```javascript
// Token payload
{
  "sub": "usr_123",
  "tenantId": "tenant_a",              // Primary tenant
  "tenants": ["tenant_a", "tenant_b"]  // All authorised tenants
}

// Middleware
function identifyTenant(req, res, next) {
  const tenantId = req.user.tenantId;
  if (!tenantId) {
    return res.status(400).json({ error: 'Tenant context required' });
  }
  req.tenantId = tenantId;
  next();
}
```

#### Subdomain Extraction

```javascript
function extractTenantFromSubdomain(req, res, next) {
  const host = req.hostname;                    // e.g., "tenant-a.example.com"
  const parts = host.split('.');
  const subdomain = parts.length > 2 ? parts[0] : null;

  if (!subdomain) {
    return res.status(400).json({ error: 'Tenant subdomain required' });
  }

  req.tenantId = subdomain;
  next();
}
```

#### Header Extraction (with validation)

```javascript
function extractTenantFromHeader(req, res, next) {
  const tenantId = req.get('X-Tenant-ID');
  if (!tenantId) {
    return res.status(400).json({ error: 'X-Tenant-ID header required' });
  }

  // CRITICAL: Validate the user is authorised for this tenant
  if (!req.user.tenants.includes(tenantId)) {
    return res.status(403).json({ error: 'Access to this tenant denied' });
  }

  req.tenantId = tenantId;
  next();
}
```

| Source | Security | Best For |
|--------|----------|----------|
| JWT claim | Highest — cryptographically bound | Authenticated APIs |
| Subdomain | Medium — requires DNS control | Web applications |
| Header | Lower — requires validation | Internal APIs, service-to-service |

**Rules:**
- **Validate** the identified tenant against the user's authorised tenants — never trust the client.
- Prefer JWT claims for authenticated APIs — the tenant is bound to the token.
- Use subdomains for web applications where natural URL separation is desired.
- Use headers only for internal service-to-service communication with mutual TLS.
- **Fail closed** — if the tenant cannot be determined, deny the request.
- Cache tenant validation results per request to avoid repeated lookups.

**Constraints:**
- Subdomain extraction requires wildcard DNS and TLS certificates.
- Header-based identification is vulnerable to spoofing if not validated server-side.
- JWT claims become stale if a user's tenant membership changes — use short-lived tokens.

### Annotated Code Example

```javascript
// tenant-identification.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

// Simulated user-tenant membership
const USER_TENANTS = {
  usr_alice: ['tenant_a', 'tenant_b'],
  usr_bob: ['tenant_b']
};

// Authentication middleware — sets req.user
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Authentication required' });

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Tenant identification middleware
function identifyTenant(req, res, next) {
  // Priority: JWT claim > header > subdomain
  const tenantId =
    req.user.tenantId ||
    req.get('X-Tenant-ID') ||
    req.hostname.split('.')[0];

  if (!tenantId) {
    return res.status(400).json({ error: 'Tenant context required' });
  }

  // Validate against user's authorised tenants
  const authorised = USER_TENANTS[req.user.id] || [];
  if (!authorised.includes(tenantId)) {
    return res.status(403).json({
      error: 'Access to this tenant denied',
      requested: tenantId,
      authorised
    });
  }

  req.tenantId = tenantId;
  next();
}

// Protected route
app.get('/api/data',
  authenticate,
  identifyTenant,
  (req, res) => {
    res.json({ tenantId: req.tenantId, data: 'tenant-scoped data' });
  }
);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/data` with valid token for `tenant_a`):**
```json
{ "tenantId": "tenant_a", "data": "tenant-scoped data" }
```

**Expected Output (for `GET /api/data` with `X-Tenant-ID: tenant_c` — not authorised):**
```json
{
  "error": "Access to this tenant denied",
  "requested": "tenant_c",
  "authorised": ["tenant_a", "tenant_b"]
}
```

**Why this output:** The middleware identifies the tenant from the JWT claim (highest priority), validates it against the user's authorised tenants, and attaches `req.tenantId` for downstream use. If the client attempts to access an unauthorised tenant via header, the request is denied with a 403.

### Real-World Cases

- **SaaS platforms:** `acme.myapp.com` and `globex.myapp.com` for different customers.
- **API services:** JWT claims carry the tenant ID for stateless verification.
- **Internal microservices:** `X-Tenant-ID` header propagated between services.
- **Enterprise:** Users belong to multiple tenants and switch between them.

---

## Core Concept 2: Tenant-Scoped Queries

### Definitions

**Core Definition:** Tenant-scoped queries automatically append a tenant filter to every database operation, ensuring that no query can return or modify data belonging to a different tenant — making cross-tenant data leaks structurally impossible.

**Technical Definition:** Tenant scoping is enforced at the data access layer by intercepting every query and injecting the `tenantId` condition. In an ORM like Prisma, this is implemented with **middleware** (`prisma.$use()`) that intercepts all operations. In Sequelize, **global scopes** or **hooks** append the tenant filter. In a query builder like Knex, a wrapper function adds `.where('tenant_id', tenantId)` to every query. The critical principle is that **no repository method should ever execute a query without the tenant filter** — this is enforced by the abstraction, not by developer discipline. Additionally, every table containing tenant data must have a `tenant_id` column with an index, and foreign keys must include the tenant ID to prevent cross-tenant references.

**Beginner-Friendly Explanation:** Tenant-scoped queries are like a librarian who automatically stamps "For Tenant A only" on every book check-out. The librarian doesn't rely on the tenant to remember which books are theirs — the system enforces it. Even if Tenant B somehow requests a book belonging to Tenant A, the librarian's stamp prevents the book from leaving the shelf. In code, this means every database query automatically includes `WHERE tenant_id = 'current_tenant'`.

### Purposes

- To append strict tenant filters automatically to all database operations to prevent cross-tenant data leaks.
- To make cross-tenant access structurally impossible, not just policy-prohibited.
- To eliminate the risk of a developer forgetting the tenant filter in a new query.
- To enforce tenant isolation at the lowest level of the application.

### Syntax Rules and Structure

#### Prisma Middleware (Automatic Scoping)

```javascript
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// Tenant scoping middleware
prisma.$use(async (params, next) => {
  const tenantId = asyncLocalStorage.getStore()?.get('tenantId');

  // Models that are tenant-scoped
  const TENANT_MODELS = ['User', 'Order', 'Product', 'Document'];

  if (TENANT_MODELS.includes(params.model)) {
    if (params.action === 'findMany' || params.action === 'findFirst') {
      params.args.where = { ...params.args.where, tenantId };
    }
    if (params.action === 'create') {
      params.args.data = { ...params.args.data, tenantId };
    }
    if (params.action === 'update' || params.action === 'delete') {
      params.args.where = { ...params.args.where, tenantId };
    }
  }

  return next(params);
});
```

#### Sequelize Global Scope

```javascript
const { Sequelize, Model } = require('sequelize');

class TenantModel extends Model {
  static init(options) {
    return super.init({
      ...options,
      tenantId: {
        type: Sequelize.UUID,
        allowNull: false
      }
    }, options);
  }
}

// Add default scope dynamically
TenantModel.addScope('tenant', {
  where: { tenantId: sequelize.Sequelize.literal('current_tenant_id') }
});
```

#### Repository Wrapper

```javascript
class TenantScopedRepository {
  constructor(model, tenantId) {
    this.model = model;
    this.tenantId = tenantId;
  }

  async findMany(where = {}) {
    return this.model.findMany({
      where: { ...where, tenantId: this.tenantId }
    });
  }

  async create(data) {
    return this.model.create({
      ...data,
      tenantId: this.tenantId
    });
  }

  async update(id, data) {
    return this.model.update({
      where: { id, tenantId: this.tenantId },
      data
    });
  }
}
```

| Approach | Enforces Automatically | Best For |
|----------|----------------------|----------|
| ORM middleware | ✅ Yes | Prisma, Sequelize |
| Global scopes | ✅ Yes | Sequelize |
| Repository wrapper | ⚠️ Manual | Custom query builders |
| Raw SQL | ❌ No | Avoid for tenant data |

**Rules:**
- **Every** table containing tenant data must have a `tenant_id` column.
- Apply tenant scoping via **ORM middleware or global scopes** — not manually in each query.
- Use `AsyncLocalStorage` to propagate the tenant ID to the data layer without parameter drilling.
- Include `tenant_id` in **foreign keys** to prevent cross-tenant references.
- Index `tenant_id` on every tenant-scoped table for query performance.
- **Fail closed** — if the tenant ID is missing, the query must fail, not return all data.

**Constraints:**
- ORM middleware adds a small overhead per query.
- Raw SQL queries bypass ORM middleware — use repository wrappers or disable raw SQL for tenant data.
- Foreign key constraints must include `tenant_id` — a foreign key on `id` alone allows cross-tenant references.

### Annotated Code Example

```javascript
// tenant-scoped-queries.js
const express = require('express');
const { PrismaClient } = require('@prisma/client');
const { AsyncLocalStorage } = require('node:async_hooks');

const prisma = new PrismaClient();
const asyncLocalStorage = new AsyncLocalStorage();

// Prisma middleware — automatic tenant scoping
prisma.$use(async (params, next) => {
  const tenantId = asyncLocalStorage.getStore()?.get('tenantId');
  const TENANT_MODELS = ['Order', 'Product', 'Document'];

  if (TENANT_MODELS.includes(params.model)) {
    if (!tenantId) {
      throw new Error('Tenant context required for tenant-scoped query');
    }

    if (['findMany', 'findFirst', 'findUnique'].includes(params.action)) {
      params.args = params.args || {};
      params.args.where = { ...params.args.where, tenantId };
    }

    if (params.action === 'create') {
      params.args.data = { ...params.args.data, tenantId };
    }

    if (['update', 'delete'].includes(params.action)) {
      params.args.where = { ...params.args.where, tenantId };
    }
  }

  return next(params);
});

// Middleware — run request in tenant context
app.use((req, res, next) => {
  const tenantId = req.headers['x-tenant-id'];
  if (!tenantId) return res.status(400).json({ error: 'X-Tenant-ID required' });

  const store = new Map([['tenantId', tenantId]]);
  asyncLocalStorage.run(store, () => next());
});

// Route — no explicit tenant filter needed
app.get('/api/orders', async (req, res) => {
  // The Prisma middleware automatically adds tenantId to the query
  const orders = await prisma.order.findMany();
  res.json({ orders });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/orders` with `X-Tenant-ID: tenant_a`):**
```json
{ "orders": [{ "id": "1", "tenantId": "tenant_a", "total": 99.99 }] }
```

**Expected Output (for `GET /api/orders` without `X-Tenant-ID`):**
```json
{ "error": "X-Tenant-ID required" }
```

**Why this output:** The Prisma middleware intercepts every query on tenant-scoped models and automatically adds `tenantId` to the `where` clause (for reads) or `data` (for creates). The route handler calls `prisma.order.findMany()` with no explicit filter — the middleware ensures only the current tenant's orders are returned. If no tenant context is set, the query throws an error (fail-closed).

### Real-World Cases

- **SaaS platforms:** Every query automatically scoped to the current tenant.
- **Multi-tenant databases:** Shared tables with `tenant_id` foreign keys.
- **Microservices:** Tenant context propagated via `AsyncLocalStorage` across service calls.
- **Data exports:** Exports automatically scoped to the requesting tenant.

---

## Core Concept 3: Data Isolation

### Definitions

**Core Definition:** Data isolation is the structural approach used to separate tenant data — ranging from logical isolation (shared tables with a tenant discriminator) to physical isolation (entirely separate databases per tenant).

**Technical Definition:** Three primary isolation paradigms exist. **Logical isolation** uses a shared database with a `tenant_id` column on every table — all tenants share the same schema and tables, but rows are filtered by tenant. This is the most cost-efficient and easiest to maintain, but requires rigorous query scoping and is vulnerable to a single misconfigured query leaking data. **Schema isolation** gives each tenant its own database schema (e.g., PostgreSQL schemas) within the same database instance — stronger isolation, but requires schema migration management across tenants. **Physical isolation** gives each tenant an entirely separate database instance — the strongest isolation (a breach in one tenant's database cannot affect another), but the highest cost and operational complexity. The choice depends on tenant count, data sensitivity, compliance requirements, and operational budget.

**Beginner-Friendly Explanation:** Data isolation is like choosing how to store tenants' belongings. **Logical isolation** is a shared storage unit with labelled shelves — cheap, but one mislabelled box can mix things up. **Schema isolation** is separate locked cabinets in the same room — better separation, still shared space. **Physical isolation** is separate storage units in different buildings — maximum separation, maximum cost. The right choice depends on how valuable the belongings are and how much you can afford.

### Purposes

- To choose and enforce structural data isolation paradigms — logical isolation vs. physical isolation.
- To match the isolation level to compliance requirements (HIPAA, GDPR, SOC 2).
- To balance cost, operational complexity, and security.
- To enable noisy-neighbour isolation for performance-critical tenants.

### Syntax Rules and Structure

#### Isolation Paradigm Comparison

| Dimension | Logical | Schema | Physical |
|-----------|---------|--------|----------|
| **Structure** | Shared tables + `tenant_id` | Separate schemas | Separate databases |
| **Cost** | Lowest | Medium | Highest |
| **Isolation** | Lowest | Medium | Highest |
| **Migration** | Single migration | Per-schema migration | Per-database migration |
| **Cross-tenant queries** | Easy (analytics) | Hard | Impossible |
| **Noisy neighbour** | High risk | Medium risk | No risk |
| **Compliance** | May not suffice for HIPAA | Good for most | Required for some |

#### Tenant Table (Logical Isolation)

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  user_id UUID NOT NULL,
  total DECIMAL(10, 2) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_orders_tenant ON orders(tenant_id);

-- Foreign key includes tenant_id to prevent cross-tenant references
ALTER TABLE orders
  ADD CONSTRAINT fk_orders_user
  FOREIGN KEY (user_id, tenant_id)
  REFERENCES users(id, tenant_id);
```

#### Schema Isolation (PostgreSQL)

```sql
-- Each tenant gets its own schema
CREATE SCHEMA tenant_a;
CREATE SCHEMA tenant_b;

-- Tables are created per schema
CREATE TABLE tenant_a.orders (...);
CREATE TABLE tenant_b.orders (...);
```

```javascript
// Set the search_path per request
async function setTenantSchema(req, res, next) {
  const tenantId = req.tenantId;
  await prisma.$executeRawUnsafe(
    `SET search_path TO ${tenantId.replace(/[^a-z0-9_]/gi, '')}`
  );
  next();
}
```

**Rules:**
- **Logical isolation** requires rigorous query scoping — one missed `WHERE tenant_id` leaks all data.
- **Schema isolation** requires per-schema migration management — automate with a migration runner.
- **Physical isolation** requires connection routing — each tenant has its own connection string.
- Choose based on **compliance requirements** — HIPAA and some financial regulations may require physical isolation.
- Use **logical isolation** for most SaaS applications with a shared database and tenant-aware middleware.
- Use **schema isolation** when tenants need schema customisation or stronger isolation.
- Use **physical isolation** for high-value tenants, regulated industries, or when a single tenant's load would affect others.

**Constraints:**
- Logical isolation provides the weakest isolation — a single bug can expose all tenants.
- Schema isolation complicates migrations — a migration must run across every schema.
- Physical isolation has the highest operational cost — connection pooling, backups, and monitoring must be per-tenant.
- Cross-tenant analytics are difficult with schema and physical isolation.

### Annotated Code Example

```javascript
// data-isolation.js
// LOGICAL ISOLATION — shared tables with tenant_id
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// All tenants share the same tables
// Every query is scoped by tenant_id via middleware
prisma.$use(async (params, next) => {
  const tenantId = asyncLocalStorage.getStore()?.get('tenantId');

  if (['Order', 'Product'].includes(params.model)) {
    if (['findMany', 'findFirst'].includes(params.action)) {
      params.args = params.args || {};
      params.args.where = { ...params.args.where, tenantId };
    }
  }

  return next(params);
});

// SCHEMA ISOLATION — per-tenant schema
async function getSchemaScopedClient(tenantId) {
  const client = new PrismaClient();
  await client.$executeRawUnsafe(
    `SET search_path TO "${tenantId.replace(/[^a-z0-9_]/gi, '')}"`
  );
  return client;
}

// PHYSICAL ISOLATION — per-tenant database
const TENANT_CONNECTIONS = {
  tenant_a: 'postgresql://localhost:5432/tenant_a_db',
  tenant_b: 'postgresql://localhost:5432/tenant_b_db'
};

function getPhysicalClient(tenantId) {
  const url = TENANT_CONNECTIONS[tenantId];
  if (!url) throw new Error(`No connection for tenant ${tenantId}`);
  return new PrismaClient({ datasources: { db: { url } } });
}
```

**Expected Output (logical isolation — query for tenant_a):**
```json
{ "orders": [{ "id": "1", "tenant_id": "tenant_a", "total": 99.99 }] }
```

**Expected Output (schema isolation — query for tenant_a):**
```json
{ "orders": [{ "id": "1", "total": 99.99 }] }
```

**Expected Output (physical isolation — query for tenant_a):**
```json
{ "orders": [{ "id": "1", "total": 99.99 }] }
```

**Why this output:** Logical isolation returns the `tenant_id` column (it is part of the shared table). Schema and physical isolation do not include `tenant_id` in the result because the isolation is structural — the data is already separated by schema or database.

### Real-World Cases

- **Logical isolation:** Most SaaS platforms (Slack, Notion, GitHub) use shared tables with tenant IDs.
- **Schema isolation:** Enterprise platforms where tenants need custom fields or schema extensions.
- **Physical isolation:** Healthcare (HIPAA), financial services, and government systems.
- **Hybrid:** Logical isolation for most tenants; physical isolation for high-value enterprise tenants.

---

## Core Concept 4: Tenant-Level Roles

### Definitions

**Core Definition:** Tenant-level roles are role assignments that are scoped to a specific tenant — meaning a user can hold different roles in different tenants, such as Admin in Tenant A but standard User in Tenant B.

**Technical Definition:** In a multi-tenant system, roles are not global — they are evaluated in the context of the tenant the user is currently accessing. A user's role is stored per tenant in a join table (`user_tenant_roles`) that maps `(userId, tenantId) → role`. The authorization middleware checks the user's role **for the current tenant** — not a global role. This prevents a user who is an Admin in one tenant from accidentally having Admin privileges in another tenant. The tenant context is established by the tenant identification middleware, and the role check reads the role for that specific tenant.

**Beginner-Friendly Explanation:** Tenant-level roles are like having different job titles at different companies. You might be a Manager at Company A but an Intern at Company B. Your Manager badge at Company A doesn't give you Manager access at Company B. In a multi-tenant system, roles are always scoped to the tenant — the same person can be an Admin in one workspace and a regular member in another.

### Purposes

- To manage overlapping roles where a user is an Admin in Tenant A but a standard User in Tenant B.
- To prevent permission pollution across tenants.
- To enable users to participate in multiple tenants with different levels of access.
- To support cross-tenant collaboration (a user who is an Admin in one tenant may need read access to another).

### Syntax Rules and Structure

#### User-Tenant-Role Schema

```javascript
const userTenantRoleSchema = new mongoose.Schema({
  userId: { type: String, required: true, index: true },
  tenantId: { type: String, required: true, index: true },
  role: { type: String, required: true },
  grantedAt: { type: Date, default: Date.now }
});

userTenantRoleSchema.index({ userId: 1, tenantId: 1 }, { unique: true });
```

#### Tenant-Aware Role Middleware

```javascript
function checkTenantRole(...allowedRoles) {
  return async (req, res, next) => {
    if (!req.user || !req.tenantId) {
      return res.status(401).json({ error: 'Authentication and tenant context required' });
    }

    // Look up the user's role in THIS tenant
    const assignment = await UserTenantRole.findOne({
      userId: req.user.id,
      tenantId: req.tenantId
    });

    if (!assignment || !allowedRoles.includes(assignment.role)) {
      return res.status(403).json({
        error: 'Insufficient permissions in this tenant',
        tenant: req.tenantId,
        required: allowedRoles
      });
    }

    req.tenantRole = assignment.role;
    next();
  };
}

// Usage
router.delete('/api/tenant/settings',
  authenticate,
  identifyTenant,
  checkTenantRole('admin', 'owner'),
  tenantController.deleteSettings
);
```

| Component | Breakdown |
|-----------|-----------|
| `userId + tenantId` | Composite key for the role assignment. |
| `role` | The user's role within this tenant. |
| `checkTenantRole(...)` | Middleware factory checking the tenant-scoped role. |

**Rules:**
- Store role assignments per `(userId, tenantId)` pair — never globally.
- Always check the role in the context of the current tenant (`req.tenantId`).
- Use a composite unique index on `(userId, tenantId)` to prevent duplicates.
- Include the tenant ID in the JWT for stateless verification, or look it up from the database.
- **Never** reuse a global role for tenant-scoped decisions.

**Constraints:**
- Looking up the tenant role on every request adds a database query — cache the result per request.
- JWT-embedded roles become stale if changed — use short-lived tokens or database lookups.
- Users with access to many tenants need efficient role lookup — index `(userId, tenantId)`.

### Annotated Code Example

```javascript
// tenant-roles.js
const express = require('express');
const app = express();

// Simulated role assignments
const ROLE_ASSIGNMENTS = [
  { userId: 'usr_alice', tenantId: 'tenant_a', role: 'admin' },
  { userId: 'usr_alice', tenantId: 'tenant_b', role: 'user' },
  { userId: 'usr_bob', tenantId: 'tenant_b', role: 'admin' }
];

// Middleware — authenticate
app.use((req, res, next) => {
  req.user = { id: req.headers['x-user-id'] };
  req.tenantId = req.headers['x-tenant-id'];
  next();
});

// Tenant-aware role check
function checkTenantRole(...allowedRoles) {
  return (req, res, next) => {
    const assignment = ROLE_ASSIGNMENTS.find(
      a => a.userId === req.user.id && a.tenantId === req.tenantId
    );

    if (!assignment) {
      return res.status(403).json({
        error: 'No access to this tenant'
      });
    }

    if (!allowedRoles.includes(assignment.role)) {
      return res.status(403).json({
        error: 'Insufficient permissions in this tenant',
        tenant: req.tenantId,
        role: assignment.role,
        required: allowedRoles
      });
    }

    req.tenantRole = assignment.role;
    next();
  };
}

// Route — admin access in tenant A only
app.delete('/api/tenant/settings',
  checkTenantRole('admin', 'owner'),
  (req, res) => {
    res.json({
      message: 'Settings deleted',
      tenant: req.tenantId,
      role: req.tenantRole
    });
  }
);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `DELETE /api/tenant/settings` with `X-User-Id: usr_alice`, `X-Tenant-Id: tenant_a`):**
```json
{ "message": "Settings deleted", "tenant": "tenant_a", "role": "admin" }
```

**Expected Output (for the same user in `tenant_b`):**
```json
{
  "error": "Insufficient permissions in this tenant",
  "tenant": "tenant_b",
  "role": "user",
  "required": ["admin", "owner"]
}
```

**Why this output:** Alice is an Admin in Tenant A but a User in Tenant B. The `checkTenantRole` middleware looks up her role **for the current tenant** — she passes in Tenant A but fails in Tenant B. Her admin role in Tenant A does not leak into Tenant B.

### Real-World Cases

- **SaaS platforms:** Users are Admins in their own workspace but Viewers in shared workspaces.
- **Agencies:** An agency user manages multiple client tenants with different access levels.
- **Marketplaces:** Buyers and sellers have different roles in different transaction contexts.
- **Enterprise:** Employees have different roles in different departments' tenants.

---

## Core Concept 5: Global vs. Local Scope

### Definitions

**Core Definition:** Global scope refers to platform-level administrators who manage the entire application across all tenants; local scope refers to tenant-level administrators who manage only their own tenant — and the two must be explicitly separated to prevent permission pollution.

**Technical Definition:** Global administrators (platform admins, superadmins, support engineers) operate **outside** the tenant context — they can manage tenants, handle support requests, and perform system maintenance. Local administrators (tenant admins, owners) operate **inside** a tenant context — they manage users, settings, and data within their tenant. The key design principle is that global roles are **not** tenant-scoped and tenant roles are **not** global. A global admin should not automatically have access to every tenant's data without an explicit support context (e.g., "support session" with audit logging). Conversely, a tenant admin should never be able to perform platform-level actions (e.g., creating new tenants). The authorization middleware must distinguish between global routes (`/admin/*`) and tenant routes (`/api/*`), and the role checks must not conflate the two.

**Beginner-Friendly Explanation:** Global scope is like the building owner who has a master key to every apartment. Local scope is like a tenant who has a key to their own apartment. The owner can enter any apartment, but should only do so with a legitimate reason (a repair, an emergency) and should log every entry. The tenant can enter their own apartment freely but cannot enter anyone else's. The danger is when the owner's master key gets copied (permission pollution) or when a tenant somehow gets a master key.

### Purposes

- To manage global application platform administrators alongside local tenant-specific administrators safely without permission pollution.
- To prevent tenant admins from performing platform-level actions.
- To prevent global admins from accidentally accessing tenant data without an explicit support context.
- To provide audit trails for global admin access to tenant data.

### Syntax Rules and Structure

#### Separate Route Namespaces

```javascript
// Global admin routes — no tenant context
app.use('/admin',
  authenticate,
  requireGlobalRole('platform_admin'),
  adminRouter
);

// Tenant routes — tenant context required
app.use('/api',
  authenticate,
  identifyTenant,
  requireTenantRole('admin', 'user'),
  tenantRouter
);
```

#### Global Role Check

```javascript
function requireGlobalRole(...roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const globalRoles = req.user.globalRoles || [];
    if (!roles.some(r => globalRoles.includes(r))) {
      return res.status(403).json({
        error: 'Platform-level access required',
        required: roles
      });
    }

    next();
  };
}
```

#### Support Session (Global Admin Accessing Tenant Data)

```javascript
async function startSupportSession(req, res, next) {
  const { tenantId, reason } = req.body;

  // Log the support session
  await SupportAudit.create({
    adminId: req.user.id,
    tenantId,
    reason,
    startedAt: new Date()
  });

  // Issue a short-lived support token scoped to the tenant
  const supportToken = jwt.sign(
    { sub: req.user.id, tenantId, scope: 'support' },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );

  res.json({ supportToken });
}
```

| Scope | Route | Role Check | Access |
|-------|-------|-----------|--------|
| Global | `/admin/*` | `requireGlobalRole('platform_admin')` | Platform-wide |
| Support | `/admin/support/*` | `requireGlobalRole('support')` + audit | Specific tenant |
| Tenant | `/api/*` | `checkTenantRole('admin')` | Own tenant |

**Rules:**
- Use **separate route namespaces** for global and tenant routes (`/admin` vs. `/api`).
- Global roles are stored in `req.user.globalRoles`; tenant roles in the tenant-role table.
- Global admins must start a **support session** before accessing tenant data — with audit logging.
- **Never** grant global admin permissions through tenant roles.
- **Never** grant tenant admin permissions through global roles.
- Audit every global admin action that touches tenant data.

**Constraints:**
- Support sessions add friction for administrators — balance security with operability.
- Global admin roles are high-value targets — require MFA and step-up authentication.
- Audit logs must be immutable and stored separately from tenant data.

### Annotated Code Example

```javascript
// global-vs-local.js
const express = require('express');
const app = express();
app.use(express.json());

// Simulated users
const USERS = {
  usr_platform_admin: { id: 'usr_platform_admin', globalRoles: ['platform_admin'] },
  usr_alice: { id: 'usr_alice', globalRoles: [] },
  usr_bob: { id: 'usr_bob', globalRoles: [] }
};

// Simulated tenant roles
const TENANT_ROLES = [
  { userId: 'usr_alice', tenantId: 'tenant_a', role: 'admin' },
  { userId: 'usr_bob', tenantId: 'tenant_b', role: 'admin' }
];

app.use((req, res, next) => {
  req.user = USERS[req.headers['x-user-id']] || null;
  req.tenantId = req.headers['x-tenant-id'];
  next();
});

// Global role check
function requireGlobalRole(...roles) {
  return (req, res, next) => {
    if (!req.user) return res.status(401).json({ error: 'Authentication required' });
    const hasRole = roles.some(r => req.user.globalRoles?.includes(r));
    if (!hasRole) {
      return res.status(403).json({ error: 'Platform-level access required' });
    }
    next();
  };
}

// Tenant role check
function checkTenantRole(...roles) {
  return (req, res, next) => {
    const assignment = TENANT_ROLES.find(
      a => a.userId === req.user?.id && a.tenantId === req.tenantId
    );
    if (!assignment || !roles.includes(assignment.role)) {
      return res.status(403).json({ error: 'Insufficient tenant permissions' });
    }
    next();
  };
}

// GLOBAL ROUTE — platform admin only
app.get('/admin/tenants',
  requireGlobalRole('platform_admin'),
  (req, res) => res.json({ tenants: ['tenant_a', 'tenant_b'] })
);

// TENANT ROUTE — tenant admin only
app.get('/api/settings',
  checkTenantRole('admin'),
  (req, res) => res.json({ settings: 'tenant settings' })
);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /admin/tenants` with `X-User-Id: usr_platform_admin`):**
```json
{ "tenants": ["tenant_a", "tenant_b"] }
```

**Expected Output (for `GET /admin/tenants` with `X-User-Id: usr_alice`):**
```json
{ "error": "Platform-level access required" }
```

**Expected Output (for `GET /api/settings` with `X-User-Id: usr_alice`, `X-Tenant-Id: tenant_a`):**
```json
{ "settings": "tenant settings" }
```

**Expected Output (for `GET /api/settings` with `X-User-Id: usr_platform_admin`, `X-Tenant-Id: tenant_a`):**
```json
{ "error": "Insufficient tenant permissions" }
```

**Why this output:** The platform admin has global access to `/admin/tenants` but **no tenant-scoped role** — they cannot access `/api/settings` without a tenant role assignment. Alice has a tenant admin role but no global role — she cannot access `/admin/tenants`. This separation prevents permission pollution in both directions.

### Real-World Cases

- **SaaS platforms:** Platform admins manage tenants; tenant admins manage their workspace.
- **Support teams:** Support engineers use audited support sessions to access tenant data.
- **Marketplaces:** Platform operators manage the marketplace; sellers manage their storefronts.
- **Enterprise:** IT administrators manage the platform; department heads manage their teams.

---

## References

- OWASP Authorization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- OWASP API Security Top 10 (2023) — API1: Broken Object Level Authorization — https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
- AWS — Multi-Tenant SaaS Architecture — https://docs.aws.amazon.com/whitepapers/latest/multi-tenant-saas-storage-strategies/multi-tenant-saas-storage-strategies.html
- Microsoft Azure — Multi-Tenant SaaS Architecture — https://learn.microsoft.com/en-us/azure/architecture/guide/multitenant/overview
- Prisma — Middleware (Tenant Scoping) — https://www.prisma.io/docs/orm/prisma-client/client-extensions/middleware
- Prisma — Multi-Tenancy with Row-Level Security — https://www.prisma.io/docs/orm/prisma-client/client-extensions/middleware#multi-tenancy
- Sequelize — Scopes — https://sequelize.org/docs/v6/other-topics/scopes/
- Postgres — Row Security Policies (RLS) — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- Auth0 — Multi-Tenant Applications Best Practices — https://auth0.com/docs/get-started/architecture-scenarios/multi-tenant-applications
- Permit.io — Multi-Tenant Authorization — https://www.permit.io/blog/multi-tenant-authorization
- Oso — Multi-Tenancy in Authorization — https://www.osohq.com/academy/multi-tenancy
- Cerbos — Multi-Tenancy Patterns — https://cerbos.dev/blog/multi-tenancy-authorization-patterns
- WorkOS — Multi-Tenant SaaS Architecture Guide — https://workos.com/blog/multi-tenant-saas-architecture
- CWE-566 — Authorization Bypass Through User-Controlled SQL Primary Key — https://cwe.mitre.org/data/definitions/566.html
- CWE-639 — Authorization Bypass Through User-Controlled Key — https://cwe.mitre.org/data/definitions/639.html
- NIST SP 800-162 — Guide to Attribute Based Access Control (ABAC) — https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-162.pdf
- Stack Overflow — Multi-Tenant Data Isolation in Node.js — https://stackoverflow.com/questions/55726961/
- GitHub — Multi-Tenant SaaS Starter Kits (Node.js) — https://github.com/search?q=multi-tenant+express+prisma