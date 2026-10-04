# Laravel Sanctum Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Sanctum is a lightweight, first-party Laravel package that provides a unified authentication system for Single Page Applications (SPAs), mobile applications, and simple token-based APIs, offering both cookie-based session authentication and API token issuance without the complexity of OAuth2.

**Technical Definition:** Sanctum is implemented as a Laravel package consisting of the `Laravel\Sanctum` namespace, the `HasApiTokens` trait (added to the `User` model), a `personal_access_tokens` database table for storing hashed API tokens, the `EnsureFrontendRequestsAreStateful` middleware for SPA cookie authentication, and the `auth:sanctum` guard for route protection. It resolves authentication by first checking for a session cookie (for first-party SPAs), then falling back to the `Authorization: Bearer` header for API tokens.

**Beginner-Friendly Explanation:** Sanctum is Laravel's simple solution for answering the question "Who is making this request?" It handles two very different scenarios with one package. For a single-page application (like a Vue or React app), it uses cookies — the same way a traditional website remembers you're logged in. For mobile apps or third-party scripts, it issues API tokens — long random strings that act like digital keys. You don't need to choose one or the other; Sanctum supports both simultaneously.

### Key Characteristics

- **Dual-purpose design:** Sanctum simultaneously supports cookie-based SPA authentication and token-based API authentication, allowing you to use either or both in the same application.
- **Lightweight and first-party:** Sanctum is an official Laravel package, requiring only a single migration and a trait on the `User` model, with no OAuth2 complexity.
- **SHA-256 token hashing:** API tokens are hashed before storage, meaning plain-text tokens are never persisted in the database. The plain-text token is returned only once at creation time.
- **Ability-based scoping:** Tokens can be granted specific abilities (scopes), limiting what actions each token can perform — analogous to OAuth2 scopes but without the OAuth2 protocol overhead.
- **Session-first resolution:** When authenticating a request, Sanctum first checks for an authentication cookie, then falls back to the `Authorization` header for an API token.
- **Stateful domain restriction:** Cookie-based authentication is only attempted for requests originating from domains explicitly listed in the `SANCTUM_STATEFUL_DOMAINS` configuration, preventing third-party domains from receiving session cookies.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+) or PHP 8.2+ (Laravel 11+).
- Composer dependency manager.
- A Laravel application with the `install:api` Artisan command available (Laravel 11+) or the `RouteServiceProvider` configured (Laravel 10-).
- A database connection configured in `.env`.
- For SPA authentication: a frontend application (Vue, React, Next.js) running on a stateful domain.
- For token authentication: no additional prerequisites beyond the base Sanctum installation.

### Related Programming Areas

- **Laravel Authentication** — Sanctum integrates with Laravel's guard system and the `auth` middleware.
- **Middleware** — `EnsureFrontendRequestsAreStateful` and `auth:sanctum` are the primary middleware for Sanctum.
- **Eloquent** — The `HasApiTokens` trait adds token-related methods to the `User` model.
- **CORS** — `config/cors.php` must be configured with `supports_credentials => true` for SPA authentication.
- **Rate Limiting** — Sanctum tokens integrate with Laravel's rate limiting middleware.
- **Laravel Passport** — The alternative for applications requiring full OAuth2 server capabilities.

### Core Concepts / Features

1. **SPA Authentication:** Cookie-based session authentication, stateful domains, and `EnsureFrontendRequestsAreStateful` middleware.
2. **API Token Authentication:** Mobile/third-party apps, personal access tokens, and the database schema for tokens.
3. **Token Abilities:** Defining scopes, checking abilities via `tokenCan()`, and enforcing structural restrictions.
4. **Authentication Middleware:** Protecting routes using the `auth:sanctum` guard.
5. **CORS & Stateful Domains:** Configuring `config/cors.php` and `.env` (`SANCTUM_STATEFUL_DOMAINS`) to prevent cross-origin blocks.
6. **Token Expiration & Pruning:** Managing token lifespans in `config/sanctum.php` and automating the cleanup of expired tokens.

---

## 1. SPA Authentication

### Definitions

**Core Definition:** Sanctum SPA authentication is a cookie-based, stateful session authentication mechanism for first-party single-page applications, using Laravel's built-in session guard instead of API tokens.

**Technical Definition:** Sanctum's SPA authentication uses Laravel's `web` authentication guard and session cookies to authenticate requests from first-party SPAs. The `EnsureFrontendRequestsAreStateful` middleware checks the request's `Origin` or `Referer` header against the domains listed in `SANCTUM_STATEFUL_DOMAINS`. If the request originates from a stateful domain, the middleware applies session middleware (`StartSession`, `VerifyCsrfToken`, `EncryptCookies`) to the API request before it reaches the route. The SPA must first request a CSRF cookie from `/sanctum/csrf-cookie`, then include the `X-XSRF-TOKEN` header (automatically handled by Axios) in subsequent requests. Sanctum does **not** use tokens for this flow — it relies entirely on session cookies.

**Beginner-Friendly Explanation:** If your SPA (a Vue or React app) lives on the same domain as your Laravel backend, you can use cookies for authentication just like a traditional website. Sanctum handles the tricky parts: it tells Laravel which domains should receive cookies, and it ensures that CSRF protection works for your SPA. This is more secure than storing tokens in `localStorage` because cookies can be `HttpOnly`, protecting against XSS attacks.

### Purposes

- To provide secure, cookie-based authentication for first-party SPAs without exposing authentication credentials to JavaScript.
- To leverage Laravel's built-in session management and CSRF protection for SPA authentication.
- To avoid the security risks of storing authentication tokens in `localStorage` or `sessionStorage`.
- To enable session-based authentication for SPAs hosted on the same top-level domain as the API.
- To integrate seamlessly with popular SPA frameworks (Vue, React, Next.js) using Axios or fetch.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Configure stateful domains in config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s%s',
    'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
    Sanctum::currentApplicationUrlWithPort(),
    env('FRONTEND_URL') ? ','.parse_url(env('FRONTEND_URL'), PHP_URL_HOST) : ''
))),
```

```env
# .env — Add your SPA domains
SANCTUM_STATEFUL_DOMAINS=localhost:3000,myapp.example.com
SESSION_DOMAIN=.example.com
```

```php
// Step 2: Add Sanctum's middleware to the api group (Laravel 10-)
// In app/Http/Kernel.php:
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],

// Laravel 11+ — bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->statefulApi();
})
```

```javascript
// Step 3: SPA frontend — Login flow (Vue/React/Next.js)
// First, request the CSRF cookie
await axios.get('http://api.example.com/sanctum/csrf-cookie');

// Then, log in
await axios.post('http://api.example.com/login', {
    email: 'user@example.com',
    password: 'password',
});

// Now all subsequent requests are authenticated via session cookie
const user = await axios.get('http://api.example.com/api/user');
```

**Component Breakdown:**

- `SANCTUM_STATEFUL_DOMAINS` — Comma-separated list of domains that should receive stateful (session) authentication cookies. These are the first-party origins from which your SPA makes requests.
- `EnsureFrontendRequestsAreStateful` — Middleware that checks the request's `Origin` or `Referer` header. If it matches a stateful domain, it applies session and CSRF middleware to the request.
- `/sanctum/csrf-cookie` — Route provided by Sanctum that sets the `XSRF-TOKEN` cookie, which Axios automatically reads and sends as the `X-XSRF-TOKEN` header.
- `$middleware->statefulApi()` — Laravel 11+ method that adds Sanctum's stateful middleware to the `api` middleware group.
- `SESSION_DOMAIN` — Set to `.example.com` to share cookies across subdomains (e.g., `app.example.com` and `api.example.com`).

**Syntax Rules:**

- The SPA domain **must** be listed in `SANCTUM_STATEFUL_DOMAINS` for cookie authentication to work.
- The SPA must send requests with credentials (`withCredentials: true` in Axios, or `credentials: 'include'` in fetch).
- The first request to the API must be to `/sanctum/csrf-cookie` to obtain the CSRF token.
- The `auth:sanctum` middleware works for both token and SPA authentication; it checks cookies first, then falls back to tokens.
- The login route should be in `routes/web.php` (not `api.php`) so it has session middleware.

**Constraints and Limitations:**

- **Cross-domain SPA authentication requires careful CORS and cookie configuration.** The `SESSION_DOMAIN` must be set to a common parent domain, and CORS must allow credentials.
- **Sanctum SPA authentication does not work for mobile apps or third-party clients.** Those should use token authentication.
- **The `EnsureFrontendRequestsAreStateful` middleware adds session middleware to API routes**, which increases overhead slightly. This is acceptable for first-party SPAs but not for high-throughput third-party APIs.
- **CSRF protection is required for stateful requests.** The SPA must include the `X-XSRF-TOKEN` header, which Axios does automatically when `withCredentials` is enabled.

### Annotated Code Examples

**Example 1: Sanctum SPA Authentication Setup**

```env
# .env
SANCTUM_STATEFUL_DOMAINS=localhost:3000,app.example.com
SESSION_DOMAIN=.example.com
```

```php
<?php
// File: config/sanctum.php (published via php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider")

return [
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
        '%s%s%s',
        'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
        Sanctum::currentApplicationUrlWithPort(),
        env('FRONTEND_URL') ? ','.parse_url(env('FRONTEND_URL'), PHP_URL_HOST) : ''
    ))),

    'guard' => ['web'], // The guard used for SPA authentication
    'expiration' => null, // Session lifetime (null = default)
];
```

```php
// File: routes/web.php — Login route for session authentication
Route::post('/login', [AuthController::class, 'login']);

// File: routes/api.php — Protected API routes
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

```javascript
// File: SPA frontend (Vue/React) — Axios configuration
import axios from 'axios';

// Configure Axios to send credentials (cookies) with every request
axios.defaults.withCredentials = true;
axios.defaults.baseURL = 'http://api.example.com';

// Login flow
async function login(email, password) {
    // Step 1: Get the CSRF cookie
    await axios.get('/sanctum/csrf-cookie');

    // Step 2: Log in (session cookie is set automatically)
    await axios.post('/login', { email, password });

    // Step 3: Fetch authenticated user
    const response = await axios.get('/api/user');
    return response.data;
}
```

**Step-by-Step Setup:**

1. Add `SANCTUM_STATEFUL_DOMAINS` and `SESSION_DOMAIN` to `.env`.
2. Ensure `EnsureFrontendRequestsAreStateful` middleware is in the `api` middleware group (Laravel 10-) or call `$middleware->statefulApi()` in `bootstrap/app.php` (Laravel 11+).
3. Configure Axios in the SPA with `withCredentials: true`.
4. Implement the login flow: call `/sanctum/csrf-cookie`, then `POST /login`, then make authenticated requests.
5. Ensure the login route is in `routes/web.php` (not `api.php`) so it has session middleware.

**Expected Output:**

- After `GET /sanctum/csrf-cookie`, the browser receives a `XSRF-TOKEN` cookie.
- `POST /login` with valid credentials sets a `laravel_session` cookie and returns a `200 OK`.
- `GET /api/user` with the session cookie returns the authenticated user's JSON.
- The SPA does not need to handle tokens manually — cookies are managed automatically by the browser.

**Why This Output Occurs:** The `EnsureFrontendRequestsAreStateful` middleware detects that the request originates from a stateful domain (`localhost:3000` or `app.example.com`) by checking the `Origin` or `Referer` header. It then applies `StartSession` and `VerifyCsrfToken` middleware to the request. The `/sanctum/csrf-cookie` route sets the `XSRF-TOKEN` cookie, and Axios automatically reads this cookie and sends it as the `X-XSRF-TOKEN` header in subsequent requests. The session cookie (`laravel_session`) is set after login and sent with every subsequent request, authenticating the user.

---

**Example 2: Next.js SPA with Sanctum SPA Authentication**

```javascript
// File: lib/axios.js — Next.js API client
import axios from 'axios';

const api = axios.create({
    baseURL: process.env.NEXT_PUBLIC_API_URL, // e.g., http://api.example.com
    withCredentials: true,
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
    },
});

export default api;
```

```javascript
// File: pages/login.js — Login page
import { useState } from 'react';
import api from '../lib/axios';

export default function Login() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();

        // Step 1: Get CSRF cookie
        await api.get('/sanctum/csrf-cookie');

        // Step 2: Log in
        await api.post('/login', { email, password });

        // Step 3: Redirect to dashboard
        window.location.href = '/dashboard';
    };

    return (
        <form onSubmit={handleSubmit}>
            <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" />
            <button type="submit">Log In</button>
        </form>
    );
}
```

**Expected Output:** After login, the Next.js app can make authenticated requests to the Laravel API using cookies. The user's session persists across page reloads because the session cookie is stored by the browser.

**Why This Output Occurs:** The Next.js app is running on `localhost:3000` and the Laravel API on `localhost:8000`. The `SANCTUM_STATEFUL_DOMAINS` includes `localhost:3000`, so Sanctum treats requests from the Next.js app as stateful. The `withCredentials: true` setting ensures that cookies are sent with every request. The CSRF cookie is set by `/sanctum/csrf-cookie`, and the session cookie is set after login.

### Real-World Cases

- **First-party SPAs (Vue, React, Next.js):** Applications where the frontend and backend are controlled by the same organisation and share a top-level domain.
- **SaaS dashboards:** Internal dashboards where security is paramount, and cookie-based authentication avoids the XSS risks of token storage.
- **Admin panels:** Laravel Nova or custom admin panels that use Sanctum SPA authentication for seamless session management.
- **Hybrid applications:** An application that uses SPA authentication for browser-based clients and token authentication for mobile clients, both through the same Sanctum setup.

---

## 2. API Token Authentication

### Definitions

**Core Definition:** Sanctum API token authentication is a stateless mechanism for issuing API tokens (personal access tokens) to users, allowing mobile applications and third-party clients to authenticate requests without relying on session cookies.

**Technical Definition:** Sanctum's API token authentication stores user API tokens in the `personal_access_tokens` database table. Tokens are hashed using SHA-256 before storage; the plain-text token is returned only once at creation time via the `createToken()` method on the `HasApiTokens` trait. Incoming requests are authenticated by the `auth:sanctum` middleware, which checks the `Authorization: Bearer` header for a valid token and resolves the associated user. The `personal_access_tokens` table uses a polymorphic relationship (`tokenable_type`, `tokenable_id`) so that any model — not just `User` — can own tokens.

**Beginner-Friendly Explanation:** Token authentication is like getting a wristband at an event. You show your ID once (log in with email and password), and you get a wristband (a token). For the rest of the event, you just show your wristband to prove who you are — you don't need to show your ID again. Sanctum is Laravel's way of creating and managing these wristbands. It is simple, fast, and perfect for mobile apps and APIs that don't need the complexity of OAuth2.

### Purposes

- To authenticate API requests statelessly, without relying on server-side sessions or cookies.
- To issue simple, long-lived tokens for mobile applications and third-party integrations.
- To provide a lightweight alternative to OAuth2 when full OAuth2 flows are not required.
- To enable per-device or per-application token management (a user can have multiple tokens for different devices).
- To integrate seamlessly with Laravel's `auth:sanctum` middleware for route protection.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Step 1: Install Sanctum (included by default in Laravel 11+)
php artisan install:api

# Step 2: Run migrations to create the personal_access_tokens table
php artisan migrate
```

```php
// Step 3: Add the HasApiTokens trait to the User model
namespace App\Models;

use Laravel\Sanctum\HasApiTokens;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasApiTokens;
}
```

```php
// Step 4: Issue a token upon login
$user = User::where('email', $request->email)->first();

if (!$user || !Hash::check($request->password, $user->password)) {
    throw ValidationException::withMessages([
        'email' => ['The provided credentials are incorrect.'],
    ]);
}

$token = $user->createToken('auth-token')->plainTextToken;

return response()->json(['token' => $token]);
```

```php
// Step 5: Protect routes with the auth:sanctum middleware
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Component Breakdown:**

- `php artisan install:api` — Installs Sanctum, creates the `routes/api.php` file, and publishes the migration for the `personal_access_tokens` table.
- `HasApiTokens` trait — Added to the `User` model, providing the `createToken()` and `tokens()` relationship methods.
- `createToken('auth-token')` — Creates a new personal access token with the given name (device/application identifier). Returns a `NewAccessToken` object.
- `->plainTextToken` — The plain-text token value that should be returned to the client. This is the only time the token is visible in plain text.
- `auth:sanctum` middleware — Applied to routes to require a valid Sanctum token for access.

**Database Schema for Tokens:**

The `personal_access_tokens` table has the following structure:

| Column | Type | Purpose |
|---|---|---|
| `id` | bigint | Primary key |
| `tokenable_type` | string | Polymorphic type (e.g., `App\Models\User`) |
| `tokenable_id` | bigint | Foreign key to owning model |
| `name` | string | Human-readable token name |
| `token` | string(64) | SHA256 hash of plaintext token |
| `abilities` | JSON | Array of granted abilities |
| `expires_at` | timestamp | Optional expiration date |
| `last_used_at` | timestamp | Last authentication timestamp |
| `created_at` | timestamp | Token creation date |

**Syntax Rules:**

- The `createToken()` method **must** be called on a model that uses the `HasApiTokens` trait.
- The `auth:sanctum` middleware is applied to routes in `routes/api.php` or route groups.
- The token is sent by the client in the `Authorization: Bearer <token>` header.
- Multiple tokens can be created for the same user, each with its own name and abilities.
- Tokens are hashed with SHA-256 before storage; the plain-text token is never persisted.

**Constraints and Limitations:**

- **Sanctum does not support OAuth2 flows.** If you need authorization code grants, client credentials, or refresh tokens, use Passport.
- **Tokens are stored hashed in the database.** The plain-text token is only available immediately after creation; if lost, it cannot be retrieved.
- **Token expiration is not enabled by default.** Tokens remain valid indefinitely unless configured otherwise.
- **Sanctum tokens are not JWTs.** They are opaque strings stored in the database, not self-contained tokens.

### Annotated Code Examples

**Example 1: Complete Token Authentication Flow**

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
    // POST /api/login — Issue a new token
    public function login(Request $request): JsonResponse
    {
        $request->validate([
            'email'       => 'required|email',
            'password'    => 'required',
            'device_name' => 'required|string|max:255',
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        // Create a token named after the device (e.g., "iPhone 15", "Postman")
        $token = $user->createToken($request->device_name)->plainTextToken;

        return response()->json([
            'user'  => $user,
            'token' => $token,
        ]);
    }

    // POST /api/logout — Revoke the current token
    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();
        return response()->json(['message' => 'Logged out successfully.']);
    }

    // GET /api/user — Return the authenticated user
    public function user(Request $request): JsonResponse
    {
        return response()->json($request->user());
    }
}
```

```php
// File: routes/api.php

use App\Http\Controllers\Api\AuthController;
use Illuminate\Support\Facades\Route;

// Public route — issues tokens
Route::post('/login', [AuthController::class, 'login']);

// Protected routes — require a valid token
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', [AuthController::class, 'user']);
    Route::post('/logout', [AuthController::class, 'logout']);
});
```

**Step-by-Step Setup:**

1. Run `php artisan install:api` and `php artisan migrate`.
2. Add the `HasApiTokens` trait to the `User` model.
3. Create the `AuthController` with `login`, `logout`, and `user` methods.
4. Define the routes in `routes/api.php`.
5. Test with `curl` or Postman: `POST /api/login` with `{"email":"user@example.com","password":"secret","device_name":"test"}`.

**Expected Output:**

```json
{
    "user": { "id": 1, "name": "Alice", "email": "user@example.com" },
    "token": "1|abc123def456..."
}
```

```bash
# Subsequent request with the token:
curl -H "Authorization: Bearer 1|abc123def456..." https://example.com/api/user
```

```json
{ "id": 1, "name": "Alice", "email": "user@example.com" }
```

**Why This Output Occurs:** The `login` method verifies credentials, then calls `createToken()` on the `HasApiTokens` trait. This creates a new row in the `personal_access_tokens` table with the token hashed, and returns the plain-text token once. The client includes this token in the `Authorization` header for subsequent requests. The `auth:sanctum` middleware resolves the token, finds the associated user, and makes it available via `$request->user()`.

---

**Example 2: Token-Based Authentication for a Mobile API**

```javascript
// Mobile app login flow (pseudo-code for a React Native / Flutter client)
async function login(email, password) {
    const response = await fetch('https://api.example.com/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
        body: JSON.stringify({ email, password, device_name: 'Pixel 8' }),
    });

    const data = await response.json();

    // Store the token securely (e.g., in Keychain or SecureStore)
    await SecureStore.setItemAsync('auth_token', data.token);
    return data;
}

async function getUser() {
    const token = await SecureStore.getItemAsync('auth_token');
    const response = await fetch('https://api.example.com/api/user', {
        headers: { 'Authorization': `Bearer ${token}`, 'Accept': 'application/json' },
    });
    return response.json();
}
```

**Expected Output:** The mobile app stores the token and includes it in all subsequent API requests. The server authenticates each request independently, with no session state stored server-side.

**Why This Output Occurs:** The mobile app is a "stateless" client — it does not rely on cookies or server-side sessions. By storing the token securely on the device and including it in the `Authorization` header, the app can authenticate any request without the server needing to remember anything between requests. This is the core advantage of token authentication for mobile apps.

### Real-World Cases

- **Mobile application backends:** A Laravel API serves JSON to iOS and Android apps, with each device receiving its own token identified by device name.
- **First-party JavaScript applications:** A Vue or React SPA hosted on a different domain uses Sanctum tokens instead of cookies for authentication.
- **CLI tools and scripts:** Developers can generate long-lived personal access tokens to authenticate command-line tools and automation scripts against your API.
- **Third-party integrations (simple):** When full OAuth2 is not required, Sanctum tokens provide a simple way for external services to authenticate.

---

## 3. Token Abilities

### Definitions

**Core Definition:** Token abilities (also called scopes) are string labels assigned to a Sanctum API token that restrict what actions the token is permitted to perform, enabling fine-grained authorization at the token level.

**Technical Definition:** Sanctum token abilities are stored as a JSON array in the `abilities` column of the `personal_access_tokens` table. When a token is created via `createToken($name, $abilities)`, the abilities array is persisted alongside the token. During request handling, the `tokenCan($ability)` method checks whether the current token's abilities array contains the specified ability or the wildcard `*`. The `abilities` and `ability` middleware can be applied to routes to enforce ability checks structurally.

**Beginner-Friendly Explanation:** A token ability is like a keycard that only opens certain doors. Instead of giving someone a master key to your entire API, you can give them a key that only opens the "read posts" door. If they try to open the "delete users" door, it won't work. This is useful when a user creates a token for a specific purpose — like a CI/CD pipeline that should only read deployment status, not delete repositories.

### Purposes

- To restrict what actions an API token can perform, following the principle of least privilege.
- To allow users to create purpose-specific tokens (e.g., a "read-only" token for dashboards, a "deploy" token for CI/CD).
- To provide OAuth2-like scopes without the complexity of the OAuth2 protocol.
- To enable structural enforcement of token permissions via middleware, preventing unauthorized actions at the routing layer.
- To complement (not replace) Laravel's authorization system (Gates and Policies) with token-level restrictions.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Creating a token with specific abilities
$token = $user->createToken('token-name', ['post:read', 'post:create'])->plainTextToken;

// Creating a token with all abilities (wildcard)
$token = $user->createToken('token-name', ['*'])->plainTextToken;

// Creating a token with no abilities (empty array)
$token = $user->createToken('token-name', [])->plainTextToken;

// Checking abilities in a controller
if ($request->user()->tokenCan('post:create')) {
    // Allow the action
}

// Checking abilities via middleware (requires ALL listed abilities)
Route::middleware(['auth:sanctum', 'abilities:post:create,post:update'])->post('/posts', ...);

// Checking abilities via middleware (requires ANY of the listed abilities)
Route::middleware(['auth:sanctum', 'ability:post:create,post:update'])->post('/posts', ...);
```

**Component Breakdown:**

- `createToken($name, $abilities)` — The second argument is an array of ability strings. If omitted, the default is `['*']` (all abilities).
- `tokenCan('ability')` — Returns `true` if the current token has the specified ability or the wildcard `*`. Always returns `true` for first-party SPA requests (session authentication).
- `abilities:ability1,ability2` middleware — Requires the token to have **all** listed abilities.
- `ability:ability1,ability2` middleware — Requires the token to have **at least one** of the listed abilities.
- `abilities` column — Stores the abilities as a JSON array in the `personal_access_tokens` table.

**Syntax Rules:**

- The default abilities array is `['*']`, which grants all abilities.
- Token abilities are checked against the token, not the user's role. A user with admin role but a token without the `post:create` ability cannot create posts.
- The `abilities` middleware (plural) requires **all** listed abilities; the `ability` middleware (singular) requires **any** of the listed abilities.
- The `tokenCan()` method is available on the `User` model via the `HasApiTokens` trait.
- Abilities are stored as a JSON array in the database, allowing flexible permission structures without additional tables.

**Constraints and Limitations:**

- **Token abilities are not a replacement for authorization policies.** Use them in conjunction with Gates and Policies for comprehensive access control. As one Laracasts discussion notes: "Sanctum: Handles authentication (who the user is), and can optionally restrict tokens to certain abilities/scopes. Spatie Permission: Handles authorization (what the user can do)."
- **`tokenCan()` always returns `true` for first-party SPA requests.** Because SPA authentication uses session cookies, there is no token to check abilities against.
- **Token abilities are stored as a JSON array** in the `abilities` column. Changing abilities requires updating the token record.
- **Abilities are not automatically synced with user permissions.** If a user's role changes, existing tokens retain their original abilities.

### Annotated Code Examples

**Example 1: Creating and Enforcing Token Abilities**

```php
<?php
// File: app/Http/Controllers/Api/TokenController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class TokenController extends Controller
{
    // POST /api/tokens — Create a new token with specific abilities
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'        => 'required|string|max:255',
            'abilities'   => 'sometimes|array',
            'abilities.*' => 'string|in:post:read,post:create,post:update,post:delete,user:read',
        ]);

        $token = $request->user()->createToken(
            $validated['name'],
            $validated['abilities'] ?? ['*']
        );

        return response()->json([
            'token'     => $token->plainTextToken,
            'name'      => $validated['name'],
            'abilities' => $validated['abilities'] ?? ['*'],
        ], 201);
    }
}
```

```php
// File: routes/api.php — Protect routes with ability middleware

use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum'])->group(function () {
    // This route requires the token to have BOTH 'post:read' AND 'post:create'
    Route::post('/posts', [PostController::class, 'store'])
        ->middleware('abilities:post:read,post:create');

    // This route requires the token to have EITHER 'post:update' OR 'post:delete'
    Route::patch('/posts/{post}', [PostController::class, 'update'])
        ->middleware('ability:post:update,post:delete');

    // This route checks abilities manually in the controller
    Route::get('/admin/data', function (Request $request) {
        if (!$request->user()->tokenCan('admin:read')) {
            return response()->json(['error' => 'Insufficient token abilities.'], 403);
        }
        return response()->json(['data' => 'sensitive admin data']);
    });
});
```

**Step-by-Step Setup:**

1. Create a token with specific abilities: `POST /api/tokens` with `{"name": "CI Pipeline", "abilities": ["post:read"]}`.
2. Use the token to access a protected route: `GET /api/posts` with `Authorization: Bearer <token>`.
3. Attempt to access a route requiring `post:create` with the same token.

**Expected Output (Token Creation):**

```json
{
    "token": "3|xyz789abc123...",
    "name": "CI Pipeline",
    "abilities": ["post:read"]
}
```

**Expected Output (Using the Token):**

- `GET /api/posts` with the token → `200 OK` (token has `post:read`).
- `POST /api/posts` with the same token → `403 Forbidden` (token lacks `post:create`).

**Why This Output Occurs:** The `createToken()` method stores the token with the specified abilities in the `abilities` JSON column. When the token is used, the `auth:sanctum` middleware resolves it, and the `abilities` middleware checks whether the token has the required ability. If not, a `403 Forbidden` response is returned. The `tokenCan()` method performs the same check manually within a controller.

---

**Example 2: Combining Token Abilities with Laravel Policies**

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
    public function store(Request $request): JsonResponse
    {
        // Step 1: Check token ability (structural restriction)
        if (!$request->user()->tokenCan('post:create')) {
            return response()->json([
                'error' => 'This token does not have the post:create ability.',
            ], 403);
        }

        // Step 2: Check user authorization (policy-based restriction)
        $this->authorize('create', Post::class);

        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body'  => 'required|string',
        ]);

        $post = $request->user()->posts()->create($validated);

        return response()->json($post, 201);
    }
}
```

**Expected Output:**

- If the token lacks `post:create` → `403 Forbidden` with `{"error": "This token does not have the post:create ability."}`.
- If the token has `post:create` but the user lacks the `create` policy permission → `403 Forbidden` (Laravel's default authorization response).
- If both checks pass → `201 Created` with the new post.

**Why This Output Occurs:** Sanctum token abilities and Laravel Policies serve different purposes. Token abilities restrict what the *token* can do, while Policies restrict what the *user* can do. Both checks are applied in sequence: the token ability check is a structural restriction (the token was created without the ability), while the policy check is a user-level authorization decision. Using both together provides defence in depth.

### Real-World Cases

- **CI/CD pipelines:** A developer creates a token with `deploy:write` ability for deployment automation, and a separate token with `read:status` for monitoring dashboards.
- **Mobile apps with limited functionality:** A mobile app creates a token with only the abilities needed for its features, reducing the impact if the token is compromised.
- **Third-party integrations with scoped access:** An external service is given a token with only `read:orders` ability, preventing it from modifying or deleting orders.
- **Multi-tenant SaaS:** Each tenant's token is scoped to their own data, with abilities like `tenant:123:read` and `tenant:123:write`.

---

## 4. Authentication Middleware

### Definitions

**Core Definition:** The `auth:sanctum` middleware is Laravel's authentication guard for Sanctum, protecting routes by requiring a valid Sanctum token or session cookie before allowing access.

**Technical Definition:** The `auth:sanctum` middleware resolves the `sanctum` guard from Laravel's authentication manager. The guard is a `RequestGuard` that delegates to Sanctum's `Guard` class, which checks for a valid session cookie (for first-party SPA requests) or a valid API token in the `Authorization: Bearer` header (for token-based requests). If authentication fails, the middleware throws an `AuthenticationException`, which Laravel converts to a `401 Unauthorized` JSON response for API requests.

**Beginner-Friendly Explanation:** The `auth:sanctum` middleware is like a bouncer at the door of a VIP lounge. It checks your ID (your token or session cookie) before letting you in. If you don't have a valid ID, you're turned away with a "401 Unauthorized" response. If you do, you can access the route. It's that simple — just add `->middleware('auth:sanctum')` to any route that should require authentication.

### Purposes

- To protect API routes by requiring authentication before the route handler executes.
- To automatically resolve the authenticated user from either a session cookie or an API token.
- To provide a single, unified authentication guard for both SPA and token-based clients.
- To reject unauthenticated requests with a `401 Unauthorized` response before they reach application logic.
- To integrate with Laravel's authorization system (Gates and Policies) by making the authenticated user available via `$request->user()`.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Applying auth:sanctum to a single route
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');

// Applying auth:sanctum to a group of routes
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::put('/profile', [ProfileController::class, 'update']);
    Route::apiResource('posts', PostController::class);
});

// Combining with other middleware
Route::middleware(['auth:sanctum', 'throttle:api', 'verified'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});
```

**Component Breakdown:**

- `->middleware('auth:sanctum')` — Applies the Sanctum guard to the route or group. The guard checks for a session cookie first, then an API token.
- `auth:sanctum` — The guard name is `sanctum` (prefixed with `auth:` to indicate the authentication middleware).
- `$request->user()` — Returns the authenticated user instance, or `null` if not authenticated.
- Group middleware — Applying `auth:sanctum` to a group protects all routes within it, reducing repetition.

**Syntax Rules:**

- The `auth:sanctum` middleware is applied to routes in `routes/api.php` or route groups.
- The `sanctum` guard is automatically registered by Sanctum's service provider. No manual configuration is required unless you need to customise the guard.
- In Laravel 10 and earlier, the `api` middleware group includes `EnsureFrontendRequestsAreStateful` by default. In Laravel 11+, run `php artisan install:api` or add `$middleware->statefulApi()` to `bootstrap/app.php`.
- The `auth:sanctum` middleware returns a `401 Unauthorized` response for unauthenticated API requests.

**Constraints and Limitations:**

- **`auth:sanctum` checks cookies first, then tokens.** If a request has both a valid session cookie and an invalid token, the session cookie takes precedence.
- **For SPA authentication, the request must originate from a stateful domain.** If the `Origin` or `Referer` header does not match `SANCTUM_STATEFUL_DOMAINS`, the session cookie is ignored.
- **`tokenCan()` always returns `true` for session-authenticated requests.** SPA requests do not have a token to check abilities against.
- **The `auth:sanctum` middleware does not automatically apply rate limiting.** Combine it with `throttle:api` for rate-limited routes.

### Annotated Code Examples

**Example 1: Protecting Routes with `auth:sanctum`**

```php
<?php
// File: routes/api.php

use App\Http\Controllers\Api\PostController;
use App\Http\Controllers\Api\ProfileController;
use App\Http\Controllers\Api\AuthController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

// Public routes — no authentication required
Route::post('/login', [AuthController::class, 'login']);
Route::post('/register', [AuthController::class, 'register']);

// Protected routes — require a valid Sanctum token or session cookie
Route::middleware('auth:sanctum')->group(function () {
    // User profile
    Route::get('/user', function (Request $request) {
        return $request->user();
    });
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::put('/profile', [ProfileController::class, 'update']);

    // Post management
    Route::apiResource('posts', PostController::class);

    // Logout — revokes the current token or invalidates the session
    Route::post('/logout', [AuthController::class, 'logout']);
});
```

**Step-by-Step Setup:**

1. Define the routes as shown in `routes/api.php`.
2. Run `php artisan install:api` (Laravel 11+) and `php artisan migrate`.
3. Add the `HasApiTokens` trait to the `User` model.
4. Test with a token: `curl -H "Authorization: Bearer <token>" https://example.com/api/user`.

**Expected Output:**

- `GET /api/user` without authentication → `401 Unauthorized` with `{"message": "Unauthenticated."}`.
- `GET /api/user` with a valid token → `200 OK` with the user JSON.
- `GET /api/user` with a valid session cookie from a stateful domain → `200 OK` with the user JSON.

**Why This Output Occurs:** The `auth:sanctum` middleware intercepts the request before it reaches the route handler. It checks for a valid session cookie (if the request originates from a stateful domain) or a valid `Authorization: Bearer` token. If neither is present or valid, the middleware throws an `AuthenticationException`, which Laravel renders as a `401 Unauthorized` JSON response for API requests.

---

**Example 2: Combining `auth:sanctum` with Token Abilities**

```php
<?php
// File: routes/api.php

use App\Http\Controllers\Api\PostController;
use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum'])->group(function () {
    // Any authenticated user can read posts
    Route::get('/posts', [PostController::class, 'index']);
    Route::get('/posts/{post}', [PostController::class, 'show']);

    // Only tokens with 'post:create' ability can create posts
    Route::post('/posts', [PostController::class, 'store'])
        ->middleware('abilities:post:create');

    // Only tokens with 'post:update' OR 'post:delete' can modify posts
    Route::put('/posts/{post}', [PostController::class, 'update'])
        ->middleware('ability:post:update,post:delete');
    Route::delete('/posts/{post}', [PostController::class, 'destroy'])
        ->middleware('ability:post:update,post:delete');
});
```

**Expected Output:**

- `GET /api/posts` with any valid token → `200 OK`.
- `POST /api/posts` with a token that has `post:create` → `201 Created`.
- `POST /api/posts` with a token that lacks `post:create` → `403 Forbidden`.
- `DELETE /api/posts/1` with a token that has `post:delete` → `204 No Content`.
- `DELETE /api/posts/1` with a token that has neither `post:update` nor `post:delete` → `403 Forbidden`.

**Why This Output Occurs:** The `auth:sanctum` middleware authenticates the request first. Then, the `abilities` or `ability` middleware checks the token's abilities array. If the token does not have the required ability, a `403 Forbidden` response is returned. This layered approach separates authentication (who you are) from authorization (what you can do).

### Real-World Cases

- **API resources:** All CRUD endpoints for a resource are protected by `auth:sanctum`, ensuring only authenticated users can access them.
- **Mixed public/private APIs:** Public endpoints (e.g., `/api/health`, `/api/docs`) are unprotected, while data endpoints are wrapped in an `auth:sanctum` group.
- **Admin-only endpoints:** Combine `auth:sanctum` with a custom `admin` middleware or Laravel Policies to restrict access to administrative routes.
- **Third-party API access:** External developers authenticate with Sanctum tokens, while first-party SPAs use session cookies — both resolved by the same `auth:sanctum` guard.

---

## 5. CORS & Stateful Domains

### Definitions

**Core Definition:** CORS (Cross-Origin Resource Sharing) is a browser security mechanism that restricts cross-origin HTTP requests, while stateful domains are the specific origins that Sanctum is configured to trust for cookie-based session authentication.

**Technical Definition:** Sanctum's SPA authentication requires that the frontend and backend share a common top-level domain (or that `SESSION_DOMAIN` is configured to allow cross-subdomain cookies). The `SANCTUM_STATEFUL_DOMAINS` environment variable lists the origins from which Sanctum will accept session cookies. The `config/cors.php` file must be configured with `supports_credentials => true` and explicit `allowed_origins` (not `*`) to allow the browser to send cookies with cross-origin requests. Without proper CORS configuration, the browser will block the SPA's requests to the API with a CORS error, even if the credentials are valid.

**Beginner-Friendly Explanation:** Browsers have a security rule: a website at `app.example.com` cannot automatically send requests to `api.example.com` and include cookies, unless the server explicitly says "I trust `app.example.com`." That's what CORS and stateful domains are for. `SANCTUM_STATEFUL_DOMAINS` tells Sanctum which frontend origins are trusted, and `config/cors.php` tells the browser which origins are allowed to make credentialed requests. If these are not configured correctly, your SPA will get a CORS error and authentication will fail.

### Purposes

- To allow a first-party SPA hosted on a different subdomain or port to authenticate with the Laravel API using cookies.
- To prevent malicious third-party websites from making authenticated requests to your API using a user's cookies.
- To configure the browser's CORS policy to allow credentialed requests from trusted origins.
- To ensure that session cookies are sent and received correctly across subdomains.
- To prevent the "Unauthenticated" error that occurs when CORS blocks the session cookie from being sent.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// File: config/cors.php

return [
    'paths' => ['api/*', 'sanctum/csrf-cookie', 'login', 'logout'],

    'allowed_methods' => ['*'],

    'allowed_origins' => [
        'http://localhost:3000',
        'https://app.example.com',
    ],

    'allowed_origins_patterns' => [],

    'allowed_headers' => ['*'],

    'exposed_headers' => [],

    'max_age' => 0,

    'supports_credentials' => true, // CRITICAL for Sanctum SPA authentication
];
```

```env
# .env
SANCTUM_STATEFUL_DOMAINS=localhost:3000,app.example.com
SESSION_DOMAIN=.example.com
```

**Component Breakdown:**

- `'paths' => ['api/*', 'sanctum/csrf-cookie', 'login']` — The URI paths that CORS should apply to. Must include `sanctum/csrf-cookie` and the login route.
- `'allowed_origins'` — The exact origins that are allowed to make cross-origin requests. **Must not be `['*']` when `supports_credentials` is `true`.**
- `'supports_credentials' => true` — Enables the `Access-Control-Allow-Credentials: true` header, which is required for the browser to send cookies with cross-origin requests.
- `SANCTUM_STATEFUL_DOMAINS` — Comma-separated list of origins that Sanctum treats as first-party (stateful). These origins receive session cookies.
- `SESSION_DOMAIN` — The domain for the session cookie. Set to `.example.com` to share cookies across `app.example.com` and `api.example.com`.

**Syntax Rules:**

- `supports_credentials` **must** be `true` for SPA cookie authentication to work.
- `allowed_origins` **must not** be `['*']` when `supports_credentials` is `true`. Browsers block credentialed requests to wildcard origins.
- `SANCTUM_STATEFUL_DOMAINS` should include the exact origin of the SPA, including the port (e.g., `localhost:3000`).
- `SESSION_DOMAIN` should be set to the common parent domain (e.g., `.example.com`) to allow cookie sharing across subdomains.
- The `paths` array must include `sanctum/csrf-cookie` for the CSRF cookie to be set.

**Constraints and Limitations:**

- **CORS is enforced by the browser, not the server.** Even if the server returns the correct headers, the browser will block the response if the origin does not match `allowed_origins`.
- **Cookies with `SameSite=Strict` or `SameSite=Lax` may not be sent cross-origin.** For cross-subdomain SPA authentication, `SESSION_SAME_SITE` should be set to `none` and `SESSION_SECURE_COOKIE` to `true` (requires HTTPS).
- **The `SANCTUM_STATEFUL_DOMAINS` and `SESSION_DOMAIN` settings must be consistent.** If the SPA origin is not listed in both, authentication will fail.
- **In Laravel 11+, the CORS configuration is handled by the `HandleCors` middleware**, which is included by default. In Laravel 10-, ensure the `HandleCors` middleware is registered.

### Annotated Code Examples

**Example 1: Complete CORS and Stateful Domain Configuration**

```env
# .env — Local development
APP_URL=http://localhost:8000
SESSION_DOMAIN=localhost
SANCTUM_STATEFUL_DOMAINS=localhost:3000,127.0.0.1:3000
SESSION_SAME_SITE=lax
SESSION_SECURE_COOKIE=false
```

```env
# .env — Production
APP_URL=https://api.example.com
SESSION_DOMAIN=.example.com
SANCTUM_STATEFUL_DOMAINS=app.example.com
SESSION_SAME_SITE=none
SESSION_SECURE_COOKIE=true
```

```php
// File: config/cors.php

return [
    'paths' => ['api/*', 'sanctum/csrf-cookie', 'login', 'logout'],

    'allowed_methods' => ['*'],

    'allowed_origins' => [
        'http://localhost:3000',
        'https://app.example.com',
    ],

    'allowed_headers' => ['*'],

    'supports_credentials' => true,
];
```

```javascript
// File: SPA frontend — Axios configuration
import axios from 'axios';

axios.defaults.withCredentials = true; // Required for cookies to be sent
axios.defaults.baseURL = 'http://localhost:8000';
```

**Step-by-Step Setup:**

1. Set `SANCTUM_STATEFUL_DOMAINS` and `SESSION_DOMAIN` in `.env`.
2. Configure `config/cors.php` with `supports_credentials => true` and explicit `allowed_origins`.
3. In the SPA, set `withCredentials: true` on Axios (or `credentials: 'include'` on fetch).
4. Test: `GET /sanctum/csrf-cookie` should return a `XSRF-TOKEN` cookie. `POST /login` should set a session cookie.

**Expected Output:**

- `GET /sanctum/csrf-cookie` → `204 No Content` with `Set-Cookie: XSRF-TOKEN=...`.
- `POST /login` with valid credentials → `200 OK` with `Set-Cookie: laravel_session=...`.
- `GET /api/user` with the session cookie → `200 OK` with the user JSON.
- No CORS errors in the browser console.

**Why This Output Occurs:** The `HandleCors` middleware checks the `Origin` header of the incoming request against the `allowed_origins` array. If it matches, it adds the `Access-Control-Allow-Origin` and `Access-Control-Allow-Credentials: true` headers to the response. The browser, seeing these headers, allows the response to be read and sends cookies with subsequent requests. The `SANCTUM_STATEFUL_DOMAINS` setting ensures Sanctum applies session middleware to requests from these origins.

---

**Example 2: Debugging CORS Issues**

```php
// Common CORS errors and their solutions:

// Error: "Access to XMLHttpRequest at 'http://api.example.com/api/user' from origin
// 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin'
// header is present on the requested resource."

// Solution 1: Add the origin to config/cors.php
'allowed_origins' => ['http://localhost:3000'],

// Solution 2: Add the origin to SANCTUM_STATEFUL_DOMAINS in .env
SANCTUM_STATEFUL_DOMAINS=localhost:3000

// Error: "The value of the 'Access-Control-Allow-Origin' header in the response must not
// be the wildcard '*' when the request's credentials mode is 'include'."

// Solution: Replace '*' with explicit origins
'allowed_origins' => ['http://localhost:3000'], // NOT ['*']
'supports_credentials' => true,
```

**Expected Output:** After applying the correct configuration, the browser console shows no CORS errors, and the SPA can make authenticated requests to the API using cookies.

**Why This Output Occurs:** The browser enforces CORS policies for security. When `withCredentials` is `true`, the browser refuses to send cookies if the server responds with `Access-Control-Allow-Origin: *`. By specifying explicit origins and enabling `supports_credentials`, the browser allows the credentialed request to proceed.

### Real-World Cases

- **SPAs on different subdomains:** `app.example.com` (frontend) and `api.example.com` (backend) with `SESSION_DOMAIN=.example.com`.
- **Local development:** SPA on `localhost:3000` and Laravel API on `localhost:8000` with `SANCTUM_STATEFUL_DOMAINS=localhost:3000`.
- **Next.js on Vercel:** Frontend on `myapp.vercel.app` and API on `api.example.com`; requires `SANCTUM_STATEFUL_DOMAINS=myapp.vercel.app` and proper CORS configuration.
- **Multi-tenant SaaS:** Each tenant has a subdomain (e.g., `tenant1.example.com`, `tenant2.example.com`), all sharing the same API at `api.example.com` with `SESSION_DOMAIN=.example.com`.

---

## 6. Token Expiration & Pruning

### Definitions

**Core Definition:** Token expiration is the configuration that determines how long a Sanctum API token remains valid before it is considered expired, while pruning is the scheduled removal of expired tokens from the `personal_access_tokens` table to prevent database bloat.

**Technical Definition:** Sanctum's token expiration is configured via the `expiration` key in `config/sanctum.php`, which specifies the number of minutes until an issued token is considered expired. This value **overrides** any `expires_at` value set on individual tokens, but does not affect first-party sessions (SPA authentication). The `sanctum:prune-expired` Artisan command deletes expired tokens from the database. It implements a two-phase pruning strategy: Phase 1 deletes tokens with an explicit `expires_at` timestamp in the past, and Phase 2 deletes tokens older than the configured `sanctum.expiration` value (if set).

**Beginner-Friendly Explanation:** By default, Sanctum tokens never expire — they are like keys that work forever until you explicitly revoke them. This can be a security risk: if a token is stolen, it remains valid indefinitely. Setting an expiration time ensures that tokens automatically become invalid after a set period (e.g., 7 days). But even after a token expires, its record remains in the database unless you run a cleanup command. Pruning removes these expired records, keeping your database tidy and your authentication queries fast.

### Purposes

- To limit the lifespan of API tokens, reducing the impact of stolen or leaked credentials.
- To comply with security best practices and regulatory requirements for credential rotation.
- To automatically invalidate tokens that have not been used for a specified period.
- To prevent the `personal_access_tokens` table from growing unbounded, which can slow authentication lookups.
- To provide a mechanism for token rotation, where clients must obtain new tokens periodically.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// File: config/sanctum.php — Token expiration configuration

return [
    // ...

    /*
    |--------------------------------------------------------------------------
    | Expiration Minutes
    |--------------------------------------------------------------------------
    |
    | This value controls the number of minutes until an issued token will be
    | considered expired. If this value is null, personal access tokens do
    | not expire. This won't tweak the lifetime of first-party sessions.
    |
    */

    'expiration' => 60 * 24 * 7, // 7 days

    // ...
];
```

```env
# .env — Optional environment variable for expiration
SANCTUM_TOKEN_EXPIRY_MINUTES=1440 # 24 hours
```

```php
// Schedule the pruning command (Laravel 11+ — routes/console.php)
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

```bash
# Manual pruning
php artisan sanctum:prune-expired

# Prune tokens expired more than 24 hours ago
php artisan sanctum:prune-expired --hours=24

# Prune tokens expired more than 7 days ago (168 hours)
php artisan sanctum:prune-expired --hours=168
```

**Component Breakdown:**

- `'expiration' => 60 * 24 * 7` — Sets token expiration to 7 days (in minutes). A value of `null` means tokens never expire.
- `SANCTUM_TOKEN_EXPIRY_MINUTES` — Optional environment variable that can be used to set the expiration dynamically (e.g., `(int) env('SANCTUM_TOKEN_EXPIRY_MINUTES', 1440)`).
- `Schedule::command('sanctum:prune-expired --hours=24')->daily()` — Schedules the pruning command to run daily, deleting tokens that expired more than 24 hours ago.
- `--hours=24` — Only deletes tokens that expired more than 24 hours ago, providing a buffer for clock skew or timezone differences.
- `--hours=168` — Deletes tokens expired more than 7 days ago (168 hours = 7 days).

**Syntax Rules:**

- The `expiration` value is in **minutes**, not seconds or days. For example, 7 days = `60 * 24 * 7 = 10080` minutes.
- A value of `null` (the default) means tokens never expire.
- The `expiration` setting **overrides** any `expires_at` value set on individual tokens. If `expiration` is set, all tokens expire after that duration regardless of their `expires_at` value.
- The `expiration` setting does **not** affect first-party SPA sessions, which use Laravel's session lifetime configuration instead.
- The `sanctum:prune-expired` command should be scheduled to run regularly (daily or hourly) to prevent table bloat.

**Constraints and Limitations:**

- **Token expiration is not enforced by default.** If `expiration` is `null`, tokens remain valid indefinitely unless manually revoked.
- **Pruning is not automatic.** Even with expiration configured, expired token records remain in the database until the `sanctum:prune-expired` command is run.
- **The `--hours` flag must be set correctly.** If `--hours=24` is used but tokens expire after 7 days, only tokens that expired more than 24 hours ago are deleted — tokens that expired 12 hours ago remain.
- **Phase 2 pruning requires `sanctum.expiration` to be set.** If `expiration` is `null`, Phase 2 pruning does not run, and only tokens with explicit `expires_at` timestamps are pruned.

### Annotated Code Examples

**Example 1: Configuring Token Expiration and Scheduling Pruning**

```php
<?php
// File: config/sanctum.php

return [
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
        '%s%s%s',
        'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
        Sanctum::currentApplicationUrlWithPort(),
        env('FRONTEND_URL') ? ','.parse_url(env('FRONTEND_URL'), PHP_URL_HOST) : ''
    ))),

    'guard' => ['web'],

    // Tokens expire after 7 days (in minutes)
    'expiration' => 60 * 24 * 7,

    // ...
];
```

```php
<?php
// File: routes/console.php (Laravel 11+)

use Illuminate\Support\Facades\Schedule;

// Prune expired tokens daily, removing tokens expired more than 24 hours ago
Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

```php
// File: app/Console/Kernel.php (Laravel 10-)

protected function schedule(Schedule $schedule)
{
    $schedule->command('sanctum:prune-expired --hours=24')->daily();
}
```

**Step-by-Step Setup:**

1. Set `'expiration' => 60 * 24 * 7` in `config/sanctum.php`.
2. Add the scheduling command to `routes/console.php` (Laravel 11+) or `app/Console/Kernel.php` (Laravel 10-).
3. Ensure Laravel's scheduler is running (cron job on the server).
4. Test: Create a token, wait for expiration (or temporarily set `expiration` to 1 minute), and verify that the token is rejected.

**Expected Output:**

- After 7 days, API tokens are rejected with `401 Unauthenticated`.
- The `sanctum:prune-expired --hours=24` command deletes tokens that expired more than 24 hours ago.
- The `personal_access_tokens` table remains clean and performant.

**Why This Output Occurs:** The `expiration` setting causes Sanctum to reject tokens older than 7 days. The `prune-expired` command deletes these expired token records from the database. Phase 1 of the command deletes tokens with an explicit `expires_at` timestamp in the past. Phase 2 deletes tokens older than the `sanctum.expiration` value (7 days). The `--hours=24` flag provides a 24-hour buffer, ensuring that tokens are not deleted immediately after expiration.

---

**Example 2: Per-Token Expiration with Global Configuration Override**

```php
<?php
// File: app/Http/Controllers/Api/AuthController.php

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

    // Create a token with a custom expiration (30 days)
    // Note: This expires_at value is overridden by config/sanctum.php
    // if the global expiration is set to a shorter duration.
    $token = $user->createToken(
        'custom-expiry-token',
        ['*'],
        now()->addDays(30)
    );

    return response()->json([
        'token'      => $token->plainTextToken,
        'expires_at' => now()->addDays(30)->toIso8601String(),
    ]);
}
```

```php
// config/sanctum.php — Global expiration set to 7 days
'expiration' => 60 * 24 * 7, // 7 days — overrides the 30-day custom expiration
```

**Expected Output:**

- The token response indicates a 30-day expiration, but the token actually expires after 7 days because the global `sanctum.expiration` value overrides the per-token `expires_at`.
- After 7 days, the token is rejected with `401 Unauthenticated`.

**Why This Output Occurs:** The `expiration` setting in `config/sanctum.php` takes precedence over any `expires_at` value set on individual tokens. This is a common source of confusion: developers set a per-token expiration, but the global expiration overrides it. To use per-token expiration exclusively, set `'expiration' => null` in `config/sanctum.php` and rely solely on the `expires_at` parameter of `createToken()`.

### Real-World Cases

- **Mobile app tokens:** Tokens expire after 30 days, requiring users to log in again monthly. This limits the impact of a stolen token.
- **CI/CD tokens:** Long-lived tokens (1 year) with aggressive pruning to prevent accumulation.
- **API integrations:** Short-lived tokens (24 hours) with refresh mechanisms, requiring clients to obtain new tokens daily.
- **High-security applications:** Tokens expire after 1 hour, with refresh tokens used to obtain new access tokens without re-authentication.
- **Low-traffic applications:** Weekly pruning is sufficient, while high-traffic APIs may require hourly pruning to prevent table bloat.

---

## References

- Laravel Sanctum Documentation (13.x) — https://laravel.com/docs/13.x/sanctum
- Laravel Sanctum Documentation (12.x) — https://laravel.com/docs/12.x/sanctum
- Laravel Sanctum Documentation (11.x) — https://laravel.com/docs/11.x/sanctum
- Laravel Sanctum GitHub Repository — https://github.com/laravel/sanctum
- Laravel Sanctum: SPA Authentication — https://laravel.com/docs/12.x/sanctum#spa-authentication
- Laravel Sanctum: API Token Authentication — https://laravel.com/docs/12.x/sanctum#api-token-authentication
- Laravel Sanctum: Token Abilities — https://laravel.com/docs/12.x/sanctum#token-abilities
- Laravel Sanctum: Token Expiration — https://laravel.com/docs/12.x/sanctum#token-expiration
- Laravel Sanctum: Revoking Tokens — https://laravel.com/docs/12.x/sanctum#revoking-tokens
- Laravel Sanctum: Pruning Expired Tokens — https://laravel.com/docs/12.x/sanctum#pruning-expired-tokens
- Laravel CORS Configuration — https://laravel.com/docs/routing#cors
- Personal Access Tokens Deep Dive (DeepWiki) — https://deepwiki.com/hypervel/sanctum/3-personal-access-tokens
- Token Abilities and Authorization (DeepWiki) — https://deepwiki.com/hypervel/sanctum/4-token-abilities-and-authorization
- Token Maintenance and Pruning (DeepWiki) — https://deepwiki.com/hypervel/sanctum/6-token-maintenance
- Sanctum Configuration File — https://github.com/laravel/sanctum/blob/4.x/config/sanctum.php
- `EnsureFrontendRequestsAreStateful` Middleware — https://github.com/laravel/sanctum/blob/4.x/src/Http/Middleware/EnsureFrontendRequestsAreStateful.php
- `sanctum:prune-expired` Command — https://github.com/laravel/sanctum/blob/4.x/src/Console/Commands/PruneExpired.php
- Laravel Sanctum CORS Issues (Stack Overflow) — https://stackoverflow.com/questions/59915500/laravel-sanctum-cors-issue
- Managing Authorization with Sanctum and Spatie Permissions (Laracasts) — https://laracasts.com/discuss/channels/laravel/managing-authorization-with-sanctum-and-spatie-permissions
- Sanctum Token Expiry (Laracasts) — https://laracasts.com/discuss/channels/laravel/sanctum-token-expiry