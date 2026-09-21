# Laravel Middleware Fundamentals: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel middleware is a mechanism that provides a convenient way to inspect and filter HTTP requests entering an application, acting as a series of layers that requests must pass through before reaching the application's core logic.

**Technical Definition:** Middleware in Laravel is implemented through the `Illuminate\Pipeline\Pipeline` class, which composes a series of invokable classes or closures into a chain of responsibility. Each middleware class implements a `handle(Request $request, Closure $next)` method that receives the request, performs its logic, and either passes the request to the next layer via `$next($request)` or returns a response directly (short-circuiting the pipeline). Middleware are resolved from the service container, allowing constructor dependency injection. The HTTP kernel (`Illuminate\Foundation\Http\Kernel`) orchestrates the middleware stack through its `sendRequestThroughRouter()` method, which uses the pipeline to process the request through global middleware, route group middleware, and route-specific middleware.

**Beginner-Friendly Explanation:** Middleware is like a series of security checkpoints at an airport. Before you reach your gate (the controller), you pass through check-in, security screening, and passport control (middleware). Each checkpoint can check your credentials, ask questions, or even deny you entry. If you pass all checkpoints, you board your flight. And on your way out (the response), you pass through the same checkpoints again—they might stamp your passport or add a stamp. Middleware in Laravel works exactly the same way: each layer can inspect, modify, or reject the request before it reaches the application, and inspect or modify the response on its way back to the user.

### Key Characteristics

- **Onion Architecture:** Middleware wraps around the application core like layers of an onion, with requests flowing inward and responses flowing outward through the same layers.
- **Pipeline Execution:** Middleware are executed sequentially through Laravel's Pipeline pattern, forming a chain of responsibility.
- **Before and After Processing:** Each middleware can execute logic before passing the request (`return $next($request)`) or after receiving the response (`$response = $next($request)`).
- **Short-Circuit Capability:** Middleware can reject a request entirely by returning a response without calling `$next($request)`.
- **Service Container Resolution:** Middleware are resolved from the IoC container, supporting constructor dependency injection.
- **Three-Level Registration:** Middleware can be registered globally (every request), per group (route groups), or per route (specific routes).
- **Terminable Middleware:** Middleware implementing the `Terminable` interface can perform work after the response has been sent to the browser.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (the Middleware API is stable across versions; Laravel 11+ moved middleware configuration to `bootstrap/app.php`).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `app/Http/Middleware` directory.
- Basic understanding of HTTP requests, responses, and the request lifecycle.

### Related Programming Areas

- **Routing:** Middleware are assigned to routes and route groups to filter requests before they reach controllers.
- **HTTP Kernel:** The kernel orchestrates the middleware stack and dispatches the request through the pipeline.
- **Service Container:** Middleware are resolved through the container, enabling dependency injection.
- **Authentication & Authorization:** Built-in middleware (`auth`, `guest`, `can`, `verified`) protect routes based on user state.
- **CSRF Protection:** The `VerifyCsrfToken` middleware validates CSRF tokens on POST, PUT, PATCH, and DELETE requests.
- **Rate Limiting:** The `throttle` middleware limits request frequency to prevent abuse.

### Core Concepts / Features

1. Middleware Definition and HTTP Lifecycle Role
2. Request Filtering and Inspection
3. Request Preprocessing
4. Response Postprocessing
5. The Onion Architecture Pattern

---

## 1. Middleware Definition and HTTP Lifecycle Role

### Definitions

**Core Definition:** Middleware is a class that sits between the HTTP request and the application's route handlers, providing a mechanism to inspect, filter, and modify requests and responses as they flow through the HTTP lifecycle.

**Technical Definition:** In Laravel's HTTP lifecycle, the request enters through `public/index.php`, is passed to the HTTP kernel (`App\Http\Kernel`), and is sent through the middleware pipeline via the `sendRequestThroughRouter()` method. The kernel's `$middleware` array defines global middleware that every request passes through. The pipeline uses `Illuminate\Pipeline\Pipeline` to compose middleware into a nested closure chain, where each middleware's `handle()` method receives the request and a `$next` closure. The pipeline executes middleware in the order they are registered, and the response travels back through the middleware stack in reverse order. The `handle()` method signature is `handle(Request $request, Closure $next): Response`.

**Beginner-Friendly Explanation:** Every time a request comes into your Laravel application, it doesn't go straight to the controller. It passes through a series of middleware classes first—like a series of checkpoints. One middleware might check if you're logged in. Another might verify your CSRF token. Another might log the request. Only after passing through all these checkpoints does the request reach your controller. When the controller generates a response, that response goes back through the same checkpoints in reverse order, giving each middleware a chance to modify it before it reaches the user.

### Purposes

- To provide a convenient mechanism for inspecting and filtering HTTP requests entering the application.
- To perform common request processing logic (authentication, CSRF verification, session management) before requests reach controllers.
- To enable cross-cutting concerns to be handled separately from business logic.
- To allow requests to be rejected or redirected before reaching the application core.
- To provide a central location for request/response logging, header manipulation, and rate limiting.

### Syntax Rules and Structure

**Complete General Syntax — Middleware Class:**

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `handle(Request $request, Closure $next)` | The middleware's entry point. |
| `$request` | The current HTTP request instance. |
| `$next` | The closure representing the next middleware in the pipeline. |
| `return $next($request)` | Passes the request to the next layer. |
| `return redirect('/home')` | Short-circuits the pipeline, returning a response directly. |

**Complete General Syntax — Generating Middleware:**

```bash
php artisan make:middleware EnsureTokenIsValid
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `make:middleware` | The Artisan command to generate a middleware class. |
| `EnsureTokenIsValid` | The class name (creates `app/Http/Middleware/EnsureTokenIsValid.php`). |

**Complete General Syntax — Middleware Registration (Laravel 11+):**

```php
// bootstrap/app.php

use App\Http\Middleware\EnsureTokenIsValid;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->append(EnsureTokenIsValid::class);
        $middleware->alias([
            'token' => EnsureTokenIsValid::class,
        ]);
    })
    ->create();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$middleware->append(...)` | Appends middleware to the global middleware stack. |
| `$middleware->alias([...])` | Registers a middleware alias for use in routes. |

**Syntax Rules:**

- Middleware classes are stored in `app/Http/Middleware` by default.
- The `handle()` method must accept a `Request` instance and a `Closure` named `$next`.
- To pass the request deeper into the application, call `$next($request)` and return its result.
- To short-circuit the pipeline, return a response directly without calling `$next($request)`.
- Middleware are resolved from the service container, so constructor injection is supported.

**Constraints and Limitations:**

- **Order dependency:** Middleware execution order is determined by registration order. Incorrect ordering can cause unexpected behaviour.
- **Short-circuiting:** Once a middleware returns a response without calling `$next`, subsequent middleware and the route handler are not executed.
- **Global middleware performance:** Global middleware runs on every request, including static assets and API routes. Excessive global middleware can impact performance.
- **Laravel 11+ changes:** The `app/Http/Kernel.php` file no longer exists; middleware are configured in `bootstrap/app.php`.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Middleware with Token Validation**

```php
<?php
// app/Http/Middleware/EnsureTokenIsValid.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        // Check if the token input matches the expected value
        if ($request->input('token') !== 'my-secret-token') {
            // Short-circuit: redirect to home if token is invalid
            return redirect('/home');
        }

        // Pass the request to the next middleware
        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Middleware\EnsureTokenIsValid;
use Illuminate\Support\Facades\Route;

Route::get('/secret', function () {
    return 'You have accessed the secret page!';
})->middleware(EnsureTokenIsValid::class);
```

**Expected Output:**
- `GET /secret?token=my-secret-token` → `You have accessed the secret page!`
- `GET /secret?token=wrong` → redirect to `/home`
- `GET /secret` → redirect to `/home`

**Why:** The middleware checks the `token` input against the expected value. If it matches, the request passes to the route handler. If not, the middleware short-circuits the pipeline and returns a redirect response.

---

**Example 2: Middleware with Dependency Injection**

```php
<?php
// app/Http/Middleware/LogRequest.php

namespace App\Http\Middleware;

use App\Services\Logger;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class LogRequest
{
    public function __construct(
        private Logger $logger
    ) {}

    public function handle(Request $request, Closure $next): Response
    {
        // Log the incoming request before processing
        $this->logger->log('Request received', [
            'method' => $request->method(),
            'path' => $request->path(),
        ]);

        $response = $next($request);

        // Log the response status after processing
        $this->logger->log('Response sent', [
            'status' => $response->getStatusCode(),
        ]);

        return $response;
    }
}
```

```php
<?php
// routes/web.php

Route::get('/dashboard', function () {
    return 'Dashboard';
})->middleware(\App\Http\Middleware\LogRequest::class);
```

**Expected Output:** `GET /dashboard` logs the request method and path before processing, and the response status after processing. The response `Dashboard` is returned to the client.

**Why:** The middleware uses constructor injection to receive a `Logger` service. It performs logging before and after the request is handled, demonstrating the "before" and "after" pattern in a single middleware.

---

**Example 3: Global Middleware Registration (Laravel 11+)**

```php
<?php
// bootstrap/app.php

use App\Http\Middleware\LogRequest;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Append to global middleware — runs on every request
        $middleware->append(LogRequest::class);

        // Register alias for route-level use
        $middleware->alias([
            'log' => LogRequest::class,
        ]);
    })
    ->create();
```

**Expected Output:** Every HTTP request to the application passes through the `LogRequest` middleware. The middleware alias `log` can be used in route definitions: `->middleware('log')`.

**Why:** The `append()` method adds the middleware to the global middleware stack, ensuring it runs on every request. The `alias()` method registers a short name for use in route definitions.

### Real-World Cases

- **Authentication:** The `auth` middleware checks if the user is logged in and redirects to the login page if not.
- **CSRF Protection:** The `VerifyCsrfToken` middleware validates CSRF tokens on state-changing requests.
- **Rate Limiting:** The `throttle` middleware limits request frequency to prevent brute-force attacks.
- **Localization:** A `SetLocale` middleware sets the application locale based on the request's `Accept-Language` header.
- **Tenant Resolution:** A `ResolveTenant` middleware identifies the tenant from the subdomain and scopes database queries.

### References

- Laravel Middleware Documentation — https://laravel.com/docs/12.x/middleware
- Laravel Request Lifecycle — https://laravel.com/docs/12.x/lifecycle
- Laravel API: Pipeline — https://api.laravel.com/docs/12.x/Illuminate/Pipeline/Pipeline.html
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know

---

## 2. Request Filtering and Inspection

### Definitions

**Core Definition:** Request filtering and inspection is the middleware capability to examine incoming HTTP requests and either allow them to proceed, modify them, or reject them entirely based on specified conditions.

**Technical Definition:** The `handle()` method receives the `Illuminate\Http\Request` instance, which provides access to all request data (input, headers, files, session, user). Middleware can inspect this data using methods like `$request->input()`, `$request->header()`, `$request->user()`, and `$request->isMethod()`. Based on inspection results, the middleware can call `$next($request)` to allow the request to proceed, return a response directly to reject it (short-circuiting), or modify the request before passing it on. The `abort()` helper can be used to throw an HTTP exception (e.g., `abort(403, 'Unauthorized')`) which Laravel's exception handler converts into an appropriate error response.

**Beginner-Friendly Explanation:** Middleware can act like a bouncer at a club. It checks your ID (inspects the request), verifies you meet the dress code (filters based on conditions), and either lets you in (passes the request) or turns you away (returns a rejection response). The bouncer doesn't just check one thing—they can check multiple conditions and make a decision based on all of them.

### Purposes

- To inspect requests for authentication status via `$request->user()`.
- To filter requests based on roles, permissions, or other authorization criteria.
- To validate API tokens or secret keys before allowing access.
- To check request headers for version compatibility, content types, or custom identifiers.
- To reject requests that do not meet specified conditions, returning appropriate HTTP status codes.
- To log request metadata for monitoring and auditing purposes.

### Syntax Rules and Structure

**Complete General Syntax — Request Inspection:**

```php
public function handle(Request $request, Closure $next): Response
{
    // Inspect: check if user is authenticated
    if (! $request->user()) {
        return redirect('/login');
    }

    // Inspect: check user role
    if (! $request->user()->hasRole('admin')) {
        abort(403, 'Admin access required');
    }

    // Allow the request to proceed
    return $next($request);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$request->user()` | Returns the authenticated user or `null`. |
| `$request->user()->hasRole('admin')` | Checks if the user has the admin role. |
| `abort(403, '...')` | Throws an HTTP exception with status 403. |
| `return $next($request)` | Passes the request to the next middleware. |

**Complete General Syntax — Header Inspection:**

```php
public function handle(Request $request, Closure $next): Response
{
    // Check for a custom header
    if (! $request->hasHeader('X-API-Key')) {
        return response()->json(['error' => 'API key required'], 401);
    }

    // Check the Accept header for JSON
    if (! $request->expectsJson()) {
        abort(406, 'This endpoint requires JSON.');
    }

    return $next($request);
}
```

**Complete General Syntax — Method Inspection:**

```php
public function handle(Request $request, Closure $next): Response
{
    if ($request->isMethod('post') && ! $request->has('_token')) {
        abort(419, 'CSRF token missing');
    }

    return $next($request);
}
```

**Syntax Rules:**

- Request inspection methods (`input()`, `header()`, `user()`, `isMethod()`, `expectsJson()`) do not modify the request.
- `abort()` throws an `HttpException` that Laravel's exception handler converts to an error response (403, 404, 419, 500, etc.).
- Returning a response directly from `handle()` short-circuits the pipeline—the request never reaches the route handler.
- Middleware inspection should be fast—avoid heavy database queries unless necessary.
- Use `$request->user()` instead of `auth()->user()` for testability and clarity.

**Constraints and Limitations:**

- **Performance:** Heavy database queries in middleware can slow down every request that passes through them. Use caching for repeated lookups.
- **Order dependency:** If a middleware depends on `$request->user()` being set, it must run after the authentication middleware.
- **Short-circuiting:** Once a response is returned, subsequent middleware and the route handler are not executed. Ensure this is intentional.
- **API vs web:** API routes do not have session middleware by default. `$request->user()` may return `null` for unauthenticated API requests.

### Multiple Annotated Complete Code Examples

**Example 1: Authentication Check Middleware**

```php
<?php
// app/Http/Middleware/CheckAdmin.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAdmin
{
    /**
     * Restrict access to admin users only.
     */
    public function handle(Request $request, Closure $next): Response
    {
        // Inspect: check if the user is authenticated
        if (! $request->user()) {
            return redirect('/login');
        }

        // Inspect: check if the user has admin privileges
        if (! $request->user()->isAdmin()) {
            abort(403, 'Admin access required');
        }

        // Allow the request to proceed
        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Middleware\CheckAdmin;
use Illuminate\Support\Facades\Route;

Route::get('/admin/users', function () {
    return 'Admin user list';
})->middleware(['auth', CheckAdmin::class]);
```

**Expected Output:**
- Unauthenticated user → redirect to `/login`
- Authenticated non-admin user → 403 Forbidden
- Authenticated admin user → `Admin user list`

**Why:** The middleware first checks authentication, then authorization. It short-circuits with a redirect or abort if either check fails, and passes the request to the next layer only when both checks pass.

---

**Example 2: API Key Validation Middleware**

```php
<?php
// app/Http/Middleware/ValidateApiKey.php

namespace App\Http\Middleware;

use App\Models\ApiKey;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ValidateApiKey
{
    public function handle(Request $request, Closure $next): Response
    {
        // Extract the API key from the header
        $apiKey = $request->header('X-API-Key');

        if (! $apiKey) {
            return response()->json([
                'error' => 'API key is required.',
            ], 401);
        }

        // Validate the key against the database
        $key = ApiKey::where('key', $apiKey)->first();

        if (! $key || $key->isExpired()) {
            return response()->json([
                'error' => 'Invalid or expired API key.',
            ], 401);
        }

        // Attach the key's user to the request for downstream use
        $request->merge(['api_user_id' => $key->user_id]);

        return $next($request);
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Middleware\ValidateApiKey;
use Illuminate\Support\Facades\Route;

Route::middleware([ValidateApiKey::class])->group(function () {
    Route::get('/user', function (Request $request) {
        return response()->json([
            'user_id' => $request->input('api_user_id'),
        ]);
    });
});
```

**Expected Output:**
- Request without `X-API-Key` header → `{"error": "API key is required."}` with 401
- Request with invalid key → `{"error": "Invalid or expired API key."}` with 401
- Request with valid key → `{"user_id": 1}`

**Why:** The middleware inspects the `X-API-Key` header, validates it against the database, and merges the associated user ID into the request for downstream use. It returns JSON error responses for API clients.

---

**Example 3: Content-Type Inspection Middleware**

```php
<?php
// app/Http/Middleware/RequireJson.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class RequireJson
{
    public function handle(Request $request, Closure $next): Response
    {
        // Inspect: check if the request expects JSON
        if (! $request->expectsJson()) {
            return response()->json([
                'error' => 'This endpoint requires the Accept header to include application/json.',
            ], 406);
        }

        // Inspect: check if the Content-Type is JSON for POST/PUT/PATCH
        if ($request->isMethod('post') || $request->isMethod('put') || $request->isMethod('patch')) {
            if (! $request->isJson()) {
                return response()->json([
                    'error' => 'The Content-Type must be application/json.',
                ], 415);
            }
        }

        return $next($request);
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Middleware\RequireJson;
use Illuminate\Support\Facades\Route;

Route::middleware([RequireJson::class])->group(function () {
    Route::post('/posts', function (Request $request) {
        return response()->json(['created' => true]);
    });
});
```

**Expected Output:**
- POST without `Accept: application/json` → 406 Not Acceptable
- POST with `Accept: application/json` but `Content-Type: text/plain` → 415 Unsupported Media Type
- POST with both JSON headers → `{"created": true}`

**Why:** The middleware inspects both the `Accept` header (for response format expectations) and the `Content-Type` header (for request body format), returning appropriate error status codes for each violation.

### Real-World Cases

- **Admin Panels:** `CheckAdmin` middleware restricts access to administrative routes based on user roles.
- **API Gateways:** `ValidateApiKey` middleware authenticates API consumers and attaches user context.
- **Multi-Tenancy:** `ResolveTenant` middleware identifies the tenant from the subdomain or header and scopes database queries.
- **Feature Flags:** `EnsureFeatureEnabled` middleware checks if a feature is enabled for the current user before allowing access.
- **Maintenance Mode:** `CheckForMaintenanceMode` middleware rejects requests when the application is in maintenance mode.

### References

- Laravel Middleware Documentation — https://laravel.com/docs/12.x/middleware
- Laravel HTTP Requests: Request Information — https://laravel.com/docs/12.x/requests#request-information
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- Laravel API: Request — https://api.laravel.com/docs/12.x/Illuminate/Http/Request.html

---

## 3. Request Preprocessing

### Definitions

**Core Definition:** Request preprocessing is the middleware capability to modify, sanitize, or augment the incoming request data before it reaches the controller, ensuring that the application receives clean, consistent, and normalized input.

**Technical Definition:** Middleware can modify the `Illuminate\Http\Request` instance using methods like `$request->merge()`, `$request->replace()`, and `$request->input()` setters. Laravel includes two built-in preprocessing middleware: `TrimStrings` (trims whitespace from all string inputs) and `ConvertEmptyStringsToNull` (converts empty strings to `null`). These are registered in the global middleware stack. Custom middleware can perform additional preprocessing such as sanitizing HTML, normalizing phone numbers, converting date formats, or injecting computed values.

**Beginner-Friendly Explanation:** Before your controller sees the request data, middleware can clean it up. Maybe the user submitted a form with extra spaces around their name, or left a field empty (which should be treated as `null` instead of an empty string). Preprocessing middleware handles these normalizations automatically, so your controller doesn't have to worry about them. It's like a maid service that tidies up the hotel room before you check in.

### Purposes

- To trim whitespace from string inputs via the `TrimStrings` middleware.
- To convert empty strings to `null` via the `ConvertEmptyStringsToNull` middleware.
- To sanitize input data by stripping HTML tags or escaping dangerous characters.
- To normalise data formats (e.g., phone numbers, dates, monetary values) before validation.
- To merge computed or default values into the request for downstream use.
- To remove sensitive or unnecessary fields from the request before it reaches the controller.
- To convert request data types (e.g., string `"true"` to boolean `true`).

### Syntax Rules and Structure

**Complete General Syntax — Trimming Strings:**

```php
// Built-in: TrimStrings middleware
// Automatically trims all string inputs
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `TrimStrings` | Global middleware that trims leading/trailing whitespace from all string inputs. |
| `ConvertEmptyStringsToNull` | Global middleware that converts empty strings to `null`. |
| Exceptions | Fields can be excluded using the `$except` property. |

**Complete General Syntax — Custom Preprocessing Middleware:**

```php
public function handle(Request $request, Closure $next): Response
{
    // Merge additional data into the request
    $request->merge([
        'slug' => Str::slug($request->input('title')),
    ]);

    // Replace specific input values
    $request->replace([
        'name' => trim($request->input('name')),
    ]);

    return $next($request);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$request->merge([...])` | Adds or overwrites request input values. |
| `$request->replace([...])` | Completely replaces the request input array. |
| `Str::slug()` | Generates a URL-friendly slug from a string. |

**Complete General Syntax — Excluding Fields from Trimming:**

```php
// In a custom middleware extending TrimStrings
protected $except = [
    'password',
    'password_confirmation',
];
```

**Syntax Rules:**

- `TrimStrings` and `ConvertEmptyStringsToNull` are registered in the global middleware stack by default since Laravel 5.4.
- The `$except` property on `TrimStrings` allows excluding specific fields from trimming.
- `merge()` adds or overwrites specific keys; `replace()` replaces the entire input array.
- Preprocessing middleware should run before validation middleware to ensure validation operates on clean data.
- Use `$request->merge()` for additive changes and `$request->replace()` for wholesale replacements.

**Constraints and Limitations:**

- **Order matters:** Preprocessing middleware must run before validation middleware. If validation runs first, it validates the original (unprocessed) data.
- **Irreversibility:** Once input is merged or replaced, the original data is lost. Ensure preprocessing is intentional.
- **Field exclusion:** `TrimStrings` excludes fields listed in its `$except` property, but custom middleware must implement its own exclusion logic.
- **Performance:** Preprocessing every input field on every request can add overhead. Only preprocess fields that require it.

### Multiple Annotated Complete Code Examples

**Example 1: Trimming Strings and Converting Empty Strings to Null**

```php
<?php
// The TrimStrings and ConvertEmptyStringsToNull middleware are included
// in Laravel's global middleware stack by default.

// Example request: POST /users
// Input: name="  Alice  ", email="", bio="  Hello World  "

// After preprocessing:
// name="Alice", email=null, bio="Hello World"
```

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function store(Request $request)
    {
        // The input is already trimmed and empty strings are null
        return response()->json([
            'name' => $request->input('name'),   // "Alice"
            'email' => $request->input('email'),  // null
            'bio' => $request->input('bio'),      // "Hello World"
        ]);
    }
}
```

**Expected Output:** `POST /users` with `name="  Alice  "&email=&bio="  Hello World  "` returns `{"name":"Alice","email":null,"bio":"Hello World"}`.

**Why:** The `TrimStrings` middleware removes leading and trailing whitespace from `name` and `bio`. The `ConvertEmptyStringsToNull` middleware converts the empty `email` string to `null`. The controller receives clean, normalized data.

---

**Example 2: Custom Preprocessing Middleware**

```php
<?php
// app/Http/Middleware/NormalizePhoneNumber.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class NormalizePhoneNumber
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->has('phone')) {
            // Strip all non-digit characters from the phone number
            $normalized = preg_replace('/[^0-9]/', '', $request->input('phone'));

            // Merge the normalized phone number back into the request
            $request->merge(['phone' => $normalized]);
        }

        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Middleware\NormalizePhoneNumber;
use Illuminate\Support\Facades\Route;

Route::post('/contact', function (Request $request) {
    return response()->json([
        'phone' => $request->input('phone'),
    ]);
})->middleware(NormalizePhoneNumber::class);
```

**Expected Output:** `POST /contact` with `phone="+1 (555) 123-4567"` returns `{"phone":"15551234567"}`.

**Why:** The middleware strips all non-digit characters from the phone number before the request reaches the route handler. The normalized value is merged back into the request, replacing the original.

---

**Example 3: Merging Computed Values into the Request**

```php
<?php
// app/Http/Middleware/AddSlugToRequest.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AddSlugToRequest
{
    public function handle(Request $request, Closure $next): Response
    {
        // If a title is present but no slug, generate one
        if ($request->has('title') && ! $request->has('slug')) {
            $request->merge([
                'slug' => Str::slug($request->input('title')),
            ]);
        }

        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Middleware\AddSlugToRequest;
use Illuminate\Support\Facades\Route;

Route::post('/posts', function (Request $request) {
    return response()->json([
        'title' => $request->input('title'),
        'slug' => $request->input('slug'),
    ]);
})->middleware(AddSlugToRequest::class);
```

**Expected Output:** `POST /posts` with `title="Hello World"` returns `{"title":"Hello World","slug":"hello-world"}`.

**Why:** The middleware detects that a `title` is present but no `slug`, generates a slug from the title, and merges it into the request. The route handler receives both the original title and the computed slug.

### Real-World Cases

- **User Registration:** Trimming and normalizing name, email, and username inputs before validation.
- **E-commerce Checkout:** Normalizing phone numbers, postal codes, and credit card numbers before processing.
- **Content Management:** Generating slugs from titles before storing posts.
- **API Payloads:** Converting empty strings to `null` for consistent JSON responses.
- **Multi-Language Applications:** Normalizing locale codes and translating input values.

### References

- Laravel HTTP Requests: Input Trimming & Normalization — https://laravel.com/docs/12.x/requests#input-trimming-and-normalization
- Laravel HTTP Requests: Merging Additional Input — https://laravel.com/docs/12.x/requests#merging-additional-input
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know

---

## 4. Response Postprocessing

### Definitions

**Core Definition:** Response postprocessing is the middleware capability to modify the HTTP response after it has been generated by the controller but before it is sent to the client, allowing headers to be added, content to be modified, or status codes to be changed.

**Technical Definition:** In middleware, the response is obtained by calling `$response = $next($request)`. After this call, the middleware can modify the response using methods like `$response->header()`, `$response->withHeaders()`, and `$response->setStatusCode()`. The modified response must be returned from the `handle()` method. For operations that should occur after the response has been sent to the browser, middleware can implement the `Terminable` interface and define a `terminate()` method, which receives both the request and response.

**Beginner-Friendly Explanation:** After your controller generates a response—whether it's an HTML page, JSON data, or a redirect—middleware can still modify it before it reaches the user. Maybe you want to add a security header to every response, or compress the HTML to make it smaller, or log the response status. Response postprocessing middleware does this work after the controller has finished but before the user sees the result.

### Purposes

- To add security headers (e.g., `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`) to every response.
- To add CORS headers for cross-origin API requests.
- To log response status codes, content types, and sizes for monitoring.
- To modify response content (e.g., minifying HTML, injecting scripts).
- To set cookies on the response for user tracking or session management.
- To perform expensive operations after the response has been sent using terminable middleware.

### Syntax Rules and Structure

**Complete General Syntax — After Middleware:**

```php
public function handle(Request $request, Closure $next): Response
{
    // 1. Process the request and get the response
    $response = $next($request);

    // 2. Modify the response
    $response->headers->set('X-Content-Type-Options', 'nosniff');
    $response->headers->set('X-Frame-Options', 'DENY');

    // 3. Return the modified response
    return $response;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$response = $next($request)` | Executes the rest of the pipeline and captures the response. |
| `$response->headers->set()` | Sets a response header using the Symfony HeaderBag API. |
| `$response->header()` | Alternative Laravel method for setting headers. |
| `return $response` | Returns the modified response. |

**Complete General Syntax — Terminable Middleware:**

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class LogRequest
{
    public function handle(Request $request, Closure $next): Response
    {
        $request->attributes->set('start_time', microtime(true));
        return $next($request);
    }

    public function terminate(Request $request, Response $response): void
    {
        $duration = microtime(true) - $request->attributes->get('start_time');
        \Log::info('Request completed', [
            'url' => $request->fullUrl(),
            'method' => $request->method(),
            'status' => $response->getStatusCode(),
            'duration_ms' => round($duration * 1000, 2),
        ]);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `handle()` | Captures the start time and passes the request. |
| `terminate()` | Executes after the response is sent to the browser. |
| `$response->getStatusCode()` | Returns the HTTP status code of the response. |

**Syntax Rules:**

- The response is obtained by calling `$response = $next($request)`.
- Modifications must occur after this call and before `return $response`.
- Use `$response->headers->set()` for Symfony-style header manipulation or `$response->header()` for Laravel's fluent API.
- Terminable middleware must implement the `Illuminate\Contracts\TerminableMiddleware` interface (or simply define a `terminate()` method).
- Terminable middleware's `terminate()` method is called after the response has been sent to the browser, so it cannot modify the response content.

**Constraints and Limitations:**

- **Response modification:** Only response headers, status codes, and content can be modified. The response body can be read and replaced, but large bodies should not be loaded into memory unnecessarily.
- **Terminable middleware:** `terminate()` runs after the response is sent, so it cannot change the response. It is suitable for logging, cleanup, and analytics.
- **Order dependency:** Response modifications occur in reverse order of middleware execution—the last middleware to call `$next` is the first to modify the response.
- **Performance:** Heavy processing in after middleware slows down response delivery. Use terminable middleware for non-critical work.

### Multiple Annotated Complete Code Examples

**Example 1: Adding Security Headers**

```php
<?php
// app/Http/Middleware/AddSecurityHeaders.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AddSecurityHeaders
{
    public function handle(Request $request, Closure $next): Response
    {
        // Get the response from the rest of the pipeline
        $response = $next($request);

        // Add security headers
        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-Frame-Options', 'DENY');
        $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
        $response->headers->set('Permissions-Policy', 'geolocation=()');

        return $response;
    }
}
```

```php
<?php
// bootstrap/app.php

use App\Http\Middleware\AddSecurityHeaders;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        // Apply to every response
        $middleware->append(AddSecurityHeaders::class);
    })
    ->create();
```

**Expected Output:** Every response includes the headers `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, and `Permissions-Policy: geolocation=()`.

**Why:** The middleware calls `$next($request)` to get the response, then modifies its headers before returning it. The headers are added to every response that passes through the middleware.

---

**Example 2: Logging Response Status**

```php
<?php
// app/Http/Middleware/LogResponseStatus.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class LogResponseStatus
{
    public function handle(Request $request, Closure $next): Response
    {
        // Process the request
        $response = $next($request);

        // Log the response status
        Log::info('Response status', [
            'status' => $response->getStatusCode(),
            'content_type' => $response->headers->get('Content-Type'),
            'url' => $request->fullUrl(),
        ]);

        return $response;
    }
}
```

**Expected Output:** Every request logs the response status code, content type, and URL to the application log.

**Why:** The middleware captures the response and logs its metadata before returning it. This provides visibility into response patterns across the application.

---

**Example 3: Terminable Middleware for Performance Logging**

```php
<?php
// app/Http/Middleware/LogRequestDuration.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class LogRequestDuration
{
    public function handle(Request $request, Closure $next): Response
    {
        // Record the start time before processing
        $request->attributes->set('start_time', microtime(true));

        return $next($request);
    }

    public function terminate(Request $request, Response $response): void
    {
        // Calculate the duration after the response is sent
        $duration = microtime(true) - $request->attributes->get('start_time');

        Log::info('Request completed', [
            'url' => $request->fullUrl(),
            'method' => $request->method(),
            'status' => $response->getStatusCode(),
            'duration_ms' => round($duration * 1000, 2),
        ]);
    }
}
```

**Expected Output:** After the response is sent to the client, the middleware logs the request URL, method, status code, and duration in milliseconds.

**Why:** The `terminate()` method runs after the response has been sent, so it does not delay the user's experience. This makes it ideal for logging, analytics, and cleanup tasks that should not block the response.

### Real-World Cases

- **Security Headers:** Adding `X-Frame-Options`, `X-Content-Type-Options`, and `Strict-Transport-Security` to every response.
- **CORS:** Adding `Access-Control-Allow-Origin` and related headers for cross-origin API requests.
- **Performance Monitoring:** Logging request duration and response status for APM (Application Performance Monitoring) tools.
- **Content Minification:** Minifying HTML responses before sending them to the browser.
- **Cache Headers:** Setting `Cache-Control` and `ETag` headers based on response content.

### References

- Laravel Middleware: Middleware and Responses — https://laravel.com/docs/12.x/middleware#middleware-and-responses
- Laravel Middleware: Terminable Middleware — https://laravel.com/docs/12.x/middleware#terminable-middleware
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know

---

## 5. The Onion Architecture Pattern

### Definitions

**Core Definition:** The onion architecture pattern describes how middleware wraps around the application core like layers of an onion, with requests passing through each layer from the outside in, and responses passing back through the same layers from the inside out.

**Technical Definition:** Laravel's middleware pipeline implements the Chain of Responsibility and Decorator patterns through the `Illuminate\Pipeline\Pipeline` class. The pipeline reduces the middleware array into a nested closure, where each middleware's `$next` parameter is a closure representing the remaining middleware and the final route handler. When a middleware calls `$next($request)`, it invokes the next layer, which in turn invokes the next, until the innermost layer (the route handler) is reached. The response then travels back through the closure chain in reverse order. This creates a symmetric "onion" structure where each middleware layer has both an inbound phase (before `$next`) and an outbound phase (after `$next`).

**Beginner-Friendly Explanation:** Imagine an onion with many layers. The center of the onion is your application's route handler. The outer layers are middleware. When a request comes in, it starts at the outermost layer, passes through each layer one by one, until it reaches the center. Then, the response is generated and travels back out through the same layers in reverse order. Each layer can do something on the way in (before passing the request) and something on the way out (after getting the response). This is why middleware is often described as having "before" and "after" phases.

### Purposes

- To provide a mental model for understanding how middleware execution order works.
- To enable both request preprocessing (inbound) and response postprocessing (outbound) in the same middleware.
- To ensure that middleware execution order is symmetric—the last middleware to process the request is the first to process the response.
- To allow middleware to short-circuit the pipeline at any layer, preventing inner layers from executing.
- To support the decorator pattern, where each middleware "decorates" the request/response with additional behaviour.

### Syntax Rules and Structure

**Complete General Syntax — Before and After in One Middleware:**

```php
public function handle(Request $request, Closure $next): Response
{
    // BEFORE: This runs on the way in (request phase)
    // Example: log the incoming request, modify input, check authentication

    $response = $next($request);

    // AFTER: This runs on the way out (response phase)
    // Example: add headers, log the response, modify content

    return $response;
}
```

**Component Breakdown:**

| Phase | Description |
|-------|-------------|
| Before `$next($request)` | Request preprocessing — runs on the way in. |
| `$response = $next($request)` | Executes the inner layers and the route handler. |
| After `$next($request)` | Response postprocessing — runs on the way out. |

**Complete General Syntax — Pipeline Execution Order:**

```
Request enters
    ↓
Middleware 1 (before)
    ↓
Middleware 2 (before)
    ↓
Middleware 3 (before)
    ↓
Route Handler
    ↑
Middleware 3 (after)
    ↑
Middleware 2 (after)
    ↑
Middleware 1 (after)
    ↑
Response exits
```

**Syntax Rules:**

- Middleware registered first executes first on the way in and last on the way out.
- Middleware registered last executes last on the way in and first on the way out.
- Short-circuiting at any layer prevents inner layers from executing, but outer layers' after phases still run.
- The `$next` closure represents the remaining middleware and the route handler.
- The pipeline uses array reduction to build the nested closure chain.

**Constraints and Limitations:**

- **Order sensitivity:** Middleware order is critical. Authentication middleware must run before authorization middleware. Session middleware must run before authentication.
- **Short-circuiting:** When a middleware returns a response without calling `$next`, inner layers are skipped, but outer layers' after phases still execute.
- **Performance:** Each middleware layer adds a function call overhead. Minimise the number of global middleware.
- **Terminable middleware:** The `terminate()` method runs after the entire pipeline, outside the onion structure.

### Multiple Annotated Complete Code Examples

**Example 1: Before and After Middleware**

```php
<?php
// app/Http/Middleware/MeasureRequestDuration.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class MeasureRequestDuration
{
    public function handle(Request $request, Closure $next): Response
    {
        // BEFORE: Record the start time
        $startTime = microtime(true);

        // Pass the request through the inner layers
        $response = $next($request);

        // AFTER: Calculate and add the duration header
        $duration = microtime(true) - $startTime;
        $response->headers->set('X-Request-Duration', round($duration * 1000, 2) . 'ms');

        return $response;
    }
}
```

```php
<?php
// routes/web.php

Route::get('/test', function () {
    // Simulate some work
    usleep(50000); // 50ms
    return 'Hello World';
})->middleware(\App\Http\Middleware\MeasureRequestDuration::class);
```

**Expected Output:** `GET /test` returns `Hello World` with the header `X-Request-Duration: 50.12ms` (or similar).

**Why:** The middleware records the start time before calling `$next($request)`, and calculates the duration after the response is generated. The duration is added as a response header. This demonstrates both the before and after phases in a single middleware.

---

**Example 2: Multiple Middleware Execution Order**

```php
<?php
// app/Http/Middleware/FirstMiddleware.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class FirstMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // BEFORE phase
        \Log::info('First: before');

        $response = $next($request);

        // AFTER phase
        \Log::info('First: after');

        return $response;
    }
}
```

```php
<?php
// app/Http/Middleware/SecondMiddleware.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class SecondMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // BEFORE phase
        \Log::info('Second: before');

        $response = $next($request);

        // AFTER phase
        \Log::info('Second: after');

        return $response;
    }
}
```

```php
<?php
// routes/web.php

Route::get('/test', function () {
    return 'Hello';
})->middleware([\App\Http\Middleware\FirstMiddleware::class, \App\Http\Middleware\SecondMiddleware::class]);
```

**Expected Output:** The log entries appear in this order:
1. `First: before`
2. `Second: before`
3. `Second: after`
4. `First: after`

**Why:** The middleware are registered as `[First, Second]`. First executes its before phase, then calls `$next` (which executes Second's before phase). Second calls `$next` (the route handler), then executes its after phase, returning the response to First, which then executes its after phase. This demonstrates the onion structure: First is the outermost layer, Second is the inner layer.

---

**Example 3: Short-Circuiting in the Onion**

```php
<?php
// app/Http/Middleware/CheckMaintenance.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckMaintenance
{
    public function handle(Request $request, Closure $next): Response
    {
        \Log::info('CheckMaintenance: before');

        // Short-circuit if maintenance mode is enabled
        if (app()->isDownForMaintenance()) {
            \Log::info('CheckMaintenance: short-circuiting');

            return response('Application is down for maintenance.', 503);
        }

        $response = $next($request);

        \Log::info('CheckMaintenance: after');

        return $response;
    }
}
```

```php
<?php
// routes/web.php

Route::get('/test', function () {
    return 'Hello World';
})->middleware([
    \App\Http\Middleware\CheckMaintenance::class,
    \App\Http\Middleware\LogRequest::class,
]);
```

**Expected Output:**
- Normal operation: `CheckMaintenance: before` → `LogRequest: before` → `LogRequest: after` → `CheckMaintenance: after`
- Maintenance mode: `CheckMaintenance: before` → `CheckMaintenance: short-circuiting` → 503 response (inner middleware and route handler are skipped)

**Why:** When `CheckMaintenance` short-circuits by returning a response without calling `$next($request)`, the inner middleware (`LogRequest`) and the route handler are never executed. However, `CheckMaintenance`'s after phase (the code after `$next`) is also skipped because `$next` was never called. The response is returned directly from the short-circuiting middleware.

### Real-World Cases

- **Authentication + Logging:** Authentication middleware short-circuits unauthenticated requests before logging middleware runs.
- **Maintenance Mode:** `CheckForMaintenanceMode` runs as the outermost middleware and short-circuits all requests during maintenance.
- **Rate Limiting:** `ThrottleRequests` runs early in the stack and short-circuits requests that exceed the rate limit.
- **CORS:** `HandleCors` runs as an early middleware and adds CORS headers to responses on the way out.
- **Session Management:** `StartSession` runs before authentication middleware so that `auth` can access the session.

### References

- Laravel Request Lifecycle — https://laravel.com/docs/12.x/lifecycle
- Laravel API: Pipeline — https://api.laravel.com/docs/12.x/Illuminate/Pipeline/Pipeline.html
- Laravel API: Router Pipeline — https://api.laravel.com/docs/12.x/Illuminate/Routing/Pipeline.html
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- Stack Overflow: Laravel Middleware Onion Structure — https://stackoverflow.com/questions/51465073

---

## References

- Laravel Middleware Documentation (12.x) — https://laravel.com/docs/12.x/middleware
- Laravel Middleware: Introduction — https://laravel.com/docs/12.x/middleware#introduction
- Laravel Middleware: Defining Middleware — https://laravel.com/docs/12.x/middleware#defining-middleware
- Laravel Middleware: Middleware and Responses — https://laravel.com/docs/12.x/middleware#middleware-and-responses
- Laravel Middleware: Terminable Middleware — https://laravel.com/docs/12.x/middleware#terminable-middleware
- Laravel Request Lifecycle — https://laravel.com/docs/12.x/lifecycle
- Laravel HTTP Requests: Input Trimming & Normalization — https://laravel.com/docs/12.x/requests#input-trimming-and-normalization
- Laravel API: Pipeline — https://api.laravel.com/docs/12.x/Illuminate/Pipeline/Pipeline.html
- Laravel API: Router Pipeline — https://api.laravel.com/docs/12.x/Illuminate/Routing/Pipeline.html
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- Stack Overflow: Laravel Middleware Onion Structure — https://stackoverflow.com/questions/51465073
- Laravel Middleware Patterns (GitHub) — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/iserter/laravel-claude-agents/laravel-middleware-patterns/SKILL.md
- Laravel 13.x Documentation — https://laravel.com/docs/