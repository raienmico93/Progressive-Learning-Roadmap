# Laravel Route Groups: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Route Groups are a Laravel routing feature that allows multiple routes to share common attributes—such as prefixes, middleware, namespaces, domains, and name prefixes—without repeating those attributes on every individual route definition.

**Technical Definition**

Route groups are implemented via the `Route::group()` method (or its fluent chainable equivalents) on the `Illuminate\Routing\Router` class. When a group is defined, Laravel's router applies the specified shared attributes to every route within the group's closure. Internally, the router merges group attributes with route-level attributes using `RouteGroup::merge()`, handling middleware (merged), namespace (appended), prefix (prepended), name prefix (prepended), and domain (set) according to specific merge rules.

**Beginner-Friendly Explanation**

Route groups are like a folder for your routes. If you have many routes that all start with `/admin`, all require the same login check, or all belong to the same controller namespace, you can put them in a group and specify those shared settings just once. Instead of writing the same prefix or middleware on every single route, you write it once on the group.

---

### Key Characteristics

- **Attribute Sharing**: Middleware, prefixes, name prefixes, namespaces, and domains can be applied to all routes within a group.
- **Nested Groups**: Groups can be nested, with attributes merging intelligently (middleware merges, prefixes append, names prepend).
- **Fluent and Array Syntax**: Groups can be defined using either the classic array syntax (`Route::group(['prefix' => 'admin'], ...)`) or the modern fluent chainable syntax (`Route::prefix('admin')->group(...)`).
- **Subdomain Routing**: Groups can be bound to specific domains or wildcard subdomains, enabling multi-tenant and multi-site architectures.
- **Controller Grouping**: The `controller()` method can bind a single controller to all routes in a group, reducing repetitive controller references.
- **Version-Specific Namespace Handling**: In Laravel 11+, the `namespace()` method is no longer necessary for most use cases, as route files are loaded without a default namespace wrapper.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Basic knowledge of Laravel routing** (defining routes, HTTP verbs, route parameters).
- **Familiarity with middleware and controllers** is helpful.
- **Understanding of namespaces** in PHP.

---

### Related Programming Areas

- **Middleware**: Route groups are the primary mechanism for applying middleware to multiple routes.
- **Controllers**: Groups can share controller namespaces and bind controllers to routes.
- **Subdomain/Multi-tenant Architecture**: Domain-based groups enable multi-tenant applications.
- **API Versioning**: Groups with URI prefixes are commonly used for versioned APIs (`/api/v1`, `/api/v2`).
- **Route Caching**: Groups affect how routes are compiled and cached.

---

## Core Concepts

---

### 1. Route Prefixes

#### Definitions

**Core Definition**

A route prefix is a URI segment that is prepended to every route within a group, allowing routes to share a common URL path without repeating it.

**Technical Definition**

The `prefix()` method (or the `'prefix'` key in the array syntax) adds a URI segment to the beginning of each route in the group. Nested prefixes are concatenated. The prefix is applied to the URI only, not to the route name. Internally, the prefix is prepended to the route's URI pattern during group merge.

**Beginner-Friendly Explanation**

A prefix is a common beginning for a set of URLs. For example, all admin URLs might start with `/admin`. Instead of writing `/admin/dashboard`, `/admin/users`, etc., you write `prefix('admin')` on the group once.

#### Purposes

- To group routes under a common URL segment (e.g., `/admin`, `/api/v1`).
- To avoid repeating the same prefix on each route definition.
- To organize routes by section (admin, API, account).
- To enable versioned APIs with clean URL structures.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Fluent syntax (recommended)
Route::prefix('admin')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/users', [UserController::class, 'index']);
});

// Array syntax
Route::group(['prefix' => 'admin'], function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `prefix('admin')` | Adds `/admin` to all grouped route URIs. |
| `group(function () { ... })` | Contains the grouped routes. |
| `'prefix' => 'admin'` | Array syntax equivalent. |

**Syntax Rules**

- The prefix string should not include a leading slash.
- Nested prefixes are concatenated (e.g., `prefix('api')->prefix('v1')` results in `/api/v1`).
- The prefix applies to URIs only, not route names.
- Prefixes cannot contain route parameters.

**Constraints and Limitations**

- A leading slash is added automatically by Laravel.
- Prefixes with parameters are not supported (use domain parameters for subdomains).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Admin Prefix**

**Code**

```php
// routes/web.php
use App\Http\Controllers\Admin\DashboardController;
use App\Http\Controllers\Admin\UserController;
use Illuminate\Support\Facades\Route;

Route::prefix('admin')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/users', [UserController::class, 'index']);
    Route::get('/settings', function () {
        return 'Admin Settings';
    });
});
```

**Expected Output**

```
// GET /admin/dashboard → Dashboard index
// GET /admin/users → User index
// GET /admin/settings → "Admin Settings"
```

**Why This Output Occurs**

The `/admin` prefix is prepended to every route in the group, resulting in `/admin/dashboard`, `/admin/users`, and `/admin/settings`.

---

**Example 2: Nested Prefixes**

**Code**

```php
Route::prefix('api')->group(function () {
    Route::prefix('v1')->group(function () {
        Route::get('/users', function () {
            return 'API v1 Users';
        });
        Route::get('/posts', function () {
            return 'API v1 Posts';
        });
    });
});
```

**Expected Output**

```
// GET /api/v1/users → "API v1 Users"
// GET /api/v1/posts → "API v1 Posts"
```

**Why This Output Occurs**

The outer `api` prefix is concatenated with the inner `v1` prefix, resulting in `/api/v1/` for all routes in the inner group.

---

#### Real-World Cases with Explanation

**Case 1: Admin Panel**

All admin routes are prefixed with `/admin`, keeping them separate from public routes. The prefix is defined once on the group, so adding a new admin route requires no prefix specification.

**Explanation**: Prefixes provide a clear URL structure and reduce repetition.

**Case 2: API Versioning**

A public API uses `/api/v1` for the current version and `/api/v2` for the next version. Each version is a separate group with its own prefix.

**Explanation**: Prefixes enable clean API versioning without duplicating middleware or controller references.

---

### 2. Route Name Prefixes

#### Definitions

**Core Definition**

A route name prefix is a string prepended to the names of all routes within a group, creating a consistent naming convention (e.g., `admin.dashboard`, `admin.users`).

**Technical Definition**

The `name()` method (or the `'as'` key in array syntax) prefixes each route name in the group with the given string. The string is prepended exactly as specified, so a trailing dot (`.`) must be included for proper name spacing. Nested name prefixes are also concatenated.

**Beginner-Friendly Explanation**

A name prefix adds a common beginning to all the names of routes in a group. For example, if you prefix with `admin.`, a route named `dashboard` becomes `admin.dashboard`.

#### Purposes

- To avoid route name collisions between different sections of an application.
- To create a consistent naming convention for route groups.
- To enable organized route name references in views and controllers.
- To support logical grouping of named routes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::name('admin.')->group(function () {
    Route::get('/dashboard', function () {
        // ...
    })->name('dashboard'); // Full name: admin.dashboard

    Route::get('/users', function () {
        // ...
    })->name('users'); // Full name: admin.users
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `name('admin.')` | Prefixes all route names with `admin.`. |
| `->name('dashboard')` | Route name becomes `admin.dashboard`. |

**Syntax Rules**

- Include the trailing dot (`.`) in the prefix for proper separation.
- Nested name prefixes are concatenated.
- The name prefix applies to route names only, not URIs.

**Constraints and Limitations**

- Route names must be unique; duplicate names cause `route:cache` to fail.
- The name prefix is purely a string prefix; Laravel does not add separators automatically.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Admin Name Prefix**

**Code**

```php
Route::name('admin.')->prefix('admin')->group(function () {
    Route::get('/dashboard', function () {
        return 'Admin Dashboard';
    })->name('dashboard');

    Route::get('/users', function () {
        return 'User Management';
    })->name('users');
});

// Generating URLs
$dashboardUrl = route('admin.dashboard'); // /admin/dashboard
$usersUrl = route('admin.users'); // /admin/users
```

**Expected Output**

```
// route('admin.dashboard') → http://example.com/admin/dashboard
// route('admin.users') → http://example.com/admin/users
```

**Why This Output Occurs**

The `name('admin.')` prefix is applied to both route names, creating `admin.dashboard` and `admin.users`. The `prefix('admin')` adds the URI prefix.

---

**Example 2: Combined Name and Prefix**

**Code**

```php
Route::prefix('api')->name('api.')->group(function () {
    Route::get('/users', function () {
        return 'API Users';
    })->name('users');
});
```

**Expected Output**

```
// GET /api/users → "API Users"
// route('api.users') → http://example.com/api/users
```

**Why This Output Occurs**

The group applies both a URI prefix (`/api`) and a name prefix (`api.`), resulting in the URI `/api/users` and the route name `api.users`.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Section Application**

A SaaS application has `admin.` and `user.` name prefixes for routes in the admin and user sections, preventing name collisions and enabling clear references.

**Explanation**: Name prefixes organize route names logically, especially in large applications.

---

### 3. Route Middleware

#### Definitions

**Core Definition**

Route middleware are filters applied to all routes within a group, used for authentication, authorization, logging, rate limiting, and other cross-cutting concerns.

**Technical Definition**

The `middleware()` method (or the `'middleware'` key) assigns one or more middleware to all routes in the group. Middleware are executed in the order they are listed. Nested groups merge middleware arrays, with the inner group's middleware appended to the outer group's middleware.

**Beginner-Friendly Explanation**

Middleware are like security guards that check requests before they reach your route. Putting middleware on a group means every route in that group is protected by those checks.

#### Purposes

- To protect a group of routes with authentication or authorization middleware.
- To apply rate limiting to API route groups.
- To enforce consistent request filtering across related routes.
- To reduce repetition of middleware assignments.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/settings', [SettingsController::class, 'index']);
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `middleware(['auth', 'verified'])` | Applies both middleware to the group. |
| `group(function () { ... })` | Contains the protected routes. |

**Syntax Rules**

- Middleware are executed in the order listed.
- Nested groups merge middleware (outer + inner).
- Middleware can be assigned as strings (aliases) or class names.
- The `web` and `api` middleware groups are applied automatically based on route file.

**Constraints and Limitations**

- Middleware order matters for authentication dependencies.
- Middleware must be registered in the application's middleware stack.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Authenticated Route Group**

**Code**

```php
Route::middleware(['auth', 'verified'])->prefix('account')->group(function () {
    Route::get('/profile', function () {
        return 'User Profile';
    });
    Route::get('/settings', function () {
        return 'Account Settings';
    });
});
```

**Expected Output**

```
// If authenticated and verified:
// GET /account/profile → "User Profile"
// GET /account/settings → "Account Settings"

// If not authenticated: redirect to /login
// If not verified: redirect to /email/verify
```

**Why This Output Occurs**

The `auth` middleware ensures the user is logged in, and `verified` ensures the email is verified. Both must pass for the routes to execute.

---

**Example 2: API Rate Limiting**

**Code**

```php
Route::middleware(['auth:sanctum', 'throttle:api'])->prefix('api/v1')->group(function () {
    Route::get('/users', [Api\UserController::class, 'index']);
    Route::post('/users', [Api\UserController::class, 'store']);
});
```

**Expected Output**

```
// Authenticated API request within rate limit → 200 with data
// Authenticated API request exceeding rate limit → 429 Too Many Requests
```

**Why This Output Occurs**

The `auth:sanctum` middleware authenticates the request via Sanctum token, and `throttle:api` enforces the API rate limit.

---

#### Real-World Cases with Explanation

**Case 1: Admin Protection**

An admin route group applies `auth` and `admin` middleware, ensuring only authenticated administrators can access admin routes.

**Explanation**: Middleware on groups provides consistent protection without repeating it on each route.

**Case 2: API Rate Limiting**

All API routes are wrapped in a group with `throttle:api` to prevent abuse.

**Explanation**: Group middleware ensures uniform rate limiting across all API endpoints.

---

### 4. Route Domain

#### Definitions

**Core Definition**

A route domain is a specific hostname (e.g., `admin.example.com` or `{account}.myapp.com`) to which a group of routes is bound, enabling multi-domain and multi-tenant routing.

**Technical Definition**

The `domain()` method (or the `'domain'` key) binds all routes in the group to the specified domain. Domain segments can include route parameters (e.g., `{account}`), which are passed to the route handler. Wildcard subdomains use curly braces with parameter names.

**Beginner-Friendly Explanation**

A route domain lets you serve different routes based on the website address someone visits. For example, `admin.example.com` can have different routes than `www.example.com`.

#### Purposes

- To serve different sections of an application on different domains or subdomains.
- To enable multi-tenant applications with tenant-specific subdomains.
- To separate public and administrative interfaces by domain.
- To support wildcard subdomains for dynamic tenant resolution.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::domain('admin.example.com')->group(function () {
    Route::get('/dashboard', function () {
        return 'Admin Dashboard';
    });
});

// Wildcard subdomain
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('/user/{id}', function (string $account, string $id) {
        return "Account: {$account}, User: {$id}";
    });
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `domain('admin.example.com')` | Binds routes to the specified domain. |
| `{account}` | A dynamic subdomain parameter. |
| `function (string $account, ...)` | Receives the subdomain parameter. |

**Syntax Rules**

- The domain can be a literal string or contain route parameters.
- Domain parameters are passed as the first arguments to the route handler.
- Subdomain routes should be registered before root domain routes to avoid conflicts.
- Wildcard subdomains are supported.

**Constraints and Limitations**

- The domain must be configured in DNS to point to the server.
- Domain parameters are available in route handlers but not in `route()` generation unless provided.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Admin Subdomain**

**Code**

```php
Route::domain('admin.myapp.com')->group(function () {
    Route::get('/', function () {
        return 'Admin Home';
    })->name('admin.home');

    Route::get('/users', function () {
        return 'Admin Users';
    })->name('admin.users');
});

Route::domain('www.myapp.com')->group(function () {
    Route::get('/', function () {
        return 'Public Home';
    })->name('home');
});
```

**Expected Output**

```
// https://admin.myapp.com/ → "Admin Home"
// https://admin.myapp.com/users → "Admin Users"
// https://www.myapp.com/ → "Public Home"
```

**Why This Output Occurs**

Each group is bound to its respective domain. Requests are routed based on the hostname.

---

**Example 2: Wildcard Subdomain (Multi-Tenant)**

**Code**

```php
Route::domain('{tenant}.myapp.com')->group(function () {
    Route::get('/', function (string $tenant) {
        return "Welcome to {$tenant}'s workspace";
    })->name('tenant.home');
});
```

**Expected Output**

```
// https://acme.myapp.com/ → "Welcome to acme's workspace"
// https://globex.myapp.com/ → "Welcome to globex's workspace"
```

**Why This Output Occurs**

The `{tenant}` parameter captures the subdomain portion and passes it to the route handler.

---

#### Real-World Cases with Explanation

**Case 1: SaaS with Tenant Subdomains**

A SaaS application gives each customer a subdomain (e.g., `acme.myapp.com`). A wildcard domain group captures the tenant parameter and scopes all queries to that tenant.

**Explanation**: Domain groups enable scalable multi-tenant architecture.

**Case 2: Separate Admin and Public Sites**

An application serves its marketing site on `www.example.com` and its admin panel on `admin.example.com`, both from the same Laravel codebase.

**Explanation**: Domain groups separate concerns without maintaining separate applications.

---

### 5. Route Controller Groups

#### Definitions

**Core Definition**

A route controller group allows all routes within a group to be bound to a single controller class, eliminating the need to specify the controller on each route.

**Technical Definition**

The `controller()` method (available in Laravel 9+) sets the default controller for all routes in the group. Each route only needs to specify the method name as a string. Internally, the router resolves the method against the group's controller.

**Beginner-Friendly Explanation**

Instead of writing the controller class name on every route, you can specify it once on the group. Then each route just references the method name.

#### Purposes

- To reduce repetition when many routes use the same controller.
- To improve readability by focusing on method names.
- To organize routes by controller logically.
- To support resourceful controllers with additional custom routes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use App\Http\Controllers\Admin\DashboardController;

Route::controller(DashboardController::class)->prefix('admin')->group(function () {
    Route::get('/dashboard', 'index')->name('admin.dashboard');
    Route::get('/stats', 'stats')->name('admin.stats');
    Route::get('/reports', 'reports')->name('admin.reports');
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `controller(DashboardController::class)` | Sets the default controller. |
| `'index'` | Method name (resolved against the controller). |

**Syntax Rules**

- The controller class must be imported.
- Route actions are specified as method name strings.
- Can be combined with other group attributes (prefix, middleware, name).

**Constraints and Limitations**

- All routes in the group must use the same controller.
- Method names must exist on the controller.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Controller Group**

**Controller**

```php
namespace App\Http\Controllers\Admin;

class ReportController
{
    public function index() { return 'Report Index'; }
    public function sales() { return 'Sales Report'; }
    public function traffic() { return 'Traffic Report'; }
}
```

**Route**

```php
use App\Http\Controllers\Admin\ReportController;

Route::controller(ReportController::class)->prefix('admin/reports')->group(function () {
    Route::get('/', 'index')->name('admin.reports.index');
    Route::get('/sales', 'sales')->name('admin.reports.sales');
    Route::get('/traffic', 'traffic')->name('admin.reports.traffic');
});
```

**Expected Output**

```
// GET /admin/reports → "Report Index"
// GET /admin/reports/sales → "Sales Report"
// GET /admin/reports/traffic → "Traffic Report"
```

**Why This Output Occurs**

The `controller()` method binds `ReportController` to all routes, so only method names are needed.

---

#### Real-World Cases with Explanation

**Case 1: Admin Dashboard Controller**

All dashboard-related routes (`index`, `stats`, `reports`) are grouped under a single controller, reducing repetitive class references.

**Explanation**: Controller groups keep route files clean and organized.

---

### 6. Route Namespace

#### Definitions

**Core Definition**

A route namespace automatically prepends a PHP namespace to all controller references within a group, eliminating the need to specify the full namespace on each route.

**Technical Definition**

The `namespace()` method (or the `'namespace'` key) prepends the given namespace to all controller actions in the group. In Laravel 10 and earlier, the `RouteServiceProvider` applied a default namespace (`App\Http\Controllers`) to all routes. In Laravel 11+, this default namespace wrapper was removed, and controller classes must be imported explicitly unless a namespace group is used.

**Beginner-Friendly Explanation**

A namespace group lets you avoid writing the full namespace path for controllers in the same folder. For example, if all your admin controllers are in `App\Http\Controllers\Admin`, you can specify that namespace once on the group.

#### Purposes

- To reduce repetition of long controller namespaces.
- To organize controllers by section (Admin, Api, etc.).
- To maintain compatibility with legacy route definitions using `Controller@method` syntax.
- To group routes logically by namespace.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::namespace('App\Http\Controllers\Admin')->prefix('admin')->group(function () {
    Route::get('/dashboard', 'DashboardController@index');
    Route::get('/users', 'UserController@index');
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `namespace('App\Http\Controllers\Admin')` | Prepends the namespace to controller references. |
| `'DashboardController@index'` | Controller action syntax without namespace. |

**Syntax Rules**

- The namespace should not include a trailing backslash.
- In Laravel 11+, the `namespace()` method is still available but less commonly needed.
- The `Controller@method` string syntax works with namespace groups.

**Constraints and Limitations**

- The `namespace()` method is deprecated in favor of explicit imports in Laravel 11+.
- Using namespaces makes it harder for IDEs to resolve controller references.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Namespace Group**

**Code**

```php
Route::namespace('App\Http\Controllers\Admin')->prefix('admin')->group(function () {
    Route::get('/dashboard', 'DashboardController@index')->name('admin.dashboard');
    Route::get('/users', 'UserController@index')->name('admin.users');
});
```

**Expected Output**

```
// GET /admin/dashboard → DashboardController@index
// GET /admin/users → UserController@index
```

**Why This Output Occurs**

The `namespace()` method prepends `App\Http\Controllers\Admin` to both controller references, so `DashboardController` resolves to `App\Http\Controllers\Admin\DashboardController`.

---

**Example 2: API Namespace**

**Code**

```php
Route::namespace('App\Http\Controllers\Api')->prefix('api/v1')->group(function () {
    Route::get('/users', 'UserController@index');
    Route::post('/users', 'UserController@store');
});
```

**Expected Output**

```
// GET  /api/v1/users  → Api\UserController@index
// POST /api/v1/users  → Api\UserController@store
```

**Why This Output Occurs**

The namespace group maps all controller references to the `Api` namespace.

---

#### Real-World Cases with Explanation

**Case 1: Legacy Application Migration**

An application upgrading from Laravel 10 to 11 retains namespace groups to avoid rewriting all route definitions.

**Explanation**: Namespace groups provide backward compatibility for legacy route syntax.

---

### 7. Route Subdomain Routing

#### Definitions

**Core Definition**

Subdomain routing is a specialized form of domain routing where routes are bound to specific subdomains (e.g., `admin.example.com`, `api.example.com`), enabling logical separation of application sections by subdomain.

**Technical Definition**

Subdomain routing uses the `domain()` method on a group with a subdomain pattern. The subdomain can be static or dynamic (wildcard). Dynamic subdomains are captured as route parameters. Laravel matches the incoming request's `Host` header against the domain pattern. Subdomain routes must be registered before root domain routes to prevent conflicts.

**Beginner-Friendly Explanation**

Subdomain routing lets you have different parts of your application on different subdomains. For example, `admin.myapp.com` for the admin panel and `api.myapp.com` for the API.

#### Purposes

- To separate application sections by subdomain for clarity and SEO.
- To implement multi-tenant architectures with tenant-specific subdomains.
- To serve different user interfaces (admin, customer, API) on distinct subdomains.
- To leverage browser security features (same-origin policy) for separation.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Static subdomain
Route::domain('admin.example.com')->group(function () {
    Route::get('/dashboard', function () {
        return 'Admin Dashboard';
    });
});

// Dynamic subdomain (multi-tenant)
Route::domain('{tenant}.example.com')->group(function () {
    Route::get('/dashboard', function (string $tenant) {
        return "Dashboard for {$tenant}";
    });
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `domain('admin.example.com')` | Static subdomain. |
| `domain('{tenant}.example.com')` | Dynamic subdomain with parameter. |
| `string $tenant` | Captured subdomain parameter. |

**Syntax Rules**

- Subdomain routes must be registered before root domain routes.
- Dynamic subdomain parameters are passed to the route handler.
- Subdomain parameters can be used in `route()` generation.

**Constraints and Limitations**

- DNS configuration is required for production subdomains.
- Wildcard subdomains require wildcard DNS records.
- Subdomain routing is less performant than path-based routing due to DNS resolution.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Static Subdomains**

**Code**

```php
Route::domain('admin.myapp.com')->group(function () {
    Route::get('/', function () {
        return 'Admin Home';
    })->name('admin.home');

    Route::get('/users', function () {
        return 'Admin Users';
    })->name('admin.users');
});

Route::domain('api.myapp.com')->group(function () {
    Route::get('/v1/users', function () {
        return 'API v1 Users';
    })->name('api.users');
});
```

**Expected Output**

```
// https://admin.myapp.com/        → "Admin Home"
// https://admin.myapp.com/users   → "Admin Users"
// https://api.myapp.com/v1/users  → "API v1 Users"
```

**Why This Output Occurs**

Each group is bound to its respective subdomain. Laravel matches the `Host` header to determine which routes to use.

---

**Example 2: Dynamic Subdomain with Model Binding**

**Code**

```php
use App\Models\Tenant;

Route::domain('{tenant}.myapp.com')->group(function () {
    Route::get('/dashboard', function (Tenant $tenant) {
        return "Dashboard for {$tenant->name}";
    })->name('tenant.dashboard');
});
```

**Expected Output**

```
// https://acme.myapp.com/dashboard → "Dashboard for Acme Corp"
```

**Why This Output Occurs**

The `{tenant}` parameter is implicitly bound to the `Tenant` model, automatically resolving the tenant from the subdomain.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant SaaS**

A SaaS platform assigns each customer a subdomain (`acme.myapp.com`). A wildcard subdomain group captures the tenant and scopes all queries accordingly.

**Explanation**: Subdomain routing is the foundation of many multi-tenant architectures.

**Case 2: API Subdomain**

An application serves its API on `api.example.com` and its web interface on `www.example.com`, keeping concerns separate.

**Explanation**: Subdomain routing provides clean separation between different client types.

---

### 8. Route Group Best Practices

#### Definitions

**Core Definition**

Route group best practices are the recommended guidelines for defining, organizing, and maintaining route groups to maximize readability, maintainability, and performance.

**Technical Definition**

Best practices encompass attribute ordering, naming conventions, nesting strategies, middleware placement, domain vs. prefix selection, and compatibility with route caching.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their route files clean, organized, and easy to work with.

#### Key Best Practices

1. **Use Fluent Chainable Syntax**: Prefer `Route::prefix('admin')->name('admin.')->group(...)` over the array syntax for better readability and IDE support.

2. **Order Attributes Logically**: Chain attributes in a consistent order: `controller()` → `middleware()` → `prefix()` → `name()` → `domain()`.

3. **Include the Trailing Dot in Name Prefixes**: Always include the trailing dot (`.`) in name prefixes for proper separation (e.g., `name('admin.')`, not `name('admin')`).

4. **Keep Groups Focused**: Each group should serve a single logical purpose (admin section, API version, tenant). Avoid grouping unrelated routes together.

5. **Use Middleware Groups for Common Patterns**: Create middleware groups in `Kernel.php` or `bootstrap/app.php` for frequently used middleware combinations.

6. **Register Subdomain Routes Before Root Domain Routes**: This prevents root domain routes from overwriting subdomain routes with the same URI path.

7. **Prefer Explicit Controller Imports Over Namespace Groups**: In Laravel 11+, use `use App\Http\Controllers\Admin\DashboardController;` and `[DashboardController::class, 'index']` instead of namespace groups for better IDE support.

8. **Use `controller()` for Controller-Specific Groups**: When all routes in a group use the same controller, use `controller()` to reduce repetition.

9. **Avoid Deeply Nested Groups**: While nesting is supported, excessive nesting reduces readability. Consider splitting into separate route files for complex sections.

10. **Clear Route Cache After Changes**: Always run `php artisan route:clear` after modifying route groups during development.

11. **Use Named Routes with Prefixes**: Always assign names to routes within groups, using the group name prefix for consistency.

12. **Document Group Purposes**: Add comments above groups explaining their purpose, especially for complex domain or middleware configurations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Well-Structured Admin Group**

**Code**

```php
// routes/web.php
use App\Http\Controllers\Admin\DashboardController;
use App\Http\Controllers\Admin\UserController;
use App\Http\Controllers\Admin\ReportController;

// Admin section: requires auth + admin middleware
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        // Dashboard
        Route::get('/dashboard', [DashboardController::class, 'index'])
            ->name('dashboard');

        // User management
        Route::resource('users', UserController::class);

        // Reports
        Route::controller(ReportController::class)->prefix('reports')->group(function () {
            Route::get('/', 'index')->name('reports.index');
            Route::get('/sales', 'sales')->name('reports.sales');
        });
    });
```

**Expected Output**

```
// GET /admin/dashboard (auth + admin) → Dashboard
// GET /admin/users (auth + admin) → User index
// POST /admin/users (auth + admin) → User store
// GET /admin/reports (auth + admin) → Report index
// GET /admin/reports/sales (auth + admin) → Sales report
```

**Why This Output Occurs**

The outer group applies middleware, prefix, and name prefix. The inner controller group reduces repetition for report routes. All routes inherit the shared attributes.

---

**Example 2: API Version Group**

**Code**

```php
// routes/api.php
use App\Http\Controllers\Api\V1\UserController as V1UserController;
use App\Http\Controllers\Api\V2\UserController as V2UserController;

// API v1
Route::prefix('v1')->name('v1.')->middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::get('/users', [V1UserController::class, 'index'])->name('users.index');
    Route::get('/users/{user}', [V1UserController::class, 'show'])->name('users.show');
});

// API v2
Route::prefix('v2')->name('v2.')->middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::get('/users', [V2UserController::class, 'index'])->name('users.index');
    Route::get('/users/{user}', [V2UserController::class, 'show'])->name('users.show');
});
```

**Expected Output**

```
// GET /api/v1/users → V1 User index
// GET /api/v1/users/1 → V1 User show
// GET /api/v2/users → V2 User index
// GET /api/v2/users/1 → V2 User show
```

**Why This Output Occurs**

Each version group has its own prefix, name prefix, middleware, and controller. The API prefix (`/api`) is applied automatically by the `api.php` route file.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise Application with Multiple Sections**

An enterprise application uses separate route files for `admin.php`, `api.php`, and `web.php`, each with their own groups. The admin file uses a namespace group for `Admin` controllers, middleware for authentication, and a name prefix for organization.

**Explanation**: Well-organized route groups scale with application complexity.

**Case 2: API-First Application**

An API-first application defines version groups (`v1`, `v2`) with Sanctum authentication and rate limiting. Each version has its own controller namespace and route names.

**Explanation**: Group best practices enable clean API versioning and maintainability.

---

## References

- Routing – Laravel 13.x Documentation — https://laravel.com/docs/13.x/routing#route-groups
- Routing – Laravel 12.x Documentation — https://laravel.com/docs/12.x/routing#route-groups
- Routing – Laravel 11.x Documentation — https://laravel.com/docs/11.x/routing#route-groups
- Routing – Laravel 10.x Documentation — https://laravel.com/docs/10.x/routing#route-groups
- Route Group Subdomain Routing – Laravel 10.x Documentation — https://laravel.com/docs/10.x/routing#route-group-subdomain-routing
- Route Prefixes – Laravel Documentation (GitHub) — https://github.com/driade/laravel-book/blob/master/laravel-docs-6.x.pdf
- Route Name Prefixes – Laravel Documentation (GitHub) — https://github.com/DevStorm-Team/laravel-book/blob/master/laravel-docs-5.5.pdf
- Registering New Routes File – Laracasts Discussion — https://laracasts.com/discuss/channels/laravel/registering-new-routes-file
- Laravel 11 Set Default Namespace for Controllers – Stack Overflow — https://stackoverflow.com/questions/79454039/laravel-11-set-default-namespace-for-controllers
- Named Auth Routes with Multiple Subdomains – Laracasts Discussion — https://laracasts.com/discuss/channels/laravel/named-auth-routes-with-multiple-subdomains
- API Skill for Laravel Developers – GitHub (JustSteveKing) — https://github.com/JustSteveKing/api-skill/blob/HEAD/SKILL.md
- Route:get('user/profile') – Laravel Documentation (GitHub) — https://github.com/driade/laravel-book/blob/master/laravel-docs-6.x.pdf
- Laravel Middleware Chaining Techniques – Yisu — https://m.yisu.com/zixun/920773.html
- Explicit Domains for Route Groups – Stack Overflow — https://stackoverflow.com/revisions/79873917/1
- Laravel 11 Controller Routes – Stack Overflow — https://stackoverflow.com/questions/79348780/laravel-11-controller-routes
- Laravel Subdomain Setup – Laracasts Discussion — https://laracasts.com/discuss/channels/forge/laravel-subdomain-setup
- Route Group Best Practices – GitHub (codeApe-7) — https://github.com/codeApe-7/ai-agent-workflowGroup/blob/master/skills/php/laravel-specialist/references/routing.md
- Krayin CRM Developer Portal – Routes — https://devdocs.krayincrm.com/2.2/packages/routes.html

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Route groups have remained stable across these versions, but key differences exist: Laravel 11 removed the `RouteServiceProvider` and the default namespace wrapper for route files, making explicit controller imports the recommended approach. The `namespace()` method is still available but less commonly used. The `controller()` method was introduced in Laravel 9 and is the preferred way to bind a controller to a group. The fluent chainable syntax (`Route::prefix()->name()->group()`) is recommended over the array syntax. Developers should always consult the documentation for their specific Laravel version.