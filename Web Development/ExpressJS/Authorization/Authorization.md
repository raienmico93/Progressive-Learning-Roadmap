# Express.js Authorization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Authorization is the process of determining whether an authenticated user has permission to access a specific resource or perform a particular action. While authentication verifies who the user is, authorization decides what that user can do within the application .

**Technical Definition**
In Express.js, authorization is implemented as middleware that runs **after** authentication and inspects the authenticated principal—conventionally available as `req.user`—against a policy defined by roles, permissions, or resource attributes. The middleware either calls `next()` to allow the request to proceed or terminates the cycle with `403 Forbidden` (authenticated but not permitted) or `401 Unauthorized` (no authenticated principal). Express deliberately ships with no built-in authorization layer, making explicit, well-structured middleware the developer's responsibility . Authorization models range from coarse role checks to granular permission matrices and resource-ownership verification .

**Beginner-Friendly Explanation**
Authentication asks “Who are you?” Authorization asks “Are you allowed to do this?” In Express, you implement authorization as a function that runs before your route handler. It looks at the logged-in user and decides whether the request should continue or be rejected. For example, an admin can delete users, an editor can create posts, and a regular user can only read. Without authorization, every logged-in user could do everything—a critical security flaw .

---

### Key Characteristics

- **Post-authentication** — Authorization middleware must run after authentication has populated `req.user`; it cannot operate on an anonymous request .
- **Middleware-centric** — Authorization is expressed as Express middleware with the `(req, res, next)` signature, composed into chains before route handlers .
- **Policy-driven** — Decisions are based on roles, permissions, resource ownership, or a combination of these attributes .
- **Deny-by-default** — The safest posture is to deny access unless a policy explicitly grants it, following least privilege .
- **Two failure modes** — `401 Unauthorized` for missing/invalid authentication; `403 Forbidden` for an authenticated user who lacks permission .

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- A working authentication mechanism (JWT, session, or API key) that populates `req.user` or an equivalent principal property.
- Understanding of Express middleware and the `(req, res, next)` signature.
- Familiarity with `req.params`, `req.body`, and database access patterns.
- (Optional) `jsonwebtoken` for embedding roles and permissions in tokens .

---

### Related Programming Areas

- **Authentication** — Authorization depends on a trustworthy identity source.
- **Role-Based Access Control (RBAC)** — The most common authorization model in Express applications .
- **Permission-Based Authorization** — Granular actions such as `read:posts` or `delete:users` .
- **Resource-Based Authorization** — Ownership and attribute checks for specific objects .
- **API security** — Broken access control has been the #1 OWASP Top 10 category since 2021 .
- **Audit logging** — Recording authorization denials for security monitoring.

---

## Core Concept 1: Authorization

### Definitions

**Core Definition**
Authorization is the security decision that grants or denies an authenticated principal access to a resource or operation based on policy.

**Technical Definition**
In Express.js, authorization is implemented as middleware that receives `(req, res, next)`. After authentication has attached the principal to `req.user`, authorization middleware evaluates the principal's roles, permissions, or ownership of the requested resource. If the policy permits, it calls `next()`; if the principal is missing, it returns `401`; if the principal is present but not permitted, it returns `403` . Authorization logic should be centralized at the routing layer so that access rules are visible and auditable .

**Beginner-Friendly Explanation**
Authorization is the bouncer inside the venue. Authentication got you through the front door by checking your ticket. Authorization checks whether your ticket is VIP, whether you are on the guest list for the backstage area, and whether you are allowed into the specific room you are trying to enter.

---

### Purposes

- To prevent authenticated users from accessing resources or operations beyond their privileges.
- To enforce least privilege, ensuring users receive only the access they need.
- To protect sensitive operations such as user management, billing, and data deletion.
- To provide clear, consistent `403` responses for policy violations.

---

### Syntax Rules and Structure

#### Complete General Syntax: Authorization Middleware

```javascript
function authorizePolicy(policyFn) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    if (!policyFn(req.user, req)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `req.user` | Authenticated principal populated by authentication middleware |
| `policyFn` | Function that returns `true` if access is allowed |
| `401` | No authenticated principal |
| `403` | Authenticated but not permitted |

#### Syntax Rules

- Authorization middleware **must** run after authentication middleware in the route chain.
- Return `401` only when `req.user` is absent; return `403` when it is present but lacks permission .
- Call `next()` only after the policy check passes.
- Never trust client-supplied user identifiers (`req.body.userId`) for authorization; use the server-side session identity `req.user` or `req.me` .

#### Constraints and Limitations

- Express has no built-in authorization layer; every protected route must explicitly attach authorization middleware .
- Authorization checks that query the database on every request add latency; caching or token-embedded claims may be necessary.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Basic Role Authorization

```javascript
const express = require('express');
const app = express();

// Authentication middleware (simplified)
function authenticate(req, res, next) {
  req.user = req.headers['x-user'] ? JSON.parse(req.headers['x-user']) : null;
  next();
}

// Authorization middleware factory
function requireRole(role) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Unauthorized: No user logged in' });
    }
    if (req.user.role !== role) {
      return res.status(403).json({ message: `Forbidden: Requires ${role} role` });
    }
    next();
  };
}

app.get('/admin/dashboard',
  authenticate,
  requireRole('admin'),
  (req, res) => res.send('Admin Dashboard')
);

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Authentication** — `authenticate` populates `req.user` from a header (in production, from a JWT or session).
2. **Authorization factory** — `requireRole('admin')` closes over the allowed role and returns middleware.
3. **Policy check** — The middleware rejects anonymous users with `401` and non-admins with `403`.

**Expected Output**

```
GET /admin/dashboard  (X-User: {"role":"admin"})  → 200 "Admin Dashboard"
GET /admin/dashboard  (X-User: {"role":"user"})   → 403 { "message": "Forbidden: Requires admin role" }
GET /admin/dashboard  (no header)                 → 401 { "message": "Unauthorized: No user logged in" }
```

**Why This Output Occurs**

- The factory produces a middleware that compares `req.user.role` with the required role; missing identity yields `401`, insufficient role yields `403` .

---

### Real-World Cases

- **Admin panels** — Restricting `/admin/*` routes to administrators.
- **Content management** — Allowing editors to create posts but not manage users.
- **Billing systems** — Limiting invoice access to finance roles.

---

## Core Concept 2: Roles

### Definitions

**Core Definition**
A role is a named collection of permissions that is assigned to a user, representing their responsibilities or job function within the application.

**Technical Definition**
Roles are the central abstraction in Role-Based Access Control. Common roles include `admin` (full access), `editor` (create and modify content but not manage users), `user` (basic access), and `guest` (limited public access) . Roles are typically stored on the user record and embedded in the JWT payload so that authorization middleware can read them without a database lookup on every request . Users may hold multiple roles, requiring middleware that checks for “any of” a set of allowed roles .

**Beginner-Friendly Explanation**
A role is a job title. An “admin” can do everything, an “editor” can create and edit content, and a “user” can read. Instead of assigning dozens of individual permissions to each person, you assign them a role, and the role carries the permissions.

---

### Purposes

- To simplify authorization by grouping permissions into manageable, human-readable categories.
- To support multi-role users who may have different responsibilities in different contexts.
- To enable readable route definitions such as `checkRole('admin', 'editor')` .
- To align access control with organizational structure and business terminology.

---

### Syntax Rules and Structure

#### Complete General Syntax: Multi-Role Middleware Factory

```javascript
function requireAnyRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Unauthorized: No user logged in' });
    }
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        message: `Forbidden: Requires one of these roles: ${allowedRoles.join(', ')}`
      });
    }
    next();
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `...allowedRoles` | Rest parameter collecting one or more permitted roles |
| `allowedRoles.includes(req.user.role)` | Checks whether the user's role is in the allowed set |
| `403` | Authenticated but role not permitted |

#### Syntax Rules

- Use rest parameters (`...allowedRoles`) to support one or many roles in a single middleware .
- Role names should be lowercase, stable, and documented.
- If a user can have multiple roles, store them as an array and check for intersection.
- Avoid hard-coding role checks inside route handlers; use middleware factories .

#### Constraints and Limitations

- Roles are coarse-grained; a role may grant more access than a specific user needs.
- Role definitions in JWTs become stale if roles change before token expiration; short token lifetimes or server-side role lookups mitigate this.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Multi-Role Authorization

```javascript
const express = require('express');
const app = express();

function authenticate(req, res, next) {
  req.user = req.headers['x-user'] ? JSON.parse(req.headers['x-user']) : null;
  next();
}

function requireAnyRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Unauthorized: No user logged in' });
    }
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        message: `Forbidden: Requires one of: ${allowedRoles.join(', ')}`
      });
    }
    next();
  };
}

app.get('/reports',
  authenticate,
  requireAnyRole('admin', 'manager'),
  (req, res) => res.send('Reports Data')
);

app.listen(3000);
```

**Expected Output**

```
GET /reports  (X-User: {"role":"admin"})    → 200 "Reports Data"
GET /reports  (X-User: {"role":"manager"})  → 200 "Reports Data"
GET /reports  (X-User: {"role":"user"})     → 403 { "message": "Forbidden: Requires one of: admin, manager" }
```

**Why This Output Occurs**

- The rest parameter collects both allowed roles; the user's role must be one of them .

---

### Real-World Cases

- **Reports dashboards** — Accessible to both admins and managers.
- **Content pipelines** — Editors and authors can create content; only admins can delete.
- **Support systems** — Agents and supervisors can view tickets; only supervisors can reassign.

---

## Core Concept 3: Permissions

### Definitions

**Core Definition**
A permission is a granular authorization rule describing a specific action on a specific resource, such as `read:posts`, `create:posts`, or `delete:users`.

**Technical Definition**
Permissions provide finer-grained control than roles. Instead of checking whether a user is an “editor,” authorization middleware checks whether the user holds the specific permission required for the operation . Permissions are often stored as an array on the user object or derived from a role-permission mapping table . The naming convention follows a `resource:action` pattern, enabling clear, auditable policy definitions.

**Beginner-Friendly Explanation**
A permission is a specific key to a specific door. A role is a keyring that holds many keys. Instead of saying “editors can do X,” you say “this operation requires the `delete:posts` key,” and you grant that key to whoever needs it.

---

### Purposes

- To enforce least privilege by granting only the exact actions a user needs.
- To support fine-grained authorization independent of job titles.
- To enable policy changes without redefining roles.
- To produce clear, auditable authorization rules at the route level.

---

### Syntax Rules and Structure

#### Complete General Syntax: Permission-Checking Middleware

```javascript
function checkPermission(permission) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Unauthorized: No user logged in' });
    }
    if (!req.user.permissions || !req.user.permissions.includes(permission)) {
      return res.status(403).json({
        message: `Forbidden: Missing required permission: ${permission}`
      });
    }
    next();
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `permission` | The required permission string (e.g., `'delete:posts'`) |
| `req.user.permissions` | Array of permissions held by the authenticated user |
| `403` | User lacks the specific permission |

#### Syntax Rules

- Use a consistent naming convention such as `resource:action`.
- Store permissions in an array on the user or derive them from a role-permission map.
- Check permissions at the route level, not inside business logic.
- Combine permission checks with role checks when both coarse and fine-grained control are needed.

#### Constraints and Limitations

- Large permission sets increase token size if embedded in JWTs.
- Role-permission mappings must be kept in sync with route requirements.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Permission-Based Route Protection

```javascript
const express = require('express');
const app = express();

function authenticate(req, res, next) {
  req.user = req.headers['x-user'] ? JSON.parse(req.headers['x-user']) : null;
  next();
}

function checkPermission(permission) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Unauthorized: No user logged in' });
    }
    if (!req.user.permissions || !req.user.permissions.includes(permission)) {
      return res.status(403).json({ message: `Forbidden: Missing ${permission}` });
    }
    next();
  };
}

app.delete('/posts/:id',
  authenticate,
  checkPermission('delete:posts'),
  (req, res) => res.send('Post deleted')
);

app.listen(3000);
```

**Expected Output**

```
DELETE /posts/1  (X-User: {"permissions":["delete:posts"]})  → 200 "Post deleted"
DELETE /posts/1  (X-User: {"permissions":["read:posts"]})   → 403 { "message": "Forbidden: Missing delete:posts" }
DELETE /posts/1  (no user)                                   → 401 { "message": "Unauthorized: No user logged in" }
```

**Why This Output Occurs**

- The middleware checks the `permissions` array for the exact required permission .

---

### Real-World Cases

- **Granular API access** — Allowing a service account to `read:users` but not `delete:users`.
- **Delegated administration** — Granting `manage:settings` without the broader `admin` role.
- **Compliance environments** — Auditing exact permissions granted to each principal.

---

## Core Concept 4: RBAC (Role-Based Access Control)

### Definitions

**Core Definition**
Role-Based Access Control (RBAC) is an authorization model in which permissions are associated with roles, and users are assigned to one or more roles .

**Technical Definition**
RBAC introduces an indirection layer between users and permissions: users are assigned roles, and roles are mapped to permissions. In Express.js, RBAC is implemented through middleware factories that check the user's role against the required role for a route . A common implementation separates **token verification** from **role checking**, producing two independent middleware functions: `verifyToken` attaches the decoded JWT to `req.user`, and `checkRole` compares `req.user.role` against the allowed roles . This separation allows routes to require authentication alone, or authentication plus a specific role.

**Beginner-Friendly Explanation**
RBAC is the standard “job title” system. You don't give each person individual permissions. Instead, you create roles like “admin” and “editor,” decide what each role can do, and assign people to roles. When someone tries to do something, the system checks their role.

---

### Purposes

- To centralize and simplify access control through role definitions.
- To enable readable, declarative route protection (`checkRole('admin')`).
- To support multi-tenant and multi-tier applications with distinct role hierarchies .
- To reduce the risk of accidentally granting excessive permissions.

---

### Syntax Rules and Structure

#### Complete General Syntax: RBAC Middleware Factory

```javascript
const checkRole = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Not authenticated' });
    }
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        message: `Access denied. Required: ${allowedRoles.join(' or ')}`
      });
    }
    next();
  };
};
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `...allowedRoles` | Roles permitted to access the route |
| `req.user.role` | Role embedded in the verified JWT or session |
| `403` | Authenticated but role not permitted |

#### Syntax Rules

- **Separate authentication from authorization** — `verifyToken` and `checkRole` should be distinct middleware .
- Embed roles in the JWT payload at login so every request carries the role without a database lookup .
- Use `checkRole(...allowedRoles)` at the route level so permissions are visible in the route definition .
- Return `403` (not `401`) when an authenticated user lacks the role .

#### Constraints and Limitations

- JWT-embedded roles can become stale if a user's role changes before token expiration.
- RBAC is coarse-grained; fine-grained actions may require additional permission checks.
- A single-role model cannot express users who need multiple roles simultaneously.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: JWT-Based RBAC with Separated Middleware

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();
app.use(express.json());

const SECRET = process.env.JWT_SECRET || 'dev-secret';

// Authentication middleware
function verifyToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  if (!token) {
    return res.status(401).json({ message: 'Access denied. No token provided.' });
  }
  try {
    req.user = jwt.verify(token, SECRET);
    next();
  } catch (err) {
    return res.status(403).json({ message: 'Invalid or expired token.' });
  }
}

// Authorization middleware factory
function checkRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Not authenticated.' });
    }
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        message: `Access denied. Required role: ${allowedRoles.join(' or ')}.`
      });
    }
    next();
  };
}

// Login: embed role in JWT
app.post('/login', (req, res) => {
  const { username, role } = req.body;
  const token = jwt.sign({ username, role }, SECRET, { expiresIn: '1h' });
  res.json({ token });
});

// Protected routes
app.get('/content',
  verifyToken,
  checkRole('user', 'editor', 'admin'),
  (req, res) => res.json({ content: [] })
);

app.delete('/content/:id',
  verifyToken,
  checkRole('admin'),
  (req, res) => res.json({ deleted: true })
);

app.listen(3000);
```

**Expected Output**

```
POST /login  { "username": "alice", "role": "admin" }
→ 200 { "token": "eyJ..." }

DELETE /content/1  (Authorization: Bearer <admin-token>)
→ 200 { "deleted": true }

DELETE /content/1  (Authorization: Bearer <user-token>)
→ 403 { "message": "Access denied. Required role: admin." }

DELETE /content/1  (no token)
→ 401 { "message": "Access denied. No token provided." }
```

**Why This Output Occurs**

- `verifyToken` validates the JWT and attaches `req.user`; `checkRole('admin')` compares the embedded role and returns `403` for non-admins .

---

### Real-World Cases

- **Multi-tenant SaaS** — Roles such as `tenant_admin`, `member`, and `viewer` scoped to each tenant .
- **Content platforms** — `admin`, `editor`, and `user` roles with distinct endpoint access .
- **Enterprise APIs** — Tiered roles with numeric levels where higher levels inherit lower-level permissions .

---

## Core Concept 5: Resource-Based Authorization

### Definitions

**Core Definition**
Resource-based authorization determines access based on the relationship between the authenticated user and a **specific resource instance**, such as ownership, tenant membership, or attribute matching.

**Technical Definition**
Unlike role-based checks, which are static for a route, resource-based authorization requires loading or identifying the target resource and evaluating a policy against it. A common pattern is an `authorizeResource(getResourceOwnerId)` middleware factory that receives a function for extracting the resource owner's ID, loads the resource, and compares it with `req.user.id` . Administrators typically bypass resource-level checks. The rule of thumb is to **never trust a client-supplied user ID**; always derive ownership from the server-side identity .

**Beginner-Friendly Explanation**
Resource-based authorization checks “Is this yours?” Even if you are allowed to edit posts in general, you can only edit **your own** posts. The middleware loads the post, checks its author, and compares it with you.

---

### Purposes

- To prevent horizontal privilege escalation (one user accessing another user's data) — the OWASP “Broken Object Level Authorization” risk .
- To enforce multi-tenant isolation by checking tenant IDs.
- To allow administrators to bypass ownership checks for support and moderation.
- To centralize ownership logic so every resource route uses the same rule .

---

### Syntax Rules and Structure

#### Complete General Syntax: Ownership Middleware Factory

```javascript
function authorizeResource(getResourceOwnerId) {
  return async (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    if (req.user.role === 'admin') {
      return next(); // admins bypass ownership
    }
    try {
      const ownerId = await getResourceOwnerId(req);
      if (ownerId === null) {
        return res.status(404).json({ error: 'Resource not found' });
      }
      if (ownerId !== req.user.id) {
        return res.status(403).json({ error: 'You do not own this resource' });
      }
      next();
    } catch (err) {
      next(err);
    }
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `getResourceOwnerId` | Async function that loads the resource owner from the database |
| `req.user.id` | Server-side authenticated identity |
| `404` | Resource does not exist |
| `403` | Resource exists but belongs to another user |
| Admin bypass | Admins skip ownership checks |

#### Syntax Rules

- **Never** use a user ID from `req.body` or `req.query` to verify ownership; use `req.user.id` .
- Return `404` when the resource does not exist, and `403` when it exists but is owned by someone else.
- Allow administrators to bypass ownership checks where appropriate .
- Load the resource once and attach it to `req` if downstream handlers need it.
- Fail fast: reject the request before validating the request body .

#### Constraints and Limitations

- Resource-based checks require a database lookup per request; caching or eager loading may be necessary.
- Ownership rules differ per resource type; the `getResourceOwnerId` function must be adapted for each.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Ownership Check for Articles

```javascript
const express = require('express');
const app = express();

// Simulated database
const articles = { '1': { id: '1', authorId: 'user-1', title: 'Hello' } };

function authenticate(req, res, next) {
  req.user = req.headers['x-user'] ? JSON.parse(req.headers['x-user']) : null;
  next();
}

function authorizeResource(getResourceOwnerId) {
  return async (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    if (req.user.role === 'admin') {
      return next();
    }
    const ownerId = await getResourceOwnerId(req);
    if (ownerId === null) {
      return res.status(404).json({ error: 'Resource not found' });
    }
    if (ownerId !== req.user.id) {
      return res.status(403).json({ error: 'You do not own this resource' });
    }
    next();
  };
}

async function getArticleOwner(req) {
  const article = articles[req.params.id];
  return article ? article.authorId : null;
}

app.put('/articles/:id',
  authenticate,
  authorizeResource(getArticleOwner),
  (req, res) => res.json({ updated: true })
);

app.listen(3000);
```

**Expected Output**

```
PUT /articles/1  (X-User: {"id":"user-1"})        → 200 { "updated": true }
PUT /articles/1  (X-User: {"id":"user-2"})        → 403 { "error": "You do not own this resource" }
PUT /articles/1  (X-User: {"id":"x","role":"admin"}) → 200 { "updated": true }
PUT /articles/99 (X-User: {"id":"user-1"})        → 404 { "error": "Resource not found" }
```

**Why This Output Occurs**

- The middleware loads the article's `authorId`, compares it with `req.user.id`, allows admins to bypass, and returns `404` for missing resources .

---

### Real-World Cases

- **User profiles** — Allowing users to edit only their own profile.
- **Multi-tenant SaaS** — Ensuring a tenant admin can only access resources within their tenant.
- **Document management** — Restricting document edits to the document owner or an administrator.

---

## Core Concept 6: Authorization Middleware

### Definitions

**Core Definition**
Authorization middleware is the Express middleware layer—typically produced by a factory function—that enforces an authorization policy before a route handler executes.

**Technical Definition**
Authorization middleware factories accept policy parameters (roles, permissions, or resource-loader functions) and return middleware closures that evaluate the policy against `req.user` and the request. The factory pattern enables reuse across routes with different policy requirements while keeping route definitions declarative . Middleware can be mounted at the **route level** for fine-grained control or at the **router level** to create an “authentication wall” that protects all subsequent routes in a router .

**Beginner-Friendly Explanation**
Authorization middleware is the reusable gatekeeper. Instead of writing an `if` check in every route handler, you create a factory once—`requireRole('admin')` or `requirePermission('delete:posts')`—and plug it into any route that needs it. The route definition itself tells you who can access it.

---

### Purposes

- To keep authorization rules visible at the routing layer, where they are easy to audit .
- To avoid duplicating authorization logic across route handlers.
- To enable composable chains such as `authenticate, checkRole('admin'), requirePermission('users:delete')`.
- To support both route-level and router-level protection boundaries.

---

### Syntax Rules and Structure

#### Complete General Syntax: Middleware Factory and Mounting

```javascript
// Route-level authorization
app.delete('/users/:id',
  authenticate,
  checkRole('admin'),
  requirePermission('users:delete'),
  deleteUserHandler
);

// Router-level authorization wall
const protectedRouter = express.Router();
protectedRouter.use(authenticate, checkRole('admin'));
protectedRouter.get('/dashboard', dashboardHandler);
protectedRouter.get('/users', usersHandler);
app.use('/admin', protectedRouter);
```

**Component Breakdown**

| Pattern | Effect |
|---------|--------|
| Route-level chain | Each route declares its own authorization requirements |
| Router-level wall | All routes after `use()` inherit the same guards |
| Factory closure | Policy parameters are captured once at setup time |

#### Syntax Rules

- **Order matters** — Authentication must precede authorization; role checks must precede permission checks that depend on roles.
- **Use factories** — `checkRole(...)`, `requirePermission(...)`, and `authorizeResource(...)` return middleware; never write inline `if` checks in handlers .
- **Mount at the narrowest scope** — Apply authorization to the router or route that needs it, not globally.
- **Fail fast** — Reject unauthorized requests before body validation or database mutations .

#### Constraints and Limitations

- Router-level walls apply to **all** subsequent routes; ensure the boundary is placed correctly.
- Overly complex middleware chains can obscure the policy; keep chains short and readable.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Router-Level Authorization Wall

```javascript
const express = require('express');
const app = express();

function authenticate(req, res, next) {
  req.user = req.headers['x-user'] ? JSON.parse(req.headers['x-user']) : null;
  next();
}

function checkRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) return res.status(401).json({ message: 'Not authenticated' });
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ message: 'Forbidden' });
    }
    next();
  };
}

// Public router
const publicRouter = express.Router();
publicRouter.get('/posts', (req, res) => res.json({ posts: [] }));

// Protected admin router
const adminRouter = express.Router();
adminRouter.use(authenticate, checkRole('admin'));
adminRouter.get('/dashboard', (req, res) => res.json({ dashboard: true }));
adminRouter.delete('/posts/:id', (req, res) => res.json({ deleted: true }));

app.use('/api', publicRouter);
app.use('/admin', adminRouter);

app.listen(3000);
```

**Expected Output**

```
GET /api/posts          → 200 { "posts": [] }          (public)
GET /admin/dashboard    (X-User: {"role":"admin"}) → 200 { "dashboard": true }
GET /admin/dashboard    (X-User: {"role":"user"})  → 403 { "message": "Forbidden" }
GET /admin/dashboard    (no header)                → 401 { "message": "Not authenticated" }
```

**Why This Output Occurs**

- `adminRouter.use(authenticate, checkRole('admin'))` creates a boundary; every route registered after it inherits the guards .

---

### Real-World Cases

- **API versioning** — A protected router for `/api/v1/admin` with role guards applied once.
- **Feature modules** — Each feature router applies its own authorization middleware.
- **Multi-tenant routing** — A tenant-resolution middleware followed by tenant-scoped authorization.

---

## Core Concept 7: Authorization Best Practices

### Definitions

**Core Definition**
Authorization best practices are the design and implementation patterns that keep access control complete, auditable, secure, and maintainable across an Express application.

**Technical Definition**
Best practices include: denying access by default and granting only explicitly permitted actions (least privilege) ; separating authentication from authorization into distinct middleware ; keeping authorization rules at the routing layer rather than buried in business logic ; failing fast before validating request data or mutating state ; never trusting client-supplied user identifiers ; using factories for reusable policy middleware; testing negative authorization paths; and logging authorization denials for security monitoring. Broken access control has been the #1 OWASP Top 10 risk since 2021, and Express's lack of built-in authorization makes disciplined patterns essential .

**Beginner-Friendly Explanation**
Best practices are the habits that keep authorization from becoming a security hole. Start from “no access” and only grant what is needed. Keep authentication and authorization separate. Put the rules where you can see them—next to the route. Reject unauthorized requests immediately. Never let a user tell you who they are via the request body. And test that the “no” paths actually say no.

---

### Purposes

- To prevent broken access control, the most critical web application security risk .
- To ensure every protected route is covered by an explicit policy.
- To make authorization decisions visible, testable, and auditable.
- To reduce the risk of privilege escalation and data leakage.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Authorization Pipeline

```javascript
// 1. Authentication
app.use(authenticate);

// 2. Route-level authorization with explicit policy
app.delete('/users/:id',
  checkRole('admin'),
  requirePermission('users:delete'),
  deleteUserHandler
);

// 3. Resource-level authorization
app.put('/articles/:id',
  authorizeResource(getArticleOwner),
  updateArticleHandler
);

// 4. Deny-by-default fallback
app.use((req, res) => res.status(404).json({ error: 'Not found' }));
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Deny by default | No route is accessible unless explicitly guarded |
| Separation of concerns | `authenticate` vs. `checkRole` vs. `requirePermission` |
| Visible rules | Guards appear in the route definition |
| Fail fast | Guards run before validation and handlers |
| Server-side identity | Use `req.user`, never `req.body.userId` |

#### Syntax Rules

- **Deny by default** — Require explicit authorization for every protected route; do not assume authentication implies access .
- **Least privilege** — Grant the minimum permissions necessary; avoid broad roles when granular permissions suffice.
- **Separate authentication and authorization** — `verifyToken` attaches identity; `checkRole` and `requirePermission` decide access .
- **Keep rules at the routing layer** — Authorization middleware in the route definition is easier to audit than checks buried in services .
- **Fail fast** — Authorization should run before body validation and database mutations .
- **Never trust client-supplied identity** — Use `req.user.id`, not `req.body.userId` .
- **Test negative paths** — Verify that unauthorized users receive `403` and anonymous users receive `401`.
- **Log denials** — Record authorization failures for security monitoring and incident response.

#### Constraints and Limitations

- Role and permission checks embedded in JWTs become stale; short token lifetimes or server-side lookups mitigate this.
- Deny-by-default can be operationally difficult in legacy applications; a route inventory and gradual migration are required.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Authorization Pipeline with Negative Tests

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();
app.use(express.json());

const SECRET = process.env.JWT_SECRET || 'dev-secret';

function verifyToken(req, res, next) {
  const header = req.headers.authorization;
  const token = header && header.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(token, SECRET);
    next();
  } catch (err) {
    return res.status(403).json({ error: 'Invalid token' });
  }
}

function checkRole(...allowed) {
  return (req, res, next) => {
    if (!req.user) return res.status(401).json({ error: 'Not authenticated' });
    if (!allowed.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

function requirePermission(permission) {
  return (req, res, next) => {
    if (!req.user.permissions || !req.user.permissions.includes(permission)) {
      return res.status(403).json({ error: `Missing ${permission}` });
    }
    next();
  };
}

// Login
app.post('/login', (req, res) => {
  const { username, role, permissions } = req.body;
  const token = jwt.sign({ username, role, permissions }, SECRET, { expiresIn: '1h' });
  res.json({ token });
});

// Admin + permission protected route
app.delete('/users/:id',
  verifyToken,
  checkRole('admin'),
  requirePermission('users:delete'),
  (req, res) => res.json({ deleted: true })
);

// Deny-by-default fallback
app.use((req, res) => res.status(404).json({ error: 'Not found' }));

app.listen(3000);
```

**Expected Output**

```
POST /login  { "username": "alice", "role": "admin", "permissions": ["users:delete"] }
→ 200 { "token": "eyJ..." }

DELETE /users/1  (admin token with users:delete)   → 200 { "deleted": true }
DELETE /users/1  (admin token without users:delete) → 403 { "error": "Missing users:delete" }
DELETE /users/1  (user token)                        → 403 { "error": "Forbidden" }
DELETE /users/1  (no token)                          → 401 { "error": "No token" }
DELETE /users/1  (invalid token)                     → 403 { "error": "Invalid token" }
```

**Why This Output Occurs**

- The route chain applies authentication, role check, and permission check in order; each guard returns the correct `401` or `403` and only calls `next()` when the policy passes .

---

### Real-World Cases

- **Production APIs** — Deny-by-default route inventories with explicit role and permission guards.
- **Multi-tenant SaaS** — Tenant isolation enforced through resource-based authorization and server-side identity .
- **Regulated industries** — Authorization denials logged and audited for compliance.

---

## Authorization Model Comparison

| Model | Granularity | Implementation Complexity | Best For |
|-------|-------------|--------------------------|----------|
| **Role-based (RBAC)** | Coarse | Low | Admin panels, content management, simple tiered access  |
| **Permission-based** | Fine | Medium | Granular API access, delegated administration  |
| **Resource-based** | Per-instance | High | Ownership checks, multi-tenant isolation, IDOR prevention  |
| **Hybrid (role + permission + resource)** | Layered | High | Production systems requiring defense in depth |

---

## References

- Express Authorization · Compile-N-Run — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/7-express-authentication/9-express-authorization.mdx
- Express.js Middleware Patterns: Authentication and Authorization · Grizzly Peak Software — https://grizzlypeaksoftware.com/library/expressjs-middleware-patterns-authentication-and-authorization-e0hbci84
- How to Implement Role-Based Access Control in a Node.js REST API with JWT · freeCodeCamp — https://www.freecodecamp.org/news/role-based-access-control-nodejs-rest-api-jwt
- Authorization · rocambille/start-express-react GitHub Wiki — https://github-wiki-see.page/m/rocambille/start-express-react/wiki/Authorization
- Preventing Broken Access Control in Express.js · Safeguard — https://safeguard.sh/resources/blog/preventing-broken-access-control-in-express-js
- OWASP Authorization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- OWASP Top 10: Broken Access Control — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- How to Implement RBAC in an Express.js Application · Permit.io — https://www.permit.io/blog/how-to-implement-rbac-in-an-express-js-application
- secure-express-rbac · npm — https://www.npmjs.com/package/secure-express-rbac
- RBAC Middleware Factory · Stack Overflow — https://stackoverflow.com/questions/70934663/express-multiple-roles-middleware