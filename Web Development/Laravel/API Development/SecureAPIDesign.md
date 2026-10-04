# Laravel Secure API Design & Best Practices — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Secure API design in Laravel is the practice of applying defence-in-depth principles — least privilege, sensitive data masking, secure error handling, token rotation, audit logging, and transport-layer security — to protect API endpoints from unauthorized access, data leaks, credential theft, and man-in-the-middle attacks.

**Technical Definition:** Laravel's security architecture provides native mechanisms for implementing secure API design: Sanctum token abilities (scopes) for least-privilege access control, `APP_DEBUG=false` combined with custom exception rendering in `bootstrap/app.php` for masked error responses, Eloquent API Resources (`JsonResource`) for explicit attribute whitelisting, community packages such as `reiarseni/sanctum-refresh-token` for refresh token rotation with reuse detection (RFC 9700), `spatie/laravel-activitylog` for model-event audit trails, and security header middleware packages for HSTS, CSP, and HTTPS enforcement.

**Beginner-Friendly Explanation:** Building a secure API is like securing a building. You do not give everyone a master key — you give each person a key that opens only the doors they need (least privilege). You do not post the building's floor plan on the front door (masked errors). You do not leave sensitive documents on the reception desk (hiding sensitive columns). You change the locks periodically (token rotation). You keep a visitor log (audit logging). And you install security cameras and deadbolts (security headers and HTTPS). Laravel provides tools for all of these, and this cheat sheet shows how to use them.

### Key Characteristics

- **Defence in depth:** Multiple independent layers — token scoping, error masking, data filtering, rotation, logging, and transport security — work together so that a failure in one layer does not compromise the entire system.
- **Principle of least privilege:** Tokens are issued with the minimum abilities required for their purpose, and database users are granted only the permissions they need.
- **Fail-safe defaults:** Production errors return generic messages; sensitive columns are excluded from serialization; tokens expire and rotate automatically.
- **Observability:** All high-risk operations (authentication, authorization changes, data mutations) are logged with sufficient context for forensic analysis.
- **Standards compliance:** Token rotation follows RFC 9700 (OAuth 2.0 Security Best Current Practice); security headers follow RFC 6797 (HSTS) and W3C CSP specifications.

### Prerequisites

- PHP 8.1+ (Laravel 10+) or PHP 8.2+ (Laravel 11+).
- Composer dependency manager.
- A Laravel application with `routes/api.php` configured (via `php artisan install:api`).
- Laravel Sanctum installed and configured for API authentication.
- For audit logging: a database table for activity logs (provided by the package migration).
- For token rotation: a Redis or database cache store for refresh token storage.
- For security headers: an HTTPS-enabled production environment.

### Related Programming Areas

- **Laravel Sanctum** — Token-based authentication and abilities.
- **Eloquent API Resources** — JSON serialization and attribute whitelisting.
- **Laravel Exception Handling** — Custom error rendering in `bootstrap/app.php`.
- **Laravel Logging** — Monolog-based logging with channels and context.
- **Middleware** — Security headers, HTTPS enforcement, and request filtering.
- **Token Rotation Packages** — Community packages for refresh token lifecycle management.

### Core Concepts / Features

1. **Principle of Least Privilege:** Restricting token abilities and database permissions to only what is strictly necessary.
2. **Secure Error Responses:** Disabling debug mode (`APP_DEBUG=false`) in production and masking sensitive stack traces.
3. **Sensitive Data Protection:** Utilizing Laravel Eloquent API Resources (`JsonResource`) to hide sensitive database columns.
4. **Token Rotation & Refresh Strategies:** Implementing secure mechanisms to cycle tokens safely without breaking user sessions.
5. **Audit Logging & Monitoring:** Tracking high-risk API activities using Laravel's logging facilities or dedicated audit trail packages.
6. **Security Headers & HTTPS:** Enforcing strict transport security (HSTS), Content Security Policies (CSP), and forcing HTTPS via middleware.

---

## 1. Principle of Least Privilege

### Definitions

**Core Definition:** The principle of least privilege is the security practice of granting users, tokens, and system components only the minimum permissions necessary to perform their intended function, and nothing more.

**Technical Definition:** In Laravel APIs, least privilege is implemented at two levels: (1) **Token abilities (scopes)** — each Sanctum token is created with an explicit array of abilities, limiting what actions it can perform, and (2) **Database permissions** — the database user configured in `.env` is granted only the SQL privileges required by the application (typically `SELECT`, `INSERT`, `UPDATE`, `DELETE` on application tables, without `DROP`, `ALTER`, or `GRANT` privileges). The `abilities` middleware enforces token scopes at the route level, while database permissions are enforced by the database management system itself.

**Beginner-Friendly Explanation:** Imagine you hire a cleaner for your office. You would not give them the keys to the safe, the server room, and your personal filing cabinet — you would give them a key that opens only the rooms they need to clean. The same logic applies to API tokens: a token used by a mobile app to display the user's profile should not be able to delete the user's account or access administrative endpoints. Granting the minimum necessary permissions reduces the damage if a token is stolen.

### Purposes

- To minimize the blast radius of a compromised token by limiting the actions it can perform.
- To prevent privilege escalation attacks where a token is used to access resources or perform actions beyond its intended scope.
- To enforce separation of concerns between different clients (mobile app, SPA, CI/CD, third-party integration), each receiving a differently scoped token.
- To comply with security frameworks (OWASP, NIST, SOC 2) that mandate least-privilege access controls.
- To provide auditable, explicit declarations of what each token and database user is permitted to do.

### Syntax Rules and Structure

#### Complete General Syntax (Token Abilities)

```php
// Creating a token with specific abilities
$token = $user->createToken('mobile-app', ['profile:read', 'orders:read'])->plainTextToken;

// Creating a token with a wildcard (full access) — use sparingly
$token = $user->createToken('admin-cli', ['*'])->plainTextToken;

// Enforcing abilities via middleware (requires ALL listed abilities)
Route::middleware(['auth:sanctum', 'abilities:profile:read,orders:read'])->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::get('/orders', [OrderController::class, 'index']);
});

// Enforcing abilities via middleware (requires ANY of the listed abilities)
Route::middleware(['auth:sanctum', 'ability:orders:read,orders:write'])->group(function () {
    Route::get('/orders', [OrderController::class, 'index']);
});

// Checking abilities manually in a controller
if ($request->user()->tokenCan('orders:write')) {
    // Allow the action
}
```

```env
# .env — Database credentials with least privilege
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_api
DB_USERNAME=api_user          # NOT root
DB_PASSWORD=strong-password
```

**Component Breakdown:**

- `createToken($name, $abilities)` — The second argument is an array of ability strings. Each string represents a permission the token is granted.
- `abilities:ability1,ability2` middleware — Requires the token to have **all** listed abilities. Returns `403 Forbidden` if any are missing.
- `ability:ability1,ability2` middleware — Requires the token to have **at least one** of the listed abilities.
- `tokenCan('ability')` — Returns `true` if the token has the specified ability. Always returns `true` for first-party SPA session authentication.
- `DB_USERNAME=api_user` — The database user should be a dedicated application user, not `root`, with only the necessary SQL privileges.

**Syntax Rules:**

- Token abilities should follow a `resource:action` naming convention (e.g., `orders:read`, `orders:write`, `admin:delete`).
- The `abilities` middleware (plural) enforces **all** listed abilities; the `ability` middleware (singular) enforces **any** of the listed abilities.
- A token created with `['*']` has all abilities — this should be reserved for administrative CLI tools, never for mobile apps or third-party integrations.
- The `tokenCan()` method is only meaningful for token-based authentication; for SPA session authentication, it always returns `true`.

**Constraints and Limitations:**

- **Token abilities are not a replacement for authorization policies.** As noted in Laracasts discussions, Sanctum handles authentication (who the user is), while policies or packages like Spatie Permission handle authorization (what the user can do). A token should not declare its own permissions, otherwise privilege escalation vulnerabilities emerge where someone can issue themselves a token with more permissions than they normally would have.
- **The `abilities` column in `personal_access_tokens` is a JSON array.** It is not indexed, so querying by ability requires parsing the JSON.
- **Database least privilege requires manual configuration.** Laravel does not enforce database user permissions; this must be done at the database level.
- **Changing abilities after token issuance requires revoking and reissuing the token.** Abilities are immutable once stored.

### Annotated Code Examples

**Example 1: Scoped Tokens for Different Client Types**

```php
<?php
// File: app/Http/Controllers/Api/TokenController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class TokenController extends Controller
{
    // POST /api/tokens — Create a token with specific abilities
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'        => 'required|string|max:255',
            'client_type' => 'required|in:mobile,spa,ci,third_party',
        ]);

        // Assign abilities based on client type — least privilege
        $abilities = match ($validated['client_type']) {
            'mobile'      => ['profile:read', 'profile:write', 'orders:read', 'orders:create'],
            'spa'         => ['profile:read', 'orders:read'],
            'ci'          => ['deploy:write', 'logs:read'],
            'third_party' => ['orders:read'],
        };

        $token = $request->user()->createToken(
            $validated['name'],
            $abilities
        );

        return response()->json([
            'token'     => $token->plainTextToken,
            'name'      => $validated['name'],
            'abilities' => $abilities,
        ], 201);
    }
}
```

```php
// File: routes/api.php — Enforce abilities at the route level

Route::middleware(['auth:sanctum'])->group(function () {
    // Mobile app routes — require profile and order abilities
    Route::middleware('abilities:profile:read')->group(function () {
        Route::get('/profile', [ProfileController::class, 'show']);
    });

    Route::middleware('abilities:orders:create')->group(function () {
        Route::post('/orders', [OrderController::class, 'store']);
    });

    // CI routes — require deploy ability
    Route::middleware('abilities:deploy:write')->group(function () {
        Route::post('/deploy', [DeployController::class, 'trigger']);
    });
});
```

**Step-by-Step Setup:**

1. Define the `TokenController::store` method with client-type-based ability assignment.
2. Define routes with `abilities` middleware for each resource.
3. Create a mobile token: `POST /api/tokens` with `{"name": "iPhone 15", "client_type": "mobile"}`.
4. Attempt to access a deploy route with the mobile token.

**Expected Output (Token Creation):**

```json
{
    "token": "5|abc123def456...",
    "name": "iPhone 15",
    "abilities": ["profile:read", "profile:write", "orders:read", "orders:create"]
}
```

**Expected Output (Using the Token):**

- `GET /api/profile` with the mobile token → `200 OK` (token has `profile:read`).
- `POST /api/deploy` with the mobile token → `403 Forbidden` (token lacks `deploy:write`).

**Why This Output Occurs:** The `abilities` middleware checks the token's abilities array against the required abilities for the route. The mobile token was created with only mobile-relevant abilities, so it cannot access deployment routes. This enforces least privilege at the token level: even if the mobile token is stolen, the attacker cannot trigger deployments or access administrative data.

### Real-World Cases

- **Mobile app tokens:** Scoped to `profile:read`, `profile:write`, `orders:read`, `orders:create` — no access to admin or deployment endpoints.
- **CI/CD tokens:** Scoped to `deploy:write` and `logs:read` — cannot access user data or modify profiles.
- **Third-party integration tokens:** Scoped to `orders:read` only — cannot create, update, or delete orders.
- **Database user:** A dedicated `api_user` with only `SELECT`, `INSERT`, `UPDATE`, `DELETE` on application tables — no `DROP`, `ALTER`, or `GRANT` privileges.

---

## 2. Secure Error Responses

### Definitions

**Core Definition:** Secure error responses are API error messages that provide enough information for legitimate clients to understand what went wrong, without exposing sensitive implementation details such as stack traces, file paths, database queries, or configuration values.

**Technical Definition:** Laravel's error handling is controlled by the `APP_DEBUG` environment variable. When `APP_DEBUG=true` (development), Laravel renders detailed error pages including stack traces, file paths, and configuration values. When `APP_DEBUG=false` (production), Laravel renders generic error responses. For API requests, custom exception rendering in `bootstrap/app.php` (Laravel 11+) or `app/Exceptions/Handler.php` (Laravel 10-) allows developers to define a consistent JSON error structure while keeping sensitive details out of responses. Errors are still logged to `storage/logs/laravel.log` with full context for debugging.

**Beginner-Friendly Explanation:** When something goes wrong with an API request, the client needs to know what happened — but not enough to exploit your system. A secure error response says "An unexpected error occurred. Please try again later." An insecure one says "SQLSTATE[42S02]: Base table or view not found: 1146 Table 'myapp.users' doesn't exist (SQL: select * from `users` where `email` = 'admin@example.com')". The first tells the client to retry; the second tells an attacker exactly how your database is structured.

### Purposes

- To prevent attackers from learning about the application's internal structure, dependencies, and configuration through error messages.
- To provide clients with a consistent, predictable error format that they can handle programmatically.
- To ensure that sensitive data (database queries, file paths, API keys) is never exposed in API responses.
- To maintain detailed error logs on the server for debugging and monitoring, while keeping the client-facing response minimal.
- To comply with security standards (OWASP, PCI DSS) that mandate the suppression of detailed error information in production.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel 11+ — `bootstrap/app.php`)

```php
<?php
// File: bootstrap/app.php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Http\Request;
use Throwable;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withExceptions(function (Exceptions $exceptions) {
        // Custom JSON error rendering for API requests
        $exceptions->render(function (Throwable $e, Request $request) {
            if ($request->expectsJson()) {
                $status = method_exists($e, 'getStatusCode')
                    ? $e->getStatusCode()
                    : 500;

                // Never expose exception messages in production
                $message = app()->isProduction()
                    ? 'An unexpected error occurred. Please try again later.'
                    : $e->getMessage();

                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'INTERNAL_ERROR',
                        'message' => $message,
                    ],
                ], $status);
            }
        });
    })
    ->create();
```

```env
# .env — Production configuration
APP_ENV=production
APP_DEBUG=false
```

**Component Breakdown:**

- `APP_ENV=production` — Sets the application environment to production. This disables detailed error reporting and enables production-specific optimizations.
- `APP_DEBUG=false` — Disables debug mode. Laravel will not render stack traces or detailed error pages.
- `$exceptions->render(function (Throwable $e, Request $request) { ... })` — Registers a custom render callback for all exceptions. The callback receives the exception and the incoming request.
- `$request->expectsJson()` — Returns `true` if the client expects a JSON response (based on the `Accept: application/json` header).
- `app()->isProduction()` — Returns `true` when `APP_ENV=production`. Used to conditionally expose or mask the exception message.

**Syntax Rules:**

- `APP_DEBUG` **must** be `false` in production. Setting it to `true` in production risks exposing sensitive configuration values to end users.
- The `render()` callback is called for every unhandled exception. Returning `null` allows Laravel's default handling to proceed.
- Exception messages in production should be generic. Use `app()->isProduction()` to conditionally mask messages.
- Full exception details are always logged to `storage/logs/laravel.log` regardless of `APP_DEBUG`, so server-side debugging remains possible.

**Constraints and Limitations:**

- **`APP_DEBUG=false` does not hide error messages from custom exception handlers.** If a custom handler returns `$e->getMessage()`, that message is exposed regardless of `APP_DEBUG`. The handler must explicitly check `app()->isProduction()`.
- **Some third-party packages may expose error details** if they do not respect Laravel's error handling. Audit package error responses in production.
- **Validation errors (422) are always returned with detailed field-level messages.** This is intentional — clients need to know which fields failed validation.
- **404 and 403 responses are not affected by `APP_DEBUG`.** They return standard status codes regardless of the debug setting.

### Annotated Code Examples

**Example 1: Custom API Error Handler with Debug Awareness**

```php
<?php
// File: bootstrap/app.php (Laravel 11+)

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Http\Request;
use Illuminate\Validation\ValidationException;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Throwable;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withExceptions(function (Exceptions $exceptions) {
        // Validation errors — always return field-level details
        $exceptions->render(function (ValidationException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'VALIDATION_ERROR',
                        'message' => 'The submitted data is invalid.',
                        'details' => $e->errors(),
                    ],
                ], 422);
            }
        });

        // Model not found — return resource-specific 404
        $exceptions->render(function (ModelNotFoundException $e, Request $request) {
            if ($request->expectsJson()) {
                $model = class_basename($e->getModel());
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'RESOURCE_NOT_FOUND',
                        'message' => "{$model} not found.",
                    ],
                ], 404);
            }
        });

        // Global fallback — mask internal details in production
        $exceptions->render(function (Throwable $e, Request $request) {
            if ($request->expectsJson()) {
                $status = method_exists($e, 'getStatusCode')
                    ? $e->getStatusCode()
                    : 500;

                // In production, return generic message.
                // In development, return the actual exception message.
                $message = app()->isProduction()
                    ? 'An unexpected error occurred. Please try again later.'
                    : $e->getMessage();

                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'INTERNAL_ERROR',
                        'message' => $message,
                    ],
                ], $status);
            }
        });
    })
    ->create();
```

**Step-by-Step Setup:**

1. Set `APP_DEBUG=false` and `APP_ENV=production` in `.env`.
2. Add the custom exception render callbacks in `bootstrap/app.php`.
3. Deploy to production.
4. Trigger an unhandled exception (e.g., a database query error).

**Expected Output (Production — `APP_DEBUG=false`):**

```json
{
    "success": false,
    "error": {
        "code": "INTERNAL_ERROR",
        "message": "An unexpected error occurred. Please try again later."
    }
}
```

**Expected Output (Development — `APP_DEBUG=true`):**

```json
{
    "success": false,
    "error": {
        "code": "INTERNAL_ERROR",
        "message": "SQLSTATE[42S02]: Base table or view not found: 1146 Table 'myapp.users' doesn't exist"
    }
}
```

**Why This Output Occurs:** The `render()` callback checks `app()->isProduction()` to determine whether to mask the exception message. In production, the client receives a generic message that reveals nothing about the internal implementation. In development, the full exception message is returned to aid debugging. Regardless of the environment, the full exception details are logged to `storage/logs/laravel.log` for server-side analysis.

### Real-World Cases

- **Public APIs:** Generic error messages prevent attackers from fingerprinting the application's framework version, database schema, or dependencies.
- **Compliance environments (PCI DSS, HIPAA):** Masked error responses ensure that no sensitive data leaks through error messages, satisfying regulatory requirements.
- **Multi-tenant SaaS:** Tenant-specific error details are masked to prevent one tenant from learning about another tenant's data structure.
- **Payment APIs:** Generic error responses prevent attackers from using error messages to probe for valid payment methods or card numbers.

---

## 3. Sensitive Data Protection

### Definitions

**Core Definition:** Sensitive data protection in Laravel APIs is the practice of explicitly controlling which database columns are included in JSON responses, preventing accidental exposure of passwords, internal IDs, tokens, and other confidential attributes.

**Technical Definition:** Laravel provides three complementary mechanisms for hiding sensitive columns: (1) the `$hidden` property on Eloquent models, which excludes attributes from `toArray()` and `toJson()` serialization; (2) the `$visible` property, which acts as an allow-list; and (3) **Eloquent API Resources** (`JsonResource`), which provide explicit transformation of model data into a controlled array structure. API Resources are the recommended approach for API responses because they decouple the JSON structure from the database schema, preventing accidental exposure when columns are added to the database.

**Beginner-Friendly Explanation:** When you return a User model from a controller, Laravel converts all its database columns into JSON — including `password`, `remember_token`, and `email_verified_at`. This is like handing someone your entire wallet instead of just your ID card. API Resources solve this by letting you explicitly define which fields appear in the JSON. Think of it as a template: you decide exactly what the client sees, and everything else stays hidden.

### Purposes

- To prevent accidental exposure of sensitive database columns (passwords, tokens, internal IDs) in API responses.
- To decouple the API's JSON structure from the database schema, allowing columns to be added or renamed without breaking clients.
- To provide different representations of the same model for different endpoints (e.g., a `UserSummaryResource` for lists and a `UserDetailResource` for single views).
- To conditionally include fields based on user permissions or request parameters.
- To comply with data protection regulations (GDPR, CCPA) that require minimizing the personal data exposed to clients.

### Syntax Rules and Structure

#### Complete General Syntax (API Resource)

```php
<?php
// File: app/Http/Resources/UserResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'name'       => $this->name,
            'email'      => $this->email,
            'created_at' => $this->created_at?->toIso8601String(),
            // Relationships only included if eager-loaded
            'orders' => OrderResource::collection($this->whenLoaded('orders')),
        ];
    }
}
```

```php
// Using the resource in a controller
return new UserResource($user);
return UserResource::collection($users);
```

```php
// Alternative: $hidden property on the model
protected $hidden = [
    'password',
    'remember_token',
    'email_verified_at',
    'two_factor_secret',
];
```

**Component Breakdown:**

- `toArray(Request $request): array` — Defines the JSON structure. `$this` refers to the underlying Eloquent model.
- `$this->id`, `$this->name`, `$this->email` — Explicitly include only the fields that should be exposed.
- `$this->created_at?->toIso8601String()` — Format dates as ISO 8601 strings.
- `$this->whenLoaded('orders')` — Include the `orders` relationship only if it was eager-loaded, preventing N+1 queries.
- `$hidden` — An array of attribute names that are excluded from `toArray()` and `toJson()` serialization.

**Syntax Rules:**

- **Never** return raw Eloquent models directly from API controllers. Always transform them using `JsonResource` or `ResourceCollection`.
- The `$hidden` property on the model provides a baseline level of protection, but API Resources provide explicit, endpoint-specific control.
- `whenLoaded()` should be used for relationships to prevent N+1 queries and to include relationships only when the controller has explicitly eager-loaded them.
- The `data` wrapper is applied by default; disable it globally via `JsonResource::withoutWrapping()` if a flat structure is required.

**Constraints and Limitations:**

- **`$hidden` is applied at the model level.** If a column is not listed in `$hidden`, it appears in JSON when the model is serialized directly. API Resources bypass `$hidden` because `toArray()` explicitly defines the output.
- **API Resources do not prevent SQL injection or mass assignment.** They only control output; input validation and mass-assignment protection are separate concerns.
- **Conditional attributes add complexity.** Overusing `when()` and `mergeWhen()` can make resources harder to debug.

### Annotated Code Examples

**Example 1: API Resource Hiding Sensitive Columns**

```php
<?php
// File: app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    // Model-level hiding — a baseline protection
    protected $hidden = [
        'password',
        'remember_token',
        'two_factor_secret',
        'two_factor_recovery_codes',
        'email_verified_at',
    ];
}
```

```php
<?php
// File: app/Http/Resources/UserResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'    => $this->id,
            'name'  => $this->name,
            'email' => $this->email,

            // Conditional: only include phone if the viewer is the user
            'phone' => $this->when(
                $request->user()?->id === $this->id,
                $this->phone
            ),

            // Conditional: only include admin flags for administrators
            $this->mergeWhen($request->user()?->isAdmin(), [
                'is_admin'       => $this->is_admin,
                'last_login_ip'  => $this->last_login_ip,
                'login_count'    => $this->login_count,
            ]),

            'created_at' => $this->created_at?->toIso8601String(),
        ];
    }
}
```

```php
// Controller
public function show(User $user): UserResource
{
    // Eager-load relationships that should be included
    return new UserResource($user->load('orders'));
}
```

**Expected Output (as the user viewing their own profile):**

```json
{
    "data": {
        "id": 1,
        "name": "Alice Johnson",
        "email": "alice@example.com",
        "phone": "+1-555-0100",
        "created_at": "2024-01-15T08:30:00+00:00"
    }
}
```

**Expected Output (as an admin viewing another user):**

```json
{
    "data": {
        "id": 1,
        "name": "Alice Johnson",
        "email": "alice@example.com",
        "is_admin": false,
        "last_login_ip": "192.168.1.100",
        "login_count": 42,
        "created_at": "2024-01-15T08:30:00+00:00"
    }
}
```

**Expected Output (as a regular user viewing another user):**

```json
{
    "data": {
        "id": 1,
        "name": "Alice Johnson",
        "email": "alice@example.com",
        "created_at": "2024-01-15T08:30:00+00:00"
    }
}
```

**Why This Output Occurs:** The `UserResource` explicitly defines which fields appear in each context. The `password`, `remember_token`, and `two_factor_secret` columns are never included because they are not in the `toArray()` return array. The `phone` field is included only when the authenticated user's ID matches the resource's ID. Admin-only fields are merged only when `isAdmin()` returns true. This provides defence in depth: even if the model's `$hidden` property is accidentally removed, the API Resource still prevents sensitive data exposure.

### Real-World Cases

- **User profile endpoints:** `UserResource` excludes `password`, `remember_token`, and internal flags; includes `phone` only for the user themselves.
- **Admin dashboards:** A separate `AdminUserResource` includes administrative fields (`last_login_ip`, `login_count`) that are hidden from regular users.
- **Public APIs:** API Resources define a stable, documented JSON contract that is independent of database schema changes.
- **GDPR compliance:** Resources ensure that only the minimum necessary personal data is exposed to clients, supporting data minimization requirements.

---

## 4. Token Rotation & Refresh Strategies

### Definitions

**Core Definition:** Token rotation is the security practice of issuing a new access token and refresh token pair each time a refresh token is used, while invalidating the old refresh token, so that a stolen token becomes useless after a single use.

**Technical Definition:** In a refresh token rotation scheme, a long-lived **refresh token** is stored securely on the client and used exclusively to obtain new short-lived **access tokens**. Each refresh operation issues a new access token and a new refresh token, and marks the old refresh token as used. If an old refresh token is presented again (indicating a replay attack), the entire token family is revoked. This follows RFC 9700 (OAuth 2.0 Security Best Current Practice), which requires rotation with reuse detection for refresh tokens. Laravel Sanctum does not provide refresh tokens natively; community packages such as `reiarseni/sanctum-refresh-token` and `d076/sanctum-refresh-tokens` add this capability.

**Beginner-Friendly Explanation:** Imagine you have a hotel key card. Every time you use it to open your room, the hotel issues you a new key card and deactivates the old one. If someone steals your old key card and tries to use it, the hotel's system recognizes it as already-used and locks down the entire room — alerting you that something is wrong. This is token rotation: access tokens are short-lived, refresh tokens are single-use, and any attempt to reuse a refresh token triggers a security response.

### Purposes

- To limit the window of opportunity for an attacker who steals a token: even if a refresh token is compromised, it becomes useless after the legitimate user refreshes.
- To detect token theft through replay detection: if an old refresh token is used, the entire token family is revoked, alerting the system to a potential compromise.
- To allow users to remain logged in for extended periods (weeks or months) without requiring frequent re-authentication, while keeping access tokens short-lived.
- To comply with RFC 9700 and OAuth 2.0 security best practices, which mandate rotation and reuse detection.
- To provide a "sliding session" experience where each use extends the session, but the tokens themselves are continuously rotated.

### Syntax Rules and Structure

#### Complete General Syntax (Using a Rotation Package)

```bash
# Installation
composer require reiarseni/sanctum-refresh-token
```

```php
// The package provides createTokenPair and refreshTokenPair methods

// Creating a token pair on login
$tokens = $user->createTokenPair('device-name', ['profile:read', 'orders:read']);
// Returns: ['access_token' => ..., 'refresh_token' => ...]

// Refreshing a token pair
$newTokens = $user->refreshTokenPair($refreshToken);
// Returns: ['access_token' => ..., 'refresh_token' => ...]
```

```php
// File: app/Http/Controllers/Api/AuthController.php — Login with token pair

use Reiarseni\SanctumRefreshToken\Traits\HasRefreshTokens;

class AuthController extends Controller
{
    public function login(Request $request): JsonResponse
    {
        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        // Create a token pair: short-lived access token + long-lived refresh token
        $tokens = $user->createTokenPair(
            $request->device_name,
            ['profile:read', 'orders:read', 'orders:create']
        );

        return response()->json([
            'access_token'  => $tokens['access_token'],
            'refresh_token' => $tokens['refresh_token'],
            'token_type'    => 'Bearer',
            'expires_in'    => 900, // 15 minutes
        ]);
    }
}
```

**Component Breakdown:**

- `createTokenPair($name, $abilities)` — Creates a short-lived access token and a long-lived refresh token. The access token is used for API requests; the refresh token is used only for obtaining new token pairs.
- `refreshTokenPair($refreshToken)` — Exchanges a valid refresh token for a new access token and refresh token pair. The old refresh token is invalidated.
- Replay detection: if an already-used refresh token is presented, the package revokes the entire token family and returns a `409 Conflict` response.
- The `HasRefreshTokens` trait adds the necessary methods to the `User` model.

**Syntax Rules:**

- Access tokens should be short-lived (15–60 minutes); refresh tokens should be long-lived (7–30 days).
- Refresh tokens must be stored securely on the client (e.g., in the iOS Keychain or Android Keystore). They should never be stored in `localStorage` or `sessionStorage`.
- The `refreshTokenPair` method should be called only from a dedicated refresh endpoint, not from regular API endpoints.
- Reuse detection requires storing token lineage (families and generations) in the database, which the package handles automatically.

**Constraints and Limitations:**

- **Sanctum does not support refresh tokens natively.** A community package is required. These packages are pre-1.0 and may change between minor versions.
- **Token rotation adds database overhead.** Each rotation creates new token records and marks old ones as used or revoked.
- **Refresh token rotation is not a substitute for short access token lifespans.** Both are required: short access tokens limit the damage of a stolen access token; rotation limits the damage of a stolen refresh token.
- **Mobile apps on unreliable networks may experience refresh failures.** Some packages handle this with `409 rotation_in_progress` responses, allowing retries.

### Annotated Code Examples

**Example 1: Complete Login and Refresh Flow**

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
    // POST /api/login
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

        // Create a short-lived access token + long-lived refresh token
        $tokens = $user->createTokenPair(
            $request->device_name,
            ['profile:read', 'orders:read']
        );

        return response()->json([
            'access_token'  => $tokens['access_token'],
            'refresh_token' => $tokens['refresh_token'],
            'token_type'    => 'Bearer',
            'expires_in'    => 900, // 15 minutes
        ]);
    }

    // POST /api/refresh
    public function refresh(Request $request): JsonResponse
    {
        $request->validate([
            'refresh_token' => 'required|string',
        ]);

        try {
            $tokens = $request->user()->refreshTokenPair(
                $request->refresh_token
            );

            return response()->json([
                'access_token'  => $tokens['access_token'],
                'refresh_token' => $tokens['refresh_token'],
                'token_type'    => 'Bearer',
                'expires_in'    => 900,
            ]);
        } catch (RefreshTokenReuseException $e) {
            // Replay detected — the token family has been revoked
            return response()->json([
                'error' => 'Token reuse detected. All sessions have been revoked for security.',
            ], 409);
        }
    }
}
```

**Step-by-Step Setup:**

1. Install the refresh token package: `composer require reiarseni/sanctum-refresh-token`.
2. Run the package migrations: `php artisan migrate`.
3. Add the `HasRefreshTokens` trait to the `User` model.
4. Define the login and refresh routes.
5. Test the flow: login → use access token → wait for expiry → refresh → use new access token → attempt to reuse old refresh token → 409.

**Expected Output (Login):**

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
    "refresh_token": "def50200abc123...",
    "token_type": "Bearer",
    "expires_in": 900
}
```

**Expected Output (Refresh):**

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
    "refresh_token": "ghi789def456...",
    "token_type": "Bearer",
    "expires_in": 900
}
```

**Expected Output (Replay Attack):**

```json
{
    "error": "Token reuse detected. All sessions have been revoked for security."
}
```

**HTTP Status Code: 409 Conflict**

**Why This Output Occurs:** The `createTokenPair` method creates a token family with a generation counter. Each refresh operation increments the generation and invalidates the previous refresh token. When an old refresh token is presented (replay attack), the package detects that the token's generation is behind the current generation, revokes the entire family (including the legitimate user's current tokens), and returns a `409 Conflict` response. This follows RFC 9700's reuse detection requirements.

### Real-World Cases

- **Mobile apps:** Users stay logged in for weeks with refresh tokens; access tokens expire every 15 minutes, limiting the damage of a stolen access token.
- **SPAs:** First-party SPAs use short-lived access tokens for API requests and refresh tokens stored in `HttpOnly` cookies for renewal.
- **Financial APIs:** Aggressive rotation (5-minute access tokens, 24-hour refresh tokens) with replay detection to detect and respond to token theft.
- **Multi-device scenarios:** Each device gets its own token family; revoking one device's refresh token does not affect other devices.

---

## 5. Audit Logging & Monitoring

### Definitions

**Core Definition:** Audit logging is the systematic recording of security-relevant events — authentication attempts, authorization changes, data mutations, and administrative actions — in a tamper-evident log for compliance, forensic analysis, and real-time threat detection.

**Technical Definition:** Laravel's audit logging can be implemented at multiple levels: (1) **Laravel's built-in logging** via `Log::info()`, `Log::warning()`, and custom channels configured in `config/logging.php`; (2) **Spatie Laravel Activitylog** (`spatie/laravel-activitylog`), a popular package that automatically logs Eloquent model events (`created`, `updated`, `deleted`) and stores them in an `activity_log` table with field-level diffs; and (3) **Specialized audit packages** such as `lunnar/laravel-audit-logging`, which provide HMAC checksums for integrity verification, request tracing, and outgoing HTTP request logging. Audit logs should include the actor (user ID), the action, the subject (model/table), the before/after values, the IP address, and a timestamp.

**Beginner-Friendly Explanation:** Audit logging is like a security camera system for your API. Every time someone logs in, changes their password, updates a record, or performs an administrative action, the system records what happened, who did it, when, and from where. If something goes wrong — a data breach, an unauthorized change, a suspicious login — you can review the logs to understand exactly what happened and take corrective action. Without audit logs, you are flying blind.

### Purposes

- To provide a forensic trail for investigating security incidents, data breaches, and unauthorized access.
- To comply with regulatory requirements (SOC 2, HIPAA, PCI DSS, GDPR) that mandate audit trails for sensitive data access and modifications.
- To detect anomalous behavior (e.g., multiple failed login attempts, unusual data export volumes) in real time or near-real time.
- To track field-level changes to critical models, providing a complete history of how data evolved.
- To attribute actions to specific users, IP addresses, and devices, enabling accountability.

### Syntax Rules and Structure

#### Complete General Syntax (Spatie Activitylog)

```bash
# Installation
composer require spatie/laravel-activitylog
php artisan vendor:publish --provider="Spatie\Activitylog\ActivitylogServiceProvider" --tag="activitylog-migrations"
php artisan migrate
```

```php
// Add the trait to models that should be audited
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Spatie\Activitylog\Traits\LogsActivity;
use Spatie\Activitylog\LogOptions;

class Order extends Model
{
    use LogsActivity;

    public function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()
            ->logOnly(['status', 'total_amount', 'shipping_address'])
            ->logOnlyDirty() // Only log changed attributes
            ->dontSubmitEmptyLogs();
    }
}
```

```php
// Manual logging
activity()
    ->causedBy($user)
    ->performedOn($order)
    ->withProperties(['old_status' => 'pending', 'new_status' => 'shipped'])
    ->log('Order status updated');
```

**Component Breakdown:**

- `use LogsActivity` — Trait that automatically logs `created`, `updated`, and `deleted` events for the model.
- `getActivitylogOptions()` — Configures which attributes are logged, whether to log only dirty (changed) attributes, and other options.
- `activity()->causedBy($user)->performedOn($order)->log('message')` — Manual logging with full context: who performed the action, on what model, and with what properties.
- The `activity_log` table stores `log_name`, `description`, `subject_type`, `subject_id`, `causer_type`, `causer_id`, `properties` (JSON), and timestamps.

**Syntax Rules:**

- The `LogsActivity` trait should be added to all models that contain sensitive data or perform critical operations.
- `logOnly()` should be used to explicitly whitelist attributes for logging, preventing accidental logging of passwords or tokens.
- `logOnlyDirty()` should be used to log only changed attributes, reducing log volume.
- Audit logs should be stored in a separate database connection or table from application data to prevent tampering.
- Log retention policies should be configured to comply with regulatory requirements (e.g., 1 year for PCI DSS, 6 years for HIPAA).

**Constraints and Limitations:**

- **Audit logging adds database write overhead.** Each logged event inserts a row into the `activity_log` table. For high-volume APIs, consider asynchronous logging via a queue.
- **Spatie Activitylog does not include HMAC integrity verification by default.** For tamper-evident logging, use a package like `lunnar/laravel-audit-logging` that provides HMAC checksums.
- **Audit logs may contain sensitive data.** Ensure that `logOnly()` does not include passwords, tokens, or personal data that would violate GDPR.
- **Log storage can grow unbounded.** Implement retention policies and archiving to prevent database bloat.

### Annotated Code Examples

**Example 1: Auditing Critical Model Changes with Spatie Activitylog**

```php
<?php
// File: app/Models/Order.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Spatie\Activitylog\Traits\LogsActivity;
use Spatie\Activitylog\LogOptions;

class Order extends Model
{
    use LogsActivity;

    protected $fillable = [
        'user_id', 'status', 'total_amount', 'shipping_address',
    ];

    public function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()
            ->logOnly(['status', 'total_amount', 'shipping_address'])
            ->logOnlyDirty()
            ->dontSubmitEmptyLogs();
    }
}
```

```php
<?php
// File: app/Http/Controllers/Api/OrderController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Order;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function update(Request $request, Order $order): JsonResponse
    {
        $this->authorize('update', $order);

        $validated = $request->validate([
            'status'           => 'sometimes|in:pending,processing,shipped,delivered,cancelled',
            'shipping_address' => 'sometimes|string|max:500',
        ]);

        // The LogsActivity trait automatically logs the changes
        // with field-level diffs, the authenticated user as causer,
        // and the IP address from the request.
        $order->update($validated);

        return response()->json($order);
    }
}
```

```php
// Querying activity logs
use Spatie\Activitylog\Models\Activity;

// Get all activity for a specific order
$activities = Activity::where('subject_type', Order::class)
    ->where('subject_id', $order->id)
    ->with('causer')
    ->latest()
    ->get();

// Each activity contains:
// - log_name: "default"
// - description: "updated"
// - subject_type: "App\Models\Order"
// - subject_id: 1
// - causer_type: "App\Models\User"
// - causer_id: 5
// - properties: {"old": {"status": "pending"}, "attributes": {"status": "shipped"}}
// - created_at: "2025-06-15 10:30:00"
```

**Step-by-Step Setup:**

1. Install Spatie Activitylog: `composer require spatie/laravel-activitylog`.
2. Publish and run migrations: `php artisan vendor:publish --provider="Spatie\Activitylog\ActivitylogServiceProvider" --tag="activitylog-migrations" && php artisan migrate`.
3. Add the `LogsActivity` trait to the `Order` model and configure `getActivitylogOptions()`.
4. Perform an update operation on an order.
5. Query the `activity_log` table to verify the log entry.

**Expected Output (Activity Log Entry):**

```json
{
    "id": 42,
    "log_name": "default",
    "description": "updated",
    "subject_type": "App\\Models\\Order",
    "subject_id": 1,
    "causer_type": "App\\Models\\User",
    "causer_id": 5,
    "properties": {
        "old": { "status": "pending" },
        "attributes": { "status": "shipped" }
    },
    "created_at": "2025-06-15T10:30:00.000000Z",
    "updated_at": "2025-06-15T10:30:00.000000Z"
}
```

**Why This Output Occurs:** The `LogsActivity` trait hooks into Eloquent's model events. When `$order->update($validated)` is called, the trait captures the old and new values of the `status` and `shipping_address` attributes (because `logOnly()` whitelists them), records the authenticated user as the causer, and inserts a row into the `activity_log` table. The `logOnlyDirty()` option ensures that only changed attributes are logged, reducing noise.

### Real-World Cases

- **Financial applications:** Every transaction, balance change, and account modification is logged with field-level diffs for audit and compliance.
- **Healthcare systems (HIPAA):** Access to patient records is logged with the actor, timestamp, and purpose, satisfying audit trail requirements.
- **E-commerce platforms:** Order status changes, price modifications, and refund processing are logged for dispute resolution.
- **Multi-tenant SaaS:** Tenant administrators can view audit logs for their own tenant, providing transparency and accountability.

---

## 6. Security Headers & HTTPS

### Definitions

**Core Definition:** Security headers are HTTP response headers that instruct browsers to enforce security policies, such as requiring HTTPS (HSTS), restricting content sources (CSP), and preventing clickjacking (X-Frame-Options).

**Technical Definition:** Laravel applications can enforce security headers through middleware packages such as `philiprehberger/laravel-security-headers`, `acolyte/laravel-security`, or `jeffersongoncalves/laravel-security-headers`. The most important headers are: **Strict-Transport-Security (HSTS)** — tells browsers to only connect via HTTPS for a specified period, **Content-Security-Policy (CSP)** — restricts which sources scripts, styles, images, and other resources can be loaded from, **X-Content-Type-Options: nosniff** — prevents MIME type sniffing, **X-Frame-Options: DENY** — prevents the site from being embedded in iframes (clickjacking protection), and **Referrer-Policy** — controls how much referrer information is sent with requests. HTTPS enforcement can be implemented via Laravel's `URL::forceScheme('https')` in a service provider or via a middleware that redirects HTTP requests to HTTPS.

**Beginner-Friendly Explanation:** Security headers are like the rules posted at the entrance of a secure building: "All visitors must use the north entrance" (HSTS), "No photography allowed in this area" (CSP), "Do not accept packages from unknown sources" (nosniff). Without these rules, browsers might connect via insecure HTTP, load malicious scripts from untrusted sources, or allow your site to be embedded in a phishing page. Laravel middleware packages make it easy to add these headers to every response.

### Purposes

- To enforce HTTPS for all connections, preventing man-in-the-middle attacks and session hijacking (HSTS).
- To restrict the sources from which scripts, styles, and other resources can be loaded, mitigating XSS attacks (CSP).
- To prevent MIME type sniffing attacks by instructing browsers to respect declared content types (X-Content-Type-Options).
- To prevent clickjacking attacks by disallowing the site from being embedded in iframes (X-Frame-Options).
- To control how much referrer information is leaked to third-party sites (Referrer-Policy).

### Syntax Rules and Structure

#### Complete General Syntax (Using a Security Headers Package)

```bash
# Installation
composer require philiprehberger/laravel-security-headers
```

```php
// Laravel 11+ — bootstrap/app.php
use PhilipRehberger\SecurityHeaders\SecurityHeaders;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        SecurityHeaders::class,
    ]);
})
```

```php
// config/security-headers.php
return [
    'hsts' => [
        'enabled'        => env('SECURITY_HEADERS_HSTS', false),
        'max_age'        => 31536000, // 1 year
        'include_subdomains' => true,
    ],
    'csp' => [
        'enabled'     => true,
        'report_only' => false,
        'nonce_view_variable' => 'cspNonce',
        'script_src'  => [],
        'style_src'   => [],
        'img_src'     => [],
        'font_src'    => [],
        'connect_src' => [],
        'frame_ancestors' => ["'self'"],
        'form_action' => ["'self'"],
    ],
];
```

```env
# .env — Enable HSTS only when fully on HTTPS
SECURITY_HEADERS_HSTS=true
```

**Component Breakdown:**

- `SecurityHeaders::class` — Middleware that applies all configured security headers to every response.
- `hsts.enabled` — Enables the `Strict-Transport-Security` header. Should only be enabled when the application is fully served over HTTPS.
- `hsts.max_age` — The duration (in seconds) that browsers should remember to only connect via HTTPS. 31536000 = 1 year.
- `hsts.include_subdomains` — Applies the HSTS policy to all subdomains.
- `csp.enabled` — Enables the `Content-Security-Policy` header.
- `csp.script_src` — Additional sources for scripts beyond `'self'` and the nonce.
- `csp.nonce_view_variable` — The variable name for the CSP nonce in Blade templates (`$cspNonce`).

**Syntax Rules:**

- HSTS **must** only be enabled when the application is fully served over HTTPS. Enabling HSTS on an HTTP site will make it inaccessible.
- CSP should start in `report_only` mode to identify violations before enforcing the policy.
- The CSP nonce should be used for inline scripts and styles. Blade templates must include the nonce attribute: `<script nonce="{{ $cspNonce }}">`.
- HTTPS should be enforced at the web server level (Nginx, Apache) or via Laravel's `URL::forceScheme('https')`.

**Constraints and Limitations:**

- **HSTS is a one-way switch.** Once a browser receives the HSTS header, it will refuse to connect via HTTP for the specified `max-age`. If you need to disable HSTS, you must wait for the max-age to expire.
- **CSP can break legitimate functionality.** Inline scripts, eval, and third-party resources may be blocked. Start in report-only mode and iterate.
- **Security headers only apply to browser-based clients.** API clients (mobile apps, CLI tools) do not process security headers.
- **HTTPS termination may occur at a load balancer.** Laravel may receive HTTP requests even when the client connects via HTTPS. Use `TrustProxies` middleware to correctly detect HTTPS.

### Annotated Code Examples

**Example 1: Enforcing HTTPS and Adding Security Headers**

```php
<?php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Force HTTPS for all generated URLs in production
        if ($this->app->environment('production')) {
            URL::forceScheme('https');
        }
    }
}
```

```php
<?php
// File: app/Http/Middleware/ForceHttps.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ForceHttps
{
    public function handle(Request $request, Closure $next): Response
    {
        if (!$request->secure() && app()->environment('production')) {
            return redirect()->secure($request->getRequestUri());
        }

        return $next($request);
    }
}
```

```php
// Laravel 11+ — bootstrap/app.php
use App\Http\Middleware\ForceHttps;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        ForceHttps::class,
    ]);
    $middleware->api(append: [
        ForceHttps::class,
    ]);
})
```

```php
// config/security-headers.php — CSP with nonce for inline scripts
'csp' => [
    'enabled'     => true,
    'report_only' => false,
    'nonce_view_variable' => 'cspNonce',
    'script_src'  => ['https://cdn.jsdelivr.net'], // Allow specific CDN
    'style_src'   => ['https://fonts.googleapis.com'],
    'img_src'     => ['https://images.example.com'],
    'connect_src' => ["'self'", 'https://api.example.com'],
],
```

**Step-by-Step Setup:**

1. Install the security headers package: `composer require philiprehberger/laravel-security-headers`.
2. Publish the config: `php artisan vendor:publish --tag=security-headers-config`.
3. Register the middleware in `bootstrap/app.php`.
4. Enable HSTS in `.env`: `SECURITY_HEADERS_HSTS=true` (only in production with HTTPS).
5. Force HTTPS in `AppServiceProvider` and via middleware.
6. Test with `curl -I https://your-api.com/api/user` to verify headers.

**Expected Output (Response Headers):**

```
HTTP/1.1 200 OK
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: https://images.example.com; connect-src 'self' https://api.example.com; frame-ancestors 'self'; form-action 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
```

**Why This Output Occurs:** The `SecurityHeaders` middleware adds the configured headers to every response. HSTS tells the browser to only connect via HTTPS for the next year. CSP restricts which sources can load scripts, styles, and images. The nonce allows specific inline scripts to execute. `X-Content-Type-Options` prevents MIME sniffing. `X-Frame-Options` prevents clickjacking. `Referrer-Policy` controls referrer information leakage.

### Real-World Cases

- **Public APIs:** HSTS ensures all API communication is encrypted; CSP is less critical for JSON-only APIs but protects any web-based documentation.
- **SPAs with API backends:** CSP protects against XSS in the SPA; HSTS ensures HTTPS; `connect-src` restricts which API endpoints the SPA can call.
- **E-commerce platforms:** CSP prevents malicious scripts from stealing payment data; HSTS ensures checkout pages are always served over HTTPS.
- **Compliance environments:** HSTS and CSP are required by PCI DSS for payment processing applications.

---

## References

- Laravel Sanctum Documentation (12.x) — https://laravel.com/docs/12.x/sanctum
- Laravel Sanctum: Token Abilities — https://laravel.com/docs/12.x/sanctum#token-abilities
- Laravel Error Handling Documentation — https://laravel.com/docs/errors
- Laravel Eloquent: API Resources — https://laravel.com/docs/eloquent-resources
- Laravel Eloquent: Serialization (Hiding Attributes) — https://laravel.com/docs/eloquent-serialization#hiding-attributes-from-json
- Laravel Routing: CORS Configuration — https://laravel.com/docs/routing#cors
- reiarseni/sanctum-refresh-token — https://packagist.org/packages/reiarseni/sanctum-refresh-token
- d076/sanctum-refresh-tokens — https://larablocks.com/package/d076/sanctum-refresh-tokens
- Mishanki/sanctum-refresh-token — https://github.com/Mishanki/sanctum-refresh-token
- Spatie Laravel Activitylog — https://github.com/spatie/laravel-activitylog
- Spatie Activitylog Documentation — https://spatie.be/docs/laravel-activitylog
- lunnar/laravel-audit-logging — https://packagist.org/packages/lunnar/laravel-audit-logging
- philiprehberger/laravel-security-headers — https://packagist.org/packages/philiprehberger/laravel-security-headers
- acolyte/laravel-security — https://packagist.org/packages/acolyte/laravel-security
- jeffersongoncalves/laravel-security-headers — https://packagist.org/packages/jeffersongoncalves/laravel-security-headers
- RFC 9700: OAuth 2.0 Security Best Current Practice — https://datatracker.ietf.org/doc/rfc9700/
- RFC 6797: HTTP Strict Transport Security (HSTS) — https://www.rfc-editor.org/rfc/rfc6797
- W3C Content Security Policy Level 3 — https://www.w3.org/TR/CSP3/
- OWASP API Security Top 10 — https://owasp.org/API-Security/
- Managing Authorization with Sanctum and Spatie Permissions (Laracasts) — https://laracasts.com/discuss/channels/laravel/managing-authorization-with-sanctum-and-spatie-permissions
- Laravel API Security in Depth (DEV Community) — https://dev.to/laravel-api-security-in-depth