# Laravel Middleware Types & Core Features: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel middleware types are the categorised set of request/response filters that Laravel provides out of the box—covering authentication, authorization, CSRF protection, rate limiting, CORS, cookie encryption, logging, and profiling—alongside the framework for defining custom application middleware.

**Technical Definition:** Laravel's middleware ecosystem consists of three tiers: (1) **Built-in global/group middleware** shipped with the framework and registered in the default `web` and `api` middleware groups, (2) **Logging and profiling middleware** that capture request/response metadata for observability, and (3) **Custom application middleware** generated via `php artisan make:middleware` and registered in `bootstrap/app.php` (Laravel 11+) or `app/Http/Kernel.php` (Laravel 10 and earlier). Each middleware implements `handle(Request $request, Closure $next): Response`, and they are composed into a pipeline that processes requests in registration order and responses in reverse order.

**Beginner-Friendly Explanation:** Laravel comes with a set of ready-made middleware that handle common security and infrastructure tasks—like checking if you're logged in, verifying you have permission to do something, protecting against form forgery, limiting how many requests you can make, and handling cross-origin requests. You can also create your own middleware for application-specific needs like logging or custom validation. All middleware work the same way: they sit between the request and your controller, and between the response and the user.

### Key Characteristics

- **Layered Security:** Built-in middleware provide defence-in-depth against common web vulnerabilities (CSRF, brute force, unauthorised access).
- **Configuration Centralisation:** Since Laravel 11, all middleware configuration is consolidated in `bootstrap/app.php` via the `withMiddleware()` method.
- **Alias System:** Built-in middleware are registered with short aliases (`auth`, `can`, `throttle`, `verified`, `guest`) for expressive route definitions.
- **Group Assignment:** Middleware are assigned to `web` and `api` groups by default, with different middleware stacks for stateful web routes and stateless API routes.
- **Extensibility:** Custom middleware follow the same contract and can be registered globally, per-group, or per-route.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (middleware configuration differs between versions; Laravel 11+ uses `bootstrap/app.php`).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `bootstrap/app.php` file (Laravel 11+) or `app/Http/Kernel.php` (Laravel 10).
- Basic understanding of the HTTP request lifecycle and the Pipeline pattern.

### Related Programming Areas

- **Authentication & Authorization:** The `auth` and `can` middleware integrate with Laravel's guard and gate systems.
- **Security:** CSRF protection, cookie encryption, and rate limiting are security-critical middleware.
- **API Development:** CORS handling and rate limiting are essential for API endpoints.
- **Observability:** Logging and profiling middleware provide visibility into request handling.
- **Pipeline Pattern:** All middleware leverage Laravel's `Illuminate\Pipeline\Pipeline` class.

### Core Concepts / Features

1. Authentication (`auth`)
2. Authorization / Gates (`can`)
3. CSRF Protection (`ValidateCsrfToken`)
4. Rate Limiting (`throttle`)
5. CORS Handling (`HandleCors`)
6. Cookie Encryption (`EncryptCookies`)
7. Logging & Profiling Middleware
8. Custom Application Middleware

---

## 1. Authentication (`auth`)

### Definitions

**Core Definition:** The `auth` middleware verifies that the incoming request is from an authenticated user, redirecting unauthenticated users to the login page (or returning a 401 JSON response for API requests).

**Technical Definition:** The `Illuminate\Auth\Middleware\Authenticate` middleware receives the request and checks each configured authentication guard to determine if a user is authenticated. If no guard returns an authenticated user, the middleware throws an `AuthenticationException`, which Laravel's exception handler converts into a redirect to the `login` route (for web requests) or a 401 JSON response (for requests expecting JSON). The middleware can be parameterised with specific guards: `auth:api`, `auth:sanctum`.

**Beginner-Friendly Explanation:** The `auth` middleware is like a security guard at the door of a members-only club. It checks if you have a valid membership (are you logged in?). If you do, it lets you in. If not, it sends you to the sign-up desk (the login page). Without this middleware, anyone could access protected pages.

### Purposes

- To protect routes that require an authenticated user before access is granted.
- To redirect unauthenticated web users to the login page automatically.
- To return a 401 JSON response for unauthenticated API requests.
- To support multiple authentication guards (session, token, Sanctum, Passport).
- To integrate with Laravel's guard and provider system for flexible authentication.

### Syntax Rules and Structure

**Complete General Syntax — Route-Level Application:**

```php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->middleware('auth')` | Applies the `auth` middleware alias. |
| `auth` | Alias for `Illuminate\Auth\Middleware\Authenticate`. |

**Complete General Syntax — Specific Guard:**

```php
Route::get('/api/user', function () {
    return $request->user();
})->middleware('auth:sanctum');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `auth:sanctum` | Authenticates using the Sanctum guard. |
| `auth:api` | Authenticates using the API guard. |

**Complete General Syntax — Middleware Group:**

```php
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::get('/settings', [SettingsController::class, 'index']);
});
```

**Syntax Rules:**

- The `auth` middleware alias is registered by default in Laravel's middleware configuration.
- Guard names are passed as parameters after a colon: `auth:api`, `auth:sanctum`.
- The `redirectTo()` method on the `Authenticate` middleware determines the redirect target for unauthenticated web requests. By default, it redirects to the `login` named route.
- For API routes, the middleware returns a 401 JSON response instead of a redirect.
- In Laravel 11+, the `Authenticate` middleware is defined in the framework and can be customised via `$middleware->redirectGuestsTo()` in `bootstrap/app.php`.

**Constraints and Limitations:**

- **Session dependency:** The `auth` middleware for web routes requires session middleware to be active. API routes use token-based guards without sessions.
- **Guard configuration:** The `auth` middleware uses the default guard unless a specific guard is provided. Ensure `config/auth.php` is configured correctly.
- **Redirect route:** The default redirect target is the `login` named route. If your login route has a different name, you must customise the redirect behaviour.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Auth Middleware on a Route**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/dashboard', function () {
    return 'Welcome to your dashboard, ' . auth()->user()->name;
})->middleware('auth');
```

**Expected Output:**
- Authenticated user visiting `/dashboard` → `Welcome to your dashboard, Alice`
- Unauthenticated user → redirect to `/login`

**Why:** The `auth` middleware checks if a user is authenticated via the session guard. If not, it redirects to the login page.

---

**Example 2: Auth Middleware on a Group with Multiple Guards**

```php
<?php
// routes/api.php

use App\Http\Controllers\Api\UserController;
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', [UserController::class, 'show']);
    Route::put('/user', [UserController::class, 'update']);
});
```

**Expected Output:**
- Request with valid Sanctum token → returns user data
- Request without token → `{"message": "Unauthenticated."}` with 401

**Why:** The `auth:sanctum` middleware uses the Sanctum guard to authenticate API requests via bearer tokens. Unlike web routes, it does not redirect—it returns a 401 JSON response.

---

**Example 3: Customising the Redirect for Unauthenticated Users**

```php
<?php
// bootstrap/app.php (Laravel 11+)

use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function ($middleware) {
        // Redirect unauthenticated users to a custom route
        $middleware->redirectGuestsTo('/custom-login');
    })
    ->create();
```

**Expected Output:** Unauthenticated web requests are redirected to `/custom-login` instead of the default `login` route.

**Why:** The `redirectGuestsTo()` method on the middleware configuration object customises the redirect target for the `auth` middleware.

### Real-World Cases

- **Admin Panels:** Protect all admin routes with `auth` and `can` middleware.
- **User Dashboards:** Require authentication for dashboard, profile, and settings pages.
- **API Endpoints:** Use `auth:sanctum` to protect API routes that require a valid token.
- **Multi-Guard Applications:** Use `auth:admin` and `auth:web` for separate admin and customer authentication.

### References

- Laravel Middleware: Authentication Middleware — https://laravel.com/docs/12.x/middleware
- Laravel Authentication Documentation — https://laravel.com/docs/12.x/authentication
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know

---

## 2. Authorization / Gates (`can`)

### Definitions

**Core Definition:** The `can` middleware authorizes an action against a gate or policy before the request reaches the route handler, returning a 403 Forbidden response if the user is not authorized.

**Technical Definition:** The `Illuminate\Auth\Middleware\Authorize` middleware is registered with the `can` alias. It accepts parameters in the format `can:ability,model` (e.g., `can:update,post`). The middleware resolves the ability and model, invokes the corresponding gate or policy method, and either allows the request to proceed or throws an `AuthorizationException` (converted to a 403 response by Laravel's exception handler). Gates are defined via `Gate::define()` and policies are resolved automatically based on the model class.

**Beginner-Friendly Explanation:** The `can` middleware checks whether the logged-in user has permission to do a specific thing—like edit a post or delete a comment. Instead of writing the permission check inside your controller, you attach it to the route. If the user doesn't have permission, they get a 403 "Forbidden" page. It's like a bouncer who checks not just if you're on the list, but whether you're allowed into the VIP section.

### Purposes

- To authorize actions against gates or policies before the request reaches the controller.
- To keep authorization logic out of controllers and in dedicated gate or policy classes.
- To return a 403 Forbidden response for unauthorized actions.
- To support both simple gates (closures) and complex policies (class-based).
- To enable parameterised authorization checks (`can:update,post`).

### Syntax Rules and Structure

**Complete General Syntax — Gate Authorization:**

```php
Route::get('/admin', [AdminController::class, 'dashboard'])
    ->middleware('can:access-admin-panel');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `can:access-admin-panel` | Checks the `access-admin-panel` gate. |
| No model | The gate receives only the user. |

**Complete General Syntax — Policy Authorization:**

```php
Route::get('/posts/{post}/edit', [PostController::class, 'edit'])
    ->middleware('can:update,post');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `can:update,post` | Checks the `update` policy method for the `post` route parameter. |
| `post` | The route parameter resolved to a model instance. |

**Complete General Syntax — Multiple Middleware:**

```php
Route::middleware(['auth', 'can:access-admin-panel'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard']);
});
```

**Syntax Rules:**

- The `can` middleware requires an authenticated user. Combine it with `auth` for web routes.
- Parameters are comma-separated: `can:ability,model`.
- The model parameter is resolved from route model binding (the parameter name must match a route parameter).
- Gates are defined in `AppServiceProvider::boot()` using `Gate::define()`.
- Policies are auto-discovered based on naming conventions (`Post` model → `PostPolicy`).

**Constraints and Limitations:**

- **Authentication required:** The `can` middleware does not authenticate the user—it assumes a user is already authenticated. Combine with `auth` or `auth:sanctum`.
- **Model resolution:** If the model parameter is provided, it must be resolvable from the route parameters. Otherwise, a 404 or binding error occurs.
- **Response customisation:** By default, unauthorized requests return a 403 response. To customise, override the `AuthorizationException` handling in `bootstrap/app.php`.

### Multiple Annotated Complete Code Examples

**Example 1: Gate-Based Authorization**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Gate::define('access-admin-panel', function (User $user) {
            return $user->isAdmin();
        });
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\AdminController;
use Illuminate\Support\Facades\Route;

Route::get('/admin', [AdminController::class, 'dashboard'])
    ->middleware(['auth', 'can:access-admin-panel']);
```

**Expected Output:**
- Admin user → dashboard is displayed.
- Non-admin user → 403 Forbidden.

**Why:** The gate closure receives the authenticated user and returns `true` if they are an admin. The `can` middleware invokes this gate and either allows or denies the request.

---

**Example 2: Policy-Based Authorization with Model**

```php
<?php
// app/Policies/PostPolicy.php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

Route::get('/posts/{post}/edit', [PostController::class, 'edit'])
    ->middleware(['auth', 'can:update,post']);
```

**Expected Output:**
- Post author → edit form is displayed.
- Other users → 403 Forbidden.

**Why:** The `can:update,post` middleware resolves the `{post}` route parameter to a `Post` model, finds the `PostPolicy`, and calls the `update` method with the authenticated user and the post.

---

**Example 3: Multiple Authorization Checks**

```php
<?php
// routes/web.php

use App\Http\Controllers\AdminController;
use Illuminate\Support\Facades\Route;

Route::middleware(['auth', 'can:access-admin-panel'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard']);
    Route::get('/admin/settings', [AdminController::class, 'settings']);
});
```

**Expected Output:** All routes in the group require both authentication and the `access-admin-panel` permission.

**Why:** The middleware array applies both `auth` and `can` to every route in the group. The `auth` middleware runs first, then `can`.

### Real-World Cases

- **Admin Panels:** `can:access-admin-panel` gates entry to the entire admin area.
- **Resource Editing:** `can:update,post` ensures only the post's author can edit it.
- **Deletion Protection:** `can:delete,comment` prevents users from deleting others' comments.
- **Team-Based Access:** Policies check team membership before allowing access to team resources.

### References

- Laravel Authorization Documentation — https://laravel.com/docs/12.x/authorization
- Laravel Middleware: Authorization Middleware — https://laravel.com/docs/12.x/middleware
- Laravel Policies: Middleware Authorization — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-07-02-laravel-policies-authorization/README.md

---

## 3. CSRF Protection (`ValidateCsrfToken`)

### Definitions

**Core Definition:** The `ValidateCsrfToken` middleware protects against Cross-Site Request Forgery (CSRF) attacks by verifying that state-changing requests (POST, PUT, PATCH, DELETE) include a valid CSRF token matching the one stored in the user's session.

**Technical Definition:** The `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` middleware (formerly `VerifyCsrfToken`) is included in the `web` middleware group by default. It checks the request's `_token` input field or `X-CSRF-TOKEN` header against the token stored in the session. If they do not match, it throws a `TokenMismatchException` (converted to a 419 response). Starting with Laravel 11, the middleware is no longer part of the application skeleton—it is configured via the `validateCsrfTokens()` method in `bootstrap/app.php`.

**Beginner-Friendly Explanation:** CSRF protection prevents malicious websites from tricking you into submitting forms on another site you're logged into. For example, a bad website could have a hidden form that submits a request to your bank's website to transfer money. Laravel's CSRF middleware stops this by requiring a secret token that only your real website knows. Every form you create in Laravel automatically includes this token.

### Purposes

- To protect against Cross-Site Request Forgery attacks on state-changing requests.
- To verify that POST, PUT, PATCH, and DELETE requests originate from the application's own forms.
- To automatically include CSRF tokens in Blade forms via the `@csrf` directive.
- To allow specific URIs (e.g., webhooks) to bypass CSRF verification.
- To integrate with JavaScript frameworks via the `X-CSRF-TOKEN` header.

### Syntax Rules and Structure

**Complete General Syntax — Blade Form Directive:**

```blade
<form method="POST" action="/profile">
    @csrf
    <!-- Other form fields -->
</form>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@csrf` | Blade directive that generates a hidden `_token` input field. |

**Complete General Syntax — Excluding URIs (Laravel 11+):**

```php
// bootstrap/app.php

->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
        'webhook/github',
    ]);
})
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `validateCsrfTokens(except: [...])` | Excludes the specified URIs from CSRF verification. |
| `'stripe/*'` | Wildcard pattern for Stripe webhooks. |

**Complete General Syntax — JavaScript Header:**

```javascript
// In your JavaScript (e.g., Axios or fetch)
headers: {
    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content
}
```

**Syntax Rules:**

- The `@csrf` directive must be included in every POST, PUT, PATCH, and DELETE HTML form.
- The CSRF token is stored in the session and changes when the session regenerates.
- The `validateCsrfTokens()` method accepts an `except` array of URI patterns to exclude.
- JavaScript frameworks typically read the token from a `<meta name="csrf-token">` tag.
- API routes (without the `web` middleware group) do not have CSRF protection by default.

**Constraints and Limitations:**

- **Session dependency:** CSRF protection requires session middleware. API routes that are stateless do not have CSRF protection.
- **Token regeneration:** The CSRF token regenerates on login and logout. Forms rendered before login may have stale tokens.
- **Webhooks:** External webhooks (Stripe, GitHub) cannot include CSRF tokens and must be excluded from verification.
- **SPA applications:** SPAs using Sanctum should use the `EnsureFrontendRequestsAreStateful` middleware for CSRF protection.

### Multiple Annotated Complete Code Examples

**Example 1: CSRF Protection in a Blade Form**

```blade
{{-- resources/views/profile/edit.blade.php --}}
<form method="POST" action="{{ route('profile.update') }}">
    @csrf
    @method('PUT')

    <label for="name">Name</label>
    <input type="text" name="name" id="name" value="{{ old('name', $user->name) }}">

    <button type="submit">Update Profile</button>
</form>
```

**Expected Output:** The form includes a hidden `_token` input with the CSRF token. Submitting the form without the token returns a 419 Page Expired error.

**Why:** The `@csrf` directive generates `<input type="hidden" name="_token" value="...">`. The `ValidateCsrfToken` middleware compares this token with the session token.

---

**Example 2: Excluding Webhook URIs from CSRF**

```php
<?php
// bootstrap/app.php (Laravel 11+)

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->validateCsrfTokens(except: [
            'stripe/*',
            'webhook/github',
        ]);
    })
    ->create();
```

**Expected Output:** POST requests to `/stripe/webhook` and `/webhook/github` bypass CSRF verification. All other POST requests require a valid token.

**Why:** External services cannot generate CSRF tokens. The `except` array excludes their webhook endpoints from verification.

---

**Example 3: CSRF Token in JavaScript**

```javascript
// resources/js/bootstrap.js

// Read the CSRF token from the meta tag
const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

// Configure Axios to include the token in all requests
window.axios.defaults.headers.common['X-CSRF-TOKEN'] = csrfToken;
```

```blade
{{-- resources/views/layouts/app.blade.php --}}
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
</head>
```

**Expected Output:** All AJAX requests made via Axios include the `X-CSRF-TOKEN` header, allowing them to pass CSRF verification.

**Why:** The `X-CSRF-TOKEN` header is one of the ways the `ValidateCsrfToken` middleware reads the token. The meta tag provides a convenient way to access it in JavaScript.

### Real-World Cases

- **All Web Forms:** Every POST form in a Laravel application includes `@csrf` for protection.
- **Webhook Endpoints:** Stripe, GitHub, and other webhook providers require CSRF exclusion.
- **SPA Applications:** Sanctum's stateful middleware provides CSRF protection for SPAs.
- **API Tokens:** Stateless API routes using token authentication do not need CSRF protection.

### References

- Laravel CSRF Protection Documentation — https://laravel.com/docs/12.x/csrf
- Laravel 11: CSRF Configuration Changes — https://stackoverflow.com/revisions/ef8af48e-61c8-465a-b27d-60bb6491d503/view-source

---

## 4. Rate Limiting (`throttle`)

### Definitions

**Core Definition:** The `throttle` middleware limits the number of requests a client can make to a route or group of routes within a specified time window, protecting against brute-force attacks and API abuse.

**Technical Definition:** The `Illuminate\Routing\Middleware\ThrottleRequests` middleware uses Laravel's `RateLimiter` facade to track request counts. Rate limiters are defined via `RateLimiter::for()` in a service provider, accepting a name and a closure returning a `Limit` instance. The `throttle` middleware accepts either a limiter name (`throttle:api`) or inline parameters (`throttle:60,1` for 60 requests per minute). When the limit is exceeded, a 429 Too Many Requests response is returned with `Retry-After` and `X-RateLimit-*` headers.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer who says "you can only come in 60 times per minute." If you try to come in more often, you're turned away with a "429 Too Many Requests" message. This prevents bots from overwhelming your application with repeated requests—like trying to guess passwords or scrape data.

### Purposes

- To restrict brute-force traffic spikes on authentication endpoints.
- To protect API endpoints from abuse by limiting request frequency.
- To enforce fair usage policies across different user tiers (free vs. paid).
- To prevent denial-of-service (DoS) attacks by capping request rates.
- To provide per-user and per-IP rate limiting with dynamic limit calculation.
- To comply with third-party API rate limits.

### Syntax Rules and Structure

**Complete General Syntax — Named Rate Limiter:**

```php
// In AppServiceProvider::boot()
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

```php
// In routes
Route::middleware(['throttle:api'])->group(function () {
    Route::get('/user', function () { /* ... */ });
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `RateLimiter::for('api', ...)` | Registers a rate limiter named `api`. |
| `Limit::perMinute(60)` | Allows 60 requests per minute. |
| `->by(...)` | Sets the key for tracking (user ID or IP). |
| `throttle:api` | Applies the `api` rate limiter. |

**Complete General Syntax — Inline Throttle:**

```php
Route::get('/profile', function () {
    return 'Profile';
})->middleware('throttle:60,1');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `throttle:60,1` | 60 requests per 1 minute. |

**Complete General Syntax — Dynamic Rate Limiting:**

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->isPremium()
        ? Limit::none()
        : Limit::perMinute(10)->by($request->user()->id);
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Limit::none()` | No limit for premium users. |
| `Limit::perMinute(10)` | 10 requests per minute for free users. |

**Syntax Rules:**

- Rate limiters must be defined in a service provider's `boot()` method.
- The `throttle` middleware accepts a limiter name or inline `limit,minutes` parameters.
- The `by()` method determines the tracking key—if omitted, the authenticated user ID or IP is used.
- Laravel 11+ supports per-second rate limiting via `Limit::perSecond()`.
- Exceeded limits return HTTP 429 with `Retry-After` and `X-RateLimit-*` headers.

**Constraints and Limitations:**

- **Cache dependency:** Rate limiting requires a cache driver. Redis is recommended for distributed applications.
- **Middleware order:** Rate limiting should run before authentication when limiting by IP, or after when limiting by user ID.
- **Named limiters vs. inline:** Named limiters are more configurable and reusable; inline parameters are simpler but less flexible.
- **API group:** API routes have a default `throttle:api` limiter applied by default.

### Multiple Annotated Complete Code Examples

**Example 1: Basic API Rate Limiting**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        RateLimiter::for('api', function (Request $request) {
            return Limit::perMinute(60)
                ->by($request->user()?->id ?: $request->ip());
        });
    }
}
```

```php
<?php
// routes/api.php

Route::middleware(['throttle:api'])->group(function () {
    Route::get('/user', function (Request $request) {
        return $request->user();
    });
});
```

**Expected Output:**
- Authenticated users: 60 requests per minute per user ID.
- Guests: 60 requests per minute per IP address.
- 61st request: HTTP 429 with `Retry-After: 60`.

**Why:** The `RateLimiter::for('api', ...)` defines the limiter. The `throttle:api` middleware applies it to the route group.

---

**Example 2: Dynamic Rate Limiting for Premium Users**

```php
<?php
// app/Providers/AppServiceProvider.php

RateLimiter::for('downloads', function (Request $request) {
    $user = $request->user();

    if ($user && $user->isPremium()) {
        return Limit::none();
    }

    if ($user) {
        return Limit::perHour(5)->by($user->id);
    }

    return Limit::perHour(1)->by($request->ip());
});
```

```php
<?php
// routes/web.php

Route::get('/download/{file}', function (string $file) {
    return response()->download(storage_path("app/files/{$file}"));
})->middleware('throttle:downloads');
```

**Expected Output:**
- Premium user: unlimited downloads.
- Free user: 5 downloads per hour.
- Guest: 1 download per hour.

**Why:** The rate limiter closure returns different `Limit` instances based on the user's subscription tier. `Limit::none()` disables limiting for premium users.

---

**Example 3: Per-Second Rate Limiting (Laravel 11+)**

```php
<?php
// app/Providers/AppServiceProvider.php

RateLimiter::for('high-frequency', function (Request $request) {
    return Limit::perSecond(10)->by($request->ip());
});
```

```php
<?php
// routes/api.php

Route::middleware(['throttle:high-frequency'])->group(function () {
    Route::post('/webhook', function () {
        return response()->json(['status' => 'received']);
    });
});
```

**Expected Output:** The webhook endpoint accepts up to 10 requests per second from a single IP. The 11th request in the same second returns HTTP 429.

**Why:** Laravel 11 introduced `Limit::perSecond()` for high-frequency endpoints like webhooks.

### Real-World Cases

- **Login Endpoints:** `throttle:5,1` (5 attempts per minute) prevents brute-force password attacks.
- **API Endpoints:** `throttle:api` with 60 requests per minute is a standard API rate limit.
- **Webhook Receivers:** Per-second rate limiting handles high-frequency webhook traffic.
- **File Downloads:** Dynamic rate limiting gives premium users unlimited downloads while capping free users.
- **Password Reset:** `throttle:3,1` (3 attempts per minute) prevents abuse of password reset endpoints.

### References

- Laravel Routing: Rate Limiting — https://laravel.com/docs/12.x/routing#rate-limiting
- Laravel Throttle & Limit Guide — https://www.cloudways.com/blog/laravel-rate-limiting-throttle/
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know

---

## 5. CORS Handling (`HandleCors`)

### Definitions

**Core Definition:** The `HandleCors` middleware manages Cross-Origin Resource Sharing (CORS) headers, allowing or restricting browser-based requests from different origins to the application's API endpoints.

**Technical Definition:** The `Illuminate\Http\Middleware\HandleCors` middleware reads the `config/cors.php` configuration file to determine allowed origins, methods, headers, and credentials. It adds the appropriate `Access-Control-Allow-*` headers to responses and handles preflight OPTIONS requests automatically. In Laravel 11+, the middleware is registered globally via `$middleware->append(\Illuminate\Http\Middleware\HandleCors::class)` in `bootstrap/app.php`.

**Beginner-Friendly Explanation:** CORS is a browser security feature that blocks JavaScript from making requests to a different domain unless that domain explicitly allows it. The `HandleCors` middleware tells the browser "it's okay for these other websites to access our API." Without it, your React or Vue front-end running on a different port (or domain) couldn't communicate with your Laravel backend.

### Purposes

- To allow or restrict cross-origin requests based on the application's configuration.
- To handle preflight OPTIONS requests automatically.
- To add `Access-Control-Allow-Origin` and related headers to responses.
- To support credentials (cookies, authorization headers) in cross-origin requests.
- To comply with browser CORS policies for SPA and mobile app backends.

### Syntax Rules and Structure

**Complete General Syntax — Global Registration (Laravel 11+):**

```php
// bootstrap/app.php

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\Illuminate\Http\Middleware\HandleCors::class);
})
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$middleware->append(...)` | Adds the CORS middleware to the global stack. |

**Complete General Syntax — Configuration (`config/cors.php`):**

```php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['https://example.com'],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => false,
];
```

**Component Breakdown:**

| Key | Description |
|-----|-------------|
| `paths` | URIs that should have CORS headers applied. |
| `allowed_methods` | HTTP methods allowed for cross-origin requests. |
| `allowed_origins` | Origins allowed to make cross-origin requests. |
| `supports_credentials` | Whether cookies and auth headers are allowed. |

**Syntax Rules:**

- The `HandleCors` middleware must be registered globally for CORS headers to apply to all routes.
- The `paths` configuration determines which URIs receive CORS headers.
- `allowed_origins` accepts exact origins or wildcards (`*`).
- `supports_credentials` must be `true` for requests that include cookies or authorization headers.
- The middleware automatically handles OPTIONS preflight requests.

**Constraints and Limitations:**

- **Credentials + wildcard:** Browsers do not allow `Access-Control-Allow-Origin: *` when `supports_credentials` is `true`. You must specify exact origins.
- **Preflight caching:** The `max_age` setting controls how long browsers cache preflight responses. Set to `0` to disable caching.
- **Path matching:** The `paths` configuration uses URI patterns. Ensure your API prefix (e.g., `api/*`) is included.
- **Laravel 11+:** The middleware class name changed from `Fruitcake\Cors\HandleCors` to `Illuminate\Http\Middleware\HandleCors`.

### Multiple Annotated Complete Code Examples

**Example 1: Basic CORS Configuration**

```php
<?php
// config/cors.php

return [
    'paths' => ['api/*'],
    'allowed_methods' => ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
    'allowed_origins' => ['https://myapp.com'],
    'allowed_headers' => ['Content-Type', 'Authorization', 'Accept'],
    'supports_credentials' => true,
];
```

```php
<?php
// bootstrap/app.php (Laravel 11+)

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\Illuminate\Http\Middleware\HandleCors::class);
})
```

**Expected Output:** Requests from `https://myapp.com` to `/api/*` receive the appropriate `Access-Control-Allow-Origin` headers. Requests from other origins are blocked by the browser.

**Why:** The configuration restricts CORS to a specific origin and allows credentials. The middleware reads this configuration and adds the appropriate headers.

---

**Example 2: Allowing All Origins**

```php
<?php
// config/cors.php

return [
    'paths' => ['api/*'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['*'],
    'allowed_headers' => ['*'],
    'supports_credentials' => false,
];
```

**Expected Output:** Any origin can make cross-origin requests to `/api/*`. Credentials (cookies, auth headers) are not supported.

**Why:** The wildcard `*` allows all origins. `supports_credentials` must be `false` when using wildcard origins.

---

**Example 3: Scoping CORS to Specific Paths**

```php
<?php
// config/cors.php

return [
    'paths' => ['api/v1/*', 'api/v2/*'],
    'allowed_methods' => ['GET', 'POST'],
    'allowed_origins' => ['https://partner.example.com'],
    'allowed_headers' => ['Content-Type', 'X-API-Key'],
    'supports_credentials' => false,
];
```

**Expected Output:** CORS headers are applied only to `api/v1/*` and `api/v2/*` paths. Requests to other paths do not receive CORS headers.

**Why:** The `paths` configuration limits CORS handling to specific URI patterns, reducing the attack surface.

### Real-World Cases

- **SPA Backends:** Vue or React front-ends running on a different origin need CORS to communicate with Laravel API.
- **Partner APIs:** Allow specific partner domains to access API endpoints with credentials.
- **Mobile Apps:** Mobile apps using WebView may need CORS configuration for embedded web content.
- **Multi-Service Architectures:** CORS allows front-end services to call backend APIs across domains.

### References

- Laravel Routing: CORS — https://laravel.com/docs/12.x/routing#cors
- Stack Overflow: Configuring CORS in Laravel 12 — https://stackoverflow.com/questions/79639332
- Laravel News: Configuring Middleware — https://laravel-news.com/configuring-middleware-in-laravel

---

## 6. Encrypting Cookies (`EncryptCookies`)

### Definitions

**Core Definition:** The `EncryptCookies` middleware automatically encrypts all outgoing cookies and decrypts incoming cookies, preventing clients from reading or tampering with cookie values.

**Technical Definition:** The `Illuminate\Cookie\Middleware\EncryptCookies` middleware uses Laravel's `Encrypter` (configured with the application's `APP_KEY`) to encrypt cookies on the response and decrypt them on the request. It is included in the `web` middleware group by default. The middleware can exclude specific cookies from encryption via the `$except` property or the `disableFor()` method. Cookie serialization is enabled by default, allowing arrays and objects to be stored.

**Beginner-Friendly Explanation:** Cookies are small pieces of data stored in the user's browser. Normally, anyone can read or modify them. The `EncryptCookies` middleware encrypts them so that only your application can read them. If a user tries to change a cookie value (like changing their user ID to someone else's), the decryption will fail and Laravel will reject the request.

### Purposes

- To encrypt all outgoing cookies, preventing clients from reading sensitive data.
- To decrypt incoming cookies, ensuring integrity and authenticity.
- To exclude specific cookies from encryption (e.g., third-party cookies).
- To integrate with Laravel's session and authentication systems.
- To prevent cookie tampering attacks.

### Syntax Rules and Structure

**Complete General Syntax — Default Registration:**

```php
// The EncryptCookies middleware is included in the web middleware group by default.
// No manual registration is required in Laravel 11+ unless customising.
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `web` middleware group | Includes `EncryptCookies` by default. |
| `$except` property | Names of cookies to exclude from encryption. |

**Complete General Syntax — Excluding Cookies:**

```php
<?php
// app/Http/Middleware/EncryptCookies.php (Laravel 10 and earlier)

namespace App\Http\Middleware;

use Illuminate\Cookie\Middleware\EncryptCookies as Middleware;

class EncryptCookies extends Middleware
{
    protected $except = [
        'cookie_name',
    ];
}
```

**Complete General Syntax — Disabling Encryption at Runtime:**

```php
// In a middleware or controller
\Illuminate\Cookie\Middleware\EncryptCookies::disableFor('cookie_name');
```

**Syntax Rules:**

- The middleware requires a valid `APP_KEY` in the `.env` file.
- Cookies are encrypted using AES-256-CBC by default.
- The `$except` property accepts an array of cookie names to exclude.
- `disableFor()` can be called at runtime to disable encryption for specific cookies.
- The middleware is part of the `web` group and applies to all web routes.

**Constraints and Limitations:**

- **APP_KEY dependency:** If `APP_KEY` changes, all previously encrypted cookies become unreadable.
- **Performance:** Encryption adds a small overhead per cookie. For high-traffic applications, exclude non-sensitive cookies.
- **API routes:** API routes do not include `EncryptCookies` by default. If using cookies in API routes, the middleware must be added manually.
- **Third-party cookies:** Cookies set by external services (e.g., analytics) cannot be encrypted and must be excluded.

### Multiple Annotated Complete Code Examples

**Example 1: Default Cookie Encryption**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/set-cookie', function () {
    return response('Cookie set')
        ->cookie('user_preference', 'dark_mode', 60);
});
```

**Expected Output:** The response sets a cookie named `user_preference` with an encrypted value. Inspecting the cookie in the browser shows an encrypted string, not `dark_mode`.

**Why:** The `EncryptCookies` middleware intercepts the outgoing response and encrypts the cookie value before it is sent to the browser.

---

**Example 2: Excluding a Cookie from Encryption**

```php
<?php
// app/Http/Middleware/EncryptCookies.php (Laravel 10 and earlier)
// In Laravel 11+, this is configured in bootstrap/app.php

namespace App\Http\Middleware;

use Illuminate\Cookie\Middleware\EncryptCookies as Middleware;

class EncryptCookies extends Middleware
{
    protected $except = [
        'tracking_id', // Third-party analytics cookie — do not encrypt
    ];
}
```

**Expected Output:** The `tracking_id` cookie is not encrypted and can be read by client-side JavaScript. All other cookies remain encrypted.

**Why:** The `$except` array tells the middleware to skip encryption for the specified cookie names.

---

**Example 3: Decrypting Cookies in Middleware**

```php
<?php
// A custom middleware that reads an encrypted cookie

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ReadUserPreference
{
    public function handle(Request $request, Closure $next)
    {
        // The EncryptCookies middleware has already decrypted the cookie
        $preference = $request->cookie('user_preference');

        if ($preference === 'dark_mode') {
            config(['app.theme' => 'dark']);
        }

        return $next($request);
    }
}
```

**Expected Output:** The middleware reads the decrypted `user_preference` cookie value and applies the dark theme configuration.

**Why:** The `EncryptCookies` middleware decrypts cookies on the way in, so subsequent middleware and controllers receive the plaintext value via `$request->cookie()`.

### Real-World Cases

- **Session Cookies:** Laravel's session cookie is encrypted by default.
- **Authentication Cookies:** "Remember me" cookies are encrypted to prevent tampering.
- **User Preferences:** Theme, language, and notification preferences stored in encrypted cookies.
- **Third-Party Cookies:** Analytics or tracking cookies are excluded from encryption.

### References

- Laravel API: EncryptCookies — https://api.laravel.com/docs/8.x/Illuminate/Cookie/Middleware/EncryptCookies.html
- Laravel HTTP Responses: Cookies — https://laravel.com/docs/12.x/responses#cookies
- OWASP: Laravel Cheat Sheet — https://cheatsheetseries.owasp.org

---

## 7. Logging & Profiling Middleware

### Definitions

**Core Definition:** Logging and profiling middleware capture request/response metadata—such as URLs, methods, status codes, execution time, and memory usage—for observability, debugging, and performance monitoring.

**Technical Definition:** Laravel does not include built-in logging middleware by default, but provides the infrastructure to create custom middleware that logs requests and responses. Several community packages (e.g., `spatie/laravel-http-logger`, `mtownsend/laravel-request-response-logger`) provide ready-made logging middleware. Profiling middleware typically measures execution time and memory usage using `microtime(true)` and `memory_get_peak_usage()`, adding the results to response headers or logs.

**Beginner-Friendly Explanation:** Logging middleware records what requests your application receives and what responses it sends—useful for debugging and monitoring. Profiling middleware measures how long each request takes and how much memory it uses—useful for finding performance bottlenecks. Together, they give you visibility into how your application behaves in production.

### Purposes

- To log incoming requests (method, URL, headers, input) for debugging and auditing.
- To log outgoing responses (status code, content type, size) for monitoring.
- To measure request execution time and memory usage for performance profiling.
- To add profiling headers (`X-Duration-ms`, `X-Memory-Peak`) to responses.
- To identify slow endpoints and N+1 query problems.
- To provide observability into production application behaviour.

### Syntax Rules and Structure

**Complete General Syntax — Basic Logging Middleware:**

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class LogRequests
{
    public function handle(Request $request, Closure $next): Response
    {
        Log::info('Request received', [
            'method' => $request->method(),
            'url' => $request->fullUrl(),
            'ip' => $request->ip(),
        ]);

        $response = $next($request);

        Log::info('Response sent', [
            'status' => $response->getStatusCode(),
            'content_type' => $response->headers->get('Content-Type'),
        ]);

        return $response;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Log::info()` | Writes a log entry with context. |
| `$request->method()` | HTTP method (GET, POST, etc.). |
| `$request->fullUrl()` | Full request URL. |
| `$response->getStatusCode()` | HTTP response status code. |

**Complete General Syntax — Profiling Middleware:**

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ProfileRequest
{
    public function handle(Request $request, Closure $next): Response
    {
        $startTime = microtime(true);
        $startMemory = memory_get_peak_usage(true);

        $response = $next($request);

        $duration = microtime(true) - $startTime;
        $memoryUsed = memory_get_peak_usage(true) - $startMemory;

        $response->headers->set('X-Duration-ms', round($duration * 1000, 2));
        $response->headers->set('X-Memory-Peak', $memoryUsed);

        return $response;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `microtime(true)` | Returns the current Unix timestamp with microseconds. |
| `memory_get_peak_usage(true)` | Returns peak memory usage in bytes. |
| `->headers->set()` | Adds profiling headers to the response. |

**Syntax Rules:**

- Logging middleware should use Laravel's `Log` facade for structured logging.
- Profiling middleware measures time using `microtime(true)` and memory using `memory_get_peak_usage()`.
- Profiling headers should be prefixed with `X-` (e.g., `X-Duration-ms`).
- Logging middleware can be registered globally or per-route depending on the volume of log data desired.
- Terminable middleware can log after the response is sent to avoid delaying the user.

**Constraints and Limitations:**

- **Performance overhead:** Logging every request can generate large volumes of data. Use sampling or route-specific logging.
- **Memory measurement:** `memory_get_peak_usage()` returns the peak memory usage for the entire PHP process, not just the current request. For per-request memory, use `memory_get_usage()` before and after.
- **Sensitive data:** Logging middleware should not log sensitive data (passwords, tokens, credit card numbers).
- **Profiling in production:** Profiling headers should be disabled or restricted in production to avoid exposing performance data.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Request/Response Logging Middleware**

```php
<?php
// app/Http/Middleware/LogRequestsAndResponses.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class LogRequestsAndResponses
{
    public function handle(Request $request, Closure $next): Response
    {
        // Log the incoming request
        Log::info('Incoming request', [
            'method' => $request->method(),
            'url' => $request->fullUrl(),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
        ]);

        $response = $next($request);

        // Log the outgoing response
        Log::info('Outgoing response', [
            'status' => $response->getStatusCode(),
            'content_type' => $response->headers->get('Content-Type'),
            'size' => strlen($response->getContent()),
        ]);

        return $response;
    }
}
```

```php
<?php
// routes/web.php

Route::get('/test', function () {
    return 'Hello World';
})->middleware(\App\Http\Middleware\LogRequestsAndResponses::class);
```

**Expected Output:** Log entries are written for both the incoming request and the outgoing response. The `/test` route returns `Hello World`.

**Why:** The middleware logs the request metadata before calling `$next()` and the response metadata after. This provides a complete request/response audit trail.

---

**Example 2: Profiling Middleware with Response Headers**

```php
<?php
// app/Http/Middleware/ProfileRequest.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ProfileRequest
{
    public function handle(Request $request, Closure $next): Response
    {
        $startTime = microtime(true);
        $startMemory = memory_get_usage(true);

        $response = $next($request);

        $duration = microtime(true) - $startTime;
        $memoryUsed = memory_get_usage(true) - $startMemory;

        $response->headers->set('X-Duration-ms', round($duration * 1000, 2));
        $response->headers->set('X-Memory-Used', $memoryUsed);

        return $response;
    }
}
```

```php
<?php
// bootstrap/app.php (Laravel 11+)

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\App\Http\Middleware\ProfileRequest::class);
})
```

**Expected Output:** Every response includes `X-Duration-ms` and `X-Memory-Used` headers showing the request's execution time and memory consumption.

**Why:** The middleware measures the time and memory before and after the request is processed, adding the results as response headers for client-side or proxy inspection.

---

**Example 3: Terminable Logging Middleware**

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

**Expected Output:** After the response is sent to the client, the middleware logs the request duration and status code. The user does not experience any delay from the logging.

**Why:** The `terminate()` method runs after the response has been sent, making it ideal for logging and analytics that should not block the response.

### Real-World Cases

- **API Monitoring:** Log all API requests and responses for debugging and compliance.
- **Performance Profiling:** Add `X-Duration-ms` headers to identify slow endpoints.
- **Audit Trails:** Log all requests to sensitive endpoints (admin actions, data exports) for security auditing.
- **Slow Query Detection:** Log requests that exceed a duration threshold for investigation.
- **User Journey Tracing:** Correlate requests using a trace ID for distributed tracing.

### References

- Laravel Middleware: Terminable Middleware — https://laravel.com/docs/12.x/middleware#terminable-middleware
- Laravel Logging Documentation — https://laravel.com/docs/12.x/logging
- Spatie Laravel HTTP Logger — https://github.com/spatie/laravel-http-logger
- Mtownsend Request Response Logger — https://packagist.org/packages/mtownsend/laravel-request-response-logger

---

## 8. Custom Application Middleware

### Definitions

**Core Definition:** Custom application middleware are user-defined middleware classes created to handle application-specific request/response processing that is not covered by Laravel's built-in middleware.

**Technical Definition:** Custom middleware are generated via `php artisan make:middleware MiddlewareName`, which creates a class in `app/Http/Middleware` implementing `handle(Request $request, Closure $next): Response`. They are registered globally via `$middleware->append()` or `$middleware->prepend()`, as aliases via `$middleware->alias()`, or per-route via `->middleware(MiddlewareName::class)`. In Laravel 11+, registration occurs in `bootstrap/app.php`; in Laravel 10 and earlier, in `app/Http/Kernel.php`.

**Beginner-Friendly Explanation:** Custom middleware let you write your own checkpoints for requests. Maybe you want to verify that a user has an active subscription, or that their account isn't suspended, or that they're accessing from an allowed country. You create a class, write the logic, and attach it to routes—just like Laravel's built-in middleware.

### Purposes

- To implement application-specific request filtering (e.g., subscription checks, account status).
- To preprocess request data for specific application needs (e.g., tenant resolution, locale detection).
- To postprocess responses with application-specific headers or transformations.
- To handle cross-cutting concerns (logging, feature flags, maintenance mode) in a reusable way.
- To enforce business rules at the HTTP layer before requests reach controllers.

### Syntax Rules and Structure

**Complete General Syntax — Generating Middleware:**

```bash
php artisan make:middleware EnsureSubscriptionIsActive
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `make:middleware` | The Artisan command to generate middleware. |
| `EnsureSubscriptionIsActive` | The class name (creates `app/Http/Middleware/EnsureSubscriptionIsActive.php`). |

**Complete General Syntax — Middleware Class:**

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureSubscriptionIsActive
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->hasActiveSubscription()) {
            return redirect()->route('billing');
        }

        return $next($request);
    }
}
```

**Complete General Syntax — Registration in `bootstrap/app.php` (Laravel 11+):**

```php
->withMiddleware(function (Middleware $middleware) {
    // Register as alias
    $middleware->alias([
        'subscribed' => \App\Http\Middleware\EnsureSubscriptionIsActive::class,
    ]);

    // Append to global middleware
    $middleware->append(\App\Http\Middleware\LogRequests::class);
})
```

**Complete General Syntax — Route Application:**

```php
Route::get('/premium', function () {
    return 'Premium content';
})->middleware(['auth', 'subscribed']);
```

**Syntax Rules:**

- Custom middleware must implement the `handle(Request $request, Closure $next): Response` method.
- Middleware are resolved from the service container, so constructor injection is supported.
- Registration is required in `bootstrap/app.php` (Laravel 11+) or `app/Http/Kernel.php` (Laravel 10).
- Aliases provide short names for route definitions.
- Global middleware run on every request; route middleware run only on specific routes.

**Constraints and Limitations:**

- **Global performance:** Global middleware run on every request, including static assets and health checks. Minimise global middleware.
- **Order dependency:** Middleware execution order is determined by registration order. Authentication middleware must run before authorization middleware.
- **Alias collisions:** Alias names must be unique and not conflict with built-in aliases.
- **Short-circuiting:** Returning a response without calling `$next()` prevents the request from reaching the controller.

### Multiple Annotated Complete Code Examples

**Example 1: Subscription Check Middleware**

```php
<?php
// app/Http/Middleware/EnsureSubscriptionIsActive.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureSubscriptionIsActive
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->hasActiveSubscription()) {
            return redirect()->route('billing')
                ->with('error', 'Your subscription is not active.');
        }

        return $next($request);
    }
}
```

```php
<?php
// bootstrap/app.php (Laravel 11+)

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'subscribed' => \App\Http\Middleware\EnsureSubscriptionIsActive::class,
    ]);
})
```

```php
<?php
// routes/web.php

Route::middleware(['auth', 'subscribed'])->group(function () {
    Route::get('/premium', function () {
        return 'Premium content';
    });
});
```

**Expected Output:**
- User with active subscription → `Premium content`
- User without active subscription → redirect to `/billing` with error message

**Why:** The middleware checks the user's subscription status and redirects to the billing page if it is not active. It is applied to all routes in the group.

---

**Example 2: Tenant Resolution Middleware**

```php
<?php
// app/Http/Middleware/ResolveTenant.php

namespace App\Http\Middleware;

use App\Models\Tenant;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ResolveTenant
{
    public function handle(Request $request, Closure $next): Response
    {
        $subdomain = $request->route('tenant');

        $tenant = Tenant::where('subdomain', $subdomain)->firstOrFail();

        // Share the tenant with the rest of the application
        app()->instance('tenant', $tenant);

        return $next($request);
    }
}
```

```php
<?php
// routes/web.php

Route::domain('{tenant}.myapp.com')->group(function () {
    Route::get('/dashboard', function () {
        $tenant = app('tenant');
        return "Dashboard for {$tenant->name}";
    })->middleware(\App\Http\Middleware\ResolveTenant::class);
});
```

**Expected Output:** `GET http://acme.myapp.com/dashboard` resolves the `acme` tenant and returns `Dashboard for Acme`.

**Why:** The middleware resolves the tenant from the subdomain parameter and binds it into the service container for the rest of the request.

---

**Example 3: Global Request Logging Middleware**

```php
<?php
// app/Http/Middleware/LogAllRequests.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class LogAllRequests
{
    public function handle(Request $request, Closure $next): Response
    {
        Log::channel('requests')->info($request->method() . ' ' . $request->fullUrl());

        return $next($request);
    }
}
```

```php
<?php
// bootstrap/app.php (Laravel 11+)

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(\App\Http\Middleware\LogAllRequests::class);
})
```

**Expected Output:** Every request is logged to the `requests` channel with the HTTP method and full URL.

**Why:** The middleware is registered globally, so it runs on every HTTP request to the application.

### Real-World Cases

- **Subscription Services:** `EnsureSubscriptionIsActive` restricts premium content to paying users.
- **Multi-Tenancy:** `ResolveTenant` identifies the tenant from the subdomain and scopes database queries.
- **API Versioning:** `SetApiVersion` reads the `Accept` header and sets the API version for the request.
- **Feature Flags:** `EnsureFeatureEnabled` checks if a feature is enabled for the current user before allowing access.
- **Maintenance Mode:** `CheckForMaintenanceMode` rejects requests when the application is under maintenance.

### References

- Laravel Middleware: Defining Middleware — https://laravel.com/docs/12.x/middleware#defining-middleware
- Laravel News: Configuring Middleware — https://laravel-news.com/configuring-middleware-in-laravel
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know

---

## References

- Laravel Middleware Documentation (12.x) — https://laravel.com/docs/12.x/middleware
- Laravel Authentication Documentation — https://laravel.com/docs/12.x/authentication
- Laravel Authorization Documentation — https://laravel.com/docs/12.x/authorization
- Laravel CSRF Protection Documentation — https://laravel.com/docs/12.x/csrf
- Laravel Routing: Rate Limiting — https://laravel.com/docs/12.x/routing#rate-limiting
- Laravel Routing: CORS — https://laravel.com/docs/12.x/routing#cors
- Laravel HTTP Responses: Cookies — https://laravel.com/docs/12.x/responses#cookies
- Laravel API: EncryptCookies — https://api.laravel.com/docs/8.x/Illuminate/Cookie/Middleware/EncryptCookies.html
- Laravel Middleware: Terminable Middleware — https://laravel.com/docs/12.x/middleware#terminable-middleware
- Laravel Logging Documentation — https://laravel.com/docs/12.x/logging
- Laravel News: Configuring Middleware in Laravel — https://laravel-news.com/configuring-middleware-in-laravel
- Laravel Daily: Middleware in Laravel — https://laraveldaily.com/post/middleware-laravel-main-things-to-know
- Cloudways: Laravel Throttle & Limit Guide — https://www.cloudways.com/blog/laravel-rate-limiting-throttle/
- Spatie Laravel HTTP Logger — https://github.com/spatie/laravel-http-logger
- Mtownsend Request Response Logger — https://packagist.org/packages/mtownsend/laravel-request-response-logger
- Stack Overflow: Configuring CORS in Laravel 12 — https://stackoverflow.com/questions/79639332
- Stack Overflow: CSRF Configuration Changes in Laravel 11 — https://stackoverflow.com/revisions/ef8af48e-61c8-465a-b27d-60bb6491d503/view-source
- OWASP: Laravel Cheat Sheet — https://cheatsheetseries.owasp.org