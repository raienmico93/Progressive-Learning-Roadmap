# Laravel Authentication Architecture and Core Components: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel's authentication architecture is a configurable, driver-based system that separates the concerns of *how users are retrieved from storage* (User Providers) from *how users are authenticated on each request* (Authentication Guards), with middleware acting as the gatekeeper for protected routes.

**Technical Definition:** The authentication system is orchestrated by the `Illuminate\Auth\AuthManager` class, which implements the `Factory` contract. It resolves guards and user providers based on the `config/auth.php` configuration file. Guards implement the `Illuminate\Contracts\Auth\Guard` interface (with `SessionGuard` and `TokenGuard` as built-in implementations), while user providers implement the `Illuminate\Contracts\Auth\UserProvider` interface (with `EloquentUserProvider` and `DatabaseUserProvider` as built-in implementations). Middleware classes such as `Authenticate`, `RedirectIfAuthenticated`, and `EnsureEmailIsVerified` intercept requests before they reach controllers.

**Beginner-Friendly Explanation:** Think of Laravel authentication as a security checkpoint with two separate roles. The **User Provider** is the filing cabinet—it knows how to look up user records (usually from a database). The **Guard** is the security guard—it checks each visitor's credentials (via session cookie or API token) and decides if they're allowed in. **Middleware** is the door—it ensures only authenticated visitors reach protected rooms. You can have multiple guards (e.g., one for regular users, one for admins) each using their own filing cabinet.

### Key Characteristics

- **Driver-Based Architecture:** Guards and providers are resolved from configuration, not hardcoded.
- **Separation of Concerns:** Providers handle storage retrieval; guards handle request authentication.
- **Multi-Guard Support:** Applications can define multiple guards for different user types (web, admin, API).
- **Middleware Enforcement:** The `auth`, `guest`, and `verified` middleware enforce access rules declaratively.
- **Config-Driven:** All guards, providers, and defaults are defined in `config/auth.php`.

### Prerequisites

- A Laravel application with the authentication scaffolding (starter kit or manual setup).
- A `User` model implementing `Illuminate\Contracts\Auth\Authenticatable`.
- A configured database with a `users` table (or equivalent).
- Understanding of HTTP middleware and route protection.

### Related Programming Areas

- **Eloquent ORM:** The `EloquentUserProvider` retrieves users via Eloquent models.
- **Session Management:** `SessionGuard` relies on Laravel's session driver.
- **Middleware:** `auth`, `guest`, and `verified` middleware intercept requests.
- **Authorization:** Gates and Policies build on top of the authentication system.

### Core Concepts / Features

1. User Providers (Eloquent vs. Database)
2. Authentication Guards (Session vs. Token)
3. Multiple Authenticators (Multi-Guard Architectures)
4. HTTP Middleware (auth, guest, verified)

---

## 1. User Providers: Managing How Users Are Retrieved

### Definitions

**Core Definition:** A User Provider is a class that knows how to retrieve user records from a persistent storage system (typically a database) by a unique identifier or set of credentials.

**Technical Definition:** User providers implement the `Illuminate\Contracts\Auth\UserProvider` interface, which defines methods `retrieveById()`, `retrieveByToken()`, `updateRememberToken()`, `retrieveByCredentials()`, and `validateCredentials()`. Laravel ships with `EloquentUserProvider` (retrieves users via an Eloquent model) and `DatabaseUserProvider` (retrieves users via the query builder). The `AuthManager` resolves providers via `createUserProvider()`, which reads the `providers` array in `config/auth.php`.

**Beginner-Friendly Explanation:** The User Provider is the "filing cabinet" of your authentication system. When the guard needs to find a user by email or ID, it asks the provider. The provider knows which table to query and which model to use. Eloquent providers use your `User` model; Database providers use raw query builder calls.

### Purposes

- To define how users are retrieved from persistent storage (database, external API, etc.).
- To decouple the authentication guard from the specific storage mechanism.
- To support different user models for different user types (users, admins, customers).
- To allow custom providers for non-relational storage systems.
- To centralise credential validation logic in a single class.

### Syntax Rules and Structure

**Complete General Syntax — Configuration:**

```php
// config/auth.php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
    'customers' => [
        'driver' => 'database',
        'table' => 'customers',
    ],
],
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'users'` | Provider name (referenced by guards). |
| `'driver' => 'eloquent'` | Uses `EloquentUserProvider`. |
| `'model' => User::class` | The Eloquent model to retrieve users from. |
| `'driver' => 'database'` | Uses `DatabaseUserProvider` (query builder). |
| `'table' => 'customers'` | The database table to query. |

**Complete General Syntax — Custom Provider:**

```php
// In AuthServiceProvider::boot()
use Illuminate\Support\Facades\Auth;

Auth::provider('riak', function ($app, array $config) {
    return new RiakUserProvider($app['riak.connection']);
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Auth::provider('riak', ...)` | Registers a custom provider driver. |
| `$config` | The provider configuration from `auth.php`. |
| Return value | Must implement `UserProvider`. |

**Syntax Rules:**
- The `providers` array in `config/auth.php` defines available providers.
- Each provider has a `driver` key (`eloquent` or `database`) and driver-specific configuration.
- Guards reference providers by name via the `provider` key.
- Custom providers are registered via `Auth::provider()` in a service provider.

**Constraints and Limitations:**
- **Eloquent provider:** Requires the model to implement `Authenticatable` and `CanResetPassword` (for password resets).
- **Database provider:** Uses the query builder; the table must have standard column names (`id`, `password`, `remember_token`).
- **Single model per provider:** Each Eloquent provider maps to exactly one model class.
- **Provider resolution:** The `AuthManager` caches resolved providers; config changes require application restart.

### Multiple Annotated Complete Code Examples

**Example 1: Eloquent Provider (Default)**

```php
<?php
// config/auth.php

return [
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,
        ],
    ],

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],
];
```

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    protected $fillable = ['name', 'email', 'password'];
    protected $hidden = ['password', 'remember_token'];
}
```

**Expected Output:** The `web` guard uses `EloquentUserProvider` to retrieve users from the `users` table via the `User` model.

**Why:** The `eloquent` driver tells the `AuthManager` to create an `EloquentUserProvider` with the specified model. The guard delegates user retrieval to this provider.

---

**Example 2: Database Provider**

```php
<?php
// config/auth.php

return [
    'providers' => [
        'users' => [
            'driver' => 'database',
            'table' => 'users',
        ],
    ],
];
```

**Expected Output:** The guard uses `DatabaseUserProvider` to retrieve users via the query builder.

**Why:** The `database` driver uses raw query builder calls instead of Eloquent. This is useful for legacy databases or when Eloquent overhead is unwanted.

---

**Example 3: Custom Provider Registration**

```php
<?php
// app/Providers/AuthServiceProvider.php

namespace App\Providers;

use App\Extensions\RiakUserProvider;
use Illuminate\Support\Facades\Auth;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Auth::provider('riak', function ($app, array $config) {
            return new RiakUserProvider($app['riak.connection']);
        });
    }
}
```

```php
// config/auth.php
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],
```

**Expected Output:** The `riak` provider driver resolves to the custom `RiakUserProvider` class.

**Why:** `Auth::provider()` registers a custom driver creator. The `AuthManager` calls this closure when resolving a provider with `driver => riak`.

### Real-World Cases

- **Multi-Tenant Applications:** Different providers for different tenant databases (e.g., `tenant_1_users`, `tenant_2_users`).
- **Legacy Systems:** Database provider for applications with non-Eloquent database access layers.
- **External Authentication:** Custom providers for LDAP, OAuth, or third-party user stores.
- **Separate User Types:** Eloquent providers for `users` and `admins` using different models.

### References

- Laravel Authentication (12.x) — https://laravel.com/framework/docs/12.x/authentication
- AuthManager API — https://api.laravel.com/docs/9.x/Illuminate/Auth/AuthManager.html
- Adding Custom User Providers — https://laravel.com/framework/docs/7.x/authentication

---

## 2. Authentication Guards: How Users Are Authenticated

### Definitions

**Core Definition:** An Authentication Guard is a class that determines whether an incoming request is from an authenticated user, using a specific mechanism (session cookie or API token) and a User Provider for storage retrieval.

**Technical Definition:** Guards implement the `Illuminate\Contracts\Auth\Guard` interface, which defines methods `check()`, `guest()`, `user()`, `id()`, `validate()`, `setUser()`, and `hasUser()`. Laravel ships with `SessionGuard` (stateful, cookie-based authentication) and `TokenGuard` (stateless, API token authentication). The `AuthManager` resolves guards via `createSessionDriver()` or `createTokenDriver()`, reading the `guards` array in `config/auth.php`.

**Beginner-Friendly Explanation:** The Guard is the "security guard" at the door. For web applications, it checks the session cookie to see if you've logged in. For APIs, it checks the `Authorization` header for a token. Each guard has its own provider (filing cabinet) to look up user records.

### Purposes

- To authenticate users for each incoming HTTP request.
- To provide a consistent API for checking authentication state (`check()`, `guest()`, `user()`).
- To support both stateful (session) and stateless (token) authentication.
- To enable different authentication mechanisms for different parts of the application.
- To manage user login, logout, and "remember me" functionality.

### Syntax Rules and Structure

**Complete General Syntax — Guard Configuration:**

```php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'api' => [
        'driver' => 'token',
        'provider' => 'users',
        'input_key' => 'api_token',
        'storage_key' => 'api_token',
    ],
],
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'web'` | Guard name (referenced by middleware). |
| `'driver' => 'session'` | Uses `SessionGuard` (cookie-based). |
| `'driver' => 'token'` | Uses `TokenGuard` (API token). |
| `'provider' => 'users'` | The user provider to use. |
| `'input_key'` | Query string parameter name for token (TokenGuard). |
| `'storage_key'` | Token column name in storage (TokenGuard). |

**Complete General Syntax — Using a Guard:**

```php
use Illuminate\Support\Facades\Auth;

// Get the default guard
$user = Auth::user();

// Get a specific guard
$user = Auth::guard('admin')->user();

// Check authentication
if (Auth::guard('web')->check()) {
    // User is authenticated
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Auth::user()` | Returns the authenticated user (or `null`). |
| `Auth::guard('name')` | Returns a specific guard instance. |
| `Auth::check()` | Returns `true` if authenticated. |
| `Auth::guest()` | Returns `true` if not authenticated. |
| `Auth::id()` | Returns the authenticated user's ID. |

**Syntax Rules:**
- The `guards` array in `config/auth.php` defines available guards.
- Each guard has a `driver` (`session` or `token`) and a `provider`.
- The `defaults.guard` setting determines which guard is used by default.
- `Auth::guard('name')` returns the guard instance for method calls.
- Middleware `auth:guard_name` specifies which guard to use for a route.

**Constraints and Limitations:**
- **SessionGuard requires sessions:** The `session` driver requires session middleware to be active.
- **TokenGuard is stateless:** No session is created; each request must include the token.
- **TokenGuard input key:** The default input key is `api_token`, configurable via the guard config.
- **Guard caching:** Resolved guards are cached by `AuthManager`; configuration changes require a restart.

### Multiple Annotated Complete Code Examples

**Example 1: Session Guard (Web)**

```php
<?php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

```php
<?php
// Login controller
use Illuminate\Support\Facades\Auth;

public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials)) {
        $request->session()->regenerate();
        return redirect()->intended('dashboard');
    }

    return back()->withErrors([
        'email' => 'The provided credentials do not match our records.',
    ]);
}
```

**Expected Output:** On successful login, the user is authenticated via session and redirected to the dashboard. The session ID is regenerated to prevent session fixation.

**Why:** `Auth::attempt()` uses the default guard (`web`), which is a `SessionGuard`. It retrieves the user via the provider, validates credentials, and stores the user ID in the session.

---

**Example 2: Token Guard (API)**

```php
<?php
// config/auth.php
'guards' => [
    'api' => [
        'driver' => 'token',
        'provider' => 'users',
        'input_key' => 'api_token',
        'storage_key' => 'api_token',
    ],
],
```

```php
// Request with token
// GET /api/user?api_token=abc123
```

```php
<?php
// Route
Route::get('/user', function (Request $request) {
    return $request->user(); // Authenticated via token
})->middleware('auth:api');
```

**Expected Output:** The request is authenticated by matching the `api_token` query parameter against the `api_token` column in the `users` table.

**Why:** `TokenGuard` reads the token from the request (via `input_key`) and looks it up in the user provider (via `storage_key`). No session is created.

---

**Example 3: Manual Guard Usage**

```php
<?php
use Illuminate\Support\Facades\Auth;

// Get the default guard's user
$user = Auth::user();

// Get the admin guard's user
$admin = Auth::guard('admin')->user();

// Check if the current user is authenticated
if (Auth::check()) {
    // ...
}

// Check a specific guard
if (Auth::guard('api')->check()) {
    // API user is authenticated
}
```

**Expected Output:** `Auth::user()` returns the user from the default guard. `Auth::guard('admin')->user()` returns the admin user (if authenticated via the admin guard).

**Why:** Each guard maintains its own authentication state. `Auth::guard()` resolves and returns the specified guard instance.

### Real-World Cases

- **Web Applications:** Session guard for browser-based authentication with "remember me" cookies.
- **API Applications:** Token guard for stateless API authentication.
- **SPA + API:** Sanctum uses a hybrid approach (session for SPA, token for API).
- **Multi-Device Sessions:** Session guard supports `logoutOtherDevices()` for invalidating sessions on other devices.

### References

- Laravel Authentication (12.x) — https://laravel.com/framework/docs/12.x/authentication
- SessionGuard API — https://api.laravel.com/docs/7.x/Illuminate/Auth/SessionGuard.html
- TokenGuard API — https://api.laravel.com/docs/8.x/Illuminate/Auth/TokenGuard.html
- AuthManager API — https://api.laravel.com/docs/9.x/Illuminate/Auth/AuthManager.html

---

## 3. Multiple Authenticators: Multi-Guard Architectures

### Definitions

**Core Definition:** A multi-guard architecture allows an application to define multiple authentication guards, each with its own user provider and authentication mechanism, enabling different user types (e.g., regular users and administrators) to authenticate independently.

**Technical Definition:** Each guard is defined in the `guards` array of `config/auth.php` with a unique name, a `driver`, and a `provider`. Routes specify which guard to use via middleware (`auth:guard_name`). The `Auth` facade resolves guards by name via `Auth::guard('name')`. Guards maintain independent authentication state, so a user can be authenticated on one guard and a guest on another.

**Beginner-Friendly Explanation:** Imagine a building with two separate entrances—one for regular users and one for administrators. Each entrance has its own security guard and its own filing cabinet. A person can be logged into the user entrance but not the admin entrance, and vice versa. Multi-guard lets you protect different parts of your application with different authentication systems.

### Purposes

- To authenticate different user types with separate models and credentials.
- To isolate web and admin authentication completely.
- To support API and web authentication simultaneously.
- To allow different providers for different guards (Eloquent, Database, custom).
- To enforce access control based on the guard a user is authenticated against.

### Syntax Rules and Structure

**Complete General Syntax — Multi-Guard Configuration:**

```php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'admin' => [
        'driver' => 'session',
        'provider' => 'admins',
    ],
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class,
    ],
],
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'admin'` | Second guard name. |
| `'driver' => 'session'` | Uses the same session driver. |
| `'provider' => 'admins'` | References the `admins` provider. |
| `'admins'` provider | Uses the `Admin` model for retrieval. |

**Complete General Syntax — Route Protection:**

```php
// Protect with default guard
Route::get('/dashboard', ...)->middleware('auth');

// Protect with admin guard
Route::get('/admin/dashboard', ...)->middleware('auth:admin');

// Multiple guards (either/or)
Route::get('/profile', ...)->middleware('auth:web,admin');
```

**Syntax Rules:**
- Each guard requires a unique name in the `guards` array.
- Each guard references a provider by name.
- Routes specify the guard via `auth:guard_name` middleware.
- Multiple guards in middleware (`auth:web,admin`) allow authentication via any of the listed guards.
- The `Auth` facade resolves guards by name: `Auth::guard('admin')`.

**Constraints and Limitations:**
- **Session conflicts:** Both guards using `session` driver share the same session, but store user IDs under different keys.
- **Route model binding:** `$request->user()` returns the user for the **default** guard; use `$request->user('admin')` for a specific guard.
- **Redirect logic:** The `auth` middleware redirects to the `login` named route; multi-guard applications may need custom redirect logic.
- **Permission systems:** Packages like Spatie Permission require explicit `guard_name` configuration for multi-guard support .

### Multiple Annotated Complete Code Examples

**Example 1: Web and Admin Guards**

```php
<?php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'admin' => [
        'driver' => 'session',
        'provider' => 'admins',
    ],
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class,
    ],
],
```

```php
// Admin login controller
use Illuminate\Support\Facades\Auth;

public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::guard('admin')->attempt($credentials)) {
        $request->session()->regenerate();
        return redirect()->intended('/admin/dashboard');
    }

    return back()->withErrors(['email' => 'Invalid credentials.']);
}
```

**Expected Output:** Admin users authenticate via the `admin` guard and are redirected to the admin dashboard. Regular users use the `web` guard independently.

**Why:** `Auth::guard('admin')->attempt()` uses the admin guard and its provider (`admins`), which retrieves users from the `admins` table via the `Admin` model.

---

**Example 2: Route Protection with Multiple Guards**

```php
<?php
// routes/web.php

// Only authenticated web users
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth');

// Only authenticated admins
Route::prefix('admin')->middleware('auth:admin')->group(function () {
    Route::get('/dashboard', function () {
        return view('admin.dashboard');
    });
});

// Either web or admin
Route::get('/profile', function () {
    $user = Auth::guard('web')->user() ?? Auth::guard('admin')->user();
    return view('profile', compact('user'));
})->middleware('auth:web,admin');
```

**Expected Output:**
- `/dashboard` requires web guard authentication.
- `/admin/dashboard` requires admin guard authentication.
- `/profile` accepts either web or admin authentication.

**Why:** The `auth:admin` middleware specifies the `admin` guard. The `auth:web,admin` middleware accepts either guard.

---

**Example 3: Accessing Users from Specific Guards**

```php
<?php
use Illuminate\Support\Facades\Auth;

// In a controller
public function index(Request $request)
{
    // Default guard user
    $user = $request->user();

    // Admin guard user
    $admin = $request->user('admin');

    // Using Auth facade
    $admin = Auth::guard('admin')->user();

    // Check which guard is authenticated
    if (Auth::guard('admin')->check()) {
        // Admin is logged in
    } elseif (Auth::guard('web')->check()) {
        // Regular user is logged in
    }
}
```

**Expected Output:** The controller can access users from both guards independently.

**Why:** `$request->user('admin')` resolves the user from the admin guard. `Auth::guard('admin')->check()` checks the admin guard's authentication state.

### Real-World Cases

- **E-commerce:** `web` guard for customers, `admin` guard for store administrators.
- **SaaS Applications:** `web` guard for account owners, `api` guard for API clients, `admin` guard for platform administrators.
- **Healthcare Systems:** `patient` guard, `doctor` guard, `admin` guard with separate models.
- **Multi-Tenant:** `tenant` guard with dynamic provider switching per tenant.

### References

- Laravel Passport: Multiple Authentication Guards — https://laravel.com/framework/docs/11.x/passport
- Laravel Permission: Multiple Guards — https://mintlify.wiki/spatie/laravel-permission/advanced-usage/multiple-guards
- Laravel Authentication (12.x) — https://laravel.com/framework/docs/12.x/authentication

---

## 4. Deep-Dive into HTTP Middleware (auth, guest, verified)

### Definitions

**Core Definition:** Laravel's authentication middleware are HTTP middleware classes that intercept requests to enforce authentication, guest-only access, or email verification before allowing access to routes.

**Technical Definition:** The `auth` middleware (`Illuminate\Auth\Middleware\Authenticate`) checks if the request is authenticated and redirects unauthenticated users to the login route or returns a 401 JSON response for APIs. The `guest` middleware (`Illuminate\Auth\Middleware\RedirectIfAuthenticated`) redirects authenticated users away from guest-only routes. The `verified` middleware (`Illuminate\Auth\Middleware\EnsureEmailIsVerified`) checks if the authenticated user's email is verified, redirecting to the verification notice if not.

**Beginner-Friendly Explanation:** Middleware are like security checkpoints at different doors. The `auth` door says "only logged-in users can pass." The `guest` door says "only non-logged-in users can pass" (for login/register pages). The `verified` door says "only users who have verified their email can pass."

### Purposes

- To protect routes from unauthenticated access using `auth`.
- To prevent authenticated users from accessing login/register pages using `guest`.
- To enforce email verification before accessing certain routes using `verified`.
- To support API-specific unauthenticated responses (401 JSON instead of redirect).
- To allow specifying which guard to use for authentication.

### Syntax Rules and Structure

**Complete General Syntax — `auth` Middleware:**

```php
Route::get('/dashboard', function () {
    // Only authenticated users
})->middleware('auth');
```

**Complete General Syntax — `guest` Middleware:**

```php
Route::get('/login', function () {
    // Only guests (non-authenticated users)
})->middleware('guest');
```

**Complete General Syntax — `verified` Middleware:**

```php
Route::get('/profile', function () {
    // Only users with verified email
})->middleware(['auth', 'verified']);
```

**Complete General Syntax — Specifying a Guard:**

```php
Route::get('/admin', function () {
    // Only admin guard authenticated users
})->middleware('auth:admin');
```

**Component Breakdown:**

| Middleware | Alias | Class | Behaviour |
|------------|-------|-------|-----------|
| `auth` | `auth` | `Authenticate` | Redirects guests to login (web) or 401 (API). |
| `guest` | `guest` | `RedirectIfAuthenticated` | Redirects authenticated users to dashboard. |
| `verified` | `verified` | `EnsureEmailIsVerified` | Redirects unverified users to verification notice. |
| `auth:admin` | `auth` | `Authenticate` | Uses the `admin` guard instead of default. |

**Syntax Rules:**
- Middleware are attached to routes using `->middleware('name')`.
- Multiple middleware can be chained: `->middleware(['auth', 'verified'])`.
- The `auth` middleware accepts guard names after a colon: `auth:admin`.
- Multiple guards can be specified: `auth:web,admin`.
- `guest` middleware accepts a guard name: `guest:admin`.

**Constraints and Limitations:**
- **Redirect targets:** The `auth` middleware redirects to the `login` named route; customise via `redirectGuestsTo()` in `bootstrap/app.php` .
- **API behaviour:** Requests expecting JSON receive a 401 response instead of a redirect.
- **`verified` requires `auth`:** The `verified` middleware assumes an authenticated user; it should be used after `auth`.
- **Email verification setup:** The `verified` middleware requires the `User` model to implement `MustVerifyEmail`.

### Multiple Annotated Complete Code Examples

**Example 1: `auth` Middleware Protection**

```php
<?php
// routes/web.php

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth');

// With specific guard
Route::prefix('admin')->middleware('auth:admin')->group(function () {
    Route::get('/dashboard', function () {
        return view('admin.dashboard');
    });
});
```

**Expected Output:**
- Unauthenticated user visiting `/dashboard` is redirected to `/login`.
- Unauthenticated AJAX request to `/dashboard` receives 401 JSON.
- Unauthenticated user visiting `/admin/dashboard` is redirected to `/login` (or custom admin login).

**Why:** The `auth` middleware checks `Auth::guard()->check()`. If false, it redirects to the `login` route for web requests or returns 401 for JSON requests.

---

**Example 2: `guest` Middleware for Login/Register**

```php
<?php
// routes/web.php

Route::get('/login', [LoginController::class, 'showLoginForm'])
    ->middleware('guest');

Route::get('/register', [RegisterController::class, 'showRegistrationForm'])
    ->middleware('guest');
```

**Expected Output:** Authenticated users visiting `/login` or `/register` are redirected to the dashboard (or home).

**Why:** The `guest` middleware checks if the user is authenticated. If true, it redirects to the `dashboard` or `home` named route.

---

**Example 3: `verified` Middleware for Email Verification**

```php
<?php
// routes/web.php

Route::get('/profile', function () {
    return view('profile');
})->middleware(['auth', 'verified']);
```

```php
<?php
// app/Models/User.php
use Illuminate\Contracts\Auth\MustVerifyEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

**Expected Output:** Authenticated users with unverified emails are redirected to the email verification notice page.

**Why:** The `verified` middleware checks `$user->hasVerifiedEmail()`. If false, it redirects to the `verification.notice` route.

---

**Example 4: Custom Redirect Targets**

```php
<?php
// bootstrap/app.php

use Illuminate\Http\Request;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->redirectGuestsTo('/custom-login');
        $middleware->redirectUsersTo('/panel');
    })
```

**Expected Output:**
- Unauthenticated users are redirected to `/custom-login` instead of `/login`.
- Authenticated users visiting guest pages are redirected to `/panel`.

**Why:** `redirectGuestsTo()` overrides the default login redirect. `redirectUsersTo()` overrides the default authenticated redirect.

### Real-World Cases

- **Dashboard Protection:** `auth` middleware on all dashboard routes.
- **Login/Register Pages:** `guest` middleware to prevent authenticated users from accessing auth forms.
- **Email Verification Gating:** `verified` middleware on profile, settings, and premium features.
- **Admin Panels:** `auth:admin` middleware on all admin routes.

### References

- Laravel Authentication: Protecting Routes — https://laravel.com/framework/docs/12.x/authentication
- Email Verification Middleware — https://laravel.com/docs/5.8/verification
- Laravel Middleware Documentation — https://laravel.com/framework/docs/12.x/middleware

---

## References

- Laravel Authentication (12.x) — https://laravel.com/framework/docs/12.x/authentication
- Laravel Authentication (13.x) — https://laravel.com/framework/docs/authentication
- Laravel Passport: Multiple Authentication Guards — https://laravel.com/framework/docs/11.x/passport
- Laravel Passport (10.x) — https://laravel.com/framework/docs/10.x/passport
- Laravel Passport (8.x) — https://laravel.com/docs/8.x/passport
- Laravel Email Verification (5.8) — https://laravel.com/docs/5.8/verification
- AuthManager API (9.x) — https://api.laravel.com/docs/9.x/Illuminate/Auth/AuthManager.html
- AuthManager API (8.x) — https://api.laravel.com/docs/8.x/Illuminate/Auth/AuthManager.html
- SessionGuard API — https://api.laravel.com/docs/7.x/Illuminate/Auth/SessionGuard.html
- TokenGuard API — https://api.laravel.com/docs/8.x/Illuminate/Auth/TokenGuard.html
- RequestGuard API — https://api.laravel.com/docs/8.x//Illuminate/Auth/RequestGuard.html
- Laravel Permission: Multiple Guards — https://mintlify.wiki/spatie/laravel-permission/advanced-usage/multiple-guards
- Laravel Multiple Guards Package — https://packagist.org/packages/shiftechafrica/laravel-multiple-guards
- Laravel Permission (Packagist) — https://packagist.org/packages/hoai/laravel-permission-main
- Laravel Authentication (Vectorial1024/docs) — https://github.com/Vectorial1024/docs/blob/12.x/authentication.md
- Laravel Authentication (GitHub 12.x) — https://raw.githubusercontent.com/laravel/docs/refs/heads/12.x/authentication.md