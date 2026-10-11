# Laravel API Authentication: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel API Authentication is the framework's system for verifying the identity of clients (mobile apps, SPAs, third-party services) making requests to an API, using stateless token-based mechanisms rather than session cookies.

**Technical Definition**

Laravel provides two first-party packages for API authentication: **Laravel Sanctum** (lightweight token and SPA cookie authentication) and **Laravel Passport** (full OAuth2 server implementation). Sanctum stores hashed API tokens in a `personal_access_tokens` table and authenticates requests via the `Authorization: Bearer` header or stateful SPA cookies. Passport implements the OAuth2 specification using the League OAuth2 server, supporting authorization code, password, client credentials, and refresh token grants. Both packages integrate with the `auth` middleware system and Eloquent models.

**Beginner-Friendly Explanation**

API authentication is how your API knows who is making a request. Since APIs are stateless—meaning the server does not remember you between requests—you need to prove who you are every time. Token authentication solves this by giving you a special key (a token) after you log in. You include this key in every request, and Laravel checks it to confirm your identity. Sanctum is the simple option for your own apps; Passport is the complex option for letting other companies' apps access your API on behalf of users.

---

### Key Characteristics

- **Statelessness**: No server-side session state; each request carries the token that proves identity.
- **Two Packages**: Sanctum for first-party clients (SPAs, mobile apps); Passport for OAuth2 third-party integrations.
- **Token Hashing**: Sanctum stores tokens as SHA-256 hashes, so plain-text tokens are only shown once at creation.
- **Token Abilities (Scopes)**: Tokens can be restricted to specific abilities (e.g., `read`, `write`), enabling least-privilege access.
- **Token Expiration**: Tokens can be configured to expire after a set duration; Passport supports short-lived access tokens with refresh tokens.
- **SPA Cookie Authentication**: Sanctum can authenticate SPAs using session cookies with CSRF protection (more secure than storing tokens in localStorage).
- **OAuth2 Grant Types**: Passport supports authorization code (with PKCE), password, client credentials, and device authorization grants.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A configured database** for storing tokens.
- **The `HasApiTokens` trait** added to the `User` model.
- **Basic understanding of HTTP headers** and JSON.

---

### Related Programming Areas

- **Session Authentication**: Token authentication is the stateless counterpart for APIs.
- **OAuth2**: Passport implements the OAuth2 protocol for third-party integrations.
- **Middleware**: `auth:sanctum` and `auth:api` protect routes.
- **Rate Limiting**: `throttle` middleware protects authentication endpoints from abuse.
- **API Resources**: Transform model data into JSON responses for API consumers.

---

## Core Concepts

---

### 1. API Authentication

#### Definitions

**Core Definition**

API authentication is the process of verifying the identity of a client making an API request, typically through tokens rather than session cookies.

**Technical Definition**

Laravel API authentication uses guards configured in `config/auth.php`. The `sanctum` guard uses the `Sanctum` driver for token and SPA authentication; the `api` guard uses the `passport` driver for OAuth2. Middleware (`auth:sanctum`, `auth:api`) resolves the authenticated user from the token. Tokens are stored in the `personal_access_tokens` table (Sanctum) or Passport's OAuth2 tables.

**Beginner-Friendly Explanation**

API authentication is how your API knows who is calling it. Instead of logging in with a session, you get a token and include it in every request.

#### Purposes

- To verify the identity of API clients before granting access.
- To protect sensitive data and operations from unauthorized access.
- To enable stateless authentication suitable for distributed systems.
- To support different client types (mobile, SPA, third-party).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Protecting routes with Sanctum
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

// Protecting routes with Passport
Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `auth:sanctum` | Middleware for Sanctum token/SPA authentication. |
| `auth:api` | Middleware for Passport OAuth2 authentication. |
| `$request->user()` | Returns the authenticated user instance. |

**Syntax Rules**

- Sanctum is the default recommendation for most APIs.
- Passport is required only when OAuth2 is necessary.
- API routes are defined in `routes/api.php` and are stateless.

**Constraints and Limitations**

- Sanctum does not support OAuth2.
- Passport is more complex to set up than Sanctum.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Sanctum Route Protection**

**Route**

```php
// routes/api.php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Expected Output (with valid token)**

```json
{"id":1,"name":"John Doe","email":"john@example.com"}
```

**Expected Output (without token)**

```json
{"message":"Unauthenticated."}
```

**Why This Output Occurs**

The `auth:sanctum` middleware validates the Bearer token and resolves the user.

---

#### Real-World Cases with Explanation

**Case 1: Mobile App Authentication**

A mobile app authenticates users via email/password, receives a Sanctum token, stores it securely, and includes it in all API requests.

**Explanation**: Sanctum is the recommended solution for first-party mobile apps.

---

### 2. Sanctum

#### Definitions

**Core Definition**

Laravel Sanctum is a lightweight authentication package for SPAs, mobile applications, and simple token-based APIs.

**Technical Definition**

Sanctum provides two authentication mechanisms: (1) API tokens stored as SHA-256 hashes in `personal_access_tokens`, and (2) SPA cookie authentication using Laravel's session guard with CSRF protection. Tokens are created via `$user->createToken()` and authenticated via `auth:sanctum` middleware.

**Beginner-Friendly Explanation**

Sanctum is the simple way to add authentication to your API. It is perfect for your own mobile app or single-page application.

#### Purposes

- To provide simple API token authentication without OAuth2 complexity.
- To authenticate SPAs using session cookies with CSRF protection.
- To support mobile applications with long-lived personal access tokens.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Issuing a token
$token = $user->createToken('mobile-app')->plainTextToken;

// Protecting routes
Route::middleware('auth:sanctum')->get('/user', ...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `createToken($name)` | Issues a new token. |
| `plainTextToken` | The unhashed token string. |
| `auth:sanctum` | Middleware for token/SPA authentication. |

**Syntax Rules**

- The `HasApiTokens` trait must be added to the `User` model.
- Sanctum is installed via `php artisan install:api`.
- SPA authentication requires `SANCTUM_STATEFUL_DOMAINS` configuration.

**Constraints and Limitations**

- Sanctum does not support OAuth2.
- Tokens have no expiration by default.

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

The token is generated, hashed, and stored. The plain-text token is returned to the client.

---

#### Real-World Cases with Explanation

**Case 1: React SPA with Laravel API**

A React SPA uses Sanctum's SPA authentication (cookie-based) to communicate with a Laravel API. The SPA benefits from CSRF protection without managing tokens manually.

**Explanation**: Sanctum's SPA mode provides session-level security with an API-first architecture.

---

### 3. Passport

#### Definitions

**Core Definition**

Laravel Passport is a full OAuth2 server implementation for Laravel applications.

**Technical Definition**

Passport is built on the League OAuth2 server and provides routes for OAuth2 token issuance, middleware for token validation, and Eloquent models for OAuth2 clients and tokens. It supports authorization code, password, client credentials, and refresh token grants.

**Beginner-Friendly Explanation**

Passport is the full-featured OAuth2 solution. Use it when you need to let other applications access your API on behalf of users—like "Log in with Google."

#### Purposes

- To provide full OAuth2 server capabilities.
- To support third-party integrations and API ecosystems.
- To issue access tokens and refresh tokens with scopes.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan install:api --passport
```

```php
// User model
use Laravel\Passport\HasApiTokens;
use Laravel\Passport\Contracts\OAuthenticatable;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `install:api --passport` | Installs Passport and runs migrations. |
| `HasApiTokens` | Trait for the `User` model. |
| `OAuthenticatable` | Interface for the `User` model. |
| `auth:api` | Middleware for Passport authentication. |

**Syntax Rules**

- The `api` guard must use the `passport` driver.
- `passport:keys` generates encryption keys.
- Passport routes are registered at `/oauth/*`.

**Constraints and Limitations**

- Passport is more complex to set up than Sanctum.
- Not recommended for simple SPAs or mobile apps.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Passport Authorization Code Grant with PKCE**

```php
// Creating a PKCE client
php artisan passport:client --public
```

**Expected Output**

```
// Client ID generated for PKCE flow
```

**Why This Output Occurs**

The `--public` option creates a PKCE-enabled client without a client secret.

---

#### Real-World Cases with Explanation

**Case 1: Third-Party Developer Platform**

A platform allows external developers to build integrations using OAuth2. Passport provides the authorization server.

**Explanation**: Passport is required when OAuth2 compliance is necessary.

---

### 4. API Tokens

#### Definitions

**Core Definition**

An API token is an opaque string that a client presents to the server to authenticate a request.

**Technical Definition**

Sanctum tokens are stored in `personal_access_tokens` with a `tokenable` morph relationship, a hashed `token` value, a `name`, optional `abilities` (JSON array), and timestamps. Tokens are created via `$user->createToken($name, $abilities)` and returned as `plainTextToken`.

**Beginner-Friendly Explanation**

An API token is a long string that works like a password. You give it to your app, and it sends the token with every request.

#### Purposes

- To provide stateless authentication for API clients.
- To allow multiple tokens per user for different devices.
- To enable token revocation without affecting other tokens.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Issuing a token
$token = $user->createToken('token-name', ['ability1'])->plainTextToken;

// Using the token
// Authorization: Bearer <plainTextToken>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `createToken($name, $abilities)` | Creates a new token. |
| `plainTextToken` | The unhashed token string. |
| `Authorization: Bearer` | Header for sending the token. |

**Syntax Rules**

- Tokens are hashed with SHA-256 before storage.
- `plainTextToken` is only available immediately after creation.
- Tokens should be transmitted over HTTPS only.

**Constraints and Limitations**

- Plain-text tokens cannot be retrieved after creation.
- Default token expiration is null (no expiration).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Issuing a Token with Abilities**

```php
$token = $user->createToken('api-token', ['post:read', 'post:create'])->plainTextToken;
```

**Expected Output**

```json
{"token":"2|xyz789..."}
```

**Why This Output Occurs**

The token is created with the specified abilities, restricting what actions it can perform.

---

#### Real-World Cases with Explanation

**Case 1: Account Settings Token Management**

A SaaS application provides an "API Tokens" page where users generate, label, and revoke tokens.

**Explanation**: Token issuance is a core feature of any API-first application.

---

### 5. Token Abilities

#### Definitions

**Core Definition**

Token abilities (also called "scopes") are strings that define what actions a token is permitted to perform.

**Technical Definition**

Abilities are stored as a JSON array in the `abilities` column of `personal_access_tokens`. The `tokenCan()` and `tokenCant()` methods check if the current token has a given ability. The `abilities` middleware (`CheckAbilities`) and `ability` middleware (`CheckForAnyAbility`) enforce abilities at the route level.

**Beginner-Friendly Explanation**

Abilities let you say "this token can read data but not delete anything." They are like permissions attached to a specific key.

#### Purposes

- To restrict what actions a token can perform.
- To implement least-privilege access for API clients.
- To enforce authorization at the route level via middleware.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Issuing with abilities
$token = $user->createToken('token', ['server:read', 'server:update'])->plainTextToken;

// Checking abilities
if ($user->tokenCan('server:update')) {
    // Allowed
}
```

```php
// Route middleware
Route::middleware(['auth:sanctum', 'abilities:server:update'])->put('/server', ...);
Route::middleware(['auth:sanctum', 'ability:server:update,server:delete'])->delete('/server', ...);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `tokenCan($ability)` | Checks if the token has the ability. |
| `tokenCant($ability)` | Checks if the token lacks the ability. |
| `abilities:...` | Middleware requiring all listed abilities. |
| `ability:...` | Middleware requiring any listed ability. |

**Syntax Rules**

- `['*']` grants all abilities.
- `tokenCan()` always returns `true` for first-party SPA requests.
- Middleware aliases must be registered in `bootstrap/app.php` (Laravel 11+).

**Constraints and Limitations**

- Abilities are stored as JSON; very large arrays may impact performance.
- `tokenCan()` returns `false` if no token is present.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Checking Abilities in a Controller**

```php
public function update(Request $request)
{
    if ($request->user()->tokenCan('server:update')) {
        return response()->json(['status' => 'updated']);
    }

    return response()->json(['error' => 'Unauthorized'], 403);
}
```

**Expected Output**

```
// Token with server:update → 200 {"status":"updated"}
// Token without server:update → 403 {"error":"Unauthorized"}
```

**Why This Output Occurs**

`tokenCan()` checks the abilities array of the current access token.

---

#### Real-World Cases with Explanation

**Case 1: Read-Only API Keys**

A monitoring integration needs to read data but never modify it. A token with only `data:read` is created.

**Explanation**: Token abilities enforce least-privilege access.

---

### 6. Token Expiration

#### Definitions

**Core Definition**

Token expiration is the automatic invalidation of a token after a specified duration.

**Technical Definition**

In Sanctum, token expiration is configured via the `expiration` key in `config/sanctum.php` (in minutes). This overrides any `expires_at` value set on individual tokens. For Passport, token lifetimes are configured via `Passport::tokensExpireIn()` and `Passport::refreshTokensExpireIn()`.

**Beginner-Friendly Explanation**

Token expiration means your token stops working after a certain time, so you need to get a new one. This limits the damage if a token is stolen.

#### Purposes

- To limit the exposure window of compromised tokens.
- To enforce regular re-authentication for security.
- To support short-lived access tokens with refresh tokens.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/sanctum.php
'expiration' => 60 * 24 * 7, // 7 days
```

```php
// Passport configuration (AppServiceProvider)
use Laravel\Passport\Passport;

Passport::tokensExpireIn(now()->addDays(15));
Passport::refreshTokensExpireIn(now()->addDays(30));
```

**Component Breakdown**

| Setting | Description |
|---------|-------------|
| Sanctum `expiration` | Token lifetime in minutes. |
| `Passport::tokensExpireIn()` | Access token lifetime. |
| `Passport::refreshTokensExpireIn()` | Refresh token lifetime. |

**Syntax Rules**

- Sanctum's `expiration` overrides individual token `expires_at`.
- Default Sanctum expiration is `null` (tokens never expire).
- Passport's default access token lifetime is one year.

**Constraints and Limitations**

- Expired tokens are rejected by `auth:sanctum` middleware.
- `sanctum:prune-expired` can be scheduled to clean up expired tokens.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Sanctum Token Expiration**

```php
// config/sanctum.php
'expiration' => 60 * 24 * 30, // 30 days
```

**Expected Output**

```
// Tokens expire after 30 days
```

**Why This Output Occurs**

The `expiration` value is in minutes; setting it to `43200` (30 days) ensures tokens are automatically invalidated.

---

#### Real-World Cases with Explanation

**Case 1: Financial API Security**

A financial API uses short-lived access tokens (15 minutes) with refresh tokens (30 days).

**Explanation**: Short-lived tokens reduce the blast radius of a compromise.

---

### 7. Token Refresh

#### Definitions

**Core Definition**

Token refresh is the process of exchanging a refresh token for a new access token, typically used with OAuth2 (Passport) when access tokens are short-lived.

**Technical Definition**

Passport issues a `refresh_token` alongside the `access_token`. The client sends a POST request to `/oauth/token` with `grant_type=refresh_token` and the refresh token. The server validates the refresh token and returns a new access token (and optionally a new refresh token).

**Beginner-Friendly Explanation**

Token refresh lets you get a new access token without asking the user to log in again. The refresh token is like a longer-lived key that can be exchanged for a short-lived access token.

#### Purposes

- To maintain long-lived sessions without re-authentication.
- To support short-lived access tokens for security.
- To enable seamless token rotation.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Requesting a refresh token
$response = Http::asForm()->post('http://your-app.com/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => '',
]);
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `grant_type` | Must be `refresh_token`. |
| `refresh_token` | The refresh token issued with the access token. |
| `client_id` | The OAuth2 client ID. |
| `client_secret` | The OAuth2 client secret. |

**Syntax Rules**

- The `/oauth/token` route is provided by Passport.
- The response includes a new `access_token` and `refresh_token`.
- Refresh tokens can be revoked.

**Constraints and Limitations**

- Refresh tokens are only issued with authorization code and password grants.
- Sanctum does not support refresh tokens natively.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Refreshing a Passport Token**

```php
$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => '',
]);

return $response->json();
```

**Expected Output**

```json
{
  "token_type": "Bearer",
  "expires_in": 31536000,
  "access_token": "eyJ0eXAiOiJKV1Qi...",
  "refresh_token": "def50200..."
}
```

**Why This Output Occurs**

The refresh token is validated, and a new access token is issued.

---

#### Real-World Cases with Explanation

**Case 1: Mobile App with Short-Lived Tokens**

A mobile app uses 15-minute access tokens and 30-day refresh tokens. When the access token expires, the app silently refreshes it.

**Explanation**: Token refresh provides a seamless user experience while maintaining security.

---

### 8. API Authentication Best Practices

#### Definitions

**Core Definition**

API authentication best practices are the recommended guidelines for securely implementing and managing API authentication in Laravel.

**Technical Definition**

Best practices encompass package selection (Sanctum vs. Passport), token security (hashing, HTTPS, expiration), ability scoping, rate limiting, token revocation, and testing.

**Beginner-Friendly Explanation**

Best practices are the security rules that keep your API tokens safe from attackers.

#### Key Best Practices

1. **Choose the Right Package**: Use Sanctum for SPAs, mobile apps, and first-party APIs. Use Passport only when OAuth2 is required.

2. **Always Use HTTPS**: Tokens must be transmitted over TLS/SSL to prevent interception.

3. **Set Token Expiration**: Configure `expiration` in `config/sanctum.php` or `Passport::tokensExpireIn()`. Tokens should not live forever.

4. **Use Token Abilities**: Assign only the abilities a token needs. Avoid `['*']` unless absolutely necessary.

5. **Hash Tokens at Rest**: Sanctum already hashes tokens with SHA-256. Never store plain-text tokens.

6. **Revoke Tokens on Logout**: Implement logout endpoints that revoke the current token.

7. **Rate Limit Authentication Endpoints**: Apply `throttle:5,1` to login and token issuance routes.

8. **Use Separate Tokens per Device**: Each device or integration should have its own token for granular revocation.

9. **Monitor Token Usage**: Track token usage and revoke tokens that appear compromised.

10. **Test Token Authentication**: Write tests for token issuance, ability checks, and revocation.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Secure Token Issuance with Rate Limiting**

```php
Route::middleware('throttle:5,1')->post('/login', [AuthController::class, 'login']);
```

**Expected Output**

```
// After 5 failed login attempts, the client is blocked for 1 minute
```

**Why This Output Occurs**

The `throttle:5,1` middleware limits login attempts to prevent brute-force attacks.

---

**Example 2: Token Revocation on Logout**

```php
public function logout(Request $request)
{
    $request->user()->currentAccessToken()->delete();
    return response()->json(['message' => 'Logged out']);
}
```

**Expected Output**

```json
{"message":"Logged out"}
```

**Why This Output Occurs**

The current token is deleted from the database, invalidating it immediately.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise API Platform**

A large organization follows all best practices: Sanctum for internal apps, short-lived tokens, ability scoping, HTTPS, rate limiting, and comprehensive tests.

**Explanation**: Following best practices ensures the API is secure, scalable, and maintainable.

---

## References

- Laravel Sanctum – Laravel 13.x Documentation — https://laravel.com/docs/13.x/sanctum
- Laravel Passport – Laravel 12.x Documentation — https://laravel.com/docs/12.x/passport
- Laravel Sanctum vs Passport: Which One Should You Use? – Bagisto — https://bagisto.com/en/laravel-sanctum-vs-passport-which-one-should-you-use/
- Laravel Sanctum vs Passport 2026: A Simple Decision Tree – Laracopilot — https://laracopilot.com/blog/laravel-sanctum-vs-passport-auth-2026/
- Laravel Sanctum vs Passport 2026: API Authentication Guide – SharpSkill — https://sharpskill.dev/en/blog/laravel/laravel-sanctum-vs-passport-api-authentication
- Sanctum Token Abilities – Laravel Documentation (GitHub) — https://github.com/DevStorm-Team/laravel-book/blob/master/laravel-docs-master.pdf
- Passport Token Refresh – Laravel Documentation (GitHub) — https://github.com/Vectorial1024/docs/blob/12.x/passport.md
- Laravel API Authentication Best Practices – GitHub (voku/agent-skills) — https://github.com/voku/agent-skills/blob/main/skills/laravel-best-practices/rules/controller-api-resources.md

---

**Version Note**: This cheat sheet is based on Laravel 12.x through 13.x documentation. Sanctum is the recommended default for most API authentication needs. Passport is required only for OAuth2. The `install:api` command (Laravel 11+) simplifies installation. Token expiration and refresh token features are configured per package. Developers should always consult the documentation for their specific Laravel version.