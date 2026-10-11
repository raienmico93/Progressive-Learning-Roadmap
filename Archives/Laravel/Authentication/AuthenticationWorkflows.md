# Laravel Core Authentication Workflows: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Core authentication workflows are the standard self-service pipelines that allow users to register, log in, log out, reset forgotten passwords, verify email addresses, and manage session security without administrative intervention.

**Technical Definition:** Laravel implements these workflows through the `Auth` facade, the `SessionGuard` (for stateful cookie-based authentication), `TokenGuard` (for stateless API tokens), and the `Password` broker for password resets. Email verification is handled by the `MustVerifyEmail` interface and signed URLs. Session security is enforced through session regeneration on login, `remember_token` management for "Remember Me" functionality, and `Auth::logoutOtherDevices()` for cross-device session invalidation. Laravel Fortify provides a headless backend implementation of these workflows with configurable features.

**Beginner-Friendly Explanation:** Laravel gives you a complete set of authentication tools out of the box. Users can register, log in (with or without staying logged in), log out, reset their password via email, verify their email address by clicking a link, and even log out of other devices if they suspect their account is compromised. These workflows follow security best practices automatically, so you don't have to build them from scratch.

### Key Characteristics

- **Self-Service Pipelines:** Users manage their own accounts without admin intervention.
- **Session Regeneration:** Session IDs are regenerated on login to prevent session fixation attacks .
- **Remember Me Tokens:** Long-lived cookies allow users to stay logged in across browser sessions using the `remember_token` column .
- **Token-Based Password Resets:** Secure, time-limited tokens are emailed to users for password resets, with throttling to prevent abuse .
- **Signed URLs for Verification:** Email verification links use signed URLs to prevent tampering .
- **Cross-Device Session Invalidation:** `Auth::logoutOtherDevices()` allows users to log out sessions on other devices .

### Prerequisites

- A Laravel application with the `users` table migration (including `remember_token` and `email_verified_at` columns).
- A `User` model extending `Illuminate\Foundation\Auth\User`.
- Session driver configured (database driver required for browser session management) .
- Mail configuration for password reset and verification emails.
- Laravel Fortify (optional, for headless authentication backend).

### Related Programming Areas

- **Laravel Fortify:** Headless authentication backend providing routes and controllers for all workflows .
- **Laravel Sanctum:** Session-based SPA authentication with CSRF protection .
- **Laravel Jetstream:** Full-stack scaffolding with browser session management UI .
- **Notifications:** Password reset and verification emails are dispatched via Laravel's notification system .

### Core Concepts / Features

1. Self-Service Pipelines (Registration, Login, Logout)
2. Session Management (Cookie Persistence vs. "Remember Me")
3. Security Fallbacks (Password Resets with Tokens and Throttling)
4. User Verification (Email Verification Lifecycle and Signed URLs)
5. Session Security (Cross-Device Session Invalidation)

---

## 1. Self-Service Pipelines: Secure Registration, Login, and Logout

### Definitions

**Core Definition:** Self-service pipelines are the authentication endpoints and workflows that allow users to create accounts, authenticate themselves, and terminate their sessions without administrative assistance.

**Technical Definition:** Registration dispatches the `Illuminate\Auth\Events\Registered` event, which triggers the `SendEmailVerificationNotification` listener if the `User` model implements `MustVerifyEmail` . Login uses `Auth::attempt()` or `Auth::login()`, which validates credentials via the `UserProvider` and stores the user ID in the session. Logout calls `Auth::logout()`, which clears the authentication state, invalidates the session, and regenerates the CSRF token .

**Beginner-Friendly Explanation:** Registration creates a new user account and (optionally) sends a verification email. Login checks credentials and starts a session. Logout ends the session and clears the user's authentication state. Laravel handles all the security details—session regeneration, CSRF protection, and secure password hashing—automatically.

### Purposes

- To allow users to create accounts with email and password.
- To authenticate users and establish a session or token-based authentication state.
- To securely terminate sessions and clear authentication state.
- To dispatch events for extending the registration workflow (e.g., sending welcome emails).
- To protect against session fixation and CSRF attacks during authentication.

### Syntax Rules and Structure

**Complete General Syntax — Registration:**

```php
use App\Models\User;
use Illuminate\Auth\Events\Registered;
use Illuminate\Support\Facades\Hash;

$user = User::create([
    'name' => $request->name,
    'email' => $request->email,
    'password' => Hash::make($request->password),
]);

// Dispatch event for email verification and other listeners
event(new Registered($user));

// Optionally log the user in immediately
Auth::login($user);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `User::create([...])` | Creates the user record with hashed password. |
| `event(new Registered($user))` | Triggers email verification and other listeners. |
| `Auth::login($user)` | Logs the user in after registration (optional). |

**Complete General Syntax — Login:**

```php
use Illuminate\Support\Facades\Auth;

$credentials = $request->validate([
    'email' => 'required|email',
    'password' => 'required',
]);

if (Auth::attempt($credentials, $request->boolean('remember'))) {
    $request->session()->regenerate();
    return redirect()->intended('dashboard');
}

return back()->withErrors(['email' => 'Invalid credentials.']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Auth::attempt($credentials, $remember)` | Validates credentials and logs in. |
| `$request->session()->regenerate()` | Prevents session fixation attacks. |
| `redirect()->intended()` | Redirects to the originally requested URL. |

**Complete General Syntax — Logout:**

```php
use Illuminate\Support\Facades\Auth;

Auth::logout();
$request->session()->invalidate();
$request->session()->regenerateToken();

return redirect('/');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Auth::logout()` | Clears authentication state. |
| `$request->session()->invalidate()` | Invalidates the entire session. |
| `$request->session()->regenerateToken()` | Generates a fresh CSRF token. |

**Syntax Rules:**
- Registration requires a `User` model with `name`, `email`, and `password` fillable attributes.
- Login requires a `UserProvider` (Eloquent by default) to retrieve users by credentials.
- The `remember` parameter is a boolean passed as the second argument to `Auth::attempt()`.
- Logout should invalidate the session and regenerate the CSRF token to prevent stale token usage .

**Constraints and Limitations:**
- **Session driver:** `SessionGuard` requires session middleware to be active. API routes using `TokenGuard` do not use sessions.
- **Password hashing:** Passwords must be hashed with `Hash::make()` before storage. Laravel's `User` model includes the `Notifiable` trait for password reset emails .
- **Rate limiting:** Login attempts should be rate-limited to prevent brute-force attacks .

### Multiple Annotated Complete Code Examples

**Example 1: Secure Registration with Email Verification**

```php
<?php
namespace App\Http\Controllers\Auth;

use App\Models\User;
use Illuminate\Auth\Events\Registered;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\Rules\Password;

class RegisterController extends Controller
{
    public function store(Request $request)
    {
        // Validate registration input
        $validated = $request->validate([
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
            'password' => ['required', 'confirmed', Password::min(8)->mixedCase()->numbers()],
        ]);

        // Create the user with a hashed password
        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        // Dispatch Registered event (triggers email verification if MustVerifyEmail)
        event(new Registered($user));

        // Log the user in
        Auth::login($user);

        return redirect()->route('dashboard');
    }
}
```

**Expected Output:** A new user is created with a hashed password. The `Registered` event fires, and if the `User` model implements `MustVerifyEmail`, a verification email is sent automatically. The user is logged in and redirected to the dashboard.

**Why:** `event(new Registered($user))` triggers Laravel's built-in listener that sends the verification email. `Auth::login($user)` establishes the session. `Hash::make()` securely hashes the password .

---

**Example 2: Login with Remember Me**

```php
<?php
namespace App\Http\Controllers\Auth;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function store(Request $request)
    {
        // Validate credentials
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        // Attempt login with "remember me" option
        if (Auth::attempt($credentials, $request->boolean('remember'))) {
            // Regenerate session to prevent fixation
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ])->onlyInput('email');
    }
}
```

**Expected Output:** On successful login with "Remember Me" checked, the user is authenticated and a long-lived `remember_token` is set. On failure, the user is redirected back with an error message.

**Why:** The second argument to `Auth::attempt()` is the "Remember Me" boolean. When `true`, Laravel generates a `remember_token` stored in the database and a long-lived cookie in the browser . `session()->regenerate()` prevents session fixation attacks .

---

**Example 3: Complete Logout with Session Cleanup**

```php
<?php
namespace App\Http\Controllers\Auth;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LogoutController extends Controller
{
    public function destroy(Request $request)
    {
        // Log out the user
        Auth::logout();

        // Invalidate the session (clears all session data)
        $request->session()->invalidate();

        // Regenerate the CSRF token
        $request->session()->regenerateToken();

        return redirect('/');
    }
}
```

**Expected Output:** The user is logged out, the session is destroyed, and a fresh CSRF token is generated. The user is redirected to the home page.

**Why:** `Auth::logout()` clears the authentication state. `session()->invalidate()` destroys the session data to prevent reuse. `session()->regenerateToken()` ensures the CSRF token is fresh .

### Real-World Cases

- **SaaS Applications:** Self-service registration with email verification for account creation.
- **E-commerce:** Login with "Remember Me" for returning customers.
- **Admin Panels:** Separate login pipelines with stricter rate limiting and 2FA.
- **API Applications:** Token-based registration and login using Laravel Sanctum .

### References

- Laravel Authentication (13.x) — https://laravel.com/framework/docs/authentication
- Laravel Authentication (11.x) — https://laravel.com/framework/docs/11.x/authentication
- Basic Authentication: Login/Logout (Laravel Learn) — https://laravel.com/learn/getting-started-with-laravel/basic-authentication-loginlogout

---

## 2. Session Management: Cookie Persistence vs. "Remember Me"

### Definitions

**Core Definition:** Session management refers to how Laravel persists authentication state across requests, either through standard session cookies (short-lived, browser-session-bound) or "Remember Me" long-lived cookies.

**Technical Definition:** Standard session authentication stores the user ID in the session store (file, database, Redis, etc.) and a session ID cookie in the browser. The session expires when the browser closes or after the configured session lifetime. "Remember Me" authentication generates a `remember_token` stored in the `users` table and a long-lived cookie (`remember_web_*`) in the browser. On subsequent visits, if the session has expired, Laravel checks the remember cookie, validates the token, and re-authenticates the user .

**Beginner-Friendly Explanation:** Normal login sessions last until you close your browser or the session times out. "Remember Me" creates a special cookie that lets Laravel log you back in automatically when you return—even days or weeks later—as long as the cookie is valid and the token matches your account.

### Purposes

- To persist authentication state across multiple HTTP requests.
- To provide a seamless login experience with "Remember Me" long-lived cookies.
- To securely store and rotate `remember_token` values.
- To allow session expiration configuration based on security requirements.
- To support session regeneration on login for security.

### Syntax Rules and Structure

**Complete General Syntax — Remember Me on Login:**

```php
if (Auth::attempt($credentials, $remember = true)) {
    $request->session()->regenerate();
    return redirect()->intended('dashboard');
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Auth::attempt($credentials, true)` | Enables "Remember Me" authentication. |
| `$request->session()->regenerate()` | Regenerates session ID after login. |

**Complete General Syntax — Remember Me Configuration:**

```php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

**Complete General Syntax — Database Preparation:**

```php
// Migration
Schema::table('users', function (Blueprint $table) {
    $table->rememberToken(); // Adds remember_token VARCHAR(100) nullable
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$table->rememberToken()` | Adds the `remember_token` column. |
| `Auth::attempt($credentials, true)` | Generates and stores a new `remember_token`. |

**Syntax Rules:**
- The `users` table must have a `remember_token` column (included in Laravel's default migration).
- The `User` model uses the `Authenticatable` trait, which provides `getRememberToken()` and `setRememberToken()` .
- The `remember_token` is rotated on each successful "Remember Me" authentication and on logout .
- Session lifetime is configured in `config/session.php` (`lifetime` key).

**Constraints and Limitations:**
- **Global remember token:** Laravel stores a single `remember_token` per user, shared across all devices. Invalidating one device's remember cookie requires cycling the token, which logs out all remembered devices .
- **Ghost devices:** When a session expires and is garbage-collected, the remember cookie may remain valid, creating "ghost" sessions that can silently re-authenticate .
- **Security:** "Remember Me" cookies should be treated as sensitive credentials. They are long-lived and should be protected with HTTPS .

### Multiple Annotated Complete Code Examples

**Example 1: Login with Remember Me**

```php
<?php
// Login form (Blade)
<form method="POST" action="/login">
    @csrf
    <input type="email" name="email" required>
    <input type="password" name="password" required>
    <input type="checkbox" name="remember" value="1">
    <label>Remember Me</label>
    <button type="submit">Log In</button>
</form>
```

```php
<?php
// LoginController
public function store(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials, $request->boolean('remember'))) {
        $request->session()->regenerate();
        return redirect()->intended('dashboard');
    }

    return back()->withErrors(['email' => 'Invalid credentials.']);
}
```

**Expected Output:** With "Remember Me" checked, Laravel generates a `remember_token` and sets a long-lived cookie. When the browser session expires, the user is automatically re-authenticated on their next visit.

**Why:** The second argument to `Auth::attempt()` enables "Remember Me." Laravel stores the token in the `remember_token` column and sets the `remember_web_*` cookie .

---

**Example 2: Session Configuration for Security**

```php
<?php
// config/session.php

return [
    'driver' => env('SESSION_DRIVER', 'database'),
    'lifetime' => env('SESSION_LIFETIME', 120), // 120 minutes
    'expire_on_close' => false,
    'encrypt' => false,
    'secure' => env('SESSION_SECURE_COOKIE', true), // HTTPS only
    'http_only' => true, // Not accessible via JavaScript
    'same_site' => 'lax', // CSRF protection
];
```

**Expected Output:** Sessions expire after 120 minutes of inactivity, cookies are HTTPS-only and HTTP-only, with `SameSite=Lax` for CSRF protection.

**Why:** These configuration values harden session security. `secure => true` prevents cookie transmission over HTTP. `http_only => true` prevents XSS access. `same_site => 'lax'` provides CSRF protection .

---

**Example 3: Custom User Provider with Remember Token**

```php
<?php
// app/Providers/AuthServiceProvider.php

use App\Extensions\MongoUserProvider;
use Illuminate\Support\Facades\Auth;

public function boot(): void
{
    Auth::provider('mongo', function ($app, array $config) {
        return new MongoUserProvider($app['mongo.connection']);
    });
}
```

```php
// config/auth.php
'providers' => [
    'users' => [
        'driver' => 'mongo',
    ],
],
```

**Expected Output:** A custom provider handles `retrieveByToken()` and `updateRememberToken()` for MongoDB-backed users.

**Why:** Custom providers must implement the `UserProvider` contract, including `retrieveByToken()` and `updateRememberToken()` for "Remember Me" functionality .

### Real-World Cases

- **Consumer Applications:** "Remember Me" for convenience on personal devices.
- **Banking/Healthcare:** Shorter session lifetimes with `expire_on_close => true` and no "Remember Me."
- **Multi-Device Users:** Understanding the global `remember_token` limitation and considering packages like `proai/laravel-footprint` for per-device tokens .
- **SPA Applications:** Session-based authentication with Sanctum and CSRF protection .

### References

- Laravel Authentication (11.x) — https://laravel.com/framework/docs/11.x/authentication
- Laravel Authentication (13.x) — https://laravel.com/framework/docs/authentication
- Laravel Footprint Package — https://packagist.org/packages/proai/laravel-footprint

---

## 3. Security Fallbacks: Dynamic Password Resets

### Definitions

**Core Definition:** Password resets allow users who have forgotten their passwords to securely set a new one via a time-limited, token-based email link, with throttling to prevent abuse.

**Technical Definition:** Laravel's password reset system uses the `Password` broker (configured in `config/auth.php` under `passwords`). The `sendResetLink()` method generates a token stored in the `password_reset_tokens` table (or cache driver), and sends a notification email with a signed URL containing the token. The `reset()` method validates the token, updates the password, and deletes the token. Throttling is configured via the `throttle` key (seconds between reset requests) .

**Beginner-Friendly Explanation:** If you forget your password, you enter your email, and Laravel sends you a link with a secret token. Clicking the link lets you set a new password. The token expires after a set time, and you can't request new links too frequently—this prevents abuse and keeps your account secure.

### Purposes

- To allow users to regain access to their accounts after forgetting passwords.
- To securely generate and validate time-limited reset tokens.
- To throttle reset requests to prevent abuse and email spam.
- To integrate with Laravel's notification system for email delivery.
- To support custom password reset drivers (database, cache, or custom).

### Syntax Rules and Structure

**Complete General Syntax — Configuration:**

```php
// config/auth.php
'passwords' => [
    'users' => [
        'driver' => 'database', // or 'cache'
        'provider' => 'users',
        'table' => 'password_reset_tokens',
        'expire' => 60, // Token expiry in minutes
        'throttle' => 60, // Seconds between requests
    ],
],
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'driver' => 'database'` | Uses the database token repository. |
| `'table' => 'password_reset_tokens'` | Table storing tokens. |
| `'expire' => 60` | Token expires after 60 minutes. |
| `'throttle' => 60` | Minimum seconds between reset requests. |

**Complete General Syntax — Requesting Reset Link:**

```php
use Illuminate\Support\Facades\Password;

$status = Password::sendResetLink(
    $request->only('email')
);

return $status === Password::ResetLinkSent
    ? back()->with(['status' => __($status)])
    : back()->withErrors(['email' => __($status)]);
```

**Complete General Syntax — Resetting Password:**

```php
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Password;

$status = Password::reset(
    $request->only('email', 'password', 'password_confirmation', 'token'),
    function (User $user, string $password) {
        $user->forceFill([
            'password' => Hash::make($password),
        ])->setRememberToken(Str::random(60));

        $user->save();

        event(new PasswordReset($user));
    }
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Password::sendResetLink()` | Generates token and sends email. |
| `Password::reset()` | Validates token and calls callback. |
| `setRememberToken(Str::random(60))` | Invalidates remember tokens. |

**Syntax Rules:**
- The `User` model must implement `CanResetPassword` (included in Laravel's default `User` model).
- The `User` model must use the `Notifiable` trait for email notifications .
- The `password_reset_tokens` table must exist (included in default migration).
- Tokens expire after the configured `expire` time.
- Throttling prevents more than one reset request per `throttle` seconds.

**Constraints and Limitations:**
- **Cache driver:** The `cache` driver does not require a database table but is cleared by `cache:clear` unless a separate cache store is configured .
- **Email enumeration:** Laravel's default behaviour returns the same status regardless of whether the email exists, preventing user enumeration.
- **Trusted hosts:** Password reset functionality requires `TrustHosts` middleware configuration to prevent host header attacks .

### Multiple Annotated Complete Code Examples

**Example 1: Forgot Password Form and Handler**

```php
<?php
// routes/web.php
Route::get('/forgot-password', function () {
    return view('auth.forgot-password');
})->middleware('guest')->name('password.request');

Route::post('/forgot-password', function (Request $request) {
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::ResetLinkSent
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
})->middleware('guest')->name('password.email');
```

**Expected Output:** Submitting a valid email sends a reset link. The status message is displayed. Invalid or non-existent emails receive the same generic message.

**Why:** `Password::sendResetLink()` retrieves the user, generates a token, stores it, and sends the notification. The status is checked to determine the response .

---

**Example 2: Password Reset Handler**

```php
<?php
// routes/web.php
Route::get('/reset-password/{token}', function (string $token) {
    return view('auth.reset-password', ['token' => $token]);
})->middleware('guest')->name('password.reset');

Route::post('/reset-password', function (Request $request) {
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function (User $user, string $password) {
            $user->forceFill([
                'password' => Hash::make($password),
            ])->setRememberToken(Str::random(60));

            $user->save();

            event(new PasswordReset($user));
        }
    );

    return $status === Password::PasswordReset
        ? redirect()->route('login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
})->middleware('guest')->name('password.update');
```

**Expected Output:** A valid token allows the user to set a new password. The `remember_token` is reset, invalidating all "Remember Me" sessions. The user is redirected to login with a success message.

**Why:** `Password::reset()` validates the token and calls the callback to update the password. `setRememberToken(Str::random(60))` invalidates all existing "Remember Me" cookies .

---

**Example 3: Custom Password Reset Driver (Cache)**

```php
<?php
// config/auth.php
'passwords' => [
    'users' => [
        'driver' => 'cache',
        'provider' => 'users',
        'store' => 'passwords',
        'expire' => 60,
        'throttle' => 60,
    ],
],
```

```php
// config/cache.php
'stores' => [
    'passwords' => [
        'driver' => 'redis',
        'connection' => 'default',
    ],
],
```

**Expected Output:** Password reset tokens are stored in a dedicated Redis cache store, independent of the default cache.

**Why:** The `cache` driver uses the configured cache store. A separate `store` prevents `cache:clear` from flushing password reset data .

### Real-World Cases

- **Consumer Applications:** Standard email-based password reset for all users.
- **High-Security Applications:** Shorter token expiry (15 minutes) and stricter throttling.
- **Enterprise SSO:** Password reset may be disabled if users authenticate via SSO.
- **Multi-Tenant:** Password reset tokens scoped per tenant database.

### References

- Resetting Passwords (Laravel 13.x) — https://laravel.com/framework/docs/13.x/passwords

---

## 4. User Verification: Email Verification Lifecycle and Signed URLs

### Definitions

**Core Definition:** Email verification is the process of confirming that a user owns the email address they registered with, using a signed URL sent to that address.

**Technical Definition:** The `User` model implements `Illuminate\Contracts\Auth\MustVerifyEmail` (which provides the `MustVerifyEmail` trait) . On registration, the `Registered` event triggers `SendEmailVerificationNotification`, which sends a notification containing a signed URL to `/email/verify/{id}/{hash}`. The `EmailVerificationRequest` validates the `id` and `hash` parameters, and calling `fulfill()` marks the email as verified (sets `email_verified_at`) and dispatches the `Verified` event. The `verified` middleware protects routes that require verification .

**Beginner-Friendly Explanation:** When you register, Laravel sends you an email with a special link. Clicking the link confirms your email address. Until you click it, you can't access certain parts of the application. The link is "signed" so it can't be tampered with—Laravel verifies the signature before accepting it.

### Purposes

- To confirm that users own the email addresses they register with.
- To prevent spam and fake accounts by requiring a valid email.
- To gate access to application features until verification is complete.
- To use signed URLs to prevent tampering with verification links.
- To dispatch events for extending verification behaviour.

### Syntax Rules and Structure

**Complete General Syntax — Model Preparation:**

```php
<?php
namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `implements MustVerifyEmail` | Enables email verification. |
| `use Notifiable` | Required for sending notifications. |

**Complete General Syntax — Routes:**

```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

// Verification notice
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');

// Verification handler (signed URL)
Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();
    return redirect('/home');
})->middleware(['auth', 'signed'])->name('verification.verify');

// Resend verification
Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();
    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

**Component Breakdown:**

| Route | Purpose |
|-------|---------|
| `verification.notice` | Displays "verify your email" page. |
| `verification.verify` | Handles signed URL click. |
| `verification.send` | Resends verification email. |

**Complete General Syntax — Protecting Routes:**

```php
Route::get('/dashboard', function () {
    // Only verified users
})->middleware(['auth', 'verified']);
```

**Syntax Rules:**
- The `User` model must implement `MustVerifyEmail` .
- The `users` table must have an `email_verified_at` column .
- The `verification.notice` route name is required for the `verified` middleware to redirect unverified users .
- The verification URL uses the `signed` middleware to validate the URL signature .
- The `EmailVerificationRequest` automatically validates the `id` and `hash` parameters .

**Constraints and Limitations:**
- **Signed URL expiry:** Signed URLs can have an expiration (e.g., `URL::temporarySignedRoute()`), but Laravel's default verification URL does not expire by default.
- **Resend throttling:** The resend route should be throttled to prevent email abuse .
- **Middleware order:** The `verified` middleware should come after `auth` on protected routes.

### Multiple Annotated Complete Code Examples

**Example 1: Complete Email Verification Setup**

```php
<?php
// app/Models/User.php
namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;
}
```

```php
<?php
// routes/web.php
use Illuminate\Foundation\Auth\EmailVerificationRequest;
use Illuminate\Http\Request;

Route::middleware('auth')->group(function () {
    // Notice page
    Route::get('/email/verify', function () {
        return view('auth.verify-email');
    })->name('verification.notice');

    // Verification handler (signed URL)
    Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
        $request->fulfill();
        return redirect('/home');
    })->middleware('signed')->name('verification.verify');

    // Resend
    Route::post('/email/verification-notification', function (Request $request) {
        $request->user()->sendEmailVerificationNotification();
        return back()->with('message', 'Verification link sent!');
    })->middleware('throttle:6,1')->name('verification.send');
});
```

**Expected Output:** Unverified users visiting protected routes are redirected to `/email/verify`. Clicking the link in the verification email marks the email as verified and redirects to `/home`.

**Why:** The `MustVerifyEmail` interface enables the verification lifecycle. The `signed` middleware validates the URL signature. `$request->fulfill()` marks the email as verified and dispatches the `Verified` event .

---

**Example 2: Verification with Custom Redirect**

```php
<?php
// routes/web.php
Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();

    // Custom redirect based on user role
    return $request->user()->isAdmin()
        ? redirect()->route('admin.dashboard')
        : redirect()->route('dashboard');
})->middleware(['auth', 'signed'])->name('verification.verify');
```

**Expected Output:** Admins are redirected to the admin dashboard after verification; regular users to the standard dashboard.

**Why:** The `EmailVerificationRequest` provides access to the authenticated user, allowing custom redirect logic based on user attributes .

### Real-World Cases

- **SaaS Applications:** Require email verification before accessing the dashboard.
- **E-commerce:** Verify email before allowing checkout or account creation.
- **Community Platforms:** Reduce spam by requiring verification before posting.
- **Enterprise:** Combined with SSO, email verification may be handled by the identity provider.

### References

- Email Verification (Laravel master) — https://laravel.com/framework/docs/master/verification
- MustVerifyEmail API — https://api.laravel.com/docs/master/Illuminate/Auth/MustVerifyEmail.html
- VerifiesEmails Trait API — https://api.laravel.com/docs/6.x/Illuminate/Foundation/Auth/VerifiesEmails.html

---

## 5. Session Security: Dynamic Session Invalidation Across Other Devices

### Definitions

**Core Definition:** Cross-device session invalidation allows users to log out sessions on other devices while remaining logged in on the current device, or to log out all other sessions simultaneously.

**Technical Definition:** Laravel provides `Auth::logoutOtherDevices($password)` which requires the user's current password, rehashes it, and updates the `password` column. This invalidates all other sessions because Laravel's `AuthenticateSession` middleware checks a hash of the password stored in the session against the current password hash . Jetstream provides a UI for this feature, requiring the `database` session driver. Packages like `proai/laravel-footprint` extend this with per-device remember tokens and session tracking .

**Beginner-Friendly Explanation:** If you logged in on a public computer and forgot to log out, you can log out of all other devices from your current device. Laravel does this by changing the password hash in your session, which makes all other sessions invalid. You stay logged in because your current session gets the new hash.

### Purposes

- To allow users to secure their account if they suspect unauthorized access.
- To log out sessions on other devices without affecting the current session.
- To invalidate all "Remember Me" cookies across devices.
- To provide visibility into active sessions per user.
- To require password confirmation before sensitive session operations.

### Syntax Rules and Structure

**Complete General Syntax — Logout Other Devices:**

```php
use Illuminate\Support\Facades\Auth;

// Requires the user's current password
Auth::logoutOtherDevices($request->password);

return back()->with('status', 'Other sessions logged out.');
```

**Complete General Syntax — With Password Rehashing:**

```php
// Jetstream/Fortify action
public function logoutOtherBrowserSessions(Request $request)
{
    $request->validate([
        'password' => ['required', 'current_password'],
    ]);

    Auth::logoutOtherDevices($request->password);

    return back()->with('status', 'other-browser-sessions-logged-out');
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Auth::logoutOtherDevices($password)` | Invalidates other sessions. |
| `current_password` rule | Ensures the provided password is correct. |

**Complete General Syntax — Middleware Requirement:**

```php
// Add to web middleware group or specific routes
protected $middlewareGroups = [
    'web' => [
        // ...
        \Illuminate\Session\Middleware\AuthenticateSession::class,
    ],
];
```

**Syntax Rules:**
- `Auth::logoutOtherDevices()` requires the user's current password as an argument.
- The `AuthenticateSession` middleware must be active for password-based session validation.
- The session driver must be `database` for Jetstream's browser session management .
- The `current_password` validation rule ensures the provided password matches .

**Constraints and Limitations:**
- **Global remember token:** Laravel's single `remember_token` per user means logging out other devices also invalidates all remembered devices .
- **Ghost devices:** Expired sessions with valid remember cookies may not appear in the session list .
- **Password required:** The built-in `logoutOtherDevices()` requires the user's password. Packages like `proai/laravel-footprint` remove this requirement .
- **Session driver:** Database session driver is required for Jetstream's session management UI .

### Multiple Annotated Complete Code Examples

**Example 1: Logout Other Devices with Password**

```php
<?php
namespace App\Http\Controllers\Profile;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class SessionController extends Controller
{
    public function destroyOthers(Request $request)
    {
        // Validate current password
        $request->validate([
            'password' => ['required', 'current_password'],
        ]);

        // Log out all other devices
        Auth::logoutOtherDevices($request->password);

        return back()->with('status', 'Other browser sessions logged out successfully.');
    }
}
```

**Expected Output:** All other sessions for the user are invalidated. The current session remains active. The user's password is rehashed (same value, new hash), which invalidates other sessions via the `AuthenticateSession` middleware.

**Why:** `Auth::logoutOtherDevices($password)` updates the password hash. The `AuthenticateSession` middleware compares the session's stored hash with the current password hash. Other sessions have the old hash and are therefore invalidated .

---

**Example 2: Jetstream Browser Session Management**

```blade
{{-- resources/views/profile/logout-other-browser-sessions-form.blade.php --}}
<form wire:submit="logoutOtherBrowserSessions">
    <div>
        <x-label for="password" value="Password" />
        <x-input id="password" type="password" wire:model="password" />
        <x-input-error for="password" />
    </div>

    <x-button type="submit">Logout Other Browser Sessions</x-button>
</form>

@if (count($this->sessions) > 0)
    @foreach ($this->sessions as $session)
        <div>
            <div>{{ $session->agent->platform() }} - {{ $session->agent->browser() }}</div>
            <div>{{ $session->ip_address }}</div>
            @if ($session->is_current_device)
                <div class="text-green-500 font-semibold">This device</div>
            @else
                <div>Last active {{ $session->last_active }}</div>
            @endif
        </div>
    @endforeach
@endif
```

**Expected Output:** The user sees a list of active sessions with device, IP, and last activity. Clicking "Logout Other Browser Sessions" invalidates all other sessions.

**Why:** Jetstream uses `Illuminate\Session\Middleware\AuthenticateSession` and the `database` session driver to list and invalidate sessions .

---

**Example 3: Per-Device Session Tracking with Package**

```php
<?php
// Using proai/laravel-footprint
use Proai\Footprint\Facades\Footprint;

// Log out a specific device
Footprint::logoutDevice($sessionId);

// Log out all other devices without password
Footprint::logoutOtherDevices();

// Get all active sessions
$sessions = Footprint::getSessions($user);
```

**Expected Output:** The package tracks per-device sessions with unique remember tokens, allowing selective logout without password requirement.

**Why:** The package extends Laravel's session guard to store per-device remember tokens, solving the global token limitation .

### Real-World Cases

- **Public Computers:** Log out other sessions after using a shared device.
- **Account Compromise:** Users can secure their account by logging out all other sessions.
- **Multi-Device Users:** Manage active sessions across phone, tablet, and desktop.
- **Enterprise Security:** Enforce session invalidation on password change or suspicious activity.

### References

- Browser Sessions (Laravel Jetstream) — https://jetstream.laravel.com/features/browser-sessions.html
- Laravel Footprint Package — https://packagist.org/packages/proai/laravel-footprint
- Laravel Authentication (13.x) — https://laravel.com/framework/docs/authentication

---

## References

- Laravel Authentication (13.x) — https://laravel.com/framework/docs/authentication
- Laravel Authentication (11.x) — https://laravel.com/framework/docs/11.x/authentication
- Resetting Passwords (Laravel 13.x) — https://laravel.com/framework/docs/13.x/passwords
- Email Verification (Laravel master) — https://laravel.com/framework/docs/master/verification
- Laravel Nova Authentication — https://nova.laravel.com/docs/v5/customization/authentication
- Browser Sessions (Laravel Jetstream) — https://jetstream.laravel.com/features/browser-sessions.html
- Basic Authentication: Login/Logout (Laravel Learn) — https://laravel.com/learn/getting-started-with-laravel/basic-authentication-loginlogout
- MustVerifyEmail API — https://api.laravel.com/docs/master/Illuminate/Auth/MustVerifyEmail.html
- Authenticatable API — https://api.laravel.com/docs/13.x/Illuminate/Auth/Authenticatable.html
- Laravel Footprint Package — https://packagist.org/packages/proai/laravel-footprint
- User Sessions Laravel Package — https://packagist.org/packages/rene-roscher/user-sessions-laravel
- Laravel Fortify Skill (GitHub) — https://github.com/whisper-money/whisper-money/blob/a1648fc4c5556b63b98d76edce1b0bcb7f4f4f7d/.claude/skills/developing-with-fortify/SKILL.md
- ADR-002: Authentication Architecture (GitHub) — https://raw.githubusercontent.com/Sourdough-start/sourdough/refs/heads/master/docs/adr/002-authentication-architecture.md
- VerifiesEmails Trait API — https://api.laravel.com/docs/6.x/Illuminate/Foundation/Auth/VerifiesEmails.html