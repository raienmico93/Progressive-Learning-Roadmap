# Role-Based Access Control (RBAC) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Role-Based Access Control (RBAC) is an authorization model in which permissions are assigned to roles, and users are assigned to roles — so a user's access rights are determined by the roles they hold rather than by individual per-user permission grants.

**Technical Definition:** RBAC is defined by three sets of relationships: (1) **users** are assigned to **roles**, (2) **roles** are assigned **permissions**, and (3) **permissions** describe specific actions on specific resources. This indirection — users never receive permissions directly — makes RBAC manageable at scale: adding a permission to a role automatically grants it to every user with that role. The NIST RBAC standard (INCITS 359) defines four levels: **Flat RBAC** (users, roles, permissions), **Hierarchical RBAC** (roles inherit from other roles), **Constrained RBAC** (separation of duties — no user holds conflicting roles), and **Symmetric RBAC** (role-permission reviews and auditing). Hierarchical RBAC introduces role inheritance — a senior role inherits all permissions of the roles beneath it — which prevents the flat, bloated role tables that emerge when every role must independently list every permission it needs.

**Beginner-Friendly Explanation:** RBAC is like a company's job-title system. Instead of giving each employee a personal list of what they can do (which would be a nightmare to manage), you give them a job title (role). The job title comes with a standard set of permissions. When a new employee joins as a "Manager," they automatically get all the Manager permissions. When you need to change what Managers can do, you update the role once — and every Manager is updated instantly. Hierarchical RBAC is like a corporate ladder: a "Senior Manager" automatically has all the permissions of a "Manager," who has all the permissions of a "Team Lead," who has all the permissions of an "Employee."

### Key Characteristics

- **Indirection:** Users → Roles → Permissions. Users never receive permissions directly.
- **Scalability:** Adding a permission to a role updates every user with that role.
- **Hierarchical inheritance:** Senior roles inherit permissions from subordinate roles automatically.
- **Declarative middleware:** Route protection is expressed as `checkRole(['Admin'])` rather than scattered if-else logic.
- **Separation of duties:** Constrained RBAC prevents a single user from holding conflicting roles.
- **Auditability:** Role assignments and permission mappings are centralised and reviewable.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **A working authentication system:** JWT or session-based, with `req.user` populated.
- **A database** to store users, roles, and permissions (PostgreSQL, MongoDB, etc.).
- **Basic JavaScript knowledge:** Functions, closures, arrays, and middleware concepts.

### Related Programming Areas

- **Authorization Fundamentals:** RBAC is one of several authorization models (alongside ABAC and ReBAC).
- **Authentication:** RBAC always follows authentication — `req.user` must be populated first.
- **Attribute-Based Access Control (ABAC):** A more granular alternative that evaluates attributes rather than roles.
- **Relationship-Based Access Control (ReBAC):** Used when access depends on relationships (e.g., "user is a member of the project").
- **Separation of Duties:** A compliance requirement enforced through constrained RBAC.

### Core Concepts

1. **Roles** — assigning groups of permissions to collection tags (e.g., Admin, Manager, User).
2. **Permissions** — mapping fine-grained privileges to specific roles for flexibility.
3. **Role Middleware** — declarative, reusable Express middleware (`checkRole(['Admin'])`).
4. **Admin Privileges** — handling superuser bypass risks and enforcing security boundaries.
5. **Hierarchical Roles** — superior roles inherit permissions from subordinate roles.

---

## Core Concept 1: Roles

### Definitions

**Core Definition:** A role is a named collection of permissions that represents a job function or responsibility level (e.g., Admin, Manager, User), and is assigned to users to grant them the permissions bundled within the role.

**Technical Definition:** In RBAC, a role is the indirection layer between users and permissions. A role has a unique name, an optional description, and a set of permissions. Users are assigned one or more roles. The assignment is typically stored in a join table (`user_roles`) or as an array on the user document. Role names should be stable and descriptive — they represent job functions, not individuals. The NIST RBAC standard recognises both **static separation of duties** (SSD), where role assignments are constrained at assignment time, and **dynamic separation of duties** (DSD), where conflicting roles can be held but not activated simultaneously.

**Beginner-Friendly Explanation:** A role is like a badge with a colour. Everyone with a blue badge (User) can enter the lobby and the break room. Everyone with a gold badge (Admin) can enter every room, including the server room. When a new person joins the company, you give them a badge (assign a role) — you don't have to write out a personal list of rooms they can enter.

### Purposes

- To assign groups of permissions to collection tags (e.g., Admin, Manager, User).
- To simplify access management — managing roles, not individual users.
- To represent job functions and responsibility levels in the access model.
- To enable auditability — reviewing who has which roles is simpler than reviewing per-user permissions.

### Syntax Rules and Structure

#### Role Schema (Mongoose Example)

```javascript
const mongoose = require('mongoose');

const roleSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    unique: true,
    enum: ['user', 'manager', 'admin']   // Controlled vocabulary
  },
  permissions: [{
    type: String,
    required: true                        // e.g., 'read:users'
  }],
  description: String,
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

#### User-Role Assignment

```javascript
const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  passwordHash: { type: String, required: true },
  roles: [{
    type: String,
    enum: ['user', 'manager', 'admin'],
    default: ['user']                     // Every user starts with 'user'
  }]
});
```

| Role | Description | Typical Permissions |
|------|-------------|---------------------|
| `user` | Standard user | `read:own-profile`, `write:own-profile`, `read:products` |
| `manager` | Team manager | `read:users`, `write:orders`, `read:reports` |
| `admin` | System administrator | All permissions including `delete:users`, `write:config` |

**Rules:**
- Use a controlled vocabulary (enum) for role names — prevents typos and unauthorised roles.
- Every user should have at least one base role (typically `user`).
- Roles should represent job functions, not individuals.
- Store the role-permission mapping in a separate collection or configuration for flexibility.
- Use static separation of duties (SSD) to prevent conflicting role assignments.

**Constraints:**
- Flat role tables (every role lists every permission) become unmanageable as the system grows — use hierarchical roles.
- Roles stored in JWTs become stale if the role assignment changes — use short-lived tokens or database lookups.
- Overly broad roles (e.g., a single "admin" role with every permission) violate the principle of least privilege.

### Annotated Code Example

```javascript
// roles.js
const mongoose = require('mongoose');

// Role model
const Role = mongoose.model('Role', new mongoose.Schema({
  name: { type: String, required: true, unique: true },
  permissions: [String],
  description: String
}));

// User model
const User = mongoose.model('User', new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  roles: [{ type: String, ref: 'Role' }]
}));

// Seed roles
async function seedRoles() {
  await Role.create([
    {
      name: 'user',
      permissions: ['read:own-profile', 'write:own-profile', 'read:products'],
      description: 'Standard user'
    },
    {
      name: 'manager',
      permissions: ['read:users', 'write:orders', 'read:reports'],
      description: 'Team manager'
    },
    {
      name: 'admin',
      permissions: ['*'],                  // Wildcard — all permissions
      description: 'System administrator'
    }
  ]);
}

// Assign role to user
async function assignRole(userId, roleName) {
  await User.findByIdAndUpdate(userId, {
    $addToSet: { roles: roleName }         // No duplicates
  });
}
```

**Expected Output (database state):**
```json
{
  "email": "alice@example.com",
  "roles": ["user", "manager"]
}
```

**Why this output:** Alice has two roles: `user` (base role) and `manager`. Her effective permissions are the union of both roles' permissions. The `$addToSet` operator prevents duplicate role assignments.

### Real-World Cases

- **SaaS platforms:** `owner`, `admin`, `member`, `viewer` roles per workspace.
- **E-commerce:** `customer`, `support`, `fulfilment`, `admin` roles.
- **Healthcare:** `patient`, `nurse`, `doctor`, `administrator` roles.
- **Content platforms:** `reader`, `author`, `editor`, `publisher` roles.

---

## Core Concept 2: Permissions

### Definitions

**Core Definition:** Permissions are fine-grained privileges that describe specific actions on specific resources (e.g., `read:users`, `write:orders`, `delete:products`), and are mapped to roles to define what each role can do.

**Technical Definition:** A permission is expressed as a `resource:action` or `action:resource` pair — the exact convention matters less than consistency. Permissions are the atoms of the authorization model; roles are collections of permissions. Permissions can be stored in the database, loaded from configuration, or embedded in tokens. For scalability, permissions can support wildcards (`users:*` matches all user actions) and hierarchical matching (`admin:*` matches everything). The permission set should be **complete** (every action the system supports has a permission) and **minimal** (no permission grants more than necessary).

**Beginner-Friendly Explanation:** Permissions are like the individual keys on a keyring. One key opens the front door (`read:home`), another opens the office (`write:office`), and another opens the safe (`delete:money`). Roles are different keyrings — the "employee" keyring has the front door and office keys; the "manager" keyring also has the safe key. The keyring (role) is a convenient way to carry multiple keys (permissions).

### Purposes

- To map fine-grained privileges to specific roles to ensure flexibility.
- To apply the principle of least privilege — each role gets only the permissions it needs.
- To enable permission checks that are more precise than role checks.
- To support wildcard and hierarchical permission matching for scalability.

### Syntax Rules and Structure

#### Permission Naming Convention

```
<action>:<resource>
```

| Permission | Description |
|-----------|-------------|
| `read:users` | View the user list. |
| `write:users` | Create or update users. |
| `delete:users` | Delete users. |
| `read:orders` | View orders. |
| `write:orders` | Create or update orders. |
| `read:reports` | View reports. |
| `*` | Wildcard — all permissions (admin only). |
| `users:*` | All actions on users. |

#### Role-Permission Mapping

```javascript
const ROLE_PERMISSIONS = {
  user: [
    'read:own-profile',
    'write:own-profile',
    'read:products'
  ],
  manager: [
    ...['read:own-profile', 'write:own-profile', 'read:products'], // Inherited
    'read:users',
    'write:orders',
    'read:reports'
  ],
  admin: ['*']                             // All permissions
};
```

**Rules:**
- Use a consistent naming convention (`action:resource` or `resource:action`).
- Define permissions **before** assigning them to roles.
- Support wildcards (`*`, `users:*`) for administrative convenience — but log wildcard usage.
- Keep permissions granular — `write:users` and `delete:users` should be separate.
- Review the permission set regularly to remove unused permissions.

**Constraints:**
- Too many permissions bloat tokens if embedded — consider role expansion or database lookups.
- Wildcard permissions (`*`) are convenient but reduce auditability — use sparingly.
- Permission names in tokens must be validated server-side — never trust client-supplied permission lists.

### Annotated Code Example

```javascript
// permissions.js
const PERMISSIONS = {
  // User permissions
  READ_OWN_PROFILE: 'read:own-profile',
  WRITE_OWN_PROFILE: 'write:own-profile',
  READ_USERS: 'read:users',
  WRITE_USERS: 'write:users',
  DELETE_USERS: 'delete:users',

  // Order permissions
  READ_ORDERS: 'read:orders',
  WRITE_ORDERS: 'write:orders',

  // Report permissions
  READ_REPORTS: 'read:reports'
};

// Role-permission mapping (with inheritance)
const ROLE_PERMISSIONS = {
  user: [
    PERMISSIONS.READ_OWN_PROFILE,
    PERMISSIONS.WRITE_OWN_PROFILE
  ],
  manager: [
    // Inherited from user
    PERMISSIONS.READ_OWN_PROFILE,
    PERMISSIONS.WRITE_OWN_PROFILE,
    // Manager-specific
    PERMISSIONS.READ_USERS,
    PERMISSIONS.READ_ORDERS,
    PERMISSIONS.WRITE_ORDERS,
    PERMISSIONS.READ_REPORTS
  ],
  admin: ['*']                             // Wildcard
};

// Expand wildcard permissions
function expandPermissions(permissions) {
  if (permissions.includes('*')) {
    return Object.values(PERMISSIONS);
  }
  return permissions;
}

// Get effective permissions for a user's roles
function getEffectivePermissions(userRoles) {
  const permissions = new Set();
  for (const role of userRoles) {
    const rolePerms = ROLE_PERMISSIONS[role] || [];
    for (const perm of expandPermissions(rolePerms)) {
      permissions.add(perm);
    }
  }
  return Array.from(permissions);
}
```

**Expected Output (for `getEffectivePermissions(['user', 'manager'])`):**
```json
[
  "read:own-profile",
  "write:own-profile",
  "read:users",
  "read:orders",
  "write:orders",
  "read:reports"
]
```

**Why this output:** The user holds both `user` and `manager` roles. The effective permissions are the union of both roles' permissions (duplicates removed). The manager role includes all the user's permissions plus the manager-specific ones.

### Real-World Cases

- **API gateways:** Permission checks at the edge determine which services a request can reach.
- **Admin panels:** `read:analytics` for analysts, `write:config` for system administrators.
- **Multi-tenant SaaS:** Permissions are scoped to a tenant (e.g., `read:orders:tenant_123`).
- **Healthcare:** `read:patient-records` for doctors, `write:prescriptions` for licensed prescribers.

---

## Core Concept 3: Role Middleware

### Definitions

**Core Definition:** Role middleware is a declarative Express middleware factory that checks whether the authenticated user holds one of the required roles before allowing the request to proceed.

**Technical Definition:** A role middleware factory accepts a list of allowed roles and returns an Express middleware function. The middleware reads `req.user.roles` (populated by the authentication middleware) and checks whether any of the user's roles appear in the allowed list. If yes, it calls `next()`; if no, it returns 403 Forbidden. The pattern is declarative: `checkRole(['Admin'])` reads as a statement of intent rather than an imperative if-else block. The middleware should be **reusable** (applied to many routes), **composable** (combined with other middleware), and **fail-closed** (deny on error). For hierarchical roles, the check should expand the user's roles to include inherited roles before comparing.

**Beginner-Friendly Explanation:** Role middleware is like a door with a sign that says "Managers and Admins only." The sign (the middleware) doesn't care who you are personally — it checks whether your badge (role) has the right colour. If your badge is green (User) and the sign says "Managers and Admins only," you can't enter. The sign is declarative — it states the rule, and the door enforces it.

### Purposes

- To write declarative, reusable Express middleware functions to protect routes based on role requirements.
- To centralise role-checking logic so it is consistent across all routes.
- To compose role checks with other middleware (authentication, validation, rate limiting).
- To support hierarchical role checks by expanding inherited roles.

### Syntax Rules and Structure

```javascript
// Role middleware factory
function checkRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const userRoles = req.user.roles || [];
    const hasRole = allowedRoles.some(role => userRoles.includes(role));

    if (!hasRole) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        required: allowedRoles,
        granted: userRoles
      });
    }

    next();
  };
}

// Usage
router.get('/admin/dashboard',
  authenticate,
  checkRole('admin'),
  adminController.dashboard
);

router.post('/products',
  authenticate,
  checkRole('manager', 'admin'),           // Either role works
  productController.create
);
```

| Component | Breakdown |
|-----------|-----------|
| `checkRole(...allowedRoles)` | Factory function accepting allowed roles. |
| `req.user.roles` | Array of roles from the authenticated user. |
| `allowedRoles.some(...)` | Any-of check — at least one role must match. |
| `next()` | Proceed if the check passes. |

**Rules:**
- Use the factory pattern — never write inline role checks in route handlers.
- Use `some()` for any-of checks (most common) or `every()` for all-of checks (rare).
- Always return **403** (not 401) when the user is authenticated but lacks the role.
- Compose with authentication middleware — role checks require `req.user`.
- For hierarchical roles, expand the user's roles to include inherited roles before checking.

**Constraints:**
- Role checks alone do not verify resource ownership — combine with ownership checks.
- Role assignments in JWTs become stale — use short-lived tokens or database lookups.
- Overly broad roles (e.g., `admin`) reduce the precision of role checks.

### Annotated Code Example

```javascript
// role-middleware.js
const express = require('express');
const app = express();

// Simulated authentication
app.use((req, res, next) => {
  req.user = {
    id: 'usr_123',
    roles: (req.headers['x-roles'] || 'user').split(',')
  };
  next();
});

// Role middleware factory
function checkRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const userRoles = req.user.roles || [];
    const hasRole = allowedRoles.some(r => userRoles.includes(r));

    if (!hasRole) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        required: allowedRoles,
        granted: userRoles
      });
    }

    next();
  };
}

// Routes with role protection
app.get('/api/products', (req, res) => {
  res.json({ products: [] });             // Public
});

app.post('/api/products',
  checkRole('manager', 'admin'),
  (req, res) => {
    res.json({ message: 'Product created' });
  }
);

app.delete('/api/products/:id',
  checkRole('admin'),
  (req, res) => {
    res.json({ message: 'Product deleted' });
  }
);

app.get('/api/admin/dashboard',
  checkRole('admin'),
  (req, res) => {
    res.json({ dashboard: 'data' });
  }
);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/products` with `X-Roles: manager`):**
```json
{ "message": "Product created" }
```

**Expected Output (for `DELETE /api/products/1` with `X-Roles: manager`):**
```json
{
  "error": "Insufficient permissions",
  "required": ["admin"],
  "granted": ["manager"]
}
```

**Why this output:** The `checkRole` middleware checks whether any of the user's roles appear in the allowed list. The manager can create products but cannot delete them — only admin can. The error response includes both the required and granted roles for debugging.

### Real-World Cases

- **Admin panels:** `checkRole('admin')` on all `/admin/*` routes.
- **Content management:** `checkRole('editor', 'admin')` on publish routes.
- **Financial operations:** `checkRole('finance', 'admin')` on payment routes.
- **Multi-tenant SaaS:** `checkRole('owner', 'admin')` on workspace settings.

---

## Core Concept 4: Admin Privileges

### Definitions

**Core Definition:** Admin privileges are the highest level of access in an RBAC system, granting broad or unrestricted capabilities — and they require special handling to prevent superuser bypass risks and enforce security boundaries around high-privilege operations.

**Technical Definition:** The admin role is typically granted a wildcard permission (`*`) or an extensive list of permissions. This creates several risks: (1) **Superuser bypass** — if admin permissions are checked before ownership, an admin can access any resource; (2) **Privilege escalation** — an attacker who compromises an admin account gains full system access; (3) **Accidental damage** — an admin can delete critical data without additional confirmation. Mitigations include: requiring **step-up authentication** (re-entering password or MFA) for destructive admin operations; applying **separation of duties** (no single admin can both create and approve a critical change); **audit logging** every admin action; and **scoping** admin permissions to specific resources or tenants rather than global wildcards.

**Beginner-Friendly Explanation:** Admin privileges are like the master key to a building. It opens every door — which is convenient, but dangerous. If a thief steals the master key, they can go anywhere. To reduce risk, you might require a second key (step-up authentication) for the most sensitive rooms (the vault), keep a log of every time the master key is used (audit logging), and make sure the person with the master key can't also approve their own expense reports (separation of duties).

### Purposes

- To handle administrative superuser bypass risks and enforce security boundaries around high-privilege operations.
- To prevent privilege escalation by scoping admin permissions.
- To require additional verification (step-up authentication) for destructive operations.
- To maintain audit trails of all admin actions.
- To enforce separation of duties — no single admin holds conflicting capabilities.

### Syntax Rules and Structure

#### Step-Up Authentication

```javascript
function requireStepUp(req, res, next) {
  const { password, mfaCode } = req.body;

  if (!password || !mfaCode) {
    return res.status(403).json({
      error: 'Step-up authentication required',
      code: 'STEP_UP_REQUIRED'
    });
  }

  // Verify password and MFA code
  // ... (implementation omitted for brevity)

  next();
}

// Destructive admin route requires step-up
router.delete('/admin/users/:id',
  authenticate,
  checkRole('admin'),
  requireStepUp,                            // Additional verification
  auditLog('DELETE_USER'),                  // Audit every action
  adminController.deleteUser
);
```

#### Audit Logging

```javascript
function auditLog(action) {
  return (req, res, next) => {
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      action,
      adminId: req.user.id,
      targetResource: req.params.id,
      ip: req.ip,
      userAgent: req.get('User-Agent')
    }));
    next();
  };
}
```

| Mitigation | Purpose |
|-----------|---------|
| Step-up authentication | Require re-verification for destructive operations. |
| Audit logging | Record every admin action for review. |
| Separation of duties | Prevent a single admin from holding conflicting capabilities. |
| Scoped permissions | Restrict admin to specific resources or tenants. |
| Time-limited privileges | Grant admin access for a limited window (just-in-time). |

**Rules:**
- **Never** grant wildcard (`*`) permissions to non-admin roles.
- Require **step-up authentication** for destructive admin operations (delete, config change).
- **Audit log** every admin action with the actor, action, target, and timestamp.
- Enforce **separation of duties** — no single admin can both create and approve critical changes.
- **Scope** admin permissions where possible (e.g., tenant admin vs. system admin).
- Consider **just-in-time** admin access — grant elevated privileges for a limited window.

**Constraints:**
- Step-up authentication adds friction — apply only to the most sensitive operations.
- Audit logs must be immutable and stored separately from the application database.
- Separation of duties requires role constraints (constrained RBAC).

### Annotated Code Example

```javascript
// admin-privileges.js
const express = require('express');
const app = express();
app.use(express.json());

// Simulated authentication
app.use((req, res, next) => {
  req.user = { id: 'usr_123', roles: (req.headers['x-roles'] || 'user').split(',') };
  next();
});

function checkRole(...allowedRoles) {
  return (req, res, next) => {
    const hasRole = allowedRoles.some(r => req.user.roles.includes(r));
    if (!hasRole) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Step-up authentication for destructive operations
function requireStepUp(req, res, next) {
  const { confirmPassword, mfaCode } = req.body;
  if (!confirmPassword || !mfaCode) {
    return res.status(403).json({
      error: 'Step-up authentication required for this operation',
      code: 'STEP_UP_REQUIRED'
    });
  }
  // Verify confirmPassword and mfaCode...
  next();
}

// Audit logging
function auditLog(action) {
  return (req, res, next) => {
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      action,
      adminId: req.user.id,
      target: req.params.id,
      ip: req.ip
    }));
    next();
  };
}

// Non-destructive admin route — no step-up needed
app.get('/admin/users', checkRole('admin'), (req, res) => {
  res.json({ users: [] });
});

// Destructive admin route — step-up + audit
app.delete('/admin/users/:id',
  checkRole('admin'),
  requireStepUp,
  auditLog('DELETE_USER'),
  (req, res) => {
    res.json({ message: 'User deleted', id: req.params.id });
  }
);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `DELETE /admin/users/42` without step-up credentials):**
```json
{
  "error": "Step-up authentication required for this operation",
  "code": "STEP_UP_REQUIRED"
}
```

**Expected Output (with valid step-up credentials):**
```json
{ "message": "User deleted", "id": "42" }
```

**Audit log output:**
```json
{
  "timestamp": "2026-01-15T10:30:00.000Z",
  "action": "DELETE_USER",
  "adminId": "usr_123",
  "target": "42",
  "ip": "203.0.113.5"
}
```

**Why this output:** The destructive route requires the admin role, step-up authentication (password + MFA), and audit logging. A regular admin request without step-up credentials is rejected with `STEP_UP_REQUIRED`. With valid credentials, the deletion proceeds and is recorded in the audit log.

### Real-World Cases

- **Banking:** Admins cannot approve their own transactions (separation of duties).
- **Healthcare:** Accessing patient records requires a reason and is audit-logged.
- **Cloud providers:** Destructive operations require MFA confirmation.
- **SaaS:** Tenant admins are scoped to their tenant — not global superusers.

---

## Core Concept 5: Hierarchical Roles

### Definitions

**Core Definition:** Hierarchical roles allow superior roles to automatically inherit all permissions from subordinate roles, so that a senior role does not need to re-list every permission it shares with a junior role.

**Technical Definition:** In hierarchical RBAC, roles are arranged in a directed acyclic graph (DAG) where an edge from role A to role B means "A inherits all of B's permissions." A user with role A effectively has role A's permissions **plus** role B's permissions. This prevents flat, bloated role tables where every role must independently list every permission it needs. Inheritance can be **transitive** (A inherits from B, B inherits from C, so A inherits from C) or **limited** (depth-1 inheritance only). The role hierarchy is defined in the database or configuration, and the permission-checking middleware expands the user's roles to include all inherited roles before checking.

**Beginner-Friendly Explanation:** Hierarchical roles are like a corporate ladder. A "Senior Manager" automatically has all the permissions of a "Manager," who automatically has all the permissions of a "Team Lead," who automatically has all the permissions of an "Employee." You don't have to list "can access the break room" for every single role — the Employee role has it, and every role above inherits it. This keeps the role table small and manageable.

### Purposes

- To design systems where superior roles automatically inherit permissions from subordinate roles.
- To prevent flat, bloated role tables where every role lists every permission.
- To simplify permission management — adding a permission to a base role grants it to all superior roles.
- To model organisational hierarchies naturally.

### Syntax Rules and Structure

#### Role Hierarchy Definition

```javascript
const ROLE_HIERARCHY = {
  user: [],                                // Base role — no parents
  manager: ['user'],                       // Manager inherits from user
  admin: ['manager'],                      // Admin inherits from manager
  superadmin: ['admin']                    // Superadmin inherits from admin
};
```

#### Expanding Inherited Roles

```javascript
function expandRoles(userRoles, hierarchy = ROLE_HIERARCHY) {
  const expanded = new Set();

  function addRole(role) {
    if (expanded.has(role)) return;
    expanded.add(role);

    // Recursively add parent roles (inherited)
    for (const parent of hierarchy[role] || []) {
      addRole(parent);
    }
  }

  for (const role of userRoles) {
    addRole(role);
  }

  return Array.from(expanded);
}
```

| User Roles | Expanded Roles | Effective Permissions |
|-----------|---------------|----------------------|
| `['user']` | `['user']` | User permissions. |
| `['manager']` | `['manager', 'user']` | Manager + User permissions. |
| `['admin']` | `['admin', 'manager', 'user']` | Admin + Manager + User permissions. |
| `['admin', 'user']` | `['admin', 'manager', 'user']` | Same as `['admin']` — no duplicates. |

**Rules:**
- Define the hierarchy as a directed acyclic graph (DAG) — no circular inheritance.
- Expand roles **before** checking permissions — the check should see all inherited roles.
- Cache expanded roles per request to avoid recomputation.
- Use **transitive** inheritance for simplicity, or **limited** inheritance (depth-1) for stricter control.
- Document the hierarchy — it is a critical part of the authorization model.

**Constraints:**
- Deep hierarchies can make permission debugging harder — trace the effective permissions.
- Circular inheritance (A inherits B, B inherits A) must be prevented — validate the hierarchy at startup.
- Role expansion adds computation per request — cache the result for the request lifecycle.

### Annotated Code Example

```javascript
// hierarchical-roles.js
const express = require('express');
const app = express();

// Role hierarchy
const ROLE_HIERARCHY = {
  user: [],
  manager: ['user'],
  admin: ['manager'],
  superadmin: ['admin']
};

// Expand roles to include inherited roles
function expandRoles(userRoles) {
  const expanded = new Set();

  function addRole(role) {
    if (expanded.has(role)) return;
    expanded.add(role);
    for (const parent of ROLE_HIERARCHY[role] || []) {
      addRole(parent);
    }
  }

  for (const role of userRoles) addRole(role);
  return Array.from(expanded);
}

// Role middleware with hierarchy expansion
function checkRole(...allowedRoles) {
  return (req, res, next) => {
    const expanded = expandRoles(req.user.roles || []);

    console.log('Expanded roles:', expanded);

    const hasRole = allowedRoles.some(r => expanded.includes(r));
    if (!hasRole) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Simulated auth
app.use((req, res, next) => {
  req.user = { id: 'usr_1', roles: (req.headers['x-roles'] || 'user').split(',') };
  next();
});

// Admin route — admin inherits from manager, so admin passes
app.get('/admin/dashboard',
  checkRole('manager', 'admin'),
  (req, res) => res.json({ dashboard: 'data' })
);

// User route — admin inherits user, so admin passes
app.get('/user/profile',
  checkRole('user'),
  (req, res) => res.json({ profile: 'data' })
);

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /admin/dashboard` with `X-Roles: admin`):**
```
Expanded roles: [ 'admin', 'manager', 'user' ]
{ "dashboard": "data" }
```

**Expected Output (for `GET /user/profile` with `X-Roles: admin`):**
```
Expanded roles: [ 'admin', 'manager', 'user' ]
{ "profile": "data" }
```

**Expected Output (for `GET /admin/dashboard` with `X-Roles: user`):**
```
Expanded roles: [ 'user' ]
{ "error": "Insufficient permissions" }
```

**Why this output:** The `admin` role expands to `['admin', 'manager', 'user']` because of the hierarchy. The admin passes both the manager/admin check and the user check. The `user` role expands to `['user']` only — it fails the manager/admin check because it does not include the `manager` or `admin` role.

### Real-World Cases

- **Corporate systems:** `employee` → `team_lead` → `manager` → `director` → `executive`.
- **E-commerce:** `customer` → `premium_customer` → `vip_customer` → `admin`.
- **Content platforms:** `reader` → `author` → `editor` → `publisher` → `admin`.
- **SaaS platforms:** `viewer` → `member` → `manager` → `owner`.

---

## References

- NIST RBAC Standard (INCITS 359) — https://csrc.nist.gov/projects/role-based-access-control
- NIST — Role-Based Access Control (RBAC) Overview — https://csrc.nist.gov/projects/role-based-access-control
- OWASP Authorization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- OWASP Top 10 (2021) — A01: Broken Access Control — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- Auth0 — Role-Based Access Control (RBAC) — https://auth0.com/docs/manage-users/access-control/rbac
- Casbin — Authorization Library for Node.js — https://casbin.org/
- node-casbin — npm — https://www.npmjs.com/package/casbin
- accesscontrol — npm — https://www.npmjs.com/package/accesscontrol
- @rbac/rbac — npm — https://www.npmjs.com/package/@rbac/rbac
- role-acl — npm — https://www.npmjs.com/package/role-acl
- Permify — RBAC vs ABAC vs ReBAC — https://permify.co/post/rbac-vs-abac-vs-rebac/
- Permit.io — RBAC vs ABAC: A Comprehensive Guide — https://www.permit.io/blog/rbac-vs-abac
- Oso — Role-Based Access Control (RBAC) — https://www.osohq.com/academy/role-based-access-control-rbac
- GitHub — rbac-express-auth — https://www.npmjs.com/package/rbac-express-auth
- Express.js — Using Middleware — https://expressjs.com/en/guide/using-middleware.html
- Stack Overflow — Hierarchical Role-Based Access Control in Node.js — https://stackoverflow.com/questions/43021590/
- CWE-269 — Improper Privilege Management — https://cwe.mitre.org/data/definitions/269.html
- CWE-250 — Execution with Unnecessary Privileges — https://cwe.mitre.org/data/definitions/250.html
- RFC 9110 — HTTP Semantics (403 Forbidden) — https://www.rfc-editor.org/rfc/rfc9110#section-15.5.4