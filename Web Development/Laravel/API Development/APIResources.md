# Laravel API Resources — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel API Resources are transformation classes that sit between Eloquent models and JSON responses, giving developers granular control over which attributes are exposed, how relationships are serialised, and how response envelopes are structured.

**Technical Definition:** API Resources are classes extending `Illuminate\Http\Resources\Json\JsonResource` (for single models) or `Illuminate\Http\Resources\Json\ResourceCollection` (for collections). Each resource defines a `toArray(Request $request): array` method that returns the array of attributes to be serialised to JSON. Resources leverage the `ConditionallyLoadsAttributes` trait, which provides methods such as `when()`, `mergeWhen()`, and `whenLoaded()` for conditional attribute inclusion. When returned from a controller or route, resources are automatically wrapped in a `data` key by default, and paginated collections receive automatic `links` and `meta` keys. The static `$wrap` property and the `withoutWrapping()` method control response enveloping.

**Beginner-Friendly Explanation:** When you return an Eloquent model directly from a controller, Laravel converts every column into JSON — including sensitive fields like passwords or internal flags. API Resources solve this by letting you define exactly what the API should output. Think of a resource as a "template" that takes a model and produces a clean, controlled JSON object. You decide which fields appear, how dates are formatted, and whether related data (like a post's author) should be included. This makes your API consistent, secure, and easy to document.

### Key Characteristics

- **Transformation layer:** Resources decouple database structure from API output, preventing accidental exposure of internal columns.
- **Single-model and collection variants:** `JsonResource` transforms one model; `ResourceCollection` transforms arrays of models with pagination support.
- **Conditional attributes:** Methods like `when()`, `mergeWhen()`, and `whenLoaded()` include data only when conditions are met.
- **Automatic wrapping:** The outermost resource is wrapped in a `data` key by default; this can be disabled or customised.
- **Pagination awareness:** Resource collections automatically include `links` and `meta` keys when given a paginator instance.
- **Relationship safety:** `whenLoaded()` prevents N+1 query problems by including relationships only when they have been eager-loaded.

### Prerequisites

- PHP 8.1+ (Laravel 10+) or PHP 8.2+ (Laravel 11+).
- Composer dependency manager.
- A Laravel application with Eloquent models and controllers configured.
- Basic understanding of Eloquent relationships (`hasMany`, `belongsTo`, etc.).
- Familiarity with Laravel's service container and dependency injection.

### Related Programming Areas

- **Eloquent ORM** — Resources transform Eloquent models and collections into JSON.
- **API Controllers** — Resources are returned from controller methods.
- **FormRequest Validation** — Ensures data integrity before resources are created or updated.
- **Pagination** — Resource collections integrate with Laravel's paginator to produce `links` and `meta`.
- **Error Handling** — Resources interact with the exception handler when models are not found.
- **JSON:API Specification** — A standard that inspired many of Laravel's resource conventions.

### Core Concepts / Features

1. **Resource Classes:** Transforming single models into structured JSON objects via `JsonResource`.
2. **Collection Resources:** Wrapping arrays of data using `ResourceCollection` and custom metadata.
3. **Conditional Attributes:** Using `$this->when()`, `$this->mergeWhen()`, and conditional loading based on user permissions.
4. **Relationship Serialization:** Eager loading checks with `$this->whenLoaded()` to prevent N+1 query problems in JSON outputs.
5. **Response Consistency:** Standardising envelopes, data wrapping, and custom root keys.

---

## 1. Resource Classes

### Definitions

**Core Definition:** A resource class is a transformation class that defines how a single Eloquent model is converted into a structured JSON object.

**Technical Definition:** A resource class extends `Illuminate\Http\Resources\Json\JsonResource`. It defines a `toArray(Request $request): array` method that returns an associative array of attributes. The resource automatically proxies property and method access to the underlying model via the `$this` variable. Resources are generated with `php artisan make:resource UserResource` and stored in `app/Http/Resources/`. When a resource is returned from a controller, Laravel serialises it to JSON and wraps it in a `data` key by default.

**Beginner-Friendly Explanation:** A resource class is like a "recipe" for turning a database row into JSON. Instead of returning the raw model and hoping for the best, you write a small class that says "include the ID, the name, and the email, but format the date this way." This keeps your API output predictable and safe.

### Purposes

- To explicitly define which model attributes are exposed in API responses, preventing accidental data leaks.
- To transform raw model data (e.g., timestamps, booleans) into client-friendly formats (e.g., ISO 8601 strings, human-readable labels).
- To compute derived fields (e.g., excerpts, full URLs, formatted prices) without modifying the database schema.
- To provide a single source of truth for a resource's JSON representation, ensuring consistency across all endpoints that return that resource.
- To enable nested resource inclusion for relationships, producing hierarchical JSON structures.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Generate the resource
// php artisan make:resource UserResource

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
            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}
```

```php
// Step 2: Return the resource from a controller
use App\Http\Resources\UserResource;
use App\Models\User;

public function show(User $user): JsonResponse
{
    return response()->json(new UserResource($user));
}
```

**Component Breakdown:**

- `class UserResource extends JsonResource` — Extends the base resource class, inheriting all conditional and serialisation methods.
- `toArray(Request $request): array` — The method that defines the JSON structure. `$this` refers to the underlying Eloquent model.
- `$this->id`, `$this->name`, `$this->email` — Model attributes accessed directly through the resource's proxy.
- `new UserResource($user)` — Instantiates the resource with a model instance, ready for serialisation.

**Syntax Rules:**

- Resource classes **must** extend `JsonResource` (for single models) or `ResourceCollection` (for collections).
- The `toArray()` method **must** return an array; nested resources and collections are automatically serialised.
- The `$request` parameter provides access to the incoming request, enabling conditional logic based on user permissions or query parameters.
- Resources can be returned directly from routes or controllers; Laravel handles serialisation and response generation.
- The `data` wrapper is applied by default; this can be disabled globally via `JsonResource::withoutWrapping()` or per-class via `public static $wrap = null`.

**Constraints and Limitations:**

- **Resources should not perform database queries.** Use `whenLoaded()` to include relationships only when they have been eager-loaded, preventing N+1 problems.
- **The `data` wrapper is applied to the outermost resource only.** Nested resources are not wrapped unless they are returned as the outermost response.
- **Resources are serialised lazily.** The `toArray()` method is called when the response is rendered, not when the resource is instantiated.

### Annotated Code Examples

**Example 1: A Basic User Resource with Derived Fields**

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
            // Derived field: extract initials from the name
            'initials'   => collect(explode(' ', $this->name))
                                ->map(fn($part) => strtoupper($part[0]))
                                ->join(''),
            // Formatted date: ISO 8601 string
            'member_since' => $this->created_at->toIso8601String(),
            // Conditional field: only include phone if the viewer is the user
            'phone'      => $this->when(
                $request->user()?->id === $this->id,
                $this->phone
            ),
        ];
    }
}
```

**Step-by-Step Setup:**

1. Run `php artisan make:resource UserResource`.
2. Define the `toArray()` method as shown.
3. Ensure the `User` model has `name`, `email`, `phone`, and `created_at` attributes.
4. Return `new UserResource($user)` from a controller.

**Expected Output:**

```json
{
    "data": {
        "id": 1,
        "name": "Alice Johnson",
        "email": "alice@example.com",
        "initials": "AJ",
        "member_since": "2024-01-15T08:30:00+00:00",
        "phone": "+1-555-0100"
    }
}
```

**Why This Output Occurs:** The resource transforms the model's `name` into `initials` using a collection pipeline. The `created_at` timestamp is formatted as an ISO 8601 string. The `phone` field is included only because the authenticated user's ID matches the resource's ID — if a different user viewed the resource, the `phone` key would be absent.

---

**Example 2: Resource with Nested Relationship (No N+1)**

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
            'excerpt'    => substr($this->body, 0, 120) . '...',
            'author'     => new UserResource($this->whenLoaded('user')),
            'comments'   => CommentResource::collection($this->whenLoaded('comments')),
            'published_at' => $this->created_at->toIso8601String(),
        ];
    }
}
```

```php
// Controller — eager-load relationships
$post = Post::with(['user', 'comments.author'])->findOrFail($id);
return new PostResource($post);
```

**Expected Output:**

```json
{
    "data": {
        "id": 5,
        "title": "Getting Started with Laravel",
        "excerpt": "Laravel is a PHP framework that makes...",
        "author": {
            "data": { "id": 2, "name": "Alice", "email": "alice@example.com" }
        },
        "comments": {
            "data": [
                { "id": 10, "body": "Great post!", "author": { "data": { "id": 7, "name": "Bob" } } }
            ]
        },
        "published_at": "2025-06-01T09:30:00+00:00"
    }
}
```

**Why This Output Occurs:** The `whenLoaded('user')` call includes the `author` key only because the `user` relationship was eager-loaded via `->with('user')`. If the relationship had not been loaded, the key would be absent, and no additional query would be executed. This pattern prevents N+1 query problems in API responses.

### Real-World Cases

- **Public user profiles:** Resources expose only the fields that are safe for public consumption (name, avatar, bio) while omitting email, phone, and internal IDs.
- **Admin panels:** A single `UserResource` can conditionally include admin-only fields (e.g., `last_login_ip`) using `$this->when($request->user()->isAdmin(), ...)`.
- **E-commerce product listings:** Resources compute derived fields like `discounted_price`, `in_stock`, and `image_url` without modifying the database.
- **API versioning:** V1 and V2 resources can transform the same model differently, supporting backward compatibility while evolving the API.

---

## 2. Collection Resources

### Definitions

**Core Definition:** A collection resource is a transformation class that wraps an array or paginator of models, producing a JSON array with optional metadata such as pagination links and totals.

**Technical Definition:** Collection resources extend `Illuminate\Http\Resources\Json\ResourceCollection` and are generated with `php artisan make:resource UserCollection` or by naming the resource with the `Collection` suffix. By default, Laravel automatically wraps a collection of models in an anonymous `ResourceCollection` using the singular resource class for transformation. Custom collection resources override the `toArray()` method to define the collection-level structure, and can use the `with()` method to add metadata to the response. When a paginator is passed, Laravel automatically injects `links` and `meta` keys containing pagination information.

**Beginner-Friendly Explanation:** When your API returns a list of things — like a list of users or products — you need more than just an array. You often need to include pagination information (which page you're on, how many total items there are) and maybe some custom metadata. A collection resource handles this: it wraps the array of items and adds all the extra information the client needs to navigate the list.

### Purposes

- To standardise the JSON structure of collections, ensuring consistency across all list endpoints.
- To include pagination metadata (`links`, `meta`) automatically when returning paginated results.
- To add custom metadata (e.g., total counts, aggregations, version information) to collection responses.
- To control how individual items within the collection are transformed by referencing a singular resource class.
- To provide a dedicated class for collection-level logic, separating it from single-model transformation.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Generate the collection resource
// php artisan make:resource UserCollection

// File: app/Http/Resources/UserCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    // The resource that this collection collects.
    public $collects = UserResource::class;

    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
        ];
    }

    // Optional: Add custom metadata to the response.
    public function with(Request $request): array
    {
        return [
            'meta' => [
                'total_users' => $this->collection->count(),
                'api_version' => '1.0',
            ],
        ];
    }
}
```

```php
// Step 2: Return the collection from a controller
use App\Http\Resources\UserCollection;
use App\Models\User;

public function index(): UserCollection
{
    return new UserCollection(User::paginate(15));
    // Or: return UserResource::collection(User::paginate(15));
}
```

**Component Breakdown:**

- `class UserCollection extends ResourceCollection` — Extends the base collection resource class.
- `public $collects = UserResource::class` — Specifies which singular resource class transforms each item in the collection.
- `toArray(Request $request): array` — Defines the collection-level structure. Returning `['data' => $this->collection]` preserves the standard `data` wrapper.
- `with(Request $request): array` — Returns additional metadata to be merged into the response. The `with` method is called only for the outermost resource.
- `UserResource::collection($users)` — A shortcut for creating an anonymous collection resource using the specified singular resource.

**Syntax Rules:**

- Collection resources **must** extend `ResourceCollection` (not `JsonResource`).
- The `$collects` property links the collection to its singular resource; if omitted, Laravel infers it from the class name (e.g., `UserCollection` → `UserResource`).
- The `with()` method is called only when the collection is the outermost resource in the response; nested collections do not invoke it.
- Paginated collections automatically include `links` and `meta` keys. Custom keys added via `with()` are merged into the `meta` array, avoiding conflicts with pagination metadata.
- The `withoutWrapping()` method does **not** remove the `data` key from paginated responses; pagination always wraps data in `data`.

**Constraints and Limitations:**

- **Custom collection metadata added via `with()` is only applied to the outermost collection.** If a collection is nested inside another resource, its `with()` method is not called.
- **The `data` key cannot be removed from paginated collections** even when `withoutWrapping()` is called. This is because pagination requires the `data`, `links`, and `meta` structure.
- **The `$collects` property must reference a valid `JsonResource` class.** If it does not exist, Laravel throws an exception during serialisation.

### Annotated Code Examples

**Example 1: Paginated Collection with Custom Metadata**

```php
<?php
// File: app/Http/Resources/PostCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class PostCollection extends ResourceCollection
{
    public $collects = PostResource::class;

    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
        ];
    }

    public function with(Request $request): array
    {
        return [
            'meta' => [
                'total_posts' => $this->collection->total(),
                'published_today' => $this->collection->filter(
                    fn($post) => $post->created_at->isToday()
                )->count(),
            ],
        ];
    }
}
```

```php
// Controller
use App\Http\Resources\PostCollection;
use App\Models\Post;

public function index(): PostCollection
{
    return new PostCollection(Post::with('user')->latest()->paginate(15));
}
```

**Expected Output:**

```json
{
    "data": [
        { "id": 1, "title": "First Post", "author": { "data": { "id": 2, "name": "Alice" } } },
        { "id": 2, "title": "Second Post", "author": { "data": { "id": 3, "name": "Bob" } } }
    ],
    "links": {
        "first": "https://example.com/api/posts?page=1",
        "last": "https://example.com/api/posts?page=5",
        "prev": null,
        "next": "https://example.com/api/posts?page=2"
    },
    "meta": {
        "current_page": 1,
        "per_page": 15,
        "total": 72,
        "total_posts": 72,
        "published_today": 3
    }
}
```

**Why This Output Occurs:** The `PostCollection` wraps the paginator in a `data` key. Laravel automatically merges the pagination `links` and `meta` keys. The custom `with()` method adds `total_posts` and `published_today` to the `meta` array. Because the collection is the outermost resource, `with()` is invoked and its metadata is merged with the pagination metadata without conflict.

---

**Example 2: Non-Paginated Collection with Custom Root Key**

```php
<?php
// File: app/Http/Resources/CountryCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class CountryCollection extends ResourceCollection
{
    public $collects = CountryResource::class;

    public function toArray(Request $request): array
    {
        // Custom root key instead of 'data'
        return [
            'countries' => $this->collection,
        ];
    }
}
```

```php
// Controller
return new CountryCollection(Country::all());
```

**Expected Output:**

```json
{
    "countries": [
        { "code": "US", "name": "United States" },
        { "code": "GB", "name": "United Kingdom" }
    ]
}
```

**Why This Output Occurs:** The `toArray()` method explicitly returns an array with a `countries` key instead of the default `data` key. This overrides the default wrapping behaviour for this specific collection resource. Note that if `withoutWrapping()` is called globally, this custom key is preserved because it is explicitly defined in `toArray()`.

### Real-World Cases

- **Paginated list endpoints:** Every `GET /api/posts`, `GET /api/users`, etc., returns a `PostCollection` or `UserCollection` with pagination links and metadata.
- **Dashboard summary endpoints:** A collection resource can include aggregate metadata (e.g., total revenue, active users) alongside the list of items.
- **Search results:** A `SearchResultCollection` can include metadata about the query (e.g., `total_results`, `search_time_ms`) in addition to the result items.
- **Multi-language content:** A collection resource can include a `meta.locale` key indicating the language of the returned content.

---

## 3. Conditional Attributes

### Definitions

**Core Definition:** Conditional attributes are resource fields that are included in the JSON output only when a specified condition evaluates to true, enabling context-aware responses based on user permissions, request parameters, or model state.

**Technical Definition:** Laravel's `ConditionallyLoadsAttributes` trait, used by `JsonResource`, provides methods such as `when(bool $condition, mixed $value, mixed $default = null)` and `mergeWhen(bool $condition, mixed $value, mixed $default = new MissingValue())`. The `when()` method returns the value if the condition is true, or a `MissingValue` instance (which is removed during filtering) if false. The `mergeWhen()` method merges an array of attributes when the condition is true. The `whenHas()` method conditionally includes an attribute if it exists on the model, and `whenAppended()` checks for accessor-appended attributes.

**Beginner-Friendly Explanation:** Not every client should see every field. An admin might see a user's email, but a regular user should not. A resource can say "include this field only if the viewer is an admin" or "include this relationship only if it was already loaded." Conditional attributes make this easy: you wrap the field in a `when()` call, and Laravel automatically removes it from the response if the condition is not met.

### Purposes

- To include sensitive or privileged data only when the authenticated user has the appropriate permissions.
- To conditionally include relationships based on whether they have been eager-loaded, preventing N+1 queries.
- To adapt the response structure based on request parameters (e.g., `?include=comments`).
- To omit fields that are null or missing, reducing payload size and avoiding null values in JSON.
- To merge multiple attributes under a single condition, reducing repetitive `when()` calls.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// When — conditionally include a single attribute
'email' => $this->when(
    $request->user()->isAdmin(),
    $this->email
),

// When with a closure — compute the value only if the condition is true
'secret' => $this->when(
    $request->user()->isAdmin(),
    fn() => 'secret-value'
),

// MergeWhen — conditionally include multiple attributes
$this->mergeWhen($request->user()->isAdmin(), [
    'first-secret' => 'value',
    'second-secret' => 'value',
]),

// WhenLoaded — include a relationship only if it was eager-loaded
'posts' => PostResource::collection($this->whenLoaded('posts')),

// WhenHas — include an attribute only if it exists on the model
'tax' => $this->whenHas('price', fn() => number_format($this->price * 0.21, 2)),

// WhenAppended — include an accessor-appended attribute
'address_lines' => $this->whenAppended('address_lines', $this->address_lines),
```

**Component Breakdown:**

- `when(bool $condition, mixed $value)` — Returns `$value` if `$condition` is true; otherwise, a `MissingValue` instance is returned and removed during filtering.
- `mergeWhen(bool $condition, array $value)` — Merges the array of attributes into the response if `$condition` is true.
- `whenLoaded(string $relationship, mixed $value = null)` — Returns the relationship value only if `relationLoaded($relationship)` returns true on the model.
- `whenHas(string $attribute, mixed $value = null)` — Returns the value only if the attribute exists in the model's attributes array.
- `whenAppended(string $attribute, mixed $value = null)` — Returns the value only if the attribute is in the model's `$appends` array.

**Syntax Rules:**

- The `when()` method accepts a boolean condition as its first argument and the value to return as its second. A closure can be passed as the second argument to defer value computation.
- The `mergeWhen()` method **should not** be used within arrays that mix string and numeric keys, or within arrays with numeric keys that are not ordered sequentially.
- The `whenLoaded()` method accepts the relationship name as a string, not the relationship itself. This prevents accidental lazy loading.
- The `whenHas()` method checks `array_key_exists()` on the model's attributes, meaning the attribute must be present in the database query result.
- The `whenAppended()` method checks the model's `$appends` array, which is populated by accessor methods.

**Constraints and Limitations:**

- **`when()` returns a `MissingValue` instance when the condition is false**, which is removed during the `filter()` phase of resource serialisation. If you return this value from a nested resource, it may cause unexpected behaviour.
- **`whenLoaded()` only works with Eloquent relationships**, not with arbitrary attributes or computed properties.
- **`mergeWhen()` cannot be used at the top level of an array with numeric keys**; it must be used within an associative array.
- **Conditional attributes are evaluated during serialisation**, not when the resource is instantiated. This means the request object must be available at serialisation time.

### Annotated Code Examples

**Example 1: Admin-Only Fields with `when()` and `mergeWhen()`**

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

            // Single conditional attribute: only include phone
            // if the authenticated user is viewing their own profile.
            'phone' => $this->when(
                $request->user()?->id === $this->id,
                $this->phone
            ),

            // Merge multiple admin-only attributes under one condition.
            // This is more efficient than multiple when() calls.
            $this->mergeWhen($request->user()?->isAdmin(), [
                'last_login_ip'  => $this->last_login_ip,
                'login_count'    => $this->login_count,
                'account_status' => $this->account_status,
            ]),

            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}
```

**Step-by-Step Setup:**

1. Ensure the `User` model has `phone`, `last_login_ip`, `login_count`, and `account_status` attributes.
2. Define the resource as shown.
3. Return `new UserResource($user)` from a controller.

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
        "last_login_ip": "192.168.1.100",
        "login_count": 42,
        "account_status": "active",
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

**Why This Output Occurs:** The `when()` call for `phone` checks if the authenticated user's ID matches the resource's ID. If true, the phone is included; if false, the key is omitted. The `mergeWhen()` call merges three admin-only fields into the response only when `isAdmin()` returns true. Because `mergeWhen()` returns an array, the fields are merged at the top level of the JSON object.

---

**Example 2: Using `whenLoaded()` and `whenHas()` to Prevent N+1 and Missing Attributes**

```php
<?php
// File: app/Http/Resources/ProductResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'    => $this->id,
            'name'  => $this->name,
            'price' => $this->price,

            // Only include category if it was eager-loaded.
            // Without this, accessing $this->category would trigger
            // a separate query for every product (N+1 problem).
            'category' => new CategoryResource($this->whenLoaded('category')),

            // Only include reviews if eager-loaded.
            'reviews' => ReviewResource::collection($this->whenLoaded('reviews')),

            // Only include tax if the price attribute exists.
            // This prevents errors when the price column is omitted
            // from a selective query.
            'tax' => $this->whenHas('price', fn() => number_format($this->price * 0.21, 2)),

            // Only include discount if it was appended via an accessor.
            'discount' => $this->whenAppended('discount'),
        ];
    }
}
```

```php
// Controller — eager-load relationships, but optionally omit price
public function index(Request $request)
{
    $query = Product::with(['category', 'reviews']);

    // If the client requests a lightweight list, omit the price column
    if ($request->boolean('lightweight')) {
        $query->select('id', 'name');
    }

    return ProductResource::collection($query->paginate(20));
}
```

**Expected Output (full query):**

```json
{
    "data": [
        {
            "id": 1,
            "name": "Widget",
            "price": 9.99,
            "category": { "data": { "id": 3, "name": "Tools" } },
            "reviews": { "data": [ { "id": 10, "rating": 5 } ] },
            "tax": "2.10",
            "discount": "10%"
        }
    ]
}
```

**Expected Output (lightweight query without `price`):**

```json
{
    "data": [
        {
            "id": 1,
            "name": "Widget",
            "category": { "data": { "id": 3, "name": "Tools" } },
            "reviews": { "data": [ { "id": 10, "rating": 5 } ] }
        }
    ]
}
```

**Why This Output Occurs:** In the lightweight query, the `select('id', 'name')` clause omits the `price` column. The `whenHas('price', ...)` method checks `array_key_exists('price', $this->resource->getAttributes())`. Because `price` is not present, the `tax` key is omitted entirely — no error is thrown, and no meaningless null value appears in the JSON. The `whenLoaded()` calls ensure that relationships are only included if they were eager-loaded.

### Real-World Cases

- **Public vs. admin API responses:** The same `UserResource` returns different fields depending on whether the requester is an admin, a regular user viewing their own profile, or a guest.
- **Query-parameter-driven responses:** A `?include=comments` parameter can control whether `whenLoaded('comments')` includes the relationship (provided the controller eager-loads it conditionally).
- **Sparse fieldsets (JSON:API style):** Conditional attributes can be combined with query parameters to let clients request only the fields they need.
- **Multi-tenant applications:** Conditional attributes can include tenant-specific fields only when the authenticated user belongs to that tenant.

---

## 4. Relationship Serialization

### Definitions

**Core Definition:** Relationship serialisation is the process of including related models (e.g., a post's author, a product's category) in a resource's JSON output, with safeguards to prevent N+1 query problems.

**Technical Definition:** Laravel resources include relationships via the `whenLoaded()` method, which checks whether a relationship has been eager-loaded on the underlying model using `relationLoaded()`. If the relationship is loaded, the method returns the relationship value, which is typically wrapped in another resource or resource collection for transformation. If the relationship is not loaded, a `MissingValue` is returned and filtered out of the response. This pattern ensures that relationships are only serialised when the controller has explicitly eager-loaded them, preventing the N+1 query problem where each model in a collection triggers a separate query for its relationship.

**Beginner-Friendly Explanation:** When you return a list of 100 posts, each post has an author. If you naively include the author for each post, Laravel would run 100 separate queries — one for each author. This is the N+1 problem. The solution is to "eager-load" all authors in a single query, then tell the resource "include the author only if it was already loaded." The `whenLoaded()` method does exactly this: it includes the relationship if it was eager-loaded, and silently omits it if not. This keeps your API fast and your database happy.

### Purposes

- To include related model data in JSON responses without triggering additional database queries.
- To prevent the N+1 query problem by requiring explicit eager-loading in controllers.
- To allow controllers to decide which relationships to include based on the endpoint or request parameters.
- To transform related models using their own resource classes, ensuring consistency.
- To support conditional relationship inclusion based on whether the relationship exists or has been loaded.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Including a hasOne or belongsTo relationship
'author' => new UserResource($this->whenLoaded('user')),

// Including a hasMany relationship
'comments' => CommentResource::collection($this->whenLoaded('comments')),

// Including a nested relationship (relationship of a relationship)
'comments' => CommentResource::collection(
    $this->whenLoaded('comments', fn() => $this->comments->load('author'))
),

// Conditional relationship based on request parameter
'posts' => PostResource::collection(
    $this->when(
        $request->boolean('include_posts'),
        fn() => $this->posts
    )
),
```

```php
// Controller — eager-load relationships
$posts = Post::with(['user', 'comments.author'])->paginate(15);
return PostResource::collection($posts);
```

**Component Breakdown:**

- `$this->whenLoaded('user')` — Returns the `user` relationship value if it has been eager-loaded; otherwise returns a `MissingValue`.
- `new UserResource($this->whenLoaded('user'))` — Wraps the relationship in its own resource class for transformation.
- `CommentResource::collection($this->whenLoaded('comments'))` — Wraps a collection relationship in its collection resource.
- `$this->whenLoaded('comments', fn() => $this->comments->load('author'))` — A closure can be passed to load nested relationships only when the parent is loaded.
- `->with(['user', 'comments.author'])` — Eager-loads relationships in the controller before returning the resource.

**Syntax Rules:**

- The `whenLoaded()` method **must** receive the relationship name as a string, not the relationship object. This is intentional: passing the relationship object would trigger a lazy load if it was not already loaded.
- The relationship **must** be eager-loaded in the controller using `->with()` or `->load()` for `whenLoaded()` to include it.
- Nested resources (e.g., `UserResource`) are wrapped in their own `data` key unless wrapping is disabled. Laravel prevents double-wrapping of the outermost resource.
- For nested relationships deeper than one level (e.g., `origin.season`), use `->when()` with a manual `relationLoaded()` check instead of `whenLoaded()`, because `whenLoaded()` may trigger lazy loading for nested relations.

**Constraints and Limitations:**

- **`whenLoaded()` does not eager-load relationships.** It only checks whether the relationship has already been loaded. If the controller does not eager-load, the relationship is silently omitted.
- **Nested `whenLoaded()` calls can trigger lazy loading.** If the parent relationship is loaded but the child is not, accessing the child can trigger a separate query. Use `->when()` with `relationLoaded()` for deep nesting.
- **Polymorphic relationships require special handling.** The `whenLoaded()` method works with polymorphic relationships, but the morph type may need to be checked before including the relationship.
- **Lazy loading prevention is critical in production.** Laravel's `Model::preventLazyLoading()` method can be enabled in development to catch accidental lazy loads.

### Annotated Code Examples

**Example 1: Eager Loading with `whenLoaded()` to Prevent N+1**

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
            'id'      => $this->id,
            'title'   => $this->title,
            'author'  => new UserResource($this->whenLoaded('user')),
            'comments' => CommentResource::collection($this->whenLoaded('comments')),
        ];
    }
}
```

```php
// Controller — eager-load both relationships
public function index(): PostCollection
{
    // Single query for posts, one for users, one for comments
    $posts = Post::with(['user', 'comments'])->paginate(15);
    return new PostCollection($posts);
}
```

**Expected Output:**

```json
{
    "data": [
        {
            "id": 1,
            "title": "First Post",
            "author": { "data": { "id": 2, "name": "Alice" } },
            "comments": { "data": [ { "id": 10, "body": "Great!" } ] }
        },
        {
            "id": 2,
            "title": "Second Post",
            "author": { "data": { "id": 3, "name": "Bob" } },
            "comments": { "data": [] }
        }
    ]
}
```

**Why This Output Occurs:** The controller eager-loads `user` and `comments` using `->with(['user', 'comments'])`. This executes three queries total: one for posts, one for all related users, and one for all related comments. The `whenLoaded()` calls in the resource check whether these relationships are loaded. Because they are, the resource includes them. Without `whenLoaded()`, accessing `$this->user` for each post would trigger a separate query — the N+1 problem.

---

**Example 2: Nested Relationship Loading with `when()`**

```php
<?php
// File: app/Http/Resources/OrderResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class OrderResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'     => $this->id,
            'total'  => $this->total,

            // Include customer if loaded
            'customer' => new UserResource($this->whenLoaded('customer')),

            // Include items and their products if loaded
            'items' => OrderItemResource::collection(
                $this->whenLoaded('items')
            ),

            // Nested relationship with manual load check
            // (whenLoaded would trigger lazy loading for items.product)
            'items_with_products' => $this->when(
                $this->relationLoaded('items'),
                fn() => OrderItemResource::collection(
                    $this->items->load('product')
                )
            ),
        ];
    }
}
```

```php
// Controller
$order = Order::with(['customer', 'items'])->findOrFail($id);
return new OrderResource($order);
```

**Expected Output:**

```json
{
    "data": {
        "id": 100,
        "total": 249.99,
        "customer": { "data": { "id": 5, "name": "Carol" } },
        "items": {
            "data": [
                { "id": 1, "product_id": 10, "quantity": 2 },
                { "id": 2, "product_id": 15, "quantity": 1 }
            ]
        },
        "items_with_products": {
            "data": [
                { "id": 1, "product": { "data": { "id": 10, "name": "Widget" } }, "quantity": 2 },
                { "id": 2, "product": { "data": { "id": 15, "name": "Gadget" } }, "quantity": 1 }
            ]
        }
    }
}
```

**Why This Output Occurs:** The controller eager-loads `customer` and `items`. The `whenLoaded('items')` call includes the items collection. The `when()` call with `relationLoaded('items')` manually checks if items are loaded, then loads the `product` relationship on the items collection via `->load('product')`. This executes one additional query for all products, rather than one query per item. Using `whenLoaded('items.product')` would not work correctly for nested relationships and could trigger lazy loading.

### Real-World Cases

- **Blog APIs:** Posts include their author and comments; comments include their author. All relationships are eager-loaded in the controller and conditionally serialised via `whenLoaded()`.
- **E-commerce orders:** Orders include customers, items, and each item's product. Nested relationships are loaded manually to avoid N+1.
- **Social media feeds:** Posts include users, likes, and comments. `whenLoaded()` ensures that only eager-loaded relationships appear in the feed, keeping response times low.
- **Admin dashboards:** Controllers decide which relationships to eager-load based on the dashboard view; resources include them conditionally, so the same resource works across different views.

---

## 5. Response Consistency

### Definitions

**Core Definition:** Response consistency refers to the standardisation of JSON response structure across all API endpoints, including the use of envelopes, data wrapping, and custom root keys.

**Technical Definition:** By default, Laravel wraps the outermost resource in a `data` key. This can be customised by defining a `public static $wrap` property on a resource class, disabling globally via `JsonResource::withoutWrapping()` in a service provider, or overriding the `toArray()` method to return a custom root key. Paginated responses always include `data`, `links`, and `meta` keys, regardless of wrapping configuration. Custom metadata can be added via the `with()` method on collection resources or the `additional()` method on individual resources.

**Beginner-Friendly Explanation:** Imagine if one API endpoint returned `{"users": [...]}` and another returned `{"data": [...]}` and a third returned just `[...]`. Clients would have to write different code for each. Response consistency means deciding on one structure — like always wrapping in a `data` key — and applying it everywhere. Laravel's default wrapping handles this automatically, but you can customise it if your API needs a different envelope.

### Purposes

- To ensure that all API endpoints return JSON in the same structure, reducing client-side parsing complexity.
- To control the root key of responses, allowing custom envelopes such as `{ "users": [...] }` or `{ "result": {...} }`.
- To disable wrapping entirely for APIs that require a flat JSON structure.
- To add consistent metadata to all collection responses (pagination, totals, version).
- To prevent double-wrapping of nested resources and ensure predictable nesting behaviour.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Default wrapping — outermost resource is wrapped in 'data'
return new UserResource($user);
// Output: { "data": { "id": 1, ... } }

// Custom root key via $wrap property
class UserResource extends JsonResource
{
    public static $wrap = 'user';
}
// Output: { "user": { "id": 1, ... } }

// Disable wrapping per-class
class UserResource extends JsonResource
{
    public static $wrap = null;
}
// Output: { "id": 1, ... }

// Disable wrapping globally
// File: app/Providers/AppServiceProvider.php
public function boot(): void
{
    JsonResource::withoutWrapping();
}
// Output: { "id": 1, ... } (no 'data' key)

// Adding metadata to a resource
return (new UserResource($user))->additional([
    'meta' => ['version' => '1.0'],
]);
// Output: { "data": {...}, "meta": {"version": "1.0"} }
```

**Component Breakdown:**

- `public static $wrap = 'user'` — Overrides the default `data` key for this resource class only.
- `public static $wrap = null` — Disables wrapping for this resource class only.
- `JsonResource::withoutWrapping()` — Globally disables wrapping for all resources extending `JsonResource`. Typically called in `AppServiceProvider::boot()`.
- `->additional(['meta' => [...]])` — Adds top-level metadata to the resource response. Merged with pagination `meta` if present.
- `with()` method on collection resources — Returns metadata that is automatically merged into the response.

**Syntax Rules:**

- The `$wrap` property **must** be `public static` to be effective.
- `withoutWrapping()` affects only the outermost resource; nested resources are not wrapped unless they are returned as the outermost response.
- Paginated collections **always** include the `data` key, even when `withoutWrapping()` is called. This is because `links` and `meta` are required for pagination.
- Custom metadata added via `additional()` or `with()` is merged into the `meta` key of the response. Pagination metadata is preserved.
- `withoutWrapping()` is typically called in a service provider's `boot()` method to apply globally.

**Constraints and Limitations:**

- **`withoutWrapping()` does not remove the `data` key from paginated responses.** If you need a flat paginated response, you must customise the collection resource's `toArray()` method.
- **Defining `public static $wrap = null` on a resource class overrides the global `withoutWrapping()` setting** for that class only.
- **Custom root keys defined in `toArray()` (e.g., `['users' => $this->collection]`) are not affected by `withoutWrapping()`** because they are explicitly returned.
- **`additional()` metadata is merged into the `meta` key.** If you use `additional()` to add a `meta` key that already exists (e.g., from pagination), the values are merged, with the resource's values taking precedence.

### Annotated Code Examples

**Example 1: Global `withoutWrapping()` for a Flat API**

```php
<?php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Disable the 'data' wrapper globally for all resources.
        // This produces a flatter JSON structure:
        // { "id": 1, "name": "Alice" }
        // instead of:
        // { "data": { "id": 1, "name": "Alice" } }
        JsonResource::withoutWrapping();
    }
}
```

```php
// Controller — no changes needed
return new UserResource($user);
```

**Expected Output:**

```json
{
    "id": 1,
    "name": "Alice Johnson",
    "email": "alice@example.com"
}
```

**Why This Output Occurs:** The `withoutWrapping()` call in the service provider's `boot()` method removes the `data` key from all resource responses. This is applied globally, so every resource returned from the application produces a flat JSON structure. This is useful for APIs that must match a pre-existing specification (e.g., legacy APIs or JSON:API without the `data` envelope).

---

**Example 2: Custom Root Key and Metadata**

```php
<?php
// File: app/Http/Resources/ProductCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class ProductCollection extends ResourceCollection
{
    public $collects = ProductResource::class;

    public function toArray(Request $request): array
    {
        // Custom root key: 'products' instead of 'data'
        return [
            'products' => $this->collection,
        ];
    }

    public function with(Request $request): array
    {
        return [
            'meta' => [
                'currency'    => 'USD',
                'api_version' => '2.0',
            ],
        ];
    }
}
```

```php
// Controller
return new ProductCollection(Product::paginate(20));
```

**Expected Output:**

```json
{
    "products": [
        { "id": 1, "name": "Widget", "price": 9.99 },
        { "id": 2, "name": "Gadget", "price": 19.99 }
    ],
    "links": {
        "first": "https://example.com/api/products?page=1",
        "last": "https://example.com/api/products?page=3",
        "prev": null,
        "next": "https://example.com/api/products?page=2"
    },
    "meta": {
        "current_page": 1,
        "per_page": 20,
        "total": 55,
        "currency": "USD",
        "api_version": "2.0"
    }
}
```

**Why This Output Occurs:** The `toArray()` method returns an array with a `products` key instead of the default `data` key. This overrides the default wrapping for this collection. The `with()` method adds `currency` and `api_version` to the `meta` array. Laravel automatically merges the pagination `links` and `meta` keys, so the final response contains both pagination metadata and custom metadata in the `meta` object.

### Real-World Cases

- **Public APIs with established specifications:** Some APIs require a flat JSON structure (no `data` wrapper) to comply with existing client implementations. `withoutWrapping()` enables this without modifying every resource.
- **Multi-version APIs:** V1 resources can use the default `data` wrapper, while V2 resources use a custom root key like `result` to signal a breaking change.
- **API gateways with response transformers:** Gateways may expect a specific envelope (e.g., `{ "status": "success", "data": {...} }`); resources can be customised to produce this structure.
- **Consistent error and success responses:** By standardising the response envelope, clients can handle all responses uniformly — checking `data` for success, `errors` for validation failures, and `meta` for pagination.

---

## References

- Laravel Eloquent: API Resources Documentation (13.x) — https://laravel.com/framework/docs/eloquent-resources
- Laravel Eloquent: API Resources Documentation (10.x) — https://laravel.com/framework/docs/10.x/eloquent-resources
- Laravel `JsonResource` API — https://api.laravel.com/docs/10.x/Illuminate/Http/Resources/Json/JsonResource.html
- Laravel `ResourceCollection` API — https://api.laravel.com/docs/10.x/Illuminate/Http/Resources/Json/ResourceCollection.html
- Laravel API Resources with Relations: Methods to Avoid N+1 Query (Laravel Daily) — https://laraveldaily.com/post/laravel-api-resources-relations-when-methods
- Using `withoutWrapping` to Flatten API Responses (Laravel News) — https://laravel-news.com/without-wrapping
- Eloquent API Resources: Best Practices for Transforming Data in Laravel (TSECURITY) — https://tsecurity.de/de/2661028/it+programmierung/eloquent+api+resources:+best+practices+for+transforming+data+in+laravel/
- Laravel `ConditionallyLoadsAttributes` Trait — https://api.laravel.com/docs/10.x/Illuminate/Http/Resources/Concerns/ConditionallyLoadsAttributes.html
- Laravel API Resources: `whenLoaded()` for Deeper Relations (Stack Overflow) — https://stackoverflow.com/questions/50002845/how-do-i-use-whenloaded-for-deeper-than-one-level-relations
- Laravel `JsonResource::withoutWrapping()` — https://api.laravel.com/docs/10.x/Illuminate/Http/Resources/Json/JsonResource.html#method_withoutWrapping
- Laravel `JsonResource::additional()` — https://api.laravel.com/docs/10.x/Illuminate/Http/Resources/Json/JsonResource.html#method_additional
- Laravel `ResourceCollection::with()` — https://api.laravel.com/docs/10.x/Illuminate/Http/Resources/Json/ResourceCollection.html#method_with