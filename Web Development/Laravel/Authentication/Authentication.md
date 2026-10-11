# Laravel Authentication Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Authentication is the framework's built-in system for verifying user identity, managing user sessions, and controlling access to protected resources within a web application.

**Technical Definition**

Laravel's authentication system is composed of "guards" and "providers." Guards define how users are authenticated for each request (e.g., session-based, token-based), while providers define how users are retrieved from persistent storage (e.g., Eloquent, database query builder). The system is configured in `config/auth.php` and provides facades (`Auth`), middleware (`auth`), and a suite of starter kits and packages for rapid implementation.

**Beginner-Friendly Explanation**

Authentication is how your application knows who is using it. When a user logs in with their email and password, Laravel checks if those credentials are correct and then remembers that user for subsequent requests. It is like a bouncer at a club who checks your ID and then gives you a wristband so you don't have to show your ID again every time you go to the bar.

---

### Key Characteristics

- **Guard-Based Architecture**: Multiple authentication strategies (session, token) can coexist, each with its own guard.
- **Provider Abstraction**: User retrieval is decoupled from authentication logic, allowing Eloquent, database, or custom providers.
- **Starter Kits**: Breeze, Jetstream, and Fortify provide pre-built authentication scaffolding for rapid development.
- **API Authentication**: Sanctum (lightweight, token-based) and Passport (full OAuth2) cover API authentication needs.
- **Session Management**: Automatic session regeneration on login prevents session fixation attacks.
- **Rate Limiting**: Built-in throttling on login attempts protects against brute-force attacks.
- **Remember Me**: Persistent authentication via a `remember_token` column.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A configured database** with a `users` table containing `email`, `password`, and a nullable `remember_token` column of at least 100 characters.
- **Basic knowledge of PHP, Blade, and Eloquent**.
- **Composer** for installing authentication packages.

---

### Related Programming Areas

- **Authorization**: Permissions and policies that determine what an authenticated user can do.
- **Session Management**: Laravel's session driver stores authentication state for web requests.
- **API Development**: Sanctum and Passport provide token-based authentication for APIs.
- **Middleware**: The `auth` middleware protects routes from unauthenticated access.
- **Frontend Integration**: Breeze and Jetstream provide Blade, Livewire, and Inertia.js frontends.

---

## Core Concepts

---

### 1. Authentication

#### Definitions

**Core Definition**

Authentication is the process of verifying that a user is who they claim to be, typically by validating credentials such as an email and password.

**Technical Definition**

Laravel's authentication system provides the `Auth` facade, the `Authenticatable` contract, and the `Guard` interface. When a user submits credentials, the `attempt()` method retrieves the user via the configured provider and verifies the password using `Hash::check()`. If successful, the user is logged into the session (or issued a token), and subsequent requests are authenticated via middleware.

**Beginner-Friendly Explanation**

Authentication is the login process. When you enter your email and password, Laravel checks if they match what is stored in the database. If they do, you are "logged in" and can access protected pages.

#### Purposes

- To verify user identity before granting access to protected resources.
- To maintain user state across multiple HTTP requests (which are stateless by nature).
- To provide a foundation for authorization (what the user is allowed to do).
- To protect sensitive data and functionality from unauthorised access.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Auth;

// Attempt to authenticate
if (Auth::attempt(['email' => $email, 'password' => $password])) {
    // Authentication passed
    $request->session()->regenerate();
    return redirect()->intended('dashboard');
}

// Check if authenticated
if (Auth::check()) {
    $user = Auth::user();
}

// Logout
Auth::logout();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Auth::attempt($credentials)` | Attempts login with credentials. |
| `Auth::check()` | Returns `true` if a user is authenticated. |
| `Auth::user()` | Returns the authenticated user instance. |
| `Auth::logout()` | Logs the user out and clears the session. |
| `$request->session()->regenerate()` | Prevents session fixation. |

**Syntax Rules**

- `attempt()` accepts an array of credentials.
- The `password` key is automatically hashed and compared.
- Session regeneration is recommended after successful login.
- The `Auth` facade is available globally.

**Constraints and Limitations**

- `attempt()` does not work with API tokens; use `auth('api')->attempt()` or token-based methods.
- The `remember` option must be passed as the second argument to `attempt()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Login**

**Controller**

```php
<?php
// app/Http/Controllers/AuthController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();
            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ]);
    }
}
```

**Route**

```php
Route::post('/login', [AuthController::class, 'login']);
```

**Expected Output**

```
// Successful: Redirect to /dashboard
// Failed: Redirect back with error message
```

**Why This Output Occurs**

`Auth::attempt()` retrieves the user by email, verifies the password hash, and if successful, logs the user into the session. The session ID is regenerated to prevent fixation.

---

**Example 2: Checking Authentication Status**

**Blade View**

```blade
@auth
    <p>Welcome, {{ auth()->user()->name }}!</p>
@endauth

@guest
    <p>Please <a href="/login">log in</a>.</p>
@endguest
```

**Expected Output (when logged in)**

```html
<p>Welcome, John!</p>
```

**Why This Output Occurs**

The `@auth` directive checks if a user is authenticated. If so, the user's name is displayed. Otherwise, the `@guest` block is rendered.

---

#### Real-World Cases with Explanation

**Case 1: User Dashboard**

A dashboard route is protected by the `auth` middleware. Only logged-in users can access it, and the controller retrieves the authenticated user via `Auth::user()`.

**Explanation**: Authentication gates access to private areas of the application.

---

### 2. Authentication Guards

#### Definitions

**Core Definition**

Authentication guards define how users are authenticated for each request, specifying the mechanism (session, token) used to verify identity.

**Technical Definition**

A guard is a class implementing `Illuminate\Contracts\Auth\Guard`. Laravel ships with `SessionGuard` (session-based) and `TokenGuard` (token-based). Guards are configured in `config/auth.php` under the `guards` key. Each guard is associated with a provider and a driver. The default guard is `web` (session), and an `api` guard is also pre-configured.

**Beginner-Friendly Explanation**

A guard is like a specific door into your application. The `web` guard uses cookies and sessions (for browser users), while the `api` guard uses tokens (for mobile apps or external services).

#### Purposes

- To support multiple authentication strategies within a single application.
- To separate web (session) and API (token) authentication concerns.
- To allow custom authentication logic through custom guard implementations.
- To provide a consistent interface for authenticating users regardless of mechanism.

#### Syntax Rules and Structure

**Complete General Syntax**

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
        'hash' => false,
    ],
],
```

**Component Breakdown**

| Key | Description |
|-----|-------------|
| `driver` | The authentication driver (`session`, `token`, `sanctum`, `passport`). |
| `provider` | The user provider (`users`, `admins`). |

**Syntax Rules**

- Guards are defined in `config/auth.php`.
- The `defaults.guard` key specifies the default guard.
- Guards can be accessed via `Auth::guard('api')`.
- Middleware can specify a guard: `auth:api`.

**Constraints and Limitations**

- A guard requires a corresponding provider.
- Custom guards must implement `Illuminate\Contracts\Auth\Guard`.
- Guards are resolved per-request.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Using the API Guard**

**Route**

```php
Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Expected Output**

```json
{"id":1,"name":"John","email":"john@example.com"}
```

**Why This Output Occurs**

The `auth:api` middleware instructs Laravel to use the `api` guard (token driver) to authenticate the request. If a valid token is provided, the user is returned.

---

**Example 2: Accessing a Specific Guard**

**Controller**

```php
$user = Auth::guard('web')->user();
```

**Expected Output**

```php
// Returns the authenticated web user or null
```

**Why This Output Occurs**

`Auth::guard('web')` explicitly uses the `web` guard, bypassing the default guard.

---

#### Real-World Cases with Explanation

**Case 1: Separate Admin and User Guards**

An application has a `web` guard for regular users and an `admin` guard for administrators, each with its own provider and session key.

**Explanation**: Multiple guards allow different user types to be authenticated independently.

---

### 3. Authentication Providers

#### Definitions

**Core Definition**

Authentication providers define how users are retrieved from persistent storage, such as a database, during authentication.

**Technical Definition**

A provider is a class implementing `Illuminate\Contracts\Auth\UserProvider`. Laravel ships with `EloquentUserProvider` and `DatabaseUserProvider`. Providers are configured in `config/auth.php` under the `providers` key. Each provider specifies a `driver` and a `model` (for Eloquent) or `table` (for database).

**Beginner-Friendly Explanation**

A provider is the system that looks up users in your database. It knows which table or model to query when someone tries to log in.

#### Purposes

- To decouple user retrieval from authentication logic.
- To support different user storage mechanisms (Eloquent, database, external API).
- To allow multiple user types (e.g., users and admins) with separate tables.
- To provide a consistent interface for retrieving users by credentials or ID.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/auth.php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
    'admins' => [
        'driver' => 'database',
        'table' => 'admins',
    ],
],
```

**Component Breakdown**

| Key | Description |
|-----|-------------|
| `driver` | `eloquent` or `database`. |
| `model` | The Eloquent model class (for Eloquent provider). |
| `table` | The table name (for database provider). |

**Syntax Rules**

- Providers are defined in `config/auth.php`.
- Each guard references a provider.
- Custom providers must implement `UserProvider`.

**Constraints and Limitations**

- The Eloquent provider requires the model to implement `Authenticatable`.
- The database provider requires a table with the necessary columns.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Eloquent Provider**

**Config**

```php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
],
```

**Expected Output**

- Users are retrieved via the User Eloquent model

**Why This Output Occurs**

The Eloquent provider uses the `User` model to query the `users` table.

---

#### Real-World Cases with Explanation

**Case 1: Custom User Table**

An application uses a `members` table instead of `users`. The provider is configured with `'table' => 'members'` and the database driver.

**Explanation**: Providers allow non-standard table names and schemas.

---

### 4. Authentication Drivers

#### Definitions

**Core Definition**

Authentication drivers are the underlying implementations that determine how authentication state is maintained (e.g., session, token).

**Technical Definition**

Drivers are specified in the `driver` key of a guard configuration. Laravel supports `session`, `token`, `sanctum`, and `passport` drivers. Each driver corresponds to a guard class: `SessionGuard`, `TokenGuard`, `Sanctum` guard, and `Passport` guard respectively.

**Beginner-Friendly Explanation**

A driver is the specific technology used for authentication. Session drivers use cookies; token drivers use API tokens.

#### Purposes

- To provide different mechanisms for maintaining authentication state.
- To allow the same provider to be used with different authentication strategies.
- To enable seamless switching between session and token authentication.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'api' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],
],
```

**Component Breakdown**

| Driver | Description |
|--------|-------------|
| `session` | Uses PHP sessions and cookies. |
| `token` | Uses API tokens in the `Authorization` header. |
| `sanctum` | Uses Sanctum's token or SPA cookie authentication. |
| `passport` | Uses Passport's OAuth2 tokens. |

**Syntax Rules**

- The driver is specified per guard.
- Changing the driver does not require changing the provider.
- Custom drivers can be registered via `Auth::extend()`.

**Constraints and Limitations**

- Some drivers require additional packages (Sanctum, Passport).
- Session driver requires session middleware.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Session Driver**

**Config**

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

**Expected Output**

```php
// Auth::attempt() logs the user into the session
```

**Why This Output Occurs**

The session driver stores the user ID in the session after successful authentication.

---

#### Real-World Cases with Explanation

**Case 1: Mixed Authentication**

A SaaS application uses session authentication for the web dashboard and Sanctum token authentication for its mobile app, both using the same `users` provider.

**Explanation**: Drivers allow different client types to authenticate using the appropriate mechanism.

---

### 5. Session Authentication

#### Definitions

**Core Definition**

Session authentication is a stateful authentication mechanism where the user's authentication status is stored in a server-side session and referenced by a cookie.

**Technical Definition**

Laravel's `SessionGuard` stores the authenticated user's ID in the session. On each request, the guard retrieves the session, finds the user ID, and loads the user via the provider. The session is identified by a cookie (default: `laravel_session`). Session authentication includes CSRF protection via the `VerifyCsrfToken` middleware.

**Beginner-Friendly Explanation**

Session authentication is the standard "log in and stay logged in" approach for websites. After you log in, the server remembers you by storing a session ID in a cookie.

#### Purposes

- To provide a seamless login experience for browser-based users.
- To maintain authentication state across multiple page requests.
- To leverage Laravel's built-in CSRF protection.
- To enable "remember me" functionality for persistent sessions.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Login
Auth::attempt($credentials);

// Check
Auth::check();

// Logout
Auth::logout();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `attempt()` | Authenticates and logs in. |
| `login($user)` | Logs in a user instance directly. |
| `logout()` | Logs out and invalidates the session. |
| `viaRemember()` | Checks if the user was authenticated via "remember me". |

**Syntax Rules**

- Requires the `StartSession` middleware.
- The `remember` option must be passed as a boolean to `attempt()`.
- Session ID regeneration is recommended after login.

**Constraints and Limitations**

- Not suitable for stateless APIs.
- Sessions require server-side storage (file, database, Redis).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Login with Remember Me**

**Controller**

```php
if (Auth::attempt($credentials, $request->boolean('remember'))) {
    $request->session()->regenerate();
    return redirect()->intended('dashboard');
}
```

**Expected Output**

```
// User is logged in and a remember_token is set in the cookie
```

**Why This Output Occurs**

The second argument to `attempt()` enables the "remember me" feature, which sets a long-lived cookie.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Checkout**

An e-commerce site uses session authentication so users can log in, browse products, and checkout without re-authenticating.

**Explanation**: Session authentication is ideal for traditional web applications.

---

### 6. Token Authentication

#### Definitions

**Core Definition**

Token authentication is a stateless authentication mechanism where a token (a string) is issued to the client and sent with each request to authenticate.

**Technical Definition**

Laravel's `TokenGuard` (or Sanctum/Passport) authenticates requests by checking the `Authorization: Bearer <token>` header or an `api_token` query parameter. Tokens are stored in the database and associated with a user. Sanctum issues personal access tokens with abilities, while Passport issues OAuth2 access tokens.

**Beginner-Friendly Explanation**

Token authentication is like a temporary key. You log in once, receive a key, and use that key for every subsequent request. It is ideal for mobile apps and APIs.

#### Purposes

- To authenticate stateless API requests.
- To support mobile applications and third-party integrations.
- To avoid the need for session cookies in API contexts.
- To enable fine-grained access control via token abilities/scopes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Issuing a token (Sanctum)
$token = $user->createToken('token-name', ['ability1'])->plainTextToken;

// Using the token
// Authorization: Bearer <token>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `createToken()` | Issues a new token. |
| `plainTextToken` | The token string to send to the client. |
| `abilities` | Permissions associated with the token. |

**Syntax Rules**

- Tokens are sent in the `Authorization` header.
- Sanctum uses the `auth:sanctum` middleware.
- Passport uses the `auth:api` middleware.

**Constraints and Limitations**

- Tokens must be stored securely on the client.
- Token revocation requires database updates.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Issuing a Sanctum Token**

**Controller**

```php
public function login(Request $request)
{
    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    $token = $user->createToken('mobile-app')->plainTextToken;

    return response()->json(['token' => $token]);
}
```

**Expected Output**

```json
{"token":"1|abc123def456..."}
```

**Why This Output Occurs**

The token is generated and returned as a plain text string. The client stores it and sends it in the `Authorization` header for subsequent requests.

---

#### Real-World Cases with Explanation

**Case 1: Mobile App API**

A mobile app authenticates users via Sanctum tokens. The token is stored in secure device storage and included in every API request.

**Explanation**: Token authentication is the standard for mobile and API-first applications.

---

### 7. Authentication Scaffolding

#### Definitions

**Core Definition**

Authentication scaffolding refers to the pre-built controllers, routes, views, and configuration that Laravel provides to implement authentication quickly.

**Technical Definition**

Laravel offers three official starter kits: Breeze (minimal), Jetstream (full-featured), and Fortify (headless backend). Each provides different levels of abstraction and frontend integration. Additionally, the `php artisan make:auth` command (deprecated in newer versions) historically generated basic scaffolding.

**Beginner-Friendly Explanation**

Scaffolding is pre-made code that gives you a complete login, registration, and password reset system without having to write everything from scratch.

#### Purposes

- To accelerate development by providing ready-to-use authentication code.
- To demonstrate Laravel's authentication best practices.
- To allow developers to choose the level of frontend integration they need.
- To provide a starting point that can be customised.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Breeze (Blade)
composer require laravel/breeze --dev
php artisan breeze:install

# Jetstream
composer require laravel/jetstream
php artisan jetstream:install livewire

# Fortify
composer require laravel/fortify
php artisan fortify:install
```

**Component Breakdown**

| Package | Description |
|---------|-------------|
| Breeze | Minimal scaffolding with Blade, Livewire, or Inertia. |
| Jetstream | Full-featured with teams, 2FA, API support. |
| Fortify | Headless backend for custom frontends. |

**Syntax Rules**

- Starter kits are installed via Composer.
- Each has an install command that publishes files.
- Breeze and Jetstream include frontend; Fortify does not.

**Constraints and Limitations**

- Starter kits are opinionated; customisation may require overriding published files.
- Jetstream requires Livewire or Inertia.js.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Installing Breeze**

**Command**

```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install && npm run dev
php artisan migrate
```

**Expected Output**

```
// Breeze publishes controllers, routes, and Blade views
// Authentication is fully functional at /login, /register, etc.
```

**Why This Output Occurs**

Breeze scaffolds a complete authentication system with Blade views and Tailwind CSS.

---

#### Real-World Cases with Explanation

**Case 1: Rapid MVP Development**

A startup uses Breeze to get authentication working in minutes, allowing the team to focus on core product features.

**Explanation**: Scaffolding dramatically reduces time-to-market for authentication.

---

### 8. Laravel Breeze

#### Definitions

**Core Definition**

Laravel Breeze is a minimal, simple implementation of Laravel's authentication features, providing Blade views, routes, and controllers for login, registration, password reset, email verification, and password confirmation.

**Technical Definition**

Breeze is a Composer package (`laravel/breeze`) that publishes authentication controllers, routes, and views to the application. It uses Tailwind CSS for styling and supports Blade, Livewire, or Inertia.js (with React/Vue) stacks. Breeze does not use Fortify; it is a standalone implementation.

**Beginner-Friendly Explanation**

Breeze is the easiest way to add authentication to a Laravel app. It gives you everything you need with a simple, clean interface.

#### Purposes

- To provide a lightweight, easy-to-understand authentication scaffold.
- To give developers full control over the authentication code.
- To serve as a learning tool for Laravel authentication.
- To support rapid prototyping and MVP development.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
composer require laravel/breeze --dev
php artisan breeze:install
php artisan migrate
npm install && npm run dev
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Published controllers | `app/Http/Controllers/Auth` |
| Published routes | `routes/auth.php` |
| Published views | `resources/views/auth` |

**Syntax Rules**

- Breeze is installed as a dev dependency.
- The install command publishes files.
- Frontend assets require `npm install` and `npm run dev`.

**Constraints and Limitations**

- Breeze is intentionally minimal; advanced features (teams, 2FA) require Jetstream or custom code.
- Breeze publishes code to the application, which must be maintained.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Customising Breeze Views**

**Published View**

```blade
<!-- resources/views/auth/login.blade.php -->
<form method="POST" action="{{ route('login') }}">
    @csrf
    <input type="email" name="email" required>
    <input type="password" name="password" required>
    <button type="submit">Log in</button>
</form>
```

**Expected Output**

```html
<!-- Custom login form rendered at /login -->
```

**Why This Output Occurs**

Breeze publishes the login view, which can be modified directly.

---

#### Real-World Cases with Explanation

**Case 1: Simple SaaS Application**

A small SaaS application uses Breeze for authentication because it needs only basic login and registration.

**Explanation**: Breeze is ideal for projects that do not require advanced authentication features.

---

### 9. Laravel Jetstream

#### Definitions

**Core Definition**

Laravel Jetstream is a more robust application starter kit that provides authentication, two-factor authentication, team management, API support via Sanctum, and optional frontend stacks (Livewire or Inertia.js).

**Technical Definition**

Jetstream is built on top of Fortify and provides a complete application scaffold. Unlike Breeze, Jetstream does not publish controllers or routes to the application; instead, its functionality is customised via "Action" classes. Jetstream includes features such as profile management, API token management, and team billing.

**Beginner-Friendly Explanation**

Jetstream is a full-featured starting point for applications that need more than just basic authentication—like teams, two-factor authentication, and API tokens.

#### Purposes

- To provide a complete application scaffold with advanced authentication features.
- To support team-based applications with membership and roles.
- To integrate with Livewire or Inertia.js for modern frontends.
- To offer API authentication out of the box via Sanctum.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
composer require laravel/jetstream
php artisan jetstream:install livewire
php artisan migrate
npm install && npm run dev
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Actions | Customisation points in `app/Actions/Jetstream`. |
| Livewire/Inertia | Frontend stack choice. |
| Features | Teams, API, 2FA, profile photos. |

**Syntax Rules**

- Jetstream requires Livewire or Inertia.js.
- Actions are used for customisation instead of controllers.
- Features are enabled/disabled in `config/jetstream.php`.

**Constraints and Limitations**

- Jetstream is more opinionated and complex than Breeze.
- Customisation requires understanding Action classes.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Enabling Two-Factor Authentication**

**Config**

```php
// config/jetstream.php
'features' => [
    Features::twoFactorAuthentication([
        'confirmPassword' => true,
    ]),
],
```

**Expected Output**

```
// 2FA setup is available in the user's profile settings
```

**Why This Output Occurs**

The `twoFactorAuthentication` feature is enabled, providing the necessary routes and views.

---

#### Real-World Cases with Explanation

**Case 1: Team Collaboration Platform**

A project management tool uses Jetstream for team management, user roles, and API tokens.

**Explanation**: Jetstream provides the infrastructure for complex multi-user applications.

---

### 10. Laravel Fortify

#### Definitions

**Core Definition**

Laravel Fortify is a frontend-agnostic authentication backend for Laravel that registers the routes and controllers needed for authentication features without providing any user interface.

**Technical Definition**

Fortify is a Composer package that implements all of Laravel's authentication features as a headless backend. It registers routes for login, registration, password reset, email verification, and two-factor authentication. Fortify is used internally by Jetstream and can be used independently with a custom frontend.

**Beginner-Friendly Explanation**

Fortify provides the "brain" of authentication—all the logic and routes—but you build your own login page and user interface.

#### Purposes

- To provide authentication logic without frontend opinions.
- To support custom frontends (SPAs, mobile apps, custom Blade views).
- To serve as the backend for Jetstream.
- To allow developers to use Laravel's authentication features without adopting a full starter kit.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
composer require laravel/fortify
php artisan fortify:install
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Routes | Registered by Fortify automatically. |
| Actions | Customisation points in `app/Actions/Fortify`. |
| Features | Configurable in `config/fortify.php`. |

**Syntax Rules**

- Fortify registers routes prefixed with `/` by default.
- Actions handle login, registration, password reset, etc.
- Features are enabled in `config/fortify.php`.

**Constraints and Limitations**

- Fortify does not provide views; you must build your own.
- Fortify is not required if using Breeze or Jetstream.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Customising the Login Action**

**Action**

```php
<?php
// app/Actions/Fortify/AuthenticateUser.php
namespace App\Actions\Fortify;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class AuthenticateUser
{
    public function __invoke(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials, $request->boolean('remember'))) {
            $request->session()->regenerate();
            return redirect()->intended('/dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ]);
    }
}
```

**Expected Output**

```
// Custom login logic is used when the /login route is hit
```

**Why This Output Occurs**

Fortify resolves the `AuthenticateUser` action for the login route, allowing custom behaviour.

---

#### Real-World Cases with Explanation

**Case 1: Custom SPA Frontend**

A team building a Vue SPA uses Fortify for the authentication backend and builds a custom Vue login page that posts to Fortify's routes.

**Explanation**: Fortify provides the backend without forcing a specific frontend technology.

---

### 11. Laravel Sanctum

#### Definitions

**Core Definition**

Laravel Sanctum is a lightweight authentication package for SPAs (single-page applications), mobile applications, and simple token-based APIs. It provides both API token issuance and SPA cookie-based authentication.

**Technical Definition**

Sanctum stores API tokens in a `personal_access_tokens` table. Tokens are issued via `$user->createToken()` and authenticated via the `auth:sanctum` middleware. For SPAs, Sanctum uses Laravel's `web` session guard with cookie-based authentication, providing CSRF protection without tokens.

**Beginner-Friendly Explanation**

Sanctum is the simple way to authenticate SPAs and mobile apps. It can use either tokens or cookies, depending on your needs.

#### Purposes

- To provide simple API token authentication without OAuth2 complexity.
- To authenticate SPAs using session cookies (more secure than tokens in the browser).
- To support mobile applications with long-lived personal access tokens.
- To offer token abilities for fine-grained access control.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
composer require laravel/sanctum
php artisan install:api
```

```php
// Issuing a token
$token = $user->createToken('token-name', ['server:update'])->plainTextToken;

// Protecting routes
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `createToken()` | Issues a new personal access token. |
| `auth:sanctum` | Middleware for token/SPA authentication. |
| `SANCTUM_STATEFUL_DOMAINS` | Config for SPA cookie authentication. |

**Syntax Rules**

- Sanctum is installed via `install:api`.
- The `HasApiTokens` trait must be added to the `User` model.
- SPA authentication requires stateful domain configuration.

**Constraints and Limitations**

- Sanctum does not support OAuth2.
- SPA authentication requires same-domain or configured stateful domains.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Issuing and Using a Token**

**Controller**

```php
public function createToken(Request $request)
{
    $token = $request->user()->createToken('api-token')->plainTextToken;
    return response()->json(['token' => $token]);
}
```

**Route**

```php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Expected Output**

```json
{"token":"1|abc123..."}
```

**Why This Output Occurs**

The token is generated and returned. Subsequent requests with `Authorization: Bearer 1|abc123...` are authenticated.

---

#### Real-World Cases with Explanation

**Case 1: SPA with Laravel API**

A React SPA uses Sanctum's SPA authentication (cookie-based) to communicate with a Laravel API, benefiting from CSRF protection.

**Explanation**: Sanctum is the recommended solution for first-party SPAs.

---

### 12. Laravel Passport

#### Definitions

**Core Definition**

Laravel Passport is a full OAuth2 server implementation for Laravel applications, providing a complete OAuth2 authorization server with support for access tokens, refresh tokens, and scopes.

**Technical Definition**

Passport is built on the League OAuth2 server and provides routes and middleware for issuing and validating OAuth2 tokens. It supports authorization code grant, password grant, client credentials grant, and personal access tokens. Passport is recommended when OAuth2 is required.

**Beginner-Friendly Explanation**

Passport is the full-featured OAuth2 solution. Use it when you need to let third-party applications access your API on behalf of users.

#### Purposes

- To provide full OAuth2 server capabilities.
- To support third-party integrations and API ecosystems.
- To issue access and refresh tokens with scopes.
- To enable authorization code flow with PKCE.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
composer require laravel/passport
php artisan install:api --passport
php artisan passport:keys
```

```php
// Protecting routes
Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `HasApiTokens` | Trait for the User model. |
| `auth:api` | Middleware for Passport authentication. |
| `passport:keys` | Generates encryption keys. |

**Syntax Rules**

- Passport is installed via `install:api --passport`.
- The `HasApiTokens` trait and `OAuthenticatable` interface must be added to the User model.
- The `api` guard must use the `passport` driver.

**Constraints and Limitations**

- Passport is more complex to set up than Sanctum.
- Not recommended for simple SPAs or mobile apps (use Sanctum instead).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Setting Up Passport**

**User Model**

```php
use Laravel\Passport\HasApiTokens;
use Laravel\Passport\Contracts\OAuthenticatable;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens;
}
```

**Config**

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

**Expected Output**

```
// Passport routes are registered at /oauth/*
```

**Why This Output Occurs**

Passport registers its OAuth2 routes and uses the `passport` driver for API authentication.

---

#### Real-World Cases with Explanation

**Case 1: Third-Party API Access**

A platform allows external developers to build integrations using OAuth2. Passport provides the authorization server.

**Explanation**: Passport is the correct choice when OAuth2 is a requirement.

---

### 13. Authentication Best Practices

#### Definitions

**Core Definition**

Authentication best practices are the recommended guidelines for implementing secure, maintainable, and performant authentication in Laravel applications.

**Technical Definition**

Best practices encompass password hashing, session security, rate limiting, token management, CSRF protection, and choosing the right authentication mechanism for the use case.

**Beginner-Friendly Explanation**

Best practices are the security rules and patterns that experienced developers follow to keep user accounts safe.

#### Key Best Practices

1. **Use Bcrypt or Argon2 for Password Hashing**: Laravel's `Hash` facade defaults to bcrypt. Never store plain-text passwords.

2. **Regenerate Session ID on Login**: Prevents session fixation attacks. Laravel does this automatically with Breeze/Jetstream, but manual implementations must call `$request->session()->regenerate()`.

3. **Rate Limit Login Attempts**: Apply `throttle:login` middleware or use `RateLimiter` to prevent brute-force attacks (e.g., 5 attempts per minute).

4. **Use CSRF Protection**: Laravel's `VerifyCsrfToken` middleware is enabled by default for web routes. Never disable it.

5. **Enable Two-Factor Authentication (2FA)**: For sensitive applications, enable 2FA via Fortify or Jetstream.

6. **Choose the Right Authentication Mechanism**:
   - Session for traditional web applications.
   - Sanctum for SPAs and mobile apps.
   - Passport for third-party OAuth2 integrations.

7. **Use HTTPS**: Always serve authentication pages over HTTPS to prevent credential interception.

8. **Set Secure Cookie Flags**: Use `HttpOnly`, `SameSite=Lax` or `Strict`, and `Secure` flags for session cookies.

9. **Hash Remember Tokens**: The `remember_token` should be hashed in the database.

10. **Revoke Tokens on Logout**: For token-based authentication, revoke tokens when the user logs out.

11. **Use Strong Password Rules**: Enforce minimum length, complexity, and check against breached password databases using `Password::defaults()`.

12. **Avoid Storing Sensitive Data in Tokens**: Tokens should be opaque; store user data server-side.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Rate Limiting Login**

**Route**

```php
Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:5,1'); // 5 attempts per minute
```

**Expected Output**

```
// After 5 failed attempts, the user is blocked for 1 minute
```

**Why This Output Occurs**

The `throttle` middleware limits requests to 5 per minute per IP address.

---

**Example 2: Session Regeneration**

**Controller**

```php
if (Auth::attempt($credentials)) {
    $request->session()->regenerate();
    return redirect()->intended('dashboard');
}
```

**Expected Output**

```
// A new session ID is issued after login, preventing session fixation
```

**Why This Output Occurs**

`regenerate()` creates a new session ID while preserving session data, preventing attackers from using a pre-set session ID.

---

#### Real-World Cases with Explanation

**Case 1: Banking Application**

A banking application uses session authentication with 2FA, rate limiting, and strict password policies. Sessions are regenerated on login, and all traffic is over HTTPS.

**Explanation**: High-security applications require multiple layers of protection.

---

## References

- Authentication – Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/authentication
- Laravel Sanctum – Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/sanctum
- Laravel Fortify – Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/fortify
- Laravel Passport – Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/passport
- Laravel Sanctum vs Passport: Which One Should You Use? – Bagisto — https://bagisto.com/en/laravel-sanctum-vs-passport-which-one-should-you-use/
- Laravel Auth Systems: Breeze, Jetstream, Fortify Compared – LinkedIn — https://www.linkedin.com/posts/saqib-hussain-99ab56287_laravel-authentication-systems-breeze-vs-activity-7425070704234184705-EeXO
- Authentication, Rate Limiting, and Session Fixation – GitHub (voku/agent-skills) — https://github.com/voku/agent-skills/blob/main/skills/laravel-owasp-security/rules/sec-authentication-rate-limiting.md
- ADR-002: Authentication Architecture – GitHub (Sourdough) — https://github.com/Sourdough-start/sourdough/blob/master/docs/adr/002-authentication-architecture.md
- Laravel Breeze vs Jetstream vs Fortify Compared 2026 – Laracopilot — https://laracopilot.com/blog/laravel-breeze-vs-jetstream-vs-fortify-compared/
- How to Add Auth to a Laravel App (Breeze, Fortify, Sanctum) – Beag — https://beag.io/how-to-add-auth-to-a-laravel-app-breeze-fortify-sanctum/

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Authentication features such as guards, providers, and session authentication have remained stable across these versions. Breeze, Jetstream, and Fortify have evolved, with Jetstream and Breeze being the primary starter kits in recent versions. Sanctum is the recommended solution for SPAs and mobile apps, while Passport is reserved for OAuth2 requirements. Developers should always consult the documentation for their specific Laravel version.