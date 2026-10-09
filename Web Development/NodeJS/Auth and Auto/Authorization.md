# Authorization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Authorization (AuthZ) is the process of determining what an authenticated principal is permitted to do — which resources they can access and which actions they can perform — based on their roles, permissions, attributes, or relationships.

**Technical Definition:** Authorization is the security discipline that governs access control after authentication. It encompasses several models: **RBAC** (Role-Based Access Control) assigns permissions to roles and roles to users; **ABAC** (Attribute-Based Access Control) evaluates policies against attributes of the subject, resource, action, and environment; **ReBAC** (Relationship-Based Access Control) authorizes based on graph relationships between subjects and objects; **resource-based authorization** evaluates ownership and resource-specific rules; **tenant-based authorization** isolates data across multi-tenant boundaries. Authorization decisions are typically enforced at the policy enforcement point (PEP) — a middleware, guard, or interceptor — using policies defined by a policy decision point (PDP) or the application itself.

**Beginner-Friendly Explanation:** Authentication is showing your ID at the airport. Authorization is what your ticket allows you to do once inside — which gates you can access, which lounges you can use, and which flights you can board. **Roles** are like passenger classes (first class, economy). **Permissions** are like specific gate access codes. **Resource-based authorization** is like "only the person whose name is on the ticket can board." **Tenant-based authorization** is like "passengers on Airline A can't board Airline B's planes." **ABAC** is like "you can only enter the lounge during its opening hours." **ReBAC** is like "you can bring a guest because you're a family member of a member."

### Key Characteristics

- **Post-authentication:** Authorization always follows authentication — you must know *who* before deciding *what*.
- **Multiple models:** RBAC, ABAC, ReBAC, and resource/tenant-based authorization address different needs.
- **Defense in depth:** Authorization is enforced at multiple layers (gateway, controller, service, data layer).
- **Principle of least privilege:** Users should have only the permissions necessary to perform their tasks.
- **Deny by default:** If no rule explicitly grants access, deny it.
- **Centralised policy:** Policies should be defined in one place (PDP) and enforced consistently (PEP).
- **Auditability:** Every authorization decision should be logged for compliance and incident response.

### Prerequisites

- **Authentication fundamentals:** Sessions, tokens, JWT claims, identity establishment.
- **HTTP and API design:** Routes, resources, methods, and status codes.
- **Database fundamentals:** Tables, relationships, row-level security, and transactions.
- **Web security concepts:** Least privilege, privilege escalation, IDOR, CSRF.
- **Node.js fundamentals:** Express, NestJS, middleware, guards, and interceptors.
- **Policy engines (optional):** Casbin, OPA, Oso, Cerbos.
- **Graph theory basics (for ReBAC):** Nodes, edges, relationship traversal.

### Related Programming Areas

- **Authentication:** AuthZ always follows AuthN; identity is the foundation of authorization.
- **API design:** Authorization determines endpoint access and response filtering.
- **Multi-tenancy:** Tenant isolation is a form of authorization.
- **Data security:** Row-level security, column masking, and field-level access control.
- **Compliance:** SOC 2, HIPAA, PCI DSS, and GDPR mandate access control.
- **Observability:** Authorization decisions are logged for audit and monitoring.

### Core Concepts

1. **Roles** — assigning user classifications like Admin, Editor, or Guest.
2. **Permissions** — mapping discrete, granular actions to application routes and resources.
3. **RBAC (Role-Based Access Control)** — frameworks that assign permissions to roles and roles to users.
4. **Resource-based Authorization** — evaluating asset ownership dynamically (e.g., only the author can edit this post).
5. **Tenant-based Authorization** — isolating software data models within strict multi-tenant SaaS application structures.
6. **ABAC (Attribute-Based Access Control)** — granting access based on contextual parameters like time of day, IP address, or department.
7. **ReBAC (Relationship-Based Access Control)** — authorizing access based on deep graphs of user-to-object relationships.

---

## Core Concept 1: Roles

### Definitions

**Core Definition:** A role is a named classification (e.g., Admin, Editor, Guest) that groups a set of permissions and is assigned to users, simplifying access management.

**Technical Definition:** In RBAC, a role is an intermediary between users and permissions: permissions are assigned to roles, and roles are assigned to users. A user may have multiple roles, and roles may be hierarchical (e.g., Admin inherits Editor permissions). Roles are typically stored in a database (`roles`, `user_roles`, `role_permissions` tables) and included in authentication tokens or session data. Roles should be coarse-grained classifications of job function, not fine-grained permissions. Common patterns include static roles (defined at deployment) and dynamic roles (assigned at runtime based on context).

**Beginner-Friendly Explanation:** A role is like a job title at a company. A "Manager" can approve expenses and view reports. An "Employee" can submit expenses and view their own data. Instead of assigning each individual permission to each person, you assign them a role — and the role carries all the permissions. It's much easier to say "Alice is a Manager" than to list every single thing Alice can do.

### Purposes

- To simplify access management by grouping permissions into logical classifications.
- To enable consistent access control across the application.
- To support hierarchical roles (Admin inherits Editor).
- To make user provisioning and deprovisioning faster.
- To align access control with organisational job functions.
- To reduce the risk of misconfiguration by centralising permission assignments.

### Syntax Rules and Structure

#### Database Schema

```prisma
model User {
  id    String @id @default(uuid())
  email String @unique
  roles UserRole[]
}

model Role {
  id          String           @id @default(uuid())
  name        String           @unique  // "admin", "editor", "guest"
  description String?
  permissions RolePermission[]
  users       UserRole[]
}

model UserRole {
  userId String
  roleId String
  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  role   Role   @relation(fields: [roleId], references: [id], onDelete: Cascade)

  @@id([userId, roleId])
}

model Permission {
  id    String           @id @default(uuid())
  name  String           @unique  // "posts:create", "posts:edit:any"
  roles RolePermission[]
}

model RolePermission {
  roleId       String
  permissionId String
  role         Role       @relation(fields: [roleId], references: [id], onDelete: Cascade)
  permission   Permission @relation(fields: [permissionId], references: [id], onDelete: Cascade)

  @@id([roleId, permissionId])
}
```

#### Role Constants

```typescript
export const ROLES = {
  ADMIN: 'admin',
  EDITOR: 'editor',
  AUTHOR: 'author',
  GUEST: 'guest',
} as const;

export type RoleName = typeof ROLES[keyof typeof ROLES];
```

#### Role Assignment

```typescript
async function assignRole(userId: string, roleName: RoleName): Promise<void> {
  const role = await prisma.role.findUniqueOrThrow({ where: { name: roleName } });
  await prisma.userRole.upsert({
    where: { userId_roleId: { userId, roleId: role.id } },
    update: {},
    create: { userId, roleId: role.id },
  });
}
```

#### Syntax Rules

- **Roles should be coarse-grained** — job functions, not individual permissions.
- **Roles should be assigned to users, not directly to resources.**
- **Roles should be stored in the database** — not hardcoded in the application.
- **Roles should be included in auth tokens or sessions** — for fast authorization checks.
- **Roles should be hierarchical when appropriate** — Admin inherits Editor.
- **Role names should be lowercase, hyphenated** — `admin`, `content-editor`, `support-agent`.
- **Never use roles for resource-specific ownership** — use resource-based authorization instead.

#### Constraints and Limitations

- **Role explosion:** Too many fine-grained roles become unmanageable — use permissions for granularity.
- **Role assignment is coarse:** A user with `admin` has all admin permissions, even if they only need some.
- **Stale roles:** Roles cached in JWTs may be outdated until the token expires.
- **No contextual evaluation:** Roles alone cannot express "only during business hours" — use ABAC.
- **No relationship evaluation:** Roles alone cannot express "only the author" — use resource-based authorization.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Role Assignment and Retrieval (NestJS + Prisma)

```typescript
// roles.service.ts
@Injectable()
export class RolesService {
  constructor(private readonly prisma: PrismaService) {}

  async assignRole(userId: string, roleName: string): Promise<void> {
    const role = await this.prisma.role.findUniqueOrThrow({ where: { name: roleName } });
    await this.prisma.userRole.upsert({
      where: { userId_roleId: { userId, roleId: role.id } },
      update: {},
      create: { userId, roleId: role.id },
    });
  }

  async revokeRole(userId: string, roleName: string): Promise<void> {
    const role = await this.prisma.role.findUniqueOrThrow({ where: { name: roleName } });
    await this.prisma.userRole.deleteMany({
      where: { userId, roleId: role.id },
    });
  }

  async getUserRoles(userId: string): Promise<string[]> {
    const userRoles = await this.prisma.userRole.findMany({
      where: { userId },
      include: { role: true },
    });
    return userRoles.map((ur) => ur.role.name);
  }

  async getUsersWithRole(roleName: string): Promise<string[]> {
    const users = await this.prisma.userRole.findMany({
      where: { role: { name: roleName } },
      select: { userId: true },
    });
    return users.map((u) => u.userId);
  }
}
```

```typescript
// auth.service.ts — include roles in JWT
async login(email: string, password: string): Promise<LoginResult> {
  const user = await this.prisma.user.findUniqueOrThrow({ where: { email } });
  if (!(await this.passwordService.verify(password, user.passwordHash))) {
    throw new UnauthorizedException('Invalid credentials');
  }

  const roles = await this.rolesService.getUserRoles(user.id);

  const accessToken = await this.jwtService.signAsync(
    { sub: user.id, roles, type: 'access' },
    { algorithm: 'RS256', expiresIn: '15m' },
  );

  return { accessToken, user: { id: user.id, email: user.email, roles } };
}
```

**Expected behaviour:** `assignRole()` adds a role to a user. `getUserRoles()` retrieves a user's roles. The login flow includes roles in the JWT payload, so authorization checks can be performed statelessly.

**Why this works:** Roles are stored in the database (source of truth) and included in the token (for fast, stateless checks). Token expiration limits staleness. Role changes take effect on the next login (or via token revocation).

### Real-World Cases

- **Content management systems:** Admin, Editor, Author, Contributor roles.
- **SaaS platforms:** Owner, Admin, Member, Viewer roles.
- **E-commerce:** Customer, Vendor, Support Agent, Admin roles.
- **Enterprise:** Employee, Manager, Director, Executive roles with hierarchy.

---

## Core Concept 2: Permissions (Granular Actions)

### Definitions

**Core Definition:** A permission is a discrete, granular action that can be performed on a resource, expressed as `resource:action` (e.g., `posts:create`, `users:delete`).

**Technical Definition:** Permissions are the atomic units of authorization — they describe exactly what a principal can do. Permissions are assigned to roles (RBAC) or evaluated directly (ABAC/ReBAC). Permission naming follows a convention: `resource:action` (e.g., `posts:create`), `resource:action:scope` (e.g., `posts:edit:any` vs. `posts:edit:own`), or `resource.action` (e.g., `post.create`). Permissions can be grouped into scopes for OAuth 2.0 (`read:posts`, `write:posts`). The principle of least privilege requires granting only the permissions necessary for a role's function.

**Beginner-Friendly Explanation:** A permission is a specific "key" that opens a specific "door." `posts:create` is the key to create posts. `users:delete` is the key to delete users. Roles are keyrings — a role bundles a set of keys. Instead of giving a user 50 individual keys, you give them a role that has exactly the keys they need.

### Purposes

- To provide fine-grained control over what actions can be performed.
- To enable least-privilege access — grant only what's needed.
- To support audit and compliance — log exactly what permissions were used.
- To decouple role definitions from permission definitions — roles can change without changing permissions.
- To enable dynamic permission evaluation (ABAC/ReBAC).

### Syntax Rules and Structure

#### Permission Naming Convention

| Pattern | Example | Use Case |
|---------|---------|----------|
| `resource:action` | `posts:create` | Simple permissions |
| `resource:action:scope` | `posts:edit:any`, `posts:edit:own` | Scoped permissions |
| `resource.action` | `post.create` | Dot notation (alternative) |
| `scope` (OAuth) | `read:posts`, `write:posts` | OAuth 2.0 scopes |

#### Permission Constants

```typescript
export const PERMISSIONS = {
  POSTS_CREATE: 'posts:create',
  POSTS_READ: 'posts:read',
  POSTS_EDIT_ANY: 'posts:edit:any',
  POSTS_EDIT_OWN: 'posts:edit:own',
  POSTS_DELETE_ANY: 'posts:delete:any',
  POSTS_DELETE_OWN: 'posts:delete:own',
  USERS_READ: 'users:read',
  USERS_CREATE: 'users:create',
  USERS_DELETE: 'users:delete',
  ADMIN_ACCESS: 'admin:access',
} as const;

export type PermissionName = typeof PERMISSIONS[keyof typeof PERMISSIONS];
```

#### Role-Permission Mapping

```typescript
export const ROLE_PERMISSIONS: Record<RoleName, PermissionName[]> = {
  admin: [
    PERMISSIONS.POSTS_CREATE,
    PERMISSIONS.POSTS_READ,
    PERMISSIONS.POSTS_EDIT_ANY,
    PERMISSIONS.POSTS_DELETE_ANY,
    PERMISSIONS.USERS_READ,
    PERMISSIONS.USERS_CREATE,
    PERMISSIONS.USERS_DELETE,
    PERMISSIONS.ADMIN_ACCESS,
  ],
  editor: [
    PERMISSIONS.POSTS_CREATE,
    PERMISSIONS.POSTS_READ,
    PERMISSIONS.POSTS_EDIT_ANY,
    PERMISSIONS.POSTS_DELETE_ANY,
  ],
  author: [
    PERMISSIONS.POSTS_CREATE,
    PERMISSIONS.POSTS_READ,
    PERMISSIONS.POSTS_EDIT_OWN,
    PERMISSIONS.POSTS_DELETE_OWN,
  ],
  guest: [PERMISSIONS.POSTS_READ],
};
```

#### Permission Check

```typescript
function hasPermission(userPermissions: string[], required: string): boolean {
  return userPermissions.includes(required);
}

function hasAnyPermission(userPermissions: string[], required: string[]): boolean {
  return required.some((p) => userPermissions.includes(p));
}

function hasAllPermissions(userPermissions: string[], required: string[]): boolean {
  return required.every((p) => userPermissions.includes(p));
}
```

#### Syntax Rules

- **Use `resource:action` naming** — consistent and self-documenting.
- **Separate `own` vs. `any` scopes** — `posts:edit:own` vs. `posts:edit:any`.
- **Assign permissions to roles, not users** — keep the model clean.
- **Use constants, not strings** — prevent typos.
- **Document every permission** — its purpose and scope.
- **Audit permissions regularly** — remove unused permissions.
- **Enforce at the API layer** — never trust the client.

#### Constraints and Limitations

- **Permission explosion:** Hundreds of permissions become unmanageable — group into roles/scopes.
- **Permission checks are string-based:** Typos in permission names can lead to security holes.
- **No contextual evaluation:** Permissions alone cannot express "only during business hours" — use ABAC.
- **No relationship evaluation:** Permissions alone cannot express "only the author" — use resource-based authorization.
- **Caching complexity:** Permissions cached in tokens may be stale.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Permission Service with Caching (NestJS + Redis)

```typescript
// permissions.service.ts
@Injectable()
export class PermissionsService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly redis: Redis,
  ) {}

  async getUserPermissions(userId: string): Promise<string[]> {
    const cacheKey = `permissions:${userId}`;
    const cached = await this.redis.get(cacheKey);
    if (cached) return JSON.parse(cached);

    const userRoles = await this.prisma.userRole.findMany({
      where: { userId },
      include: {
        role: {
          include: {
            permissions: { include: { permission: true } },
          },
        },
      },
    });

    const permissions = Array.from(
      new Set(
        userRoles.flatMap((ur) =>
          ur.role.permissions.map((rp) => rp.permission.name),
        ),
      ),
    );

    await this.redis.set(cacheKey, JSON.stringify(permissions), 'EX', 300); // 5 minutes
    return permissions;
  }

  async invalidateCache(userId: string): Promise<void> {
    await this.redis.del(`permissions:${userId}`);
  }

  async hasPermission(userId: string, permission: string): Promise<boolean> {
    const permissions = await this.getUserPermissions(userId);
    return permissions.includes(permission);
  }
}
```

```typescript
// permissions.guard.ts
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private readonly permissionsService: PermissionsService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const required = this.reflector.get<string[]>('permissions', context.getHandler());
    if (!required || required.length === 0) return true;

    const req = context.switchToHttp().getRequest();
    const userId = req.user?.id;
    if (!userId) throw new UnauthorizedException();

    const hasAll = await Promise.all(
      required.map((p) => this.permissionsService.hasPermission(userId, p)),
    );

    if (!hasAll.every(Boolean)) {
      throw new ForbiddenException('Insufficient permissions');
    }
    return true;
  }
}

// Usage
@Controller('posts')
export class PostsController {
  @Post()
  @UseGuards(AuthGuard, PermissionsGuard)
  @RequirePermissions(PERMISSIONS.POSTS_CREATE)
  async create(@Body() dto: CreatePostDto) {
    return this.postsService.create(dto);
  }

  @Delete(':id')
  @UseGuards(AuthGuard, PermissionsGuard)
  @RequirePermissions(PERMISSIONS.POSTS_DELETE_ANY)
  async delete(@Param('id') id: string) {
    return this.postsService.delete(id);
  }
}
```

**Expected behaviour:** The guard checks the user's permissions against the required permissions for the route. Permissions are cached in Redis for 5 minutes. Cache invalidation happens on role/permission changes.

**Why this works:** Permissions are granular (`posts:create`, `posts:delete:any`) and assigned to roles. The guard enforces them consistently. Caching reduces database load.

### Real-World Cases

- **Content management:** `posts:create`, `posts:edit:own`, `posts:publish`, `posts:delete:any`.
- **E-commerce:** `products:create`, `orders:read:own`, `orders:refund`.
- **SaaS:** `billing:manage`, `users:invite`, `settings:read`.
- **Healthcare:** `patients:read`, `prescriptions:write`, `audit:read`.

---

## Core Concept 3: RBAC (Role-Based Access Control)

### Definitions

**Core Definition:** RBAC is an access control model in which permissions are assigned to roles, roles are assigned to users, and authorization decisions are based on whether the user's roles include the required permission.

**Technical Definition:** RBAC (NIST INCITS 359) defines three levels: **RBAC0** (base) — users, roles, permissions, and sessions; **RBAC1** (hierarchical) — roles can inherit permissions from other roles; **RBAC2** (constrained) — separation of duties, cardinality constraints, and prerequisite roles; **RBAC3** — combines RBAC1 and RBAC2. In practice, most applications implement RBAC0 with some RBAC1 (role hierarchy). RBAC is the most widely used authorization model because it is easy to understand, easy to implement, and aligns with organisational job functions.

**Beginner-Friendly Explanation:** RBAC is like a company's badge system. Each employee gets a badge (role) that determines which doors they can open (permissions). Managers can open more doors than employees. Instead of programming each door for each person, you program the badge types. When someone changes jobs, you just change their badge — the doors update automatically.

### Purposes

- To simplify access management by centralising permission assignments in roles.
- To align access control with organisational job functions.
- To support role hierarchies (Admin inherits Editor permissions).
- To enable fast, stateless authorization checks (roles in tokens).
- To reduce the risk of misconfiguration.
- To support audit and compliance through role assignments.

### Syntax Rules and Structure

#### RBAC Data Model

```
User ──── UserRole ──── Role ──── RolePermission ──── Permission
  │                       │
  │                       │
  └─── Session ───────────┘
```

| Entity | Purpose |
|--------|---------|
| **User** | Principal (person, service). |
| **Role** | Classification (Admin, Editor). |
| **Permission** | Granular action (`posts:create`). |
| **UserRole** | Many-to-many: users ↔ roles. |
| **RolePermission** | Many-to-many: roles ↔ permissions. |
| **Session** | Active user-role assignments during a session. |

#### Role Hierarchy

```typescript
export const ROLE_HIERARCHY: Record<RoleName, RoleName[]> = {
  admin: ['editor', 'author', 'guest'],
  editor: ['author', 'guest'],
  author: ['guest'],
  guest: [],
};

function getAllRoles(userRoles: RoleName[]): RoleName[] {
  const all = new Set<RoleName>();
  const queue = [...userRoles];
  while (queue.length > 0) {
    const role = queue.shift()!;
    if (all.has(role)) continue;
    all.add(role);
    queue.push(...(ROLE_HIERARCHY[role] ?? []));
  }
  return [...all];
}
```

#### RBAC Guard (NestJS)

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<RoleName[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles || requiredRoles.length === 0) return true;

    const req = context.switchToHttp().getRequest();
    const userRoles: RoleName[] = req.user?.roles ?? [];

    // Expand hierarchy
    const allRoles = getAllRoles(userRoles);
    const hasRole = requiredRoles.some((role) => allRoles.includes(role));

    if (!hasRole) {
      throw new ForbiddenException('Insufficient role');
    }
    return true;
  }
}

// Usage
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
@Roles(ROLES.ADMIN)
export class AdminController {
  @Get('users')
  async listUsers() {
    return this.usersService.findAll();
  }
}
```

#### Syntax Rules

- **Define roles based on job functions** — not individual permissions.
- **Assign permissions to roles** — not directly to users.
- **Support role hierarchy** — Admin inherits Editor permissions.
- **Include roles in auth tokens** — for fast, stateless checks.
- **Revoke tokens on role change** — or use short TTLs to limit staleness.
- **Audit role assignments** — log who assigned what role to whom and when.
- **Enforce separation of duties** — RBAC2 constraints (e.g., a user cannot be both approver and requester).
- **Use deny-by-default** — if no role grants access, deny.

#### Constraints and Limitations

- **Role explosion:** Too many roles become unmanageable.
- **Coarse-grained:** Roles cannot express resource-specific ownership.
- **No contextual evaluation:** Roles cannot express "only during business hours."
- **Stale roles in tokens:** Role changes take effect only after token refresh.
- **Separation of duties is hard:** RBAC2 constraints are rarely implemented.
- **Not suitable for relationship-based access:** Use ReBAC for social/graph scenarios.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full RBAC Implementation (NestJS + Prisma)

```typescript
// rbac.service.ts
@Injectable()
export class RbacService {
  constructor(private readonly prisma: PrismaService) {}

  async getUserRoles(userId: string): Promise<string[]> {
    const userRoles = await this.prisma.userRole.findMany({
      where: { userId },
      include: { role: true },
    });
    return userRoles.map((ur) => ur.role.name);
  }

  async getUserPermissions(userId: string): Promise<string[]> {
    const userRoles = await this.prisma.userRole.findMany({
      where: { userId },
      include: {
        role: {
          include: {
            permissions: { include: { permission: true } },
          },
        },
      },
    });

    return Array.from(
      new Set(
        userRoles.flatMap((ur) =>
          ur.role.permissions.map((rp) => rp.permission.name),
        ),
      ),
    );
  }

  async hasRole(userId: string, roleName: string): Promise<boolean> {
    const roles = await this.getUserRoles(userId);
    return roles.includes(roleName);
  }

  async hasPermission(userId: string, permission: string): Promise<boolean> {
    const permissions = await this.getUserPermissions(userId);
    return permissions.includes(permission);
  }

  async assignRole(userId: string, roleName: string): Promise<void> {
    const role = await this.prisma.role.findUniqueOrThrow({ where: { name: roleName } });
    await this.prisma.userRole.upsert({
      where: { userId_roleId: { userId, roleId: role.id } },
      update: {},
      create: { userId, roleId: role.id },
    });
  }

  async revokeRole(userId: string, roleName: string): Promise<void> {
    const role = await this.prisma.role.findUniqueOrThrow({ where: { name: roleName } });
    await this.prisma.userRole.deleteMany({ where: { userId, roleId: role.id } });
  }
}
```

**Expected behaviour:** `getUserRoles()` retrieves roles; `getUserPermissions()` retrieves all permissions across roles; `hasRole()` and `hasPermission()` perform checks; `assignRole()` and `revokeRole()` manage assignments.

**Why this works:** The RBAC model separates users, roles, and permissions. The service provides a clean API for authorization checks. Role hierarchy can be added via `ROLE_HIERARCHY`.

### Real-World Cases

- **Enterprise applications:** Employee, Manager, Director, Executive roles.
- **SaaS platforms:** Owner, Admin, Member, Viewer roles per tenant.
- **Content management:** Admin, Editor, Author, Contributor roles.
- **E-commerce:** Customer, Vendor, Support Agent, Admin roles.

---

## Core Concept 4: Resource-Based Authorization (Ownership)

### Definitions

**Core Definition:** Resource-based authorization evaluates whether a user can perform an action on a specific resource based on their relationship to that resource — typically ownership (e.g., only the author can edit their post).

**Technical Definition:** Resource-based authorization extends RBAC by evaluating the resource instance, not just the user's roles. The check involves: (1) loading the resource; (2) comparing the user's ID (or team, organisation) to the resource's owner (or `authorId`, `tenantId`); (3) applying any additional rules (e.g., only during draft status). Permission names typically use the `:own` vs. `:any` scope (`posts:edit:own`, `posts:edit:any`) to distinguish ownership-based from unrestricted access. Resource-based authorization is enforced in the application service, not just at the route level, because the resource must be loaded first.

**Beginner-Friendly Explanation:** Resource-based authorization is like a shared document. Anyone with the link can read it, but only the author can edit it. The check isn't just "do you have the edit permission?" — it's "do you have the edit permission **and** did you create this specific document?" The resource itself is part of the decision.

### Purposes

- To enforce ownership rules (only the author can edit/delete their posts).
- To support team or organisation-based ownership (only team members can access team resources).
- To prevent IDOR (Insecure Direct Object Reference) vulnerabilities.
- To complement RBAC with resource-instance checks.
- To support collaborative applications with per-resource sharing.
- To enable dynamic access decisions based on resource state (e.g., only draft posts can be edited).

### Syntax Rules and Structure

#### Permission Scopes

| Permission | Meaning |
|------------|---------|
| `posts:edit:any` | Can edit any post (admin, editor). |
| `posts:edit:own` | Can edit only posts they authored. |
| `posts:delete:any` | Can delete any post. |
| `posts:delete:own` | Can delete only their own posts. |

#### Resource-Based Authorization Check

```typescript
@Injectable()
export class PostsService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly rbac: RbacService,
  ) {}

  async editPost(postId: string, userId: string, dto: UpdatePostDto): Promise<Post> {
    // 1. Load the resource
    const post = await this.prisma.post.findUnique({ where: { id: postId } });
    if (!post) throw new NotFoundException('Post not found');

    // 2. Check resource-based authorization
    const canEditAny = await this.rbac.hasPermission(userId, 'posts:edit:any');
    const canEditOwn = await this.rbac.hasPermission(userId, 'posts:edit:own');

    const isOwner = post.authorId === userId;

    if (!canEditAny && !(canEditOwn && isOwner)) {
      throw new ForbiddenException('You cannot edit this post');
    }

    // 3. Apply business rules
    if (post.status === 'published' && !canEditAny) {
      throw new ForbiddenException('Published posts can only be edited by editors');
    }

    // 4. Persist
    return this.prisma.post.update({
      where: { id: postId },
      data: dto,
    });
  }
}
```

#### Syntax Rules

- **Load the resource before checking** — authorization is resource-specific.
- **Use `:own` and `:any` scopes** — distinguish ownership from unrestricted.
- **Check ownership in the service layer** — not just in middleware.
- **Prevent IDOR** — never trust resource IDs from the client without authorization.
- **Consider team/organisation ownership** — `post.teamId === user.teamId`.
- **Consider resource state** — `post.status === 'draft'` before allowing edit.
- **Return 403, not 404** — do not leak resource existence (or return 404 to hide it).
- **Log authorization failures** — for security monitoring.

#### Constraints and Limitations

- **Requires a resource load** — adds a database query to every check.
- **Ownership alone is insufficient** — combine with RBAC for comprehensive coverage.
- **Resource state matters** — published posts may be locked, regardless of ownership.
- **Team/organisation ownership is complex** — requires a membership model.
- **Delegated access is hard** — "share with a colleague" requires a sharing model (ReBAC).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Ownership-Based Authorization (NestJS + Prisma)

```typescript
// posts.controller.ts
@Controller('posts')
export class PostsController {
  constructor(private readonly postsService: PostsService) {}

  @Patch(':id')
  @UseGuards(AuthGuard)
  async update(
    @Param('id') id: string,
    @Body() dto: UpdatePostDto,
    @CurrentUser() user: AuthUser,
  ): Promise<PostDto> {
    return this.postsService.update(id, user, dto);
  }

  @Delete(':id')
  @UseGuards(AuthGuard)
  @HttpCode(204)
  async remove(
    @Param('id') id: string,
    @CurrentUser() user: AuthUser,
  ): Promise<void> {
    await this.postsService.remove(id, user);
  }
}
```

```typescript
// posts.service.ts
@Injectable()
export class PostsService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly rbac: RbacService,
  ) {}

  async update(id: string, user: AuthUser, dto: UpdatePostDto): Promise<PostDto> {
    const post = await this.prisma.post.findUnique({ where: { id } });
    if (!post) throw new NotFoundException('Post not found');

    await this.assertCanEdit(post, user);

    const updated = await this.prisma.post.update({ where: { id }, data: dto });
    return PostMapper.toDto(updated);
  }

  async remove(id: string, user: AuthUser): Promise<void> {
    const post = await this.prisma.post.findUnique({ where: { id } });
    if (!post) throw new NotFoundException('Post not found');

    await this.assertCanDelete(post, user);

    await this.prisma.post.delete({ where: { id } });
  }

  private async assertCanEdit(post: Post, user: AuthUser): Promise<void> {
    const canEditAny = await this.rbac.hasPermission(user.id, 'posts:edit:any');
    if (canEditAny) return;

    const canEditOwn = await this.rbac.hasPermission(user.id, 'posts:edit:own');
    const isOwner = post.authorId === user.id;

    if (!canEditOwn || !isOwner) {
      throw new ForbiddenException('You cannot edit this post');
    }

    if (post.status === 'published') {
      throw new ForbiddenException('Published posts cannot be edited by authors');
    }
  }

  private async assertCanDelete(post: Post, user: AuthUser): Promise<void> {
    const canDeleteAny = await this.rbac.hasPermission(user.id, 'posts:delete:any');
    if (canDeleteAny) return;

    const canDeleteOwn = await this.rbac.hasPermission(user.id, 'posts:delete:own');
    const isOwner = post.authorId === user.id;

    if (!canDeleteOwn || !isOwner) {
      throw new ForbiddenException('You cannot delete this post');
    }
  }
}
```

**Expected behaviour:**
- An admin with `posts:edit:any` can edit any post.
- An author with `posts:edit:own` can edit only their own draft posts.
- An author cannot edit a published post (business rule).
- A user without either permission receives `403 Forbidden`.

**Why this works:** The service loads the resource, checks the user's permissions (`:any` or `:own`), compares ownership (`post.authorId === user.id`), and applies business rules (`post.status === 'published'`). This prevents IDOR and enforces ownership.

### Real-World Cases

- **Social media:** Only the author can edit/delete their posts.
- **Project management:** Only project members can edit project tasks.
- **E-commerce:** Only the order owner can view their order details.
- **Healthcare:** Only the treating physician can modify patient records.

---

## Core Concept 5: Tenant-Based Authorization (Multi-Tenancy)

### Definitions

**Core Definition:** Tenant-based authorization isolates data and operations across multiple tenants (organisations, customers) within a shared application, ensuring that users of one tenant cannot access another tenant's data.

**Technical Definition:** Multi-tenancy is an architecture where a single application instance serves multiple tenants, with data isolation enforced at the application, database, or schema level. Tenant-based authorization requires: (1) a `tenantId` on every tenant-scoped resource; (2) a `tenantId` claim in the user's identity (JWT or session); (3) enforcement at every data access point (repository, query, or database row-level security). Isolation models include: **shared database, shared schema** (tenantId column + row-level filtering); **shared database, separate schemas** (schema per tenant); **separate databases** (database per tenant). Row-level security (RLS) in PostgreSQL provides defense-in-depth by enforcing tenant isolation at the database level.

**Beginner-Friendly Explanation:** Imagine an apartment building. Each tenant has their own apartment (their data), and the building (the application) is shared. Tenant-based authorization ensures that Tenant A cannot enter Tenant B's apartment. Every door has a lock that only the right tenant's key can open. Even if Tenant A learns Tenant B's apartment number, the lock still stops them. In software, the "apartment number" is the tenant ID, and the "lock" is enforced at every data access point.

### Purposes

- To isolate data across multiple customers in a shared application.
- To prevent cross-tenant data leakage (a critical security risk).
- To enable efficient resource utilisation (shared infrastructure).
- To support compliance with data residency and privacy regulations.
- To enable per-tenant customisation (branding, features, limits).
- To provide a foundation for tenant-level billing and analytics.

### Syntax Rules and Structure

#### Tenant Model

```prisma
model Tenant {
  id        String   @id @default(uuid())
  name      String
  slug      String   @unique
  plan      String   @default("free")
  createdAt DateTime @default(now())
  users     User[]
  posts     Post[]
}

model User {
  id       String @id @default(uuid())
  email    String @unique
  tenantId String
  tenant   Tenant @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  // ...
}

model Post {
  id       String @id @default(uuid())
  title    String
  tenantId String
  tenant   Tenant @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  // ...

  @@index([tenantId])
}
```

#### Tenant Context Middleware

```typescript
@Injectable()
export class TenantMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const user = req.user as AuthUser | undefined;
    if (user) {
      // Tenant ID comes from the authenticated user, never from the request
      req.tenantId = user.tenantId;
    }
    next();
  }
}
```

#### Tenant-Scoped Repository

```typescript
@Injectable()
export class TenantScopedPostRepository {
  constructor(private readonly prisma: PrismaService) {}

  async findById(id: string, tenantId: string): Promise<Post | null> {
    return this.prisma.post.findFirst({
      where: { id, tenantId }, // Always filter by tenantId
    });
  }

  async findAll(tenantId: string): Promise<Post[]> {
    return this.prisma.post.findMany({ where: { tenantId } });
  }

  async create(data: CreatePostData, tenantId: string): Promise<Post> {
    return this.prisma.post.create({
      data: { ...data, tenantId },
    });
  }

  async update(id: string, tenantId: string, data: UpdatePostData): Promise<Post> {
    const result = await this.prisma.post.updateMany({
      where: { id, tenantId },
      data,
    });
    if (result.count === 0) throw new NotFoundException();
    return this.prisma.post.findUniqueOrThrow({ where: { id } });
  }
}
```

#### PostgreSQL Row-Level Security (Defense in Depth)

```sql
-- Enable RLS on the posts table
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Create a policy that filters by the current tenant
CREATE POLICY tenant_isolation ON posts
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- Application sets the tenant before queries
SET app.current_tenant = 'tenant-uuid-here';
```

#### Syntax Rules

- **Always include `tenantId` on tenant-scoped resources.**
- **Derive `tenantId` from the authenticated user** — never from the request.
- **Filter every query by `tenantId`** — use a tenant-scoped repository.
- **Use `findFirst` with `tenantId`** instead of `findUnique` by ID alone.
- **Add composite indexes** — `@@index([tenantId, id])`.
- **Enable PostgreSQL RLS** — defense in depth.
- **Validate tenant membership** — ensure the user belongs to the tenant.
- **Audit cross-tenant access attempts** — log and alert.
- **Consider separate databases** for high-security or compliance-heavy tenants.

#### Constraints and Limitations

- **A single missed filter can leak data** — use repository abstractions and RLS.
- **Tenant isolation adds query overhead** — composite indexes are essential.
- **Cross-tenant operations are complex** — admin tools must explicitly bypass isolation.
- **Tenant migration is hard** — moving a tenant to a separate database requires planning.
- **Shared schema limits customisation** — separate schemas/databases offer more flexibility.
- **RLS requires careful setup** — incorrect policies can block legitimate access.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Tenant-Scoped Multi-Tenancy (NestJS + Prisma + RLS)

```typescript
// tenant.guard.ts — extract tenant from JWT and set RLS context
@Injectable()
export class TenantGuard implements CanActivate {
  constructor(private readonly prisma: PrismaService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const req = context.switchToHttp().getRequest();
    const user = req.user as AuthUser;

    if (!user?.tenantId) {
      throw new ForbiddenException('No tenant context');
    }

    // Set the tenant for PostgreSQL RLS
    await this.prisma.$executeRaw`SET app.current_tenant = ${user.tenantId}`;
    req.tenantId = user.tenantId;

    return true;
  }
}
```

```typescript
// posts.service.ts — tenant-scoped queries
@Injectable()
export class PostsService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string): Promise<Post[]> {
    return this.prisma.post.findMany({
      where: { tenantId },
      orderBy: { createdAt: 'desc' },
    });
  }

  async findById(id: string, tenantId: string): Promise<Post> {
    const post = await this.prisma.post.findFirst({
      where: { id, tenantId }, // Tenant filter prevents cross-tenant access
    });
    if (!post) throw new NotFoundException('Post not found');
    return post;
  }

  async create(data: CreatePostDto, tenantId: string, userId: string): Promise<Post> {
    return this.prisma.post.create({
      data: {
        ...data,
        tenantId,
        authorId: userId,
      },
    });
  }

  async update(id: string, data: UpdatePostDto, tenantId: string): Promise<Post> {
    const result = await this.prisma.post.updateMany({
      where: { id, tenantId },
      data,
    });
    if (result.count === 0) throw new NotFoundException('Post not found');
    return this.prisma.post.findUniqueOrThrow({ where: { id } });
  }
}
```

```typescript
// posts.controller.ts
@Controller('posts')
@UseGuards(AuthGuard, TenantGuard)
export class PostsController {
  constructor(private readonly postsService: PostsService) {}

  @Get()
  async findAll(@CurrentTenant() tenantId: string): Promise<PostDto[]> {
    return this.postsService.findAll(tenantId);
  }

  @Get(':id')
  async findById(
    @Param('id') id: string,
    @CurrentTenant() tenantId: string,
  ): Promise<PostDto> {
    return this.postsService.findById(id, tenantId);
  }

  @Post()
  async create(
    @Body() dto: CreatePostDto,
    @CurrentTenant() tenantId: string,
    @CurrentUser() user: AuthUser,
  ): Promise<PostDto> {
    return this.postsService.create(dto, tenantId, user.id);
  }
}
```

**Expected behaviour:**
- All queries are filtered by `tenantId` from the authenticated user.
- `findById()` uses `findFirst` with `tenantId` — a user from Tenant A cannot access Tenant B's post.
- PostgreSQL RLS provides defense in depth — even a raw query without the filter is blocked.
- Cross-tenant access returns `404 Not Found` (not `403`, to hide resource existence).

**Why this works:** The tenant ID is derived from the authenticated user (never from the request). Every query includes `tenantId`. PostgreSQL RLS enforces isolation at the database level. The `TenantGuard` sets the RLS context before any query.

### Real-World Cases

- **SaaS platforms:** Each customer organisation is a tenant; users belong to one tenant.
- **Healthcare:** Each hospital or clinic is a tenant; patient data is isolated.
- **Financial services:** Each financial institution is a tenant; accounts are isolated.
- **Education:** Each school or university is a tenant; student data is isolated.
- **Enterprise:** Each subsidiary or department is a tenant; data is isolated.

---

## Core Concept 6: ABAC (Attribute-Based Access Control)

### Definitions

**Core Definition:** ABAC is an access control model that grants or denies access based on attributes of the subject, resource, action, and environment, evaluated against policies.

**Technical Definition:** ABAC (NIST SP 800-162) evaluates policies against four attribute categories: **subject** (user attributes — department, clearance, role), **resource** (resource attributes — classification, owner, type), **action** (operation — read, write, delete), and **environment** (context — time, IP address, location, device). Policies are boolean expressions over these attributes (e.g., `subject.department == resource.department AND environment.time BETWEEN 09:00 AND 17:00`). ABAC is more expressive than RBAC but more complex to manage. Policy engines like OPA (Open Policy Agent), Casbin, Oso, and Cerbos provide ABAC evaluation. ABAC enables fine-grained, context-aware access control.

**Beginner-Friendly Explanation:** RBAC says "Managers can approve expenses." ABAC says "Managers can approve expenses **up to $5,000**, **during business hours**, **from a company IP address**, **for employees in their department**." ABAC considers the full context — who you are, what you're accessing, what you're doing, and where/when you're doing it. It's much more powerful, but also more complex.

### Purposes

- To grant access based on contextual parameters (time, IP, department).
- To enforce fine-grained, dynamic policies that RBAC cannot express.
- To support regulatory compliance (data residency, time-based access).
- To enable risk-based authentication (require MFA for high-risk contexts).
- To support dynamic, policy-driven access control.
- To decouple policy from application code (externalised policies).

### Syntax Rules and Structure

#### ABAC Attribute Categories

| Category | Attributes | Examples |
|----------|------------|----------|
| **Subject** | User attributes | `department`, `clearance`, `role`, `location` |
| **Resource** | Resource attributes | `classification`, `owner`, `type`, `status` |
| **Action** | Operation attributes | `read`, `write`, `delete`, `approve` |
| **Environment** | Context attributes | `time`, `ip`, `device`, `mfaVerified` |

#### Policy Expression

```
permit(
  subject: User,
  action: "approve",
  resource: Expense
) when {
  subject.role == "manager" &&
  resource.department == subject.department &&
  resource.amount <= 5000 &&
  context.time.hour >= 9 &&
  context.time.hour < 17 &&
  context.ip.isCorporateNetwork
};
```

#### ABAC Policy Engine (OPA/Cedar-style)

```typescript
// policy.ts
export interface AuthContext {
  subject: {
    id: string;
    role: string;
    department: string;
    clearance: 'public' | 'internal' | 'confidential';
  };
  resource: {
    type: string;
    department: string;
    classification: 'public' | 'internal' | 'confidential';
    ownerId: string;
  };
  action: string;
  environment: {
    time: Date;
    ip: string;
    mfaVerified: boolean;
    deviceTrusted: boolean;
  };
}

export function evaluatePolicy(ctx: AuthContext): boolean {
  // Policy: Managers can approve expenses in their department up to $5,000
  if (ctx.action === 'approve' && ctx.resource.type === 'expense') {
    const hour = ctx.environment.time.getHours();
    return (
      ctx.subject.role === 'manager' &&
      ctx.resource.department === ctx.subject.department &&
      hour >= 9 &&
      hour < 17 &&
      ctx.environment.mfaVerified &&
      ctx.environment.deviceTrusted
    );
  }

  // Policy: Users can read resources up to their clearance level
  if (ctx.action === 'read') {
    const levels = { public: 0, internal: 1, confidential: 2 };
    return levels[ctx.subject.clearance] >= levels[ctx.resource.classification];
  }

  return false; // Deny by default
}
```

#### Syntax Rules

- **Define attributes consistently** — subject, resource, action, environment.
- **Use policy engines for complex rules** — OPA, Casbin, Cerbos, Oso.
- **Externalise policies** — separate from application code.
- **Deny by default** — if no policy permits, deny.
- **Validate attributes** — never trust client-provided attributes.
- **Cache policy decisions** — with short TTLs for performance.
- **Log policy decisions** — for audit and debugging.
- **Test policies thoroughly** — policy bugs are security bugs.

#### Constraints and Limitations

- **Complexity:** ABAC policies can become hard to understand and maintain.
- **Attribute availability:** Policies require accurate, timely attributes.
- **Performance:** Evaluating complex policies on every request adds latency.
- **Policy conflicts:** Multiple policies may conflict; precedence rules are needed.
- **Testing difficulty:** Combinatorial explosion of attribute values makes testing hard.
- **Tooling:** Policy engines add operational complexity.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: ABAC with Casbin (Node.js)

```typescript
// casbin.model.conf
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub_rule, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = eval(p.sub_rule) && r.obj == p.obj && r.act == p.act
```

```typescript
// authz.service.ts
import { newEnforcer } from 'casbin';

@Injectable()
export class AuthzService {
  private enforcer: Enforcer;

  async onModuleInit() {
    this.enforcer = await newEnforcer('casbin.model.conf', 'casbin.policy.csv');
  }

  async canApproveExpense(
    user: { role: string; department: string },
    expense: { department: string; amount: number },
    context: { time: Date; ip: string; mfaVerified: boolean },
  ): Promise<boolean> {
    const hour = context.time.getHours();
    const isBusinessHours = hour >= 9 && hour < 17;

    return (
      user.role === 'manager' &&
      user.department === expense.department &&
      expense.amount <= 5000 &&
      isBusinessHours &&
      context.mfaVerified
    );
  }
}

// Usage in a controller
@Post('expenses/:id/approve')
@UseGuards(AuthGuard)
async approveExpense(
  @Param('id') id: string,
  @CurrentUser() user: AuthUser,
  @Req() req: Request,
) {
  const expense = await this.expensesService.findById(id);
  const allowed = await this.authzService.canApproveExpense(
    { role: user.role, department: user.department },
    { department: expense.department, amount: expense.amount },
    { time: new Date(), ip: req.ip, mfaVerified: user.mfaVerified },
  );

  if (!allowed) throw new ForbiddenException('Cannot approve this expense');
  return this.expensesService.approve(id);
}
```

**Expected behaviour:**
- A manager in the same department can approve expenses up to $5,000 during business hours with MFA.
- Outside business hours, approval is denied.
- Without MFA, approval is denied.
- For a different department, approval is denied.

**Why this works:** ABAC evaluates attributes from the subject (role, department), resource (department, amount), action (approve), and environment (time, MFA). Policies are explicit and testable. Deny by default.

### Real-World Cases

- **Financial services:** Approvals based on amount, department, time, and MFA.
- **Healthcare:** Access to patient records based on department, shift, and location.
- **Government:** Access based on clearance level, need-to-know, and time.
- **Cloud providers:** Access based on IP, device trust, and MFA status.
- **E-commerce:** Discounts based on customer tier, location, and time.

---

## Core Concept 7: ReBAC (Relationship-Based Access Control)

### Definitions

**Core Definition:** ReBAC authorizes access based on the relationships between users and objects in a graph, enabling fine-grained, relationship-driven access control (e.g., "only friends of the document owner can view").

**Technical Definition:** ReBAC (Google Zanzibar model) models authorization as a graph of relationships between subjects (users, groups) and objects (documents, folders, projects). Relationships are tuples: `(object, relation, subject)` — e.g., `(document:1, owner, user:alice)`, `(document:1, viewer, group:eng)`. Authorization checks traverse the graph to determine if a relationship path exists. ReBAC supports **relationship inheritance** (a folder's viewer is a viewer of all files in it) and **nested groups** (a member of a group is a member of all parent groups). Implementations include Google Zanzibar, OpenFGA, SpiceDB, Authzed, and Ory Keto.

**Beginner-Friendly Explanation:** ReBAC is like a social network for permissions. Instead of saying "Alice is a Manager, so she can edit posts," ReBAC says "Alice is a member of the Engineering group, which is a viewer of the Q1 Report, which contains this spreadsheet — so Alice can view this spreadsheet." It's about relationships, not just roles. If Alice leaves the Engineering group, she loses access automatically because the relationship is gone.

### Purposes

- To authorize access based on deep relationship graphs (social, organisational, hierarchical).
- To support collaborative applications with per-resource sharing.
- To enable relationship inheritance (folder → file).
- To support nested groups and teams.
- To decouple authorization from static roles — access flows from relationships.
- To enable "share with" functionality (Google Docs-style).

### Syntax Rules and Structure

#### Relationship Tuples

```
(object, relation, subject)
```

| Object | Relation | Subject | Meaning |
|--------|----------|---------|---------|
| `document:1` | `owner` | `user:alice` | Alice owns document 1. |
| `document:1` | `viewer` | `group:engineering` | Engineering can view document 1. |
| `group:engineering` | `member` | `user:bob` | Bob is a member of engineering. |
| `folder:1` | `parent` | `document:1` | Folder 1 contains document 1. |
| `folder:1` | `viewer` | `group:engineering` | Engineering can view folder 1. |

#### Authorization Check

```
Can user:bob view document:1?

Path 1: document:1 → viewer → group:engineering
Path 2: group:engineering → member → user:bob

→ ALLOW (relationship path exists)
```

#### ReBAC with OpenFGA (Node.js)

```typescript
import { OpenFgaClient } from '@openfga/sdk';

const fga = new OpenFgaClient({
  apiUrl: process.env.FGA_API_URL,
  storeId: process.env.FGA_STORE_ID,
  authorizationModelId: process.env.FGA_MODEL_ID,
});

// Write relationship tuples
await fga.write({
  writes: [
    { user: 'user:alice', relation: 'owner', object: 'document:1' },
    { user: 'group:engineering#member', relation: 'viewer', object: 'document:1' },
    { user: 'user:bob', relation: 'member', object: 'group:engineering' },
    { user: 'document:1', relation: 'parent', object: 'folder:1' },
    { user: 'group:engineering#member', relation: 'viewer', object: 'folder:1' },
  ],
});

// Check authorization
const { allowed } = await fga.check({
  user: 'user:bob',
  relation: 'viewer',
  object: 'document:1',
});

console.log(allowed); // true (via group membership)
```

#### OpenFGA Authorization Model (DSL)

```dsl
model
  schema 1.1

type user

type group
  relations
    define member: [user]

type folder
  relations
    define viewer: [user, group#member]
    define parent: [folder]

type document
  relations
    define owner: [user]
    define viewer: [user, group#member]
    define parent: [folder]
    define can_view: viewer or owner or viewer from parent
    define can_edit: owner or owner from parent
```

#### Syntax Rules

- **Model relationships as tuples** — `(object, relation, subject)`.
- **Use a dedicated ReBAC engine** — OpenFGA, SpiceDB, Authzed, Ory Keto.
- **Define authorization models** — types, relations, and computed permissions.
- **Support nested groups** — `group:engineering#member`.
- **Support relationship inheritance** — `viewer from parent`.
- **Write tuples on resource creation and sharing** — keep the graph up to date.
- **Delete tuples on resource deletion and unsharing.**
- **Cache check results** — with short TTLs for performance.
- **Log relationship changes** — for audit and debugging.

#### Constraints and Limitations

- **Graph traversal can be expensive** — deep or wide graphs slow down checks.
- **Consistency challenges** — relationship changes must propagate; eventual consistency is common.
- **Model design is critical** — poorly designed models lead to complex checks.
- **Operational complexity** — running a ReBAC engine adds infrastructure.
- **Learning curve** — ReBAC concepts (tuples, models, checks) require training.
- **Not suitable for simple applications** — RBAC is simpler for basic needs.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: ReBAC with OpenFGA (NestJS)

```typescript
// fga.service.ts
@Injectable()
export class FgaService {
  private readonly client: OpenFgaClient;

  constructor() {
    this.client = new OpenFgaClient({
      apiUrl: process.env.FGA_API_URL,
      storeId: process.env.FGA_STORE_ID,
      authorizationModelId: process.env.FGA_MODEL_ID,
    });
  }

  async grantAccess(
    user: string,
    relation: string,
    object: string,
  ): Promise<void> {
    await this.client.write({
      writes: [{ user, relation, object }],
    });
  }

  async revokeAccess(
    user: string,
    relation: string,
    object: string,
  ): Promise<void> {
    await this.client.write({
      deletes: [{ user, relation, object }],
    });
  }

  async check(
    user: string,
    relation: string,
    object: string,
  ): Promise<boolean> {
    const { allowed } = await this.client.check({ user, relation, object });
    return allowed;
  }

  async listObjects(
    user: string,
    relation: string,
    type: string,
  ): Promise<string[]> {
    const { objects } = await this.client.listObjects({
      user,
      relation,
      type,
    });
    return objects;
  }
}
```

```typescript
// documents.service.ts
@Injectable()
export class DocumentsService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly fga: FgaService,
  ) {}

  async create(data: CreateDocumentDto, userId: string): Promise<Document> {
    const doc = await this.prisma.document.create({
      data: { ...data, ownerId: userId },
    });

    // Write relationship tuple
    await this.fga.grantAccess(`user:${userId}`, 'owner', `document:${doc.id}`);

    return doc;
  }

  async share(
    documentId: string,
    targetUserOrGroup: string,
    relation: 'viewer' | 'editor',
    actorId: string,
  ): Promise<void> {
    // Check that the actor can share
    const canShare = await this.fga.check(
      `user:${actorId}`,
      'owner',
      `document:${documentId}`,
    );
    if (!canShare) throw new ForbiddenException('Only the owner can share');

    await this.fga.grantAccess(targetUserOrGroup, relation, `document:${documentId}`);
  }

  async findById(documentId: string, userId: string): Promise<Document> {
    const canView = await this.fga.check(
      `user:${userId}`,
      'can_view',
      `document:${documentId}`,
    );
    if (!canView) throw new ForbiddenException('Cannot view this document');

    return this.prisma.document.findUniqueOrThrow({ where: { id: documentId } });
  }

  async listAccessible(userId: string): Promise<Document[]> {
    const objectIds = await this.fga.listObjects(
      `user:${userId}`,
      'can_view',
      'document',
    );
    const ids = objectIds.map((o) => o.replace('document:', ''));

    return this.prisma.document.findMany({ where: { id: { in: ids } } });
  }
}
```

**Expected behaviour:**
- Creating a document writes an `owner` tuple.
- Sharing grants `viewer` or `editor` to a user or group.
- `findById()` checks the `can_view` relation (traverses the graph: direct viewer, owner, or parent folder viewer).
- `listAccessible()` returns all documents the user can view.

**Why this works:** Relationships are stored as tuples. Authorization checks traverse the graph (document → group → user, or document → parent folder → group → user). Sharing is a matter of writing a tuple. Revocation is a matter of deleting a tuple.

### Real-World Cases

- **Google Docs:** Share documents with users or groups; permissions inherit through folders.
- **GitHub:** Repository access via teams, organisations, and individual collaborators.
- **Slack:** Channel access via workspace membership and channel membership.
- **Notion:** Page sharing with users, teams, and workspace members.
- **Enterprise:** Project access via organisational hierarchy and team membership.

---

## References

- NIST INCITS 359 — Role-Based Access Control (RBAC) — https://csrc.nist.gov/projects/role-based-access-control
- NIST SP 800-162 — Guide to Attribute-Based Access Control (ABAC) — https://csrc.nist.gov/publications/detail/sp/800-162/final
- NIST SP 800-205 — Attribute Considerations for Access Control Systems — https://csrc.nist.gov/publications/detail/sp/800-205/final
- Google Zanzibar — Relationship-Based Access Control — https://research.google/pubs/pub48190/
- OpenFGA Documentation — https://openfga.dev/docs
- SpiceDB Documentation — https://authzed.com/docs
- Ory Keto Documentation — https://www.ory.sh/keto/docs/
- Casbin Documentation — https://casbin.org/docs/overview
- Open Policy Agent (OPA) — https://www.openpolicyagent.org/docs/latest/
- Cerbos Documentation — https://docs.cerbos.dev/
- Oso Documentation — https://www.osohq.com/docs
- Auth0 — RBAC Documentation — https://auth0.com/docs/manage-users/access-control/rbac
- Auth0 — ABAC Documentation — https://auth0.com/docs/manage-users/access-control/abac
- OWASP Cheat Sheet Series — Access Control — https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html
- OWASP — Insecure Direct Object Reference (IDOR) — https://owasp.org/www-project-top-ten/2017/A4_2017-Insecure_Direct_Object_References
- OWASP — Multi-Tenant Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Multitenant_Security_Cheat_Sheet.html
- AWS — Multi-Tenant SaaS Architecture — https://docs.aws.amazon.com/whitepapers/latest/saas-architecture-fundamentals/saas-architecture-fundamentals.html
- PostgreSQL Documentation — Row-Level Security — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- NestJS Documentation — Guards — https://docs.nestjs.com/guards
- NestJS Documentation — Authorization — https://docs.nestjs.com/security/authorization
- @openfga/sdk — npm package — https://www.npmjs.com/package/@openfga/sdk
- node-casbin — npm package — https://www.npmjs.com/package/casbin
- @cerbos/sdk — npm package — https://www.npmjs.com/package/@cerbos/sdk