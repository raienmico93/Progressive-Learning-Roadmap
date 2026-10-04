# Laravel API Routing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel API Routing is the mechanism by which HTTP requests are mapped to application logic within a stateless API context, using the `routes/api.php` file, route groups, middleware, and rate limiters to define, organise, protect, and throttle API endpoints.

**Technical Definition:** Laravel API Routing is implemented through the `Illuminate\Routing\Router` class, which registers route definitions from `routes/api.php` (or versioned route files) and dispatches incoming HTTP requests to the appropriate controller actions or closures. API routes are distinguished from web routes by their assignment to the `api` middleware group, which omits `StartSession` and `VerifyCsrfToken` middleware, thereby enforcing statelessness. Route prefixes, middleware assignment, versioning, and rate limiting are all configured either within the route file itself (using `Route::prefix()`, `Route::middleware()`, and `Route::group()`), in the application bootstrap file (`bootstrap/app.php` in Laravel 11+), or in service providers via the `RateLimiter` facade.

**Beginner-Friendly Explanation:** When you build an API, you need a way to tell Laravel "when someone visits this URL with this HTTP method, run this code." That is what routing does. Laravel gives you a special file called `routes/api.php` where you write these rules. Because APIs are meant to be stateless — each request stands alone, with no memory of previous requests — Laravel treats API routes differently from web routes. It does not start sessions or check CSRF tokens for API routes. You can organise routes into groups (like putting all "admin" routes under `/api/admin`), protect them with authentication, and limit how many requests a user can make.

### Key Characteristics

- **Stateless by default:** API routes do not start sessions, set cookies, or verify CSRF tokens.
- **Automatic `/api` prefix:** Routes in `routes/api.php` are automatically prefixed with `/api`.
- **Dedicated middleware group:** The `api` middleware group is separate from the `web` group and can be customised.
- **Group-based organisation:** Routes can be grouped by prefix, middleware, name, domain, and controller namespace.
- **Versioning support:** Multiple API versions can coexist through URL prefixes or separate route files.
- **Authentication agnosticism:** API routes can use token-based authentication (Sanctum, Passport) instead of session-based authentication.
- **Built-in rate limiting:** Laravel provides the `throttle` middleware and `RateLimiter` facade for request throttling.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+); PHP 8.2+ for Laravel 11+.
- Composer dependency manager.
- A Laravel application with `routes/api.php` (created via `php artisan install:api` in Laravel 11+).
- Basic understanding of HTTP methods, status codes, and middleware.
- (For authentication) Laravel Sanctum or Laravel Passport installed.
- (For rate limiting with Redis) A Redis server configured and the `predis/predis` package or `phpredis` extension installed.

### Related Programming Areas

- **HTTP Protocol** — The transport layer for API requests and responses.
- **Middleware** — The mechanism for filtering and modifying HTTP requests.
- **Service Providers** — Where rate limiters and custom middleware are typically registered.
- **Eloquent ORM** — Route model binding resolves Eloquent models from route parameters.
- **Laravel Sanctum** — Token-based authentication for APIs and SPAs.
- **Laravel Passport** — Full OAuth2 server implementation for API authentication.
- **Rate Limiting** — The `Illuminate\Cache\RateLimiting\Limit` class and `RateLimiter` facade.

### Core Concepts / Features

1. API Routes: `routes/api.php`, stateless behaviour, and differences from web routes.
2. Route Prefixes & Scoping: Grouping routes by features, modules, or resource owners.
3. Versioning: URI versioning, header versioning, and route file separation.
4. Authentication Middleware: Protecting routes using `auth:sanctum` or `auth:api`.
5. Rate Limiting: Custom rate limiters via the `RateLimiter` facade, dynamic throttling, and per-user/IP restrictions.

---

## 1. API Routes

### Definitions

**Core Definition:** API routes are route definitions registered in the `routes/api.php` file that are assigned to the `api` middleware group and automatically prefixed with `/api`, designed for stateless HTTP communication.

**Technical Definition:** The `routes/api.php` file is loaded by Laravel's routing bootstrap process and wrapped in a route group that applies the `api` middleware group and the `/api` URI prefix. In Laravel 11 and later, this behaviour is configured in `bootstrap/app.php` via the `withRouting()` method. The `api` middleware group, by default, includes `Illuminate\Routing\Middleware\SubstituteBindings` and, when Sanctum is installed, `Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful` (for SPA authentication) and `Illuminate\Routing\Middleware\ThrottleRequests`. It does **not** include `StartSession`, `ShareErrorsFromSession`, `VerifyCsrfToken`, or `SubstituteBindings` from the web group.

**Beginner-Friendly Explanation:** Think of `routes/web.php` as the routes for your website — pages that users visit in a browser, where the server needs to remember who they are (sessions) and protect against form hijacking (CSRF). `routes/api.php` is for your API — endpoints that other programs call, where every request is independent and self-contained. Laravel automatically handles the differences: API routes get the `/api` prefix and skip session/CSRF middleware, making them truly stateless.

### Purposes

- To separate API routing logic from web routing logic, enforcing statelessness.
- To apply the `/api` prefix automatically to all API endpoints without manual repetition.
- To assign a dedicated middleware group that can be customised independently of the web group.
- To enable token-based authentication (Sanctum, Passport) without session interference.
- To provide a clean, conventional location for API route definitions that is separate from browser-facing routes.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel 11+)

```php
// bootstrap/app.php — Configuring API routing
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        apiPrefix: 'api', // Default; change to customise
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Customise the api middleware group here
    })
    ->create();
```

```php
// routes/api.php — Example API routes
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PostController;

// Public route — no authentication required
Route::get('/health', function () {
    return response()->json(['status' => 'ok']);
});

// Protected route — requires a valid Sanctum token
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

// Resource routes
Route::apiResource('posts', PostController::class);
```

**Component Breakdown:**

- `withRouting(api: ...)` — Registers the API route file. In Laravel 11+, this replaces the `RouteServiceProvider` approach used in earlier versions.
- `apiPrefix: 'api'` — The URI prefix applied to all routes in the API file. Change this to `'api/admin'` or another value to customise.
- `routes/api.php` — The default location for API route definitions.
- `Route::middleware('auth:sanctum')` — Applies the Sanctum authentication guard to the route.

**Syntax Rules:**

- API routes **must** be defined in `routes/api.php` (or a file registered with the `api:` parameter in `withRouting()`).
- The `/api` prefix is applied automatically; do not add it manually to individual routes.
- The `api` middleware group is applied automatically; additional middleware can be added via `Route::middleware()`.
- In Laravel 10 and earlier, API routes are registered in `RouteServiceProvider::boot()` using `Route::middleware('api')->prefix('api')->group(base_path('routes/api.php'))`.

**Constraints and Limitations:**

- **Statelessness is broken if session middleware is added to the API group.** Adding `StartSession` to the `api` group reintroduces state and may cause unexpected behaviour.
- **The `api` middleware group is empty by default in Laravel 11+** unless `install:api` has been run. Without it, API routes receive no middleware (including no throttle or bindings).
- **Route caching (`php artisan route:cache`) does not support closure-based routes** in any route file, including `routes/api.php`. Use controller actions instead.
- **API routes are not accessible via the browser without the `Accept: application/json` header** when using Sanctum SPA authentication; without it, Sanctum may attempt a redirect to a login page.

### Annotated Code Examples

**Example 1: Setting Up API Routing in Laravel 11+**

```bash
# Step 1: Run the install:api Artisan command
php artisan install:api
```

**Expected Output:**

```
INFO  Installing API routes...

   LARAVEL\SANCTUM ........................................ DONE
   API routes file ........................................ DONE
   Personal access tokens migration ....................... DONE

INFO  API routes installed successfully.
```

```php
<?php
// File: routes/api.php — Auto-generated by install:api

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

// Step 2: A protected route that returns the authenticated user.
// The auth:sanctum middleware ensures only requests with a valid
// Bearer token can access this endpoint.
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

```php
<?php
// File: bootstrap/app.php — Verify API routing configuration

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php', // API routes registered
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // The 'api' middleware group can be customised here.
        // By default, install:api adds Sanctum's middleware.
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

**Step-by-Step Setup:**

1. Run `php artisan install:api` to install Sanctum, create `routes/api.php`, and publish the personal access tokens migration.
2. Run `php artisan migrate` to create the `personal_access_tokens` table.
3. Add the `HasApiTokens` trait to your `User` model.
4. Define API routes in `routes/api.php` as shown.
5. Test with `php artisan route:list` to verify the routes are registered with the `/api` prefix.

**Expected Output:**

- `GET /api/user` without a token returns `401 Unauthorized`.
- `GET /api/user` with a valid `Authorization: Bearer <token>` header returns the user JSON.
- `php artisan route:list` shows all API routes prefixed with `/api`.

**Why This Output Occurs:** The `install:api` command registers the `routes/api.php` file in the application bootstrap process and configures Sanctum as the default API authentication guard. The `api` middleware group is populated with Sanctum's middleware, which validates Bearer tokens. The `/api` prefix is applied automatically. The `auth:sanctum` middleware rejects unauthenticated requests with a 401 response.

---

**Example 2: Differences Between Web and API Routes**

```php
<?php
// File: routes/web.php — Web routes (stateful)

use Illuminate\Support\Facades\Route;

// This route has session state and CSRF protection.
// Visiting it in a browser starts a session.
Route::get('/dashboard', function () {
    // Session is available
    session(['last_visit' => now()]);
    return view('dashboard');
});

// POST routes in web.php require a CSRF token.
Route::post('/profile', function () {
    // CSRF token was verified by middleware
    return 'Profile updated';
});
```

```php
<?php
// File: routes/api.php — API routes (stateless)

use Illuminate\Support\Facades\Route;

// This route has NO session state and NO CSRF protection.
// It is accessed via /api/status, not /status.
Route::get('/status', function () {
    // Session is NOT available — no StartSession middleware
    // CSRF token is NOT required — no VerifyCsrfToken middleware
    return response()->json(['status' => 'operational']);
});
```

**Expected Output:**

- `GET /dashboard` (web) starts a session and returns the dashboard view.
- `POST /profile` (web) requires a CSRF token; without it, returns `419 Page Expired`.
- `GET /api/status` (API) returns `{"status":"operational"}` with no session cookie set.
- `POST /api/profile` (if defined) does **not** require a CSRF token.

**Why This Output Occurs:** The `web` middleware group includes `StartSession`, `ShareErrorsFromSession`, `VerifyCsrfToken`, and `SubstituteBindings`. The `api` middleware group does **not** include these by default. This is the fundamental difference: web routes are stateful and browser-oriented; API routes are stateless and programmatic. The `/api` prefix is automatically applied to all routes in `routes/api.php`, so the actual URI is `/api/status`, not `/status`.

### Real-World Cases

- **Mobile application backends:** A Laravel API serves JSON to iOS and Android apps via `/api/` endpoints, with token-based authentication (Sanctum) instead of session cookies.
- **Single-page applications (SPAs):** A Vue or React frontend consumes a Laravel API hosted on the same or a different domain, using `auth:sanctum` with SPA stateful authentication.
- **Third-party integrations:** External partners consume your API using Bearer tokens, with no session or CSRF concerns.
- **Microservices architecture:** Internal services communicate via stateless HTTP API calls, each request carrying its own authentication token and data.

---

## 2. Route Prefixes & Scoping

### Definitions

**Core Definition:** Route prefixes and scoping are techniques for grouping related routes under a common URI segment, middleware set, name prefix, or domain, reducing repetition and organising routes by feature, module, or resource owner.

**Technical Definition:** Route groups in Laravel allow shared route attributes — prefix, middleware, name prefix, domain, controller namespace, and `where` constraints — to be defined once for a group of routes. The `Route::prefix()` method prepends a URI segment to all routes in the group; `Route::name()` prepends a string to all route names; `Route::middleware()` applies middleware to all routes in the group; and `Route::domain()` restricts the group to a specific domain. These attributes can be nested for multi-level organisation.

**Beginner-Friendly Explanation:** Imagine you have twenty routes that all start with `/admin` and all require an "admin" middleware. Instead of writing `/admin` and the middleware on every single route, you can wrap them in a group that says "all of these routes start with `/admin` and require admin access." This is route grouping. Prefixes let you organise URLs (like `/api/v1/users`), middleware scoping lets you protect groups of routes at once, and name prefixes let you refer to routes consistently (like `admin.users.index`).

### Purposes

- To eliminate repetition by applying a common URI prefix to multiple routes at once.
- To apply middleware to a group of routes without defining it on each route individually.
- To organise routes by feature, module, or domain area (e.g., admin, API versions, account-specific).
- To create consistent route naming conventions across a group using name prefixes.
- To restrict a group of routes to a specific domain or subdomain.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\Admin\UserController;
use App\Http\Controllers\Api\Admin\ReportController;

// Basic prefix grouping
Route::prefix('admin')->group(function () {
    Route::get('/users', [UserController::class, 'index']);
    Route::get('/reports', [ReportController::class, 'index']);
});
// Generates: /api/admin/users and /api/admin/reports

// Prefix with name prefix and middleware
Route::prefix('admin')
    ->name('admin.')
    ->middleware(['auth:sanctum', 'admin'])
    ->group(function () {
        Route::get('/users', [UserController::class, 'index'])->name('users.index');
        Route::get('/reports', [ReportController::class, 'index'])->name('reports.index');
    });
// Route names: admin.users.index, admin.reports.index

// Nested groups
Route::prefix('v1')->group(function () {
    Route::prefix('admin')->middleware('auth:sanctum')->group(function () {
        Route::apiResource('users', UserController::class);
    });
});
// Generates: /api/v1/admin/users
```

**Component Breakdown:**

- `Route::prefix('admin')` — Prepends `admin` to the URI of every route in the group. Combined with the automatic `/api` prefix, the final URI is `/api/admin/...`.
- `Route::name('admin.')` — Prepends `admin.` to the name of every route in the group. A route named `users.index` becomes `admin.users.index`.
- `Route::middleware(['auth:sanctum', 'admin'])` — Applies both middleware to every route in the group.
- `Route::group(function () { ... })` — Defines the group closure containing the routes.
- Nested groups — Groups can be nested to any depth; prefixes and names are concatenated.

**Syntax Rules:**

- The order of chained methods matters: `prefix()`, `name()`, `middleware()`, and `domain()` can be chained in any order, but all must be called before `group()`.
- Nested groups concatenate prefixes: `prefix('v1')->group(fn() => Route::prefix('admin')->group(...))` produces `/api/v1/admin/...`.
- Name prefixes are also concatenated: `name('v1.')->group(fn() => Route::name('admin.')->group(...))` produces `v1.admin.` as the name prefix.
- Middleware defined in nested groups is merged; a route in a nested group inherits all middleware from all enclosing groups.

**Constraints and Limitations:**

- **Route name collisions:** If two groups use the same name prefix and define a route with the same name, the later definition overwrites the earlier one. Always use unique name prefixes.
- **Middleware order matters:** Middleware is executed in the order it is defined. If a group inherits middleware from a parent group, the parent's middleware runs first.
- **Over-nesting can reduce readability.** Deeply nested groups are harder to reason about; consider flattening where possible.
- **Route caching flattens all groups.** After `php artisan route:cache`, all routes are stored as flat definitions; group structure is not preserved in the cache.

### Annotated Code Examples

**Example 1: Grouping Routes by Feature and Access Level**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PublicController;
use App\Http\Controllers\Api\User\ProfileController;
use App\Http\Controllers\Api\User\OrderController;
use App\Http\Controllers\Api\Admin\UserController;
use App\Http\Controllers\Api\Admin\ReportController;

// Public routes — no authentication
Route::get('/health', [PublicController::class, 'health']);
Route::get('/version', [PublicController::class, 'version']);

// Authenticated user routes
Route::middleware('auth:sanctum')->group(function () {
    // Profile routes: /api/profile, /api/profile/avatar
    Route::prefix('profile')->name('profile.')->group(function () {
        Route::get('/', [ProfileController::class, 'show'])->name('show');
        Route::put('/', [ProfileController::class, 'update'])->name('update');
        Route::post('/avatar', [ProfileController::class, 'uploadAvatar'])->name('avatar');
    });

    // Order routes: /api/orders, /api/orders/{order}
    Route::apiResource('orders', OrderController::class);
});

// Admin routes — require authentication AND admin middleware
Route::middleware(['auth:sanctum', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        // /api/admin/users
        Route::apiResource('users', UserController::class)->names([
            'index'   => 'admin.users.index',
            'store'   => 'admin.users.store',
            'show'    => 'admin.users.show',
            'update'  => 'admin.users.update',
            'destroy' => 'admin.users.destroy',
        ]);

        // /api/admin/reports
        Route::prefix('reports')->name('reports.')->group(function () {
            Route::get('/', [ReportController::class, 'index'])->name('index');
            Route::get('/sales', [ReportController::class, 'sales'])->name('sales');
            Route::get('/users', [ReportController::class, 'users'])->name('users');
        });
    });
```

**Step-by-Step Setup:**

1. Create the controllers: `PublicController`, `ProfileController`, `OrderController`, `UserController`, `ReportController`.
2. Register the `admin` middleware alias in `bootstrap/app.php` (Laravel 11+) or `app/Http/Kernel.php` (Laravel 10-).
3. Define the routes as shown.
4. Run `php artisan route:list` to verify the generated routes.

**Expected Output:**

```
GET|HEAD  api/health ............................ PublicController@health
GET|HEAD  api/version ........................... PublicController@version
GET|HEAD  api/profile ........................... ProfileController@show
PUT       api/profile ........................... ProfileController@update
POST      api/profile/avatar .................... ProfileController@uploadAvatar
GET|HEAD  api/orders ............................ OrderController@index
POST      api/orders ............................ OrderController@store
GET|HEAD  api/orders/{order} .................... OrderController@show
PUT|PATCH api/orders/{order} .................... OrderController@update
DELETE    api/orders/{order} .................... OrderController@destroy
GET|HEAD  api/admin/users ....................... Admin\UserController@index
POST      api/admin/users ....................... Admin\UserController@store
...
GET|HEAD  api/admin/reports ..................... Admin\ReportController@index
GET|HEAD  api/admin/reports/sales ............... Admin\ReportController@sales
GET|HEAD  api/admin/reports/users ............... Admin\ReportController@users
```

**Why This Output Occurs:** The `prefix('profile')` group prepends `profile` to the URI, producing `/api/profile`. The `prefix('admin')` group prepends `admin`, producing `/api/admin/users`. The `middleware('auth:sanctum')` group ensures only authenticated users can access profile and order routes. The `middleware(['auth:sanctum', 'admin'])` group adds both authentication and the custom admin check. Name prefixes (`profile.`, `admin.`, `reports.`) create consistent route names.

---

**Example 2: Resource Owner Scoping with Nested Prefixes**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\Account\InvoiceController;
use App\Http\Controllers\Api\Account\PaymentController;

// Routes scoped to a specific account (resource owner)
// URI pattern: /api/accounts/{account}/invoices
Route::prefix('accounts/{account}')
    ->middleware('auth:sanctum')
    ->name('accounts.')
    ->group(function () {
        // Invoices: /api/accounts/{account}/invoices
        Route::apiResource('invoices', InvoiceController::class);

        // Payments: /api/accounts/{account}/payments
        Route::apiResource('payments', PaymentController::class);

        // Nested: /api/accounts/{account}/invoices/{invoice}/payments
        Route::apiResource('invoices.payments', PaymentController::class)->shallow();
    });
```

**Expected Output:**

- `GET /api/accounts/1/invoices` → Lists invoices for account 1.
- `POST /api/accounts/1/payments` → Creates a payment for account 1.
- `GET /api/invoices/5/payments` → Lists payments for invoice 5 (shallow route).
- Accessing an account the user does not own returns `403 Forbidden` (if ownership middleware is applied).

**Why This Output Occurs:** The `prefix('accounts/{account}')` group makes the account ID a required route parameter for all routes within it. Route model binding resolves `{account}` to an `Account` model. Combined with a custom ownership middleware or policy, the application can verify that the authenticated user has access to the specified account. The `shallow()` method on the nested payments resource generates nested `index` and `store` routes but flat `show`, `update`, and `destroy` routes.

### Real-World Cases

- **Multi-tenant SaaS platforms:** Routes scoped to `/api/accounts/{account}/...` ensure that all operations are performed within the context of a specific tenant.
- **Admin panels:** A `/api/admin/` prefix groups all administrative endpoints under a single URI segment and applies admin middleware once.
- **API versioning:** A `/api/v1/` prefix groups all version 1 routes, and `/api/v2/` groups version 2 routes.
- **Modular applications:** Different feature modules (billing, reporting, user management) can each have their own prefix and middleware stack.

---

## 3. Versioning

### Definitions

**Core Definition:** API versioning is the practice of maintaining multiple versions of an API simultaneously, allowing clients to continue using older versions while newer versions are introduced with breaking changes.

**Technical Definition:** In Laravel, API versioning is typically implemented through URI path versioning (e.g., `/api/v1/`, `/api/v2/`) or header versioning (e.g., `Accept: application/vnd.api.v1+json` or `X-API-Version: 1`). URI versioning is implemented using route prefixes and separate route files per version. Header versioning requires custom middleware to read the version from the request headers and dispatch to the appropriate controller. Both approaches typically involve organising controllers, resources, and route files into version-specific directories (e.g., `App\Http\Controllers\Api\V1`).

**Beginner-Friendly Explanation:** When you make changes to your API that might break existing clients — like renaming a field, changing a data type, or removing an endpoint — you cannot simply overwrite the old API. Instead, you create a new version. Clients that depend on the old version keep using `/api/v1/`, while new clients use `/api/v2/`. This way, nobody's application breaks. Laravel makes this easy by letting you put each version's routes in a separate file and prefix them accordingly.

### Purposes

- To maintain backward compatibility for existing API clients while introducing breaking changes.
- To allow gradual migration of clients from older versions to newer versions.
- To separate concerns between different API versions, keeping code organised and maintainable.
- To provide a clear, predictable URI structure that indicates the API version.
- To support deprecation policies where older versions are eventually retired.

### Syntax Rules and Structure

#### Complete General Syntax (URI Versioning with Separate Files)

```php
// Step 1: Create version-specific route files
// routes/api_v1.php
// routes/api_v2.php

// Step 2: Register versioned routes in routes/api.php (Laravel 11+)
Route::prefix('v1')->group(base_path('routes/api_v1.php'));
Route::prefix('v2')->group(base_path('routes/api_v2.php'));

// Step 3: Define routes in each version file
// routes/api_v1.php
use App\Http\Controllers\Api\V1\PostController;
Route::apiResource('posts', PostController::class);

// routes/api_v2.php
use App\Http\Controllers\Api\V2\PostController;
Route::apiResource('posts', PostController::class);
```

**Component Breakdown:**

- `Route::prefix('v1')->group(base_path('routes/api_v1.php'))` — Loads the `api_v1.php` file within a group that prepends `v1` to all routes. Combined with the automatic `/api` prefix, the final URI is `/api/v1/...`.
- `routes/api_v1.php` — Contains only the routes for version 1. Controllers are namespaced under `Api\V1`.
- `App\Http\Controllers\Api\V1\PostController` — Version-specific controller. The V2 controller can have different logic, validation, or resource transformation.

#### Complete General Syntax (Header Versioning)

```php
// Step 1: Create a middleware that reads the version header
// File: app/Http/Middleware/ApiVersionMiddleware.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ApiVersionMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        $version = $request->header('X-API-Version', '1');

        // Store the version on the request for downstream use
        $request->attributes->set('api_version', $version);

        // Optionally, reject unsupported versions
        if (!in_array($version, ['1', '2'])) {
            return response()->json([
                'error' => 'Unsupported API version.',
            ], 400);
        }

        return $next($request);
    }
}

// Step 2: Register the middleware in the api group
// bootstrap/app.php (Laravel 11+)
->withMiddleware(function (Middleware $middleware) {
    $middleware->api(prepend: [
        \App\Http\Middleware\ApiVersionMiddleware::class,
    ]);
})

// Step 3: Use the version in the controller
$version = $request->attributes->get('api_version');
```

**Syntax Rules:**

- **URI versioning** is implemented via `Route::prefix('v1')` and separate route files. The prefix is concatenated with the automatic `/api` prefix.
- **Header versioning** requires a custom middleware that reads the version header (e.g., `X-API-Version` or `Accept`) and either stores it on the request or routes to different controllers.
- **Route file separation** is the recommended approach for URI versioning: each version has its own route file, and `routes/api.php` includes them via prefix groups.
- **Controller namespacing** should follow the version: `App\Http\Controllers\Api\V1\...`, `App\Http\Controllers\Api\V2\...`.

**Constraints and Limitations:**

- **Header versioning is less discoverable** than URI versioning. Clients must know to send a header, and the API documentation must specify which header to use.
- **URI versioning pollutes the URL space** but is more explicit and easier to debug. It is the most common approach in practice.
- **Versioned route files must be registered manually.** Laravel does not auto-discover `api_v1.php`; it must be loaded via `Route::prefix('v1')->group(base_path('routes/api_v1.php'))`.
- **Maintaining multiple versions increases code duplication.** Consider using shared traits or base controllers to reduce duplication between versions.

### Annotated Code Examples

**Example 1: URI Versioning with Separate Route Files (Laravel 11+)**

```php
<?php
// File: routes/api.php — Main API route file

use Illuminate\Support\Facades\Route;

// Step 1: The default /api routes (e.g., Sanctum user endpoint)
Route::get('/user', function (\Illuminate\Http\Request $request) {
    return $request->user();
})->middleware('auth:sanctum');

// Step 2: Version 1 routes — /api/v1/...
Route::prefix('v1')->group(base_path('routes/api_v1.php'));

// Step 3: Version 2 routes — /api/v2/...
Route::prefix('v2')->group(base_path('routes/api_v2.php'));
```

```php
<?php
// File: routes/api_v1.php — Version 1 routes

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\V1\PostController;

// /api/v1/posts
Route::apiResource('posts', PostController::class);
```

```php
<?php
// File: routes/api_v2.php — Version 2 routes

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\V2\PostController;

// /api/v2/posts — V2 controller returns a different structure
Route::apiResource('posts', PostController::class);
```

```php
<?php
// File: app/Http/Controllers/Api/V1/PostController.php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\JsonResponse;

class PostController extends Controller
{
    public function index(): JsonResponse
    {
        // V1 returns a flat array of posts
        return response()->json(Post::all());
    }

    public function show(Post $post): JsonResponse
    {
        // V1 returns the post with a simple structure
        return response()->json([
            'id'    => $post->id,
            'title' => $post->title,
            'body'  => $post->body,
        ]);
    }
}
```

```php
<?php
// File: app/Http/Controllers/Api/V2/PostController.php

namespace App\Http\Controllers\Api\V2;

use App\Http\Controllers\Controller;
use App\Http\Resources\V2\PostResource;
use App\Models\Post;
use Illuminate\Http\JsonResponse;

class PostController extends Controller
{
    public function index(): JsonResponse
    {
        // V2 returns posts wrapped in a resource collection with metadata
        return response()->json(PostResource::collection(Post::paginate(15)));
    }

    public function show(Post $post): JsonResponse
    {
        // V2 returns the post with nested author data and links
        return response()->json(new PostResource($post->load('user')));
    }
}
```

**Step-by-Step Setup:**

1. Create the version-specific route files: `touch routes/api_v1.php routes/api_v2.php`.
2. Create version-specific controllers: `php artisan make:controller Api/V1/PostController --api` and `php artisan make:controller Api/V2/PostController --api`.
3. Register the versioned route files in `routes/api.php` as shown.
4. Run `php artisan route:list` to verify the versioned routes.

**Expected Output:**

```
GET|HEAD  api/v1/posts ......................... Api\V1\PostController@index
POST      api/v1/posts ......................... Api\V1\PostController@store
GET|HEAD  api/v1/posts/{post} .................. Api\V1\PostController@show
...
GET|HEAD  api/v2/posts ......................... Api\V2\PostController@index
POST      api/v2/posts ......................... Api\V2\PostController@store
GET|HEAD  api/v2/posts/{post} .................. Api\V2\PostController@show
...
```

**Why This Output Occurs:** The `Route::prefix('v1')->group(base_path('routes/api_v1.php'))` call loads the version 1 route file within a group that prepends `v1`. Because this is inside `routes/api.php`, the automatic `/api` prefix is also applied, resulting in `/api/v1/posts`. The same applies to version 2. The controllers are namespaced by version, allowing completely different logic and response structures.

---

**Example 2: Header-Based Versioning with Middleware**

```php
<?php
// File: app/Http/Middleware/ApiVersionMiddleware.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ApiVersionMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Step 1: Read the version from the X-API-Version header,
        // defaulting to '1' if not provided.
        $version = $request->header('X-API-Version', '1');

        // Step 2: Validate the requested version.
        if (!in_array($version, ['1', '2'])) {
            return response()->json([
                'error'   => 'Unsupported API version.',
                'supported_versions' => ['1', '2'],
            ], 400);
        }

        // Step 3: Store the version on the request for downstream use.
        $request->attributes->set('api_version', $version);

        // Step 4: Add the version to the response headers for transparency.
        $response = $next($request);
        $response->headers->set('X-API-Version', $version);

        return $response;
    }
}
```

```php
<?php
// File: app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function index(Request $request): JsonResponse
    {
        $version = $request->attributes->get('api_version', '1');

        if ($version === '1') {
            // Version 1: flat structure
            return response()->json(Post::all());
        }

        // Version 2: wrapped structure with metadata
        return response()->json([
            'data' => Post::all(),
            'meta' => ['version' => '2'],
        ]);
    }
}
```

```php
// Step 5: Register the middleware in bootstrap/app.php (Laravel 11+)

->withMiddleware(function (Middleware $middleware) {
    $middleware->api(prepend: [
        \App\Http\Middleware\ApiVersionMiddleware::class,
    ]);
})
```

**Expected Output:**

- Request without `X-API-Version` header: returns V1 structure `[{"id":1,...}]`.
- Request with `X-API-Version: 2`: returns V2 structure `{"data":[...],"meta":{"version":"2"}}`.
- Request with `X-API-Version: 3`: returns `400 Bad Request` with `{"error":"Unsupported API version.","supported_versions":["1","2"]}`.
- All responses include the `X-API-Version` header reflecting the version used.

**Why This Output Occurs:** The middleware reads the `X-API-Version` header (or defaults to `1`), validates it against the supported versions, and stores it on the request attributes. The controller retrieves the version from the request attributes and returns the appropriate response structure. The middleware adds the version to the response headers, allowing clients to confirm which version was used. This approach keeps the URL clean while still supporting multiple versions.

### Real-World Cases

- **Public APIs with external consumers:** Stripe, GitHub, and Twilio use URI versioning (e.g., `/v1/`, `/v2/`) to maintain backward compatibility for years.
- **Internal microservices:** Header-based versioning can be used when the URL space is managed by an API gateway and version information is conveyed through headers.
- **Mobile app backends:** Older versions of a mobile app may continue to use `/api/v1/` while newer app versions use `/api/v2/`, with the server supporting both.
- **Deprecation workflows:** A version can be marked as deprecated (via response headers or documentation), giving clients time to migrate before the version is retired.

---

## 4. Authentication Middleware

### Definitions

**Core Definition:** Authentication middleware in Laravel API routing is the mechanism that verifies the identity of the client making a request before allowing access to protected routes, typically using token-based authentication guards such as Sanctum or Passport.

**Technical Definition:** Laravel's authentication middleware (`auth`) accepts a guard name as a parameter, such as `auth:sanctum` or `auth:api`. The middleware resolves the specified guard from the authentication manager and calls its `authenticate()` method. If authentication fails, the middleware throws an `AuthenticationException`, which Laravel converts to a `401 Unauthorized` JSON response for API requests. Sanctum's guard checks for a valid Bearer token in the `Authorization` header (for token-based authentication) or a stateful session cookie (for SPA authentication). Passport's `api` guard validates OAuth2 access tokens.

**Beginner-Friendly Explanation:** When you have API endpoints that should only be accessible to logged-in users, you need a way to verify who is making the request. Authentication middleware does this: it checks for a valid token in the request headers (like a digital key card). If the token is valid, the request proceeds; if not, the server responds with "401 Unauthorized." Laravel makes this easy by letting you attach `auth:sanctum` to any route or group of routes.

### Purposes

- To verify the identity of clients accessing protected API endpoints.
- To restrict access to sensitive data and operations to authenticated users only.
- To support multiple authentication mechanisms (token-based, session-based, OAuth2) through guards.
- To integrate with Laravel's authorization system (Gates and Policies) after authentication succeeds.
- To provide a stateless authentication mechanism suitable for APIs, mobile apps, and SPAs.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Applying auth middleware to a single route
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');

// Applying auth middleware to a group of routes
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::put('/profile', [ProfileController::class, 'update']);
});

// Using auth:api (Passport) instead of auth:sanctum
Route::middleware('auth:api')->group(function () {
    Route::apiResource('posts', PostController::class);
});

// Checking the authenticated user in a controller
$user = $request->user();
$userId = $request->user()->id;

// Checking a specific ability (Sanctum)
if ($request->user()->tokenCan('post:create')) {
    // Allow the action
}
```

**Component Breakdown:**

- `->middleware('auth:sanctum')` — Applies the Sanctum guard. The middleware checks for a valid Bearer token or SPA session.
- `->middleware('auth:api')` — Applies the Passport guard. The middleware checks for a valid OAuth2 access token.
- `$request->user()` — Returns the authenticated user instance, or `null` if not authenticated.
- `$request->user()->tokenCan('ability')` — Checks whether the current access token has the specified ability (Sanctum-specific).

**Syntax Rules:**

- The `auth` middleware requires a guard name after the colon: `auth:sanctum`, `auth:api`, `auth:web`. Without a guard name, it uses the default guard defined in `config/auth.php`.
- Multiple guards can be specified: `auth:sanctum,api` — the middleware will try each guard in order.
- The `auth:sanctum` middleware must be applied to routes in `routes/api.php`; it does not work in `routes/web.php` for token-based authentication (though Sanctum SPA authentication can work in web routes).
- Sanctum's `HasApiTokens` trait must be added to the `User` model for token creation and `tokenCan()` to work.

**Constraints and Limitations:**

- **`auth:api` (Passport) is heavier than `auth:sanctum`.** Passport implements a full OAuth2 server, which is overkill for simple token-based APIs. Sanctum is recommended for most applications.
- **Sanctum SPA authentication requires the SPA and API to share the same top-level domain** (or `SANCTUM_STATEFUL_DOMAINS` must be configured). Cross-domain SPA authentication with cookies requires additional configuration.
- **Token abilities are not roles.** Sanctum's `tokenCan()` checks abilities assigned to a token, not the user's role. For role-based access control, use Gates and Policies.
- **`auth:sanctum` returns 401 for unauthenticated requests** but may redirect to a login page for browser requests that do not send the `Accept: application/json` header. Always send this header from API clients.

### Annotated Code Examples

**Example 1: Protecting Routes with Sanctum Authentication**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PostController;
use App\Http\Controllers\Api\ProfileController;

// Step 1: Public routes — no authentication required
Route::post('/register', [\App\Http\Controllers\Api\AuthController::class, 'register']);
Route::post('/login', [\App\Http\Controllers\Api\AuthController::class, 'login']);

// Step 2: Protected routes — require a valid Sanctum token
Route::middleware('auth:sanctum')->group(function () {
    // User profile
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::put('/profile', [ProfileController::class, 'update']);

    // Post management
    Route::apiResource('posts', PostController::class);

    // Logout — revokes the current access token
    Route::post('/logout', [\App\Http\Controllers\Api\AuthController::class, 'logout']);
});
```

```php
<?php
// File: app/Http/Controllers/Api/AuthController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function register(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users',
            'password' => 'required|string|min:8|confirmed',
        ]);

        $user = User::create([
            'name'     => $validated['name'],
            'email'    => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'user'  => $user,
            'token' => $token,
        ], 201);
    }

    public function login(Request $request): JsonResponse
    {
        $request->validate([
            'email'    => 'required|email',
            'password' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'user'  => $user,
            'token' => $token,
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message' => 'Logged out successfully.']);
    }
}
```

**Step-by-Step Setup:**

1. Run `php artisan install:api` to install Sanctum.
2. Add the `HasApiTokens` trait to the `User` model:
   ```php
   use Laravel\Sanctum\HasApiTokens;
   class User extends Authenticatable {
       use HasApiTokens;
   }
   ```
3. Create the `AuthController` with the methods shown.
4. Define the routes in `routes/api.php`.
5. Run `php artisan migrate` to create the `personal_access_tokens` table.

**Expected Output:**

- `POST /api/register` with valid data → `201 Created` with `{"user": {...}, "token": "1|abc123..."}`.
- `POST /api/login` with valid credentials → `200 OK` with a new token.
- `GET /api/profile` without a token → `401 Unauthorized`.
- `GET /api/profile` with `Authorization: Bearer 1|abc123...` → `200 OK` with the user's profile.
- `POST /api/logout` with a token → `200 OK` and the token is revoked.

**Why This Output Occurs:** The `auth:sanctum` middleware intercepts requests to protected routes and checks for a valid Bearer token in the `Authorization` header. If the token is missing or invalid, the middleware throws an `AuthenticationException`, which Laravel converts to a 401 JSON response for API requests. If the token is valid, the middleware resolves the associated user and makes it available via `$request->user()`. The `createToken()` method generates a new personal access token and stores its hashed value in the `personal_access_tokens` table.

---

**Example 2: Using Passport's `auth:api` Guard for OAuth2**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\OrderController;

// Passport's 'api' guard validates OAuth2 access tokens.
Route::middleware('auth:api')->group(function () {
    Route::apiResource('orders', OrderController::class);
});
```

```php
<?php
// File: config/auth.php — Passport guard configuration

'guards' => [
    'web' => [
        'driver'   => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver'   => 'passport', // Uses Passport's TokenGuard
        'provider' => 'users',
        'hash'     => false,
    ],
],
```

```php
<?php
// File: app/Http/Controllers/Api/OrderController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Order;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function index(Request $request): JsonResponse
    {
        // The authenticated user is resolved via the Passport guard
        $user = $request->user();

        // Check the token's scopes
        if (!$user->token()->scopes->contains('orders:read')) {
            return response()->json(['error' => 'Insufficient scope.'], 403);
        }

        return response()->json(Order::where('user_id', $user->id)->get());
    }
}
```

**Expected Output:**

- `GET /api/orders` without an OAuth2 access token → `401 Unauthorized`.
- `GET /api/orders` with a valid token that has the `orders:read` scope → `200 OK` with the user's orders.
- `GET /api/orders` with a valid token that lacks the `orders:read` scope → `403 Forbidden`.

**Why This Output Occurs:** The `auth:api` middleware uses Passport's `TokenGuard` to validate the OAuth2 access token in the `Authorization: Bearer` header. Passport checks the token against the `oauth_access_tokens` table and resolves the associated user. The controller then checks the token's scopes (OAuth2 permissions) and returns a 403 if the required scope is missing. Passport is suitable for APIs that need full OAuth2 flows (authorization code, client credentials, password grant).

### Real-World Cases

- **Mobile app backends:** Sanctum tokens are issued on login and included as Bearer tokens in subsequent API calls. Tokens can be revoked individually or in bulk (e.g., on logout or password change).
- **SPA authentication:** Sanctum's SPA authentication uses session cookies for first-party SPAs, with CSRF protection, while still allowing token-based access for third parties.
- **Third-party API access:** Passport is used when external developers need to authenticate via OAuth2, with scopes controlling what data they can access.
- **Microservices authentication:** Services authenticate to each other using client credentials tokens (Passport) or service-specific Sanctum tokens.

---

## 5. Rate Limiting

### Definitions

**Core Definition:** Rate limiting is the practice of restricting the number of requests a client can make to an API within a specified time window, protecting the server from abuse, overload, or brute-force attacks.

**Technical Definition:** Laravel's rate limiting is implemented through the `Illuminate\Cache\RateLimiting\Limit` class and the `Illuminate\Support\Facades\RateLimiter` facade. Named rate limiters are defined in a service provider (typically `AppServiceProvider::boot()` in Laravel 11+) using `RateLimiter::for('name', function (Request $request) { return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip()); })`. These named limiters are applied to routes via the `throttle:name` middleware. Laravel also provides a default `api` rate limiter (60 requests per minute per user/IP) configured in `RouteServiceProvider` (Laravel 10-) or `bootstrap/app.php` (Laravel 11+).

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer at a club: you can only enter a certain number of times per hour. If you try to enter too many times, you are turned away (usually with a "429 Too Many Requests" response). This protects the server from being overwhelmed by too many requests from a single user or IP address. Laravel makes it easy to set different limits for different routes: a login endpoint might allow only 5 attempts per minute, while a data retrieval endpoint might allow 60 requests per minute.

### Purposes

- To protect the API from denial-of-service (DoS) attacks and brute-force login attempts.
- To ensure fair resource allocation among users by preventing any single client from monopolising server capacity.
- To enforce tiered access (e.g., free users vs. premium users) with different rate limits.
- To reduce infrastructure costs by limiting excessive API usage.
- To comply with external API rate limits when your application acts as a client to other services.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Define a named rate limiter in a service provider
// File: app/Providers/AppServiceProvider.php (Laravel 11+)

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Basic rate limiter: 60 requests per minute per user or IP
        RateLimiter::for('api', function (Request $request) {
            return Limit::perMinute(60)->by(
                $request->user()?->id ?: $request->ip()
            );
        });

        // Dynamic rate limiter: different limits for VIP vs. regular users
        RateLimiter::for('uploads', function (Request $request) {
            return $request->user()?->isVip()
                ? Limit::none()
                : Limit::perHour(10)->by($request->user()?->id ?: $request->ip());
        });

        // Segmented rate limiter: different limits for authenticated vs. guests
        RateLimiter::for('downloads', function (Request $request) {
            return $request->user()
                ? Limit::perMinute(100)->by($request->user()->id)
                : Limit::perMinute(10)->by($request->ip());
        });
    }
}
```

```php
// Step 2: Apply the rate limiter to routes via the throttle middleware
// File: routes/api.php

use Illuminate\Support\Facades\Route;

// Apply the 'api' limiter to all routes in the group
Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('posts', PostController::class);
});

// Apply a custom limiter to a specific route
Route::post('/upload', [UploadController::class, 'store'])
    ->middleware(['auth:sanctum', 'throttle:uploads']);

// Apply an inline limiter (no named limiter needed)
Route::get('/search', [SearchController::class, 'index'])
    ->middleware('throttle:30,1'); // 30 requests per minute
```

**Component Breakdown:**

- `RateLimiter::for('api', function (Request $request) { ... })` — Defines a named rate limiter called `api`.
- `Limit::perMinute(60)` — Creates a limit of 60 requests per minute.
- `->by($request->user()?->id ?: $request->ip())` — Segments the limit by user ID (if authenticated) or IP address (if guest).
- `Limit::none()` — Removes the rate limit entirely (useful for VIP users).
- `Limit::perHour(10)` — Limits to 10 requests per hour.
- `throttle:api` — Applies the `api` rate limiter to the route or group.
- `throttle:30,1` — Inline limiter: 30 requests per 1 minute.

**Syntax Rules:**

- Named rate limiters are defined in a service provider's `boot()` method. In Laravel 11+, `AppServiceProvider` is the default location. In Laravel 10-, `RouteServiceProvider::configureRateLimiting()` is used.
- The `throttle` middleware accepts either a named limiter (`throttle:api`) or an inline limit (`throttle:60,1` for 60 requests per 1 minute).
- The `by()` method is used to segment limits. Without it, the limit is applied globally across all requests.
- Multiple rate limits can be returned as an array: `return [Limit::perMinute(10), Limit::perDay(1000)]`.
- The `response()` method customises the 429 response: `Limit::perMinute(10)->response(fn() => response('Too many requests', 429))`.

**Constraints and Limitations:**

- **Rate limiting requires a cache store.** Laravel uses the default cache driver (configured in `config/cache.php`) to store rate limit counters. For distributed applications, Redis or Memcached is recommended; the `array` cache driver does not persist across requests.
- **The `throttle` middleware must be applied to routes.** Defining a rate limiter does not automatically apply it; you must attach `throttle:name` to the routes.
- **Dynamic rate limiters based on user attributes require the user to be authenticated** before the rate limiter is evaluated. Apply `auth:sanctum` before `throttle` in the middleware stack.
- **Rate limit headers are added automatically** (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`), but only when the `throttle` middleware is applied.

### Annotated Code Examples

**Example 1: Defining and Applying Custom Rate Limiters**

```php
<?php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Step 1: General API rate limiter — 60 requests per minute
        // segmented by authenticated user ID or IP address for guests.
        RateLimiter::for('api', function (Request $request) {
            return Limit::perMinute(60)->by(
                $request->user()?->id ?: $request->ip()
            );
        });

        // Step 2: Login rate limiter — 5 attempts per minute per email+IP.
        // This prevents brute-force attacks without locking out legitimate
        // users who mistype their password occasionally.
        RateLimiter::for('login', function (Request $request) {
            return [
                Limit::perMinute(5)->by($request->input('email') . '|' . $request->ip()),
                Limit::perMinute(20)->by($request->ip()),
            ];
        });

        // Step 3: Premium tier rate limiter — VIP users get no limit,
        // regular users get 10 uploads per hour.
        RateLimiter::for('uploads', function (Request $request) {
            return $request->user()?->isVip()
                ? Limit::none()
                : Limit::perHour(10)->by($request->user()?->id ?: $request->ip());
        });

        // Step 4: Response-based rate limiter — only count 404 responses
        // toward the limit, allowing legitimate retries of other status codes.
        RateLimiter::for('resource-not-found', function (Request $request) {
            return Limit::perMinute(10)
                ->by($request->ip())
                ->response(function (Request $request, array $headers) {
                    return response()->json([
                        'error' => 'Too many not-found requests. Please check your URLs.',
                    ], 429, $headers);
                });
        });
    }
}
```

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\AuthController;
use App\Http\Controllers\Api\PostController;
use App\Http\Controllers\Api\UploadController;

// Login routes — throttled to prevent brute force
Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:login');

// General API routes — authenticated and throttled
Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('posts', PostController::class);
});

// Upload routes — dynamic throttling based on user tier
Route::post('/upload', [UploadController::class, 'store'])
    ->middleware(['auth:sanctum', 'throttle:uploads']);
```

**Step-by-Step Setup:**

1. Ensure a cache driver is configured in `.env` (Redis recommended for production).
2. Define the rate limiters in `AppServiceProvider::boot()` as shown.
3. Apply the `throttle` middleware to the appropriate routes.
4. Test by sending multiple requests and observing the `429 Too Many Requests` response.

**Expected Output:**

- `POST /api/login` with incorrect credentials: after 5 attempts, returns `429 Too Many Requests` with `{"message": "Too many attempts."}` and a `Retry-After` header.
- `GET /api/posts` as an authenticated user: 60 requests per minute allowed; the 61st returns `429`.
- `POST /api/upload` as a regular user: 10 requests per hour; the 11th returns `429`. As a VIP user: no limit.
- `GET /api/nonexistent` repeatedly: after 10 404 responses within a minute, returns `429` with the custom error message.

**Why This Output Occurs:** The `RateLimiter::for()` calls define named limiters that are stored in the `RateLimiter` facade's internal registry. When the `throttle:login` middleware is applied, Laravel resolves the `login` limiter, evaluates the `by()` segmentation key, and increments the counter in the cache. If the counter exceeds the limit, a `429` response is returned with the appropriate headers. The `response()` method customises the 429 response body. The `Limit::none()` return for VIP users bypasses the counter entirely.

---

**Example 2: Inline Rate Limiting Without Named Limiters**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\SearchController;

// Inline rate limit: 30 requests per minute.
// This does not require a named limiter defined in a service provider.
Route::get('/search', [SearchController::class, 'index'])
    ->middleware('throttle:30,1');

// Inline rate limit with a custom response
Route::get('/data', function () {
    return response()->json(['data' => 'sensitive']);
})->middleware('throttle:10,1')->name('data');
```

**Expected Output:**

- `GET /api/search`: the first 30 requests within a minute succeed; the 31st returns `429 Too Many Requests`.
- `GET /api/data`: the first 10 requests within a minute succeed; the 11th returns `429`.

**Why This Output Occurs:** The `throttle:30,1` middleware syntax creates an inline rate limiter of 30 requests per 1 minute. Laravel's `ThrottleRequests` middleware parses the parameters, resolves the rate limiter (or creates an ad-hoc one), and enforces the limit using the cache store. The segment key defaults to the authenticated user's ID or the IP address. This approach is convenient for simple limits but lacks the flexibility of named limiters (no dynamic segmentation, no custom responses).

### Real-World Cases

- **Login and registration endpoints:** Throttled to 5–10 attempts per minute to prevent brute-force attacks and credential stuffing.
- **Public API endpoints:** Throttled to 60–100 requests per minute per user/IP to ensure fair usage.
- **File upload endpoints:** Throttled with different limits for free vs. premium users (e.g., 10 uploads/hour for free, unlimited for premium).
- **Third-party API proxies:** Your application acts as a proxy to an external API with its own rate limits; you throttle your own users to stay within those limits.
- **Webhook endpoints:** Throttled to prevent abuse from malicious actors who discover the webhook URL.

---

## References

- Laravel Routing Documentation (13.x) — https://laravel.com/framework/docs/routing
- Laravel Routing Documentation (11.x) — https://laravel.com/framework/docs/11.x/routing
- Laravel Routing Documentation (10.x) — https://laravel.com/framework/docs/10.x/routing
- Laravel Sanctum Documentation — https://laravel.com/framework/docs/sanctum
- Laravel Passport Documentation — https://laravel.com/framework/docs/passport
- Laravel Rate Limiting Documentation — https://laravel.com/framework/docs/routing#rate-limiting
- Laravel Middleware Documentation — https://laravel.com/framework/docs/middleware
- API Versioning in Laravel 11 (Laravel News) — https://laravel-news.com/api-versioning-in-laravel-11
- Laravel API Versionizer Package — https://github.com/ahmedessam/api-versionizer
- Laravel API Versioning Middleware (philiprehberger) — https://github.com/philiprehberger/laravel-api-versioning
- Laravel Rate Limiting Guide (Laravel Magazine) — https://laravelmagazine.com/rate-limiting-laravel-routes-and-actions-the-right-way
- Illuminate\Cache\RateLimiting\Limit API — https://api.laravel.com/docs/11.x/Illuminate/Cache/RateLimiting/Limit.html
- Laravel Route Groups Documentation — https://laravel.com/framework/docs/routing#route-groups
- Laravel `install:api` Artisan Command — https://laravel.com/framework/docs/artisan#install-api