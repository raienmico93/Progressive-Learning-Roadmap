# Laravel API Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A Laravel API (Application Programming Interface) is a structured interface built with Laravel that allows external applications—mobile apps, single-page applications, third-party services—to communicate with your application's backend over HTTP, typically exchanging data in JSON format.

**Technical Definition**

Laravel APIs are built on the framework's routing, controller, middleware, and Eloquent systems, combined with specialized tools for response transformation (`JsonResource`), authentication (Sanctum/Passport), rate limiting (`ThrottleRequests`), and versioning. API routes are defined in `routes/api.php`, assigned the `api` middleware group (stateless, no session/CSRF), and automatically prefixed with `/api`. The `install:api` Artisan command scaffolds the API layer, installing Sanctum and creating the route file.

**Beginner-Friendly Explanation**

An API is like a waiter at a restaurant. You (the client) tell the waiter what you want, the waiter goes to the kitchen (your Laravel application), gets what you asked for, and brings it back to you. Laravel makes it easy to build these "waiters" so that mobile apps, websites, or other services can request data from your application in a structured way.

---

### Key Characteristics

- **Stateless Communication**: API routes do not use sessions or CSRF tokens; each request is independent and carries its own authentication credentials.
- **JSON-First Responses**: Data is exchanged in JSON format, transformed through Eloquent API Resources for consistency.
- **Token-Based Authentication**: Sanctum (lightweight) or Passport (OAuth2) authenticate API requests via Bearer tokens.
- **Versioned Endpoints**: APIs are versioned (`/api/v1/`, `/api/v2/`) to manage breaking changes without disrupting existing clients.
- **Rate Limiting**: Throttle middleware protects APIs from abuse by limiting requests per client.
- **Automatic Documentation**: Tools like Scribe and Scramble generate API documentation from code.
- **RESTful Conventions**: HTTP verbs (GET, POST, PUT, DELETE) and resource-based URIs map to CRUD operations.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A configured database** for storing data and API tokens.
- **Basic understanding of HTTP methods** and JSON.
- **Familiarity with Eloquent models and controllers**.

---

### Related Programming Areas

- **REST Architecture**: APIs follow REST (Representational State Transfer) principles for stateless, resource-oriented design.
- **Authentication**: Sanctum and Passport provide API authentication mechanisms.
- **Eloquent ORM**: API Resources transform Eloquent models into JSON responses.
- **Middleware**: Authentication, rate limiting, and versioning are implemented via middleware.
- **API Documentation**: Tools like Scribe and Scramble generate OpenAPI-compliant documentation.

---

## Core Concepts

---

### 1. APIs

#### Definitions

**Core Definition**

An API (Application Programming Interface) is a set of rules and protocols that allows different software applications to communicate with each other. In the context of Laravel, an API typically refers to an HTTP-based interface that exchanges JSON data.

**Technical Definition**

A Laravel API is a collection of routes defined in `routes/api.php` (or custom API route files) that handle HTTP requests without session state. API routes are assigned the `api` middleware group and are automatically prefixed with `/api`. Responses are typically JSON, generated via Eloquent API Resources. Authentication is handled via Bearer tokens (Sanctum or Passport). The `install:api` command installs Sanctum and creates the `api.php` route file.

**Beginner-Friendly Explanation**

An API is how your application talks to other applications. For example, a mobile app needs to get a list of products from your Laravel store. Instead of showing HTML pages, your Laravel app exposes an API endpoint like `/api/products` that returns JSON data, which the mobile app can display however it wants.

#### Purposes

- To provide a structured interface for external applications to access your application's data and functionality.
- To separate the backend logic from the frontend presentation, enabling multiple clients (web, mobile, third-party).
- To enable integration with other services and platforms.
- To support single-page applications (SPAs) and mobile apps that require data without full page reloads.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// routes/api.php
use Illuminate\Support\Facades\Route;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `routes/api.php` | Default API route file (created by `install:api`). |
| `api` middleware group | Stateless middleware group (no session, no CSRF). |
| `/api` prefix | Automatically applied to all API routes. |

**Syntax Rules**

- API routes are defined in `routes/api.php` or custom files loaded via `withRouting()`.
- API routes are stateless; they do not use sessions or CSRF tokens.
- The `/api` prefix is applied automatically.
- Responses should be JSON, ideally using API Resources.

**Constraints and Limitations**

- `routes/api.php` is not created by default in Laravel 11+; run `php artisan install:api`.
- API routes cannot use session-based authentication (use Sanctum tokens instead).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic API Endpoint**

**Route**

```php
// routes/api.php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/ping', function () {
    return response()->json(['message' => 'pong']);
});
```

**Expected Output**

```json
{"message":"pong"}
```

**Why This Output Occurs**

The route returns a JSON response directly. The `/api` prefix is applied automatically, so the endpoint is accessible at `/api/ping`.

---

**Example 2: Protected API Endpoint**

**Route**

```php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Expected Output (with valid token)**

```json
{"id":1,"name":"John Doe","email":"john@example.com"}
```

**Why This Output Occurs**

The `auth:sanctum` middleware validates the Bearer token and resolves the authenticated user.

---

#### Real-World Cases with Explanation

**Case 1: Mobile App Backend**

A mobile app communicates with a Laravel API to fetch product listings, submit orders, and manage user profiles. The API returns JSON data that the app renders natively.

**Explanation**: APIs decouple the backend from the frontend, enabling native mobile experiences.

---

### 2. REST APIs

#### Definitions

**Core Definition**

A REST API (Representational State Transfer API) is an API that follows REST architectural principles: stateless communication, resource-based URIs, standard HTTP verbs, and a uniform interface.

**Technical Definition**

REST APIs in Laravel use HTTP verbs (GET, POST, PUT/PATCH, DELETE) to perform CRUD operations on resources. Resource URIs are noun-based (e.g., `/api/posts`, `/api/posts/{id}`). The framework's `Route::apiResource()` method generates the five standard RESTful routes (index, store, show, update, destroy) without the HTML form routes (`create`, `edit`). Responses use appropriate HTTP status codes (200, 201, 204, 400, 401, 403, 404, 422).

**Beginner-Friendly Explanation**

REST is a set of rules for designing APIs. It is like a standard language that all developers understand. When you follow REST, your API is predictable: `GET /posts` gets all posts, `POST /posts` creates a post, `GET /posts/1` gets post 1, and so on.

#### Purposes

- To provide a predictable, standardized interface for API consumers.
- To leverage HTTP's built-in semantics for operations and status codes.
- To enable caching, statelessness, and scalability.
- To align with industry conventions, making APIs easier to learn and use.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// API resource routes (5 routes)
Route::apiResource('posts', PostController::class);

// With versioning
Route::prefix('v1')->group(function () {
    Route::apiResource('posts', PostController::class);
});
```

**RESTful Route Table**

| Verb | URI | Action | Status Code |
|------|-----|--------|-------------|
| GET | `/api/posts` | index | 200 |
| POST | `/api/posts` | store | 201 |
| GET | `/api/posts/{id}` | show | 200 |
| PUT/PATCH | `/api/posts/{id}` | update | 200 |
| DELETE | `/api/posts/{id}` | destroy | 204 |

**Syntax Rules**

- Use `apiResource()` for API routes (excludes `create` and `edit`).
- Resource names should be plural nouns.
- Use `PUT` for full updates, `PATCH` for partial updates.
- Return 201 for creation, 204 for deletion.

**Constraints and Limitations**

- REST is not a formal standard; conventions vary slightly.
- Complex operations may not map cleanly to CRUD verbs.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: RESTful API Resource**

**Controller**

```php
<?php
// app/Http/Controllers/Api/PostController.php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\PostResource;
use App\Models\Post;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class PostController extends Controller
{
    public function index()
    {
        return PostResource::collection(Post::all());
    }

    public function store(Request $request)
    {
        $post = Post::create($request->validated());
        return new PostResource($post);
    }

    public function show(Post $post)
    {
        return new PostResource($post);
    }

    public function update(Request $request, Post $post)
    {
        $post->update($request->validated());
        return new PostResource($post);
    }

    public function destroy(Post $post)
    {
        $post->delete();
        return response()->noContent();
    }
}
```

**Route**

```php
Route::apiResource('posts', PostController::class);
```

**Expected Output**

```
// GET /api/posts → 200 with JSON collection
// POST /api/posts → 201 with created post JSON
// GET /api/posts/1 → 200 with post JSON
// PUT /api/posts/1 → 200 with updated post JSON
// DELETE /api/posts/1 → 204 No Content
```

**Why This Output Occurs**

`apiResource()` generates the five standard RESTful routes, mapping HTTP verbs and URIs to the controller methods. The controller returns JSON responses with appropriate status codes.

---

#### Real-World Cases with Explanation

**Case 1: Blog API**

A blog platform exposes a REST API for posts and comments. Consumers can list, create, update, and delete posts via standard HTTP methods.

**Explanation**: RESTful design makes the API intuitive and consistent with industry standards.

---

### 3. API Resources

#### Definitions

**Core Definition**

API Resources are Laravel classes that transform Eloquent models into structured JSON responses, providing a transformation layer between the database and the API output.

**Technical Definition**

API Resources extend `Illuminate\Http\Resources\Json\JsonResource`. The `toArray()` method defines which model attributes are included in the response. Collections are handled via `Resource::collection()`. Resources support conditional attributes (`when()`, `mergeWhen()`), pagination, and nesting. Responses are wrapped in a `data` key by default, reserving other keys for metadata (errors, links).

**Beginner-Friendly Explanation**

API Resources are like a translator between your database and your API. They decide exactly what data gets sent to the client and in what format. Instead of sending all your model's columns, you choose only the fields you want to expose.

#### Purposes

- To control exactly which model attributes are returned in API responses.
- To transform attribute values (format dates, compute derived fields).
- To provide a consistent JSON response structure across all endpoints.
- To prevent accidental exposure of sensitive model attributes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Http/Resources/PostResource.php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
            'created_at' => $this->created_at->toISOString(),
        ];
    }
}
```

```php
// Controller usage
return new PostResource($post);
return PostResource::collection($posts);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends JsonResource` | Base class for API resources. |
| `toArray($request)` | Defines the response fields. |
| `$this` | The underlying model instance. |
| `Resource::collection()` | Transforms a collection of models. |

**Syntax Rules**

- Resources are created via `php artisan make:resource PostResource`.
- `toArray()` must return an array.
- Use `when()` for conditional fields.
- Use `Resource::collection()` for collections.

**Constraints and Limitations**

- Resources add a `data` wrapper by default (can be disabled).
- Nested resources can cause N+1 queries if not eager-loaded.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Post Resource**

**Resource**

```php
<?php
// app/Http/Resources/PostResource.php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
            'author' => $this->author->name,
        ];
    }
}
```

**Controller**

```php
public function show(Post $post)
{
    return new PostResource($post);
}
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "title": "My First Post",
        "body": "This is the content.",
        "author": "John Doe"
    }
}
```

**Why This Output Occurs**

The `PostResource` transforms the `Post` model into a JSON object with only the specified fields, wrapped in a `data` key.

---

**Example 2: Resource Collection**

```php
public function index()
{
    return PostResource::collection(Post::with('author')->paginate(10));
}
```

**Expected Output**

```json
{
    "data": [
        {"id": 1, "title": "Post 1", "author": "John"},
        {"id": 2, "title": "Post 2", "author": "Jane"}
    ],
    "links": {
        "first": "/api/posts?page=1",
        "last": "/api/posts?page=5",
        "prev": null,
        "next": "/api/posts?page=2"
    },
    "meta": {
        "current_page": 1,
        "total": 50
    }
}
```

**Why This Output Occurs**

`PostResource::collection()` transforms the paginated Eloquent collection, and Laravel automatically includes pagination links and metadata.

---

#### Real-World Cases with Explanation

**Case 1: Public API with Field Control**

A public API returns user data but must exclude sensitive fields like `email` and `password`. An API Resource includes only `id`, `name`, and `avatar_url`.

**Explanation**: API Resources provide a safety layer that prevents accidental data exposure.

---

### 4. API Routes

#### Definitions

**Core Definition**

API routes are route definitions that handle HTTP requests to API endpoints, defined in `routes/api.php` and assigned the stateless `api` middleware group.

**Technical Definition**

API routes are registered in `routes/api.php` (created via `php artisan install:api`) and are automatically prefixed with `/api`. They do not include session or CSRF middleware. In Laravel 11+, the API route file is loaded via the `withRouting()` method in `bootstrap/app.php`. API routes can use resource controllers (`apiResource()`), middleware groups (`auth:sanctum`, `throttle:api`), and version prefixes.

**Beginner-Friendly Explanation**

API routes are like the address book for your API. Each route tells Laravel: "When someone visits this URL with this HTTP method, run this code."

#### Purposes

- To define the endpoints that API clients can access.
- To apply API-specific middleware (authentication, rate limiting).
- To organize API routes separately from web routes.
- To support versioning and resource routing.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// routes/api.php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PostController;

Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('posts', PostController::class);
});

Route::get('/public', function () {
    return response()->json(['status' => 'ok']);
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `routes/api.php` | The API route file. |
| `api` middleware group | Stateless middleware (no session, no CSRF). |
| `/api` prefix | Automatically applied. |
| `auth:sanctum` | Middleware for token authentication. |

**Syntax Rules**

- API routes are defined in `routes/api.php`.
- The `/api` prefix is applied automatically.
- Use `apiResource()` for RESTful resources.
- Apply `auth:sanctum` to protect routes.

**Constraints and Limitations**

- API routes cannot use session-based features (flash messages, session data).
- The `api.php` file is not created by default in Laravel 11+.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Creating API Routes**

**Command**

```bash
php artisan install:api
```

**Route File**

```php
// routes/api.php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

**Expected Output**

```
// GET /api/user with valid token → user JSON
// GET /api/user without token → 401 Unauthenticated
```

**Why This Output Occurs**

The `install:api` command creates `routes/api.php` with a sample protected route.

---

**Example 2: Versioned API Routes**

```php
// routes/api.php
Route::prefix('v1')->group(function () {
    Route::apiResource('posts', V1\PostController::class);
});

Route::prefix('v2')->group(function () {
    Route::apiResource('posts', V2\PostController::class);
});
```

**Expected Output**

```
// GET /api/v1/posts → V1 PostController
// GET /api/v2/posts → V2 PostController
```

**Why This Output Occurs**

The `prefix('v1')` and `prefix('v2')` groups create versioned API endpoints.

---

#### Real-World Cases with Explanation

**Case 1: SPA with API Routes**

A Vue SPA communicates with a Laravel API via routes defined in `routes/api.php`. Authentication uses Sanctum tokens, and data is returned via API Resources.

**Explanation**: API routes provide the backend for SPA and mobile applications.

---

### 5. API Authentication

#### Definitions

**Core Definition**

API authentication is the process of verifying the identity of a client (user, application, or service) making API requests, typically via tokens.

**Technical Definition**

Laravel provides two first-party packages for API authentication: **Sanctum** (lightweight, token-based, for SPAs and mobile apps) and **Passport** (full OAuth2 server for third-party integrations). Sanctum stores hashed API tokens in the `personal_access_tokens` table and authenticates requests via the `Authorization: Bearer` header or stateful SPA cookies. Passport implements OAuth2 grant types (authorization code, password, client credentials) using the League OAuth2 server.

**Beginner-Friendly Explanation**

API authentication is how your API knows who is making a request. Sanctum is the simple option: users get a token, and they include it in every request. Passport is the complex option used when you need to let other companies' apps access your API on behalf of users (like "Log in with Google").

#### Purposes

- To verify the identity of API clients before granting access.
- To protect sensitive data and operations from unauthorized access.
- To enable stateless authentication suitable for distributed systems.
- To support different client types (mobile, SPA, third-party).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Issuing a Sanctum token
$token = $user->createToken('mobile-app')->plainTextToken;

// Protecting routes
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `createToken()` | Issues a new API token. |
| `plainTextToken` | The unhashed token string. |
| `auth:sanctum` | Middleware for token authentication. |
| `Authorization: Bearer` | Header for sending the token. |

**Syntax Rules**

- The `HasApiTokens` trait must be added to the `User` model.
- Sanctum is installed via `php artisan install:api`.
- Tokens are sent in the `Authorization: Bearer` header.
- Passport uses the `passport` driver and `auth:api` middleware.

**Constraints and Limitations**

- Sanctum does not support OAuth2.
- Passport is more complex to set up than Sanctum.
- Tokens should be transmitted over HTTPS only.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Sanctum Token Issuance**

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

The token is generated, hashed, and stored. The plain-text token is returned to the client for use in subsequent requests.

---

**Example 2: Protected API Route**

```php
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

**Explanation**: Sanctum is the recommended solution for first-party mobile apps and SPAs.

**Case 2: Third-Party OAuth2 Integration**

A platform allows external developers to build integrations using OAuth2. Passport provides the authorization server, enabling the authorization code flow with PKCE.

**Explanation**: Passport is required when OAuth2 compliance is necessary.

---

### 6. API Versioning

#### Definitions

**Core Definition**

API versioning is the practice of maintaining multiple versions of an API simultaneously, allowing clients to continue using older versions while newer versions introduce changes.

**Technical Definition**

Versioning can be implemented via URI path (`/api/v1/posts`), request headers (`X-API-Version: 1` or `Accept: application/vnd.api.v1+json`), or query parameters (`?version=1`). URI path versioning is the most common and easiest to implement in Laravel via route groups with `prefix('v1')`. Versioning should be introduced from the beginning, even if V2 is not yet planned.

**Beginner-Friendly Explanation**

API versioning is like having different editions of a book. When you make major changes to your API that would break existing apps, you create a new version (V2) while keeping the old version (V1) running. Clients can choose which version to use.

#### Purposes

- To introduce breaking changes without disrupting existing clients.
- To maintain backward compatibility for older applications.
- To allow gradual migration from one version to another.
- To manage the API lifecycle professionally.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// URI path versioning
Route::prefix('v1')->group(function () {
    Route::apiResource('posts', V1\PostController::class);
});

Route::prefix('v2')->group(function () {
    Route::apiResource('posts', V2\PostController::class);
});
```

**Component Breakdown**

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| URI Path | `/api/v1/posts` | Simple, explicit, cacheable | URL clutter |
| Header | `X-API-Version: 1` | Clean URLs | Harder to debug |
| Accept Header | `Accept: application/vnd.api.v1+json` | RESTful | Complex to implement |

**Syntax Rules**

- Use route groups with `prefix()` for URI versioning.
- Organize controllers into versioned namespaces (`V1`, `V2`).
- Always version from day one.

**Constraints and Limitations**

- Maintaining multiple versions increases code complexity.
- Deprecation policies should be documented.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: URI Path Versioning**

```php
// routes/api.php
Route::prefix('v1')->group(function () {
    Route::apiResource('posts', App\Http\Controllers\Api\V1\PostController::class);
});

Route::prefix('v2')->group(function () {
    Route::apiResource('posts', App\Http\Controllers\Api\V2\PostController::class);
});
```

**Expected Output**

```
// GET /api/v1/posts → V1 controller (returns old format)
// GET /api/v2/posts → V2 controller (returns new format)
```

**Why This Output Occurs**

Each version group routes to a different controller namespace, allowing independent evolution.

---

**Example 2: Versioned Controllers**

```php
// app/Http/Controllers/Api/V1/PostController.php
namespace App\Http\Controllers\Api\V1;

class PostController extends Controller
{
    public function index()
    {
        return PostResource::collection(Post::all());
    }
}
```

**Expected Output**

```
// V1 returns posts with only id and title
// V2 returns posts with id, title, body, author, and created_at
```

**Why This Output Occurs**

Versioned controllers and resources allow different response structures for each API version.

---

#### Real-World Cases with Explanation

**Case 1: Public API with Mobile Clients**

A public API is used by mobile apps that cannot be forced to upgrade. V1 remains stable while V2 introduces new features. Both versions run simultaneously.

**Explanation**: Versioning protects existing clients from breaking changes.

---

### 7. API Rate Limiting

#### Definitions

**Core Definition**

API rate limiting restricts the number of requests a client can make to an API within a specified time window, protecting the API from abuse and ensuring fair usage.

**Technical Definition**

Laravel provides the `throttle` middleware for rate limiting. The format is `throttle:maxAttempts,decayMinutes` (e.g., `throttle:60,1` for 60 requests per minute). Named rate limiters can be defined via the `RateLimiter` facade in a service provider, allowing dynamic limits based on user tier, IP, or other criteria. The default `api` middleware group includes `throttle:api` in Laravel 11+.

**Beginner-Friendly Explanation**

Rate limiting is like a bouncer who only lets you into a club a certain number of times per minute. It prevents anyone from overwhelming your API with too many requests.

#### Purposes

- To protect APIs from abuse, brute-force attacks, and denial-of-service.
- To ensure fair resource allocation among all clients.
- To implement tiered rate limits (free, premium, enterprise).
- To return informative 429 responses with retry headers.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Simple throttle
Route::middleware('throttle:60,1')->group(function () {
    Route::get('/posts', [PostController::class, 'index']);
});

// Named limiter
Route::middleware('throttle:api')->group(function () {
    Route::get('/posts', [PostController::class, 'index']);
});
```

```php
// Defining a named limiter (AppServiceProvider)
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `throttle:60,1` | 60 requests per 1 minute. |
| `throttle:api` | Uses the named `api` limiter. |
| `RateLimiter::for()` | Defines a named limiter. |
| `Limit::perMinute()` | Creates a per-minute limit. |

**Syntax Rules**

- The `throttle` middleware accepts `limit,decay` or a named limiter.
- Named limiters are defined in `AppServiceProvider`.
- The `by()` method customizes the key (user ID, IP).
- 429 status is returned when the limit is exceeded.

**Constraints and Limitations**

- Rate limiting requires a cache driver (Redis recommended).
- Named limiters must be registered before routes.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Throttling**

```php
Route::middleware('throttle:60,1')->group(function () {
    Route::get('/posts', [PostController::class, 'index']);
});
```

**Expected Output**

```
// First 60 requests in a minute → 200 OK
// 61st request → 429 Too Many Requests
```

**Why This Output Occurs**

The throttle middleware allows 60 requests per minute per client.

---

**Example 2: Dynamic Rate Limiting**

```php
// AppServiceProvider.php
RateLimiter::for('premium-api', function (Request $request) {
    if ($request->user()?->isPremium()) {
        return Limit::perMinute(1000)->by($request->user()->id);
    }
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

**Expected Output**

```
// Premium users: 1000 requests per minute
// Free users: 60 requests per minute
```

**Why This Output Occurs**

The named limiter dynamically adjusts the limit based on the user's subscription tier.

---

#### Real-World Cases with Explanation

**Case 1: SaaS API Tiers**

A SaaS platform offers different rate limits for free (60/min), pro (1000/min), and enterprise (10000/min) plans.

**Explanation**: Rate limiting enforces fair usage and monetization strategies.

---

### 8. API Documentation

#### Definitions

**Core Definition**

API documentation is structured, human-readable documentation that describes an API's endpoints, request/response formats, authentication requirements, and usage examples.

**Technical Definition**

In Laravel, API documentation is typically generated from code using tools like **Scribe** (annotation-based, generates HTML docs) or **Scramble** (zero-annotation, generates OpenAPI 3.1 specs). Scribe uses PHPDoc annotations and can call real API endpoints to generate response examples. Scramble infers documentation from routes, controllers, Form Requests, and API Resources without requiring annotations.

**Beginner-Friendly Explanation**

API documentation is the instruction manual for your API. It tells developers how to use your API: what URLs to call, what data to send, and what responses to expect.

#### Purposes

- To help developers understand and use your API.
- To reduce support requests by providing self-service documentation.
- To enable API testing via interactive documentation (Swagger UI).
- To serve as a contract between the API provider and consumers.

#### Syntax Rules and Structure

**Complete General Syntax**

**Scribe:**

```bash
composer require knuckleswtf/scribe
php artisan vendor:publish --tag=scribe-config
php artisan scribe:generate
```

**Scramble:**

```bash
composer require dedoc/scramble
# Access /docs/api in the browser
```

**Component Breakdown**

| Tool | Approach | Output |
|------|----------|--------|
| Scribe | PHPDoc annotations | HTML docs, Postman collection |
| Scramble | Zero-annotation inference | OpenAPI 3.1 spec, Swagger UI |

**Syntax Rules**

- Scribe requires annotations (`@response`, `@queryParam`).
- Scramble requires no annotations, inferring from code.
- Both tools support OpenAPI generation.

**Constraints and Limitations**

- Scribe requires manual annotation for complete documentation.
- Scramble may not capture all edge cases without hints.
- Documentation must be regenerated when code changes.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Scribe Documentation**

**Annotation**

```php
/**
 * @response {
 *   "data": {
 *     "id": 1,
 *     "title": "My Post",
 *     "body": "Content"
 *   }
 * }
 */
public function show(Post $post)
{
    return new PostResource($post);
}
```

**Command**

```bash
php artisan scribe:generate
```

**Expected Output**

```
// Documentation generated at /docs
// Interactive HTML with request/response examples
```

**Why This Output Occurs**

Scribe reads the annotation and generates documentation for the `show` endpoint.

---

**Example 2: Scramble Documentation**

**Command**

```bash
composer require dedoc/scramble
```

**Expected Output**

```
// Access /docs/api → auto-generated OpenAPI documentation
```

**Why This Output Occurs**

Scramble analyzes the route, controller, and resource to generate documentation without annotations.

---

#### Real-World Cases with Explanation

**Case 1: Developer Portal**

A SaaS platform uses Scribe to generate a developer portal with interactive API documentation, enabling third-party developers to integrate quickly.

**Explanation**: Auto-generated documentation reduces maintenance burden and improves developer experience.

---

### 9. API Best Practices

#### Definitions

**Core Definition**

API best practices are the recommended guidelines for designing, building, and maintaining production-ready APIs using Laravel.

**Technical Definition**

Best practices encompass route organization, controller design, request validation, response transformation, authentication, versioning, rate limiting, error handling, and testing.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to build APIs that are secure, maintainable, and easy for others to use.

#### Key Best Practices

1. **Version from Day One**: Always introduce `/v1/` in API URLs from the start, even if V2 is not planned.

2. **Use API Resources for All Responses**: Transform models through `JsonResource` classes for consistent JSON structure.

3. **Keep Controllers Thin**: Delegate business logic to service or action classes. Controllers should only handle HTTP concerns.

4. **Use Form Requests for Validation**: Never validate directly in controllers. Use dedicated `FormRequest` classes.

5. **Apply Rate Limiting to All API Routes**: Use `throttle:api` middleware to protect against abuse.

6. **Use Sanctum for First-Party APIs, Passport for OAuth2**: Sanctum is simpler and sufficient for most applications. Use Passport only when OAuth2 is required.

7. **Return Appropriate HTTP Status Codes**: 200 for success, 201 for creation, 204 for deletion, 400 for bad requests, 401 for unauthenticated, 403 for forbidden, 404 for not found, 422 for validation errors.

8. **Use Named Routes with Version Prefix**: Name routes like `v1:posts:index` for clarity and consistency.

9. **Eager Load Relationships**: Use `with()` to prevent N+1 queries in API responses.

10. **Document Your API**: Use Scribe or Scramble to generate and maintain API documentation.

11. **Test API Endpoints**: Write feature tests for all API endpoints, verifying status codes, JSON structure, and database state.

12. **Never Trust Request Payloads**: Validate and sanitize all input. Never use request data for derived fields without server-side computation.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Well-Structured API Controller**

```php
<?php
declare(strict_types=1);

namespace App\Http\Controllers\Api\V1;

use App\Actions\Posts\StorePostAction;
use App\Http\Requests\Api\V1\StorePostRequest;
use App\Http\Resources\PostResource;
use Illuminate\Http\JsonResponse;
use Symfony\Component\HttpFoundation\Response;

final class StoreController
{
    public function __construct(
        private readonly StorePostAction $action,
    ) {}

    public function __invoke(StorePostRequest $request): JsonResponse
    {
        $post = $this->action->handle($request->validated());

        return new JsonResponse(
            data: new PostResource($post),
            status: Response::HTTP_CREATED,
        );
    }
}
```

**Route**

```php
Route::prefix('v1')->middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::post('/posts', StoreController::class)->name('v1:posts:store');
});
```

**Expected Output**

```
// POST /api/v1/posts with valid token and data → 201 Created with post JSON
// POST /api/v1/posts without token → 401 Unauthenticated
// POST /api/v1/posts with invalid data → 422 Unprocessable Entity
```

**Why This Output Occurs**

The controller is thin (delegates to an action), validation is handled by a form request, the response is transformed via a resource, and the route is versioned, authenticated, and rate-limited.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise API Platform**

A large organization builds a public API following all best practices: versioning, Sanctum authentication, rate limiting, API Resources, Scribe documentation, and comprehensive tests.

**Explanation**: Best practices ensure the API is secure, scalable, and maintainable as it grows.

---

## References

- Routing – Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/routing
- Eloquent API Resources – Laravel 13.x Documentation — https://laravel.com/docs/eloquent-resources
- Laravel Sanctum – Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/sanctum
- Laravel Passport – Laravel 12.x Documentation — https://laravel.com/docs/12.x/passport
- Rate Limiting – Laravel 12.x Documentation — https://laravel.com/docs/12.x/routing#rate-limiting
- API Skill for Laravel Developers (JustSteveKing) — https://github.com/JustSteveKing/api-skill/blob/HEAD/SKILL.md
- Laravel API Versioning: All You Need To Know About V1/V2 (Laravel Daily) — https://laraveldaily.com/post/laravel-api-versioning-v1-v2
- Getting Single Record and API Resources (Laravel Daily) — https://laraveldaily.com/lesson/api-laravel/getting-single-record-and-api-resources
- How to Implement Rate Limiting in Laravel (OneUptime) — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-02-03-laravel-rate-limiting/README.md
- Laravel Sanctum vs Passport: Which One Should You Use? (Bagisto) — https://bagisto.com/en/laravel-sanctum-vs-passport-which-one-should-you-use/
- Scribe – Laravel API Documentation Generator — https://scribe.knuckles.wtf/laravel
- Scramble – Laravel OpenAPI Documentation Generator — https://scramble.dedoc.co

---

**Version Note**: This cheat sheet is based on Laravel 11.x through 13.x documentation. Key version-specific changes include: Laravel 11 introduced the `install:api` command, which installs Sanctum and creates `routes/api.php` in one step; the `api` middleware group now includes `throttle:api` by default; Scramble has emerged as a popular zero-annotation documentation tool alongside Scribe. In Laravel 10 and earlier, Sanctum and Passport were installed via `composer require`. Developers should always consult the documentation for their specific Laravel version.