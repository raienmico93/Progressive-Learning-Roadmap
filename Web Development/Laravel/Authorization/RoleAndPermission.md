# Role and Permission Architecture (RBAC): A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Role-Based Access Control (RBAC) is an authorization model where permissions are assigned to roles, and roles are assigned to users, creating a many-to-many relationship structure that simplifies permission management at scale.

**Technical Definition:** RBAC in Laravel is typically implemented through a five-table schema: `permissions`, `roles`, `model_has_permissions` (direct user-permission links), `model_has_roles` (user-role links), and `role_has_permissions` (role-permission links). This polymorphic structure allows any Eloquent model to have roles and permissions. Permission checks are resolved through cached lookups to avoid repeated database queries. Enterprise implementations often extend this with role hierarchies, wildcard permissions, and team-scoped contexts.

**Beginner-Friendly Explanation:** Instead of giving each user individual permissions (which becomes unmanageable with hundreds of users), you create roles like "Admin," "Editor," and "Viewer." Each role has a set of permissions. You assign users to roles. When you need to change what a group of users can do, you change the role, not each user. The `spatie/laravel-permission` package is the de facto standard for this in Laravel.

### Key Characteristics

- **Many-to-Many Structure:** Users ↔ Roles ↔ Permissions with polymorphic pivot tables.
- **Cache-Backed Performance:** Permissions are cached (24-hour default) to eliminate N+1 queries.
- **Multi-Guard Support:** Roles and permissions are scoped by guard (web, api, admin).
- **Direct Permissions:** Users can receive permissions directly, bypassing roles for exceptions.
- **Role Hierarchies (Extended):** Advanced packages support inheritance (admin inherits editor permissions).
- **Context/Team Scoping:** Permissions can be scoped to teams, projects, or tenants.

### Prerequisites

- A Laravel application with authentication configured.
- A `User` model extending `Illuminate\Foundation\Auth\User`.
- Composer for installing packages.
- For Spatie: `spatie/laravel-permission` package installed and migrated.
- For caching: A cache driver configured (Redis recommended for production).

### Related Programming Areas

- **Authentication:** RBAC operates on authenticated users.
- **Middleware:** `role:` and `permission:` middleware enforce access at the route level.
- **Blade Directives:** `@role`, `@hasrole`, `@can` conditionally render UI.
- **Caching:** Permission data is cached to avoid database bottlenecks.
- **Multi-Tenancy:** Team-scoped roles isolate permissions per tenant.

---

## 1. Database Schema Design: Many-to-Many Structures

### Definitions

**Core Definition:** The RBAC database schema links users to roles and roles to permissions through pivot tables, enabling flexible assignment and revocation of access rights.

**Technical Definition:** Spatie Laravel Permission creates five tables. The `permissions` and `roles` tables store definitions with `name` and `guard_name` columns. The `model_has_permissions` and `model_has_roles` tables are polymorphic pivots linking any model (via `model_type` and `model_id`) to permissions or roles. The `role_has_permissions` table links roles to permissions. The `model_has_permissions` table allows direct user-permission assignments bypassing roles.

**Beginner-Friendly Explanation:** Think of it as three layers. First, you define what actions exist (permissions like "edit articles"). Second, you group those actions into jobs (roles like "Editor"). Third, you assign people to jobs (users to roles). The database tables record these connections, and Laravel can trace from any user to all their permissions through the chain.

### Purposes

- To store role and permission definitions centrally.
- To link users to roles through a polymorphic pivot table.
- To link roles to permissions through a dedicated pivot table.
- To support direct user-permission assignments for exceptions.
- To enable guard-scoped permissions for multi-auth systems.

### Syntax Rules and Structure

**Complete Table Schema (Spatie Laravel Permission):**

```sql
-- permissions table
CREATE TABLE permissions (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    guard_name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    UNIQUE KEY (name, guard_name)
);

-- roles table
CREATE TABLE roles (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    guard_name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    UNIQUE KEY (name, guard_name)
);

-- model_has_permissions (direct user-permission links)
CREATE TABLE model_has_permissions (
    permission_id BIGINT UNSIGNED NOT NULL,
    model_type VARCHAR(255) NOT NULL,
    model_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (permission_id, model_id, model_type),
    INDEX (model_id, model_type),
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE
);

-- model_has_roles (user-role links)
CREATE TABLE model_has_roles (
    role_id BIGINT UNSIGNED NOT NULL,
    model_type VARCHAR(255) NOT NULL,
    model_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (role_id, model_id, model_type),
    INDEX (model_id, model_type),
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);

-- role_has_permissions (role-permission links)
CREATE TABLE role_has_permissions (
    permission_id BIGINT UNSIGNED NOT NULL,
    role_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (permission_id, role_id),
    FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);
```

**Component Breakdown:**

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `permissions` | Stores permission definitions | `name`, `guard_name` |
| `roles` | Stores role definitions | `name`, `guard_name` |
| `model_has_permissions` | Direct model-permission links | `permission_id`, `model_type`, `model_id` |
| `model_has_roles` | Model-role links | `role_id`, `model_type`, `model_id` |
| `role_has_permissions` | Role-permission links | `role_id`, `permission_id` |

**Syntax Rules:**
- The `guard_name` column scopes roles and permissions to specific authentication guards .
- The polymorphic `model_type` and `model_id` columns allow any Eloquent model to have roles/permissions .
- Unique composite indexes on `(name, guard_name)` prevent duplicate definitions.
- Foreign keys with `ON DELETE CASCADE` ensure cleanup when definitions are removed.

**Constraints and Limitations:**
- **Index size:** `VARCHAR(255)` on `name` and `guard_name` can exceed MySQL index limits; reduce lengths or use `utf8mb4_bin` collation .
- **UUID support:** Requires migration modifications (change `unsignedBigInteger` to `uuid` for `model_id` and primary keys) .
- **Performance:** Without indexes on `model_id` and `model_type`, permission lookups become slow.

### Multiple Annotated Complete Code Examples

**Example 1: Publishing and Running Migrations**

```bash
# Publish the package migrations
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"

# Run migrations
php artisan migrate
```

**Expected Output:** Five tables are created: `permissions`, `roles`, `model_has_permissions`, `model_has_roles`, `role_has_permissions`.

**Why:** The `vendor:publish` command copies the migration file to `database/migrations/`. `migrate` executes it, creating the RBAC schema .

---

**Example 2: Seeding Roles and Permissions**

```php
<?php
// database/seeders/RolesAndPermissionsSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

class RolesAndPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        // Create permissions
        Permission::create(['name' => 'edit articles']);
        Permission::create(['name' => 'delete articles']);
        Permission::create(['name' => 'publish articles']);

        // Create roles and assign permissions
        $editor = Role::create(['name' => 'editor']);
        $editor->givePermissionTo(['edit articles', 'publish articles']);

        $admin = Role::create(['name' => 'admin']);
        $admin->givePermissionTo(Permission::all());
    }
}
```

**Expected Output:** Three permissions and two roles are created. The `admin` role has all permissions; `editor` has edit and publish permissions.

**Why:** `Permission::create()` and `Role::create()` insert definitions. `givePermissionTo()` inserts rows into `role_has_permissions` .

### Real-World Cases

- **CMS Systems:** `PostPolicy` with `edit articles` permission assigned to editor role.
- **E-commerce:** `manage orders`, `process refunds` permissions grouped into `support` and `admin` roles.
- **SaaS Multi-Tenant:** Team-scoped roles where a user is `admin` in one tenant and `member` in another .

### References

- Spatie Laravel Permission: Database Tables — https://mintlify.wiki/spatie/laravel-permission/configuration/database-tables 
- Spatie Laravel Permission: UUID/ULID Support — https://spatie.be/docs/laravel-permission/v7/advanced-usage/uuid 
- Spatie Laravel Permission: UUID (v5) — https://spatie.be/docs/laravel-permission/v5/advanced-usage/uuid 

---

## 2. Multi-Role Systems vs. Strict Hierarchical Inheritance

### Definitions

**Core Definition:** Multi-role systems assign users multiple independent roles, while hierarchical inheritance models allow roles to inherit permissions from other roles, creating a hierarchy where higher roles possess all permissions of lower roles.

**Technical Definition:** In a flat multi-role system, permissions are resolved by unioning the permissions of all assigned roles. In a hierarchical model, roles form a directed acyclic graph (DAG); a role's effective permissions are the union of its own permissions and all inherited roles' permissions. Hierarchy requires cycle detection to prevent infinite loops. Spatie Laravel Permission supports flat multi-role by default; packages like `hosseinhezami/laravel-permission-manager` and `whilesmart/eloquent-roles` add hierarchy.

**Beginner-Friendly Explanation:** In a flat system, you might give a user both "Editor" and "Moderator" roles. Their permissions are the sum of both. In a hierarchical system, "Admin" might automatically inherit all "Editor" permissions, so you only assign "Admin" and get everything. Hierarchies are simpler to manage but can become complex with overlapping permissions.

### Purposes

- To assign multiple independent roles to a user for combined permissions (flat).
- To model organizational hierarchies where higher roles automatically possess lower role permissions (hierarchical).
- To reduce assignment overhead by inheriting permissions through role levels.
- To support complex approval workflows with escalating privilege levels.

### Syntax Rules and Structure

**Complete General Syntax — Flat Multi-Role (Spatie):**

```php
use Spatie\Permission\Models\Role;

// User has both Editor and Moderator roles
$user->assignRole('editor', 'moderator');

// Permissions are the union of both roles
$user->hasPermissionTo('edit articles'); // true if either role has it
```

**Complete General Syntax — Role Hierarchy (Extended Package):**

```php
use Hosseinhezami\PermissionManager\Models\Role;

// Create hierarchy: super-admin > admin > editor > viewer
$superAdmin = Role::create(['name' => 'Super Admin', 'slug' => 'super-admin']);
$admin = Role::create(['name' => 'Admin', 'slug' => 'admin']);
$editor = Role::create(['name' => 'Editor', 'slug' => 'editor']);
$viewer = Role::create(['name' => 'Viewer', 'slug' => 'viewer']);

// Assign base permissions
$viewer->assignPermission('posts.view');
$editor->assignPermission('posts.edit');
$admin->assignPermission('posts.delete');

// Build hierarchy
$admin->inheritFrom('editor');       // admin gets editor's permissions
$editor->inheritFrom('viewer');      // editor gets viewer's permissions
$superAdmin->inheritFrom('admin');   // super-admin gets admin's permissions

// User with super-admin has ALL permissions via inheritance
$user->assignRole('super-admin');
$user->hasPermissionTo('posts.view');  // ✅ inherited
$user->hasPermissionTo('posts.edit');  // ✅ inherited
$user->hasPermissionTo('posts.delete'); // ✅ inherited
```

**Component Breakdown:**

| Model | Approach | Permission Resolution |
|-------|----------|----------------------|
| Flat Multi-Role | Union of assigned roles | Sum of role permissions |
| Hierarchical | Role inheritance tree | Union of role + inherited permissions |

**Syntax Rules:**
- Flat systems use `assignRole()` multiple times or pass an array.
- Hierarchical systems use `inheritFrom()` or `assignChildRole()` to build the tree.
- Cycle detection must be enforced (e.g., `$roleA->inheritFrom($roleB); $roleB->inheritFrom($roleA)` throws an exception) .
- Permission resolution order: direct user permissions > role permissions > inherited permissions.

**Constraints and Limitations:**
- **Spatie has no native hierarchy:** Requires third-party packages for inheritance .
- **Cycle risk:** Hierarchical systems must detect and prevent circular inheritance .
- **Performance:** Deep hierarchies increase resolution complexity (cached, but still more data).
- **Deny overrides:** Some packages support explicit deny permissions that override allows .

### Multiple Annotated Complete Code Examples

**Example 1: Flat Multi-Role Assignment**

```php
<?php
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

// Create permissions
Permission::create(['name' => 'edit articles']);
Permission::create(['name' => 'delete articles']);
Permission::create(['name' => 'ban users']);

// Create roles
$editor = Role::create(['name' => 'editor']);
$editor->givePermissionTo('edit articles');

$moderator = Role::create(['name' => 'moderator']);
$moderator->givePermissionTo(['delete articles', 'ban users']);

// Assign both roles to a user
$user->assignRole('editor', 'moderator');

// User now has all three permissions
$user->hasPermissionTo('edit articles');   // true (editor)
$user->hasPermissionTo('delete articles'); // true (moderator)
$user->hasPermissionTo('ban users');       // true (moderator)
```

**Expected Output:** The user has the union of Editor and Moderator permissions.

**Why:** `assignRole()` inserts rows into `model_has_roles` for both roles. Permission resolution unions all role permissions .

---

**Example 2: Hierarchical Role Inheritance**

```php
<?php
use Hosseinhezami\PermissionManager\Models\Role;

// Build hierarchy: super-admin > admin > editor
$superAdmin = Role::create(['name' => 'Super Admin', 'slug' => 'super-admin']);
$admin = Role::create(['name' => 'Admin', 'slug' => 'admin']);
$editor = Role::create(['name' => 'Editor', 'slug' => 'editor']);

// Assign permissions at each level
$editor->assignPermission('posts.edit');
$admin->assignPermission('posts.delete');
$superAdmin->assignPermission('system.config');

// Build inheritance
$admin->inheritFrom('editor');       // admin inherits editor permissions
$superAdmin->inheritFrom('admin');   // super-admin inherits admin permissions

// User with super-admin has all permissions
$user->assignRole('super-admin');
$user->hasPermissionTo('posts.edit');    // ✅ via editor
$user->hasPermissionTo('posts.delete');  // ✅ via admin
$user->hasPermissionTo('system.config'); // ✅ direct
```

**Expected Output:** The super-admin user possesses all permissions through the inheritance chain.

**Why:** `inheritFrom()` creates a parent-child relationship. Permission resolution traverses the hierarchy, unioning permissions at each level .

### Real-World Cases

- **Flat Multi-Role:** A user who is both a "Content Editor" and "Community Moderator" with independent permission sets.
- **Hierarchical:** An organization where "Department Head" inherits all "Team Lead" permissions, which inherits all "Staff" permissions.
- **Enterprise RBAC:** Complex approval chains with escalating privilege levels .

### References

- Hosseinhezami Laravel Permission Manager: Role Hierarchy — https://github.com/hosseinhezami/laravel-permission-manager 
- Whilesmart Eloquent Roles: Hierarchical Permissions — https://packagist.org/packages/whilesmart/eloquent-roles 
- Fanmade Delegated Permissions: Inheritance-based — https://packagist.org/packages/fanmade/laravel-delegated-permissions 

---

## 3. Implementing Custom Cache-Backed Role Checks

### Definitions

**Core Definition:** Cache-backed role checks store permission and role data in a fast-access cache (Redis, Memcached) to eliminate repeated database queries during authorization checks, preventing performance bottlenecks in high-traffic applications.

**Technical Definition:** Spatie Laravel Permission caches all permissions and their role relationships for 24 hours (configurable) using the `PermissionRegistrar` class. The cache key is `spatie.permission.cache`. The cache is automatically invalidated when roles or permissions are created, updated, deleted, or when role-permission assignments change. User-specific role assignments are kept in-memory per request (since v4.4.0) and do not flush the global cache. For multi-tenant applications, cache prefixes must be unique per tenant.

**Beginner-Friendly Explanation:** Checking permissions on every request would normally mean hitting the database constantly. Spatie caches the entire permission structure in memory (Redis), so permission checks are nearly instant. The cache automatically refreshes when you change roles or permissions. You don't need to manually cache user permissions—Spatie handles it.

### Purposes

- To eliminate database queries during permission checks.
- To improve response times for high-traffic applications.
- To automatically invalidate cached data when permissions change.
- To support multi-tenant cache isolation via prefixes.
- To provide a central cache reset mechanism for deployments.

### Syntax Rules and Structure

**Complete General Syntax — Configuration:**

```php
<?php
// config/permission.php
return [
    'cache' => [
        // Cache duration (default: 24 hours)
        'expiration_time' => \DateInterval::createFromDateString('24 hours'),

        // Cache key (do not change)
        'key' => 'spatie.permission.cache',

        // Cache store (use Redis for production)
        'store' => 'redis', // or 'default', 'memcached', 'array'
    ],
];
```

**Complete General Syntax — Manual Cache Reset:**

```php
<?php
use Spatie\Permission\PermissionRegistrar;

// Programmatic reset
app(PermissionRegistrar::class)->forgetCachedPermissions();

// Artisan command
php artisan permission:cache-reset

// Or via cache:forget
php artisan cache:forget spatie.permission.cache
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `expiration_time` | How long the cache persists (default: 24 hours). |
| `key` | The cache key (`spatie.permission.cache`). |
| `store` | The cache driver (`redis`, `memcached`, `default`). |
| `forgetCachedPermissions()` | Manually clears the cache. |

**Complete General Syntax — Eager Loading for Performance:**

```php
<?php
// ✅ Good: Eager load relationships
$users = User::with(['roles.permissions', 'permissions'])->get();

foreach ($users as $user) {
    if ($user->hasPermissionTo('edit-articles')) {
        // Uses cached data + preloaded relationships
    }
}

// ✅ Good: Query scope (database-level filtering)
$editors = User::permission('edit-articles')->get();
```

**Syntax Rules:**
- Configure `cache.store` to `redis` for production performance .
- The cache is automatically invalidated on permission/role CRUD operations .
- User-specific assignments do not flush the global cache (performance optimization) .
- For multi-tenant apps, set a unique cache prefix per tenant .
- Use `User::permission()` query scope instead of loading all users and filtering in PHP .

**Constraints and Limitations:**
- **Direct database manipulation:** If you modify permission tables directly (not via package methods), the cache will not auto-invalidate .
- **Octane:** May need `register_octane_reset_listener` in config for proper cache reset .
- **Cache stampede:** The package implements thread-safe loading to prevent concurrent cache misses .
- **Manual user caching unnecessary:** Spatie already caches efficiently; custom user-level caching adds complexity .

### Multiple Annotated Complete Code Examples

**Example 1: Redis Cache Configuration**

```php
<?php
// config/permission.php
return [
    'cache' => [
        'expiration_time' => \DateInterval::createFromDateString('24 hours'),
        'key' => 'spatie.permission.cache',
        'store' => 'redis', // Use Redis for high-traffic applications
    ],
];
```

**Expected Output:** Permission and role data is cached in Redis for 24 hours. Subsequent permission checks read from Redis instead of the database.

**Why:** Setting `store` to `redis` directs the `PermissionRegistrar` to use the Redis cache driver. Redis provides sub-millisecond reads for permission data .

---

**Example 2: Automatic Cache Invalidation**

```php
<?php
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

// These operations automatically flush the cache:
Permission::create(['name' => 'new-permission']);        // Cache flushed
$permission->update(['name' => 'updated-name']);         // Cache flushed
$permission->delete();                                    // Cache flushed

$role = Role::create(['name' => 'new-role']);            // Cache flushed
$role->givePermissionTo('edit articles');                 // Cache flushed
$role->revokePermissionTo('edit articles');               // Cache flushed

// User-level changes do NOT flush the cache (performance optimization):
$user->assignRole('editor');                              // Cache NOT flushed
$user->givePermissionTo('edit articles');                 // Cache NOT flushed
```

**Expected Output:** Global permission structure changes invalidate the cache. User-level assignments do not, as they don't change the global structure.

**Why:** The `RefreshesPermissionCache` trait on Role and Permission models fires on save/delete. User-level changes are in-memory per request since v4.4.0 .

---

**Example 3: Multi-Tenant Cache Prefix**

```php
<?php
// config/cache.php
return [
    'prefix' => env('CACHE_PREFIX', 'laravel_cache'),
];
```

```env
# .env — Set a unique prefix per application/tenant
CACHE_PREFIX=tenant_123_
```

**Expected Output:** Each tenant's permission cache is isolated, preventing cache clashes between tenants sharing the same Redis instance.

**Why:** Setting a unique cache prefix ensures that tenant A's cached permissions are not read by tenant B. This is critical for multi-tenant applications .

---

**Example 4: Eager Loading for N+1 Prevention**

```php
<?php
// ❌ Bad: N+1 queries
$users = User::all();
foreach ($users as $user) {
    if ($user->hasPermissionTo('edit-articles')) {
        // Each check may query the database
    }
}

// ✅ Good: Eager loading
$users = User::with(['roles.permissions', 'permissions'])->get();
foreach ($users as $user) {
    if ($user->hasPermissionTo('edit-articles')) {
        // Uses preloaded relationships + cached permission data
    }
}

// ✅ Best: Database-level filtering
$editors = User::permission('edit-articles')->get();
```

**Expected Output:** The eager-loaded version eliminates N+1 queries. The query scope version filters at the database level.

**Why:** Eager loading preloads role and permission relationships. The `permission()` query scope performs a database-level join, avoiding loading unnecessary users .

### Real-World Cases

- **High-Traffic SaaS:** Redis-backed permission cache handles thousands of concurrent permission checks per second.
- **Multi-Tenant Applications:** Unique cache prefixes isolate tenant permission data.
- **CI/CD Deployments:** `php artisan permission:cache-reset` after deploying new permissions.
- **Octane Applications:** Automatic cache reset listener prevents stale data across requests .

### References

- Spatie Laravel Permission: Cache — https://spatie.be/docs/laravel-permission/v6/advanced-usage/cache 
- Spatie Laravel Permission: Performance Optimization — https://mintlify.wiki/spatie/laravel-permission/best-practices/performance 
- Spatie Laravel Permission: Caching (Advanced) — https://mintlify.wiki/spatie/laravel-permission/advanced-usage/caching 
- Is it necessary to manually cache user permissions? (Laracasts) — https://laracasts.com/discuss/channels/general-discussion/is-it-necessary-to-manually-cache-user-permissions-when-using-spatie-laravel-permission 

---

## 4. Ecosystem Standards: Spatie Laravel-Permission

### Definitions

**Core Definition:** Spatie Laravel Permission is the de facto standard community package for implementing RBAC in Laravel, providing a well-tested, feature-rich, and actively maintained solution for roles and permissions.

**Technical Definition:** The package provides `HasRoles` and `HasPermissions` traits for models, `Role` and `Permission` models, middleware (`role`, `permission`, `role_or_permission`), Blade directives (`@role`, `@hasrole`, `@can`), Artisan commands (`permission:create-role`, `permission:create-permission`), and a comprehensive caching layer. It supports multi-guard authentication, teams/context scoping, wildcard permissions, and direct user-permission assignments.

**Beginner-Friendly Explanation:** Spatie Laravel Permission is the package that 90% of Laravel applications use for roles and permissions. It's well-documented, actively maintained, and handles all the complex parts (caching, polymorphic relationships, middleware, Blade directives) so you can focus on your application logic. If you're building RBAC, start here.

### Purposes

- To provide a battle-tested RBAC implementation without building from scratch.
- To offer middleware, Blade directives, and Artisan commands for common tasks.
- To support multi-guard and team-scoped permissions.
- To handle caching automatically for performance.
- To provide a clear migration path from custom implementations.

### Syntax Rules and Structure

**Complete General Syntax — Installation:**

```bash
composer require spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

**Complete General Syntax — Model Trait:**

```php
<?php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
}
```

**Complete General Syntax — Middleware Registration (Laravel 11+):**

```php
<?php
// bootstrap/app.php
use Spatie\Permission\Middleware\PermissionMiddleware;
use Spatie\Permission\Middleware\RoleMiddleware;
use Spatie\Permission\Middleware\RoleOrPermissionMiddleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => RoleMiddleware::class,
            'permission' => PermissionMiddleware::class,
            'role_or_permission' => RoleOrPermissionMiddleware::class,
        ]);
    })
    ->create();
```

**Complete General Syntax — Route Protection:**

```php
<?php
// Single role
Route::get('/admin/dashboard', [AdminController::class, 'dashboard'])
    ->middleware('role:admin');

// Single permission
Route::get('/articles/edit/{article}', [ArticleController::class, 'edit'])
    ->middleware('permission:edit articles');

// Either role OR permission
Route::put('/articles/{article}', [ArticleController::class, 'update'])
    ->middleware('role_or_permission:admin|edit articles');

// Multiple roles (OR)
Route::get('/content/edit', function () {
    // ...
})->middleware('role:admin|editor|writer');
```

**Complete General Syntax — Blade Directives:**

```blade
@role('admin')
    <a href="{{ route('admin.dashboard') }}">Admin Dashboard</a>
@endrole

@hasrole('editor')
    <button>Edit Article</button>
@endhasrole

@can('edit articles')
    <a href="{{ route('articles.edit', $article) }}">Edit</a>
@endcan

@hasanyrole('admin|editor')
    <div>Content Management</div>
@endhasanyrole
```

**Complete General Syntax — Artisan Commands:**

```bash
# Create roles
php artisan permission:create-role writer
php artisan permission:create-role writer web "create articles|edit articles"

# Create permissions
php artisan permission:create-permission "edit articles"
php artisan permission:create-permission "edit articles" web

# Cache reset
php artisan permission:cache-reset
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `HasRoles` trait | Adds role/permission methods to the model. |
| `role:` middleware | Checks user roles. |
| `permission:` middleware | Checks user permissions. |
| `role_or_permission:` middleware | Checks either role or permission. |
| `@role`, `@hasrole`, `@can` | Blade directives for conditional UI. |
| `permission:create-role` | Artisan command for role creation. |

**Syntax Rules:**
- The `HasRoles` trait must be added to the User model .
- Middleware aliases must be registered in `bootstrap/app.php` (Laravel 11+) or `Kernel.php` (Laravel 10-) .
- Multiple roles/permissions use `|` as separator for OR logic .
- Blade directives accept a guard name as the second argument for multi-guard systems .
- Artisan commands accept a guard name as the second argument .

**Constraints and Limitations:**
- **No native hierarchy:** Spatie is flat multi-role; hierarchies require extensions .
- **Cache invalidation:** Direct database manipulation bypasses automatic cache invalidation .
- **Guard mismatch:** Roles/permissions must match the user's guard, or `GuardDoesNotMatch` is thrown .
- **Testing:** In tests, re-register permissions after seeding if the Gate was already resolved .

### Multiple Annotated Complete Code Examples

**Example 1: Complete Installation and Setup**

```bash
# Install the package
composer require spatie/laravel-permission

# Publish migrations and config
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"

# Run migrations
php artisan migrate
```

```php
<?php
// app/Models/User.php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
}
```

```php
<?php
// bootstrap/app.php (Laravel 11+)
use Spatie\Permission\Middleware\PermissionMiddleware;
use Spatie\Permission\Middleware\RoleMiddleware;
use Spatie\Permission\Middleware\RoleOrPermissionMiddleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => RoleMiddleware::class,
            'permission' => PermissionMiddleware::class,
            'role_or_permission' => RoleOrPermissionMiddleware::class,
        ]);
    })
    ->create();
```

**Expected Output:** The package is installed, migrations create five tables, the User model has role/permission methods, and middleware aliases are registered.

**Why:** The `HasRoles` trait adds methods like `assignRole()`, `hasRole()`, and `hasPermissionTo()`. Middleware aliases enable route protection .

---

**Example 2: Middleware and Route Protection**

```php
<?php
use App\Http\Controllers\ArticleController;
use App\Http\Controllers\AdminController;

// Permission middleware
Route::get('/articles/edit/{article}', [ArticleController::class, 'edit'])
    ->middleware('permission:edit articles');

// Role middleware
Route::get('/admin/dashboard', [AdminController::class, 'dashboard'])
    ->middleware('role:admin');

// Role OR permission middleware
Route::put('/articles/{article}', [ArticleController::class, 'update'])
    ->middleware('role_or_permission:admin|edit articles');

// Multiple roles (OR)
Route::get('/content', function () {
    return 'Content Management';
})->middleware('role:admin|editor|writer');
```

**Expected Output:** Only users with the required role or permission can access each route. Unauthorized users receive a 403 response.

**Why:** `PermissionMiddleware` checks `hasPermissionTo()`, `RoleMiddleware` checks `hasRole()`, and `RoleOrPermissionMiddleware` checks either. Pipe-separated values use OR logic .

---

**Example 3: Blade Directives for UI Control**

```blade
{{-- resources/views/articles/show.blade.php --}}

@role('admin')
    <a href="{{ route('admin.dashboard') }}">Admin Dashboard</a>
@endrole

@hasrole('editor')
    <a href="{{ route('articles.edit', $article) }}">Edit Article</a>
@endhasrole

@can('delete articles')
    <form method="POST" action="{{ route('articles.destroy', $article) }}">
        @csrf
        @method('DELETE')
        <button type="submit">Delete</button>
    </form>
@endcan

@hasanyrole('admin|editor')
    <div class="content-management">
        <h3>Content Management</h3>
    </div>
@endhasanyrole
```

**Expected Output:** UI elements are conditionally rendered based on the user's roles and permissions.

**Why:** Blade directives map to `hasRole()`, `hasPermissionTo()`, and `hasAnyRole()` methods. They provide a clean syntax for conditional rendering .

---

**Example 4: Artisan Commands**

```bash
# Create a role
php artisan permission:create-role writer

# Create a role with permissions (for a specific guard)
php artisan permission:create-role writer web "create articles|edit articles"

# Create a permission
php artisan permission:create-permission "edit articles"

# Create a permission for a specific guard
php artisan permission:create-permission "edit articles" api

# Reset the permission cache
php artisan permission:cache-reset
```

**Expected Output:** Roles and permissions are created via the command line. The cache is reset after changes.

**Why:** Artisan commands provide a CLI interface for role/permission management, useful in deployment scripts and CI/CD pipelines .

### Real-World Cases

- **SaaS Multi-Tenant:** Spatie with teams feature for per-tenant roles .
- **CMS Systems:** Spatie with Blade directives for UI permission control.
- **API Gateways:** Spatie with `permission` middleware for API route protection .
- **Enterprise RBAC:** Spatie as the foundation, extended with custom hierarchies for complex organizations.

### References

- Spatie Laravel Permission: Using Middleware — https://mintlify.wiki/spatie/laravel-permission/basic-usage/using-middleware 
- Spatie Laravel Permission: Database Tables — https://mintlify.wiki/spatie/laravel-permission/configuration/database-tables 
- Spatie Laravel Permission: Cache — https://spatie.be/docs/laravel-permission/v6/advanced-usage/cache 
- Spatie Laravel Permission: RoleMiddleware — https://mintlify.wiki/spatie/laravel-permission/api/middleware/role-middleware 
- Spatie Laravel Permission: Performance — https://mintlify.wiki/spatie/laravel-permission/best-practices/performance 
- Spatie Laravel Permission: Multi-Guard — https://packagist.org/packages/wakjoko/laravel-permission 
- Jetstream Teams + Spatie (Laracasts) — https://laracasts.com/index.php/discuss/channels/design/jetstream-teams-spatielaravel-permissions 
- Hosseinhezami Laravel Permission Manager — https://github.com/hosseinhezami/laravel-permission-manager 
- Is manual user permission caching necessary? (Laracasts) — https://laracasts.com/discuss/channels/general-discussion/is-it-necessary-to-manually-cache-user-permissions-when-using-spatie-laravel-permission 
- Spatie Laravel Permission: Caching (Advanced) — https://mintlify.wiki/spatie/laravel-permission/advanced-usage/caching 

---

## References

- Spatie Laravel Permission: Using Middleware — https://mintlify.wiki/spatie/laravel-permission/basic-usage/using-middleware 
- Spatie Laravel Permission: Database Tables — https://mintlify.wiki/spatie/laravel-permission/configuration/database-tables 
- Spatie Laravel Permission: Cache — https://spatie.be/docs/laravel-permission/v6/advanced-usage/cache 
- Whilesmart Eloquent Roles: Hierarchical Permissions — https://packagist.org/packages/whilesmart/eloquent-roles 
- Hosseinhezami Laravel Permission Manager — https://github.com/hosseinhezami/laravel-permission-manager 
- Spatie Laravel Permission: RoleMiddleware — https://mintlify.wiki/spatie/laravel-permission/api/middleware/role-middleware 
- Spatie Laravel Permission: UUID (v5) — https://spatie.be/docs/laravel-permission/v5/advanced-usage/uuid 
- Spatie Laravel Permission: Performance — https://mintlify.wiki/spatie/laravel-permission/best-practices/performance 
- Noxomix Laravel Rollo: Context-based RBAC — https://packagist.org/packages/noxomix/laravel-rollo 
- Hosseinhezami Permission Manager: Role Hierarchy — https://packagist.org/packages/hosseinhezami/laravel-permission-manager 
- Wakjoko Laravel Permission: Multi-Guard — https://packagist.org/packages/wakjoko/laravel-permission 
- Laravel Permissions for Institutions and Campuses (Laracasts) — https://laracasts.com/discuss/channels/site-improvements/laravel-permissions-for-insitutions-and-campuses 
- Spatie Laravel Permission: Caching (Advanced) — https://mintlify.wiki/spatie/laravel-permission/advanced-usage/caching 
- Fanmade Laravel Delegated Permissions — https://packagist.org/packages/fanmade/laravel-delegated-permissions 
- Jetstream Teams + Spatie (Laracasts) — https://laracasts.com/index.php/discuss/channels/design/jetstream-teams-spatielaravel-permissions 
- Spatie Laravel Permission: UUID/ULID (v7) — https://spatie.be/docs/laravel-permission/v7/advanced-usage/uuid 
- Is manual user permission caching necessary? (Laracasts) — https://laracasts.com/discuss/channels/general-discussion/is-it-necessary-to-manually-cache-user-permissions-when-using-spatie-laravel-permission 