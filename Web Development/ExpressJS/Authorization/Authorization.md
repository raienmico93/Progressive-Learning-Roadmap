# Authorization Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Authorization (AuthZ) is the process of determining what an authenticated user is permitted to do — which resources they may access and which actions they may perform — after their identity has been verified.

**Technical Definition:** Authorization is distinct from authentication: authentication (AuthN) verifies *who* the user is, while authorization determines *what* the user is allowed to do. Authorization decisions are made by evaluating the authenticated principal (user or service), the requested resource, and the requested action against a policy — typically expressed as roles, permissions, or attribute-based rules. **Broken Object Level Authorization (BOLA)**, also known as **Insecure Direct Object Reference (IDOR)**, is the #1 risk in the OWASP API Security Top 10 (2023) and occurs when an API accepts a user-supplied identifier and returns or modifies a record the user does not own. The **fail-closed principle** requires that authorization middleware default to denying access when a configuration error, missing parameter, or runtime exception occurs — never default to allowing access.

**Beginner-Friendly Explanation:** Authentication is showing your ID at the airport. Authorization is what your boarding pass lets you do — which gate you can enter, which seat you can sit in, and which areas are off-limits. A passenger (authenticated user) with a first-class ticket (role) can enter the lounge, but an economy passenger cannot. And if the boarding pass scanner breaks (an error), the safe default is to deny entry, not to wave everyone through.

### Key Characteristics

- **Decision after identity:** Authorization always follows successful authentication — you must know who the user is before deciding what they can do.
- **Granular permissions:** Permissions describe specific actions on specific resources (`read:users`, `write:orders`) rather than broad role checks.
- **Resource ownership:** Authorisation must verify not just the user's role but their relationship to the specific record being accessed.
- **BOLA/IDOR mitigation:** Every endpoint that accepts a resource ID must verify the requesting user's authority over that specific resource.
- **Fail-closed default:** Deny access when anything is uncertain — missing parameters, configuration errors, or exceptions.
- **Defence in depth:** Enforce at the route level (middleware) and again at the data level (repository query scoping).

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A working authentication system:** JWT or session-based, with `req.user` populated.
- **Basic JavaScript knowledge:** Functions, closures, and middleware concepts.
- **Understanding of HTTP status codes:** 401 (unauthenticated), 403 (unauthorised), 404 (not found).

### Related Programming Areas

- **Authentication:** The identity verification step that precedes authorisation.
- **RBAC and ABAC:** Role-Based and Attribute-Based Access Control models.
- **API Security:** BOLA/IDOR is the top API security risk.
- **Error Handling:** Distinguishing 401 from 403 and avoiding information leakage.
- **Data Access Layer:** Repository queries must be scoped by user/tenant to prevent BOLA.

### Core Concepts

1. **Authentication vs. Authorization** — identity verification vs. access privileges.
2. **Permission Checks** — granular action boundaries (`read:users`, `write:orders`).
3. **Resource Ownership** — verifying the user owns the specific record.
4. **Broken Object Level Authorization (BOLA/IDOR)** — architectural guardrails.
5. **Fail-Closed Principle** — default to denying access on uncertainty.

---

## Core Concept 1: Authentication vs. Authorization

### Definitions

**Core Definition:** Authentication verifies *who* a user is (identity); authorization determines *what* that verified user is allowed to do (privileges).

**Technical Definition:** Authentication establishes a trusted identity claim through credentials (password, MFA, passkeys) and produces a session or token. Authorization consumes that identity claim — typically reading `req.user` — and evaluates it against a policy to decide whether the requested action on the requested resource should be permitted. Authentication failures return **401 Unauthorized** (the server doesn't know who you are); authorization failures return **403 Forbidden** (the server knows who you are but you lack permission). Conflating the two leads to security holes: an endpoint that checks only authentication but not authorization allows any logged-in user to access any resource.

**Beginner-Friendly Explanation:** Authentication is the bouncer checking your ID at the door — verifying you are who you say you are. Authorization is the usher inside the building checking your ticket — deciding which sections you can enter. The bouncer (authentication) doesn't care which concert you're here for; the usher (authorization) checks whether your ticket (permissions) grants access to the VIP section (resource).

### Purposes

- To dissect the boundary between identity verification (who you are) and access privileges (what you are allowed to do).
- To ensure both checks are performed — authentication alone is insufficient.
- To return the correct HTTP status code for each failure type (401 vs. 403).
- To prevent the common anti-pattern of treating authentication as authorisation.

### Syntax Rules and Structure

```javascript
// Authentication middleware — WHO are you?
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// Authorization middleware — WHAT can you do?
function authorize(...permissions) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    const hasPermission = permissions.every(p => req.user.permissions?.includes(p));
    if (!hasPermission) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Usage — both checks applied
router.delete('/users/:id',
  authenticate,                          // AuthN: who are you?
  authorize('delete:users'),             // AuthZ: can you do this?
  userController.delete
);
```

| Check | Question | Failure Status | Middleware |
|-------|----------|---------------|------------|
| Authentication | Who are you? | 401 Unauthorized | `authenticate` |
| Authorization | What can you do? | 403 Forbidden | `authorize(...)` |

**Rules:**
- Authentication must always run **before** authorization — you cannot authorise an unknown user.
- Use **401** for missing or invalid credentials; use **403** for valid credentials with insufficient permissions.
- Every protected endpoint must apply **both** authentication and authorization.
- Never rely on the client to enforce authorization — always enforce server-side.

**Constraints:**
- A user can be authenticated but have no permissions (403) — this is a valid state.
- Authorization logic must not reveal whether a resource exists if the user lacks access (use 404 to hide existence).

### Annotated Code Example

```javascript
// authn-vs-authz.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();
app.use(express.json());

// Authentication middleware
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

// Authorization middleware
function authorize(...permissions) {
  return (req, res, next) => {
    if (!req.user) return res.status(401).json({ error: 'Authentication required' });

    const hasAll = permissions.every(p => req.user.permissions?.includes(p));
    if (!hasAll) return res.status(403).json({ error: 'Insufficient permissions' });

    next();
  };
}

// Protected route — both checks
app.delete('/api/users/:id',
  authenticate,
  authorize('delete:users'),
  (req, res) => {
    res.json({ message: `User ${req.params.id} deleted` });
  }
);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (no token → 401):**
```json
{ "error": "Authentication required" }
```

**Expected Output (valid token without `delete:users` → 403):**
```json
{ "error": "Insufficient permissions" }
```

**Expected Output (valid token with `delete:users` → success):**
```json
{ "message": "User 42 deleted" }
```

**Why this output:** The `authenticate` middleware runs first — if no valid token is present, it returns 401 and the request never reaches authorization. If a valid token is present but lacks the `delete:users` permission, `authorize` returns 403. Only when both checks pass does the route handler execute.

### Real-World Cases

- **Admin panels:** Authenticated users without the `admin` role receive 403.
- **API gateways:** Authentication at the edge; authorization per-service.
- **Multi-tenant SaaS:** Authentication establishes identity; authorization scopes to the tenant.

---

## Core Concept 2: Permission Checks

### Definitions

**Core Definition:** Permission checks enforce granular action boundaries by verifying that the authenticated user possesses the specific permission required for the requested operation, rather than relying on broad role checks.

**Technical Definition:** Permissions are fine-grained capabilities expressed as `action:resource` pairs (e.g., `read:users`, `write:orders`, `delete:products`). Roles are collections of permissions. A permission-checking middleware accepts one or more required permissions and verifies that `req.user.permissions` includes all of them. This is more precise than role checks because a user may have a role that grants some but not all capabilities on a resource. Permissions can be stored in the token (for stateless verification) or loaded from a database (for real-time revocation).

**Beginner-Friendly Explanation:** A role check is like saying "only managers can enter." A permission check is like saying "anyone with the 'edit price' badge can change prices." Permissions are more precise — you can grant the 'edit price' badge to a specific employee without making them a manager. This is the principle of least privilege: give users only the permissions they need, nothing more.

### Purposes

- To implement granular action boundaries (e.g., `read:users`, `write:orders`) rather than broad checks.
- To apply the principle of least privilege — users receive only the permissions they need.
- To decouple permissions from roles, allowing flexible assignment.
- To enable permission checks at the route level (middleware) and the data level (repository).

### Syntax Rules and Structure

```javascript
// Permission constants
const PERMISSIONS = {
  READ_USERS: 'read:users',
  WRITE_USERS: 'write:users',
  DELETE_USERS: 'delete:users',
  READ_ORDERS: 'read:orders',
  WRITE_ORDERS: 'write:orders'
};

// Permission-checking middleware
function requirePermission(...required) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const userPermissions = req.user.permissions || [];
    const hasAll = required.every(p => userPermissions.includes(p));

    if (!hasAll) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        required,
        granted: userPermissions
      });
    }

    next();
  };
}

// Usage
router.get('/users',
  authenticate,
  requirePermission(PERMISSIONS.READ_USERS),
  userController.list
);

router.post('/orders',
  authenticate,
  requirePermission(PERMISSIONS.WRITE_ORDERS),
  orderController.create
);
```

| Component | Breakdown |
|-----------|-----------|
| `action:resource` | Permission naming convention (e.g., `read:users`). |
| `required.every(...)` | All required permissions must be present. |
| `req.user.permissions` | Array of permissions from the token or database. |

**Rules:**
- Use `action:resource` naming for permissions (e.g., `read:users`, `write:orders`).
- Require **all** specified permissions by default (use `.some()` for any-of checks).
- Load permissions from the token for stateless verification, or from the database for real-time revocation.
- Apply permission checks at the **route level** before the handler runs.
- Grant permissions, not roles, wherever possible — permissions are more precise.

**Constraints:**
- Permissions in JWTs become stale if revoked — use short-lived tokens or database lookups.
- Too many permissions can bloat the token — consider permission groups or role expansion.
- Permission checks alone do not verify ownership — combine with resource-ownership checks.

### Annotated Code Example

```javascript
// permission-checks.js
const express = require('express');
const router = express.Router();

// Role-to-permission mapping (stored in database in production)
const ROLE_PERMISSIONS = {
  user: ['read:own-profile', 'write:own-profile', 'read:products'],
  editor: ['read:own-profile', 'write:own-profile', 'read:products', 'write:products'],
  admin: ['read:users', 'write:users', 'delete:users', 'read:products', 'write:products', 'delete:products']
};

function requirePermission(...required) {
  return (req, res, next) => {
    const permissions = ROLE_PERMISSIONS[req.user.role] || [];
    const hasAll = required.every(p => permissions.includes(p));

    if (!hasAll) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        required
      });
    }
    next();
  };
}

// Routes with granular permissions
router.get('/products', requirePermission('read:products'), (req, res) => {
  res.json({ products: [] });
});

router.post('/products', requirePermission('write:products'), (req, res) => {
  res.json({ message: 'Product created' });
});

router.delete('/products/:id', requirePermission('delete:products'), (req, res) => {
  res.json({ message: 'Product deleted' });
});

module.exports = router;
```

**Expected Output (for `editor` role on `POST /products`):**
```json
{ "message": "Product created" }
```

**Expected Output (for `editor` role on `DELETE /products/1`):**
```json
{ "error": "Insufficient permissions", "required": ["delete:products"] }
```

**Why this output:** The `editor` role includes `write:products` but not `delete:products`. The POST request succeeds; the DELETE request fails with 403 because the editor lacks the specific deletion permission — even though they have write access.

### Real-World Cases

- **E-commerce:** `read:orders` for support staff, `write:orders` for fulfilment, `delete:orders` for admins.
- **Content platforms:** `write:posts` for authors, `publish:posts` for editors.
- **SaaS:** `read:billing` for accountants, `write:billing` for finance managers.
- **Healthcare:** `read:patient-records` for doctors, `write:prescriptions` for licensed prescribers.

---

## Core Concept 3: Resource Ownership

### Definitions

**Core Definition:** Resource ownership verification ensures that the authenticated user has a legitimate relationship to the specific record being accessed — not just the general permission to perform the action — preventing users from accessing or modifying other users' data.

**Technical Definition:** Resource ownership is enforced by comparing the authenticated user's identity (`req.user.id`) against the owner identifier stored on the resource (`resource.userId`, `resource.ownerId`, `resource.tenantId`). This check must occur **after** loading the resource and **before** performing the action. The critical implementation detail is that the ownership check must be enforced at the **data query level** — the repository query should filter by both the resource ID and the user ID, so that a resource owned by another user is never returned in the first place. This "query-scoping" approach is more robust than post-load checks because it fails closed: if the resource isn't found, the response is 404 (not found) rather than 403 (forbidden), which also hides the existence of the resource.

**Beginner-Friendly Explanation:** Resource ownership is like a hotel room key. Your key opens your room (your resource), not anyone else's room. Even though you're a registered guest (authenticated) with permission to use hotel rooms (authorised), you can only use the room you paid for (owned). The hotel's lock system checks both that you're a guest and that this specific room is yours.

### Purposes

- To verify if the requesting user owns the specific record (e.g., preventing a user from editing someone else's profile).
- To prevent horizontal privilege escalation — accessing data belonging to other users at the same privilege level.
- To enforce ownership at the data query level, not just in the controller.
- To hide the existence of resources the user does not own (return 404 instead of 403).

### Syntax Rules and Structure

#### Query-Scoped Ownership (Recommended)

```javascript
// Repository — scoped by user ID
async function findUserPost(postId, userId) {
  return Post.findOne({
    _id: postId,
    authorId: userId        // Ownership filter in the query
  });
}

// Controller — 404 if not found (hides existence)
async function updatePost(req, res) {
  const post = await findUserPost(req.params.id, req.user.id);
  if (!post) {
    return res.status(404).json({ error: 'Post not found' });
  }
  // ... update post
}
```

#### Post-Load Ownership Check

```javascript
async function updatePost(req, res) {
  const post = await Post.findById(req.params.id);
  if (!post) return res.status(404).json({ error: 'Post not found' });

  // Ownership check after loading
  if (post.authorId.toString() !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  // ... update post
}
```

| Approach | Security | Information Leakage | Recommendation |
|----------|----------|---------------------|----------------|
| Query-scoped | Higher — 404 for non-owned | None | Preferred |
| Post-load check | Adequate — 403 for non-owned | Reveals existence | Acceptable |

**Rules:**
- Enforce ownership at the **data query level** — filter by both resource ID and user ID.
- Return **404 Not Found** (not 403 Forbidden) when the resource exists but is not owned by the user — this prevents enumeration.
- Never trust client-supplied ownership identifiers (e.g., `userId` in the request body) — use `req.user.id`.
- For multi-tenant systems, scope queries by `tenantId` as well as `userId`.

**Constraints:**
- Query-scoping requires the ownership field to be indexed for performance.
- Shared resources (e.g., team documents) require a more complex access model than simple ownership.
- Ownership checks must be applied consistently across all endpoints that accept a resource ID.

### Annotated Code Example

```javascript
// resource-ownership.js
const express = require('express');
const app = express();
app.use(express.json());

// Simulated database
const posts = [
  { id: '1', authorId: 'usr_alice', title: 'Alice Post' },
  { id: '2', authorId: 'usr_bob', title: 'Bob Post' }
];

// Middleware — simulated authentication
app.use((req, res, next) => {
  req.user = { id: req.headers['x-user-id'] || 'usr_alice' };
  next();
});

// GET /posts/:id — ownership-scoped query
app.get('/posts/:id', async (req, res) => {
  const post = posts.find(
    p => p.id === req.params.id && p.authorId === req.user.id
  );

  if (!post) {
    return res.status(404).json({ error: 'Post not found' });
  }

  res.json({ data: post });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /posts/1` with `X-User-Id: usr_alice`):**
```json
{ "data": { "id": "1", "authorId": "usr_alice", "title": "Alice Post" } }
```

**Expected Output (for `GET /posts/2` with `X-User-Id: usr_alice`):**
```json
{ "error": "Post not found" }
```

**Why this output:** The query filters by both `id` and `authorId`. Alice owns post 1 but not post 2 — so post 2 returns 404. This prevents horizontal privilege escalation: Alice cannot access Bob's posts by guessing the ID. The 404 also hides the existence of Bob's post from Alice.

### Real-World Cases

- **Social media:** Users can edit their own posts but not others'.
- **E-commerce:** Customers can view their own orders but not others'.
- **SaaS:** Users can access resources within their tenant only.
- **Healthcare:** Patients can view their own records but not other patients'.

---

## Core Concept 4: Broken Object Level Authorization (BOLA / IDOR)

### Definitions

**Core Definition:** Broken Object Level Authorization (BOLA), also known as Insecure Direct Object Reference (IDOR), is a vulnerability where an API accepts a user-supplied identifier and returns or modifies a record the user does not own — because the endpoint fails to verify that the authenticated user has authority over the specific resource.

**Technical Definition:** BOLA is the #1 risk in the OWASP API Security Top 10 (2023). It occurs when an endpoint uses an identifier from the URL, query string, or request body to look up a record but does not verify that the record belongs to the requesting user. Attackers exploit BOLA by incrementing, guessing, or enumerating IDs to access other users' data. The fix is **architectural**: every repository query for a user-specific resource must be scoped by the authenticated user's ID (or tenant ID) in addition to the resource ID. This makes it impossible to retrieve a non-owned resource — the query simply returns nothing. A defence-in-depth approach combines query-scoping with an explicit ownership check and automated testing that attempts IDOR attacks.

**Beginner-Friendly Explanation:** BOLA is like a hotel where the front desk hands out room keys without checking whether the guest actually booked that room. You ask for room 101, and they give you the key — even though you booked room 205. The fix is simple: the front desk must check the guest's booking (user ID) against the requested room (resource ID) before issuing the key.

### Purposes

- To design architectural guardrails that prevent attackers from manipulating resource IDs in the URL or payload to access unauthorised data.
- To make BOLA structurally impossible by scoping every query by the authenticated user.
- To apply defence in depth — query scoping, ownership checks, and automated IDOR testing.
- To use opaque, non-sequential identifiers (UUIDs) to make ID guessing harder.

### Syntax Rules and Structure

#### Vulnerable Pattern

```javascript
// ❌ VULNERABLE: No ownership check
app.get('/api/orders/:id', authenticate, async (req, res) => {
  const order = await Order.findById(req.params.id); // Any ID works!
  res.json(order); // Returns anyone's order
});
```

#### Secure Pattern

```javascript
// ✅ SECURE: Query scoped by user ID
app.get('/api/orders/:id', authenticate, async (req, res) => {
  const order = await Order.findOne({
    _id: req.params.id,
    userId: req.user.id        // Ownership enforced in the query
  });

  if (!order) return res.status(404).json({ error: 'Order not found' });
  res.json(order);
});
```

| Mitigation | Description |
|-----------|-------------|
| Query scoping | Filter every query by `userId` or `tenantId`. |
| Opaque IDs | Use UUIDs instead of sequential integers. |
| Ownership middleware | Centralise the check for reuse across routes. |
| Automated testing | Write tests that attempt IDOR attacks. |
| Defence in depth | Combine all of the above. |

**Rules:**
- **Never** look up a user-specific resource by ID alone — always scope by the authenticated user.
- Use **UUIDs** instead of sequential integers — they are harder to guess but not a replacement for authorization.
- Centralise ownership checks in middleware or repository methods for consistency.
- Write **automated tests** that attempt to access resources with a different user's token.
- Log and alert on failed ownership checks — repeated attempts indicate an attack.

**Constraints:**
- UUIDs reduce guessability but do not prevent BOLA if authorization is missing.
- Shared or collaborative resources require a different access model (e.g., permission tables).
- Legacy systems with sequential IDs are still vulnerable even with authorization — consider migrating to UUIDs.

### Annotated Code Example

```javascript
// bola-prevention.js
const express = require('express');
const app = express();

// Simulated database
const orders = [
  { id: 'ord_001', userId: 'usr_alice', total: 99.99 },
  { id: 'ord_002', userId: 'usr_bob', total: 49.99 }
];

app.use((req, res, next) => {
  req.user = { id: req.headers['x-user-id'] };
  next();
});

// ❌ VULNERABLE ENDPOINT
app.get('/vulnerable/orders/:id', (req, res) => {
  const order = orders.find(o => o.id === req.params.id);
  if (!order) return res.status(404).json({ error: 'Not found' });
  res.json(order); // BOLA! Alice can see Bob's order
});

// ✅ SECURE ENDPOINT
app.get('/secure/orders/:id', (req, res) => {
  const order = orders.find(
    o => o.id === req.params.id && o.userId === req.user.id
  );
  if (!order) return res.status(404).json({ error: 'Not found' });
  res.json(order);
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /vulnerable/orders/ord_002` with `X-User-Id: usr_alice`):**
```json
{ "id": "ord_002", "userId": "usr_bob", "total": 49.99 }
```

**Expected Output (for `GET /secure/orders/ord_002` with `X-User-Id: usr_alice`):**
```json
{ "error": "Not found" }
```

**Why this output:** The vulnerable endpoint looks up the order by ID alone — Alice can access Bob's order. The secure endpoint scopes the query by `userId` — Alice gets a 404 because the order doesn't belong to her. This is the architectural guardrail that prevents BOLA.

### Real-World Cases

- **Banking:** Viewing another customer's transactions by guessing the account ID.
- **Social media:** Reading another user's private messages.
- **Healthcare:** Accessing another patient's medical records.
- **SaaS:** Accessing another tenant's data by manipulating the tenant ID.

---

## Core Concept 5: Fail-Closed Principle

### Definitions

**Core Definition:** The fail-closed principle requires that authorization middleware default to **denying** access when anything is uncertain — a missing parameter, a configuration error, an exception, or an unexpected state — rather than defaulting to allowing access.

**Technical Definition:** In security-critical code, the default behaviour on error must be denial. If an authorization middleware throws an exception, cannot parse a required parameter, cannot reach the permission store, or encounters an unexpected configuration, it must return 403 (or 500) and **not** call `next()`. The fail-closed principle is the opposite of fail-open, where an error results in access being granted. Fail-open is a critical vulnerability: if an attacker can cause the authorization system to error, they bypass authorization entirely. Express middleware naturally supports fail-closed patterns when `next(err)` is called on error — the error propagates to the error handler and the request is not processed. The anti-pattern is catching errors in authorization middleware and calling `next()` without denying access.

**Beginner-Friendly Explanation:** Fail-closed is like a bank vault that locks automatically if the power goes out. Fail-open is like a bank vault that opens automatically if the power goes out — obviously catastrophic. In authorization, if anything goes wrong — the permission database is down, a parameter is missing, an unexpected error occurs — the safe default is to deny access. Users can always retry; they can't un-see data they shouldn't have accessed.

### Purposes

- To design authorization middleware that defaults to denying access if a configuration error, missing parameter, or runtime exception occurs.
- To prevent attackers from bypassing authorization by inducing errors.
- To ensure that exceptions in authorization code result in denial, not access.
- To provide a safety net against misconfiguration.

### Syntax Rules and Structure

#### Fail-Closed Middleware

```javascript
function requirePermission(...required) {
  return (req, res, next) => {
    try {
      // 1. Missing user → deny
      if (!req.user || !req.user.id) {
        return res.status(401).json({ error: 'Authentication required' });
      }

      // 2. Missing permissions → deny
      if (!Array.isArray(req.user.permissions)) {
        return res.status(403).json({ error: 'No permissions configured' });
      }

      // 3. Missing required parameters → deny
      if (required.length === 0) {
        return res.status(403).json({ error: 'No permission required specified' });
      }

      // 4. Check permissions
      const hasAll = required.every(p => req.user.permissions.includes(p));
      if (!hasAll) {
        return res.status(403).json({ error: 'Insufficient permissions' });
      }

      next();
    } catch (err) {
      // 5. Any exception → deny (fail closed)
      console.error('Authorization error:', err);
      res.status(403).json({ error: 'Authorization failed' });
    }
  };
}
```

| Scenario | Fail-Open (❌) | Fail-Closed (✅) |
|----------|---------------|------------------|
| Missing user | Allow | Deny (401) |
| Missing permissions | Allow | Deny (403) |
| Missing parameters | Allow | Deny (403) |
| Exception thrown | Allow | Deny (403) |
| Permission store down | Allow | Deny (503) |

**Rules:**
- **Default to deny** — every code path that does not explicitly allow must deny.
- **Catch exceptions** in authorization middleware and return 403, not `next()`.
- **Validate parameters** — if required permissions are not specified, deny.
- **Never** call `next()` in a `catch` block of authorization middleware.
- **Log** authorization failures for security monitoring.

**Constraints:**
- Fail-closed can cause availability issues if the permission store is down — consider a cached permission snapshot with a short TTL.
- Overly aggressive fail-closed behaviour can lock out legitimate users during outages — balance security with availability.
- The fail-closed principle applies to all security-critical code, not just authorization.

### Annotated Code Example

```javascript
// fail-closed.js
const express = require('express');
const app = express();

// ❌ FAIL-OPEN: Exception allows access
function failOpenMiddleware(req, res, next) {
  try {
    const permissions = JSON.parse(req.headers['x-permissions'] || '[]');
    if (!permissions.includes('read:data')) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  } catch (err) {
    next(); // ❌ BUG: Exception grants access!
  }
}

// ✅ FAIL-CLOSED: Exception denies access
function failClosedMiddleware(req, res, next) {
  try {
    const raw = req.headers['x-permissions'];
    if (!raw) {
      return res.status(403).json({ error: 'No permissions provided' });
    }

    const permissions = JSON.parse(raw);
    if (!Array.isArray(permissions) || !permissions.includes('read:data')) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    next();
  } catch (err) {
    console.error('Authorization error:', err.message);
    res.status(403).json({ error: 'Authorization failed' }); // ✅ Deny
  }
}

app.get('/open', failOpenMiddleware, (req, res) => res.json({ data: 'secret' }));
app.get('/closed', failClosedMiddleware, (req, res) => res.json({ data: 'secret' }));

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /open` with malformed `X-Permissions`):**
```json
{ "data": "secret" }
```

**Expected Output (for `GET /closed` with malformed `X-Permissions`):**
```json
{ "error": "Authorization failed" }
```

**Why this output:** The fail-open middleware catches the `JSON.parse` error and calls `next()` — granting access. The fail-closed middleware catches the same error and returns 403 — denying access. This demonstrates why the fail-closed principle is critical: an attacker who can cause the authorization system to throw can bypass authorization entirely in a fail-open implementation.

### Real-World Cases

- **Permission store outages:** If Redis is down, deny access rather than allowing everything.
- **Malformed tokens:** If a token cannot be parsed, deny access rather than treating it as unauthenticated.
- **Missing configuration:** If a required permission is not specified in the route, deny access rather than allowing.
- **Unexpected exceptions:** Any exception in authorization code must result in denial.

---

## References

- OWASP API Security Top 10 (2023) — API1: Broken Object Level Authorization — https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
- OWASP Authorization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- OWASP Top 10 (2021) — A01: Broken Access Control — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- Auth0 — Authentication vs. Authorization — https://auth0.com/docs/get-started/identity-fundamentals/authentication-and-authorization
- RFC 9110 — HTTP Semantics (401, 403) — https://www.rfc-editor.org/rfc/rfc9110#section-15.5
- NVIDIA — Authorization Fundamentals — https://docs.nvidia.com
- Security Stack Exchange — 401 vs 403 — https://security.stackexchange.com/questions/140620
- CWE-639 — Authorization Bypass Through User-Controlled Key — https://cwe.mitre.org/data/definitions/639.html
- CWE-284 — Improper Access Control — https://cwe.mitre.org/data/definitions/284.html
- CWE-863 — Incorrect Authorization — https://cwe.mitre.org/data/definitions/863.html
- PortSwigger — Insecure Direct Object References (IDOR) — https://portswigger.net/web-security/access-control/idor
- OWASP — Insecure Direct Object Reference Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
- CVE-2026-32255 — Parse Server BOLA/IDOR vulnerability — https://nvd.nist.gov/vuln/detail/CVE-2026-32255
- CVE-2026-32943 — Parse Server Password Reset Single-Use Bypass — https://security.glexia.com
- TRAE-Skills — Authorization Best Practices — https://github.com/MarcoNasi/TRAE-Skills/blob/main/backend/Authorization_Best_Practices.md
- Safeguard.sh — Broken Object Level Authorization (BOLA) Prevention Guide — https://safeguard.sh/resources/blog/bola-prevention
- StackHawk — BOLA: The OWASP API #1 Risk and How to Test It — https://www.stackhawk.com/blog/bola-owasp-api
- Wallarm — What is BOLA (Broken Object Level Authorization)? — https://www.wallarm.com/what/what-is-bola-broken-object-level-authorization
- freeCodeCamp — How to Prevent IDOR Vulnerabilities in Node.js — https://www.freecodecamp.org/news/how-to-prevent-idor-vulnerabilities-in-nodejs
- Mozilla — Fail-Safe Defaults — https://developer.mozilla.org/en-US/docs/Glossary/Fail-safe
- OWASP — Fail Securely — https://owasp.org/www-community/Fail_securely