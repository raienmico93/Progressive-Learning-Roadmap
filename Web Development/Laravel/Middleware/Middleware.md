# Laravel Middleware: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Middleware is a mechanism for filtering HTTP requests entering an application, acting as a series of "layers" that requests must pass through before reaching the application's core logic, and responses must pass through on the way out.

**Technical Definition**

Middleware in Laravel is implemented as PHP classes with a `handle()` method that receives an `Illuminate\Http\Request` instance and a `Closure $next` callback. Each middleware can perform actions before or after passing the request deeper into the application by calling `$next($request)`. Middleware are resolved via the service container, support dependency injection in their constructors, can accept parameters, and may define a `terminate()` method for post-response processing. Since Laravel 11, middleware configuration has moved from `app/Http/Kernel.php` to the `bootstrap/app.php` file using the `withMiddleware()` method.

**Beginner-Friendly Explanation**

Middleware is like a series of security checkpoints at an airport. When you (the request) arrive, you pass through checkpoints (middleware) that verify your identity, check your baggage, and ensure you have permission to proceed. Each checkpoint can either let you through or turn you away. Once you reach your destination (the controller), the response travels back through the same checkpoints. Middleware is useful for tasks like checking if a user is logged in, protecting against CSRF attacks, and limiting how many requests someone can make.

---

### Key Characteristics

- **Request Filtering**: Middleware inspects and filters HTTP requests before they reach the application, and can also process responses on the way out.
- **Layered Architecture**: Requests pass through a stack of middleware in a defined order; each layer can examine the request and either pass it on or reject it entirely.
- **Service Container Resolution**: All middleware are resolved via Laravel's service container, enabling constructor dependency injection.
- **Multiple Registration Points**: Middleware can be registered globally, assigned to routes, grouped, or aliased for convenience.
- **Parameter Support**: Middleware can receive additional parameters (e.g., `role:editor`) for dynamic behaviour.
- **Terminable Middleware**: Middleware can define a `terminate()` method that runs after the response has been sent to the browser.
- **Priority Sorting**: Middleware execution order can be controlled via the `priority()` method (Laravel 11+) or `$middlewarePriority` property (Laravel 10 and earlier).
- **Built-in Middleware**: Laravel ships with middleware for authentication, authorization, CSRF protection, CORS, rate limiting, trusted proxies, trusted hosts, cookie encryption, signature validation, string trimming, and empty string conversion.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Basic understanding of HTTP requests and responses**.
- **Familiarity with Laravel's routing system**.
- **Knowledge of service container and dependency injection** is helpful.

---

### Related Programming Areas

- **Routing**: Middleware is assigned to routes and route groups to filter requests.
- **Authentication**: The `auth` middleware verifies user identity before granting access.
- **Authorization**: The `can` middleware checks user permissions for specific actions.
- **CSRF Protection**: The `VerifyCsrfToken` middleware protects against cross-site request forgery.
- **Rate Limiting**: The `throttle` middleware limits request frequency.
- **API Development**: The `api` middleware group provides stateless request handling for APIs.
- **Service Container**: Middleware dependencies are resolved via Laravel's service container.

---

## Core Concepts

---

### 1. Middleware

#### Definitions

**Core Definition**

Middleware is a PHP class that filters HTTP requests entering an application, performing actions before or after the request is handled by the application's core logic.

**Technical Definition**

Middleware classes implement a `handle(Request $request, Closure $next): Response` method. The `$request` parameter contains the incoming HTTP request, and `$next` is a closure that passes the request to the next middleware in the stack. Calling `$next($request)` allows the request to proceed; returning a response without calling `$next($request)` short-circuits the stack. Middleware can perform tasks before or after calling `$next($request)`, and can be resolved with dependencies via the service container.

**Beginner-Friendly Explanation**

Middleware is code that runs before or after your application handles a request. For example, a middleware might check if a user is logged in before allowing them to access a page. If they are not logged in, the middleware redirects them to the login page instead of letting the request continue.

#### Purposes

- To filter HTTP requests before they reach the application's core logic.
- To perform cross-cutting concerns (authentication, logging, CORS) without cluttering controllers.
- To reject unauthorized or invalid requests early in the request lifecycle.
- To modify requests or responses as they pass through the stack.
- To perform post-response tasks such as logging or session persistence.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Http/Middleware/EnsureTokenIsValid.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `handle(Request $request, Closure $next)` | The method executed for each request. |
| `$request` | The incoming HTTP request instance. |
| `$next` | Closure to pass the request to the next middleware. |
| `$next($request)` | Passes the request deeper into the stack. |
| Return value | Must return a `Response` instance. |

**Syntax Rules**

- Middleware classes are stored in `app/Http/Middleware`.
- The `handle` method is required and must return a `Response`.
- Calling `$next($request)` passes the request to the next middleware.
- Returning a response without calling `$next($request)` short-circuits the stack.
- Dependencies can be type-hinted in the constructor for service container resolution.

**Constraints and Limitations**

- Middleware runs on every request (if global) or on assigned routes, so performance-critical code should be minimal.
- Middleware should not contain business logic; delegate to services or actions.
- The `handle` method must return a response; returning `null` causes an error.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Token Validation Middleware**

**Middleware**

```php
<?php
// app/Http/Middleware/EnsureTokenIsValid.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    public function handle(Request $request, Closure $next): Response
    {
        // Check if the request has the correct token
        if ($request->input('token') !== 'my-secret-token') {
            // Redirect to home if token is invalid
            return redirect('/home');
        }

        // Pass the request to the next middleware
        return $next($request);
    }
}
```

**Route**

```php
// routes/web.php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/secret', function () {
    return 'Secret content';
})->middleware(EnsureTokenIsValid::class);
```

**Expected Output**

```
// GET /secret?token=my-secret-token → "Secret content"
// GET /secret?token=wrong → redirect to /home
// GET /secret (no token) → redirect to /home
```

**Why This Output Occurs**

The middleware checks the `token` input against the expected value. If it matches, `$next($request)` is called and the request proceeds to the route handler. If it does not match, a redirect response is returned immediately, short-circuiting the middleware stack.

---

**Example 2: Before and After Middleware**

**Before Middleware**

```php
public function handle(Request $request, Closure $next): Response
{
    // Perform action before the request is handled
    Log::info('Request received: ' . $request->path());

    return $next($request);
}
```

**After Middleware**

```php
public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);

    // Perform action after the request is handled
    Log::info('Response sent: ' . $response->status());

    return $response;
}
```

**Expected Output**

```
// Before middleware logs "Request received: /dashboard"
// After middleware logs "Response sent: 200"
```

**Why This Output Occurs**

The before middleware logs before calling `$next($request)`, so it runs before the route handler. The after middleware calls `$next($request)` first, stores the response, logs after the response is generated, and then returns the response.

---

#### Real-World Cases with Explanation

**Case 1: API Request Logging**

A logging middleware captures every API request's method, path, and response time. It is registered globally so all API requests are logged without modifying controllers.

**Explanation**: Middleware provides a clean, centralized way to implement cross-cutting concerns like logging.

**Case 2: Multi-Tenant Application**

A tenant identification middleware runs on every request, resolves the tenant from the subdomain or header, and sets the tenant context before the request reaches the controller.

**Explanation**: Middleware is ideal for request-scoped context setup in multi-tenant architectures.

---

### 2. Middleware Creation

#### Definitions

**Core Definition**

Middleware creation is the process of generating a new middleware class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:middleware` command generates a PHP class in `app/Http/Middleware` extending no base class but containing a `handle()` method with the standard signature. The generated class is ready for customization.

**Beginner-Friendly Explanation**

Creating middleware is a one-command process. Laravel makes the file for you, and you fill in the logic.

#### Purposes

- To automate the generation of middleware class boilerplate.
- To ensure middleware are placed in the correct namespace and directory.
- To provide a consistent structure for middleware files.
- To reduce manual file creation errors.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:middleware EnsureTokenIsValid
php artisan make:middleware CheckRole
php artisan make:middleware LogRequests
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:middleware EnsureTokenIsValid` | Creates `app/Http/Middleware/EnsureTokenIsValid.php`. |
| `make:middleware CheckRole` | Creates `app/Http/Middleware/CheckRole.php`. |

**Syntax Rules**

- The middleware name should be PascalCase and descriptive.
- The generated file contains a `handle()` method stub.
- Generated middleware must be registered before use.

**Constraints and Limitations**

- The command does not register the middleware automatically.
- Generated middleware must be edited to add logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Middleware**

**Command**

```bash
php artisan make:middleware EnsureTokenIsValid
```

**Generated File**

```php
<?php
// app/Http/Middleware/EnsureTokenIsValid.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }
}
```

**Expected Output (Terminal)**

```
INFO  Middleware [app/Http/Middleware/EnsureTokenIsValid.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the middleware class with the `handle` method stub, ready for customization.

---

#### Real-World Cases with Explanation

**Case 1: Role-Based Access Middleware**

A developer runs `php artisan make:middleware CheckRole` and implements logic to verify that the authenticated user has a specific role before allowing access.

**Explanation**: Artisan accelerates development by generating the necessary boilerplate.

---

### 3. `artisan make:middleware`

#### Definitions

**Core Definition**

`artisan make:middleware` is the Artisan command used to generate new middleware classes.

**Technical Definition**

The command is implemented by `Illuminate\Foundation\Console\MiddlewareMakeCommand`. It generates a PHP class file in `app/Http/Middleware` with a `handle()` method stub. The class does not extend any base class; it is a plain PHP class with the required method signature.

**Beginner-Friendly Explanation**

`artisan make:middleware` is the terminal command you type to create a new middleware file.

#### Purposes

- To generate middleware classes with minimal effort.
- To provide consistent structure for middleware files.
- To enforce Laravel's naming and directory conventions.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:middleware <Name>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `make:middleware` | The Artisan command. |
| `<Name>` | The middleware class name (PascalCase). |

**Syntax Rules**

- The middleware name is required.
- The generated file is placed in `app/Http/Middleware`.
- The class contains a `handle()` method stub.

**Constraints and Limitations**

- Middleware are not automatically registered.
- Generated stubs must be filled in with logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating and Registering a Middleware**

**Command**

```bash
php artisan make:middleware CheckAge
```

**Generated File (edited)**

```php
<?php
// app/Http/Middleware/CheckAge.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAge
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->age < 18) {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

**Registration (Laravel 11+)**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'age' => \App\Http\Middleware\CheckAge::class,
    ]);
})
```

**Expected Output**

```
// GET /restricted?age=20 → proceeds
// GET /restricted?age=15 → redirect to /home
```

**Why This Output Occurs**

The middleware checks the `age` input. If the age is below 18, it redirects; otherwise, it passes the request on.

---

#### Real-World Cases with Explanation

**Case 1: API Versioning Middleware**

A developer generates a middleware to check the `Accept` header for API version and route the request to the appropriate version's controllers.

**Explanation**: Middleware can handle API version negotiation centrally.

---

### 4. Middleware Registration

#### Definitions

**Core Definition**

Middleware registration is the process of informing Laravel about a middleware class and how it should be applied (globally, to routes, or as part of a group).

**Technical Definition**

In Laravel 11+, middleware are registered in `bootstrap/app.php` using the `withMiddleware()` method. Global middleware are added via `append()` or `prepend()`. Aliases are registered via `alias()`. Groups are defined via `group()`, `appendToGroup()`, or `prependToGroup()`. In Laravel 10 and earlier, registration was done in `app/Http/Kernel.php` using the `$middleware`, `$middlewareGroups`, and `$middlewareAliases` properties.

**Beginner-Friendly Explanation**

Registration is how you tell Laravel "this middleware exists and here is how I want to use it." You can make it run on every request, only on specific routes, or as part of a group.

#### Purposes

- To make Laravel aware of custom middleware classes.
- To define how middleware should be applied (globally, per-route, or per-group).
- To create convenient aliases for middleware.
- To organize middleware into logical groups.

#### Syntax Rules and Structure

**Complete General Syntax (Laravel 11+)**

```php
// bootstrap/app.php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\LogRequests;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Global middleware
        $middleware->append(LogRequests::class);
        $middleware->prepend(EnsureTokenIsValid::class);

        // Middleware aliases
        $middleware->alias([
            'token' => EnsureTokenIsValid::class,
        ]);

        // Middleware groups
        $middleware->group('custom', [
            EnsureTokenIsValid::class,
            LogRequests::class,
        ]);

        // Modify existing groups
        $middleware->web(append: [LogRequests::class]);
        $middleware->api(prepend: [EnsureTokenIsValid::class]);
    })
    ->create();
```

**Complete General Syntax (Laravel 10 and earlier)**

```php
// app/Http/Kernel.php
protected $middleware = [
    \App\Http\Middleware\TrustProxies::class,
    \Illuminate\Http\Middleware\HandleCors::class,
    // ...
];

protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
        // ...
    ],
    'api' => [
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];

protected $middlewareAliases = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    // ...
];
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `append()` | Adds middleware to the end of the global stack. |
| `prepend()` | Adds middleware to the beginning of the global stack. |
| `alias()` | Registers a short name for a middleware class. |
| `group()` | Defines a new middleware group. |
| `appendToGroup()` | Adds middleware to the end of an existing group. |
| `prependToGroup()` | Adds middleware to the beginning of an existing group. |

**Syntax Rules**

- Global middleware run on every request.
- Aliases allow short names in route definitions.
- Groups bundle multiple middleware for easy assignment.
- The `web` and `api` groups are predefined and can be modified.

**Constraints and Limitations**

- Global middleware impact performance as they run on every request.
- Middleware aliases must be unique.
- Middleware groups cannot be nested.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Global Middleware Registration**

```php
// bootstrap/app.php (Laravel 11+)
->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\App\Http\Middleware\LogRequests::class);
})
```

**Expected Output**

```
// LogRequests middleware runs on every request to the application
```

**Why This Output Occurs**

The `append()` method adds the middleware to the global stack, which is executed for all requests.

---

**Example 2: Middleware Alias Registration**

```php
// bootstrap/app.php (Laravel 11+)
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
    ]);
})
```

**Usage**

```php
// routes/web.php
Route::get('/admin', function () {
    // ...
})->middleware('admin');
```

**Expected Output**

```
// The 'admin' alias resolves to EnsureUserIsAdmin middleware
```

**Why This Output Occurs**

The `alias()` method maps the short name `admin` to the middleware class, allowing it to be used in route definitions.

---

#### Real-World Cases with Explanation

**Case 1: Custom Middleware Group for Admin Routes**

An application defines an `admin` middleware group containing authentication, admin role checking, and logging middleware. Admin routes are grouped under this middleware group for consistent protection.

**Explanation**: Middleware groups reduce repetition and ensure consistent protection across related routes.

---

### 5. Global Middleware

#### Definitions

**Core Definition**

Global middleware are middleware that run on every HTTP request to the application, regardless of the route being accessed.

**Technical Definition**

Global middleware are registered in the `$middleware` property of the HTTP Kernel (Laravel 10 and earlier) or via `append()` / `prepend()` in `bootstrap/app.php` (Laravel 11+). They are executed for every request, forming the outermost layers of the middleware stack. Examples include `TrustProxies`, `HandleCors`, `PreventRequestsDuringMaintenance`, and `TrimStrings`.

**Beginner-Friendly Explanation**

Global middleware is like the security at the entrance of a building—everyone who enters must pass through it, no matter where they are going inside.

#### Purposes

- To apply consistent filtering or processing to all requests.
- To handle cross-cutting concerns such as CORS, proxy trust, and maintenance mode.
- To ensure that certain tasks always run before any route-specific logic.
- To provide a baseline of security and request normalization.

#### Syntax Rules and Structure

**Complete General Syntax (Laravel 11+)**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\App\Http\Middleware\LogRequests::class);
    $middleware->prepend(\App\Http\Middleware\EnsureTokenIsValid::class);
})
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `append()` | Adds middleware to the end of the global stack. |
| `prepend()` | Adds middleware to the beginning of the global stack. |

**Syntax Rules**

- Global middleware are executed in the order they are registered.
- `prepend()` adds to the beginning; `append()` adds to the end.
- Global middleware run before route-specific middleware.

**Constraints and Limitations**

- Global middleware run on every request, so they should be lightweight.
- Adding too many global middleware can slow down the application.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Adding Global Middleware**

```php
// bootstrap/app.php
use App\Http\Middleware\LogRequests;

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(LogRequests::class);
})
```

**Expected Output**

```
// LogRequests middleware runs on every request before route-specific middleware
```

**Why This Output Occurs**

The `append()` method adds the middleware to the global stack, which is executed for all requests.

---

#### Real-World Cases with Explanation

**Case 1: Trusted Proxies**

The `TrustProxies` middleware is registered globally to ensure that Laravel correctly interprets the client IP address when the application is behind a load balancer or proxy.

**Explanation**: Global middleware handles infrastructure-level concerns that apply to all requests.

---

### 6. Route Middleware

#### Definitions

**Core Definition**

Route middleware are middleware assigned to specific routes or route groups, running only when those routes are accessed.

**Technical Definition**

Route middleware are assigned via the `middleware()` method on route definitions or through route groups. They are stored in the route's action array and resolved during route dispatching. Aliases registered in `bootstrap/app.php` (Laravel 11+) or `$middlewareAliases` (Laravel 10 and earlier) allow short names to be used instead of full class names.

**Beginner-Friendly Explanation**

Route middleware is like a specific security check for a particular room. Only people going to that room need to pass through it.

#### Purposes

- To apply authorization checks to specific routes.
- To protect routes with authentication requirements.
- To enforce rate limits on individual routes or groups.
- To apply middleware only where it is needed, reducing overhead.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Single route
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth');

// Multiple middleware
Route::get('/admin', function () {
    return view('admin');
})->middleware(['auth', 'admin']);

// Route group
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/dashboard', function () {
        return view('admin.dashboard');
    });
});
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `->middleware('alias')` | Assigns middleware to a single route. |
| `->middleware(['a', 'b'])` | Assigns multiple middleware. |
| `Route::middleware([...])->group()` | Assigns middleware to a group of routes. |

**Syntax Rules**

- Middleware aliases can be used for brevity.
- Multiple middleware execute in the order listed.
- Route middleware run after global middleware.

**Constraints and Limitations**

- Route middleware do not run on routes they are not assigned to.
- Middleware order matters; authentication should run before authorization.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Single Route Middleware**
\
Step 1: Create the Middleware and run this command in your terminal:
```bash
php artisan make:middleware AuthenticateUser
```
\
Step 2: Define the Middleware Logic
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
use Illuminate\Support\Facades\Auth;

class AuthenticateUser
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        // Check if the user is NOT authenticated
        if (!Auth::check()) {
            // Redirect them to the login page with an error message
            return redirect()->route('login')->with('error', 'You must log in first.');
        }

        // If authenticated, allow the request to proceed
        return $next($request);
    }
}
```
\
Step 3: Register the Middleware (Laravel 11+)

In Laravel 11 and newer, middleware is registered in the bootstrap/app.php file using an alias.
Open bootstrap/app.php and register your middleware like this:
```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Register your single middleware alias here
        $middleware->alias([
            'auth.custom' => \App\Http\Middleware\AuthenticateUser::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```
(Note: If you are using Laravel 10 or older, you would register this inside the $middlewareAliases array in app/Http/Kernel.php instead).
\
Step 4: Apply the Middleware to your Route
```php
<?php

use Illuminate\Support\Facades\Route;

// Public Login Route (The fallback destination)
Route::get('/login', function () {
    return 'Please log in here. (Login Page)';
})->name('login');

// Protected Profile Route using Single Route Middleware
Route::get('/profile', function () {
    return 'Welcome to your profile page!';
})->middleware('auth.custom');
```


**Expected Output**

```
// Authenticated user → profile view
// Guest → redirect to login
```

**Why This Output Occurs**

The `auth` middleware checks if the user is authenticated. If not, it redirects to the login page.

---

**Example 2: Route Group Middleware**

```php
Route::middleware(['auth', 'admin'])->prefix('admin')->group(function () {
    Route::get('/dashboard', function () {
        return 'Admin Dashboard';
    });
    Route::get('/users', function () {
        return 'User Management';
    });
});
```

**Expected Output**

```
// Authenticated admin → access granted
// Authenticated non-admin → 403 Forbidden
// Guest → redirect to login
```

**Why This Output Occurs**

Both `auth` and `admin` middleware must pass for the grouped routes to be accessed.

---

#### Real-World Cases with Explanation

**Case 1: API Rate Limiting**

API routes are assigned the `throttle:api` middleware to limit requests per minute per client.

**Explanation**: Route middleware applies rate limiting only where needed.

---

### 7. Middleware Groups

#### Definitions

**Core Definition**

Middleware groups are named collections of middleware that can be assigned to routes as a single unit, simplifying route definitions and ensuring consistent middleware application.

**Technical Definition**

Middleware groups are defined in the `$middlewareGroups` property of the HTTP Kernel (Laravel 10 and earlier) or via the `group()` method in `bootstrap/app.php` (Laravel 11+). Laravel provides two default groups: `web` (session, cookies, CSRF) and `api` (rate limiting, binding substitution). Groups can be modified with `appendToGroup()`, `prependToGroup()`, or by using the `web()` and `api()` methods.

**Beginner-Friendly Explanation**

A middleware group is like a package deal. Instead of listing five middleware individually on every route, you create a group and assign the group name.

#### Purposes

- To simplify route definitions by grouping commonly used middleware.
- To ensure consistent middleware application across related routes.
- To allow customization of the default `web` and `api` groups.
- To organize middleware by purpose or context.

#### Syntax Rules and Structure

**Complete General Syntax (Laravel 11+)**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    // Define a new group
    $middleware->group('custom', [
        \App\Http\Middleware\EnsureTokenIsValid::class,
        \App\Http\Middleware\LogRequests::class,
    ]);

    // Modify the web group
    $middleware->web(append: [
        \App\Http\Middleware\LogRequests::class,
    ]);

    // Modify the api group
    $middleware->api(prepend: [
        \App\Http\Middleware\EnsureTokenIsValid::class,
    ]);
})
```

**Complete General Syntax (Laravel 10 and earlier)**

```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
    'api' => [
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

**Component Breakdown**

| Group | Purpose | Typical Middleware |
|-------|---------|-------------------|
| `web` | Browser-based routes | Session, cookies, CSRF, binding substitution |
| `api` | Stateless API routes | Rate limiting, binding substitution |

**Syntax Rules**

- Groups are assigned to routes using the group name.
- The `web` and `api` groups are applied automatically to `routes/web.php` and `routes/api.php`.
- Groups can be modified by adding or removing middleware.

**Constraints and Limitations**

- Groups cannot be nested.
- Removing default middleware from `web` or `api` can break functionality.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Defining a Custom Group**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->group('admin', [
        'auth',
        'admin.role',
        \App\Http\Middleware\LogAdminActions::class,
    ]);
})
```

**Usage**

```php
Route::middleware('admin')->prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard']);
});
```

**Expected Output**

```
// All admin routes run auth, admin.role, and LogAdminActions middleware
```

**Why This Output Occurs**

The `admin` group bundles three middleware, which are applied to all routes in the group.

---

**Example 2: Modifying the Web Group**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        \App\Http\Middleware\LogWebRequests::class,
    ]);
})
```

**Expected Output**

```
// LogWebRequests runs on all web routes after the default web middleware
```

**Why This Output Occurs**

The `web(append: [...])` method adds middleware to the end of the `web` group.

---

#### Real-World Cases with Explanation

**Case 1: API Group Customization**

An API-only application removes unnecessary middleware from the `web` group and adds custom middleware to the `api` group for authentication and request logging.

**Explanation**: Middleware groups can be tailored to the application's specific needs.

---

### 8. Middleware Parameters

#### Definitions

**Core Definition**

Middleware parameters are additional values passed to middleware's `handle()` method after the `$next` closure, allowing middleware behaviour to be customized per route.

**Technical Definition**

Parameters are specified in the route definition using a colon after the middleware name, with multiple parameters separated by commas (e.g., `role:editor,author`). The `handle()` method receives these parameters after the `$next` argument. The `Illuminate\Routing\Middleware\ThrottleRequests` middleware, for example, accepts `maxAttempts` and `decayMinutes` as parameters.

**Beginner-Friendly Explanation**

Middleware parameters let you customize a middleware's behaviour for different routes. For example, a role-checking middleware can be told which role is required for a specific route.

#### Purposes

- To customize middleware behaviour on a per-route basis.
- To pass configuration values to middleware without creating separate classes.
- To support dynamic rate limits, role requirements, and other parameterized checks.
- To avoid duplicating middleware for similar but slightly different use cases.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Route definition
Route::put('/post/{id}', function ($id) {
    // ...
})->middleware('role:editor');

Route::get('/admin', function () {
    // ...
})->middleware('role:admin,super-admin');
```

**Middleware**

```php
public function handle(Request $request, Closure $next, string $role, string ...$additionalRoles): Response
{
    if (! $request->user()->hasRole($role) && ! $request->user()->hasAnyRole($additionalRoles)) {
        return redirect('/home');
    }

    return $next($request);
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `middleware('name:param1,param2')` | Passes parameters to the middleware. |
| `$role` | The first parameter. |
| `...$additionalRoles` | Variadic parameters for additional values. |

**Syntax Rules**

- Parameters are separated from the middleware name by a colon.
- Multiple parameters are separated by commas.
- Parameters are passed to `handle()` after `$next`.
- Variadic parameters (`...`) can accept multiple values.

**Constraints and Limitations**

- Parameters are always strings; cast them as needed.
- Middleware must be designed to accept parameters; not all middleware do.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Role-Based Middleware with Parameter**

**Middleware**

```php
<?php
// app/Http/Middleware/CheckRole.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckRole
{
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

**Route**

```php
Route::put('/post/{id}', function ($id) {
    // ...
})->middleware('role:editor');
```

**Expected Output**

```
// User with 'editor' role → proceeds
// User without 'editor' role → redirect to /home
```

**Why This Output Occurs**

The `role:editor` parameter is passed to the middleware's `handle` method as `$role`, which checks the user's role.

---

**Example 2: Multiple Parameters**

```php
Route::get('/reports', function () {
    // ...
})->middleware('role:admin,super-admin');
```

**Middleware**

```php
public function handle(Request $request, Closure $next, string ...$roles): Response
{
    if (! $request->user()->hasAnyRole($roles)) {
        return redirect('/home');
    }

    return $next($request);
}
```

**Expected Output**

```
// User with 'admin' or 'super-admin' role → proceeds
// User without either role → redirect to /home
```

**Why This Output Occurs**

The variadic `$roles` parameter receives all comma-separated values, and `hasAnyRole()` checks if the user has any of them.

---

#### Real-World Cases with Explanation

**Case 1: Rate Limiting with Custom Parameters**

The `throttle` middleware accepts parameters for maximum attempts and decay minutes. A route can be configured with `throttle:60,1` for 60 requests per minute, or `throttle:5,1` for 5 requests per minute.

**Explanation**: Parameters allow the same middleware to be reused with different configurations.

---

### 9. Middleware Ordering

#### Definitions

**Core Definition**

Middleware ordering is the sequence in which middleware are executed as a request passes through the stack, which determines the order of operations and can affect application behaviour.

**Technical Definition**

Middleware execute in the order they are registered. Global middleware run first (in registration order), followed by route middleware (in the order they are listed). Middleware priority can be enforced via the `priority()` method in `bootstrap/app.php` (Laravel 11+) or the `$middlewarePriority` property in the HTTP Kernel (Laravel 10 and earlier). The `SortedMiddleware` collection is used internally to sort middleware based on the priority map.

**Beginner-Friendly Explanation**

Middleware order is like the order of checkpoints at the airport: passport control before security, security before boarding. The order matters because each step may depend on the previous one.

#### Purposes

- To ensure middleware execute in the correct sequence.
- To enforce dependencies between middleware (e.g., session before authentication).
- To resolve conflicts when middleware are assigned in different orders.
- To control the behaviour of the application reliably.

#### Syntax Rules and Structure

**Complete General Syntax (Laravel 11+)**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Auth\Middleware\Authenticate::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

**Complete General Syntax (Laravel 10 and earlier)**

```php
// app/Http/Kernel.php
protected $middlewarePriority = [
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \Illuminate\Auth\Middleware\Authenticate::class,
    \Illuminate\Routing\Middleware\ThrottleRequests::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `priority([...])` | Defines the execution order. |
| `$middlewarePriority` | Property for Laravel 10 and earlier. |

**Syntax Rules**

- Middleware listed earlier in the priority array execute first.
- Middleware not in the priority array execute after those that are, in their registration order.
- The priority array is global and applies to all routes.

**Constraints and Limitations**

- Overriding the default priority can break built-in middleware dependencies.
- Priority only affects middleware that are actually assigned to a route.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Middleware Priority**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \App\Http\Middleware\LogRequests::class,
        \Illuminate\Auth\Middleware\Authenticate::class,
        \App\Http\Middleware\CheckRole::class,
    ]);
})
```

**Expected Output**

```
// LogRequests always runs before Authenticate, which always runs before CheckRole
```

**Why This Output Occurs**

The priority array enforces the execution order regardless of how the middleware are assigned to routes.

---

#### Real-World Cases with Explanation

**Case 1: Session Before Authentication**

The `StartSession` middleware must run before the `Authenticate` middleware because authentication depends on session data. The default priority array ensures this order.

**Explanation**: Middleware priority resolves dependencies between middleware.

---

### 10. Middleware Termination

#### Definitions

**Core Definition**

Terminable middleware is middleware that defines a `terminate()` method, which is called after the HTTP response has been sent to the browser.

**Technical Definition**

If a middleware class defines a `terminate(Request $request, Response $response): void` method, Laravel automatically calls it after the response has been sent to the client, provided the web server uses FastCGI (e.g., PHP-FPM). The `terminate()` method receives both the request and the response. Terminable middleware must be registered as global middleware to function correctly. Laravel resolves a fresh instance of the middleware from the service container when calling `terminate()`.

**Beginner-Friendly Explanation**

Terminable middleware lets you do work after the user has already received the response. For example, you can log the response time or save session data without making the user wait.

#### Purposes

- To perform post-response tasks without delaying the user.
- To log requests and responses after they have been completed.
- To save session data after the response is sent.
- To perform cleanup or asynchronous work in the same PHP process.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function terminate(Request $request, Response $response): void
{
    // Store session data, log request, etc.
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `terminate(Request $request, Response $response)` | Method called after the response is sent. |
| `$request` | The original request instance. |
| `$response` | The response that was sent. |

**Syntax Rules**

- The `terminate()` method is optional.
- It is called after the response is sent to the client.
- It receives the request and response instances.
- The middleware must be registered as global middleware.

**Constraints and Limitations**

- `terminate()` only works with FastCGI-based web servers (PHP-FPM, Octane).
- It runs in the same PHP process, not asynchronously.
- Modifying the response in `terminate()` has no effect because the response has already been sent.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Logging Middleware with Terminate**

```php
<?php
// app/Http/Middleware/LogRequestDuration.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class LogRequestDuration
{
    public function handle(Request $request, Closure $next): Response
    {
        $request->attributes->set('start_time', microtime(true));

        return $next($request);
    }

    public function terminate(Request $request, Response $response): void
    {
        $duration = microtime(true) - $request->attributes->get('start_time');
        Log::info("Request to {$request->path()} took {$duration} seconds");
    }
}
```

**Registration**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\App\Http\Middleware\LogRequestDuration::class);
})
```

**Expected Output**

```
// After the response is sent, the log records the request duration
```

**Why This Output Occurs**

The `handle()` method stores the start time, and the `terminate()` method calculates the duration after the response is sent and logs it.

---

#### Real-World Cases with Explanation

**Case 1: Session Persistence**

Laravel's own `StartSession` middleware uses `terminate()` to save session data to storage after the response has been sent, so the user does not wait for the session write.

**Explanation**: Terminable middleware is ideal for post-response tasks that do not affect the user experience.

---

### 11. Middleware Aliases

#### Definitions

**Core Definition**

Middleware aliases are short, memorable names assigned to middleware classes, allowing them to be referenced by a simple string in route definitions instead of the full class name.

**Technical Definition**

Aliases are registered in `bootstrap/app.php` using the `alias()` method (Laravel 11+) or in the `$middlewareAliases` property of the HTTP Kernel (Laravel 10 and earlier). Laravel ships with default aliases including `auth`, `auth.basic`, `auth.session`, `cache.headers`, `can`, `guest`, `password.confirm`, `precognitive`, `signed`, `throttle`, and `verified`.

**Beginner-Friendly Explanation**

An alias is a nickname for a middleware. Instead of typing the full class name every time, you use a short name like `auth` or `admin`.

#### Purposes

- To simplify route definitions by using short names.
- To make routes more readable.
- To decouple route definitions from middleware class names.
- To provide a consistent API for commonly used middleware.

#### Syntax Rules and Structure

**Complete General Syntax (Laravel 11+)**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
        'token' => \App\Http\Middleware\EnsureTokenIsValid::class,
    ]);
})
```

**Usage**

```php
Route::get('/admin', function () {
    // ...
})->middleware('admin');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `alias(['key' => Class])` | Registers an alias. |
| `'admin'` | The alias name. |
| `EnsureUserIsAdmin::class` | The middleware class. |

**Syntax Rules**

- Aliases are registered in `bootstrap/app.php` (Laravel 11+) or `$middlewareAliases` (Laravel 10 and earlier).
- Alias names should be short, descriptive, and unique.
- Aliases can be used with parameters (e.g., `throttle:60,1`).

**Constraints and Limitations**

- Alias names must be unique.
- Aliases are resolved at route dispatch time.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Middleware Alias**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'age' => \App\Http\Middleware\CheckAge::class,
    ]);
})
```

**Route**

```php
Route::get('/restricted', function () {
    return 'Restricted content';
})->middleware('age');
```

**Expected Output**

```
// The 'age' alias resolves to CheckAge middleware
```

**Why This Output Occurs**

The alias `age` is registered for the `CheckAge` middleware, allowing it to be used in route definitions.

---

#### Real-World Cases with Explanation

**Case 1: Authentication Alias**

The `auth` alias is used throughout the application to protect routes, providing a consistent and readable way to enforce authentication.

**Explanation**: Aliases make route definitions cleaner and more maintainable.

---

### 12. Middleware Sorting

#### Definitions

**Core Definition**

Middleware sorting is the process of arranging middleware in a specific execution order based on a priority map, ensuring that middleware with dependencies execute in the correct sequence.

**Technical Definition**

The `SortedMiddleware` class (`Illuminate\Routing\SortedMiddleware`) extends `Collection` and provides a `sortMiddleware()` method that reorders middleware based on the priority map defined in `$middlewarePriority` (Laravel 10 and earlier) or the `priority()` method (Laravel 11+). The algorithm iterates through the middleware and moves each to the correct position relative to the priority map, ensuring that middleware earlier in the priority list execute first.

**Beginner-Friendly Explanation**

Middleware sorting is like arranging a queue so that people with priority (e.g., first-class passengers) go first, regardless of when they arrived.

#### Purposes

- To enforce execution order for middleware with dependencies.
- To resolve conflicts when middleware are assigned in different orders on different routes.
- To ensure that critical middleware (e.g., session) always run before dependent middleware (e.g., authentication).
- To provide a consistent middleware execution order across the application.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Laravel 11+ (bootstrap/app.php)
$middleware->priority([
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \Illuminate\Auth\Middleware\Authenticate::class,
    \Illuminate\Routing\Middleware\ThrottleRequests::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
]);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `priority([...])` | Defines the execution order. |
| `SortedMiddleware` | Internal collection class that sorts middleware. |
| `$middlewarePriority` | Laravel 10 and earlier property. |

**Syntax Rules**

- Middleware listed earlier in the priority array execute first.
- Middleware not in the array execute after those in the array, in their registration order.
- The priority map is global.

**Constraints and Limitations**

- Overriding the default priority can break built-in dependencies.
- Sorting only affects middleware that are actually assigned to a route.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Priority Map**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \App\Http\Middleware\LogRequests::class,
        \Illuminate\Auth\Middleware\Authenticate::class,
        \App\Http\Middleware\CheckRole::class,
    ]);
})
```

**Expected Output**

```
// LogRequests always runs first, then Authenticate, then CheckRole
// regardless of the order they are listed on routes
```

**Why This Output Occurs**

The priority map instructs the `SortedMiddleware` collection to reorder the middleware array so that the specified order is enforced.

---

#### Real-World Cases with Explanation

**Case 1: Session Before Authentication**

The default priority map ensures `StartSession` runs before `Authenticate`, preventing authentication failures due to missing session data.

**Explanation**: Middleware sorting enforces critical dependencies.

---

### 13. Middleware Best Practices

#### Definitions

**Core Definition**

Middleware best practices are the recommended guidelines for designing, implementing, and maintaining middleware to maximise performance, security, and maintainability.

**Technical Definition**

Best practices encompass keeping middleware focused on request filtering, avoiding expensive database operations, using aliases for readability, grouping related middleware, separating business logic from middleware, and avoiding N+1 queries in middleware.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their middleware fast, clean, and easy to maintain.

#### Key Best Practices

1. **Keep Middleware Focused on Request Filtering**: Middleware should handle cross-cutting concerns (auth, logging, CORS), not business logic. Delegate business logic to controllers or service classes.

2. **Avoid Expensive Database Operations**: Middleware runs on every request (if global) or on many routes. Avoid lazy loading or complex queries that cause N+1 problems. If database access is necessary, ensure queries are efficient.

3. **Use Aliases for Cleaner Routes**: Register short aliases for middleware to improve route readability and maintainability.

4. **Group Related Middleware**: Use middleware groups to bundle commonly used middleware and ensure consistent application across routes.

5. **Order Middleware Carefully**: Authentication should run before authorization; session middleware should run before authentication. Use the priority map to enforce order.

6. **Use Terminable Middleware for Post-Response Tasks**: Logging, session persistence, and cleanup should be done in `terminate()` to avoid delaying the user.

7. **Avoid Duplicating Responsibilities**: Middleware should not duplicate work already handled by other middleware. Clear responsibility boundaries improve performance and maintainability.

8. **Test Middleware**: Write tests that verify middleware allows or denies requests as expected.

9. **Use `bootstrap/app.php` for Laravel 11+**: Register middleware in `bootstrap/app.php` using the `withMiddleware()` method instead of the legacy `Kernel.php` file.

10. **Prefer Declarative Controller Middleware (Laravel 13+)**: Use the `#[Middleware]` attribute on controller classes or methods for clearer, co-located security boundaries.

11. **Keep Global Middleware Minimal**: Every global middleware adds overhead to every request. Only register what truly needs to run globally.

12. **Document Middleware Order Dependencies**: When middleware depends on another middleware having run first, document this requirement.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Well-Structured Middleware Setup**

**Middleware**

```php
<?php
// app/Http/Middleware/EnsureUserIsAdmin.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->isAdmin()) {
            abort(403, 'Unauthorized');
        }

        return $next($request);
    }
}
```

**Registration**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
    ]);

    $middleware->priority([
        \Illuminate\Auth\Middleware\Authenticate::class,
        \App\Http\Middleware\EnsureUserIsAdmin::class,
    ]);
})
```

**Route**

```php
Route::middleware(['auth', 'admin'])->prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard']);
});
```

**Expected Output**

```
// Guest → redirect to login (auth middleware)
// Authenticated non-admin → 403 Forbidden (admin middleware)
// Authenticated admin → admin dashboard
```

**Why This Output Occurs**

The middleware setup follows best practices: focused middleware, alias for readability, priority to ensure `auth` runs before `admin`, and grouped routes for consistent protection.

---

#### Real-World Cases with Explanation

**Case 1: High-Traffic API**

A high-traffic API uses lightweight middleware for rate limiting and authentication, avoiding database queries in middleware. Post-response logging is done in terminable middleware to keep response times fast.

**Explanation**: Following best practices ensures middleware does not become a performance bottleneck.

---

### 14. Built-in Middleware

#### Definitions

**Core Definition**

Built-in middleware are middleware classes included with Laravel that provide common request filtering and processing functionality out of the box.

**Technical Definition**

Laravel ships with middleware for authentication, authorization, CSRF protection, CORS handling, rate limiting, trusted proxies, trusted hosts, cookie encryption, signature validation, string trimming, and empty string conversion. These are registered in the `web` and `api` middleware groups and as aliases in `bootstrap/app.php` (Laravel 11+) or `app/Http/Kernel.php` (Laravel 10 and earlier).

**Beginner-Friendly Explanation**

Laravel comes with a set of pre-built middleware that handle common tasks, so you do not have to write them yourself.

#### Authentication

**Core Definition**: The `auth` middleware verifies that the user is authenticated, redirecting guests to the login page. The `auth.basic` middleware provides HTTP Basic Authentication, and `auth.session` re-authenticates users with a session cookie.

**Syntax**:

```php
Route::get('/dashboard', function () {
    // ...
})->middleware('auth');
```

**Explanation**: The `auth` middleware is the primary gatekeeper for protected routes.

#### Authorization

**Core Definition**: The `can` middleware (`Illuminate\Auth\Middleware\Authorize`) checks if the authenticated user is authorized for a given action, using Gates or Policies.

**Syntax**:

```php
Route::get('/posts/{post}/edit', function (Post $post) {
    // ...
})->middleware('can:update,post');
```

**Explanation**: The `can` middleware delegates to Laravel's authorization system.

#### CSRF Protection

**Core Definition**: The `VerifyCsrfToken` middleware protects against cross-site request forgery by verifying that POST, PUT, PATCH, and DELETE requests include a valid CSRF token.

**Syntax**: Included automatically in the `web` middleware group.

**Explanation**: CSRF protection is enabled by default for web routes.

#### CORS

**Core Definition**: The `HandleCors` middleware adds the appropriate CORS headers to responses, allowing cross-origin requests based on the `config/cors.php` configuration.

**Syntax**: Registered globally by default.

**Explanation**: CORS middleware enables safe cross-origin API access.

#### Rate Limiting

**Core Definition**: The `throttle` middleware (`Illuminate\Routing\Middleware\ThrottleRequests`) limits the number of requests a client can make within a specified time window.

**Syntax**:

```php
Route::middleware('throttle:60,1')->group(function () {
    // ...
});
```

**Explanation**: Rate limiting protects APIs from abuse.

#### Trusted Proxies

**Core Definition**: The `TrustProxies` middleware ensures that Laravel correctly interprets the client IP address when behind a load balancer or proxy.

**Syntax**: Registered globally by default.

**Explanation**: Essential for applications behind reverse proxies.

#### Trusted Hosts

**Core Definition**: The `TrustHosts` middleware validates that the incoming request's `Host` header matches a list of trusted hosts, preventing host header attacks.

**Syntax**: Registered globally by default (commented out in new applications).

**Explanation**: Protects against host header injection.

#### Encrypt Cookies

**Core Definition**: The `EncryptCookies` middleware encrypts all cookies sent to the browser, preventing tampering.

**Syntax**: Included in the `web` middleware group.

**Explanation**: Cookie encryption enhances security for session cookies.

#### Validate Signatures

**Core Definition**: The `signed` middleware (`ValidateSignature`) verifies that the request has a valid signature, used for signed URLs.

**Syntax**:

```php
Route::get('/unsubscribe/{user}', function () {
    // ...
})->middleware('signed');
```

**Explanation**: Ensures that signed URLs have not been tampered with.

#### Trim Strings

**Core Definition**: The `TrimStrings` middleware trims leading and trailing whitespace from all request input.

**Syntax**: Registered globally by default.

**Explanation**: Ensures clean input data.

#### Convert Empty Strings to Null

**Core Definition**: The `ConvertEmptyStringsToNull` middleware converts empty string values in request input to `null`.

**Syntax**: Registered globally by default.

**Explanation**: Provides consistent handling of empty input values.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Checkout Security**

An e-commerce application uses the `auth` middleware to protect checkout routes, the `can` middleware to authorize order cancellation, and the `throttle` middleware to limit checkout attempts.

**Explanation**: Laravel's built-in middleware provide comprehensive security without custom code.

---

## References

- Middleware – Laravel 13.x Documentation — https://laravel.com/docs/13.x/middleware
- Middleware – Laravel 12.x Documentation — https://laravel.com/docs/12.x/middleware
- Middleware – Laravel 11.x Documentation — https://laravel.com/docs/11.x/middleware
- Middleware – Laravel 10.x Documentation — https://laravel.com/docs/10.x/middleware
- How to Configure Middleware in Laravel – Laravel News — https://laravel-news.com/configuring-middleware-in-laravel
- Middleware in Laravel: Main Things to Know – Laravel Daily — https://laraveldaily.com/post/middleware-in-laravel-main-things-to-know
- Laravel Middleware: Route-Level, Groups & Ordering Pitfalls – DEV Community — https://dev.to/laravel-middleware-route-level-groups-ordering-pitfalls
- Mastering Middleware Execution Order in Laravel 12 – DEV Community — https://dev.to/mastering-middleware-execution-order-in-laravel-12
- SortedMiddleware Class – Laravel API Reference — https://api.laravel.com/docs/8.x/Illuminate/Routing/SortedMiddleware.html
- Kernel Class – Laravel API Reference — https://api.laravel.com/docs/7.x/Illuminate/Foundation/Http/Kernel.html
- Middleware Best Practices – voku/agent-skills (GitHub) — https://github.com/voku/agent-skills/blob/main/skills/laravel-best-practices/AGENTS.md
- Middleware Best Practices – azaharizaman/nexus (GitHub) — https://github.com/azaharizaman/nexus/blob/main/CODING_GUIDELINES.md

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific changes include: Laravel 11 moved middleware configuration from `app/Http/Kernel.php` to `bootstrap/app.php` using the `withMiddleware()` method; the `priority()` method replaced the `$middlewarePriority` property; middleware aliases are now registered via `alias()` in `bootstrap/app.php`; and Laravel 13 introduces declarative `#[Middleware]` attributes for controller-level middleware. Developers should always consult the documentation for their specific Laravel version.