# Laravel API Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel API Fundamentals refers to the set of architectural principles, conventions, and framework tools used to build HTTP-based application programming interfaces (APIs) in Laravel that conform to REST (Representational State Transfer) principles and exchange data in JSON format.

**Technical Definition:** Laravel API Fundamentals encompasses the implementation of REST architectural constraints (statelessness, client-server separation, uniform interface, cacheability, layered system) within the Laravel framework using its routing system (`routes/api.php`), Eloquent API Resources for JSON transformation, HTTP verb handling, standardized status code responses, and JSON payload formatting compliant with RFC 8259. Laravel's API routes are stateless by default (no session or CSRF middleware) and are prefixed with `/api`, making them suitable for consumption by mobile applications, single-page applications (SPAs), and third-party services.

**Beginner-Friendly Explanation:** When you build an API, you are building a way for two computer programs to talk to each other. One program (the client — a mobile app, a website, or another server) sends a request, and your Laravel application (the server) sends back a response. REST is a set of rules that makes this conversation predictable and organized: you use specific URLs to talk about specific "things" (resources), specific HTTP verbs to say what you want to do with them, and specific numeric codes to say what happened. Laravel provides tools that make following these rules straightforward and consistent.

### Key Characteristics

- **Statelessness:** Each request contains all information needed to process it; the server does not store client context between requests.
- **Resource-oriented URIs:** URLs represent nouns (resources), not actions.
- **HTTP verb semantics:** GET, POST, PUT/PATCH, and DELETE map to read, create, update, and delete operations.
- **JSON as the interchange format:** Requests and responses use `application/json` as the media type.
- **Standardized status codes:** Success and failure are communicated through HTTP status codes (2xx, 4xx, 5xx).
- **Framework support:** Laravel provides `Route::apiResource()`, Eloquent API Resources, validation with automatic 422 responses, and rate limiting out of the box.
- **Versioning capability:** API routes can be versioned via URL prefixes (e.g., `/api/v1/`).

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- Composer dependency manager.
- A Laravel application with the `routes/api.php` file present.
- Basic understanding of HTTP protocol and JSON syntax.
- Familiarity with Eloquent models and database migrations.
- (For testing) A tool such as cURL, Postman, or HTTPie.

### Related Programming Areas

- **HTTP Protocol** — The transport layer for all API communication (RFC 9110).
- **REST Architectural Style** — The design philosophy underlying resource-oriented APIs (Fielding, 2000).
- **JSON Data Interchange** — The payload format standardized in RFC 8259.
- **Eloquent ORM** — The database abstraction layer used to retrieve and persist resources.
- **Laravel Sanctum / Passport** — Authentication packages for securing API endpoints.
- **API Versioning** — The practice of maintaining multiple API versions simultaneously.
- **HATEOAS** — Hypermedia as the Engine of Application State, an advanced REST constraint.

### Core Concepts / Features

1. REST Architecture: Statelessness, Client-Server Separation, Uniform Interface
2. Resource-Oriented Endpoints: Naming Conventions, Plural Nouns, Nested Resource Design
3. HTTP Methods: GET, POST, PUT, PATCH, DELETE
4. HTTP Status Codes: 200, 201, 204, 400, 401, 403, 404, 422, 500
5. JSON: Data Payloads, Media Types, Structural Consistency

---

## 1. REST Architecture

### Definitions

**Core Definition:** REST (Representational State Transfer) is an architectural style for distributed systems that defines a set of constraints governing how clients and servers communicate, centred on the concept of "resources" identified by URIs.

**Technical Definition:** REST is an architectural style introduced by Roy Fielding in his 2000 doctoral dissertation. It defines six constraints: (1) Client-Server separation, (2) Statelessness, (3) Cacheability, (4) Uniform Interface, (5) Layered System, and (6) Code-on-Demand (optional). When these constraints are applied to an HTTP-based API, the result is a RESTful API — one where resources are identified by URIs, manipulated through a uniform interface (HTTP methods), and represented in a standard format (JSON). An API that satisfies all mandatory REST constraints is said to be "RESTful".

**Beginner-Friendly Explanation:** REST is a set of guidelines for how websites and apps should talk to each other over the internet. Think of it like a restaurant: you (the client) look at a menu (the API documentation), place an order using a standard format (an HTTP request), and the kitchen (the server) prepares your dish and sends it back. The kitchen does not remember you between orders — every order must be complete and self-contained. This "no memory" rule (statelessness) is what makes REST APIs so scalable: any waiter can handle any order.

### Purposes

- To provide a consistent, predictable interface for clients to interact with server-side resources.
- To enable stateless communication so that any server instance can handle any request, facilitating horizontal scaling.
- To separate the concerns of the client (user interface) from the server (data storage and business logic).
- To support multiple client types (browsers, mobile apps, third-party services) through a uniform interface.
- To enable caching of responses at various layers, improving performance and reducing server load.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel API Route)

```php
// routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PostController;

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider and all of them will
| be assigned the "api" middleware group. Make something great!
*/

Route::apiResource('posts', PostController::class);
```

**Component Breakdown:**

- `routes/api.php` — The dedicated route file for API endpoints. Routes here are automatically prefixed with `/api` and assigned the `api` middleware group (which does **not** include session state or CSRF protection, enforcing statelessness).
- `Route::apiResource()` — Registers the five standard RESTful routes for a resource: `index`, `store`, `show`, `update`, and `destroy` (excludes `create` and `edit`, which are HTML form routes).
- `PostController::class` — The controller that handles the resource operations.

**Syntax Rules:**

- API routes must be defined in `routes/api.php`; they are automatically prefixed with `/api` (configurable via `RouteServiceProvider`).
- The `api` middleware group does not include `StartSession` or `VerifyCsrfToken`, which enforces statelessness.
- Route model binding is automatically applied when the controller method type-hints an Eloquent model.
- For Laravel 11+, the `api` routes are registered by default only if the application was created with the `--api` flag or the `install:api` command was run.

**Constraints and Limitations:**

- **Statelessness is broken if you introduce server-side sessions.** If your API relies on session cookies or server-stored context, it ceases to be RESTful. Use token-based authentication (Sanctum, Passport) instead.
- **REST is not a standard.** It is an architectural style; there is no RFC that mandates REST compliance. Different APIs claiming to be "RESTful" may interpret the constraints differently.
- **Pure REST requires HATEOAS** (Hypermedia as the Engine of Application State), which most practical APIs do not fully implement. Such APIs are sometimes called "REST-like" or "HTTP+JSON" rather than truly RESTful.

### Annotated Code Examples

**Example 1: A Minimal RESTful API Route with Stateless Authentication**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PostController;

// Step 1: Define a public health-check endpoint
// This endpoint does not require authentication and is stateless.
Route::get('/health', function () {
    return response()->json([
        'status' => 'ok',
        'timestamp' => now()->toIso8601String(),
    ]);
});

// Step 2: Define resource routes protected by token authentication.
// Sanctum's 'auth:sanctum' middleware validates the Bearer token
// on each request without relying on server-side session state.
Route::middleware('auth:sanctum')->group(function () {
    // Step 3: Register all five RESTful routes for the Post resource.
    // GET    /api/posts         → index   (list all posts)
    // POST   /api/posts         → store   (create a new post)
    // GET    /api/posts/{post}  → show    (retrieve a single post)
    // PUT    /api/posts/{post}  → update  (replace a post)
    // PATCH  /api/posts/{post}  → update  (partially modify a post)
    // DELETE /api/posts/{post}  → destroy (delete a post)
    Route::apiResource('posts', PostController::class);
});
```

**Step-by-Step Setup:**

1. Ensure `routes/api.php` exists. In Laravel 11+, run `php artisan install:api` if it does not.
2. Install Laravel Sanctum: `composer require laravel/sanctum`.
3. Run `php artisan migrate` to create the `personal_access_tokens` table.
4. Add the `HasApiTokens` trait to your `User` model.
5. Create the `PostController` with the five standard methods.

**Expected Output:**

- `GET /api/health` returns `{"status":"ok","timestamp":"2025-06-15T10:00:00+00:00"}` without requiring authentication.
- `GET /api/posts` without a valid Bearer token returns `401 Unauthorized`.
- `GET /api/posts` with a valid token returns a JSON array of posts.
- `POST /api/posts` with valid data and a token creates a post and returns `201 Created`.

**Why This Output Occurs:** The `api` middleware group excludes session and CSRF middleware, making the routes stateless. The `auth:sanctum` middleware authenticates each request independently by validating the Bearer token against the database. No session cookie or server-side context is stored between requests. The `apiResource` call generates the five standard RESTful routes, each mapped to the corresponding controller method.

---

**Example 2: A Custom RESTful Route Demonstrating Client-Server Separation**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\ReportController;

// The client (SPA or mobile app) requests a report.
// The server generates it and returns JSON. The client is
// responsible for rendering it; the server does not generate HTML.
Route::get('/reports/sales', [ReportController::class, 'sales']);
```

```php
<?php
// File: app/Http/Controllers/Api/ReportController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Order;
use Illuminate\Http\JsonResponse;

class ReportController extends Controller
{
    public function sales(): JsonResponse
    {
        // Step 1: Aggregate data on the server
        $total = Order::where('created_at', '>=', now()->startOfMonth())
            ->sum('total_amount');

        $byDay = Order::where('created_at', '>=', now()->startOfMonth())
            ->selectRaw('DATE(created_at) as date, SUM(total_amount) as total')
            ->groupBy('date')
            ->orderBy('date')
            ->get();

        // Step 2: Return structured JSON — the client handles presentation
        return response()->json([
            'period' => now()->format('F Y'),
            'total_sales' => $total,
            'daily_breakdown' => $byDay,
        ]);
    }
}
```

**Expected Output:**

```json
{
    "period": "June 2025",
    "total_sales": 45230.50,
    "daily_breakdown": [
        {"date": "2025-06-01", "total": 1200.00},
        {"date": "2025-06-02", "total": 980.50}
    ]
}
```

**Why This Output Occurs:** The server's only responsibility is to query the database and return structured data. It does not generate HTML, format charts, or handle any presentation logic. The client (a Vue, React, or mobile app) receives the JSON and renders the report. This separation of concerns is the Client-Server constraint of REST: the user interface is entirely the client's concern, while data storage and business logic are the server's concern.

### Real-World Cases

- **Mobile application backends:** A Laravel API serves JSON to iOS and Android apps. The same endpoints serve both platforms without modification, demonstrating the uniform interface.
- **Single-page applications (SPAs):** A Vue or React frontend consumes a Laravel API hosted on a separate domain. Authentication uses Sanctum tokens instead of session cookies, maintaining statelessness.
- **Microservices communication:** Internal services call each other's APIs using stateless HTTP requests, each request carrying its own authentication token and data payload.
- **Third-party integrations:** External partners consume your API to retrieve or modify data programmatically, relying on the uniform interface and standard HTTP semantics.

---

## 2. Resource-Oriented Endpoints

### Definitions

**Core Definition:** Resource-oriented endpoints are URI patterns that identify "resources" (entities or collections) using nouns, with HTTP methods expressing the action to be performed on those resources.

**Technical Definition:** In a resource-oriented API design, each URI uniquely identifies a resource or a collection of resources. The URI itself contains only nouns (e.g., `/users`, `/posts/5/comments`), and the operation to be performed is determined entirely by the HTTP method. Laravel's `Route::apiResource()` method generates the standard set of resource endpoints following this convention, mapping plural resource names to controller actions.

**Beginner-Friendly Explanation:** Instead of creating URLs like `/getAllUsers` or `/deleteUser?id=5` (which mix verbs into the URL), RESTful APIs use clean, noun-based URLs like `/users` and `/users/5`. The URL says *what* you are working with, and the HTTP method says *what you want to do*. This makes URLs predictable and self-explanatory.

### Purposes

- To create predictable, self-documenting URL structures that developers can infer without consulting documentation.
- To separate the identification of a resource (the URI) from the operation on that resource (the HTTP method).
- To enable uniform caching and routing behaviour across different resources.
- To support nested relationships (e.g., comments belonging to posts) through hierarchical URIs.
- To align API design with the underlying domain model and database schema.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// routes/api.php — Standard resource routes
Route::apiResource('posts', PostController::class);

// Nested resource routes
Route::apiResource('posts.comments', CommentController::class);

// Shallow nested resource routes
Route::apiResource('posts.comments', CommentController::class)->shallow();

// Only / except for specific routes
Route::apiResource('posts', PostController::class)->only(['index', 'show']);
Route::apiResource('posts', PostController::class)->except(['destroy']);
```

**Component Breakdown:**

- `Route::apiResource('posts', PostController::class)` — Generates five routes using the plural resource name `posts` as the URI segment.
- `Route::apiResource('posts.comments', ...)` — Generates nested routes where comments are scoped to a specific post: `/posts/{post}/comments` and `/posts/{post}/comments/{comment}`.
- `->shallow()` — Generates nested `index` and `store` routes but non-nested `show`, `update`, and `destroy` routes (e.g., `/comments/{comment}` instead of `/posts/{post}/comments/{comment}`).
- `->only(['index', 'show'])` — Restricts the generated routes to only those listed.
- `->except(['destroy'])` — Excludes the listed routes.

**Generated Route Table for `apiResource('posts', PostController::class)`:**

| Verb | URI | Action | Route Name |
|------|-----|--------|------------|
| GET | `/api/posts` | index | posts.index |
| POST | `/api/posts` | store | posts.store |
| GET | `/api/posts/{post}` | show | posts.show |
| PUT/PATCH | `/api/posts/{post}` | update | posts.update |
| DELETE | `/api/posts/{post}` | destroy | posts.destroy |

**Syntax Rules:**

- Resource names in URIs must be **plural nouns** (e.g., `users`, `posts`, `orders`). Laravel does not enforce this, but it is a REST convention.
- Route parameters are automatically bound to Eloquent models via **route model binding** when the controller method type-hints the model class.
- Nested resources use dot notation: `posts.comments` generates `/posts/{post}/comments`.
- The `shallow()` method must be called before any `only()` or `except()` methods.

**Constraints and Limitations:**

- **Over-nesting is discouraged.** Deeply nested resources (e.g., `/users/{user}/posts/{post}/comments/{comment}/replies`) become unwieldy. REST practitioners often recommend limiting nesting to one or two levels and using query parameters for deeper relationships.
- **Route model binding for nested resources requires the parent relationship to exist.** If a comment does not belong to the specified post, a `404 Not Found` is returned.
- **Resource routes assume a controller exists.** If the controller does not implement all five methods, calling an unimplemented route results in a `500 Internal Server Error` (or a `MethodNotAllowed` exception depending on the route registration).

### Annotated Code Examples

**Example 1: Standard Resource Routes with a Controller**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\ProductController;

// Generates all five RESTful routes for products.
Route::apiResource('products', ProductController::class);
```

```php
<?php
// File: app/Http/Controllers/Api/ProductController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class ProductController extends Controller
{
    // GET /api/products
    public function index(): JsonResponse
    {
        return response()->json(Product::paginate(15));
    }

    // POST /api/products
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'  => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
        ]);

        $product = Product::create($validated);

        return response()->json($product, 201);
    }

    // GET /api/products/{product}
    public function show(Product $product): JsonResponse
    {
        return response()->json($product);
    }

    // PUT/PATCH /api/products/{product}
    public function update(Request $request, Product $product): JsonResponse
    {
        $validated = $request->validate([
            'name'  => 'sometimes|string|max:255',
            'price' => 'sometimes|numeric|min:0',
        ]);

        $product->update($validated);

        return response()->json($product);
    }

    // DELETE /api/products/{product}
    public function destroy(Product $product): JsonResponse
    {
        $product->delete();

        return response()->json(null, 204);
    }
}
```

**Step-by-Step Setup:**

1. Create the `products` table migration with `name` and `price` columns.
2. Create the `Product` Eloquent model.
3. Generate the controller: `php artisan make:controller Api/ProductController --api`.
4. Define the route in `routes/api.php` as shown.
5. Seed the database with sample products.

**Expected Output:**

- `GET /api/products` → `200 OK` with a paginated JSON list of products.
- `POST /api/products` with `{"name": "Widget", "price": 9.99}` → `201 Created` with the new product JSON.
- `GET /api/products/1` → `200 OK` with the product JSON.
- `PUT /api/products/1` with `{"name": "Updated Widget"}` → `200 OK` with the updated product.
- `DELETE /api/products/1` → `204 No Content` with an empty body.

**Why This Output Occurs:** The `apiResource` method registers the five routes with the correct HTTP verbs and URIs. The `Product $product` parameter in `show`, `update`, and `destroy` triggers route model binding, which automatically resolves the `{product}` segment to a `Product` model instance (or returns a 404 if not found). The controller methods return JSON responses with the appropriate status codes.

---

**Example 2: Nested Resources with Shallow Routing**

```php
<?php
// File: routes/api.php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PostController;
use App\Http\Controllers\Api\CommentController;

// Nested routes: comments are always accessed in the context of a post
Route::apiResource('posts', PostController::class);

// Shallow nested routes: index and store require a post,
// but show, update, and destroy operate on the comment directly.
Route::apiResource('posts.comments', CommentController::class)->shallow();
```

**Generated Routes with `shallow()`:**

| Verb | URI | Action |
|------|-----|--------|
| GET | `/api/posts/{post}/comments` | index |
| POST | `/api/posts/{post}/comments` | store |
| GET | `/api/comments/{comment}` | show |
| PUT/PATCH | `/api/comments/{comment}` | update |
| DELETE | `/api/comments/{comment}` | destroy |

**Expected Output:**

- `GET /api/posts/1/comments` → `200 OK` with all comments belonging to post 1.
- `POST /api/posts/1/comments` → `201 Created` with the new comment.
- `GET /api/comments/5` → `200 OK` with comment 5 (no post ID required).
- `DELETE /api/comments/5` → `204 No Content`.

**Why This Output Occurs:** The shallow routing strategy generates the `index` and `store` routes with the full nested URI (because creating or listing comments requires knowing which post they belong to), but the `show`, `update`, and `destroy` routes use a flat URI (because a comment's ID is globally unique). This avoids unnecessarily long URLs like `/posts/1/comments/5` for operations that do not need the post context.

### Real-World Cases

- **Blog platforms:** `/posts` for articles, `/posts/{post}/comments` for comments on a specific article.
- **E-commerce systems:** `/products`, `/products/{product}/reviews`, `/orders/{order}/items`.
- **Project management tools:** `/projects/{project}/tasks`, `/projects/{project}/members`.
- **Social networks:** `/users/{user}/posts`, `/users/{user}/followers`.

---

## 3. HTTP Methods

### Definitions

**Core Definition:** HTTP methods (also called HTTP verbs) are the standard actions defined by the HTTP protocol that indicate the desired operation to be performed on a resource identified by a URI.

**Technical Definition:** HTTP methods are defined in RFC 9110 (which obsoletes RFC 7231). The five methods most commonly used in RESTful APIs are GET, POST, PUT, PATCH, and DELETE. Each method has defined semantics regarding safety (whether it is read-only) and idempotency (whether repeating the request produces the same result). GET and DELETE are idempotent; PUT is idempotent; POST is not idempotent; PATCH is not necessarily idempotent.

**Beginner-Friendly Explanation:** HTTP methods are like the verbs in a sentence. The URL is the noun ("the users"), and the method is the verb ("get", "create", "update", "delete"). When you say `GET /users`, you are asking to read the list of users. When you say `POST /users`, you are asking to create a new user. Using the correct method matters because it tells the server — and any caching system in between — exactly what you intend to do.

### Purposes

- To communicate the intended action on a resource using a standardized, universally understood vocabulary.
- To enable idempotent operations (PUT, DELETE) that can be safely retried without unintended side effects.
- To allow caching systems to automatically cache safe, read-only requests (GET).
- To support partial updates (PATCH) without requiring the client to send the entire resource.
- To provide a uniform interface across all resources, reducing the need for custom action URLs.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// routes/api.php — Explicit method routes
Route::get('/posts', [PostController::class, 'index']);        // Read all
Route::get('/posts/{post}', [PostController::class, 'show']);   // Read one
Route::post('/posts', [PostController::class, 'store']);        // Create
Route::put('/posts/{post}', [PostController::class, 'update']); // Replace
Route::patch('/posts/{post}', [PostController::class, 'update']); // Partial update
Route::delete('/posts/{post}', [PostController::class, 'destroy']); // Delete
```

| Method | Semantics | Safe? | Idempotent? | Request Body? | Response Body? |
|--------|-----------|-------|-------------|---------------|----------------|
| GET | Retrieve a representation | Yes | Yes | No (should be ignored) | Yes |
| POST | Create a new resource | No | No | Yes | Yes |
| PUT | Replace a resource entirely | No | Yes | Yes | Optional |
| PATCH | Partially modify a resource | No | No (not guaranteed) | Yes | Optional |
| DELETE | Remove a resource | No | Yes | No | Optional |

**Syntax Rules:**

- **GET** must be safe: it must not modify server state. It should never have side effects.
- **POST** is used for creating resources and for operations that are not idempotent.
- **PUT** replaces the entire resource. If a field is omitted, it should be set to its default or null.
- **PATCH** applies a partial modification. Only the fields present in the request body should be modified.
- **DELETE** removes the resource. A successful DELETE typically returns `204 No Content`.

**Constraints and Limitations:**

- **Laravel does not enforce HTTP method semantics.** You can define a `GET` route that modifies data, but doing so violates REST principles and may cause caching systems to behave incorrectly.
- **HTML forms only support GET and POST.** To send a PUT, PATCH, or DELETE request from a browser form, you must use Laravel's `@method` Blade directive, which injects a hidden `_method` field.
- **PATCH idempotency is not guaranteed.** Applying the same PATCH request twice may produce different results if the operation is not idempotent (e.g., incrementing a counter).
- **PUT requires the full resource representation.** Sending a partial body with PUT may result in unintended data loss (fields not included are often set to null).

### Annotated Code Examples

**Example 1: A Controller Demonstrating All Five HTTP Methods**

```php
<?php
// File: app/Http/Controllers/Api/ArticleController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Article;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class ArticleController extends Controller
{
    // GET /api/articles — Safe and idempotent; retrieves a list
    public function index(): JsonResponse
    {
        return response()->json(Article::all());
    }

    // POST /api/articles — Not idempotent; creates a new resource
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body'  => 'required|string',
        ]);

        $article = Article::create($validated);

        return response()->json($article, 201);
    }

    // GET /api/articles/{article} — Safe and idempotent
    public function show(Article $article): JsonResponse
    {
        return response()->json($article);
    }

    // PUT/PATCH /api/articles/{article}
    // PUT replaces the entire resource; PATCH modifies only provided fields.
    public function update(Request $request, Article $article): JsonResponse
    {
        // The 'sometimes' rule only applies validation if the field is present.
        // For PUT, you would typically require all fields; for PATCH, use 'sometimes'.
        $validated = $request->validate([
            'title' => 'sometimes|string|max:255',
            'body'  => 'sometimes|string',
        ]);

        $article->update($validated);

        return response()->json($article);
    }

    // DELETE /api/articles/{article} — Idempotent; deleting twice yields 404 the second time
    public function destroy(Article $article): JsonResponse
    {
        $article->delete();

        return response()->json(null, 204);
    }
}
```

**Step-by-Step Setup:**

1. Create the `articles` table with `title` and `body` columns.
2. Create the `Article` model.
3. Define the resource route: `Route::apiResource('articles', ArticleController::class)`.
4. Seed a few articles for testing.

**Expected Output:**

- `GET /api/articles` → `200 OK` with `[{"id":1,"title":"First Article","body":"..."}]`.
- `POST /api/articles` with `{"title": "New", "body": "Content"}` → `201 Created` with the new article.
- `GET /api/articles/1` → `200 OK` with article 1.
- `PATCH /api/articles/1` with `{"title": "Updated Title"}` → `200 OK` with the updated article (body unchanged).
- `PUT /api/articles/1` with `{"title": "New Title", "body": "New Body"}` → `200 OK` with the replaced article.
- `DELETE /api/articles/1` → `204 No Content`.
- `DELETE /api/articles/1` again → `404 Not Found` (article already deleted).

**Why This Output Occurs:** The controller methods map directly to the HTTP methods. The `index` method returns a list (safe operation). The `store` method creates a new resource and returns `201`. The `update` method handles both PUT and PATCH; with PATCH, the `sometimes` validation rule ensures that omitted fields are not modified. The `destroy` method deletes the resource and returns `204`. The second DELETE returns `404` because route model binding fails to find the already-deleted article.

---

**Example 2: Using `_method` Spoofing in an HTML Form (Laravel Blade)**

```blade
{{-- File: resources/views/articles/edit.blade.php --}}

{{-- HTML forms only support GET and POST. Laravel's @method directive --}}
{{-- injects a hidden _method field that Laravel reads to override the --}}
{{-- HTTP verb for the incoming request. --}}
<form action="{{ route('articles.update', $article) }}" method="POST">
    @csrf
    @method('PUT')  {{-- Spoofs a PUT request --}}

    <input type="text" name="title" value="{{ $article->title }}">
    <textarea name="body">{{ $article->body }}</textarea>
    <button type="submit">Update Article</button>
</form>

{{-- For PATCH (partial update) --}}
<form action="{{ route('articles.update', $article) }}" method="POST">
    @csrf
    @method('PATCH')

    <input type="text" name="title" value="{{ $article->title }}">
    <button type="submit">Update Title Only</button>
</form>

{{-- For DELETE --}}
<form action="{{ route('articles.destroy', $article) }}" method="POST">
    @csrf
    @method('DELETE')

    <button type="submit">Delete Article</button>
</form>
```

**Expected Output:**

- The first form submits a `PUT` request to `/api/articles/{id}` (or the web route equivalent).
- The second form submits a `PATCH` request.
- The third form submits a `DELETE` request.
- Laravel's `MethodOverride` middleware reads the `_method` field and routes the request to the appropriate controller method.

**Why This Output Occurs:** Browsers natively support only GET and POST in HTML forms. The `@method('PUT')` directive generates `<input type="hidden" name="_method" value="PUT">`. Laravel's `Illuminate\Http\Middleware\HandleMethodOverride` middleware checks for this field and overrides `$request->method()` accordingly. The route is still matched as if the request used the actual HTTP verb.

### Real-World Cases

- **RESTful CRUD applications:** Every resource controller maps the five HTTP methods to five controller actions.
- **File upload APIs:** POST is used to upload a new file; PUT replaces an existing file; DELETE removes it.
- **Partial profile updates:** PATCH is used to update only the user's email without sending the entire profile.
- **Idempotent payment processing:** PUT is used to create or replace a payment record with a client-supplied idempotency key, ensuring that retrying the request does not create duplicate charges.

---

## 4. HTTP Status Codes

### Definitions

**Core Definition:** HTTP status codes are three-digit integers returned by the server in the HTTP response that indicate the outcome of the request.

**Technical Definition:** HTTP status codes are organized into five classes: 1xx (Informational), 2xx (Success), 3xx (Redirection), 4xx (Client Error), and 5xx (Server Error). Each code has a standardized meaning defined in RFC 9110 (which obsoletes RFC 7231). In RESTful APIs, the status code is the primary mechanism for communicating success or failure, supplementing the response body.

**Beginner-Friendly Explanation:** Status codes are like the server's way of saying "here's what happened" in a single number. A code starting with 2 (like 200 or 201) means everything went well. A code starting with 4 (like 404 or 422) means the client made a mistake. A code starting with 5 (like 500) means the server had a problem. Using the right code helps the client understand the result without having to parse the response body.

### Purposes

- To communicate the outcome of a request in a standardized, machine-readable format.
- To enable clients to handle success and failure cases programmatically without parsing response bodies.
- To distinguish between client errors (4xx) and server errors (5xx), aiding debugging and monitoring.
- To support caching and conditional requests through codes like `304 Not Modified`.
- To provide semantic meaning for automated tools, load balancers, and API gateways.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel Status Code Responses)

```php
// Returning a status code with a JSON body
return response()->json(['message' => 'Resource created'], 201);

// Returning a status code with no body
return response()->noContent(); // 204

// Returning a status code with a redirect
return redirect()->route('posts.index'); // 302

// Using the response() helper with an array
return response(['data' => $data], 200);
```

**Component Breakdown:**

- `response()->json($data, $status)` — Returns a JSON response with the given data and HTTP status code.
- `response()->noContent()` — Returns a `204 No Content` response with an empty body.
- `$status` — The integer HTTP status code (e.g., 200, 201, 204, 400, 401, 403, 404, 422, 500).

#### The Nine Required Status Codes

| Code | Name | Usage |
|------|------|-------|
| 200 | OK | Standard success response for GET, PUT, PATCH, and sometimes POST. |
| 201 | Created | A new resource was created (typically after POST). Include a `Location` header with the new resource's URI. |
| 204 | No Content | The request succeeded but there is no body to return (typically after DELETE or an update that returns nothing). |
| 400 | Bad Request | The request is malformed or contains invalid syntax (e.g., malformed JSON, missing required parameters at the protocol level). |
| 401 | Unauthorized | Authentication is required and has failed or has not been provided. |
| 403 | Forbidden | The authenticated user does not have permission to perform the action. |
| 404 | Not Found | The requested resource does not exist. |
| 422 | Unprocessable Entity | The request is well-formed but contains semantic errors (e.g., validation failures). |
| 500 | Internal Server Error | An unexpected condition prevented the server from fulfilling the request. |

**Syntax Rules:**

- **2xx codes indicate success.** The client's request was received, understood, and accepted.
- **4xx codes indicate client errors.** The client sent something the server cannot process.
- **5xx codes indicate server errors.** The server failed to fulfil a valid request.
- **Laravel's validation automatically returns 422** for AJAX/JSON requests when validation fails. No manual intervention is required.
- **`abort(404)`** in Laravel throws an `HttpException` that renders a 404 response automatically.
- **`$request->validate()`** throws a `ValidationException` that Laravel converts to a 422 JSON response for API requests.

**Constraints and Limitations:**

- **401 vs. 403:** `401 Unauthorized` means "you are not authenticated." `403 Forbidden` means "you are authenticated, but you do not have permission." Confusing these is a common mistake.
- **422 vs. 400:** `422 Unprocessable Entity` is for semantic errors (e.g., "email must be valid"). `400 Bad Request` is for syntax errors (e.g., "the JSON is malformed"). Laravel's validation system uses 422.
- **500 errors should not expose internal details** in production. Laravel's default error handler hides stack traces when `APP_DEBUG=false`.
- **Some proxies and gateways may intercept certain status codes.** For example, some CDNs treat `404` responses as cacheable, while others do not.

### Annotated Code Examples

**Example 1: Controller Demonstrating All Required Status Codes**

```php
<?php
// File: app/Http/Controllers/Api/UserController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Auth;

class UserController extends Controller
{
    // 200 OK — Successful retrieval
    public function index(): JsonResponse
    {
        return response()->json(User::all(), 200);
    }

    // 201 Created — Successful creation
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users',
            'password' => 'required|string|min:8',
        ]);

        $user = User::create($validated);

        // Include a Location header pointing to the new resource
        return response()->json($user, 201)
            ->header('Location', route('users.show', $user));
    }

    // 200 OK — Successful single retrieval
    public function show(User $user): JsonResponse
    {
        return response()->json($user, 200);
    }

    // 200 OK — Successful update
    public function update(Request $request, User $user): JsonResponse
    {
        $validated = $request->validate([
            'name'  => 'sometimes|string|max:255',
            'email' => 'sometimes|email|unique:users,email,' . $user->id,
        ]);

        $user->update($validated);

        return response()->json($user, 200);
    }

    // 204 No Content — Successful deletion
    public function destroy(User $user): JsonResponse
    {
        $user->delete();

        return response()->json(null, 204);
    }

    // 401 Unauthorized — Authentication required
    public function profile(Request $request): JsonResponse
    {
        if (!Auth::check()) {
            return response()->json([
                'error' => 'Authentication required.'
            ], 401);
        }

        return response()->json($request->user(), 200);
    }

    // 403 Forbidden — Authenticated but not authorized
    public function adminDashboard(Request $request): JsonResponse
    {
        if (!$request->user()->is_admin) {
            return response()->json([
                'error' => 'You do not have permission to access this resource.'
            ], 403);
        }

        return response()->json(['message' => 'Admin dashboard data'], 200);
    }

    // 404 Not Found — Resource does not exist
    public function find(int $id): JsonResponse
    {
        $user = User::find($id);

        if (!$user) {
            return response()->json([
                'error' => 'User not found.'
            ], 404);
        }

        return response()->json($user, 200);
    }

    // 422 Unprocessable Entity — Automatic via validation
    // Laravel automatically returns 422 when $request->validate() fails
    // for JSON requests. The response body contains the validation errors.

    // 500 Internal Server Error — Automatic via exception
    // If an unhandled exception occurs, Laravel returns 500.
    // In production (APP_DEBUG=false), the response body is generic.
}
```

**Step-by-Step Setup:**

1. Create the `users` table with `name`, `email`, `password`, and `is_admin` columns.
2. Ensure the `User` model has the appropriate fillable fields.
3. Define routes for each controller method.
4. For the 422 example, send a POST request with invalid data (e.g., missing `name`).

**Expected Output:**

- `GET /api/users` → `200 OK` with `[{"id":1,"name":"John","email":"john@example.com"}]`.
- `POST /api/users` with valid data → `201 Created` with the new user and a `Location` header.
- `POST /api/users` with `{"name": ""}` → `422 Unprocessable Entity` with `{"message": "The name field is required.", "errors": {"name": ["The name field is required."]}}`.
- `DELETE /api/users/1` → `204 No Content`.
- `GET /api/profile` without authentication → `401 Unauthorized`.
- `GET /api/admin` as a non-admin user → `403 Forbidden`.
- `GET /api/users/999` → `404 Not Found`.
- If an unhandled exception occurs → `500 Internal Server Error`.

**Why This Output Occurs:** Each controller method explicitly sets the status code based on the outcome. Laravel's `validate()` method automatically returns 422 for JSON requests when validation fails, without the need for manual intervention. Route model binding automatically returns 404 when the model is not found. The `Location` header in the 201 response tells the client where to find the newly created resource.

---

**Example 2: Automatic 422 Validation Response**

```php
<?php
// File: app/Http/Controllers/Api/RegistrationController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class RegistrationController extends Controller
{
    public function register(Request $request): JsonResponse
    {
        // Step 1: Validate the incoming request.
        // If validation fails and the request expects JSON (Accept: application/json),
        // Laravel automatically returns a 422 response with the error details.
        $validated = $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users',
            'password' => 'required|string|min:8|confirmed',
        ]);

        // Step 2: If validation passes, create the user.
        $user = \App\Models\User::create($validated);

        return response()->json($user, 201);
    }
}
```

**Expected Output (Validation Failure):**

```json
{
    "message": "The email has already been taken. (and 1 more error)",
    "errors": {
        "email": ["The email has already been taken."],
        "password": ["The password field confirmation does not match."]
    }
}
```

**HTTP Status Code: 422 Unprocessable Entity**

**Why This Output Occurs:** The `$request->validate()` method throws a `ValidationException` when the validation rules fail. Laravel's exception handler detects that the request expects JSON (via the `Accept: application/json` header) and converts the exception into a `422 Unprocessable Entity` response with the validation errors formatted as JSON. No manual status code assignment is required.

### Real-World Cases

- **User registration APIs:** Return 201 on success, 422 on validation failure, 500 on database error.
- **Authentication endpoints:** Return 200 on successful login, 401 on invalid credentials, 403 if the account is suspended.
- **E-commerce order processing:** Return 201 when an order is created, 400 if the request body is malformed, 422 if the cart is empty, 500 if payment processing fails.
- **Content management systems:** Return 200 for successful retrieval, 404 for missing articles, 403 for unauthorized edits, 204 for successful deletion.

---

## 5. JSON

### Definitions

**Core Definition:** JSON (JavaScript Object Notation) is a lightweight, text-based, language-independent data interchange format used to structure the payloads of API requests and responses.

**Technical Definition:** JSON is defined by RFC 8259 (STD 90). It represents four primitive types (strings, numbers, booleans, and null) and two structured types (objects and arrays). JSON text must be encoded in UTF-8, and the media type is `application/json`. The IANA registration for JSON specifies no charset parameter; UTF-8 is always assumed. In Laravel, JSON responses are generated using the `response()->json()` method, which serializes PHP arrays and objects into JSON using `json_encode()`.

**Beginner-Friendly Explanation:** JSON is the "language" that APIs speak. It is a simple way to represent data using curly braces `{}` for objects and square brackets `[]` for lists. For example, `{"name": "Alice", "age": 30}` is a JSON object. It is easy for both humans and computers to read, and almost every programming language can work with it. When you build an API in Laravel, the data you send back is almost always JSON.

### Purposes

- To provide a standardized, human-readable format for exchanging structured data between clients and servers.
- To ensure interoperability across programming languages and platforms.
- To support nested and hierarchical data structures (objects within objects, arrays of objects).
- To enable efficient parsing and generation in JavaScript environments (browsers, Node.js).
- To maintain a consistent structure across all API responses, simplifying client-side handling.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel JSON Response)

```php
// Returning a JSON response with a data wrapper
return response()->json([
    'data' => [
        'id'    => $post->id,
        'title' => $post->title,
        'body'  => $post->body,
    ],
    'meta' => [
        'version' => '1.0',
    ],
]);

// Using an Eloquent API Resource (automatically wrapped in 'data')
return new PostResource($post);

// Returning a collection of resources
return PostResource::collection(Post::all());
```

**Component Breakdown:**

- `response()->json($data, $status, $headers, $options)` — Creates a JSON response. The `$data` array is serialized to JSON.
- `new PostResource($post)` — Wraps a single model in a resource class. The default wrapper key is `data`.
- `PostResource::collection($posts)` — Wraps a collection of models in the resource's collection wrapper.

**JSON Data Types:**

| Type | Example | Notes |
|------|---------|-------|
| Object | `{"key": "value"}` | Unordered collection of key-value pairs. Keys must be strings. |
| Array | `[1, 2, 3]` | Ordered list of values. |
| String | `"hello"` | Must be enclosed in double quotes. |
| Number | `42`, `3.14` | Integer or floating-point. |
| Boolean | `true`, `false` | Lowercase. |
| Null | `null` | Lowercase. |

**Syntax Rules:**

- JSON keys and string values **must** use double quotes (`"`), not single quotes (`'`).
- Trailing commas are **not permitted** in JSON.
- Comments are **not permitted** in JSON.
- JSON text **must** be encoded in UTF-8.
- The media type for JSON is `application/json`. No `charset` parameter is defined for this media type.
- Laravel's `response()->json()` automatically sets the `Content-Type: application/json` header.

**Constraints and Limitations:**

- **JSON has no date type.** Dates must be serialized as strings (typically ISO 8601). Laravel's Eloquent models serialize dates as ISO 8601 strings by default.
- **JSON numbers have limited precision.** Very large integers or high-precision decimals may lose precision when parsed by JavaScript.
- **JSON does not support binary data.** Binary data must be base64-encoded or handled via multipart form data.
- **Deeply nested JSON can be difficult to consume.** REST practitioners often recommend flattening deeply nested structures or using sparse fieldsets.

### Annotated Code Examples

**Example 1: Structuring a Consistent JSON Response with Eloquent API Resources**

```php
<?php
// Step 1: Generate the API resource
// php artisan make:resource PostResource

// File: app/Http/Resources/PostResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'title'      => $this->title,
            'excerpt'    => substr($this->body, 0, 100) . '...',
            'author'     => [
                'id'   => $this->user->id,
                'name' => $this->user->name,
            ],
            'published_at' => $this->created_at->toIso8601String(),
            'links' => [
                'self' => route('posts.show', $this->id),
            ],
        ];
    }
}
```

```php
<?php
// File: app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\PostResource;
use App\Models\Post;
use Illuminate\Http\JsonResponse;

class PostController extends Controller
{
    // GET /api/posts — Returns a collection of PostResource objects
    public function index(): JsonResponse
    {
        return response()->json(PostResource::collection(Post::with('user')->paginate(10)));
    }

    // GET /api/posts/{post} — Returns a single PostResource
    public function show(Post $post): JsonResponse
    {
        return response()->json(new PostResource($post->load('user')));
    }
}
```

**Expected Output:**

```json
{
    "data": [
        {
            "id": 1,
            "title": "Getting Started with Laravel",
            "excerpt": "Laravel is a PHP framework that makes...",
            "author": {
                "id": 5,
                "name": "Alice Johnson"
            },
            "published_at": "2025-06-01T09:30:00+00:00",
            "links": {
                "self": "https://example.com/api/posts/1"
            }
        }
    ],
    "links": {
        "first": "https://example.com/api/posts?page=1",
        "last": "https://example.com/api/posts?page=5",
        "prev": null,
        "next": "https://example.com/api/posts?page=2"
    },
    "meta": {
        "current_page": 1,
        "per_page": 10,
        "total": 50
    }
}
```

**Why This Output Occurs:** The `PostResource` class defines exactly which fields are exposed and how they are formatted. The `substr()` call shortens the body to create an excerpt. The `author` field is a nested object containing only the user's ID and name (not the entire user model). The `published_at` field is formatted as an ISO 8601 string. When a resource collection is returned, Laravel automatically wraps the array in a `data` key and adds pagination `links` and `meta` when the collection is paginated.

---

**Example 2: Manually Constructing a JSON Response Without a Resource**

```php
<?php
// File: app/Http/Controllers/Api/HealthController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;

class HealthController extends Controller
{
    public function check(): JsonResponse
    {
        // Manually construct a JSON response with a consistent structure.
        // Always set the Content-Type header to application/json.
        return response()->json([
            'status'  => 'ok',
            'version' => '1.0.0',
            'checks'  => [
                'database' => $this->checkDatabase(),
                'cache'    => $this->checkCache(),
            ],
            'timestamp' => now()->toIso8601String(),
        ], 200, [
            'Content-Type' => 'application/json',
        ]);
    }

    private function checkDatabase(): array
    {
        try {
            \DB::connection()->getPdo();
            return ['status' => 'connected'];
        } catch (\Exception $e) {
            return ['status' => 'error', 'message' => 'Connection failed'];
        }
    }

    private function checkCache(): array
    {
        try {
            \Cache::store('redis')->get('health_check');
            return ['status' => 'connected'];
        } catch (\Exception $e) {
            return ['status' => 'error', 'message' => 'Cache unavailable'];
        }
    }
}
```

**Expected Output:**

```json
{
    "status": "ok",
    "version": "1.0.0",
    "checks": {
        "database": {
            "status": "connected"
        },
        "cache": {
            "status": "connected"
        }
    },
    "timestamp": "2025-06-15T10:30:00+00:00"
}
```

**HTTP Status Code: 200 OK**
**Content-Type: application/json**

**Why This Output Occurs:** The `response()->json()` method automatically serializes the PHP associative array into a JSON object and sets the `Content-Type: application/json` header. The nested arrays for `checks` become nested JSON objects. The `now()->toIso8601String()` call produces a valid ISO 8601 date string, which is JSON-compatible (JSON has no native date type).

### Real-World Cases

- **Public API documentation:** Tools like Swagger/OpenAPI use JSON Schema to describe request and response payloads.
- **Mobile app data synchronization:** Mobile apps consume JSON APIs to display lists of products, user profiles, or messages.
- **Webhook payloads:** Third-party services send JSON payloads to your webhook endpoints, and your application processes them and returns JSON acknowledgements.
- **Configuration APIs:** Applications expose JSON endpoints that return configuration data (feature flags, settings) to client applications.

---

## References

- Laravel Eloquent: API Resources — https://laravel.com/docs/eloquent-resources
- Laravel Routing: API Resource Routes — https://laravel.com/docs/controllers#api-resource-routes
- Laravel Validation — https://laravel.com/docs/validation
- Laravel HTTP Responses — https://laravel.com/docs/responses
- RFC 9110: HTTP Semantics (obsoletes RFC 7231) — https://www.rfc-editor.org/rfc/rfc9110
- RFC 8259: The JavaScript Object Notation (JSON) Data Interchange Format — https://www.rfc-editor.org/rfc/rfc8259
- RFC 5789: PATCH Method for HTTP — https://www.rfc-editor.org/rfc/rfc5789
- IANA Media Types: application/json — https://www.iana.org/assignments/media-types/application/json
- Fielding, R. T. (2000). Architectural Styles and the Design of Network-based Software Architectures (Doctoral dissertation, University of California, Irvine) — https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm
- MDN Web Docs: HTTP request methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- MDN Web Docs: HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- Laravel Sanctum Documentation — https://laravel.com/docs/sanctum
- JSON:API Specification — https://jsonapi.org/