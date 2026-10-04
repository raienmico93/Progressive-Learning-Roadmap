# Laravel API Authentication — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel API Authentication is the set of mechanisms provided by the framework to verify the identity of clients accessing API endpoints, supporting token-based, session-based, and OAuth2 authentication flows through first-party packages Sanctum and Passport.

**Technical Definition:** Laravel API Authentication is implemented through "guards" defined in `config/auth.php`, which determine how users are authenticated for each request. The framework provides two official packages: **Laravel Sanctum**, a lightweight authentication system for SPAs, mobile applications, and simple token-based APIs, and **Laravel Passport**, a full OAuth2 server implementation built on the League OAuth2 server. Sanctum stores API tokens in a single database table and authenticates requests via the `Authorization: Bearer` header, while also offering cookie-based session authentication for first-party SPAs. Passport provides OAuth2 grants including authorization code, client credentials, and password grants, issuing JSON Web Tokens (JWT) as access tokens.

**Beginner-Friendly Explanation:** When you build an API, you need a way to know who is making each request. Laravel gives you two main tools for this. **Sanctum** is the simple option: users log in, get a token (a long random string), and include that token in every subsequent request. It is perfect for mobile apps and single-page applications. **Passport** is the full-featured option: it implements the OAuth2 standard, which is what you need when third-party applications want to let their users log in with your service (like "Sign in with Google"). Most applications should start with Sanctum and only use Passport if they genuinely need OAuth2.

### Key Characteristics

- **Guard-based architecture:** Authentication is resolved through guards defined in `config/auth.php`, with `auth:sanctum` and `auth:api` being the most common for APIs.
- **Two-in-one Sanctum:** Sanctum provides both API token authentication (for mobile/third-party) and cookie-based SPA authentication (for first-party JavaScript applications).
- **Token abilities (scopes):** Sanctum tokens can be granted specific abilities that limit what actions the token can perform, analogous to OAuth2 scopes.
- **Full OAuth2 with Passport:** Passport implements all standard OAuth2 grants (authorization code, client credentials, password, implicit) and issues JWT access tokens.
- **Stateless vs. stateful:** Token authentication is stateless (no server-side session), while SPA authentication with Sanctum uses Laravel's session guard and is stateful for first-party requests.
- **Easy installation:** Sanctum is included by default in recent Laravel versions; Passport is installed via `php artisan install:api --passport`.

### Prerequisites

- PHP 8.1+ (Laravel 10+) or PHP 8.2+ (Laravel 11+).
- Composer dependency manager.
- A Laravel application with `routes/api.php` configured (via `php artisan install:api` in Laravel 11+).
- Basic understanding of HTTP headers and JSON.
- For SPA authentication: a frontend application (Vue, React, Next.js) running on a stateful domain.
- For Passport: familiarity with OAuth2 concepts and terminology.

### Related Programming Areas

- **Laravel Sanctum** — Lightweight token and SPA authentication.
- **Laravel Passport** — Full OAuth2 server implementation.
- **Middleware** — Authentication guards are applied via middleware (`auth:sanctum`, `auth:api`).
- **Eloquent** — The `HasApiTokens` trait is added to the `User` model for token management.
- **SPA Frameworks** — Vue, React, and Next.js applications use Sanctum's cookie-based SPA authentication.
- **JWT (JSON Web Tokens)** — Passport issues JWTs as access tokens.

### Core Concepts / Features

1. **Token Authentication:** Implementing Laravel Sanctum for lightweight token issuance and management.
2. **SPA Authentication:** Cookie-based, stateful session authentication for single-page apps (Vue/React/Next.js).
3. **Personal Access Tokens:** Generating, managing, expiring, and revoking tokens with specific abilities/scopes.
4. **OAuth-Related Concepts:** Implementing Laravel Passport for full OAuth2 server capabilities, client grants, and JWT management.

---

## 1. Token Authentication (Laravel Sanctum)

### Definitions

**Core Definition:** Laravel Sanctum token authentication is a lightweight mechanism for issuing API tokens to users, allowing stateless authentication of API requests without the complexity of OAuth2.

**Technical Definition:** Sanctum's token authentication stores user API tokens in a `personal_access_tokens` database table, hashing the token value with SHA-256 for security. Tokens are issued via the `createToken()` method on the `HasApiTokens` trait, which returns a `NewAccessToken` instance containing the plain-text token. Incoming requests are authenticated by the `auth:sanctum` middleware, which checks the `Authorization: Bearer` header for a valid token and resolves the associated user. Tokens are typically long-lived (years) but can be manually revoked or configured to expire.

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

**Syntax Rules:**

- The `createToken()` method **must** be called on a model that uses the `HasApiTokens` trait.
- The `auth:sanctum` middleware is applied to routes in `routes/api.php` or route groups.
- The token is sent by the client in the `Authorization: Bearer <token>` header.
- Multiple tokens can be created for the same user, each with its own name and abilities.

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
            'email'    => 'required|email',
            'password' => 'required',
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

```php
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

## 2. SPA Authentication (Cookie-Based)

### Definitions

**Core Definition:** Sanctum SPA authentication is a cookie-based, stateful session authentication mechanism for single-page applications running on a stateful domain, using Laravel's built-in session guard instead of API tokens.

**Technical Definition:** Sanctum's SPA authentication uses Laravel's `web` authentication guard and session cookies to authenticate requests from first-party SPAs. The `EnsureFrontendRequestsAreStateful` middleware (part of Sanctum's middleware stack) checks whether the incoming request originates from a configured stateful domain. If so, it applies session middleware (including CSRF protection) before the request reaches the API routes. The SPA must first request a CSRF cookie from `/sanctum/csrf-cookie`, then include the `X-XSRF-TOKEN` header (automatically handled by Axios) in subsequent requests. Sanctum does **not** use tokens for this flow — it relies entirely on session cookies.

**Beginner-Friendly Explanation:** If your SPA (a Vue or React app) lives on the same domain as your Laravel backend (like `app.example.com` and `api.example.com`), you can use cookies for authentication just like a traditional website. Sanctum handles the tricky parts: it tells Laravel which domains should receive cookies, and it ensures that CSRF protection works for your SPA. This is more secure than storing tokens in `localStorage` because cookies can be `HttpOnly` (JavaScript cannot read them), protecting against XSS attacks.

### Purposes

- To provide secure, cookie-based authentication for first-party SPAs without exposing tokens to JavaScript.
- To leverage Laravel's built-in session management and CSRF protection for SPAs.
- To avoid the security risks of storing authentication tokens in `localStorage` or `sessionStorage`.
- To enable session-based authentication for SPAs hosted on the same top-level domain as the API.
- To integrate seamlessly with popular SPA frameworks (Vue, React, Next.js) using Axios or fetch.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Configure stateful domains in config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
    Sanctum::currentApplicationUrlWithPort()
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

- `SANCTUM_STATEFUL_DOMAINS` — Comma-separated list of domains that should receive stateful (session) authentication cookies.
- `EnsureFrontendRequestsAreStateful` — Middleware that checks the request's `Origin` or `Referer` header; if it matches a stateful domain, it applies session and CSRF middleware.
- `/sanctum/csrf-cookie` — Route provided by Sanctum that sets the `XSRF-TOKEN` cookie, which Axios automatically reads and sends as the `X-XSRF-TOKEN` header.
- `SESSION_DOMAIN` — Set to `.example.com` to share cookies across subdomains (e.g., `app.example.com` and `api.example.com`).

**Syntax Rules:**

- The SPA domain **must** be listed in `SANCTUM_STATEFUL_DOMAINS` for cookie authentication to work.
- The SPA must send requests with credentials (`withCredentials: true` in Axios, or `credentials: 'include'` in fetch).
- The first request to the API must be to `/sanctum/csrf-cookie` to obtain the CSRF token.
- The `auth:sanctum` middleware works for both token and SPA authentication; it checks cookies first, then falls back to tokens.
- Sanctum SPA authentication requires the SPA and API to share the same top-level domain, or `SESSION_DOMAIN` must be configured appropriately.

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
        '%s%s',
        'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
        Sanctum::currentApplicationUrlWithPort()
    ))),

    'guard' => ['web'], // The guard used for SPA authentication
    'expiration' => null, // Session lifetime (null = default)
];
```

```php
// File: routes/api.php — Note: login route is in web.php for session auth
// routes/web.php
Route::post('/login', [AuthController::class, 'login']);

// routes/api.php
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
2. Ensure `EnsureFrontendRequestsAreStateful` middleware is in the `api` middleware group (Laravel 10-). In Laravel 11+, run `php artisan install:api`.
3. Configure Axios in the SPA with `withCredentials: true`.
4. Implement the login flow: call `/sanctum/csrf-cookie`, then `POST /login`, then make authenticated requests.
5. Ensure the login route is in `routes/web.php` (not `api.php`) so it has session middleware.

**Expected Output:**

- After `GET /sanctum/csrf-cookie`, the browser receives a `XSRF-TOKEN` cookie.
- `POST /login` with valid credentials sets a `laravel_session` cookie and returns a `200 OK`.
- `GET /api/user` with the session cookie returns the authenticated user's JSON.
- The SPA does not need to handle tokens manually — cookies are managed automatically by the browser.

**Why This Output Occurs:** The `EnsureFrontendRequestsAreStateful` middleware detects that the request originates from a stateful domain (`localhost:3000` or `app.example.com`) by checking the `Origin` header. It then applies `StartSession` and `VerifyCsrfToken` middleware to the request. The `/sanctum/csrf-cookie` route sets the `XSRF-TOKEN` cookie, and Axios automatically reads this cookie and sends it as the `X-XSRF-TOKEN` header in subsequent requests. The session cookie (`laravel_session`) is set after login and sent with every subsequent request, authenticating the user.

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

## 3. Personal Access Tokens

### Definitions

**Core Definition:** Personal access tokens are API tokens issued to a user for their own account, allowing them to authenticate programmatically without going through a login flow each time.

**Technical Definition:** Personal access tokens in Sanctum are created via the `createToken()` method, which accepts a token name, an optional array of abilities (scopes), and an optional `DateTime` expiration. The token is hashed (SHA-256) and stored in the `personal_access_tokens` table alongside the token name, abilities, and expiration timestamp. The plain-text token is returned only once. Tokens can be revoked by deleting them from the database, and can be configured to expire globally in `config/sanctum.php` via the `expiration` key.

**Beginner-Friendly Explanation:** Personal access tokens are like API keys that users generate for themselves. For example, on GitHub, you can go to your settings and create a "personal access token" to use with the GitHub CLI or a script. The same concept applies here: a user of your application can create a token named "My Script" with specific permissions (abilities), and use that token to authenticate API requests. The token can be revoked at any time if it is compromised.

### Purposes

- To allow users to authenticate programmatically without sharing their password.
- To enable per-device or per-application tokens with descriptive names (e.g., "iPhone 15", "CI/CD Pipeline").
- To restrict what a token can do via abilities/scopes, following the principle of least privilege.
- To provide a mechanism for users to manage their own API access (create, list, and revoke tokens).
- To support long-lived tokens for automation, scripts, and third-party integrations.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Creating a token with abilities
$token = $user->createToken('token-name', ['post:read', 'post:create'])->plainTextToken;

// Creating a token with an expiration (Laravel 11+)
$token = $user->createToken('token-name', ['post:read'], now()->addDays(30))->plainTextToken;

// Creating a token with no abilities (all abilities)
$token = $user->createToken('token-name')->plainTextToken;

// Listing a user's tokens
$tokens = $user->tokens; // Collection of PersonalAccessToken models

// Revoking a specific token
$user->tokens()->where('id', $tokenId)->delete();

// Revoking all tokens
$user->tokens()->delete();

// Revoking the current token
$request->user()->currentAccessToken()->delete();
```

```php
// Checking abilities in a controller
if ($request->user()->tokenCan('post:create')) {
    // Allow the action
}

// Checking abilities via middleware
Route::middleware(['auth:sanctum', 'abilities:post:create'])->post('/posts', ...);
Route::middleware(['auth:sanctum', 'ability:post:create,post:update'])->post('/posts', ...);
```

**Component Breakdown:**

- `createToken(string $name, array $abilities = ['*'], DateTimeInterface $expiresAt = null)` — Creates a new token. The `$name` is a human-readable identifier. The `$abilities` array limits what the token can do. The `$expiresAt` sets an expiration.
- `$user->tokens` — Eloquent relationship returning all tokens for the user.
- `$request->user()->tokenCan('ability')` — Checks whether the current token has the specified ability.
- `abilities:ability1,ability2` middleware — Requires the token to have **all** listed abilities.
- `ability:ability1,ability2` middleware — Requires the token to have **at least one** of the listed abilities.
- `config/sanctum.php` — The `expiration` key sets a global expiration time (in minutes) for all tokens.

**Syntax Rules:**

- The default abilities array is `['*']`, which grants all abilities.
- Token abilities are checked against the token, not the user's role. A user with admin role but a token without the `post:create` ability cannot create posts.
- The `abilities` middleware (plural) requires **all** listed abilities; the `ability` middleware (singular) requires **any** of the listed abilities.
- Tokens can be expired globally via `config/sanctum.php` (`'expiration' => 60 * 24 * 7` for 7 days) or per-token via the third argument to `createToken()`.

**Constraints and Limitations:**

- **Token abilities are not a replacement for authorization policies.** Use them in conjunction with Gates and Policies for comprehensive access control.
- **Global expiration overrides per-token expiration.** If `sanctum.expiration` is set, it takes precedence over any `expires_at` value on individual tokens.
- **Revoked tokens cannot be un-revoked.** Once deleted from the database, the token is permanently invalid.
- **Token abilities are stored as a JSON array** in the `abilities` column of the `personal_access_tokens` table.

### Annotated Code Examples

**Example 1: Token Management CRUD Endpoints**

```php
<?php
// File: app/Http/Controllers/Api/TokenController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class TokenController extends Controller
{
    // GET /api/tokens — List the user's tokens
    public function index(Request $request): JsonResponse
    {
        return response()->json($request->user()->tokens);
    }

    // POST /api/tokens — Create a new token
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'       => 'required|string|max:255',
            'abilities'  => 'sometimes|array',
            'abilities.*' => 'string',
            'expires_in_days' => 'sometimes|integer|min:1|max:365',
        ]);

        $expiresAt = isset($validated['expires_in_days'])
            ? now()->addDays($validated['expires_in_days'])
            : null;

        $token = $request->user()->createToken(
            $validated['name'],
            $validated['abilities'] ?? ['*'],
            $expiresAt
        );

        return response()->json([
            'token'      => $token->plainTextToken,
            'name'       => $validated['name'],
            'abilities'  => $validated['abilities'] ?? ['*'],
            'expires_at' => $expiresAt?->toIso8601String(),
        ], 201);
    }

    // DELETE /api/tokens/{id} — Revoke a token
    public function destroy(Request $request, int $id): JsonResponse
    {
        $deleted = $request->user()->tokens()->where('id', $id)->delete();

        if (!$deleted) {
            return response()->json(['error' => 'Token not found.'], 404);
        }

        return response()->json(null, 204);
    }

    // DELETE /api/tokens — Revoke all tokens
    public function destroyAll(Request $request): JsonResponse
    {
        $request->user()->tokens()->delete();
        return response()->json(['message' => 'All tokens revoked.'], 200);
    }
}
```

**Step-by-Step Setup:**

1. Define the routes: `Route::apiResource('tokens', TokenController::class)->except(['show', 'update']);`
2. Protect the routes with `auth:sanctum` middleware.
3. Test creating a token with specific abilities: `POST /api/tokens` with `{"name": "CI Pipeline", "abilities": ["post:read"], "expires_in_days": 90}`.
4. Test using the token to access a protected route.

**Expected Output (Token Creation):**

```json
{
    "token": "3|xyz789abc123...",
    "name": "CI Pipeline",
    "abilities": ["post:read"],
    "expires_at": "2025-09-15T10:00:00+00:00"
}
```

**Expected Output (Using the Token):**

- `GET /api/posts` with `Authorization: Bearer 3|xyz789abc123...` → `200 OK` (token has `post:read`).
- `POST /api/posts` with the same token → `403 Forbidden` (token lacks `post:create`).

**Why This Output Occurs:** The `createToken()` method stores the token with the specified abilities and expiration. When the token is used, the `auth:sanctum` middleware resolves it, and the `abilities` middleware checks whether the token has the required ability. If not, a `403 Forbidden` response is returned. The global expiration (if set) overrides the per-token expiration.

---

**Example 2: Global Token Expiration and Pruning**

```php
<?php
// File: config/sanctum.php

return [
    // Tokens expire after 30 days (in minutes)
    'expiration' => 60 * 24 * 30,
];
```

```bash
# Schedule the pruning command to run daily
# In app/Console/Kernel.php (Laravel 10-) or routes/console.php (Laravel 11+):

# Laravel 11+: routes/console.php
Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

```php
// Manual pruning
use Laravel\Sanctum\PersonalAccessToken;

PersonalAccessToken::where('expires_at', '<', now())->delete();
```

**Expected Output:** Tokens older than 30 days are automatically rejected by the `auth:sanctum` middleware. The `sanctum:prune-expired` command removes expired token rows from the database, keeping the table clean.

**Why This Output Occurs:** The `sanctum.expiration` configuration sets a global expiration time. When a token is used, Sanctum checks whether it has exceeded the expiration time and rejects it if so. The `sanctum:prune-expired` command deletes expired tokens from the database. Without pruning, expired tokens remain in the database but are rejected at authentication time.

### Real-World Cases

- **API access for users:** A SaaS application allows users to generate tokens for their own scripts and integrations, with specific abilities limiting what each token can do.
- **CI/CD pipelines:** A developer creates a token with the `deploy` ability, scoped to their repository, and uses it in a GitHub Actions workflow.
- **Mobile apps with multiple devices:** A user can create a separate token for each device (iPhone, iPad, Android), allowing them to revoke access for a lost device without affecting others.
- **Third-party integrations:** When a third-party service needs API access, the user creates a token with limited abilities (e.g., `read:orders`) and provides it to the service.

---

## 4. OAuth-Related Concepts (Laravel Passport)

### Definitions

**Core Definition:** Laravel Passport is a full OAuth2 server implementation that provides OAuth2 authorization flows, client management, and JWT access token issuance for Laravel applications.

**Technical Definition:** Passport is built on the League OAuth2 server and implements the OAuth2 specification (RFC 6749). It provides authorization server endpoints (`/oauth/authorize`, `/oauth/token`) and supports multiple grant types: authorization code, client credentials, password, and implicit. Access tokens issued by Passport are JSON Web Tokens (JWT) signed with RSA keys, containing claims such as `sub` (user ID), `aud` (client ID), and `exp` (expiration). Passport also includes a JSON API for managing OAuth2 clients and tokens.

**Beginner-Friendly Explanation:** OAuth2 is the standard that powers "Log in with Google" or "Connect to Facebook." It lets users grant third-party applications limited access to their data without sharing their password. Passport is Laravel's implementation of OAuth2. It is more complex than Sanctum, but it is the right choice when you are building a public API that other developers will integrate with. For example, if you want other applications to let their users log in with their account on your platform, you need Passport.

### Purposes

- To implement a full OAuth2 authorization server for your application.
- To enable third-party applications to request access to user data with user consent.
- To support machine-to-machine authentication via the client credentials grant.
- To issue JWT access tokens that can be validated without database lookups.
- To provide refresh tokens that allow clients to obtain new access tokens without re-authentication.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Step 1: Install Passport
php artisan install:api --passport

# Step 2: Run migrations
php artisan migrate

# Step 3: Generate encryption keys
php artisan passport:keys
```

```php
// Step 4: Add HasApiTokens trait and OAuthenticatable interface to User model
namespace App\Models;

use Laravel\Passport\HasApiTokens;
use Laravel\Passport\Contracts\OAuthenticatable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens;
}
```

```php
// Step 5: Configure the api guard in config/auth.php
'guards' => [
    'api' => [
        'driver'   => 'passport',
        'provider' => 'users',
    ],
],
```

```php
// Step 6: Protect routes with auth:api middleware
Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});
```

#### Client Credentials Grant (Machine-to-Machine)

```bash
# Create a client credentials grant client
php artisan passport:client --client
```

```php
// Protect routes with EnsureClientIsResourceOwner middleware
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

Route::get('/orders', function (Request $request) {
    // Access token is valid and the client is resource owner
})->middleware(EnsureClientIsResourceOwner::class);
```

```php
// Request a token using client credentials
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type'    => 'client_credentials',
    'client_id'     => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope'         => 'servers:read servers:create',
]);

return $response->json()['access_token'];
```

#### Authorization Code Grant

```php
// Redirect the user to the authorization endpoint
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id'     => 'your-client-id',
        'redirect_uri'  => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope'         => 'user:read orders:create',
        'state'         => $state,
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

**Component Breakdown:**

- `php artisan install:api --passport` — Installs Passport, publishes migrations, and creates encryption keys.
- `HasApiTokens` trait and `OAuthenticatable` interface — Added to the `User` model to enable Passport integration.
- `'driver' => 'passport'` — Configures the `api` guard to use Passport's `TokenGuard` for authentication.
- `EnsureClientIsResourceOwner` — Middleware for client credentials grant routes; ensures the token belongs to a client, not a user.
- `oauth/token` endpoint — The OAuth2 token endpoint; accepts grant type, client credentials, and scope.
- `oauth/authorize` endpoint — The OAuth2 authorization endpoint; users approve or deny access requests.

**Syntax Rules:**

- The `api` guard **must** be set to `driver => passport` in `config/auth.php`.
- Routes protected by Passport use the `auth:api` middleware (not `auth:sanctum`).
- Client credentials grant routes **must** use the `EnsureClientIsResourceOwner` middleware, not `auth:api`.
- Passport encryption keys **must** be generated and kept secure; they are not committed to source control.
- The `oauth/token` and `oauth/authorize` routes are defined automatically by Passport.

**Constraints and Limitations:**

- **Passport is significantly more complex than Sanctum.** For simple API token authentication, Sanctum is recommended.
- **Passport requires RSA encryption keys.** These must be generated and managed securely; losing them invalidates all issued tokens.
- **Client credentials tokens have no user context.** The `sub` claim is the client ID, not a user ID. Routes must use `EnsureClientIsResourceOwner` middleware.
- **Passport's password grant is deprecated** in OAuth 2.1 and should be avoided for new applications; use authorization code with PKCE instead.
- **JWT tokens cannot be revoked** in the same way as opaque tokens; Passport maintains a revocation list, but token invalidation is not instantaneous.

### Annotated Code Examples

**Example 1: Client Credentials Grant for Machine-to-Machine Authentication**

```php
<?php
// File: routes/api.php — Protect a route for machine-to-machine access

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

// This route is accessible only to authenticated clients (no user context)
Route::get('/servers', function (Request $request) {
    return response()->json(['servers' => ['server-1', 'server-2']]);
})->middleware(EnsureClientIsResourceOwner::using('servers:read'));
```

```php
<?php
// File: app/Console/Commands/SyncServers.php — A scheduled job that uses client credentials

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;

class SyncServers extends Command
{
    protected $signature = 'sync:servers';
    protected $description = 'Sync server data using client credentials grant';

    public function handle(): void
    {
        // Step 1: Request an access token using client credentials
        $response = Http::asForm()->post(config('app.url') . '/oauth/token', [
            'grant_type'    => 'client_credentials',
            'client_id'     => config('services.passport.client_id'),
            'client_secret' => config('services.passport.client_secret'),
            'scope'         => 'servers:read servers:create',
        ]);

        $accessToken = $response->json()['access_token'];

        // Step 2: Use the token to call the protected API
        $servers = Http::withToken($accessToken)
            ->get(config('app.url') . '/api/servers');

        $this->info('Servers: ' . $servers->body());
    }
}
```

**Step-by-Step Setup:**

1. Run `php artisan install:api --passport` and `php artisan migrate`.
2. Run `php artisan passport:client --client` to create a client credentials client. Note the client ID and secret.
3. Add the client credentials to `.env` or `config/services.php`.
4. Protect the `/servers` route with `EnsureClientIsResourceOwner::using('servers:read')`.
5. Run the command: `php artisan sync:servers`.

**Expected Output:**

```json
{ "servers": ["server-1", "server-2"] }
```

**Why This Output Occurs:** The `EnsureClientIsResourceOwner` middleware validates the client credentials access token and checks that the token has the `servers:read` scope. Because the token is a client credentials token (not a user token), the middleware ensures the client is the resource owner. The route then returns the server data. The client credentials grant is ideal for machine-to-machine communication where no user context is required.

---

**Example 2: Authorization Code Grant with PKCE for a Third-Party SPA**

```php
<?php
// File: routes/web.php — Redirect to authorization endpoint
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/oauth/redirect', function (Request $request) {
    // Generate a code verifier and challenge (PKCE)
    $codeVerifier = Str::random(128);
    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $codeVerifier, true)),
        '='
    ), '+/', '-_');

    $request->session()->put('code_verifier', $codeVerifier);

    $query = http_build_query([
        'client_id'             => 'your-client-id',
        'redirect_uri'          => 'https://third-party-app.com/callback',
        'response_type'         => 'code',
        'scope'                 => 'user:read orders:create',
        'state'                 => Str::random(40),
        'code_challenge'        => $codeChallenge,
        'code_challenge_method' => 'S256',
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

```php
// File: Third-party app callback — Exchange authorization code for token
Route::get('/callback', function (Request $request) {
    $codeVerifier = $request->session()->pull('code_verifier');

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type'    => 'authorization_code',
        'client_id'     => 'your-client-id',
        'redirect_uri'  => 'https://third-party-app.com/callback',
        'code'          => $request->code,
        'code_verifier' => $codeVerifier,
    ]);

    $tokens = $response->json();

    // Store access_token and refresh_token for the user
    return response()->json($tokens);
});
```

**Expected Output:**

```json
{
    "token_type": "Bearer",
    "expires_in": 31536000,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
    "refresh_token": "def50200..."
}
```

**Why This Output Occurs:** The authorization code grant with PKCE (Proof Key for Code Exchange) is the most secure OAuth2 flow for SPAs and mobile apps. The client generates a code verifier and challenge, redirects the user to the authorization endpoint, and receives an authorization code. The client then exchanges the code (along with the code verifier) for an access token and refresh token. PKCE prevents authorization code interception attacks.

### Real-World Cases

- **Public APIs with third-party developers:** A platform like Stripe or GitHub uses OAuth2 to allow third-party applications to access user data with consent.
- **"Sign in with..." functionality:** An application uses the authorization code grant to let users log in with their account on another platform.
- **Machine-to-machine communication:** Microservices authenticate to each other using the client credentials grant, with no user context.
- **Enterprise SSO:** Passport can be used as an OAuth2 provider for internal single sign-on across multiple applications.

---

## References

- Laravel Sanctum Documentation (12.x) — https://laravel.com/docs/12.x/sanctum
- Laravel Sanctum Documentation (10.x) — https://laravel.com/docs/10.x/sanctum
- Laravel Passport Documentation (12.x) — https://laravel.com/docs/12.x/passport
- Laravel Passport Documentation (13.x) — https://laravel.com/framework/docs/13.x/passport
- Authentication in Laravel, Part 2: Token Authentication (CODE Magazine) — https://www.codemag.com/article/2309081
- Laravel Sanctum vs Passport: Which One Should You Use? (Bagisto) — https://bagisto.com/en/laravel-sanctum-vs-passport-which-one-should-you-use/
- SecPal API Authentication Documentation — https://github.com/SecPal/api/blob/main/api/docs/api/authentication.md
- Laravel Sanctum GitHub Repository — https://github.com/laravel/sanctum
- Laravel Passport GitHub Repository — https://github.com/laravel/passport
- RFC 6749: The OAuth 2.0 Authorization Framework — https://www.rfc-editor.org/rfc/rfc6749
- OAuth 2.0 PKCE (RFC 7636) — https://www.rfc-editor.org/rfc/rfc7636
- JWT (JSON Web Token) RFC 7519 — https://www.rfc-editor.org/rfc/rfc7519
- Laravel Sanctum: Token Abilities Documentation — https://laravel.com/docs/12.x/sanctum#token-abilities
- Laravel Sanctum: Revoking Tokens — https://laravel.com/docs/12.x/sanctum#revoking-tokens
- Laravel Sanctum: Token Expiration — https://laravel.com/docs/12.x/sanctum#token-expiration
- Laravel Passport: Client Credentials Grant — https://laravel.com/docs/12.x/passport#client-credentials-grant
- Laravel Passport: Authorization Code Grant — https://laravel.com/docs/12.x/passport#authorization-code-grant
- Laravel Passport: Managing Clients — https://laravel.com/docs/12.x/passport#managing-clients
- Laravel Passport: Managing Tokens — https://laravel.com/docs/12.x/passport#managing-tokens
- Laravel Passport: JWT Custom Claims (Packagist) — https://packagist.org/packages/benbjurstrom/passport-custom-jwt-claims