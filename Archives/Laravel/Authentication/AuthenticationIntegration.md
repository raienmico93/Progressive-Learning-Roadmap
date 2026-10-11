# API and Single Page Application (SPA) Authentication: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** API and SPA authentication in Laravel encompasses the tools and patterns for authenticating clients that are not traditional server-rendered web browsers—including single-page applications (SPAs) running in the browser and native mobile applications communicating with a stateless API.

**Technical Definition:** Laravel provides two primary packages for API authentication: **Laravel Sanctum** (a lightweight system for SPAs and mobile apps) and **Laravel Passport** (a full OAuth2 server implementation). Sanctum offers two distinct authentication mechanisms: (1) stateful, cookie-based session authentication for first-party SPAs via `EnsureFrontendRequestsAreStateful` middleware and CSRF protection, and (2) stateless API tokens for mobile applications and third-party clients via the `HasApiTokens` trait. Passport implements the full OAuth2 specification with Authorization Code, Password, Client Credentials, and Device Authorization grants, along with Scopes for fine-grained permission control. Cross-Origin Resource Sharing (CORS) and credential handling (`withCredentials: true`) are configured in `config/cors.php` and the client's HTTP library to enable secure cross-domain cookie transmission.

**Beginner-Friendly Explanation:** When your frontend is a separate application (like a React or Vue SPA) running on a different domain or port than your Laravel backend, you need special authentication setup. Sanctum is the lightweight option: for SPAs, it uses cookies and sessions (like traditional web auth but with CORS handling); for mobile apps, it issues API tokens. Passport is the heavy-duty option: it's a complete OAuth2 server for when you need to let third-party applications access your users' data with their permission. The key challenge with SPAs is making sure cookies are sent cross-origin (CORS with credentials) and CSRF tokens are handled properly.

### Key Characteristics

- **Two Sanctum Modes:** Stateful cookie-based SPA auth and stateless token-based mobile/API auth in one package.
- **CSRF Protection for SPAs:** The `/sanctum/csrf-cookie` endpoint initializes CSRF protection before login.
- **Token Abilities:** Sanctum tokens can be scoped with abilities for granular permission control.
- **Full OAuth2 with Passport:** Supports Authorization Code, Password, Client Credentials, and Device grants.
- **Scopes for Enterprise:** Passport's scopes define what actions a token can perform on behalf of a user.
- **CORS with Credentials:** `supports_credentials => true` and `withCredentials: true` are required for cross-domain cookies.
- **Stateful Domains:** Sanctum treats only configured domains as first-party, enabling session-based auth for SPAs.

### Prerequisites

- A Laravel application with `users` table and `User` model.
- Composer for package installation.
- For Sanctum: `laravel/sanctum` package installed and migrated.
- For Passport: `laravel/passport` package installed and migrated.
- For SPA: A frontend application (React, Vue, etc.) on a separate domain/port.
- For CORS: `config/cors.php` published and configured.
- HTTPS in production for secure cookie transmission.

### Related Programming Areas

- **Session Management:** Sanctum's SPA mode relies on Laravel's session driver.
- **CSRF Protection:** Sanctum's SPA mode uses Laravel's CSRF token system.
- **OAuth2 Specification:** Passport implements the OAuth2 RFC standards.
- **Middleware:** `EnsureFrontendRequestsAreStateful` and `auth:sanctum` are central to Sanctum.
- **Token Hashing:** Sanctum stores hashed tokens in the database.

### Core Concepts / Features

1. Laravel Sanctum (SPA Cookie Auth vs. Mobile API Tokens)
2. Laravel Passport (OAuth2 Server, Grants, Scopes, Clients)
3. CORS and Credentials for SPAs

---

## 1. Laravel Sanctum: SPA Cookie Auth vs. Mobile API Tokens

### Definitions

**Core Definition:** Laravel Sanctum is a lightweight authentication package that provides both stateful, cookie-based session authentication for first-party SPAs and stateless API token issuance for mobile applications and third-party clients.

**Technical Definition:** Sanctum operates in two modes. In **SPA mode**, the `EnsureFrontendRequestsAreStateful` middleware checks the request's `Origin` or `Referer` header against the `SANCTUM_STATEFUL_DOMAINS` configuration. If the origin is first-party, the request is routed through Laravel's session and CSRF middleware, enabling cookie-based authentication. In **token mode**, users generate API tokens via `$user->createToken($deviceName, $abilities)`, which returns a `NewAccessToken` object containing a `plainTextToken`. The token is stored hashed in the `personal_access_tokens` table. Protected routes use the `auth:sanctum` middleware.

**Beginner-Friendly Explanation:** Sanctum gives you two ways to authenticate. For your own SPA (React/Vue app), it uses cookies just like a traditional web app—you log in, get a session cookie, and the browser sends it automatically. For mobile apps, you create an API token (like a password) that the app stores and sends in the `Authorization: Bearer` header. The same package handles both, and you can use both simultaneously.

### Purposes

- To authenticate first-party SPAs using stateful cookies and sessions.
- To issue API tokens for mobile applications and third-party API clients.
- To provide token abilities (scopes) for granular permission control.
- To enable token expiration and revocation.
- To avoid the complexity of OAuth2 for simple token-based APIs.

### Syntax Rules and Structure

**Complete General Syntax — SPA Mode Setup:**

```php
// bootstrap/app.php (Laravel 11+)
->withMiddleware(function (Middleware $middleware) {
    $middleware->statefulApi();
})
```

```php
// config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost:3000,localhost:8000')),
```

```env
# .env
SESSION_DRIVER=database
SANCTUM_STATEFUL_DOMAINS=localhost:3000,localhost:8000
SESSION_DOMAIN=.localhost
```

**Complete General Syntax — SPA Login Flow:**

```javascript
// 1. Get CSRF cookie
await axios.get('/sanctum/csrf-cookie');

// 2. Login
await axios.post('/login', {
    email: 'user@example.com',
    password: 'password',
}, {
    withCredentials: true,
    withXSRFToken: true,
});
```

**Complete General Syntax — Mobile Token Issuance:**

```php
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});
```

**Complete General Syntax — Token Abilities and Expiration:**

```php
// Create token with abilities
$token = $user->createToken('mobile-app', ['post:read', 'post:write']);

// Check abilities
if ($user->tokenCan('post:write')) {
    // Allow write
}

// Token expiration (config/sanctum.php)
'expiration' => 60 * 24 * 7, // 7 days in minutes
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `statefulApi()` | Enables Sanctum's stateful middleware for SPAs. |
| `SANCTUM_STATEFUL_DOMAINS` | First-party origins treated as stateful. |
| `/sanctum/csrf-cookie` | Initializes CSRF protection. |
| `createToken($name, $abilities)` | Issues a new API token. |
| `plainTextToken` | The plain-text token returned to the client. |
| `tokenCan('ability')` | Checks if the token has a specific ability. |

**Syntax Rules:**
- The `HasApiTokens` trait must be added to the `User` model.
- SPA mode requires `SESSION_DRIVER=database` (or another persistent driver).
- `SANCTUM_STATEFUL_DOMAINS` must include the SPA's origin (e.g., `localhost:3000`).
- The `/sanctum/csrf-cookie` endpoint must be called before login.
- Tokens are stored hashed; only the plain-text token is returned once.
- Token abilities are checked via `$user->tokenCan('ability')` or `$request->user()->tokenCan('ability')`.

**Constraints and Limitations:**
- **SPA cookie domain:** The session cookie domain must support subdomains (`'.domain.com'`).
- **CORS requirements:** `supports_credentials` must be `true` in `config/cors.php`.
- **Mobile token storage:** Tokens must be stored securely on the device (Keychain, Keystore).
- **Token revocation:** Tokens can be revoked via `$user->tokens()->delete()` or deleting specific tokens.
- **Expiration:** Token expiration is configured globally in `config/sanctum.php`.

### Multiple Annotated Complete Code Examples

**Example 1: SPA Login Flow with CSRF**

```javascript
// Frontend (React/Vue) using Axios
import axios from 'axios';

// Configure Axios globally
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
axios.defaults.baseURL = 'http://localhost:8000';

// Login function
async function login(email, password) {
    // Step 1: Get CSRF cookie
    await axios.get('/sanctum/csrf-cookie');

    // Step 2: Attempt login
    try {
        const response = await axios.post('/login', {
            email,
            password,
        });
        return response.data;
    } catch (error) {
        if (error.response.status === 422) {
            // Validation errors
        }
        throw error;
    }
}

// Fetch authenticated user
async function getUser() {
    const response = await axios.get('/api/user');
    return response.data;
}
```

**Expected Output:** On successful login, the browser receives a session cookie and is authenticated for subsequent requests. `getUser()` returns the authenticated user's data.

**Why:** The `/sanctum/csrf-cookie` request initializes the CSRF token. The login request includes the `X-XSRF-TOKEN` header automatically via `withXSRFToken`. The session cookie is sent automatically due to `withCredentials` .

---

**Example 2: Mobile Token Issuance and Usage**

```php
<?php
// Backend: Token endpoint
Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});
```

```javascript
// Mobile app: Login and store token
async function mobileLogin(email, password, deviceName) {
    const response = await fetch('/sanctum/token', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
        },
        body: JSON.stringify({
            email,
            password,
            device_name: deviceName,
        }),
    });

    const token = await response.text();
    // Store token securely (Keychain/Keystore)
    await SecureStore.setItemAsync('api_token', token);
    return token;
}

// Authenticated request
async function fetchUser() {
    const token = await SecureStore.getItemAsync('api_token');
    const response = await fetch('/api/user', {
        headers: {
            'Authorization': `Bearer ${token}`,
            'Accept': 'application/json',
        },
    });
    return response.json();
}
```

**Expected Output:** The mobile app receives a plain-text API token, stores it securely, and uses it as a Bearer token for subsequent API requests.

**Why:** `createToken()` returns a `NewAccessToken` object. `plainTextToken` is the only time the plain-text token is available. The client must store it securely and send it in the `Authorization: Bearer` header .

---

**Example 3: Token Abilities in Controllers**

```php
<?php
// Backend: Checking token abilities
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/posts', function (Request $request) {
        // Check if token has 'post:read' ability
        if (! $request->user()->tokenCan('post:read')) {
            abort(403, 'Token does not have read permission.');
        }

        return Post::all();
    });

    Route::post('/posts', function (Request $request) {
        // Check if token has 'post:write' ability
        if (! $request->user()->tokenCan('post:write')) {
            abort(403, 'Token does not have write permission.');
        }

        return Post::create($request->validated());
    });
});
```

**Expected Output:** Tokens with `post:read` ability can retrieve posts; tokens with `post:write` ability can create posts. Tokens without the required ability receive a 403.

**Why:** `tokenCan()` checks the token's abilities array. First-party UI requests (via session cookie) always return `true` for `tokenCan()`, so policies should also check user permissions .

### Real-World Cases

- **First-Party SPA:** A React dashboard communicating with a Laravel API using session cookies and CSRF.
- **Mobile App:** An iOS/Android app that authenticates via email/password and stores a long-lived API token.
- **Third-Party Integrations:** Simple token-based APIs for trusted partners.
- **Multi-Client:** A single backend serving both a SPA (cookie auth) and a mobile app (token auth) simultaneously.

### References

- Laravel Sanctum Documentation (13.x) — https://laravel.com/framework/docs/13.x/sanctum 
- Laravel Sanctum Documentation (11.x) — https://raw.githubusercontent.com/laravel/docs/11.x/sanctum.md 
- Sanctum 12.x Docs — https://github.com/Vectorial1024/docs/blob/12.x/sanctum.md 
- How does Sanctum SPA authentication work exactly? (Laracasts) — https://laracasts.com/discuss/channels/laravel/how-does-sanctum-spa-authentication-work-exactly 

---

## 2. Laravel Passport: Full OAuth2 Server Implementation

### Definitions

**Core Definition:** Laravel Passport is a full OAuth2 server implementation that allows your application to issue OAuth2 access tokens to third-party clients, enabling them to access your API on behalf of your users with their explicit permission.

**Technical Definition:** Passport implements the OAuth2 specification via the `league/oauth2-server` library. It provides routes for authorization (`/oauth/authorize`), token issuance (`/oauth/token`), client management, and token revocation. Supported grants include **Authorization Code** (for third-party web apps), **Password** (for first-party mobile apps), **Client Credentials** (for machine-to-machine), and **Device Authorization** (for devices with limited input). Scopes define the permissions a token can perform, and are checked via `$token->can('scope')` or `$user->tokenCan('scope')`.

**Beginner-Friendly Explanation:** Passport is the enterprise-grade OAuth2 solution. When you want to let a third-party app (like a calendar app accessing a user's Google Calendar) access your API with the user's permission, Passport handles the entire OAuth2 flow. The user is redirected to your app, approves the request, and the third-party app receives a token. Passport is more complex than Sanctum but necessary when you need proper OAuth2 with third-party clients.

### Purposes

- To implement a full OAuth2 authorization server for third-party API access.
- To support multiple OAuth2 grant types (Authorization Code, Password, Client Credentials, Device).
- To define and enforce scopes for fine-grained permission control.
- To manage OAuth2 clients (first-party and third-party).
- To issue access tokens and refresh tokens with configurable lifetimes.
- To support enterprise-tier API architectures with strict security requirements.

### Syntax Rules and Structure

**Complete General Syntax — Installation:**

```bash
composer require laravel/passport
php artisan migrate
php artisan passport:install
```

**Complete General Syntax — Configuration:**

```php
// config/auth.php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

```php
// app/Models/User.php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens;
}
```

**Complete General Syntax — Defining Scopes:**

```php
// AuthServiceProvider::boot()
use Laravel\Passport\Passport;

Passport::tokensCan([
    'place-orders' => 'Place orders',
    'check-status' => 'Check order status',
]);

Passport::setDefaultScope(['check-status']);
```

**Complete General Syntax — Authorization Code Grant (Third-Party):**

```php
// Third-party app redirects user to:
// https://your-app.com/oauth/authorize?client_id=...&redirect_uri=...&response_type=code&scope=place-orders

// After approval, third-party receives authorization code
// Third-party exchanges code for token:
$response = Http::asForm()->post('https://your-app.com/oauth/token', [
    'grant_type' => 'authorization_code',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'redirect_uri' => 'https://third-party-app.com/callback',
    'code' => $authorizationCode,
]);
```

**Complete General Syntax — Password Grant (First-Party Mobile):**

```php
// Create password grant client
php artisan passport:client --password

// Request token
$response = Http::asForm()->post('https://your-app.com/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'username' => 'user@example.com',
    'password' => 'password',
    'scope' => 'place-orders',
]);
```

**Complete General Syntax — Device Authorization Grant:**

```php
// Create device flow client
php artisan passport:client --device

// Request device code
$response = Http::asForm()->post('https://your-app.com/oauth/device/code', [
    'client_id' => 'client-id',
    'scope' => 'user:read',
]);

// Response: device_code, user_code, verification_uri, interval, expires_in

// Poll for token
$response = Http::asForm()->post('https://your-app.com/oauth/token', [
    'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'device_code' => 'the-device-code',
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `/oauth/authorize` | Authorization endpoint (user approval). |
| `/oauth/token` | Token issuance/refresh endpoint. |
| `/oauth/device/code` | Device code request endpoint. |
| `Passport::tokensCan()` | Defines available scopes. |
| `passport:client` | Creates OAuth2 clients. |
| `HasApiTokens` trait | Adds token methods to the User model. |

**Syntax Rules:**
- Passport must be installed and the `api` guard configured with `driver => passport`.
- The `HasApiTokens` trait must be added to the `User` model.
- OAuth2 clients must be created via `passport:client` or the `ClientRepository` class.
- Scopes must be defined in `AuthServiceProvider::boot()`.
- Access tokens are sent as `Authorization: Bearer <token>`.
- The `passport:keys` command generates encryption keys (required in production).

**Constraints and Limitations:**
- **Password grant deprecated:** The OAuth2 Password Grant is no longer recommended by OAuth2 standards .
- **Complexity:** Passport is significantly more complex to set up and maintain than Sanctum.
- **Token storage:** Passport stores tokens in the database; high-volume applications may need database optimization.
- **Scopes required:** Without scopes, tokens have full access; scopes should always be defined.

### Multiple Annotated Complete Code Examples

**Example 1: Authorization Code Grant Flow**

```php
<?php
// Step 1: Redirect user to authorization endpoint
Route::get('/redirect-to-provider', function () {
    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'place-orders check-status',
        'state' => Str::random(40),
    ]);

    return redirect('https://your-app.com/oauth/authorize?' . $query);
});
```

```php
<?php
// Step 2: Handle callback and exchange code for token
Route::get('/callback', function (Request $request) {
    $response = Http::asForm()->post('https://your-app.com/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    $token = $response->json();
    // $token['access_token'], $token['refresh_token'], $token['expires_in']

    // Store tokens securely
    session(['access_token' => $token['access_token']]);
    session(['refresh_token' => $token['refresh_token']]);

    return redirect('/dashboard');
});
```

**Expected Output:** The user is redirected to the authorization page, approves the request, and the third-party app receives access and refresh tokens.

**Why:** The Authorization Code grant is the standard OAuth2 flow for third-party web applications. The user explicitly approves access, and the client exchanges the authorization code for tokens .

---

**Example 2: Password Grant for First-Party Mobile**

```php
<?php
// Backend: Create password grant client
// php artisan passport:client --password

// Mobile app: Request token
$response = Http::asForm()->post('https://api.example.com/oauth/token', [
    'grant_type' => 'password',
    'client_id' => '2', // Password grant client ID
    'client_secret' => 'client-secret',
    'username' => $email,
    'password' => $password,
    'scope' => 'place-orders check-status',
]);

$token = $response->json('access_token');
// Store token securely on device
```

**Expected Output:** The mobile app receives an access token and refresh token without the redirect flow.

**Why:** The Password grant is designed for first-party clients (your own mobile apps) where users trust the application with their credentials .

---

**Example 3: Scope-Based Route Protection**

```php
<?php
// Backend: Protect routes with scopes
Route::middleware('auth:api')->group(function () {
    Route::get('/orders', function (Request $request) {
        if (! $request->user()->tokenCan('check-status')) {
            abort(403, 'Token does not have check-status scope.');
        }
        return Order::all();
    })->middleware('scope:check-status');

    Route::post('/orders', function (Request $request) {
        return Order::create($request->validated());
    })->middleware('scope:place-orders');
});
```

**Expected Output:** Tokens with the `check-status` scope can retrieve orders; tokens with `place-orders` scope can create orders. Tokens without the required scope receive a 403.

**Why:** The `scope` middleware checks the token's scopes. `tokenCan()` provides programmatic checks within controllers .

### Real-World Cases

- **Enterprise APIs:** Companies exposing APIs to third-party partners (e.g., payment gateways, CRMs).
- **Social Login Providers:** OAuth2 servers allowing third-party apps to authenticate users.
- **Device Authorization:** Smart TVs, IoT devices, and CLI tools that need user approval on another device.
- **Machine-to-Machine:** Client Credentials grant for server-to-server communication without user context.

### References

- Laravel Passport Documentation (12.x) — https://raw.githubusercontent.com/laravel/docs/refs/heads/12.x/passport.md 
- Laravel Passport Documentation (master) — https://laravel.com/framework/docs/master/passport 
- Laravel Passport Documentation (5.4) — https://laravel.com/docs/5.4/passport 
- Laravel Passport Documentation (6.x) — https://laravel.com/framework/docs/6.x/passport 
- API Authentication (Passport) 5.5 — https://raw.githubusercontent.com/jameshfisher/docs/refs/heads/5.5/passport.md 

---

## 3. CORS and Credentials for SPAs

### Definitions

**Core Definition:** Cross-Origin Resource Sharing (CORS) is a browser security mechanism that controls which origins can access resources on a different origin. For SPA authentication, CORS must be configured to allow credentials (cookies) to be sent cross-origin.

**Technical Definition:** Laravel's `HandleCors` middleware reads the `config/cors.php` configuration to set response headers. For credentialed requests (cookies), three conditions must be met: (1) `supports_credentials` must be `true` in `config/cors.php`, (2) `allowed_origins` must list explicit origins (wildcard `*` is forbidden with credentials), and (3) the client must set `withCredentials: true`. Additionally, the session cookie domain must support the SPA's domain (`.domain.com`), and Sanctum's `stateful` domains must include the SPA's origin.

**Beginner-Friendly Explanation:** When your SPA on `localhost:3000` talks to your API on `localhost:8000`, the browser considers them different origins and blocks cookies by default. CORS configuration tells the browser "it's okay to send cookies from this origin." You must explicitly allow the SPA's origin, enable credentials, and tell your HTTP client to include cookies. Without this, the browser won't send the session cookie, and your API will return 401 Unauthorized.

### Purposes

- To allow SPAs on different origins to make credentialed requests to the API.
- To securely transmit session cookies across domains.
- To prevent unauthorized cross-origin access while allowing legitimate first-party clients.
- To handle CSRF tokens in cross-origin requests.
- To support subdomain-based SPA architectures (e.g., `app.domain.com` and `api.domain.com`).

### Syntax Rules and Structure

**Complete General Syntax — `config/cors.php`:**

```php
<?php
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

    'supports_credentials' => true,
];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `paths` | The routes that CORS applies to. |
| `allowed_origins` | Explicit origins allowed (no wildcard with credentials). |
| `allowed_methods` | HTTP methods allowed. |
| `allowed_headers` | Request headers allowed. |
| `supports_credentials` | Must be `true` for cookies. |

**Complete General Syntax — Axios Configuration:**

```javascript
// resources/js/app.js or a dedicated axios config
import axios from 'axios';

axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
axios.defaults.baseURL = 'http://localhost:8000';
```

**Complete General Syntax — Sanctum Stateful Domains:**

```env
# .env
SESSION_DRIVER=database
SESSION_DOMAIN=.localhost
SANCTUM_STATEFUL_DOMAINS=localhost:3000,localhost:8000
```

```php
// config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost:3000,localhost:8000')),
```

**Complete General Syntax — Session Cookie Domain:**

```php
// config/session.php
'domain' => env('SESSION_DOMAIN', null), // '.domain.com' for subdomains
'secure' => env('SESSION_SECURE_COOKIE', true), // HTTPS only in production
'http_only' => true,
'same_site' => 'lax',
```

**Syntax Rules:**
- `supports_credentials` must be `true` for cookies to be sent cross-origin.
- `allowed_origins` cannot use `*` when `supports_credentials` is `true`; explicit origins are required.
- `SANCTUM_STATEFUL_DOMAINS` must include the SPA's origin (with port if non-standard).
- `SESSION_DOMAIN` should be set to `.domain.com` (with leading dot) to share cookies across subdomains.
- The client must set `withCredentials: true` in Axios (or equivalent).
- The `/sanctum/csrf-cookie` endpoint must be included in `paths`.
- In production, `SESSION_SECURE_COOKIE` must be `true` (HTTPS only).

**Constraints and Limitations:**
- **Wildcard origins forbidden:** With `supports_credentials: true`, `allowed_origins` cannot be `*`.
- **Cookie domain mismatch:** If `SESSION_DOMAIN` doesn't match the SPA's domain, cookies won't be sent.
- **localhost vs 127.0.0.1:** Mixing `localhost` and `127.0.0.1` causes CORS failures; use one consistently.
- **CSRF token mismatch:** The CSRF token must be properly decrypted and matched; `withXSRFToken: true` handles this in Axios.
- **HTTPS required in production:** Secure cookies require HTTPS; `SESSION_SECURE_COOKIE=false` only for local development.

### Multiple Annotated Complete Code Examples

**Example 1: Complete SPA CORS Setup**

```php
<?php
// config/cors.php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie', 'login', 'logout'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['http://localhost:3000'],
    'allowed_headers' => ['*'],
    'supports_credentials' => true,
];
```

```env
# .env
APP_URL=http://localhost:8000
SESSION_DRIVER=database
SESSION_DOMAIN=.localhost
SANCTUM_STATEFUL_DOMAINS=localhost:3000,localhost:8000
SESSION_SECURE_COOKIE=false
```

```javascript
// resources/js/axios.js
import axios from 'axios';

const api = axios.create({
    baseURL: 'http://localhost:8000',
    withCredentials: true,
    withXSRFToken: true,
    headers: {
        'Accept': 'application/json',
        'X-Requested-With': 'XMLHttpRequest',
    },
});

export default api;
```

```javascript
// Login flow
import api from './axios';

async function login(email, password) {
    // Step 1: Initialize CSRF
    await api.get('/sanctum/csrf-cookie');

    // Step 2: Login
    const response = await api.post('/login', { email, password });
    return response.data;
}
```

**Expected Output:** The SPA successfully authenticates via session cookie. Subsequent requests to `/api/*` are authenticated.

**Why:** `supports_credentials` allows cookies. `SANCTUM_STATEFUL_DOMAINS` marks the SPA as first-party. `withCredentials` sends cookies. The CSRF cookie is obtained before login .

---

**Example 2: Subdomain SPA Architecture**

```php
<?php
// config/cors.php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie', 'login', 'logout'],
    'allowed_origins' => ['https://app.example.com'],
    'supports_credentials' => true,
];
```

```env
# .env
SESSION_DOMAIN=.example.com
SANCTUM_STATEFUL_DOMAINS=app.example.com,api.example.com
SESSION_SECURE_COOKIE=true
```

**Expected Output:** The SPA at `app.example.com` authenticates with the API at `api.example.com` using a shared session cookie.

**Why:** `SESSION_DOMAIN=.example.com` allows the cookie to be sent to all subdomains. `SANCTUM_STATEFUL_DOMAINS` includes both origins .

---

**Example 3: Debugging Common CORS Issues**

```javascript
// Common issue: 401 Unauthorized after login
// Check 1: Is withCredentials true?
axios.defaults.withCredentials = true; // MUST be true

// Check 2: Is the CSRF cookie obtained first?
await axios.get('/sanctum/csrf-cookie'); // MUST call before login

// Check 3: Is withXSRFToken true?
axios.defaults.withXSRFToken = true; // MUST be true for Axios to send X-XSRF-TOKEN

// Check 4: Is the origin allowed?
// config/cors.php: allowed_origins must include the exact SPA origin

// Check 5: Is SANCTUM_STATEFUL_DOMAINS correct?
// Must include the SPA origin with port
```

**Expected Output:** Following these checks resolves the most common CORS/credentials issues.

**Why:** The most common causes of 401 errors in SPA auth are missing `withCredentials`, missing CSRF initialization, incorrect `allowed_origins`, or misconfigured `SANCTUM_STATEFUL_DOMAINS` .

### Real-World Cases

- **Decoupled SPA:** React/Vue app on `app.example.com` talking to Laravel API on `api.example.com`.
- **Local Development:** SPA on `localhost:3000` and API on `localhost:8000`.
- **Same-Origin Proxy:** Next.js API routes proxying to Laravel, avoiding CORS entirely .
- **Multi-Subdomain:** Different subdomains for different parts of the application sharing cookies.

### References

- Laravel Sanctum: CORS and Cookies (13.x) — https://laravel.com/framework/docs/13.x/sanctum 
- Laravel Sanctum CSRF Token Issue (Stack Overflow) — https://stackoverflow.com/questions/79184995/laravel-sanctum-csrf-token-issue-401-unauthorized 
- How can I send authenticated requests from React to a Laravel API using Sanctum? (Stack Overflow) — https://stackoverflow.com/questions/79622283/how-can-i-send-authenticated-requests-from-react-to-a-laravel-api-using-sanctum 
- HandleCors Middleware API — https://api.laravel.com/docs/13.x/Illuminate/Http/Middleware/HandleCors.html 

---

## References

- Laravel Sanctum Documentation (13.x) — https://laravel.com/framework/docs/13.x/sanctum 
- Laravel Sanctum Documentation (11.x) — https://raw.githubusercontent.com/laravel/docs/11.x/sanctum.md 
- Sanctum 12.x Docs (Vectorial1024) — https://github.com/Vectorial1024/docs/blob/12.x/sanctum.md 
- Laravel Passport Documentation (12.x) — https://raw.githubusercontent.com/laravel/docs/refs/heads/12.x/passport.md 
- Laravel Passport Documentation (master) — https://laravel.com/framework/docs/master/passport 
- Laravel Passport Documentation (5.4) — https://laravel.com/docs/5.4/passport 
- Laravel Passport Documentation (6.x) — https://laravel.com/framework/docs/6.x/passport 
- API Authentication (Passport) 5.5 — https://raw.githubusercontent.com/jameshfisher/docs/refs/heads/5.5/passport.md 
- HandleCors Middleware API — https://api.laravel.com/docs/13.x/Illuminate/Http/Middleware/HandleCors.html 
- Airlock Renamed to Sanctum — https://laravel.com/blog/airlock-renamed-to-sanctum 
- Laravel Sanctum CSRF Token Issue (Stack Overflow) — https://stackoverflow.com/questions/79184995/laravel-sanctum-csrf-token-issue-401-unauthorized 
- How can I send authenticated requests from React to Laravel using Sanctum? (Stack Overflow) — https://stackoverflow.com/questions/79622283/how-can-i-send-authenticated-requests-from-react-to-a-laravel-api-using-sanctum 
- How does Sanctum SPA authentication work exactly? (Laracasts) — https://laracasts.com/discuss/channels/laravel/how-does-sanctum-spa-authentication-work-exactly 
- Jetstream API Documentation — https://jetstream.laravel.com/features/api.html 
- Laravel Sanctum (8.x) — https://laravel.com/docs/8.x/sanctum 
- Laravel 的 API 认证系统 Passport — https://raw.githubusercontent.com/laravel-china/laravel-docs/refs/heads/5.5/passport.md 
- aliziodev/laravel-next-starter-kit (Packagist) — https://packagist.org/packages/aliziodev/laravel-next-starter-kit 
- mmedia/laravel-spa (Packagist) — https://packagist.org/packages/mmedia/laravel-spa 
- Update object storage bucket (Laravel Cloud) — https://laravel.com/cloud/docs/api/object-storage-buckets/update-object-storage-bucket 