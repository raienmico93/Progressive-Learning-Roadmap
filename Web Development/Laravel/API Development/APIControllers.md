# Laravel API Controllers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel API Controllers are classes that group related HTTP request-handling logic for API endpoints, responsible for receiving requests, delegating business logic, and returning JSON responses to clients.

**Technical Definition:** Controllers in Laravel reside in the `App\Http\Controllers` namespace and extend the base `Illuminate\Routing\Controller` class. API controllers are typically separated from web controllers via a dedicated `Api` sub-namespace and are registered in `routes/api.php`. They leverage Laravel's dependency injection container to resolve dependencies, route model binding to automatically inject Eloquent models, FormRequest classes for validation, and API Resource classes for JSON transformation. In Laravel 11+, the framework's leaner structure allows controllers to be created with `php artisan make:controller` and configured with `--api` or `--invokable` flags.

**Beginner-Friendly Explanation:** A controller is like a receptionist at a company. When a request comes in (a phone call), the controller decides who should handle it (which service or model), processes the request according to the rules (validation), and sends back the appropriate response (JSON data). Instead of putting all your logic in route files, you organise it into controllers — one controller per resource (like Users, Posts, Orders). This keeps your code clean and easy to find.

### Key Characteristics

- **Single Responsibility:** Controllers should be thin — they orchestrate requests and delegate heavy logic to services or models.
- **Dependency Injection:** Laravel's container automatically resolves dependencies type-hinted in controller methods.
- **Route Model Binding:** Eloquent models are automatically resolved from route parameters and injected into methods.
- **API Resource Transformation:** Eloquent models are transformed into JSON via API Resource classes, never serialised directly.
- **FormRequest Validation:** Validation rules are extracted into dedicated FormRequest classes, keeping controllers focused on flow.
- **JSON Error Handling:** Validation failures and exceptions return consistent JSON responses with appropriate HTTP status codes.

### Prerequisites

- PHP 8.1+ (Laravel 10+) or PHP 8.2+ (Laravel 11+).
- Composer dependency manager.
- A Laravel application with `routes/api.php` configured (via `php artisan install:api` in Laravel 11+).
- Basic understanding of Eloquent models and HTTP fundamentals.
- Familiarity with Laravel's service container and dependency injection.

### Related Programming Areas

- **Eloquent ORM** — Controllers interact with database records through Eloquent models.
- **Middleware** — Applied to controller routes for authentication, throttling, and request filtering.
- **FormRequest Validation** — Dedicated validation classes that keep controllers clean.
- **API Resources** — Transformation layer between models and JSON responses.
- **Exception Handling** — Centralised error rendering configured in `bootstrap/app.php` (Laravel 11+).
- **Service Container** — Resolves controller dependencies automatically.

### Core Concepts / Features

1. **CRUD Endpoints:** Single-action invokable controllers (`__invoke`) versus resource controllers.
2. **Request Validation:** Dedicated FormRequest classes overriding default redirection to return JSON validation errors.
3. **Resource Responses:** Correctly binding models and returning JSON responses with custom headers.
4. **Error Handling:** Customising the exception handler (`bootstrap/app.php` or `Handler.php`), catching `ModelNotFoundException`, and formatting global JSON error responses.

---

## 1. CRUD Endpoints

### Definitions

**Core Definition:** CRUD endpoints are controller actions that map to the five standard RESTful operations — Create, Read, Update, Delete — implemented either as single-action invokable controllers or as multi-method resource controllers.

**Technical Definition:** In Laravel, resource controllers are classes that implement the standard RESTful methods: `index`, `store`, `show`, `update`, and `destroy`. They are registered via `Route::apiResource()` or `Route::resource()`, which automatically generates the corresponding routes with correct HTTP verbs and URIs. Single-action invokable controllers implement the `__invoke()` magic method and are registered by passing the class name directly to a route definition. Invokable controllers are generated with `php artisan make:controller ProvisionServer --invokable` and are suited to endpoints that perform exactly one action.

**Beginner-Friendly Explanation:** When you build an API for a "resource" like posts, you typically need five operations: list all posts, create a post, show one post, update a post, and delete a post. A "resource controller" gives you all five methods in one class. But sometimes you have an endpoint that does just one thing — like processing a webhook or generating a report. For those, a "single-action controller" is cleaner: it has only one method (`__invoke`), so the class name itself tells you what it does.

### Purposes

- To organise CRUD operations for a resource into a single, discoverable class.
- To leverage Laravel's automatic route generation via `apiResource()`, reducing boilerplate.
- To separate concerns: controllers handle HTTP flow, while services or models handle business logic.
- To enable route model binding, automatically resolving route parameters to Eloquent models.
- To provide a clear, conventional structure that any Laravel developer can immediately understand.

### Syntax Rules and Structure

#### Complete General Syntax: Resource Controller

```php
// Generate the controller
// php artisan make:controller Api/PostController --api

// Register the routes
Route::apiResource('posts', PostController::class);

// Registering nested resource routes
Route::apiResource('posts.comments', CommentController::class)->shallow();

// Restricting to specific routes
Route::apiResource('posts', PostController::class)->only(['index', 'show']);
Route::apiResource('posts', PostController::class)->except(['destroy']);
```

**Component Breakdown:**

- `--api` flag — Generates a controller with only the five API-relevant methods (excludes `create` and `edit`, which are HTML form views).
- `Route::apiResource('posts', PostController::class)` — Registers five routes: `GET /posts`, `POST /posts`, `GET /posts/{post}`, `PUT/PATCH /posts/{post}`, `DELETE /posts/{post}`.
- `->shallow()` — Nested resources generate `index` and `store` with the full nested URI, but `show`, `update`, and `destroy` with a flat URI.
- `->only()` / `->except()` — Restrict which of the five standard routes are registered.

#### Complete General Syntax: Invokable Controller

```php
// Generate the controller
// php artisan make:controller Api/ProvisionServerController --invokable

// Register the route — no method name required
Route::post('/server/provision', ProvisionServerController::class);

// Using __invoke with dependency injection
class ProvisionServerController extends Controller
{
    public function __invoke(Request $request, ServerService $service): JsonResponse
    {
        $server = $service->provision($request->validated());
        return response()->json($server, 201);
    }
}
```

**Component Breakdown:**

- `--invokable` flag — Generates a controller with a single `__invoke()` method.
- `Route::post('/server/provision', ProvisionServerController::class)` — Registering an invokable controller requires only the class name, not a method name.
- `__invoke(Request $request, ServerService $service)` — The method receives dependencies via dependency injection; route model binding also works here.

**Syntax Rules:**

- Resource controllers **must** implement the standard method names (`index`, `store`, `show`, `update`, `destroy`) to work with `apiResource()`.
- Invokable controllers **must** implement `public function __invoke()` to be used as route callbacks.
- Route model binding works in both controller types; type-hint the model in the method signature.
- Controllers should be kept thin: delegate complex logic to services, actions, or the model itself.

**Constraints and Limitations:**

- **Invokable controllers should not contain branching logic.** If you find yourself adding `if` statements to handle different request types or response formats, refactor to a resource controller or separate actions.
- **Resource controllers expose all five methods by default.** If an endpoint should not support `destroy`, use `->except(['destroy'])` or implement a policy that denies the action.
- **`apiResource()` does not generate `create` and `edit` routes**, but it does generate the `update` method for both PUT and PATCH.
- **Controller middleware defined in the constructor does not run for `__invoke` in some contexts** — middleware should be applied via routes instead.

### Annotated Code Examples

**Example 1: Resource Controller for a Post Resource**

```php
<?php
// File: app/Http/Controllers/Api/PostController.php
// Generated with: php artisan make:controller Api/PostController --api

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    // GET /api/posts — List all posts (paginated)
    public function index(): JsonResponse
    {
        // Paginate to prevent returning thousands of records at once
        $posts = Post::with('user')->latest()->paginate(15);
        return response()->json($posts);
    }

    // POST /api/posts — Create a new post
    public function store(Request $request): JsonResponse
    {
        // Validation will be extracted to a FormRequest in a real application
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body'  => 'required|string',
        ]);

        $post = $request->user()->posts()->create($validated);

        return response()->json($post, 201);
    }

    // GET /api/posts/{post} — Show a single post
    // Route model binding automatically resolves {post} to a Post model.
    // If not found, a ModelNotFoundException is thrown → 404 JSON.
    public function show(Post $post): JsonResponse
    {
        return response()->json($post->load('user', 'comments'));
    }

    // PUT/PATCH /api/posts/{post} — Update a post
    public function update(Request $request, Post $post): JsonResponse
    {
        $validated = $request->validate([
            'title' => 'sometimes|string|max:255',
            'body'  => 'sometimes|string',
        ]);

        $post->update($validated);

        return response()->json($post);
    }

    // DELETE /api/posts/{post} — Delete a post
    public function destroy(Post $post): JsonResponse
    {
        $post->delete();

        return response()->json(null, 204);
    }
}
```

**Step-by-Step Setup:**

1. Run `php artisan make:controller Api/PostController --api` to generate the controller.
2. Create the `posts` table migration with `title`, `body`, and `user_id` columns.
3. Create the `Post` Eloquent model with the appropriate relationships.
4. Register the route in `routes/api.php`: `Route::apiResource('posts', PostController::class);`
5. Run `php artisan route:list` to verify the five routes are registered.

**Expected Output:**

```
GET|HEAD  api/posts ......................... Api\PostController@index
POST      api/posts ......................... Api\PostController@store
GET|HEAD  api/posts/{post} .................. Api\PostController@show
PUT|PATCH api/posts/{post} .................. Api\PostController@update
DELETE    api/posts/{post} .................. Api\PostController@destroy
```

**Why This Output Occurs:** The `apiResource()` method registers all five routes with the correct HTTP verbs and URIs. The `--api` flag ensures the controller only has the five API methods (no `create` or `edit` view methods). Route model binding automatically resolves the `{post}` parameter to a `Post` model instance.

---

**Example 2: Invokable Controller for a Webhook Endpoint**

```php
<?php
// File: app/Http/Controllers/Api/ProcessWebhookController.php
// Generated with: php artisan make:controller Api/ProcessWebhookController --invokable

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Services\WebhookService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class ProcessWebhookController extends Controller
{
    /**
     * Handle an incoming webhook from a payment provider.
     *
     * This is a single-action controller because webhook processing
     * is one discrete operation — there is no "list webhooks" or
     * "delete webhook" endpoint. The class name describes exactly
     * what it does.
     */
    public function __invoke(Request $request, WebhookService $service): JsonResponse
    {
        // Step 1: Validate the webhook signature (provider-specific)
        if (!$service->verifySignature($request)) {
            return response()->json(['error' => 'Invalid signature.'], 401);
        }

        // Step 2: Process the webhook payload
        $result = $service->handle($request->all());

        // Step 3: Return a 200 acknowledgement (providers often require this)
        return response()->json(['received' => true, 'event_id' => $result->id]);
    }
}
```

```php
// routes/api.php — Registering the invokable controller
use App\Http\Controllers\Api\ProcessWebhookController;

// Only the class name is passed — no method name needed.
Route::post('/webhooks/payment', ProcessWebhookController::class);
```

**Step-by-Step Setup:**

1. Run `php artisan make:controller Api/ProcessWebhookController --invokable`.
2. Create the `WebhookService` class with `verifySignature()` and `handle()` methods.
3. Register the route in `routes/api.php` by passing the class name directly.
4. Test by sending a POST request with a valid webhook payload.

**Expected Output:**

- `POST /api/webhooks/payment` with a valid signature → `200 OK` with `{"received": true, "event_id": 123}`.
- `POST /api/webhooks/payment` with an invalid signature → `401 Unauthorized` with `{"error": "Invalid signature."}`.

**Why This Output Occurs:** The invokable controller has a single `__invoke()` method, so Laravel routes the request directly to that method when the class is passed to `Route::post()`. The `WebhookService` is injected via the container. The class name makes the endpoint's purpose immediately clear — it processes webhooks, nothing else. Adding a method name to the route registration would be redundant.

### Real-World Cases

- **RESTful API resources (posts, users, orders):** Resource controllers are the standard approach for CRUD operations on a resource.
- **Webhook endpoints (payment providers, CI/CD systems):** Invokable controllers are ideal for single-purpose endpoints that perform one discrete action.
- **Report generation endpoints (`/api/reports/sales`):** Invokable controllers keep the "generate report" action isolated from any CRUD logic.
- **File upload endpoints (`/api/uploads`):** A single-action controller handles the upload, delegates storage to a service, and returns the file URL.
- **Authentication endpoints (`/api/login`, `/api/register`):** Each authentication action can be its own invokable controller, making the routing explicit.

---

## 2. Request Validation

### Definitions

**Core Definition:** FormRequest validation is Laravel's mechanism for extracting request validation rules into dedicated classes, with automatic JSON error responses for API requests when validation fails.

**Technical Definition:** A FormRequest is a class extending `Illuminate\Foundation\Http\FormRequest` that defines a `rules()` method returning an array of validation rules and an `authorize()` method determining whether the user is permitted to make the request. When a FormRequest is type-hinted in a controller method, Laravel resolves it from the container, runs validation before the controller method executes, and throws a `ValidationException` on failure. For requests expecting JSON (determined by the `Accept: application/json` header or the `expectsJson()` method), the exception handler renders a `422 Unprocessable Entity` response with a JSON body containing the validation errors.

**Beginner-Friendly Explanation:** Instead of writing validation rules inside every controller method, you create a separate "FormRequest" class that contains all the rules for that request. Laravel runs the validation automatically before your controller code even starts. If the data is invalid, the client gets back a structured JSON error response with a `422` status code — no need to write any error-handling code yourself. This keeps controllers clean and validation reusable.

### Purposes

- To extract validation logic from controllers into dedicated, reusable classes.
- To automatically return JSON validation errors with a `422` status code for API requests.
- To centralise authorisation checks (`authorize()`) alongside validation rules.
- To enable custom error messages and attribute names per request.
- To allow validation rules to be reused across multiple endpoints (e.g., create and update).

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Generate the FormRequest
// php artisan make:request StorePostRequest

// File: app/Http/Requests/StorePostRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    // Determine if the user is authorized to make this request.
    public function authorize(): bool
    {
        return true; // Set to false if you want to deny by default
    }

    // Get the validation rules that apply to the request.
    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'body'  => ['required', 'string'],
            'tags'  => ['sometimes', 'array'],
            'tags.*' => ['string', 'max:50'],
        ];
    }

    // Optional: Custom error messages
    public function messages(): array
    {
        return [
            'title.required' => 'A title is required for the post.',
            'body.required'  => 'The post body cannot be empty.',
        ];
    }

    // Optional: Custom attribute names for error messages
    public function attributes(): array
    {
        return [
            'body' => 'content',
        ];
    }
}
```

```php
// Step 2: Type-hint the FormRequest in the controller
use App\Http\Requests\StorePostRequest;

public function store(StorePostRequest $request): JsonResponse
{
    // Validation has already passed — $request->validated() returns only validated data
    $post = Post::create($request->validated());
    return response()->json($post, 201);
}
```

**Component Breakdown:**

- `authorize(): bool` — Returns `true` to allow the request; `false` throws an `AuthorizationException` (403). For public endpoints, return `true`.
- `rules(): array` — An array of validation rules keyed by field name. Uses Laravel's standard validation rule syntax (string with pipes or arrays).
- `messages(): array` — Optional. Overrides the default error messages for specific rules.
- `attributes(): array` — Optional. Maps field names to human-readable labels for error messages.
- `$request->validated()` — Returns only the data that passed validation, safely filtered.

**Syntax Rules:**

- FormRequest classes are stored in `app/Http/Requests/` by convention.
- The `authorize()` method **must** return a boolean; returning `false` results in a `403 Forbidden` response.
- FormRequests are automatically resolved by the container when type-hinted in controller methods — no manual instantiation is needed.
- For API requests (with `Accept: application/json`), validation failures return `422 Unprocessable Entity` with a JSON body. For web requests, they redirect back with flashed error messages.
- The `failedValidation()` method can be overridden to customise the exception thrown on failure.

**Constraints and Limitations:**

- **FormRequest validation runs before the controller method.** If you need to perform logic before validation, you must use middleware or handle it within `prepareForValidation()`.
- **The default JSON response format for validation errors is not configurable via a simple method** — you must override `failedValidation()` or customise the exception handler to change the structure.
- **FormRequests do not validate the `Content-Type` header.** Malformed JSON bodies result in `null` request data, not a validation error. A separate middleware is required to catch JSON parse errors.
- **Custom error messages in `messages()` must use the `field.rule` syntax** (e.g., `'title.required'`), not the full rule name.

### Annotated Code Examples

**Example 1: FormRequest with JSON Validation Errors**

```php
<?php
// File: app/Http/Requests/StorePostRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        // Only authenticated users can create posts
        return $this->user() !== null;
    }

    public function rules(): array
    {
        return [
            'title'   => ['required', 'string', 'max:255'],
            'body'    => ['required', 'string', 'min:10'],
            'tags'    => ['sometimes', 'array', 'max:5'],
            'tags.*'  => ['string', 'max:50'],
            'publish' => ['sometimes', 'boolean'],
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Every post must have a title.',
            'title.max'      => 'The title must not exceed 255 characters.',
            'body.min'       => 'The post body must be at least 10 characters.',
            'tags.max'       => 'You may add at most 5 tags.',
        ];
    }
}
```

```php
<?php
// File: app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StorePostRequest;
use App\Models\Post;
use Illuminate\Http\JsonResponse;

class PostController extends Controller
{
    public function store(StorePostRequest $request): JsonResponse
    {
        // $request->validated() contains only the fields that passed validation.
        // No manual validation or error handling is needed.
        $post = $request->user()->posts()->create($request->validated());

        return response()->json($post, 201);
    }
}
```

**Step-by-Step Setup:**

1. Run `php artisan make:request StorePostRequest`.
2. Define the `rules()`, `authorize()`, and optionally `messages()` methods.
3. Type-hint `StorePostRequest` in the controller's `store()` method.
4. Register the route: `Route::apiResource('posts', PostController::class)`.
5. Test with an invalid payload.

**Expected Output (Validation Failure):**

```json
{
    "message": "Every post must have a title. (and 1 more error)",
    "errors": {
        "title": ["Every post must have a title."],
        "body": ["The post body must be at least 10 characters."]
    }
}
```

**HTTP Status Code: 422 Unprocessable Entity**

**Why This Output Occurs:** When the `StorePostRequest` is resolved, Laravel runs the validation rules defined in `rules()`. If validation fails, a `ValidationException` is thrown. Because the request expects JSON (the client sent `Accept: application/json`), Laravel's exception handler converts the exception into a `422 Unprocessable Entity` JSON response. The `messages()` method overrides the default error messages. The `authorize()` method runs first; if it returns `false`, a `403 Forbidden` is returned instead.

---

**Example 2: Customising the Failed Validation Response for API Consistency**

```php
<?php
// File: app/Http/Requests/BaseApiRequest.php

namespace App\Http\Requests;

use Illuminate\Contracts\Validation\Validator;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;

abstract class BaseApiRequest extends FormRequest
{
    /**
     * Override the default failed validation behaviour to return
     * a consistently structured JSON response for all API requests.
     */
    protected function failedValidation(Validator $validator): void
    {
        throw new HttpResponseException(
            response()->json([
                'success' => false,
                'error'   => [
                    'code'    => 'VALIDATION_ERROR',
                    'message' => 'The submitted data is invalid.',
                    'details' => $validator->errors()->toArray(),
                ],
            ], 422)
        );
    }

    /**
     * Override the default failed authorization behaviour.
     */
    protected function failedAuthorization(): void
    {
        throw new HttpResponseException(
            response()->json([
                'success' => false,
                'error'   => [
                    'code'    => 'UNAUTHORIZED',
                    'message' => 'You are not authorized to perform this action.',
                ],
            ], 403)
        );
    }
}
```

```php
<?php
// File: app/Http/Requests/StorePostRequest.php — Extends the base

namespace App\Http\Requests;

class StorePostRequest extends BaseApiRequest
{
    public function authorize(): bool
    {
        return true; // Authorization handled by middleware
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'body'  => ['required', 'string'],
        ];
    }
}
```

**Expected Output (Validation Failure):**

```json
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "The submitted data is invalid.",
        "details": {
            "title": ["The title field is required."],
            "body": ["The body field is required."]
        }
    }
}
```

**HTTP Status Code: 422 Unprocessable Entity**

**Why This Output Occurs:** The `failedValidation()` method is overridden in the base class to throw an `HttpResponseException` with a custom JSON structure. All FormRequests extending `BaseApiRequest` inherit this behaviour, ensuring every API validation error has the same structure. This is essential for API consistency: clients can rely on `success: false` and `error.code` to programmatically handle errors.

### Real-World Cases

- **User registration and profile updates:** FormRequests validate email uniqueness, password strength, and required fields, returning structured JSON errors to mobile apps and SPAs.
- **E-commerce order creation:** Validation ensures the cart is not empty, quantities are valid, and shipping addresses are complete before creating an order.
- **Multi-step forms and wizards:** Each step can have its own FormRequest, with validation running incrementally as the user progresses.
- **Third-party API integrations:** When your API is consumed by external developers, consistent validation error responses reduce integration friction.

---

## 3. Resource Responses

### Definitions

**Core Definition:** Resource responses in Laravel refer to the use of API Resource classes to transform Eloquent models and collections into structured JSON responses, optionally with custom headers and metadata.

**Technical Definition:** API Resource classes extend `Illuminate\Http\Resources\Json\JsonResource` and define a `toArray(Request $request)` method that returns an array of attributes to be serialised to JSON. Resource collections extend `Illuminate\Http\Resources\Json\ResourceCollection` and handle arrays of resources, including pagination metadata. Resources can be returned directly from routes and controllers; Laravel automatically wraps them in a `data` key by default. Custom HTTP headers can be added via the `response()` method or the `withResponse()` method on the resource class.

**Beginner-Friendly Explanation:** When you return a model directly from a controller, Laravel serialises every column — including sensitive ones like passwords. API Resources solve this: you define exactly which fields should appear in the JSON, how they should be formatted, and what relationships to include. Think of a resource as a "filter" or "template" that sits between your database model and the JSON your API returns.

### Purposes

- To control exactly which model attributes are exposed in API responses, preventing accidental data leaks.
- To transform model data into a consistent, client-friendly JSON structure.
- To include computed fields, nested relationships, and conditional attributes.
- To add pagination metadata to collections automatically.
- To attach custom HTTP headers or status codes to resource responses.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Generate the resource
// php artisan make:resource PostResource

// File: app/Http/Resources/PostResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'title'      => $this->title,
            'excerpt'    => substr($this->body, 0, 100) . '...',
            'author'     => new UserResource($this->whenLoaded('user')),
            'comments'   => CommentResource::collection($this->whenLoaded('comments')),
            'published_at' => $this->created_at->toIso8601String(),
            'links' => [
                'self' => route('posts.show', $this->id),
            ],
        ];
    }
}
```

```php
// Step 2: Return the resource from the controller
use App\Http\Resources\PostResource;

// Single resource
return new PostResource($post);

// Collection of resources
return PostResource::collection(Post::paginate(15));

// Fluent method (Laravel 11+)
return $post->toResource();
return $posts->toResourceCollection();
```

**Component Breakdown:**

- `toArray(Request $request): array` — Defines the JSON structure. `$this` refers to the underlying Eloquent model.
- `$this->whenLoaded('relation')` — Includes a relationship only if it has been eager-loaded, preventing N+1 queries.
- `UserResource($this->whenLoaded('user'))` — Nests another resource for a relationship.
- `PostResource::collection($posts)` — Wraps a collection (including paginated collections) in the resource's collection wrapper.
- `$post->toResource()` — Laravel 11+ fluent method that automatically discovers the resource class based on the model name.
- `withResponse(Request $request, Response $response)` — Optional method on the resource class to customise the outgoing HTTP response.

**Syntax Rules:**

- Resource classes are stored in `app/Http/Resources/` by convention.
- The `toArray()` method **must** return an array; nested resources and collections are automatically serialised.
- By default, resources are wrapped in a `data` key. This can be disabled globally via `JsonResource::withoutWrapping()`.
- Paginated collections automatically include `links` and `meta` keys with pagination metadata.
- Custom headers can be added by chaining `->response()->header('X-Custom', 'value')` or by overriding `withResponse()`.

**Constraints and Limitations:**

- **Resources should not perform database queries.** Use `whenLoaded()` to include relationships only when they have been eager-loaded, or use `when()` to conditionally include data.
- **The `data` wrapper is applied by default.** If your API must match a specific structure (e.g., JSON:API), disable wrapping or customise the wrapper key.
- **Resources are serialised lazily.** If you return a resource from a route, Laravel serialises it when the response is sent, not when it is instantiated.
- **Conditional attributes (`when()`, `whenLoaded()`, `whenHas()`) are essential** to avoid exposing data that was not requested or loaded.

### Annotated Code Examples

**Example 1: Resource with Relationship Loading and Custom Headers**

```php
<?php
// File: app/Http/Resources/PostResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'title'      => $this->title,
            'body'       => $this->body,
            'author'     => new UserResource($this->whenLoaded('user')),
            'comments'   => CommentResource::collection($this->whenLoaded('comments')),
            'created_at' => $this->created_at->toIso8601String(),
            'updated_at' => $this->updated_at->toIso8601String(),
        ];
    }

    /**
     * Customise the outgoing HTTP response for this resource.
     * This method is called automatically when the resource is returned.
     */
    public function withResponse(Request $request, $response): void
    {
        $response->header('X-API-Version', '1.0');
        $response->header('Cache-Control', 'no-cache, private');
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
    public function show(Post $post): JsonResponse
    {
        // Eager-load relationships to avoid N+1 queries.
        // The resource includes them only because they are loaded.
        return response()->json(
            new PostResource($post->load('user', 'comments.author'))
        );
    }

    public function index()
    {
        // Paginated collection — links and meta are added automatically.
        $posts = Post::with('user')->latest()->paginate(15);
        return PostResource::collection($posts);
    }
}
```

**Step-by-Step Setup:**

1. Run `php artisan make:resource PostResource` and `php artisan make:resource UserResource` and `php artisan make:resource CommentResource`.
2. Define the `toArray()` methods for each resource.
3. Type-hint the model in the controller and call `->load()` to eager-load relationships.
4. Return the resource from the controller — Laravel serialises it automatically.

**Expected Output:**

```json
{
    "data": {
        "id": 1,
        "title": "Getting Started with Laravel",
        "body": "Laravel is a PHP framework...",
        "author": {
            "data": {
                "id": 5,
                "name": "Alice Johnson",
                "email": "alice@example.com"
            }
        },
        "comments": {
            "data": [
                {
                    "id": 10,
                    "body": "Great post!",
                    "author": {
                        "data": { "id": 7, "name": "Bob" }
                    }
                }
            ]
        },
        "created_at": "2025-06-01T09:30:00+00:00",
        "updated_at": "2025-06-01T09:30:00+00:00"
    }
}
```

**Response Headers:**
```
X-API-Version: 1.0
Cache-Control: no-cache, private
```

**Why This Output Occurs:** The `PostResource` transforms the model into a structured array. The `whenLoaded()` calls ensure that `author` and `comments` are only included if they were eager-loaded via `->load()`. Each nested relationship is wrapped in its own resource, producing nested `data` keys. The `withResponse()` method adds custom headers to the outgoing HTTP response automatically.

---

**Example 2: Fluent Resource Methods (Laravel 11+)**

```php
<?php
// File: app/Http/Controllers/Api/ProductController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // Laravel 11+ fluent method: automatically discovers ProductResource
    public function show(Product $product)
    {
        return $product->toResource();
        // Equivalent to: new ProductResource($product)
    }

    public function index(Request $request)
    {
        $products = Product::query()
            ->when($request->has('featured'), fn($q) => $q->featured())
            ->when($request->has('category'), fn($q) => $q->where('category_id', $request->category))
            ->latest()
            ->paginate();

        return $products->toResourceCollection();
        // Equivalent to: ProductResource::collection($products)
    }
}
```

**Expected Output:** Identical to `new ProductResource($product)` and `ProductResource::collection($products)`, but with more readable, left-to-right code flow. The `toResource()` method automatically searches for a resource class matching the model name (`Product` → `ProductResource`).

**Why This Output Occurs:** Laravel 11 introduced fluent resource methods that attach transformation capabilities directly to Eloquent models and collections. When `toResource()` is called without arguments, Laravel uses convention-over-configuration to discover the matching resource class. If the resource is in a different namespace, you can pass the class explicitly: `$product->toResource(CustomProductResource::class)`.

### Real-World Cases

- **Public APIs with sensitive data:** Resources prevent password hashes, internal IDs, or soft-delete timestamps from leaking into JSON responses.
- **Multi-platform APIs:** Different resources can be defined for the same model (e.g., `PostSummaryResource` for list views, `PostDetailResource` for single views), optimising payload size.
- **API versioning:** Resources for V1 and V2 can transform the same model differently, supporting backward compatibility.
- **Mobile app optimisation:** Resources can include only the fields the mobile app needs, reducing payload size and improving performance on slow networks.

---

## 4. Error Handling

### Definitions

**Core Definition:** API error handling in Laravel is the centralised process of catching exceptions, transforming them into consistent JSON error responses, and returning appropriate HTTP status codes.

**Technical Definition:** In Laravel 11+, exception handling is configured in `bootstrap/app.php` using the `withExceptions()` method, which provides an `Exceptions` configuration object. The `render()` method on this object accepts a closure that receives the exception and request, and returns a response or `null`. If `null` is returned, Laravel's default handling applies. For `ModelNotFoundException`, which is thrown automatically by route model binding when a model is not found, the exception's `getModel()` method returns the class name of the missing model, enabling dynamic, resource-specific error messages. Validation exceptions (`ValidationException`) are automatically rendered as `422` JSON responses for API requests.

**Beginner-Friendly Explanation:** When something goes wrong in your API — a resource is not found, validation fails, or an unexpected error occurs — you want the client to receive a clean JSON error message, not an HTML error page. Laravel 11 lets you configure this in one place (`bootstrap/app.php`), where you can say "if the request expects JSON, return a JSON error with the right status code." This ensures every error response from your API has the same structure, making it easy for clients to handle errors programmatically.

### Purposes

- To return consistent, structured JSON error responses for all API exceptions.
- To distinguish between client errors (4xx) and server errors (5xx) with appropriate status codes.
- To catch `ModelNotFoundException` and return a descriptive `404` JSON response instead of an HTML page.
- To provide meaningful error messages without exposing internal implementation details.
- To centralise error handling logic, avoiding repetitive try-catch blocks in every controller.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel 11+)

```php
// File: bootstrap/app.php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Illuminate\Database\Eloquent\ModelNotFoundException;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Handle ModelNotFoundException — thrown by route model binding
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

        // Handle all HTTP exceptions (404, 403, etc.)
        $exceptions->render(function (NotFoundHttpException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'NOT_FOUND',
                        'message' => 'The requested endpoint does not exist.',
                    ],
                ], 404);
            }
        });

        // Handle authentication exceptions
        $exceptions->render(function (\Illuminate\Auth\AuthenticationException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'UNAUTHENTICATED',
                        'message' => 'Authentication is required.',
                    ],
                ], 401);
            }
        });

        // Handle authorization exceptions
        $exceptions->render(function (\Illuminate\Auth\Access\AuthorizationException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'FORBIDDEN',
                        'message' => 'You do not have permission to perform this action.',
                    ],
                ], 403);
            }
        });

        // Global fallback for all other exceptions (500 errors)
        $exceptions->render(function (\Throwable $e, Request $request) {
            if ($request->expectsJson()) {
                $status = method_exists($e, 'getStatusCode') ? $e->getStatusCode() : 500;
                $message = app()->isProduction()
                    ? 'An unexpected error occurred.'
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

**Component Breakdown:**

- `withExceptions(function (Exceptions $exceptions) { ... })` — Configures exception handling in Laravel 11+. Replaces the `App\Exceptions\Handler` class from Laravel 10-.
- `$exceptions->render(function (ExceptionType $e, Request $request) { ... })` — Registers a render callback for a specific exception type.
- `$request->expectsJson()` — Returns `true` if the request expects a JSON response (based on the `Accept` header or the request being an API request).
- `$e->getModel()` — On `ModelNotFoundException`, returns the fully qualified class name of the model that was not found.
- `class_basename($e->getModel())` — Extracts the short class name (e.g., `Post` from `App\Models\Post`).

**Syntax Rules:**

- The `withExceptions()` method is available in Laravel 11+. In Laravel 10 and earlier, exception handling is configured in `app/Exceptions/Handler.php` via the `render()` method.
- Multiple `render()` callbacks can be registered; Laravel uses the first one that returns a non-null response.
- The order of `render()` callbacks matters: more specific exceptions should be registered before more general ones.
- Returning `null` from a `render()` callback allows Laravel's default handling to proceed.
- For production safety, never expose exception messages in `500` responses when `APP_DEBUG=false`.

**Constraints and Limitations:**

- **`ModelNotFoundException` is thrown by route model binding**, not by `findOrFail()` in all cases. The exception is also thrown by `firstOrFail()`.
- **Validation exceptions are handled automatically** — you do not need a custom render callback unless you want to change the response structure.
- **The `render()` callbacks receive the request**, so you can differentiate between web and API responses using `$request->expectsJson()`.
- **In Laravel 10 and earlier**, the `Handler.php` approach is used; the `bootstrap/app.php` approach is Laravel 11+ only.
- **Custom exception classes can implement the `render()` method** to self-render, which takes precedence over the exception handler.

### Annotated Code Examples

**Example 1: Complete Exception Handler in bootstrap/app.php (Laravel 11+)**

```php
<?php
// File: bootstrap/app.php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use Illuminate\Http\Request;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Illuminate\Auth\AuthenticationException;
use Illuminate\Auth\Access\AuthorizationException;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
use Symfony\Component\HttpKernel\Exception\TooManyRequestsHttpException;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {

        // 1. Model not found (route model binding)
        $exceptions->render(function (ModelNotFoundException $e, Request $request) {
            if ($request->expectsJson()) {
                $model = class_basename($e->getModel());
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'RESOURCE_NOT_FOUND',
                        'message' => "{$model} not found.",
                        'model'   => $model,
                    ],
                ], 404);
            }
        });

        // 2. Endpoint not found (route not registered)
        $exceptions->render(function (NotFoundHttpException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'ENDPOINT_NOT_FOUND',
                        'message' => 'The requested endpoint does not exist.',
                    ],
                ], 404);
            }
        });

        // 3. Authentication failed (no token / invalid token)
        $exceptions->render(function (AuthenticationException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'UNAUTHENTICATED',
                        'message' => 'Authentication is required. Please provide a valid token.',
                    ],
                ], 401);
            }
        });

        // 4. Authorization failed (authenticated but not permitted)
        $exceptions->render(function (AuthorizationException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'FORBIDDEN',
                        'message' => 'You do not have permission to perform this action.',
                    ],
                ], 403);
            }
        });

        // 5. Validation failed (custom structure for all FormRequests)
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

        // 6. Rate limit exceeded
        $exceptions->render(function (TooManyRequestsHttpException $e, Request $request) {
            if ($request->expectsJson()) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'RATE_LIMIT_EXCEEDED',
                        'message' => 'Too many requests. Please try again later.',
                    ],
                ], 429);
            }
        });

        // 7. Global fallback — all other exceptions (500, etc.)
        $exceptions->render(function (\Throwable $e, Request $request) {
            if ($request->expectsJson()) {
                $status = method_exists($e, 'getStatusCode') ? $e->getStatusCode() : 500;

                // In production, hide internal error messages
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

1. Open `bootstrap/app.php` in a Laravel 11+ application.
2. Add the `render()` callbacks inside the `withExceptions()` closure.
3. Order matters: register specific exceptions (`ModelNotFoundException`, `NotFoundHttpException`) before general ones (`Throwable`).
4. Test each scenario by sending requests that trigger the corresponding exceptions.

**Expected Outputs:**

- `GET /api/posts/999` (post not found) → `404` with `{"success":false,"error":{"code":"RESOURCE_NOT_FOUND","message":"Post not found.","model":"Post"}}`
- `GET /api/nonexistent` → `404` with `{"success":false,"error":{"code":"ENDPOINT_NOT_FOUND","message":"The requested endpoint does not exist."}}`
- `GET /api/user` without a token → `401` with `{"success":false,"error":{"code":"UNAUTHENTICATED","message":"Authentication is required..."}}`
- `DELETE /api/posts/1` as a non-owner → `403` with `{"success":false,"error":{"code":"FORBIDDEN","message":"You do not have permission..."}}`
- `POST /api/posts` with invalid data → `422` with `{"success":false,"error":{"code":"VALIDATION_ERROR","message":"The submitted data is invalid.","details":{"title":["The title field is required."]}}}`
- An unhandled exception → `500` with `{"success":false,"error":{"code":"INTERNAL_ERROR","message":"An unexpected error occurred..."}}` (in production)

**Why This Output Occurs:** Each `render()` callback is registered for a specific exception type. When that exception is thrown, Laravel calls the callback with the exception and request. The callback checks `$request->expectsJson()` to ensure it only handles API requests (web requests fall through to Laravel's default HTML error pages). The `ModelNotFoundException` callback uses `$e->getModel()` to dynamically determine which model was not found, producing a resource-specific error message. The global `Throwable` callback acts as a catch-all for any unhandled exception, returning a generic 500 response in production to avoid leaking internal details.

---

**Example 2: Legacy Handler.php Approach (Laravel 10 and Earlier)**

```php
<?php
// File: app/Exceptions/Handler.php (Laravel 10 and earlier)

namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Illuminate\Auth\AuthenticationException;
use Illuminate\Auth\Access\AuthorizationException;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Throwable;

class Handler extends ExceptionHandler
{
    protected $dontReport = [
        //
    ];

    protected $dontFlash = [
        'current_password',
        'password',
        'password_confirmation',
    ];

    public function register(): void
    {
        $this->reportable(function (Throwable $e) {
            //
        });
    }

    public function render($request, Throwable $e)
    {
        if ($request->expectsJson()) {
            // Model not found
            if ($e instanceof ModelNotFoundException) {
                $model = class_basename($e->getModel());
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'RESOURCE_NOT_FOUND',
                        'message' => "{$model} not found.",
                    ],
                ], 404);
            }

            // Endpoint not found
            if ($e instanceof NotFoundHttpException) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'ENDPOINT_NOT_FOUND',
                        'message' => 'The requested endpoint does not exist.',
                    ],
                ], 404);
            }

            // Authentication
            if ($e instanceof AuthenticationException) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'UNAUTHENTICATED',
                        'message' => 'Authentication is required.',
                    ],
                ], 401);
            }

            // Authorization
            if ($e instanceof AuthorizationException) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'FORBIDDEN',
                        'message' => 'You do not have permission to perform this action.',
                    ],
                ], 403);
            }

            // Validation
            if ($e instanceof ValidationException) {
                return response()->json([
                    'success' => false,
                    'error'   => [
                        'code'    => 'VALIDATION_ERROR',
                        'message' => 'The submitted data is invalid.',
                        'details' => $e->errors(),
                    ],
                ], 422);
            }

            // Global fallback
            $status = method_exists($e, 'getStatusCode') ? $e->getStatusCode() : 500;
            $message = app()->isProduction()
                ? 'An unexpected error occurred.'
                : $e->getMessage();

            return response()->json([
                'success' => false,
                'error'   => [
                    'code'    => 'INTERNAL_ERROR',
                    'message' => $message,
                ],
            ], $status);
        }

        return parent::render($request, $e);
    }
}
```

**Expected Output:** Identical to the Laravel 11+ version, but configured in `app/Exceptions/Handler.php` instead of `bootstrap/app.php`.

**Why This Output Occurs:** The `render()` method is called by Laravel's exception handling pipeline for every unhandled exception. The method checks `$request->expectsJson()` to determine whether to return a JSON response. Each exception type is handled with a specific status code and error structure. The final `parent::render()` call allows Laravel's default handling for non-JSON requests (web requests). In Laravel 11, the `Handler.php` file was removed and its functionality moved to `bootstrap/app.php`.

### Real-World Cases

- **Public APIs consumed by third parties:** Consistent JSON error responses (with stable error codes) allow external developers to handle errors programmatically.
- **Mobile app backends:** Mobile apps rely on status codes and error structures to display appropriate messages to users (e.g., "Invalid credentials" for 401, "Validation failed" for 422).
- **Microservices architectures:** Services need to distinguish between client errors (4xx) and server errors (5xx) to decide whether to retry or fail.
- **Compliance and security:** Global error handlers ensure that sensitive information (stack traces, database queries) is never exposed in production API responses.

---

## References

- Laravel Controllers Documentation — https://laravel.com/docs/controllers
- Laravel Validation Documentation — https://laravel.com/docs/validation
- Laravel Eloquent: API Resources — https://laravel.com/docs/eloquent-resources
- Laravel Error Handling Documentation — https://laravel.com/docs/errors
- Laravel Requests Documentation — https://laravel.com/docs/requests
- Laravel Fluent Resource Methods (Laravel News) — https://laravel-news.com/api-resources-fluent-methods
- Laravel `install:api` Artisan Command — https://laravel.com/docs/artisan#install-api
- Symfony `ModelNotFoundException` — https://api.symfony.com/6.0/Symfony/Component/HttpKernel/Exception/NotFoundHttpException.html
- Laravel API Resources: Fluent Methods — https://laravel.com/docs/eloquent-resources#fluent-methods
- Laravel Form Requests: `failedValidation()` Override — https://laravel.com/docs/validation#form-request-validation
- Laravel `bootstrap/app.php` Exception Configuration — https://laravel.com/docs/errors#rendering-exceptions
- Laravel JSON Parse Error Handling (Laracasts) — https://laracasts.com/discuss/channels/laravel/surfacing-json-parsing-errors-in-form-request