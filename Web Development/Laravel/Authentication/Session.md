# Laravel Session Authentication: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Session Authentication is a stateful authentication mechanism that verifies user identity by comparing submitted credentials against stored records and persists the authenticated state across multiple HTTP requests using server-side sessions and browser cookies.

**Technical Definition**

Session authentication in Laravel is implemented by the `SessionGuard` class (`Illuminate\Auth\SessionGuard`). When a user submits credentials, the guard retrieves the user via the configured `UserProvider`, verifies the password using `Hash::check()`, and stores the user's identifier in the session. The session itself is managed by Laravel's session subsystem, which persists session data to a configured driver (file, database, Redis, etc.) and identifies the session via a cookie (default: `laravel_session`). On subsequent requests, the session middleware loads the session from the cookie, and the session guard retrieves the user ID from the session, loading the user via the provider.

**Beginner-Friendly Explanation**

Session authentication is the standard "log in and stay logged in" approach for websites. When you log in with your email and password, Laravel checks if they are correct and then remembers you by storing a session ID in a cookie. Every time you visit a new page, your browser sends that cookie, and Laravel knows who you are. It is like getting a wristband at a concert—you show your ID once at the entrance, and the wristband lets you come and go without showing your ID again.

---

### Key Characteristics

- **Session Guard**: The default `web` guard uses the `session` driver, which stores authentication state in the server-side session.
- **Cookie-Based Session Identification**: A session cookie (default: `laravel_session`) identifies the session on the client side.
- **Persistent Authentication**: Sessions persist across requests until they expire or the user logs out.
- **Remember Me**: Optional persistent authentication via a `remember_token` column, keeping users logged in indefinitely or until they log out.
- **CSRF Protection**: Session authentication includes automatic CSRF token generation and validation.
- **Multiple Guards**: Supports multiple authentication guards (e.g., `web` for users, `admin` for administrators).
- **Driver Flexibility**: Session data can be stored using file, database, Redis, Memcached, DynamoDB, or cookie drivers.
- **Security Features**: Session ID regeneration on login, rate limiting, encryption, and secure cookie flags.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A configured database** with a `users` table containing `email`, `password`, and a nullable `remember_token` column of at least 100 characters.
- **Session middleware** enabled for the routes (the `web` middleware group includes `StartSession` and `VerifyCsrfToken`).
- **A session driver** configured in `config/session.php` or `.env`.

---

### Related Programming Areas

- **Authentication Guards**: Session authentication is implemented by the `SessionGuard`; other guards include `TokenGuard` and Sanctum/Passport guards.
- **Session Management**: Laravel's session subsystem handles session data storage, retrieval, and expiration.
- **CSRF Protection**: Session authentication relies on CSRF tokens for form submissions.
- **Middleware**: The `auth` middleware protects routes using session authentication.
- **Eloquent ORM**: The `User` model is retrieved by the session guard via the Eloquent user provider.
- **Authorization**: After authentication, authorization (permissions, policies) determines what the user can do.

---

## Core Concepts

---

### 1. Session Authentication

#### Definitions

**Core Definition**

Session authentication is a stateful authentication mechanism where the user's authenticated identity is stored in a server-side session and referenced by a browser cookie on subsequent requests.

**Technical Definition**

Laravel's `SessionGuard` implements session authentication. When `Auth::attempt()` is called, the guard retrieves the user by the provided credentials (e.g., email), verifies the password hash, and if successful, stores the user's ID in the session via `$this->session->put($this->getName(), $user->getAuthIdentifier())`. The session is persisted by the session driver, and a cookie (`laravel_session` by default) is sent to the browser. On each subsequent request, the `StartSession` middleware loads the session, and the `SessionGuard` retrieves the user ID, loading the user via the provider.

**Beginner-Friendly Explanation**

Session authentication is how Laravel remembers who you are after you log in. It stores your user ID in a session on the server and gives your browser a cookie with a session ID. Every time you visit a page, your browser sends the cookie, and Laravel looks up your session to know who you are.

#### Purposes

- To verify user identity before granting access to protected resources.
- To maintain authentication state across multiple HTTP requests (which are stateless by nature).
- To provide a seamless login experience for browser-based users.
- To serve as the foundation for authorization (what the user is allowed to do).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Auth;

// Attempt to authenticate
if (Auth::attempt(['email' => $email, 'password' => $password])) {
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
| `$request->session()->regenerate()` | Regenerates the session ID to prevent fixation. |

**Syntax Rules**

- `attempt()` accepts an array of credentials, with `password` automatically hashed and compared.
- The `remember` option is passed as the second argument.
- Session ID regeneration is recommended after successful login.
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
            'email'    => ['required', 'email'],
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

**Explanation**: Session authentication gates access to private areas of the application.

---

### 2. Login

#### Definitions

**Core Definition**

Login is the process by which a user submits credentials (typically email and password), which are verified against stored records, resulting in an authenticated session.

**Technical Definition**

Laravel's login process involves calling `Auth::attempt($credentials)`, which retrieves the user via the configured provider, verifies the password using `Hash::check()`, and if successful, calls `Auth::login($user)`. The `login()` method stores the user's identifier in the session and optionally sets a "remember me" cookie. The session ID should be regenerated after login to prevent session fixation attacks.

**Beginner-Friendly Explanation**

Login is when you enter your email and password to prove who you are. If they are correct, Laravel logs you in and remembers you.

#### Purposes

- To verify user identity before granting access to protected resources.
- To establish an authenticated session for subsequent requests.
- To provide a secure entry point into the application.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Basic login
if (Auth::attempt(['email' => $email, 'password' => $password])) {
    $request->session()->regenerate();
    return redirect()->intended('dashboard');
}

// Login with remember me
if (Auth::attempt($credentials, $remember)) {
    // ...
}

// Login by user instance
Auth::login($user);

// Login by ID
Auth::loginUsingId(1);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `attempt($credentials)` | Authenticates and logs in. |
| `login($user)` | Logs in a user instance directly. |
| `loginUsingId($id)` | Logs in a user by primary key. |
| `$remember` | Boolean for "remember me". |

**Syntax Rules**

- `attempt()` accepts an array of credentials.
- The `remember` option is passed as a boolean second argument.
- Session regeneration is recommended after login.
- `login()` and `loginUsingId()` accept an optional `$remember` boolean.

**Constraints and Limitations**

- Login attempts should be rate-limited to prevent brute-force attacks.
- Session regeneration is necessary to prevent session fixation.

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

**Example 2: Login by ID**

```php
Auth::loginUsingId(1);
```

**Expected Output**

```
// The user with ID 1 is logged in
```

**Why This Output Occurs**

`loginUsingId()` retrieves the user by primary key and logs them in.

---

#### Real-World Cases with Explanation

**Case 1: Standard Login Form**

A login form submits credentials to a controller, which uses `Auth::attempt()` to authenticate. On success, the user is redirected to the dashboard; on failure, the form is redisplayed with errors.

**Explanation**: The login flow is the gateway to the application.

---

### 3. Logout

#### Definitions

**Core Definition**

Logout is the process of terminating a user's authenticated session, removing their authentication state and invalidating the session.

**Technical Definition**

Logout is performed via `Auth::logout()`, which removes the user identifier from the session, invalidates the session, and regenerates the CSRF token. The `logout()` method also deletes the "remember me" cookie if it exists. For enhanced security, `logoutOtherDevices()` can be used to invalidate sessions on other devices.

**Beginner-Friendly Explanation**

Logout is when you click the "Log out" button and Laravel forgets who you are. It clears your session and any "remember me" cookie.

#### Purposes

- To terminate the authenticated session securely.
- To prevent unauthorized access after the user has finished.
- To clear sensitive session data.
- To invalidate the "remember me" cookie.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Auth;

public function logout(Request $request)
{
    Auth::logout();

    $request->session()->invalidate();
    $request->session()->regenerateToken();

    return redirect('/');
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Auth::logout()` | Logs the user out and clears auth data. |
| `$request->session()->invalidate()` | Flushes all session data. |
| `$request->session()->regenerateToken()` | Regenerates the CSRF token. |
| `Auth::logoutOtherDevices($password)` | Logs out all other sessions. |

**Syntax Rules**

- `logout()` clears the authenticated user from the session.
- `invalidate()` flushes all session data and regenerates the session ID.
- `regenerateToken()` regenerates the CSRF token.
- `logoutOtherDevices()` requires the user's current password.

**Constraints and Limitations**

- `invalidate()` destroys all session data, so flash messages should be set after logout if needed.
- `logoutOtherDevices()` requires the user to provide their current password.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Standard Logout**

**Controller**

```php
public function logout(Request $request)
{
    Auth::logout();
    $request->session()->invalidate();
    $request->session()->regenerateToken();

    return redirect('/');
}
```

**Expected Output**

```
// User is logged out, session is invalidated, CSRF token regenerated
// Redirect to homepage
```

**Why This Output Occurs**

`Auth::logout()` clears the authentication state, `invalidate()` flushes session data, and `regenerateToken()` creates a new CSRF token.

---

**Example 2: Logout Other Devices**

```php
Auth::logoutOtherDevices($request->current_password);
```

**Expected Output**

```
// All other sessions for the user are invalidated
```

**Why This Output Occurs**

`logoutOtherDevices()` updates the user's `remember_token` and invalidates sessions on other devices.

---

#### Real-World Cases with Explanation

**Case 1: Secure Logout**

An application logs users out and invalidates the session to ensure no residual authentication data remains.

**Explanation**: Proper logout is essential for security, especially on shared devices.

---

### 4. Registration

#### Definitions

**Core Definition**

Registration is the process by which a new user creates an account, typically providing a name, email address, and password, which are validated and stored in the database.

**Technical Definition**

Registration involves validating user input (typically via a form request), creating a new `User` model instance with a hashed password, saving it to the database, and optionally logging the user in. The `Registered` event is fired, which can trigger email verification notifications. Laravel's starter kits (Breeze, Jetstream) provide complete registration scaffolding.

**Beginner-Friendly Explanation**

Registration is when a new user signs up for your application by providing their details and choosing a password.

#### Purposes

- To allow new users to create accounts.
- To validate and securely store user credentials.
- To optionally log in the user immediately after registration.
- To trigger post-registration actions (e.g., welcome email, email verification).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Illuminate\Auth\Events\Registered;

public function register(Request $request)
{
    $validated = $request->validate([
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
        'password' => ['required', 'confirmed', Password::defaults()],
    ]);

    $user = User::create([
        'name' => $validated['name'],
        'email' => $validated['email'],
        'password' => Hash::make($validated['password']),
    ]);

    event(new Registered($user));

    Auth::login($user);

    return redirect()->route('dashboard');
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Validation | Ensures name, email, and password meet requirements. |
| `User::create()` | Creates the user with a hashed password. |
| `event(new Registered($user))` | Fires the Registered event. |
| `Auth::login($user)` | Logs in the new user. |

**Syntax Rules**

- The email must be unique.
- The password must be confirmed (`password_confirmation` field).
- The `Registered` event should be fired to trigger verification emails.
- `Hash::make()` or the `hashed` cast should be used for passwords.

**Constraints and Limitations**

- Registration should be rate-limited to prevent abuse.
- Email verification can be enforced by implementing `MustVerifyEmail`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Registration**

**Controller**

```php
public function register(Request $request)
{
    $validated = $request->validate([
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
        'password' => ['required', 'confirmed', Password::defaults()],
    ]);

    $user = User::create([
        'name' => $validated['name'],
        'email' => $validated['email'],
        'password' => Hash::make($validated['password']),
    ]);

    event(new Registered($user));

    Auth::login($user);

    return redirect()->route('dashboard');
}
```

**Expected Output**

```
// User is created, logged in, and redirected to dashboard
```

**Why This Output Occurs**

The validated data is used to create a new user, the `Registered` event is fired (triggering verification email if applicable), and the user is logged in.

---

#### Real-World Cases with Explanation

**Case 1: SaaS Signup**

A SaaS application uses registration to onboard new customers, with email verification required before accessing the dashboard.

**Explanation**: Registration is the entry point for new users.

---

### 5. Password Reset

#### Definitions

**Core Definition**

Password reset is the process by which a user who has forgotten their password can securely set a new one, typically by receiving a reset link via email.

**Technical Definition**

Laravel provides password reset services via the `Password` facade and `PasswordBroker` class. The flow involves: (1) requesting a reset link via `sendResetLink()`, which generates a token stored in the `password_reset_tokens` table (or cache) and sends an email; (2) the user clicks the link and submits a new password; (3) `reset()` validates the token and updates the password. Configuration is in `config/auth.php` under the `passwords` key.

**Beginner-Friendly Explanation**

Password reset lets users who forgot their password set a new one by clicking a link sent to their email.

#### Purposes

- To allow users to regain access to their accounts when they forget their password.
- To provide a secure, token-based password reset flow.
- To prevent unauthorized password changes.
- To invalidate old sessions after a password reset.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Password;

// Send reset link
$status = Password::sendResetLink(
    $request->only('email')
);

// Reset password
$status = Password::reset(
    $request->only('email', 'password', 'password_confirmation', 'token'),
    function ($user, $password) {
        $user->forceFill([
            'password' => Hash::make($password)
        ])->setRememberToken(Str::random(60));

        $user->save();

        event(new PasswordReset($user));
    }
);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Password::sendResetLink()` | Sends the reset link. |
| `Password::reset()` | Resets the password with a valid token. |
| `password_reset_tokens` table | Stores reset tokens. |
| `PasswordReset` event | Fired after a successful reset. |

**Syntax Rules**

- The `User` model must use the `Notifiable` trait and implement `CanResetPassword`.
- The reset token is validated before the password is changed.
- `forceFill()` is used to update the password without mass-assignment restrictions.
- The `remember_token` should be regenerated after a password reset.

**Constraints and Limitations**

- Password reset links expire (default: 60 minutes).
- The `password_reset_tokens` table must exist (included in default migrations).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Sending a Reset Link**

**Controller**

```php
use Illuminate\Support\Facades\Password;

public function sendResetLink(Request $request)
{
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::RESET_LINK_SENT
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
}
```

**Expected Output**

```
// If email exists: "We have emailed your password reset link!"
// If email not found: error message
```

**Why This Output Occurs**

`sendResetLink()` generates a token, stores it, and sends an email with the reset link.

---

**Example 2: Resetting the Password**

**Controller**

```php
use Illuminate\Support\Facades\Password;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;
use Illuminate\Auth\Events\PasswordReset;

public function reset(Request $request)
{
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function ($user, $password) {
            $user->forceFill([
                'password' => Hash::make($password)
            ])->setRememberToken(Str::random(60));

            $user->save();

            event(new PasswordReset($user));
        }
    );

    return $status === Password::PASSWORD_RESET
        ? redirect()->route('login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
}
```

**Expected Output**

```
// Valid token: password updated, redirect to login
// Invalid token: error message
```

**Why This Output Occurs**

`Password::reset()` validates the token, updates the password, and fires the `PasswordReset` event.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Account Recovery**

An e-commerce site allows customers to reset their passwords via email link, ensuring they can regain access to their accounts.

**Explanation**: Password reset is essential for user account recovery.

---

### 6. Email Verification

#### Definitions

**Core Definition**

Email verification is the process of confirming that a user's email address is valid and belongs to them, typically by sending a verification link that the user must click.

**Technical Definition**

Laravel's email verification feature is enabled by implementing the `MustVerifyEmail` interface on the `User` model. The `Registered` event triggers a verification email via the `SendEmailVerificationNotification` listener. The `verified` middleware (`EnsureEmailIsVerified`) protects routes that require verified emails. The verification URL is signed and includes the user's ID and email hash.

**Beginner-Friendly Explanation**

Email verification checks that the email address a user provided is real. Laravel sends them a link, and when they click it, their email is marked as verified.

#### Purposes

- To confirm that a user's email address is valid and accessible.
- To prevent fake or mistyped email addresses.
- To enable features that require a verified email (e.g., password reset).
- To improve security and reduce spam.

#### Syntax Rules and Structure

**Complete General Syntax**

**User Model:**

```php
use Illuminate\Contracts\Auth\MustVerifyEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

**Routes:**

```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');

Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();
    return redirect('/dashboard');
})->middleware(['auth', 'signed'])->name('verification.verify');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `MustVerifyEmail` | Interface to implement on the User model. |
| `verified` middleware | Protects routes requiring verified emails. |
| `EmailVerificationRequest` | Handles the verification link. |
| `sendEmailVerificationNotification()` | Sends the verification email. |

**Syntax Rules**

- The `User` model must implement `MustVerifyEmail`.
- The `Registered` event must be fired after registration.
- Verification routes require `auth` and `signed` middleware.
- The `verified` middleware can be applied to routes.

**Constraints and Limitations**

- The `MustVerifyEmail` interface requires the `email_verified_at` column.
- Verification links are signed and expire.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Enabling Email Verification**

**User Model**

```php
use Illuminate\Contracts\Auth\MustVerifyEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

**Route**

```php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified']);
```

**Expected Output**

```
// Users without verified email are redirected to /email/verify
// Users with verified email can access the dashboard
```

**Why This Output Occurs**

The `verified` middleware checks if the user's email is verified. If not, they are redirected to the verification notice page.

---

**Example 2: Sending Verification Email**

```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

**Expected Output**

```
// A verification email is sent to the user's email address
```

**Why This Output Occurs**

The `Registered` event triggers the `SendEmailVerificationNotification` listener, which sends the verification email.

---

#### Real-World Cases with Explanation

**Case 1: SaaS Platform**

A SaaS platform requires email verification before users can access the dashboard, ensuring that all users have valid email addresses.

**Explanation**: Email verification reduces fake accounts and ensures communication channels are valid.

---

### 7. Remember Me

#### Definitions

**Core Definition**

"Remember Me" is a feature that allows users to stay authenticated across browser sessions, even after the session expires or the browser is closed, by using a long-lived `remember_token` cookie.

**Technical Definition**

When `Auth::attempt($credentials, $remember = true)` is called, Laravel sets a `remember_token` cookie (default name: `remember_web_{hash}`) containing the user's ID, a random token, and a hash of the token. The `remember_token` column in the `users` table stores a hashed version of the token. On a subsequent request without an active session, Laravel checks the remember cookie, validates the token against the database, and re-authenticates the user. The `viaRemember()` method checks if the user was authenticated via the remember cookie.

**Beginner-Friendly Explanation**

"Remember Me" is a checkbox on the login form. If you check it, Laravel will keep you logged in even after you close your browser, until you explicitly log out.

#### Purposes

- To provide a convenient login experience by keeping users authenticated across sessions.
- To reduce the frequency of login prompts.
- To support long-lived authentication for personal devices.
- To balance security (short sessions) with convenience (persistent login).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Login with remember me
if (Auth::attempt($credentials, $remember)) {
    // ...
}

// Check if authenticated via remember me
if (Auth::viaRemember()) {
    // ...
}
```

**Component Breakdown**

| Component | Description |
|--------|-------------|
| `attempt($credentials, $remember)` | The `$remember` boolean enables the feature. |
| `remember_token` column | Stores the hashed remember token. |
| `viaRemember()` | Checks if authenticated via remember cookie. |
| `remember_web_{hash}` | The remember cookie name. |

**Syntax Rules**

- The `users` table must have a nullable `remember_token` column of at least 100 characters.
- The `remember` option is passed as the second argument to `attempt()` or `login()`.
- The remember cookie is automatically managed by Laravel.

**Constraints and Limitations**

- Laravel stores a single `remember_token` per user, shared across all devices. Logging out of one device invalidates the token for all devices.
- Remember tokens are long-lived and should be used cautiously on shared devices.

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
// User is logged in and a remember_token cookie is set
// After session expiry, the user is re-authenticated via the cookie
```

**Why This Output Occurs**

The second argument to `attempt()` enables the "remember me" feature, which sets a long-lived cookie.

---

**Example 2: Checking `viaRemember`**

```php
if (Auth::viaRemember()) {
    // User was authenticated via the remember cookie
}
```

**Expected Output**

```
// Returns true if the user was authenticated via the remember cookie
```

**Why This Output Occurs**

`viaRemember()` checks the authentication method used for the current request.

---

#### Real-World Cases with Explanation

**Case 1: Personal Devices**

Users on personal devices check "Remember Me" to avoid logging in every time they visit the site. On shared devices, they leave it unchecked.

**Explanation**: "Remember Me" balances convenience and security.

---

### 8. Session Configuration

#### Definitions

**Core Definition**

Session configuration refers to the settings in `config/session.php` that control how sessions are stored, how long they last, and how they are secured.

**Technical Definition**

Laravel's session configuration file (`config/session.php`) contains options including `driver` (storage mechanism), `lifetime` (minutes until idle session expires), `expire_on_close` (whether session expires when browser closes), `encrypt` (whether session data is encrypted), `files` (session file location), `connection` (database/Redis connection), `table` (session table name), `store` (cache store), `lottery` (garbage collection odds), `cookie` (cookie name), `path`, `domain`, `secure`, `http_only`, and `same_site`.

**Beginner-Friendly Explanation**

Session configuration is where you tell Laravel how to store sessions, how long they last, and how secure they should be.

#### Purposes

- To control where session data is stored (file, database, Redis, etc.).
- To set session lifetime and expiration behaviour.
- To configure session cookie security (secure, http_only, same_site).
- To enable or disable session encryption.
- To manage garbage collection of expired sessions.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/session.php
return [
    'driver' => env('SESSION_DRIVER', 'database'),
    'lifetime' => (int) env('SESSION_LIFETIME', 120),
    'expire_on_close' => env('SESSION_EXPIRE_ON_CLOSE', false),
    'encrypt' => env('SESSION_ENCRYPT', false),
    'files' => storage_path('framework/sessions'),
    'connection' => env('SESSION_CONNECTION'),
    'table' => 'sessions',
    'store' => env('SESSION_STORE'),
    'lottery' => [2, 100],
    'cookie' => env('SESSION_COOKIE', Str::slug(env('APP_NAME', 'laravel'), '_').'_session'),
    'path' => env('SESSION_PATH', '/'),
    'domain' => env('SESSION_DOMAIN'),
    'secure' => env('SESSION_SECURE_COOKIE'),
    'http_only' => env('SESSION_HTTP_ONLY', true),
    'same_site' => env('SESSION_SAME_SITE', 'lax'),
];
```

**Component Breakdown**

| Option | Description |
|--------|-------------|
| `driver` | Storage driver (`file`, `database`, `redis`, etc.). |
| `lifetime` | Minutes until idle session expires. |
| `expire_on_close` | Expire session when browser closes. |
| `encrypt` | Encrypt session data. |
| `secure` | Cookie only sent over HTTPS. |
| `http_only` | Cookie inaccessible to JavaScript. |
| `same_site` | CSRF protection (`lax`, `strict`, `none`). |

**Syntax Rules**

- Configuration is loaded from `config/session.php`.
- Environment variables can override defaults via `.env`.
- `lifetime` is in minutes.
- `expire_on_close` is a boolean.

**Constraints and Limitations**

- Changing `driver` requires the corresponding backend to be configured.
- `secure` should be `true` in production (requires HTTPS).
- `same_site` should be `lax` or `strict` for most applications.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Configuring Session Lifetime**

**`.env`**

```env
SESSION_LIFETIME=1440
SESSION_EXPIRE_ON_CLOSE=false
```

**Expected Output**

```
// Sessions expire after 24 hours of inactivity
```

**Why This Output Occurs**

`SESSION_LIFETIME=1440` sets the session lifetime to 1440 minutes (24 hours).

---

**Example 2: Secure Cookie Configuration**

**`.env`**

```env
SESSION_SECURE_COOKIE=true
SESSION_HTTP_ONLY=true
SESSION_SAME_SITE=lax
```

**Expected Output**

```
// Session cookie is only sent over HTTPS, inaccessible to JavaScript, and same-site lax
```

**Why This Output Occurs**

These settings enforce secure cookie behaviour for session authentication.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Session Configuration**

An e-commerce site sets `SESSION_LIFETIME=120` (2 hours) and `expire_on_close=false` to balance security and convenience.

**Explanation**: Session lifetime should match the application's security requirements.

---

### 9. Session Drivers

#### Definitions

**Core Definition**

Session drivers are the storage backends that Laravel uses to persist session data between requests.

**Technical Definition**

Laravel supports the following session drivers: `file` (stores sessions in `storage/framework/sessions`), `cookie` (stores sessions in encrypted cookies), `database` (stores sessions in a relational database table), `memcached` / `redis` (stores sessions in cache-based stores), `dynamodb` (stores sessions in AWS DynamoDB), and `array` (stores sessions in a PHP array, not persisted; used for testing). The driver is configured via the `SESSION_DRIVER` environment variable or the `driver` option in `config/session.php`.

**Beginner-Friendly Explanation**

A session driver is where Laravel stores your session data. You can store it in files, a database, or a fast cache like Redis.

#### Purposes

- To provide flexibility in how session data is stored.
- To enable performance optimization (Redis/Memcached for speed).
- To support distributed applications (database/Redis for shared sessions).
- To allow testing with the array driver.

#### Syntax Rules and Structure

**Complete General Syntax**

```env
SESSION_DRIVER=database
```

**Driver Comparison**

| Driver | Storage Location | Best For |
|--------|-----------------|----------|
| `file` | `storage/framework/sessions` | Development, single-server |
| `cookie` | Encrypted cookies | Stateless, low-volume |
| `database` | `sessions` table | Production, multi-server |
| `redis` | Redis server | High-performance, distributed |
| `memcached` | Memcached server | High-performance, distributed |
| `dynamodb` | AWS DynamoDB | AWS-based applications |
| `array` | PHP array (not persisted) | Testing |

**Syntax Rules**

- The driver is set in `.env` or `config/session.php`.
- `database` driver requires a `sessions` table.
- `redis` driver requires the `predis/predis` package or PhpRedis extension.
- `memcached` driver requires the Memcached PHP extension.

**Constraints and Limitations**

- `file` driver is not suitable for multi-server setups.
- `cookie` driver has a 4KB limit and should be used with encryption.
- `array` driver does not persist sessions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Database Session Driver**

**`.env`**

```env
SESSION_DRIVER=database
```

**Migration**

```php
Schema::create('sessions', function (Blueprint $table) {
    $table->string('id')->primary();
    $table->foreignId('user_id')->nullable()->index();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->longText('payload');
    $table->integer('last_activity')->index();
});
```

**Expected Output**

```
// Sessions are stored in the sessions table
```

**Why This Output Occurs**

The database driver stores session data in a relational database table.

---

**Example 2: Redis Session Driver**

**`.env`**

```env
SESSION_DRIVER=redis
SESSION_CONNECTION=default
```

**Expected Output**

```
// Sessions are stored in Redis for fast access
```

**Why This Output Occurs**

The Redis driver stores session data in a Redis server, providing high performance.

---

#### Real-World Cases with Explanation

**Case 1: High-Traffic Application**

A high-traffic application uses Redis for session storage to handle thousands of concurrent users efficiently.

**Explanation**: Redis provides fast, shared session storage suitable for distributed systems.

---

### 10. Session Security

#### Definitions

**Core Definition**

Session security encompasses the practices and configurations that protect session data from unauthorized access, tampering, and session-based attacks such as session fixation and session hijacking.

**Technical Definition**

Laravel provides several session security features: session ID regeneration on login (`$request->session()->regenerate()`), session invalidation on logout (`$request->session()->invalidate()`), CSRF token regeneration (`$request->session()->regenerateToken()`), secure cookie flags (`secure`, `http_only`, `same_site`), session encryption, and rate limiting on authentication endpoints. Additional protections include fingerprinting middleware and session driver selection (Redis/database preferred over file).

**Beginner-Friendly Explanation**

Session security is about keeping your session safe from attackers. Laravel has built-in features like changing your session ID when you log in and setting secure cookie flags to prevent common attacks.

#### Purposes

- To prevent session fixation attacks (regenerate session ID on login).
- To prevent session hijacking (secure cookies, fingerprinting).
- To protect session data from tampering (encryption).
- To prevent CSRF attacks (same_site cookies, CSRF tokens).
- To limit brute-force attacks (rate limiting).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Regenerate session ID on login
$request->session()->regenerate();

// Invalidate session on logout
$request->session()->invalidate();
$request->session()->regenerateToken();

// Regenerate CSRF token
$request->session()->regenerateToken();
```

**Security Configuration**

```env
SESSION_SECURE_COOKIE=true
SESSION_HTTP_ONLY=true
SESSION_SAME_SITE=lax
SESSION_ENCRYPT=true
SESSION_DRIVER=redis
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Session regeneration | New session ID on login. |
| Session invalidation | Clear session data on logout. |
| `secure` cookie | Only sent over HTTPS. |
| `http_only` cookie | Inaccessible to JavaScript. |
| `same_site` cookie | CSRF protection. |
| Session encryption | Encrypts session data. |
| Rate limiting | Prevents brute-force attacks. |

**Syntax Rules**

- Session ID should be regenerated after login.
- Session should be invalidated on logout.
- `secure` should be `true` in production.
- `http_only` should be `true` (default).
- `same_site` should be `lax` or `strict`.
- Rate limiting should be applied to authentication routes.

**Constraints and Limitations**

- Session regeneration should not be done on every request (only on privilege changes).
- `same_site=None` requires `secure=true` and should only be used for cross-site scenarios.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Session Regeneration on Login**

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

**Example 2: Secure Cookie Configuration**

**`.env`**

```env
SESSION_SECURE_COOKIE=true
SESSION_HTTP_ONLY=true
SESSION_SAME_SITE=strict
```

**Expected Output**

```
// Session cookie is only sent over HTTPS, inaccessible to JavaScript, and same-site strict
```

**Why This Output Occurs**

These settings enforce secure cookie behaviour for session authentication.

---

#### Real-World Cases with Explanation

**Case 1: Banking Application**

A banking application uses session regeneration on login, secure cookies, and rate limiting to protect user accounts.

**Explanation**: High-security applications require multiple layers of session protection.

---

### 11. Session Best Practices

#### Definitions

**Core Definition**

Session best practices are the recommended guidelines for implementing, configuring, and managing sessions to ensure security, performance, and maintainability.

**Technical Definition**

Best practices encompass driver selection, session lifetime configuration, cookie security flags, session ID regeneration, rate limiting, session encryption, and regular session cleanup. They aim to balance security, performance, and user experience.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep sessions secure and efficient.

#### Key Best Practices

1. **Use Redis or Database Driver in Production**: Avoid the `file` driver in production, especially in multi-server environments. Redis and database drivers provide shared, scalable session storage.

2. **Set Reasonable Session Lifetime**: Configure `SESSION_LIFETIME` based on application sensitivity. Shorter lifetimes (e.g., 120 minutes) for sensitive applications; longer for convenience.

3. **Enable Secure Cookie Flags**: Set `SESSION_SECURE_COOKIE=true` (HTTPS only), `SESSION_HTTP_ONLY=true` (JavaScript inaccessible), and `SESSION_SAME_SITE=lax` or `strict`.

4. **Regenerate Session ID on Login**: Always call `$request->session()->regenerate()` after successful login to prevent session fixation.

5. **Invalidate Session on Logout**: Call `$request->session()->invalidate()` and `$request->session()->regenerateToken()` on logout.

6. **Enable Session Encryption**: Set `SESSION_ENCRYPT=true` if session data contains sensitive information.

7. **Rate Limit Authentication Endpoints**: Apply `throttle:5,1` or a named limiter to login, registration, and password reset routes.

8. **Use `expire_on_close` for Shared Devices**: Set `SESSION_EXPIRE_ON_CLOSE=true` for applications accessed on shared devices.

9. **Avoid Storing Sensitive Data in Sessions**: Store only identifiers (user ID) and non-sensitive data. Never store passwords or credit card numbers.

10. **Use `logoutOtherDevices()` for Security-Critical Changes**: After password changes, call `Auth::logoutOtherDevices($password)` to invalidate sessions on other devices.

11. **Regularly Clear Expired Sessions**: The `lottery` configuration controls garbage collection. Ensure it is enabled for the chosen driver.

12. **Test Session Configuration**: Write tests that verify session behaviour (login, logout, expiration, security flags).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Secure Session Configuration**

**`.env`**

```env
SESSION_DRIVER=redis
SESSION_LIFETIME=120
SESSION_EXPIRE_ON_CLOSE=false
SESSION_SECURE_COOKIE=true
SESSION_HTTP_ONLY=true
SESSION_SAME_SITE=lax
SESSION_ENCRYPT=true
```

**Expected Output**

```
// Sessions stored in Redis, encrypted, with secure cookies and 2-hour lifetime
```

**Why This Output Occurs**

This configuration balances security (encryption, secure cookies, same-site) with performance (Redis) and convenience (2-hour lifetime).

---

**Example 2: Session Security in Login Controller**

```php
public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => ['required', 'email'],
        'password' => ['required'],
    ]);

    if (Auth::attempt($credentials, $request->boolean('remember'))) {
        $request->session()->regenerate();
        return redirect()->intended('dashboard');
    }

    return back()->withErrors([
        'email' => 'The provided credentials do not match our records.',
    ]);
}

public function logout(Request $request)
{
    Auth::logout();
    $request->session()->invalidate();
    $request->session()->regenerateToken();
    return redirect('/');
}
```

**Expected Output**

```
// Login: session regenerated; Logout: session invalidated, CSRF token regenerated
```

**Why This Output Occurs**

Session regeneration prevents fixation; invalidation and token regeneration ensure clean logout.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise SaaS Security**

An enterprise SaaS application uses Redis sessions, encryption, secure cookies, rate limiting, and session regeneration on login to meet security compliance requirements.

**Explanation**: Following best practices ensures sessions are secure and performant.

---

## References

- Authentication – Laravel 13.x Documentation — https://laravel.com/docs/13.x/authentication
- Authentication – Laravel 12.x Documentation — https://laravel.com/docs/12.x/authentication
- Authentication – Laravel 11.x Documentation — https://laravel.com/docs/11.x/authentication
- Authentication – Laravel 10.x Documentation — https://laravel.com/docs/10.x/authentication
- HTTP Session – Laravel 13.x Documentation — https://laravel.com/docs/13.x/session
- HTTP Session – Laravel 12.x Documentation — https://laravel.com/docs/12.x/session
- Resetting Passwords – Laravel 12.x Documentation — https://laravel.com/docs/12.x/passwords
- Email Verification – Laravel 12.x Documentation — https://laravel.com/docs/12.x/verification
- Securing Laravel Sessions with ID Regeneration – Laravel News — https://laravel-news.com/index.php/session-regeneration
- Laravel Session Configuration (config/session.php) – GitHub — https://github.com/laravel/laravel/blob/12.x/config/session.php
- OWASP Laravel Cheat Sheet – Session Security — https://cheatsheetseries.owasp.org/cheatsheets/Laravel_Cheat_Sheet.html
- Laravel Authentication Guards – CODE Magazine — https://www.codemag.com/article/2305081
- Configuring Trusted Hosts for Password Reset – Laravel 12.x Documentation — https://laravel.com/docs/12.x/passwords#configuring-trusted-hosts
- Laravel Session Regeneration – Stack Overflow — https://stackoverflow.com/questions/34429971/why-do-you-need-to-regenerate-laravel-session-after-login
- Laravel Session Security – Yisu — https://m.yisu.com/zixun/920773.html
- Laravel Session Best Practices – Laracasts Discussion — https://laracasts.com/discuss/channels/laravel/keep-the-same-laravel_session-between-api-requests

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Session authentication fundamentals (login, logout, registration, password reset, email verification, remember me) have remained stable across these versions. Key version-specific changes include: Laravel 11 replaced the `RouteServiceProvider` with the `bootstrap/app.php` `withRouting()` method; the `HasMiddleware` interface was introduced in Laravel 11; and session configuration defaults (e.g., default driver) may vary slightly between versions. Developers should always consult the documentation for their specific Laravel version.