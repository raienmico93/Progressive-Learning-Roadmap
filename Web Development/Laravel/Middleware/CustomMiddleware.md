# Laravel Custom Middleware & Advanced Concepts: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Custom middleware in Laravel are user-defined classes that intercept HTTP requests and responses, providing application-specific filtering, inspection, preprocessing, and postprocessing logic that runs before or after the request reaches the application's core.

**Technical Definition:** Custom middleware implement the `handle(Request $request, Closure $next): Response` method and are resolved from the service container. They can perform logic before calling `$next($request)` (request phase), after receiving the response (response phase), or both. Middleware may also implement a `terminate(Request $request, Response $response): void` method (Terminable Middleware) that executes after the response has been sent to the client. Middleware are created via `php artisan make:middleware`, registered in `bootstrap/app.php` (Laravel 11+) or `app/Http/Kernel.php` (Laravel 10 and earlier), and assigned to routes via aliases or class references.

**Beginner-Friendly Explanation:** Custom middleware are your own personal checkpoints for HTTP requests. Laravel comes with built-in middleware for common tasks like authentication and CSRF protection, but sometimes you need to check something specific to your application—like whether a user has an active subscription, whether they're accessing from an allowed country, or whether the API request includes a valid custom header. You create a middleware class, write the logic, and attach it to routes. It's like writing your own security guard for your specific building.

### Key Characteristics

- **Service Container Resolution:** Middleware are resolved through Laravel's IoC container, enabling full dependency injection.
- **Before/After Phases:** Middleware can execute code before the request reaches the controller, after the response is generated, or both.
- **Short-Circuit Capability:** Middleware can reject a request entirely by returning a response without calling `$next($request)`.
- **Terminable Middleware:** Middleware can perform work after the response has been sent to the browser, without delaying the user.
- **Selective Exclusion:** The `withoutMiddleware()` method allows specific routes to bypass middleware that would otherwise apply.
- **Parameterized Configuration:** Middleware can receive runtime parameters (e.g., `role:admin`), enabling reusable middleware for multiple scenarios.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (middleware configuration differs between versions; Laravel 11+ uses `bootstrap/app.php`).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `app/Http/Middleware` directory.
- Basic understanding of HTTP requests, responses, and the Pipeline pattern.

### Related Programming Areas

- **Routing:** Middleware are assigned to routes and route groups for request filtering.
- **Service Container:** Middleware are resolved through the IoC container, supporting constructor and method injection.
- **Authentication & Authorization:** Built-in middleware (`auth`, `can`) protect routes based on user state.
- **Logging & Profiling:** Custom middleware can log requests, measure performance, and add profiling headers.
- **API Development:** Middleware validate API keys, enforce rate limits, and handle CORS for API endpoints.

### Core Concepts / Features

1. Creating Middleware
2. Dependency Injection
3. Request Checks & Redirection
4. Response Modification
5. Terminable Middleware
6. Excluding Middleware

---

## 1. Creating Middleware

### Definitions

**Core Definition:** Creating middleware is the process of generating a new middleware class via the Artisan CLI, which scaffolds a class implementing the `handle()` method that receives an HTTP request and a `$next` closure.

**Technical Definition:** The `php artisan make:middleware MiddlewareName` command generates a class in `app/Http/Middleware` that implements the `handle(Request $request, Closure $next): Response` method. The generated class serves as a template for request/response logic. Middleware are registered in `bootstrap/app.php` (Laravel 11+) via methods like `$middleware->alias()`, `$middleware->append()`, or `$middleware->prepend()`, or in `app/Http/Kernel.php` (Laravel 10 and earlier).

**Beginner-Friendly Explanation:** Laravel makes it easy to create middleware—just run one Artisan command and it generates a template file with all the boilerplate code already written. You fill in the logic inside the `handle()` method, register the middleware in the configuration file, and attach it to your routes. It's like getting a pre-built form that you just need to fill in with your specific rules.

### Purposes

- To scaffold middleware classes quickly with the correct structure and imports.
- To provide a consistent starting point for middleware implementation.
- To ensure middleware follow the standard `handle()` method signature.
- To integrate with Laravel's registration and alias system.

### Syntax Rules and Structure

**Complete General Syntax — Artisan Generation:**

```bash
php artisan make:middleware EnsureTokenIsValid
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `make:middleware` | The Artisan command to generate middleware. |
| `EnsureTokenIsValid` | The class name (creates `app/Http/Middleware/EnsureTokenIsValid.php`). |

**Complete General Syntax — Generated Middleware Class:**

```php
<?php

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

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `handle(Request $request, Closure $next): Response` | The middleware's entry point. |
| `$request` | The current HTTP request instance. |
| `$next` | The closure representing the next middleware in the pipeline. |
| `return $next($request)` | Passes the request to the next layer. |
| `return redirect('/home')` | Short-circuits the pipeline, returning a response directly. |

**Complete General Syntax — Registration (Laravel 11+):**

```php
// bootstrap/app.php

use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'token' => EnsureTokenIsValid::class,
    ]);
})
```

**Syntax Rules:**

- The middleware class name should describe its purpose (e.g., `EnsureTokenIsValid`, `CheckAdmin`, `LogRequests`).
- The `handle()` method must accept a `Request` instance and a `Closure` named `$next`.
- To pass the request deeper into the application, call `$next($request)` and return its result.
- To short-circuit the pipeline, return a response directly without calling `$next($request)`.
- Middleware are resolved from the service container, so constructor injection is supported.

**Constraints and Limitations:**

- **Namespace convention:** Middleware classes are placed in `App\Http\Middleware` by default.
- **Method signature:** The `handle()` method must return a `Response` instance or a value convertible to one.
- **Registration required:** Middleware must be registered in `bootstrap/app.php` (Laravel 11+) before they can be used.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Token Validation Middleware**

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
     */
    public function handle(Request $request, Closure $next): Response
    {
        // Check if the supplied token matches the expected value
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
// bootstrap/app.php

use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'token' => EnsureTokenIsValid::class,
    ]);
})
```

```php
<?php
// routes/web.php

Route::get('/secret', function () {
    return 'You have accessed the secret page!';
})->middleware('token');
```

**Expected Output:**
- `GET /secret?token=my-secret-token` → `You have accessed the secret page!`
- `GET /secret?token=wrong` → redirect to `/home`
- `GET /secret` → redirect to `/home`

**Why:** The middleware checks the `token` input against the expected value. If it matches, the request passes to the route handler. If not, the middleware short-circuits the pipeline and returns a redirect response.

---

**Example 2: Admin Check Middleware**

```php
<?php
// app/Http/Middleware/CheckAdmin.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user() || ! $request->user()->isAdmin()) {
            abort(403, 'Admin access required');
        }

        return $next($request);
    }
}
```

```php
<?php
// bootstrap/app.php

use App\Http\Middleware\CheckAdmin;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'admin' => CheckAdmin::class,
    ]);
})
```

```php
<?php
// routes/web.php

Route::get('/admin/users', [UserController::class, 'index'])->middleware(['auth', 'admin']);
```

**Expected Output:**
- Non-authenticated user → redirect to login
- Authenticated non-admin → 403 Forbidden
- Authenticated admin → user list

**Why:** The middleware first checks if the user is authenticated, then if they have admin privileges. It aborts with a 403 if either check fails, and passes the request to the next layer only when both checks pass.

### Real-World Cases

- **Admin Panels:** `CheckAdmin` middleware restricts access to administrative routes based on user roles.
- **API Key Validation:** `ValidateApiKey` middleware authenticates API consumers and attaches user context.
- **Subscription Services:** `EnsureSubscriptionIsActive` restricts premium content to paying users.
- **Multi-Tenancy:** `ResolveTenant` middleware identifies the tenant from the subdomain and scopes database queries.
- **Logging:** `LogRequests` middleware records request metadata for monitoring and auditing.

### References

- Laravel Middleware Documentation — https://laravel.com/docs/11.x/middleware
- Middleware in Laravel: Main Things to Know (Laravel Daily) — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- How to Configure Middleware in Laravel (Laravel News) — https://laravel-news.com/configuring-middleware-in-laravel

---

## 2. Dependency Injection

### Definitions

**Core Definition:** Dependency injection in middleware is the automatic resolution and injection of service class instances into middleware constructors or `handle()` method parameters, enabling middleware to use application services without manual instantiation.

**Technical Definition:** Laravel's service container resolves middleware through reflection. Dependencies type-hinted in the constructor are resolved when the middleware instance is created. Dependencies type-hinted in the `handle()` method are resolved at invocation time. The container recursively resolves dependencies and injects them. This enables middleware to receive services like `Guard`, repositories, SDK clients, and custom services without explicit instantiation.

**Beginner-Friendly Explanation:** Instead of creating service objects inside your middleware (like `new PaymentGateway()`), you just type-hint what you need in the constructor or method signature, and Laravel creates it for you and hands it over. This makes your middleware easier to test because you can swap out dependencies with mock objects.

### Purposes

- To automatically resolve service dependencies without manual instantiation.
- To improve testability by allowing dependencies to be mocked or swapped.
- To reduce boilerplate code by eliminating manual dependency creation.
- To leverage Laravel's service container for recursive dependency resolution.
- To decouple middleware from concrete implementations through interfaces.

### Syntax Rules and Structure

**Complete General Syntax — Constructor Injection:**

```php
<?php

namespace App\Http\Middleware;

use App\Services\AuditLogger;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class LogAdminActions
{
    public function __construct(
        private AuditLogger $logger
    ) {}

    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user()?->isAdmin()) {
            $this->logger->log('Admin action', [
                'url' => $request->fullUrl(),
            ]);
        }

        return $next($request);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `__construct(private AuditLogger $logger)` | Constructor type-hints the dependency. |
| `$this->logger->log(...)` | The injected dependency is used in `handle()`. |

**Complete General Syntax — Method Injection:**

```php
public function handle(Request $request, Closure $next, Guard $guard): Response
{
    // $guard is resolved from the container
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Guard $guard` | The dependency is injected into the `handle()` method. |
| Resolution order | Dependencies are resolved before middleware parameters. |

**Syntax Rules:**

- Constructor injection resolves dependencies when the middleware is instantiated.
- Method injection resolves dependencies per invocation.
- Middleware parameters (e.g., `role:admin`) are passed after injected dependencies in the `handle()` signature.
- Any class resolvable by the container can be type-hinted.
- Use constructor injection for dependencies used across multiple methods; use method injection for single-method dependencies.

**Constraints and Limitations:**

- **Constructor timing:** Constructor-injected dependencies are resolved before the request is fully available. Do not attempt to access request data in the constructor.
- **Parameter order:** In the `handle()` method, injected dependencies must come before middleware parameters.
- **Performance:** Constructor injection adds a slight overhead per middleware instance. Use method injection for dependencies used in only one method.

### Multiple Annotated Complete Code Examples

**Example 1: Constructor Injection with a Service**

```php
<?php
// app/Http/Middleware/LogApiRequests.php

namespace App\Http\Middleware;

use App\Services\ApiLogger;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class LogApiRequests
{
    public function __construct(
        private ApiLogger $logger
    ) {}

    public function handle(Request $request, Closure $next): Response
    {
        $this->logger->logRequest($request);

        $response = $next($request);

        $this->logger->logResponse($response);

        return $response;
    }
}
```

```php
<?php
// bootstrap/app.php

use App\Http\Middleware\LogApiRequests;

->withMiddleware(function (Middleware $middleware) {
    $middleware->api(append: [
        LogApiRequests::class,
    ]);
})
```

**Expected Output:** Every API request and response is logged by the injected `ApiLogger` service.

**Why:** The constructor type-hints `ApiLogger`. Laravel's container resolves it automatically and injects it. The service is stored as a property and used in the `handle()` method for both request and response logging.

---

**Example 2: Method Injection with a Guard**

```php
<?php
// app/Http/Middleware/CheckUserRole.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckUserRole
{
    public function handle(Request $request, Closure $next, Guard $guard, string $role): Response
    {
        // $guard is injected, $role is the middleware parameter
        if (! $guard->user()?->hasRole($role)) {
            abort(403, 'Insufficient permissions');
        }

        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

Route::get('/admin', function () {
    return 'Admin area';
})->middleware('role:admin');
```

**Expected Output:** The `/admin` route requires the user to have the `admin` role, checked via the injected `Guard` instance.

**Why:** The `Guard` dependency is method-injected before the middleware parameter `$role`. Laravel resolves the guard from the container and passes `admin` as the middleware parameter.

---

**Example 3: Constructor Injection with an Interface**

```php
<?php
// app/Contracts/PaymentGateway.php

namespace App\Contracts;

interface PaymentGateway
{
    public function charge(int $amount, string $token): bool;
}
```

```php
<?php
// app/Http/Middleware/ValidatePaymentToken.php

namespace App\Http\Middleware;

use App\Contracts\PaymentGateway;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ValidatePaymentToken
{
    public function __construct(
        private PaymentGateway $gateway
    ) {}

    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->has('payment_token')) {
            abort(422, 'Payment token required');
        }

        // Validate the token using the gateway (could be Stripe, PayPal, etc.)
        if (! $this->gateway->validateToken($request->input('payment_token'))) {
            abort(422, 'Invalid payment token');
        }

        return $next($request);
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

use App\Contracts\PaymentGateway;
use App\Services\StripePaymentGateway;

public function register(): void
{
    $this->app->bind(PaymentGateway::class, StripePaymentGateway::class);
}
```

**Expected Output:** The middleware validates the payment token using the bound gateway implementation.

**Why:** The constructor type-hints the `PaymentGateway` interface. The container resolves the bound implementation (`StripePaymentGateway`) and injects it. This allows the gateway to be swapped without modifying the middleware.

### Real-World Cases

- **Audit Logging:** Injecting an `AuditLogger` service to record admin actions.
- **Payment Validation:** Injecting a `PaymentGateway` to validate payment tokens before processing.
- **API Authentication:** Injecting a `Guard` to check user roles or permissions.
- **Tenant Resolution:** Injecting a `TenantRepository` to resolve the current tenant from the subdomain.
- **Feature Flags:** Injecting a `FeatureFlagService` to check if a feature is enabled for the current user.

### References

- Laravel Service Container Documentation — https://laravel.com/docs/12.x/container
- Middleware in Laravel: Main Things to Know (Laravel Daily) — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- Laravel Middleware Documentation — https://laravel.com/docs/11.x/middleware

---

## 3. Request Checks & Redirection

### Definitions

**Core Definition:** Request checks and redirection is the middleware capability to inspect incoming requests and either allow them to proceed, redirect them to a different URL, or abort them with an HTTP error response.

**Technical Definition:** The `handle()` method receives the `Illuminate\Http\Request` instance, which provides access to all request data. Middleware can inspect this data using methods like `$request->input()`, `$request->header()`, `$request->user()`, and `$request->isMethod()`. Based on inspection results, the middleware can call `$next($request)` to allow the request to proceed, return a redirect response via `redirect()` or `back()`, or throw an HTTP exception via `abort()`. The `abort()` helper throws an `HttpException` that Laravel's exception handler converts into an appropriate error response (403, 404, 419, 500, etc.).

**Beginner-Friendly Explanation:** Middleware can act like a bouncer at a club. It checks your ID (inspects the request), verifies you meet the dress code (filters based on conditions), and either lets you in (passes the request), redirects you to a different line (redirect response), or turns you away entirely (abort). The bouncer doesn't just check one thing—they can check multiple conditions and make a decision based on all of them.

### Purposes

- To inspect requests for authentication status via `$request->user()`.
- To filter requests based on roles, permissions, or other authorization criteria.
- To redirect unauthorized users to a login page or a relevant alternative.
- To abort requests with appropriate HTTP status codes (403, 404, 422).
- To validate API tokens or secret keys before allowing access.
- To log request metadata for monitoring and auditing purposes.

### Syntax Rules and Structure

**Complete General Syntax — Request Inspection with Redirect:**

```php
public function handle(Request $request, Closure $next): Response
{
    if (! $request->user()) {
        return redirect('/login');
    }

    if (! $request->user()->hasRole('admin')) {
        return redirect()->route('dashboard')
            ->with('error', 'Admin access required.');
    }

    return $next($request);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$request->user()` | Returns the authenticated user or `null`. |
| `redirect('/login')` | Returns a redirect response to the login page. |
| `->route('dashboard')` | Redirects to a named route. |
| `->with('error', '...')` | Flashes an error message to the session. |

**Complete General Syntax — Request Inspection with Abort:**

```php
public function handle(Request $request, Closure $next): Response
{
    if (! $request->hasHeader('X-API-Key')) {
        abort(401, 'API key required');
    }

    if (! $request->user()->isAdmin()) {
        abort(403, 'Admin access required');
    }

    return $next($request);
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `abort(401, '...')` | Throws a 401 Unauthorized exception. |
| `abort(403, '...')` | Throws a 403 Forbidden exception. |
| `abort(422, '...')` | Throws a 422 Unprocessable Entity exception. |

**Complete General Syntax — JSON Response for API:**

```php
public function handle(Request $request, Closure $next): Response
{
    if (! $request->bearerToken()) {
        return response()->json([
            'error' => 'Unauthenticated.',
        ], 401);
    }

    return $next($request);
}
```

**Syntax Rules:**

- `abort()` throws an `HttpException` that Laravel's exception handler converts to an error response.
- Returning a redirect response from `handle()` short-circuits the pipeline.
- `redirect()->route()` requires the route to be named.
- `back()` redirects to the previous URL (requires session middleware).
- For API routes, return JSON responses instead of redirects.

**Constraints and Limitations:**

- **Session dependency:** `back()` and `->with()` require session middleware. API routes do not have session support by default.
- **Redirect loops:** Careless redirect logic can cause infinite redirect loops.
- **Abort vs. redirect:** Use `abort()` for API requests and `redirect()` for web requests to provide appropriate responses for each client type.

### Multiple Annotated Complete Code Examples

**Example 1: Authentication Check with Redirect**

```php
<?php
// app/Http/Middleware/EnsureUserIsAuthenticated.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsAuthenticated
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()) {
            return redirect()->route('login')
                ->with('message', 'Please log in to continue.');
        }

        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

Route::get('/dashboard', function () {
    return 'Dashboard';
})->middleware(\App\Http\Middleware\EnsureUserIsAuthenticated::class);
```

**Expected Output:**
- Authenticated user → `Dashboard`
- Unauthenticated user → redirect to `/login` with flash message

**Why:** The middleware checks if a user is authenticated. If not, it redirects to the login route with a flash message. If yes, the request proceeds to the route handler.

---

**Example 2: API Key Validation with JSON Abort**

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
        $apiKey = $request->header('X-API-Key');

        if (! $apiKey) {
            return response()->json([
                'error' => 'API key is required.',
            ], 401);
        }

        $key = ApiKey::where('key', $apiKey)->first();

        if (! $key || $key->isExpired()) {
            return response()->json([
                'error' => 'Invalid or expired API key.',
            ], 401);
        }

        $request->merge(['api_user_id' => $key->user_id]);

        return $next($request);
    }
}
```

```php
<?php
// routes/api.php

Route::middleware([\App\Http\Middleware\ValidateApiKey::class])->group(function () {
    Route::get('/user', function (Request $request) {
        return response()->json(['user_id' => $request->input('api_user_id')]);
    });
});
```

**Expected Output:**
- Without `X-API-Key` header → `{"error": "API key is required."}` with 401
- With invalid key → `{"error": "Invalid or expired API key."}` with 401
- With valid key → `{"user_id": 1}`

**Why:** The middleware inspects the `X-API-Key` header, validates it against the database, and merges the associated user ID into the request for downstream use.

---

**Example 3: Role Check with Abort and Redirect**

```php
<?php
// app/Http/Middleware/CheckRole.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckRole
{
    public function handle(Request $request, Closure $next, string ...$roles): Response
    {
        if (! $request->user()) {
            return redirect()->route('login');
        }

        if (! $request->user()->hasAnyRole($roles)) {
            abort(403, 'Insufficient permissions');
        }

        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

Route::get('/admin', function () {
    return 'Admin area';
})->middleware(['auth', 'role:admin']);

Route::get('/editor', function () {
    return 'Editor area';
})->middleware(['auth', 'role:admin,editor']);
```

**Expected Output:**
- `/admin` → only users with the `admin` role
- `/editor` → users with either `admin` or `editor` role
- Unauthenticated → redirect to login
- Insufficient role → 403 Forbidden

**Why:** The middleware first checks authentication, then role permissions. It redirects unauthenticated users to login and aborts with 403 for insufficient permissions.

### Real-World Cases

- **Admin Panels:** Redirect non-admin users to the dashboard with an error message.
- **API Gateways:** Return JSON 401 responses for missing or invalid API keys.
- **Subscription Services:** Redirect users without active subscriptions to the billing page.
- **Multi-Tenancy:** Abort with 404 if the tenant subdomain does not exist.
- **Maintenance Mode:** Redirect all requests to a maintenance page during scheduled downtime.

### References

- Laravel Middleware Documentation — https://laravel.com/docs/11.x/middleware
- Middleware in Laravel: Main Things to Know (Laravel Daily) — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- Customizing Middleware Authorization Response (Laracasts) — https://laracasts.com/discuss/channels/laravel/customizing-middleware-authorization-response

---

## 4. Response Modification

### Definitions

**Core Definition:** Response modification is the middleware capability to alter the HTTP response—including headers, status codes, and body content—after the controller has generated it but before it is sent to the client.

**Technical Definition:** In middleware, the response is obtained by calling `$response = $next($request)`. After this call, the middleware can modify the response using methods like `$response->header()`, `$response->withHeaders()`, and `$response->setStatusCode()`. The modified response must be returned from the `handle()` method. The response body can be read and replaced using `$response->getContent()` and `$response->setContent()`. Terminable middleware's `terminate()` method runs after the response is sent and cannot modify the response.

**Beginner-Friendly Explanation:** After your controller generates a response—whether it's an HTML page, JSON data, or a redirect—middleware can still modify it before it reaches the user. Maybe you want to add a security header to every response, or compress the HTML to make it smaller, or log the response status. Response modification middleware does this work after the controller has finished but before the user sees the result.

### Purposes

- To add security headers (e.g., `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`) to every response.
- To add CORS headers for cross-origin API requests.
- To log response status codes, content types, and sizes for monitoring.
- To modify response content (e.g., minifying HTML, injecting scripts).
- To set cookies on the response for user tracking or session management.
- To add custom headers for debugging or observability (e.g., `X-Request-Duration`).

### Syntax Rules and Structure

**Complete General Syntax — After Middleware:**

```php
public function handle(Request $request, Closure $next): Response
{
    // Process the request and get the response
    $response = $next($request);

    // Modify the response
    $response->headers->set('X-Content-Type-Options', 'nosniff');
    $response->headers->set('X-Frame-Options', 'DENY');

    // Return the modified response
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

**Complete General Syntax — Modifying Response Content:**

```php
public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);

    // Get and modify the response body
    $content = $response->getContent();
    $modified = str_replace('</body>', '<script>/* injected */</script></body>', $content);
    $response->setContent($modified);

    return $response;
}
```

**Complete General Syntax — Adding Cookies:**

```php
public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);

    return $response->withCookie(
        cookie('tracking_id', 'abc123', 60)
    );
}
```

**Syntax Rules:**

- The response is obtained by calling `$response = $next($request)`.
- Modifications must occur after this call and before `return $response`.
- Use `$response->headers->set()` for Symfony-style header manipulation or `$response->header()` for Laravel's fluent API.
- `$response->getContent()` returns the response body as a string.
- `$response->setContent()` replaces the response body.
- Cookies are added via `$response->withCookie()`.

**Constraints and Limitations:**

- **Response body size:** Reading and modifying large response bodies can consume memory. Avoid full-body modifications for large responses.
- **Streamed responses:** Responses created via `streamDownload()` cannot have their content modified after creation.
- **Terminable middleware:** The `terminate()` method runs after the response is sent, so it cannot modify the response.
- **Order dependency:** Response modifications occur in reverse order of middleware execution.

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
        $response = $next($request);

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
// bootstrap/app.php (Laravel 11+)

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\App\Http\Middleware\AddSecurityHeaders::class);
})
```

**Expected Output:** Every response includes the headers `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, and `Permissions-Policy: geolocation=()`.

**Why:** The middleware calls `$next($request)` to get the response, then modifies its headers before returning it.

---

**Example 2: Adding a Request Duration Header**

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
        $startTime = microtime(true);

        $response = $next($request);

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
    usleep(50000); // Simulate 50ms of work
    return 'Hello World';
})->middleware(\App\Http\Middleware\MeasureRequestDuration::class);
```

**Expected Output:** `GET /test` returns `Hello World` with the header `X-Request-Duration: 50.12ms` (or similar).

**Why:** The middleware records the start time before calling `$next($request)` and calculates the duration after the response is generated. The duration is added as a response header.

---

**Example 3: Adding a Cookie to Every Response**

```php
<?php
// app/Http/Middleware/AddTrackingCookie.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AddTrackingCookie
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Add a tracking cookie if not already present
        if (! $request->hasCookie('tracking_id')) {
            $response->withCookie(
                cookie('tracking_id', uniqid(), 60 * 24 * 365)
            );
        }

        return $response;
    }
}
```

**Expected Output:** Every response includes a `tracking_id` cookie if the client does not already have one.

**Why:** The middleware checks if the client already has a `tracking_id` cookie. If not, it adds one to the response via `withCookie()`.

### Real-World Cases

- **Security Headers:** Adding `X-Frame-Options`, `X-Content-Type-Options`, and `Strict-Transport-Security` to every response.
- **CORS:** Adding `Access-Control-Allow-Origin` and related headers for cross-origin API requests.
- **Performance Monitoring:** Adding `X-Request-Duration` and `X-Memory-Peak` headers for profiling.
- **Content Minification:** Minifying HTML responses before sending them to the browser.
- **Cache Headers:** Setting `Cache-Control` and `ETag` headers based on response content.

### References

- Laravel Middleware: Middleware and Responses — https://laravel.com/docs/11.x/middleware#middleware-and-responses
- Laravel Middleware Documentation — https://laravel.com/docs/11.x/middleware
- Laravel News: Configuring Middleware in Laravel — https://laravel-news.com/configuring-middleware-in-laravel

---

## 5. Terminable Middleware

### Definitions

**Core Definition:** Terminable middleware is middleware that implements a `terminate(Request $request, Response $response): void` method, which is automatically called after the HTTP response has been sent to the client, allowing expensive or non-critical work to be performed without delaying the user's response.

**Technical Definition:** Terminable middleware implement the `Illuminate\Contracts\TerminableMiddleware` interface (or simply define a `terminate()` method). After the response is sent to the browser, Laravel's HTTP kernel calls the `terminate()` method on all registered terminable middleware, passing both the request and response instances. This method is called outside the middleware pipeline, so it cannot modify the response. The `terminate()` method is executed after the response has been sent to the client, and on servers using FastCGI (e.g., PHP-FPM), it runs after the connection is closed.

**Beginner-Friendly Explanation:** Terminable middleware let you do work *after* the user has already received their response. For example, you might want to log the request duration, send analytics data, or clean up temporary files—tasks that don't need to delay the user. The user gets their page quickly, and the cleanup happens in the background.

### Purposes

- To execute code after the HTTP response has been sent to the client, without delaying the user.
- To log request duration and response metadata for performance monitoring.
- To perform cleanup tasks (e.g., deleting temporary files, closing connections).
- To send analytics or monitoring data asynchronously.
- To write session data to storage after the response is prepared.
- To perform non-critical operations that should not block the response.

### Syntax Rules and Structure

**Complete General Syntax — Terminable Middleware Class:**

```php
<?php

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

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `handle()` | Captures the start time and passes the request through. |
| `terminate(Request $request, Response $response)` | Executes after the response is sent. |
| `$request->attributes->set()` | Stores data on the request for use in `terminate()`. |
| `$request->attributes->get()` | Retrieves the stored data in `terminate()`. |

**Complete General Syntax — Registration (Laravel 11+):**

```php
// bootstrap/app.php

use App\Http\Middleware\LogRequestDuration;

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(LogRequestDuration::class);
})
```

**Syntax Rules:**

- The `terminate()` method must be `public` and accept both `Request` and `Response` instances.
- The `handle()` method should store any data needed in `terminate()` on the request's `attributes` bag.
- Terminable middleware are automatically detected—no interface implementation is strictly required (though implementing `TerminableMiddleware` is recommended for clarity).
- The `terminate()` method runs after the response is sent, so it cannot modify the response.
- On FastCGI-based servers (PHP-FPM), `terminate()` runs after the connection to the client is closed.

**Constraints and Limitations:**

- **No response modification:** The `terminate()` method cannot change the response—it runs after the response has been sent.
- **Server dependency:** On servers using FastCGI (the standard for PHP), `terminate()` runs after the response is sent. On other server configurations, it may run before the connection is closed.
- **Error handling:** Exceptions thrown in `terminate()` are not handled by Laravel's exception handler. Wrap logic in try-catch blocks.
- **Testing:** Testing terminable middleware requires calling `$kernel->terminate()` manually in tests.

### Multiple Annotated Complete Code Examples

**Example 1: Logging Request Duration**

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
        $request->attributes->set('start_time', microtime(true));

        return $next($request);
    }

    public function terminate(Request $request, Response $response): void
    {
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

```php
<?php
// bootstrap/app.php

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\App\Http\Middleware\LogRequestDuration::class);
})
```

**Expected Output:** After the response is sent to the client, a log entry is written with the request URL, method, status code, and duration in milliseconds.

**Why:** The `handle()` method records the start time. The `terminate()` method calculates the duration and logs it after the response has been sent. The user does not experience any delay from the logging.

---

**Example 2: Writing Session Data After Response**

```php
<?php
// app/Http/Middleware/WriteSessionAfterResponse.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Session\SessionManager;
use Symfony\Component\HttpFoundation\Response;

class WriteSessionAfterResponse
{
    public function __construct(
        private SessionManager $session
    ) {}

    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    public function terminate(Request $request, Response $response): void
    {
        // Write session data to storage after the response is sent
        $this->session->save();
    }
}
```

**Expected Output:** Session data is written to storage after the response has been sent, improving perceived performance.

**Why:** The `terminate()` method calls `$this->session->save()` after the response is sent. This is similar to how Laravel's built-in `StartSession` middleware works.

---

**Example 3: Cleaning Up Temporary Files**

```php
<?php
// app/Http/Middleware/CleanUpTempFiles.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\File;
use Symfony\Component\HttpFoundation\Response;

class CleanUpTempFiles
{
    public function handle(Request $request, Closure $next): Response
    {
        // Mark the start of the request for temp file tracking
        $request->attributes->set('temp_file_start', now());

        return $next($request);
    }

    public function terminate(Request $request, Response $response): void
    {
        // Delete temp files created during this request
        $tempDir = storage_path('app/temp/' . $request->ip());

        if (File::isDirectory($tempDir)) {
            File::deleteDirectory($tempDir);
        }
    }
}
```

**Expected Output:** Temporary files created during the request are deleted after the response is sent.

**Why:** The `terminate()` method cleans up temporary files without delaying the user's response. This is useful for applications that generate temporary files during request processing.

### Real-World Cases

- **Performance Logging:** Log request duration and memory usage after the response is sent.
- **Session Management:** Write session data to storage after the response is prepared (like Laravel's built-in `StartSession`).
- **Analytics:** Send analytics events to external services after the response is delivered.
- **Cleanup:** Delete temporary files or close external connections after the request.
- **Audit Logging:** Record admin actions to an audit log after the response is sent.

### References

- Laravel Middleware: Terminable Middleware — https://laravel.com/docs/11.x/middleware#terminable-middleware
- Do some work after a response has been sent to the browser (Laravel Daily) — https://laraveldaily.com/tip/do-some-work-after-a-response-has-been-sent-to-the-browser
- Laravel API: TerminableMiddleware — https://api.laravel.com/docs/11.x/Illuminate/Contracts/TerminableMiddleware.html

---

## 6. Excluding Middleware

### Definitions

**Core Definition:** Excluding middleware is the capability to bypass specific middleware on individual routes or route groups using the `withoutMiddleware()` method, allowing certain routes to opt out of middleware that would otherwise apply.

**Technical Definition:** The `withoutMiddleware()` method is called on a `Route` instance and accepts an array of middleware class names or aliases to exclude. The excluded middleware are removed from the route's middleware stack during dispatch. This method cannot remove global middleware—only route-level and group-level middleware. The method can be chained with other route methods: `Route::get('/path', ...)->withoutMiddleware([...])`. It can also be used within a route group to exclude middleware from all routes in the group.

**Beginner-Friendly Explanation:** Sometimes you have middleware applied to a group of routes, but one specific route needs to bypass it. For example, you might have CSRF protection on all web routes, but a webhook endpoint needs to skip CSRF verification. The `withoutMiddleware()` method lets you say "this route is the exception—don't apply this middleware here."

### Purposes

- To bypass specific filters on individual routes (e.g., CSRF for webhooks).
- To exclude middleware from routes that have special requirements.
- To allow health-check endpoints to bypass authentication.
- To exclude rate limiting from internal API calls.
- To provide flexibility in middleware application without restructuring groups.

### Syntax Rules and Structure

**Complete General Syntax — Single Route:**

```php
Route::post('/webhook/stripe', [StripeWebhookController::class, 'handle'])
    ->withoutMiddleware([VerifyCsrfToken::class]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->withoutMiddleware([VerifyCsrfToken::class])` | Excludes the CSRF middleware from this route. |
| `[VerifyCsrfToken::class]` | Array of middleware to exclude. |

**Complete General Syntax — Route Group:**

```php
Route::middleware(['auth', 'throttle:api'])->group(function () {
    Route::get('/user', [UserController::class, 'show']);

    // This route bypasses both auth and throttle
    Route::get('/health', function () {
        return 'OK';
    })->withoutMiddleware(['auth', 'throttle:api']);
});
```

**Complete General Syntax — Excluding by Alias:**

```php
Route::get('/public', function () {
    return 'Public page';
})->withoutMiddleware(['auth']);
```

**Complete General Syntax — Excluding from Group:**

```php
Route::withoutMiddleware(['throttle:api'])->group(function () {
    Route::get('/internal/data', [InternalController::class, 'data']);
});
```

**Syntax Rules:**

- `withoutMiddleware()` accepts an array of middleware class names or aliases.
- The method can be chained after any route definition.
- It can be used on individual routes or on route groups.
- Global middleware cannot be excluded via `withoutMiddleware()`.
- The method returns the route instance, allowing further chaining.

**Constraints and Limitations:**

- **Global middleware:** `withoutMiddleware()` cannot remove global middleware—only route and group middleware.
- **Order dependency:** If the middleware is applied in a group and the route is within that group, the exclusion works. If the middleware is applied globally, exclusion does not work.
- **Named routes:** `withoutMiddleware()` can be used on named routes without affecting the route name.
- **Performance:** Excluding middleware removes its processing overhead, which can improve performance for the excluded route.

### Multiple Annotated Complete Code Examples

**Example 1: Excluding CSRF from a Webhook Route**

```php
<?php
// routes/web.php

use App\Http\Controllers\StripeWebhookController;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;
use Illuminate\Support\Facades\Route;

// Stripe webhook — excludes CSRF verification
Route::post('/webhook/stripe', [StripeWebhookController::class, 'handle'])
    ->withoutMiddleware([VerifyCsrfToken::class])
    ->name('webhook.stripe');
```

**Expected Output:** `POST /webhook/stripe` accepts requests without a CSRF token, allowing Stripe to send webhook payloads.

**Why:** External services cannot generate CSRF tokens. The `withoutMiddleware()` method excludes CSRF verification from this specific route, allowing the webhook to be processed.

---

**Example 2: Excluding Authentication from a Health Check**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', function () {
        return 'Dashboard';
    });

    // Health check bypasses authentication
    Route::get('/health', function () {
        return response()->json(['status' => 'ok']);
    })->withoutMiddleware(['auth']);
});
```

**Expected Output:**
- `/dashboard` → requires authentication
- `/health` → accessible without authentication

**Why:** The `/health` route is within the `auth` middleware group but excludes authentication, allowing monitoring services to check application health without credentials.

---

**Example 3: Excluding Multiple Middleware**

```php
<?php
// routes/api.php

use App\Http\Middleware\ValidateApiKey;
use Illuminate\Support\Facades\Route;

Route::middleware([ValidateApiKey::class, 'throttle:api'])->group(function () {
    Route::get('/data', function () {
        return response()->json(['data' => 'example']);
    });

    // Internal endpoint bypasses API key validation and rate limiting
    Route::get('/internal/status', function () {
        return response()->json(['status' => 'healthy']);
    })->withoutMiddleware([ValidateApiKey::class, 'throttle:api']);
});
```

**Expected Output:**
- `/api/data` → requires API key and is rate limited
- `/api/internal/status` → no API key or rate limiting required

**Why:** The `withoutMiddleware()` method accepts multiple middleware, allowing the internal status endpoint to bypass both validation and rate limiting.

### Real-World Cases

- **Webhooks:** Stripe, GitHub, and other webhook providers require CSRF exclusion.
- **Health Checks:** Monitoring endpoints bypass authentication for load balancer health checks.
- **Internal APIs:** Internal service-to-service calls bypass rate limiting.
- **Public Pages:** Specific public pages within an authenticated area bypass authentication.
- **Testing:** Test routes bypass middleware during development.

### References

- Laravel Middleware: Excluding Middleware — https://laravel.com/docs/11.x/middleware#excluding-middleware
- Exclude middleware from a route (Laravel Daily) — https://laraveldaily.com/tip/exclude-middleware-from-a-route
- How to bypass middleware in Laravel with route groups (LinkedIn) — https://www.linkedin.com/posts/mohamed-salman

---

## References

- Laravel Middleware Documentation (11.x) — https://laravel.com/docs/11.x/middleware
- Laravel Service Container Documentation — https://laravel.com/docs/12.x/container
- Middleware in Laravel: Main Things to Know (Laravel Daily) — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- How to Configure Middleware in Laravel (Laravel News) — https://laravel-news.com/configuring-middleware-in-laravel
- Parameterized Middleware in Laravel (Laravel News) — https://laravel-news.com/parameterized-middleware
- Do some work after a response has been sent to the browser (Laravel Daily) — https://laraveldaily.com/tip/do-some-work-after-a-response-has-been-sent-to-the-browser
- Exclude middleware from a route (Laravel Daily) — https://laraveldaily.com/tip/exclude-middleware-from-a-route
- Customizing Middleware Authorization Response (Laracasts) — https://laracasts.com/discuss/channels/laravel/customizing-middleware-authorization-response
- Laravel API: TerminableMiddleware — https://api.laravel.com/docs/11.x/Illuminate/Contracts/TerminableMiddleware.html
- timacdonald/has-parameters (Packagist) — https://packagist.org/packages/timacdonald/has-parameters
- GitHub: agent-config laravel-middleware SKILL.md — https://github.com/event4u-app/agent-config
- Laravel Middleware Patterns (GitHub) — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/iserter/laravel-claude-agents/laravel-middleware-patterns/SKILL.md