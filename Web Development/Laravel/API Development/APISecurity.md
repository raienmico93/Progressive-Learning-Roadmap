# Laravel API Security & Architecture — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel API Security & Architecture encompasses the framework's integrated systems for verifying user identity (authentication), controlling what authenticated users can do (authorization), protecting tokens and sessions from abuse, preventing common injection and mass-assignment vulnerabilities, and enforcing strict input validation — all working together to build secure, production-ready APIs.

**Technical Definition:** Laravel's API security architecture is built on four pillars: (1) the authentication guard system (`config/auth.php`) with Sanctum for API tokens and SPA sessions, (2) the authorization system comprising Gates (closure-based) and Policies (class-based, model-oriented) that centralize access rules, (3) middleware-based protections including CSRF verification, rate limiting via the `RateLimiter` facade and `throttle` middleware, and (4) Eloquent's `$fillable`/`$guarded` mass-assignment protection combined with FormRequest validation and sanitization to secure the database layer against injection and tampering.

**Beginner-Friendly Explanation:** Building an API is not just about making endpoints that work — it is about making sure the right people can do the right things, and the wrong people cannot do anything harmful. Think of it like securing a building: **authentication** is the front door (proving who you are), **authorization** is the keycard system (which rooms you can enter), **token management** is issuing and revoking keys, **CSRF protection** prevents someone from forging your signature, **rate limiting** stops people from trying the door a million times, **mass assignment protection** stops people from slipping extra instructions into a form, and **input validation** checks that everything coming through the door is exactly what it claims to be.

### Key Characteristics

- **Separation of concerns:** Authentication (who you are) is handled by Sanctum; authorization (what you can do) is handled by Gates and Policies. Keeping these separate prevents security gaps.
- **Defense in depth:** Multiple layers protect the API — CSRF middleware, rate limiting, input validation, mass-assignment guards, and token abilities all work together.
- **Centralized authorization:** Policies and Gates put all access rules in dedicated classes, preventing scattered permission checks in controllers.
- **Token-centric API security:** Sanctum provides both cookie-based SPA authentication and Bearer token authentication, with abilities (scopes) and expiration/pruning.
- **Database-layer protection:** Eloquent's `$fillable` and `$guarded` properties prevent mass-assignment attacks, while query builder parameter binding prevents SQL injection.
- **Validation as a security boundary:** FormRequests enforce type checking, sanitization, and business rules before any data reaches the database.

### Prerequisites

- PHP 8.1+ (Laravel 10+) or PHP 8.2+ (Laravel 11+).
- Composer dependency manager.
- A Laravel application with `routes/api.php` configured (via `php artisan install:api`).
- Basic understanding of HTTP, REST, and Eloquent models.
- Familiarity with Laravel's service container and middleware.

### Related Programming Areas

- **Laravel Sanctum** — Token and SPA authentication.
- **Laravel Gates & Policies** — Authorization for model-specific and general actions.
- **Eloquent ORM** — Mass-assignment protection and safe query building.
- **FormRequest Validation** — Input validation and sanitization.
- **Middleware** — CSRF verification, rate limiting, and authentication guards.
- **RateLimiter Facade** — Dynamic, per-user, and per-IP rate limiting.

### Core Concepts / Features

1. **Authentication vs. Authorization:** Sanctum for authentication vs. Gates and Policies for authorization.
2. **Token Management:** Secure creation, revocation (individual and bulk), and token metadata.
3. **CSRF Protection:** How Sanctum handles CSRF for SPAs via `/sanctum/csrf-cookie`.
4. **Rate Limiting:** Custom rate limiters using the `RateLimiter` facade and `throttle` middleware.
5. **Mass Assignment & Injection:** Protecting the database layer using `$guarded`/`$fillable` and safe query building.
6. **Input Validation & Sanitization:** Using FormRequests to enforce strict type checking and data sanitization.

---

## 1. Authentication vs. Authorization

### Definitions

**Core Definition:** Authentication is the process of verifying a user's identity, while authorization is the process of determining what actions an authenticated user is permitted to perform.

**Technical Definition:** In Laravel, authentication is handled by guards defined in `config/auth.php`, with Sanctum providing lightweight token-based authentication for SPAs and mobile apps, and session-based authentication for web applications. Authorization is handled by two complementary systems: Gates (closure-based checks for simple, general actions) and Policies (class-based, model-oriented rules for resource-specific operations). Policies are automatically discovered when following standard naming conventions (e.g., `PostPolicy` for the `Post` model).

**Beginner-Friendly Explanation:** Authentication answers the question "Who are you?" — it is like showing your ID at the door. Authorization answers the question "What are you allowed to do?" — it is like the keycard system that determines which rooms you can enter. Laravel makes both easy: Sanctum handles authentication, and Gates/Policies handle authorization. Keeping them separate is critical because knowing who someone is does not mean they should have access to everything.

### Purposes

- To verify user identity through multiple authentication mechanisms (tokens, sessions, OAuth2) while maintaining a consistent guard interface.
- To centralize authorization logic in dedicated Policy classes, preventing scattered permission checks in controllers.
- To provide simple, closure-based authorization checks for actions that are not tied to a specific model (Gates).
- To enable model-specific authorization through automatic policy discovery and method resolution.
- To enforce the principle of least privilege by requiring explicit authorization for every sensitive operation.

### Syntax Rules and Structure

#### Complete General Syntax (Authentication with Sanctum)

```php
// 1. Protect routes with the auth:sanctum middleware
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::apiResource('posts', PostController::class);
});

// 2. Access the authenticated user
$user = $request->user();
```

#### Complete General Syntax (Authorization with Gates)

```php
// Define a gate in AppServiceProvider::boot()
use Illuminate\Support\Facades\Gate;

Gate::define('update-post', function (User $user, Post $post) {
    return $user->id === $post->user_id;
});

// Check the gate in a controller
if (Gate::allows('update-post', $post)) {
    // Allow the action
}

// Check via the user model
if ($request->user()->can('update-post', $post)) {
    // Allow the action
}
```

#### Complete General Syntax (Authorization with Policies)

```php
// Generate a policy
// php artisan make:policy PostPolicy --model=Post

// File: app/Policies/PostPolicy.php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }
}

// Use the policy in a controller
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post); // Throws 403 if not authorized
    // ...
}
```

**Component Breakdown:**

- `auth:sanctum` middleware — Authenticates the request via Sanctum. Returns 401 if unauthenticated.
- `Gate::define('name', closure)` — Defines a gate with a closure that receives the authenticated user and optional additional arguments.
- `Gate::allows('name', $model)` — Returns `true` if the gate permits the action.
- `$this->authorize('method', $model)` — Calls the corresponding policy method. Throws `AuthorizationException` (403) if denied.
- Policy methods receive the authenticated `User` as the first argument and the model instance as the second.

**Syntax Rules:**

- Policies are auto-discovered if named `{Model}Policy` in `app/Policies/`. Otherwise, register them in `AuthServiceProvider` or `AppServiceProvider`.
- The `authorize()` method in controllers is available when the controller extends `Illuminate\Routing\Controller` (Laravel 9+) or uses the `AuthorizesRequests` trait.
- Gates and Policies can both be checked via `$user->can()` or `$user->cannot()`.
- Middleware-based authorization can be applied via `->middleware('can:update,post')` on routes.

**Constraints and Limitations:**

- **Sanctum's `tokenCan()` always returns `true` for session-authenticated SPA requests.** Token abilities cannot be enforced for cookie-based SPA authentication.
- **Gates are not model-specific.** If your authorization logic depends on a model instance, use a Policy instead.
- **Policy auto-discovery requires the model and policy to be in the same namespace hierarchy.** If your models are in a different namespace, manual registration is required.
- **Authorization checks in controllers are not sufficient alone.** Middleware-based authorization should also be applied to routes for defense in depth.

### Annotated Code Examples

**Example 1: Combining Sanctum Authentication with Policy Authorization**

```php
<?php
// File: app/Policies/PostPolicy.php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    // Any authenticated user can view published posts
    public function view(User $user, Post $post): bool
    {
        return $post->status === 'published' || $user->id === $post->user_id;
    }

    // Only the author can update their own post
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    // Only the author or an admin can delete a post
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }
}
```

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
    public function update(Request $request, Post $post): JsonResponse
    {
        // Step 1: Authenticate (handled by auth:sanctum middleware)
        // Step 2: Authorize (via policy)
        $this->authorize('update', $post);

        $validated = $request->validate([
            'title' => 'sometimes|string|max:255',
            'body'  => 'sometimes|string',
        ]);

        $post->update($validated);

        return response()->json($post);
    }

    public function destroy(Post $post): JsonResponse
    {
        $this->authorize('delete', $post);
        $post->delete();
        return response()->json(null, 204);
    }
}
```

```php
// File: routes/api.php

Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('posts', PostController::class);
});
```

**Step-by-Step Setup:**

1. Create the `PostPolicy` with `php artisan make:policy PostPolicy --model=Post`.
2. Define the policy methods (`view`, `update`, `delete`).
3. Call `$this->authorize()` in controller methods.
4. Protect all routes with `auth:sanctum`.

**Expected Output:**

- `PUT /api/posts/1` as the post's author → `200 OK` with the updated post.
- `PUT /api/posts/1` as a different user → `403 Forbidden`.
- `DELETE /api/posts/1` as the post's author or an admin → `204 No Content`.
- `DELETE /api/posts/1` as a non-owner, non-admin → `403 Forbidden`.

**Why This Output Occurs:** The `auth:sanctum` middleware authenticates the request first. Then, the `authorize('update', $post)` call resolves the `PostPolicy` (via auto-discovery), invokes the `update` method with the authenticated user and the post, and returns `true` or `false`. If `false`, Laravel throws an `AuthorizationException`, which the exception handler renders as a `403 Forbidden` JSON response.

### Real-World Cases

- **Multi-tenant SaaS:** Users can only view and modify resources belonging to their own tenant; policies check the tenant ID on every model.
- **Content management systems:** Authors can edit their own posts, editors can edit any post, and admins can delete posts — all enforced by policy methods.
- **E-commerce platforms:** Customers can view their own orders; administrators can view all orders and update order status.
- **Team collaboration tools:** Project members can update tasks assigned to them; project owners can manage all tasks.

---

## 2. Token Management

### Definitions

**Core Definition:** Token management is the process of securely creating, storing, revoking, and monitoring API tokens, including managing token abilities (scopes) and metadata.

**Technical Definition:** Sanctum tokens are stored in the `personal_access_tokens` table with the token hashed using SHA-256. Tokens are created via `createToken($name, $abilities, $expiresAt)`, which returns a `NewAccessToken` instance; the `plainTextToken` property contains the only plain-text representation of the token. Tokens can be revoked individually via `$token->delete()` or `currentAccessToken()->delete()`, or in bulk via `$user->tokens()->delete()`. Token abilities are stored as a JSON array in the `abilities` column, and are checked via `tokenCan()` or enforced structurally via the `abilities` and `ability` middleware.

**Beginner-Friendly Explanation:** Token management is like managing the keys to your house. You can create a key for your dog walker (with limited access to only the front door), a key for your cleaner (with access to all rooms except your office), and a key for yourself (with full access). If you lose a key, you revoke it — it stops working immediately. If your cleaner quits, you revoke their key without affecting anyone else's. Sanctum gives you this exact control: create tokens with specific abilities, revoke individual tokens or all tokens at once, and monitor when tokens were last used.

### Purposes

- To issue API tokens with specific abilities (scopes) that limit what each token can do, following the principle of least privilege.
- To revoke individual tokens when a device is lost or an integration is discontinued, without affecting other tokens.
- To revoke all tokens at once when a user's credentials are compromised or their account is suspended.
- To monitor token usage via the `last_used_at` timestamp, enabling identification of unused or suspicious tokens.
- To set token expiration periods that automatically invalidate tokens after a specified duration, reducing the impact of leaked tokens.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Creating a token with abilities
$token = $user->createToken('device-name', ['post:read', 'post:create'])->plainTextToken;

// Creating a token with expiration
$token = $user->createToken('device-name', ['*'], now()->addDays(30))->plainTextToken;

// Listing a user's tokens
$tokens = $user->tokens; // Collection of PersonalAccessToken models

// Revoking a specific token
$user->tokens()->where('id', $tokenId)->delete();

// Revoking the current token
$request->user()->currentAccessToken()->delete();

// Revoking all tokens
$user->tokens()->delete();

// Checking abilities
if ($request->user()->tokenCan('post:create')) {
    // Allow the action
}

// Enforcing abilities via middleware
Route::middleware(['auth:sanctum', 'abilities:post:create'])->post('/posts', ...);
Route::middleware(['auth:sanctum', 'ability:post:update,post:delete'])->put('/posts/{post}', ...);
```

**Component Breakdown:**

- `createToken($name, $abilities, $expiresAt)` — Creates a new token. The `$name` is a human-readable identifier (e.g., "iPhone 15"). The `$abilities` array limits what the token can do. The `$expiresAt` sets an optional expiration.
- `$user->tokens` — Eloquent relationship returning all tokens for the user.
- `$request->user()->currentAccessToken()` — Returns the token used to authenticate the current request.
- `tokenCan('ability')` — Returns `true` if the current token has the specified ability or the wildcard `*`.
- `abilities:ability1,ability2` middleware — Requires the token to have **all** listed abilities.
- `ability:ability1,ability2` middleware — Requires the token to have **at least one** of the listed abilities.

**Syntax Rules:**

- The `HasApiTokens` trait **must** be added to the `User` model for `createToken()` and `tokens()` to work.
- Token abilities are stored as a JSON array in the `abilities` column of the `personal_access_tokens` table.
- The default abilities array is `['*']` (all abilities) if none is specified.
- Tokens are hashed with SHA-256 before storage; the plain-text token is returned only once.

**Constraints and Limitations:**

- **Revoked tokens cannot be un-revoked.** Once deleted from the database, the token is permanently invalid.
- **Token abilities are not a replacement for Policies.** Use them together: abilities restrict what the token can do, policies restrict what the user can do.
- **`tokenCan()` always returns `true` for SPA session authentication.** Token abilities are only meaningful for token-based requests.
- **Global expiration in `config/sanctum.php` overrides per-token expiration.** If a global expiration is set, it takes precedence over the `$expiresAt` parameter.

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
    // GET /api/tokens — List all tokens
    public function index(Request $request): JsonResponse
    {
        return response()->json($request->user()->tokens);
    }

    // POST /api/tokens — Create a new token
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'        => 'required|string|max:255',
            'abilities'   => 'sometimes|array',
            'abilities.*' => 'string|in:post:read,post:create,post:update,post:delete',
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

    // DELETE /api/tokens/{id} — Revoke a specific token
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

1. Define routes: `Route::middleware('auth:sanctum')->group(function () { Route::apiResource('tokens', TokenController::class)->except(['show', 'update']); });`
2. Create a token: `POST /api/tokens` with `{"name": "CI Pipeline", "abilities": ["post:read"], "expires_in_days": 90}`.
3. Use the token to access a protected route.
4. Revoke the token: `DELETE /api/tokens/{id}`.

**Expected Output (Token Creation):**

```json
{
    "token": "3|xyz789abc123...",
    "name": "CI Pipeline",
    "abilities": ["post:read"],
    "expires_at": "2025-09-15T10:00:00+00:00"
}
```

**Expected Output (Token Revocation):**

- `DELETE /api/tokens/1` → `204 No Content`.
- Subsequent requests with the revoked token → `401 Unauthenticated`.

**Why This Output Occurs:** The `createToken()` method stores the token with the specified abilities and expiration in the `personal_access_tokens` table. The `destroy()` method deletes the token record, immediately invalidating the token. The `destroyAll()` method revokes all tokens, useful when a user's account is compromised.

### Real-World Cases

- **Mobile app with multiple devices:** A user creates a separate token for each device (iPhone, iPad, Android), allowing them to revoke access for a lost device without affecting others.
- **CI/CD pipelines:** A developer creates a token with only the `deploy:write` ability, scoped to their repository, and uses it in a GitHub Actions workflow.
- **Third-party integrations:** An external service is given a token with only `read:orders` ability, preventing it from modifying or deleting orders.
- **Account security:** When a user changes their password, all existing tokens are revoked automatically, forcing all devices to re-authenticate.

---

## 3. CSRF Protection

### Definitions

**Core Definition:** CSRF (Cross-Site Request Forgery) protection is a security mechanism that prevents malicious websites from making unauthorized requests on behalf of an authenticated user by requiring a secret token that only the legitimate frontend possesses.

**Technical Definition:** For Sanctum SPA authentication, the frontend must first request a CSRF cookie from the `/sanctum/csrf-cookie` endpoint. This endpoint sets an `XSRF-TOKEN` cookie containing the current CSRF token. Axios (and similar HTTP clients) automatically read this cookie and include it as the `X-XSRF-TOKEN` header in subsequent requests. Laravel's `VerifyCsrfToken` middleware validates this header against the session's token before allowing the request to proceed. For API token authentication (Bearer tokens), CSRF protection is not required because tokens are not automatically sent by the browser.

**Beginner-Friendly Explanation:** CSRF is like someone forging your signature on a cheque. Imagine you are logged into your bank's website, and you visit a malicious site. That site could secretly submit a form to your bank's website, and because your browser automatically includes your login cookie, the bank might think the request came from you. CSRF protection prevents this by requiring a secret token that only your legitimate bank website knows. Sanctum handles this for SPAs: before making any authenticated request, your SPA asks Laravel for a CSRF cookie, and then includes that token in every subsequent request. The malicious site does not have access to this token, so its forged requests are rejected.

### Purposes

- To prevent malicious third-party websites from making authenticated requests on behalf of a user who is logged into the SPA.
- To ensure that state-changing requests (POST, PUT, PATCH, DELETE) originate from the legitimate SPA frontend, not from a forged source.
- To provide a standard, browser-compatible mechanism for CSRF protection that works seamlessly with Axios and other HTTP clients.
- To allow API token authentication (Bearer tokens) to bypass CSRF protection, since tokens are not automatically sent by the browser and are therefore not vulnerable to CSRF.
- To integrate CSRF protection with Laravel's session management, ensuring that the CSRF token is tied to the authenticated session.

### Syntax Rules and Structure

#### Complete General Syntax

```env
# .env — Ensure stateful domains are configured
SANCTUM_STATEFUL_DOMAINS=localhost:3000,app.example.com
SESSION_DOMAIN=.example.com
```

```php
// File: config/cors.php — Ensure CSRF cookie path is included
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie', 'login', 'logout'],
    'supports_credentials' => true,
    'allowed_origins' => ['http://localhost:3000', 'https://app.example.com'],
];
```

```javascript
// File: SPA frontend — Axios configuration
import axios from 'axios';

axios.defaults.withCredentials = true;

// Step 1: Request the CSRF cookie before any authenticated request
await axios.get('/sanctum/csrf-cookie');

// Step 2: Log in (Axios automatically includes the X-XSRF-TOKEN header)
await axios.post('/login', { email, password });

// Step 3: Make authenticated requests (CSRF token is automatically included)
const user = await axios.get('/api/user');
```

**Component Breakdown:**

- `/sanctum/csrf-cookie` — Sanctum's built-in endpoint that sets the `XSRF-TOKEN` cookie containing the CSRF token.
- `XSRF-TOKEN` — The cookie name that Axios reads automatically. Axios then sends the value as the `X-XSRF-TOKEN` header.
- `VerifyCsrfToken` middleware — Laravel's middleware that validates the `X-XSRF-TOKEN` header against the session token. Applied automatically to stateful API requests.
- `withCredentials: true` — Axios setting that ensures cookies (including the CSRF cookie) are sent with every request.
- `SANCTUM_STATEFUL_DOMAINS` — Must include the SPA's origin for the CSRF cookie to be set.

**Syntax Rules:**

- The `/sanctum/csrf-cookie` endpoint **must** be called before any state-changing request (POST, PUT, PATCH, DELETE).
- The `paths` array in `config/cors.php` **must** include `sanctum/csrf-cookie`.
- Axios automatically reads the `XSRF-TOKEN` cookie and sends it as the `X-XSRF-TOKEN` header. For other HTTP clients (fetch, custom), the header must be set manually.
- CSRF protection applies only to stateful (cookie-based) requests. Token-based requests (Bearer tokens) bypass CSRF verification.

**Constraints and Limitations:**

- **CSRF protection is only relevant for SPA authentication.** For token-based API authentication, CSRF is not applicable because the browser does not automatically include Bearer tokens.
- **The `/sanctum/csrf-cookie` endpoint must be accessible from the SPA's origin.** CORS must be configured with `supports_credentials => true` and explicit allowed origins.
- **The CSRF cookie is scoped to the session domain.** If `SESSION_DOMAIN` is not set correctly, the cookie may not be sent with requests to the API subdomain.
- **Using `SameSite=Strict` or `SameSite=Lax` may prevent the CSRF cookie from being sent cross-origin.** For cross-subdomain SPA authentication, use `SameSite=None` and `Secure=true` (requires HTTPS).

### Annotated Code Examples

**Example 1: Complete SPA CSRF Flow**

```javascript
// File: lib/axios.js — Axios configuration for CSRF
import axios from 'axios';

const api = axios.create({
    baseURL: process.env.NEXT_PUBLIC_API_URL, // e.g., http://localhost:8000
    withCredentials: true, // Required: send cookies with every request
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
    },
});

export default api;
```

```javascript
// File: pages/login.js — Complete login flow with CSRF
import api from '../lib/axios';
import { useState } from 'react';

export default function Login() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();

        // Step 1: Get the CSRF cookie
        await api.get('/sanctum/csrf-cookie');

        // Step 2: Log in (Axios automatically includes X-XSRF-TOKEN header)
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

```php
// File: routes/web.php — Login route with session middleware
Route::post('/login', [AuthController::class, 'login']);
```

**Step-by-Step Setup:**

1. Configure `SANCTUM_STATEFUL_DOMAINS` and `SESSION_DOMAIN` in `.env`.
2. Ensure `config/cors.php` includes `sanctum/csrf-cookie` in `paths` and `supports_credentials => true`.
3. In the SPA, configure Axios with `withCredentials: true`.
4. Call `GET /sanctum/csrf-cookie` before login.
5. Make authenticated requests — Axios includes the CSRF token automatically.

**Expected Output:**

- `GET /sanctum/csrf-cookie` → `204 No Content` with `Set-Cookie: XSRF-TOKEN=...`.
- `POST /login` → `200 OK` with `Set-Cookie: laravel_session=...`.
- `GET /api/user` → `200 OK` with the authenticated user's JSON.

**Why This Output Occurs:** The `/sanctum/csrf-cookie` endpoint sets the `XSRF-TOKEN` cookie. Axios reads this cookie and sends it as the `X-XSRF-TOKEN` header in subsequent requests. Laravel's `VerifyCsrfToken` middleware validates this header against the session's token. If the token is missing or invalid, the middleware returns a `419 Page Expired` error. If valid, the request proceeds.

### Real-World Cases

- **First-party SPAs (Vue, React, Next.js):** SPAs hosted on the same top-level domain as the API use Sanctum SPA authentication with CSRF protection.
- **Next.js on Vercel with separate API:** Frontend on `myapp.vercel.app` and API on `api.example.com` — CSRF protection works when `SANCTUM_STATEFUL_DOMAINS` and CORS are configured correctly.
- **Multi-tenant SaaS:** Each tenant subdomain (`tenant1.example.com`, `tenant2.example.com`) shares the same API and uses CSRF protection via shared session cookies.
- **Enterprise applications:** Compliance requirements often mandate CSRF protection for all state-changing requests; Sanctum provides this out of the box.

---

## 4. Rate Limiting

### Definitions

**Core Definition:** Rate limiting is the practice of restricting the number of requests a client can make to an API within a specified time window, protecting the server from abuse, overload, and brute-force attacks.

**Technical Definition:** Laravel's rate limiting is implemented through the `RateLimiter` facade and the `throttle` middleware. Named rate limiters are defined in `AppServiceProvider::boot()` using `RateLimiter::for('name', function (Request $request) { return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip()); })`. The `throttle` middleware accepts either a named limiter (`throttle:api`) or an inline limit (`throttle:60,1` for 60 requests per 1 minute). Laravel uses a token bucket algorithm: each client has a bucket of attempts that refills over time; when the bucket is empty, requests are rejected with a `429 Too Many Requests` status code and a `Retry-After` header.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer at a club who lets in a certain number of people per hour. If you try to enter too many times, the bouncer says "wait 30 seconds and try again." The `Retry-After` header is that "wait 30 seconds" message. Laravel makes it easy to set different limits for different routes: a login endpoint might allow only 5 attempts per minute, while a data retrieval endpoint might allow 60 requests per minute. You can also set different limits for different users — premium users get more requests than free users.

### Purposes

- To protect the API from denial-of-service (DoS) attacks and brute-force login attempts.
- To ensure fair resource allocation by preventing any single client from monopolising server capacity.
- To enforce tiered access (e.g., free users vs. premium users) with different rate limits.
- To reduce infrastructure costs by limiting excessive API usage.
- To provide clients with clear, actionable information about when they can retry via the `Retry-After` header.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Basic API rate limiter — 60 requests per minute per user/IP
        RateLimiter::for('api', function (Request $request) {
            return Limit::perMinute(60)->by(
                $request->user()?->id ?: $request->ip()
            );
        });

        // Login rate limiter — 5 attempts per minute per email+IP
        RateLimiter::for('login', function (Request $request) {
            return [
                Limit::perMinute(5)->by($request->input('email') . '|' . $request->ip()),
                Limit::perMinute(20)->by($request->ip()),
            ];
        });

        // Tiered rate limiter — premium users get 1,000 requests per minute
        RateLimiter::for('premium-api', function (Request $request) {
            if ($request->user()?->isPremium()) {
                return Limit::perMinute(1000)->by($request->user()->id);
            }
            return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
        });

        // Custom response with Retry-After header
        RateLimiter::for('custom', function (Request $request) {
            return Limit::perMinute(10)
                ->by($request->ip())
                ->response(function (Request $request, array $headers) {
                    return response()->json([
                        'error' => 'Rate limit exceeded.',
                        'retry_after' => $headers['Retry-After'] ?? null,
                    ], 429, $headers);
                });
        });
    }
}
```

```php
// File: routes/api.php — Apply rate limiters to routes

Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('posts', PostController::class);
});

Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:login');
```

**Component Breakdown:**

- `RateLimiter::for('name', closure)` — Defines a named rate limiter.
- `Limit::perMinute(60)` — Creates a limit of 60 requests per minute.
- `->by($request->user()?->id ?: $request->ip())` — Segments the limit by user ID (if authenticated) or IP address (if guest).
- `->response(function (Request $request, array $headers) { ... })` — Customises the 429 response. The `$headers` array contains `Retry-After`, `X-RateLimit-Limit`, and `X-RateLimit-Remaining`.
- `throttle:api` — Applies the `api` rate limiter to the route or group.
- `throttle:60,1` — Inline limiter: 60 requests per 1 minute.

**Syntax Rules:**

- Named rate limiters are defined in a service provider's `boot()` method.
- The `by()` method segments the limit. Without it, the limit is applied globally.
- Multiple limits can be returned as an array; the most restrictive limit applies.
- Rate limiting requires a cache store (Redis recommended for distributed applications).

**Constraints and Limitations:**

- **Rate limiting counters are stored in the cache.** If the cache is cleared, all limits are reset.
- **The `Retry-After` header value is in seconds.** Clients should parse it as an integer and wait that many seconds before retrying.
- **Custom responses must return a response with a 429 status code.** Returning a different status code may confuse clients.
- **Rate limiters defined in a service provider are resolved at boot time.** Dynamic limits based on request attributes are evaluated per request.

### Annotated Code Examples

**Example 1: Tiered Rate Limiting with Custom 429 Response**

```php
<?php
// File: app/Providers/AppServiceProvider.php

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
            $user = $request->user();

            // Premium users: 1,000 requests per minute
            if ($user && $user->isPremium()) {
                return Limit::perMinute(1000)
                    ->by($user->id)
                    ->response(function (Request $request, array $headers) {
                        return response()->json([
                            'error'   => 'Rate limit exceeded.',
                            'message' => 'You have exceeded your premium rate limit of 1,000 requests per minute.',
                            'retry_after_seconds' => $headers['Retry-After'] ?? null,
                        ], 429, $headers);
                    });
            }

            // Free users: 60 requests per minute
            if ($user) {
                return Limit::perMinute(60)
                    ->by($user->id)
                    ->response(function (Request $request, array $headers) {
                        return response()->json([
                            'error'   => 'Rate limit exceeded.',
                            'message' => 'Free tier limit of 60 requests per minute exceeded. Upgrade to premium for higher limits.',
                            'upgrade_url' => route('premium.upgrade'),
                            'retry_after_seconds' => $headers['Retry-After'] ?? null,
                        ], 429, $headers);
                    });
            }

            // Unauthenticated: 20 requests per minute per IP
            return Limit::perMinute(20)
                ->by($request->ip())
                ->response(function (Request $request, array $headers) {
                    return response()->json([
                        'error'   => 'Rate limit exceeded.',
                        'message' => 'Unauthenticated rate limit of 20 requests per minute exceeded.',
                        'retry_after_seconds' => $headers['Retry-After'] ?? null,
                    ], 429, $headers);
                });
        });
    }
}
```

**Expected Output (Premium User Exceeding Limit):**

```
HTTP/1.1 429 Too Many Requests
Retry-After: 12
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
Content-Type: application/json

{
    "error": "Rate limit exceeded.",
    "message": "You have exceeded your premium rate limit of 1,000 requests per minute.",
    "retry_after_seconds": 12
}
```

**Expected Output (Free User Exceeding Limit):**

```json
{
    "error": "Rate limit exceeded.",
    "message": "Free tier limit of 60 requests per minute exceeded. Upgrade to premium for higher limits.",
    "upgrade_url": "https://example.com/premium/upgrade",
    "retry_after_seconds": 45
}
```

**Why This Output Occurs:** The rate limiter checks the authenticated user's tier and returns the corresponding `Limit` object. The `response()` method customises the 429 response with tier-specific messaging. The `Retry-After` header is automatically added by Laravel's `ThrottleRequests` middleware and is included in the `$headers` array passed to the response callback.

### Real-World Cases

- **Public API tiers:** Free users get 60 requests/minute, premium users get 1,000 requests/minute, enterprise users get custom limits.
- **Authentication endpoints:** Login and registration endpoints are throttled to 5–10 attempts per minute per IP to prevent brute-force attacks.
- **AI and ML APIs:** Expensive endpoints (e.g., image generation, large language model inference) are throttled aggressively to control costs.
- **Webhook delivery:** Outgoing webhooks are rate-limited to prevent overwhelming the receiving service.

---

## 5. Mass Assignment & Injection

### Definitions

**Core Definition:** Mass assignment is the process of assigning an array of user-supplied data directly to an Eloquent model's attributes; unprotected mass assignment allows attackers to set fields they should not have access to (e.g., `is_admin`, `role`).

**Technical Definition:** Eloquent provides two properties to control mass assignment: `$fillable` (an allow-list of attributes that may be mass-assigned) and `$guarded` (a deny-list of attributes that may not be mass-assigned). By default, all attributes are mass-assignable unless explicitly guarded. The dangerous pattern is `protected $guarded = []`, which disables mass-assignment protection entirely. For SQL injection prevention, Laravel's query builder uses PDO parameter binding automatically when using Eloquent and the query builder's fluent methods; raw expressions (`whereRaw`, `selectRaw`) must use parameter binding explicitly.

**Beginner-Friendly Explanation:** Imagine you have a form that lets users update their profile. The form only shows fields for `name` and `email`. But a clever attacker could modify the request to include an extra field like `is_admin=1`. If your model does not protect against mass assignment, Eloquent would happily set `is_admin` to `1`, giving the attacker admin access. The `$fillable` and `$guarded` properties prevent this by specifying exactly which fields can be updated. Think of it as a guest list: only people on the list can enter, and everyone else is turned away.

### Purposes

- To prevent attackers from setting sensitive fields (e.g., `is_admin`, `role`, `balance`) through mass-assignment vulnerabilities.
- To explicitly declare which model attributes are safe for mass assignment, making the application's security posture clear and auditable.
- To protect the database layer from SQL injection by using Eloquent's parameter binding instead of raw SQL string concatenation.
- To enforce a clear boundary between user input and trusted application logic.
- To comply with security best practices and regulatory requirements for data integrity.

### Syntax Rules and Structure

#### Complete General Syntax ($fillable Allow-List)

```php
<?php
// File: app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that are mass assignable.
     * Only these attributes can be set via create() or update().
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];
}
```

#### Complete General Syntax ($guarded Deny-List)

```php
<?php
// File: app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that are NOT mass assignable.
     * All other attributes are mass-assignable by default.
     */
    protected $guarded = [
        'id',
        'is_admin',
        'role',
        'email_verified_at',
    ];
}
```

#### Complete General Syntax (Safe Query Building)

```php
// SAFE: Eloquent uses parameter binding automatically
User::where('email', $request->email)->first();

// SAFE: Query builder uses parameter binding
DB::table('users')->where('email', $request->email)->first();

// UNSAFE: Raw SQL with string interpolation
DB::select("SELECT * FROM users WHERE email = '$request->email'");

// SAFE: Raw expression with parameter binding
DB::select('SELECT * FROM users WHERE email = ?', [$request->email]);

// SAFE: whereRaw with binding
User::whereRaw('email = ?', [$request->email])->first();
```

**Component Breakdown:**

- `$fillable` — An array of attribute names that **may** be mass-assigned. Any attribute not in this list is silently ignored during `create()` and `update()`.
- `$guarded` — An array of attribute names that **may not** be mass-assigned. All other attributes are assignable. `protected $guarded = []` disables protection entirely.
- Parameter binding — Laravel's query builder and Eloquent automatically bind parameters when using fluent methods (`where`, `insert`, `update`). Raw expressions require explicit binding (`?` placeholders or named bindings).

**Syntax Rules:**

- Use `$fillable` when you want to explicitly allow only certain fields (allow-list approach). This is the recommended approach for most applications.
- Use `$guarded` when you want to protect only sensitive fields (deny-list approach). This is useful for models with many fields where most are safe.
- **Never** use `protected $guarded = []` unless you are absolutely certain that all input is validated and trusted.
- Always use Eloquent or query builder fluent methods for database queries. Avoid raw SQL with string interpolation.
- When using `whereRaw`, `selectRaw`, `orderByRaw`, or `DB::raw`, always use parameter binding.

**Constraints and Limitations:**

- **`$fillable` and `$guarded` are not validation.** They prevent mass assignment of sensitive fields but do not validate data types or formats. Always use FormRequest validation alongside mass-assignment protection.
- **The `fill()` method respects `$fillable` and `$guarded`, but direct attribute assignment does not.** `$user->is_admin = true` bypasses mass-assignment protection entirely.
- **Relationship methods (`associate`, `sync`, etc.) are not subject to mass-assignment protection.** They must be validated separately.
- **Raw SQL with string interpolation is vulnerable to SQL injection** even if `$fillable` is configured correctly. Mass-assignment protection and SQL injection prevention are separate concerns.

### Annotated Code Examples

**Example 1: Protecting Against Mass Assignment**

```php
<?php
// File: app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    // Allow-list: only these fields can be mass-assigned
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    // These fields are never mass-assignable
    protected $guarded = [
        'id',
        'is_admin',
        'role',
        'email_verified_at',
    ];
}
```

```php
<?php
// File: app/Http/Controllers/Api/AuthController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function register(Request $request): JsonResponse
    {
        // Step 1: Validate the input
        $validated = $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users',
            'password' => 'required|string|min:8|confirmed',
        ]);

        // Step 2: Create the user using only validated data
        // is_admin and role are NOT in $fillable, so they cannot be set
        // even if the attacker includes them in the request.
        $user = User::create([
            'name'     => $validated['name'],
            'email'    => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        return response()->json($user, 201);
    }
}
```

**Step-by-Step Setup:**

1. Define `$fillable` and `$guarded` on the `User` model.
2. Validate input using a FormRequest or `$request->validate()`.
3. Use `User::create($validated)` to create the model.
4. Test: Send a registration request with `{"name": "Hacker", "email": "hacker@example.com", "password": "secret123", "is_admin": true}`.

**Expected Output:**

```json
{
    "id": 1,
    "name": "Hacker",
    "email": "hacker@example.com",
    "is_admin": 0,
    "role": "user"
}
```

**Why This Output Occurs:** The `is_admin` and `role` fields are in the `$guarded` array, so they are silently ignored during mass assignment. Even though the attacker sent `"is_admin": true`, the `User::create()` call does not set it. The user is created with the default `is_admin = 0` and `role = 'user'`. If `$fillable` had been used instead, the same result would occur because `is_admin` is not in the allow-list.

---

**Example 2: Preventing SQL Injection with Parameter Binding**

```php
<?php
// File: app/Http/Controllers/Api/SearchController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class SearchController extends Controller
{
    public function search(Request $request): JsonResponse
    {
        $request->validate([
            'email' => 'required|email',
            'role'  => 'sometimes|string|in:admin,user,editor',
        ]);

        // SAFE: Eloquent uses parameter binding automatically
        $query = User::where('email', $request->email);

        // SAFE: Additional where clause with parameter binding
        if ($request->has('role')) {
            $query->where('role', $request->role);
        }

        // SAFE: Raw expression with parameter binding
        if ($request->has('created_after')) {
            $query->whereRaw('created_at >= ?', [$request->created_after]);
        }

        $users = $query->get();

        return response()->json($users);
    }
}
```

**Expected Output:**

- `GET /api/search?email=alice@example.com` → `200 OK` with the matching user.
- `GET /api/search?email=alice@example.com&role=admin` → `200 OK` with matching users.
- `GET /api/search?email=alice@example.com&created_after=2025-01-01` → `200 OK` with matching users.

**Why This Output Occurs:** Eloquent's `where()` method automatically binds the value as a parameter, so malicious input like `' OR 1=1 --` is treated as a literal string, not SQL code. The `whereRaw()` method uses the `?` placeholder with an explicit binding array, providing the same protection for raw expressions.

### Real-World Cases

- **User registration:** `$fillable` prevents attackers from setting `is_admin` or `role` during registration.
- **Profile updates:** `$guarded` protects sensitive fields like `email_verified_at` or `balance` from being modified through mass assignment.
- **API search endpoints:** Parameter binding prevents SQL injection even when users search with malicious input.
- **Multi-tenant applications:** `$fillable` ensures that `tenant_id` cannot be set by the client, preventing cross-tenant data access.

---

## 6. Input Validation & Sanitization

### Definitions

**Core Definition:** Input validation is the process of verifying that incoming request data meets specified criteria (type, format, range, etc.), while sanitization is the process of cleaning or transforming data to remove potentially harmful content before it is stored or used.

**Technical Definition:** Laravel's FormRequest classes (generated via `php artisan make:request`) encapsulate validation rules in the `rules()` method and authorization checks in the `authorize()` method. When a FormRequest is type-hinted in a controller method, Laravel resolves it from the container, runs validation before the controller method executes, and throws a `ValidationException` on failure. For sanitization, packages like `arondeparon/laravel-request-sanitizer` provide a `SanitizesInputs` trait that allows developers to define sanitizers in a `$sanitizers` property, which are applied to the request data before validation.

**Beginner-Friendly Explanation:** Validation is like a quality inspector at a factory: it checks that every item coming in meets the specifications (is it a valid email? is the password long enough?). Sanitization is like a cleaning crew: it removes anything that should not be there (extra spaces, HTML tags, non-numeric characters). Laravel's FormRequest classes let you define both validation rules and sanitization in one place, keeping your controllers clean and your data safe.

### Purposes

- To ensure that all incoming data meets the application's expected format, type, and business rules before it is processed or stored.
- To sanitize user input by removing or transforming potentially harmful content (e.g., HTML tags, script tags, unnecessary whitespace).
- To centralize validation and sanitization logic in dedicated FormRequest classes, keeping controllers focused on application flow.
- To return standardized, structured validation errors to API clients with appropriate HTTP status codes (422 Unprocessable Entity).
- To prevent injection attacks, data corruption, and business logic errors by rejecting invalid input at the earliest possible stage.

### Syntax Rules and Structure

#### Complete General Syntax (FormRequest Validation)

```php
// Generate the FormRequest
// php artisan make:request StoreUserRequest

// File: app/Http/Requests/StoreUserRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // Or check user permissions
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => ['required', 'email', 'unique:users'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
            'age'      => ['sometimes', 'integer', 'min:18', 'max:120'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required' => 'Please provide your name.',
            'age.min'       => 'You must be at least 18 years old.',
        ];
    }
}
```

#### Complete General Syntax (Sanitization with arondeparon/laravel-request-sanitizer)

```php
// Installation: composer require arondeparon/laravel-request-sanitizer

// File: app/Http/Requests/StoreCustomerRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Arondeparon\LaravelRequestSanitizer\SanitizesInputs;
use Arondeparon\LaravelRequestSanitizer\Sanitizers\Trim;
use Arondeparon\LaravelRequestSanitizer\Sanitizers\Capitalize;
use Arondeparon\LaravelRequestSanitizer\Sanitizers\RemoveNonNumeric;
use Arondeparon\LaravelRequestSanitizer\Sanitizers\Lowercase;

class StoreCustomerRequest extends FormRequest
{
    use SanitizesInputs;

    protected $sanitizers = [
        'name'    => [Trim::class, Capitalize::class],
        'email'   => [Trim::class, Lowercase::class],
        'phone'   => [RemoveNonNumeric::class],
        'address' => [Trim::class],
    ];

    public function rules(): array
    {
        return [
            'name'  => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:customers'],
            'phone' => ['required', 'string', 'max:20'],
            'address' => ['sometimes', 'string', 'max:500'],
        ];
    }
}
```

**Component Breakdown:**

- `rules(): array` — Returns an array of validation rules keyed by field name. Rules can be strings with pipe separators or arrays.
- `authorize(): bool` — Returns `true` to allow the request; `false` throws an `AuthorizationException` (403).
- `messages(): array` — Optional. Overrides the default error messages for specific rules.
- `SanitizesInputs` trait — Provides the sanitization pipeline for FormRequests.
- `$sanitizers` property — Maps field names to arrays of sanitizer classes to apply before validation.
- `Trim::class` — Removes whitespace from both ends of a string.
- `Capitalize::class` — Capitalizes the first character of a string.
- `Lowercase::class` — Converts a string to lowercase.
- `RemoveNonNumeric::class` — Removes all non-numeric characters from a string.

**Syntax Rules:**

- FormRequests **must** be type-hinted in controller methods to be automatically resolved and validated.
- Sanitizers are applied **before** validation rules, ensuring that validation operates on clean data.
- Wildcard patterns (`users.*.email`) can be used to apply sanitizers to array or nested fields.
- Validation failures for API requests return `422 Unprocessable Entity` with a JSON body containing the errors.

**Constraints and Limitations:**

- **Sanitization is not a substitute for output escaping.** Storing sanitized data is fine, but output must still be escaped (e.g., using Blade's `{{ }}` syntax) to prevent XSS.
- **The `arondeparon/laravel-request-sanitizer` package is community-maintained.** Check compatibility with your Laravel version before use.
- **FormRequest validation runs before the controller method.** If you need to perform logic before validation, use middleware or `prepareForValidation()`.
- **Validation rules are evaluated in order.** If a `required` rule fails, subsequent rules for that field may not be evaluated.

### Annotated Code Examples

**Example 1: FormRequest with Validation and Sanitization**

```php
<?php
// File: app/Http/Requests/StoreUserRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Arondeparon\LaravelRequestSanitizer\SanitizesInputs;
use Arondeparon\LaravelRequestSanitizer\Sanitizers\Trim;
use Arondeparon\LaravelRequestSanitizer\Sanitizers\Capitalize;
use Arondeparon\LaravelRequestSanitizer\Sanitizers\Lowercase;

class StoreUserRequest extends FormRequest
{
    use SanitizesInputs;

    protected $sanitizers = [
        'name'  => [Trim::class, Capitalize::class],
        'email' => [Trim::class, Lowercase::class],
    ];

    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => ['required', 'email', 'unique:users'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
            'age'      => ['sometimes', 'integer', 'min:18', 'max:120'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required' => 'Please provide your name.',
            'age.min'       => 'You must be at least 18 years old.',
        ];
    }
}
```

```php
<?php
// File: app/Http/Controllers/Api/UserController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreUserRequest;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;

class UserController extends Controller
{
    public function store(StoreUserRequest $request): JsonResponse
    {
        // Validation and sanitization have already run.
        // $request->validated() contains only the data that passed validation.
        $user = User::create([
            'name'     => $request->validated('name'),
            'email'    => $request->validated('email'),
            'password' => Hash::make($request->validated('password')),
        ]);

        return response()->json($user, 201);
    }
}
```

**Step-by-Step Setup:**

1. Install the sanitizer package: `composer require arondeparon/laravel-request-sanitizer`.
2. Generate the FormRequest: `php artisan make:request StoreUserRequest`.
3. Define the `$sanitizers` property and `rules()` method.
4. Type-hint `StoreUserRequest` in the controller method.
5. Test with invalid data to see the validation errors.

**Expected Output (Validation Failure):**

```json
{
    "message": "Please provide your name. (and 1 more error)",
    "errors": {
        "name": ["Please provide your name."],
        "email": ["The email has already been taken."]
    }
}
```

**HTTP Status Code: 422 Unprocessable Entity**

**Expected Output (Sanitization in Action):**

```php
// Input:  {"name": "  john doe  ", "email": "JOHN@EXAMPLE.COM", "password": "secret123", "password_confirmation": "secret123"}
// After sanitization:
// name  => "John doe"  (trimmed and capitalized)
// email => "john@example.com" (trimmed and lowercased)
```

**Why This Output Occurs:** The `SanitizesInputs` trait applies the defined sanitizers to the request data before validation runs. The `Trim` sanitizer removes leading and trailing whitespace, and the `Capitalize` sanitizer capitalizes the first letter of the name. The `Lowercase` sanitizer converts the email to lowercase. Validation then runs on the clean data. If validation fails, Laravel returns a `422 Unprocessable Entity` response with the error messages defined in `messages()`.

### Real-World Cases

- **User registration:** Validation ensures email uniqueness, password strength, and required fields; sanitization trims whitespace and normalizes case.
- **E-commerce checkout:** Validation checks that quantities are positive integers, prices are numeric, and addresses are within allowed lengths; sanitization removes non-numeric characters from phone numbers.
- **Content management:** Validation ensures that titles and bodies do not exceed maximum lengths; sanitization trims whitespace and removes potentially harmful HTML tags.
- **API integrations:** Third-party API payloads are validated against strict schemas before being stored or processed, preventing data corruption and injection attacks.

---

## References

- Laravel Sanctum Documentation — https://laravel.com/docs/sanctum
- Laravel Authorization Documentation (Gates and Policies) — https://laravel.com/docs/authorization
- Laravel Authentication Documentation — https://laravel.com/docs/authentication
- Laravel Eloquent: Getting Started (Mass Assignment) — https://laravel.com/docs/eloquent
- Laravel Validation Documentation — https://laravel.com/docs/validation
- Laravel Rate Limiting Documentation — https://laravel.com/docs/routing#rate-limiting
- Laravel CSRF Protection Documentation — https://laravel.com/docs/csrf
- DeepWiki: Authentication & Authorization — https://deepwiki.com/laravel/docs/2.3-authentication-and-authorization
- Laravel Sanctum GitHub Repository — https://github.com/laravel/sanctum
- Laravel Sanctum: Token Abilities — https://laravel.com/docs/sanctum#token-abilities
- Laravel Sanctum: Revoking Tokens — https://laravel.com/docs/sanctum#revoking-tokens
- Laravel Sanctum: SPA Authentication — https://laravel.com/docs/sanctum#spa-authentication
- arondeparon/laravel-request-sanitizer — https://packagist.org/packages/arondeparon/laravel-request-sanitizer
- Laravel Rate Limiting Guide (OneUptime) — https://github.com/OneUptime/blog/blob/master/posts/2026-02-03-laravel-rate-limiting/README.md
- Laravel Mass Assignment Protection (Laravel Daily) — https://laraveldaily.com/lesson/laravel-eloquent/guarded-fillable
- PHP Laravel Security Best Practices Guide (Safeguard.sh) — https://safeguard.sh
- Laravel CSRF Protection for SPAs (Laracasts Discussion) — https://laracasts.com/discuss/channels/laravel/sanctum-spa-csrf
- Laravel FormRequest Validation (Laravel Documentation) — https://laravel.com/docs/validation#form-request-validation