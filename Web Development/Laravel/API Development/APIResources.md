# Laravel API Resources: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel API Resources are transformation classes that sit between Eloquent models and the JSON responses returned to API consumers, providing a controlled layer for selecting, formatting, and structuring model data.

**Technical Definition**

API Resources are classes extending `Illuminate\Http\Resources\Json\JsonResource` (for single models) or `Illuminate\Http\Resources\Json\ResourceCollection` (for collections). They define a `toArray(Request $request): array` method that returns the attributes to be serialized to JSON. Resources support conditional attributes (`when()`, `whenLoaded()`, `whenNotNull()`, `whenHas()`), relationship inclusion via `whenLoaded()`, pagination metadata, and custom wrapping. The `make:resource` Artisan command generates resource classes in `app/Http/Resources`.

**Beginner-Friendly Explanation**

Imagine you have a database table full of user records. If you sent all that data to your API users, you would expose everything—including maybe the password hash and internal IDs. API Resources are a filter that decides exactly what data goes out. You write a simple class that says "only send the name, email, and avatar" and Laravel handles the rest. It is like a bouncer for your data: only the allowed fields get through.

---

### Key Characteristics

- **Data Transformation Layer**: Resources sit between models and JSON responses, decoupling database schema from API output.
- **Explicit Field Selection**: Only the fields defined in `toArray()` are included in the response.
- **Conditional Attributes**: The `when()`, `whenLoaded()`, `whenNotNull()`, and `whenHas()` methods allow attributes to be included based on request context, permissions, or relationship loading state.
- **Automatic Pagination Metadata**: When used with paginated data, resources automatically include `links` and `meta` keys with pagination information.
- **Relationship Handling**: Relationships can be included via nested resources and conditionally via `whenLoaded()` to prevent N+1 queries.
- **Data Wrapping**: Responses are wrapped in a `data` key by default, reserving other keys for metadata and errors.
- **Works with Any Data**: Resources can transform not only Eloquent models but also plain arrays and objects.
- **JSON:API Support**: Laravel 12.45+ includes a `JsonApiResource` class for JSON:API-compliant responses via the `--json-api` flag.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Eloquent models** set up for the resources to transform.
- **A configured API route file** (`routes/api.php`), created via `php artisan install:api`.
- **Basic understanding of JSON** and HTTP responses.

---

### Related Programming Areas

- **Eloquent ORM**: Resources transform Eloquent models and their relationships into JSON.
- **API Development**: Resources are the response formatting layer for all API endpoints.
- **Pagination**: Resources integrate with `paginate()` and `cursorPaginate()` to produce paginated JSON responses.
- **API Documentation**: Tools like Scramble and Scribe can generate documentation from API Resources.
- **JSON:API Specification**: Laravel's `JsonApiResource` provides a compliant implementation of the JSON:API standard.

---

## Core Concepts

---

### 1. API Resources

#### Definitions

**Core Definition**

An API Resource is a PHP class that transforms an Eloquent model or other data structure into a structured JSON array for API responses.

**Technical Definition**

`JsonResource` wraps a single model instance. The `toArray(Request $request)` method defines the output structure. The `$this` variable inside the resource proxies property and method access to the underlying model. Resources are returned directly from controllers or routes and are automatically serialized to JSON. The `toResource()` method on models provides framework-convention-based resource discovery.

**Beginner-Friendly Explanation**

An API Resource is a class that acts as a translator. You give it a model, and it outputs a clean JSON array with only the fields you want. For example, a `UserResource` might output `id`, `name`, and `email` but hide `password` and `remember_token`.

#### Purposes

- To control exactly which model attributes are exposed in API responses.
- To transform attribute values (e.g., format dates, compute derived fields).
- To provide a consistent JSON structure across all endpoints.
- To prevent accidental exposure of sensitive or internal data.
- To decouple the API contract from the database schema.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Http/Resources/UserResource.php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at?->toIso8601String(),
        ];
    }
}
```

```php
// Controller usage
use App\Http\Resources\UserResource;
use App\Models\User;

return new UserResource(User::findOrFail($id));
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends JsonResource` | Base class for single-model resources. |
| `toArray(Request $request)` | Defines the JSON output structure. |
| `$this` | Proxies to the underlying model instance. |
| `new UserResource($user)` | Wraps a model for transformation. |

**Syntax Rules**

- Resources are generated via `php artisan make:resource UserResource`.
- Resources are placed in `app/Http/Resources`.
- `toArray()` must return an array.
- The underlying model is accessible via `$this`.
- Resources can be returned directly from controllers or routes.

**Constraints and Limitations**

- Resources add a `data` wrapper by default (can be disabled via `JsonResource::withoutWrapping()`).
- `toArray()` should not perform database queries; all relationships must be eager-loaded before the resource is created.
- Resource classes should not contain business logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic User Resource**

**Resource**

```php
<?php
// app/Http/Resources/UserResource.php
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
        ];
    }
}
```

**Controller**

```php
// app/Http/Controllers/Api/UserController.php
use App\Http\Resources\UserResource;
use App\Models\User;

public function show(string $id)
{
    return new UserResource(User::findOrFail($id));
}
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com"
    }
}
```

**Why This Output Occurs**

The `UserResource` transforms the `User` model into a JSON object with only the specified fields, wrapped in a `data` key.

---

**Example 2: Using `toResource()` for Automatic Discovery**

```php
// In a controller
return User::findOrFail($id)->toResource();
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com"
    }
}
```

**Why This Output Occurs**

Laravel automatically discovers the `UserResource` class based on the model name and uses it to transform the model.

---

#### Real-World Cases with Explanation

**Case 1: Public API with Sensitive Data Protection**

A public API returns user profiles but must never expose `email`, `password_hash`, or `remember_token`. A `UserResource` includes only `id`, `name`, and `avatar_url`, preventing sensitive data leakage.

**Explanation**: API Resources provide a safety layer that prevents accidental exposure of database schema and sensitive attributes.

---

### 2. Resource Creation

#### Definitions

**Core Definition**

Resource creation is the process of generating a new API Resource class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:resource` command generates a PHP class in `app/Http/Resources` extending `JsonResource`. The `--collection` flag (or appending `Collection` to the name) generates a `ResourceCollection` class for transforming collections. The `--json-api` flag (Laravel 12.45+) generates a `JsonApiResource` class for JSON:API-compliant responses.

**Beginner-Friendly Explanation**

Creating a resource is a one-command process. Laravel generates the file for you, and you fill in the fields you want to expose.

#### Purposes

- To automate the generation of resource class boilerplate.
- To ensure resources are placed in the correct directory and namespace.
- To support both single-resource and collection-resource generation.
- To enable JSON:API-compliant resources.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Single resource
php artisan make:resource UserResource

# Resource collection
php artisan make:resource User --collection
php artisan make:resource UserCollection

# JSON:API resource (Laravel 12.45+)
php artisan make:resource PostResource --json-api
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:resource UserResource` | Creates `app/Http/Resources/UserResource.php`. |
| `make:resource User --collection` | Creates `app/Http/Resources/UserCollection.php`. |
| `make:resource UserCollection` | Creates a collection resource (name-based detection). |
| `--json-api` | Creates a `JsonApiResource`. |

**Syntax Rules**

- The resource name should be PascalCase and end with `Resource` (convention).
- Collection resources extend `ResourceCollection`.
- The `--collection` flag and `Collection` suffix are equivalent.

**Constraints and Limitations**

- Generated files must be edited to add the `toArray()` logic.
- The `--json-api` flag is available in Laravel 12.45+ only.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Resource**

**Command**

```bash
php artisan make:resource UserResource
```

**Generated File**

```php
<?php
// app/Http/Resources/UserResource.php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }
}
```

**Expected Output (Terminal)**

```
INFO  Resource [app/Http/Resources/UserResource.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the resource class with a default `toArray()` method.

---

**Example 2: Generating a Collection Resource**

**Command**

```bash
php artisan make:resource User --collection
```

**Generated File**

```php
<?php
// app/Http/Resources/UserCollection.php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }
}
```

**Expected Output (Terminal)**

```
INFO  Resource [app/Http/Resources/UserCollection.php] created successfully.
```

**Why This Output Occurs**

The `--collection` flag generates a `ResourceCollection` class instead of a `JsonResource`.

---

#### Real-World Cases with Explanation

**Case 1: Rapid API Development**

A developer runs `php artisan make:resource ProductResource` and immediately edits it to expose `id`, `name`, `price`, and `category`—a fully functional API transformation layer in minutes.

**Explanation**: Artisan accelerates development by generating appropriate resource stubs.

---

### 3. `artisan make:resource`

#### Definitions

**Core Definition**

`artisan make:resource` is the Artisan command used to generate new API Resource classes.

**Technical Definition**

The command is implemented by `Illuminate\Foundation\Console\ResourceMakeCommand`. It generates a PHP class file in `app/Http/Resources`. The `--collection` option generates a `ResourceCollection` subclass; the `--json-api` option generates a `JsonApiResource`. By default, the generated class extends `JsonResource`.

**Beginner-Friendly Explanation**

`artisan make:resource` is the terminal command you type to create a new resource class file.

#### Purposes

- To generate resource classes with minimal effort.
- To provide consistent structure for resource files.
- To support collection and JSON:API resource generation.
- To enforce Laravel's naming and directory conventions.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:resource <Name> [options]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `make:resource` | The Artisan command. |
| `<Name>` | The resource class name (PascalCase). |
| `--collection` | Generate a collection resource. |
| `--json-api` | Generate a JSON:API resource (Laravel 12.45+). |

**Syntax Rules**

- The resource name is required.
- The `--collection` and `--json-api` flags are optional.
- Generated files are placed in `app/Http/Resources`.

**Constraints and Limitations**

- Routes are not registered automatically.
- Generated stubs must be filled in with `toArray()` logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Resource with Collection**

**Command**

```bash
php artisan make:resource PostResource --collection
```

**Expected Output (Terminal)**

```
INFO  Resource [app/Http/Resources/PostResource.php] created successfully.
```

**Why This Output Occurs**

The `--collection` flag generates a collection resource class.

---

#### Real-World Cases with Explanation

**Case 1: API-First Development**

A team building an API-first application generates resources for each model upfront, establishing the API contract before implementing controllers.

**Explanation**: Resource generation is the first step in building a structured API.

---

### 4. Resource Collections

#### Definitions

**Core Definition**

A Resource Collection is a resource class that transforms a collection of models (e.g., from `Model::all()` or `Model::paginate()`) into a structured JSON array, optionally including pagination metadata and collection-level links.

**Technical Definition**

Collection resources extend `Illuminate\Http\Resources\Json\ResourceCollection`. They are used via `Resource::collection($models)`. When the underlying data is a paginator (`LengthAwarePaginator`, `Paginator`, or `CursorPaginator`), Laravel automatically includes `links` and `meta` keys. The `collection()` static method on `JsonResource` creates an anonymous collection resource for single-model resources.

**Beginner-Friendly Explanation**

A Resource Collection is like a Resource but for multiple records at once. Instead of transforming one user, it transforms a list of users and automatically adds pagination information like "page 1 of 5" and "next page" links.

#### Purposes

- To transform collections of models into JSON arrays.
- To automatically include pagination metadata and links.
- To provide collection-level customisation (e.g., adding summary data).
- To maintain a consistent structure between single resources and collections.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Controller usage
use App\Http\Resources\UserResource;

// Using collection() on a resource
return UserResource::collection(User::paginate(15));

// Using a dedicated collection resource
return new UserCollection(User::paginate(15));
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Resource::collection($models)` | Transforms a collection using a single-model resource. |
| `new UserCollection($models)` | Uses a dedicated collection resource class. |
| `ResourceCollection` | Base class for collection resources. |
| `links` and `meta` | Automatically included for paginated data. |

**Syntax Rules**

- Use `Resource::collection()` for most cases.
- Use a dedicated collection resource when you need collection-level customisation.
- Pagination metadata is added automatically when the collection is a paginator.

**Constraints and Limitations**

- Collection resources cannot be used for single models.
- Pagination metadata is only included for paginator instances.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Collection**

**Controller**

```php
use App\Http\Resources\UserResource;
use App\Models\User;

public function index()
{
    return UserResource::collection(User::all());
}
```

**Expected Output**

```json
{
    "data": [
        {"id": 1, "name": "John", "email": "john@example.com"},
        {"id": 2, "name": "Jane", "email": "jane@example.com"}
    ]
}
```

**Why This Output Occurs**

`UserResource::collection()` transforms each model in the collection using the `UserResource` class.

---

**Example 2: Paginated Collection**

```php
public function index()
{
    return UserResource::collection(User::paginate(15));
}
```

**Expected Output**

```json
{
    "data": [
        {"id": 1, "name": "John", "email": "john@example.com"}
    ],
    "links": {
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=5",
        "prev": null,
        "next": "http://example.com/users?page=2"
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 5,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 15,
        "total": 75
    }
}
```

**Why This Output Occurs**

Laravel automatically adds `links` and `meta` keys when the collection is a paginator.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Product Listing**

An e-commerce API returns a paginated list of products. The collection resource includes pagination links so the frontend can implement "next page" and "previous page" navigation.

**Explanation**: Resource collections with pagination are essential for scalable APIs.

---

### 5. Resource Methods

#### Definitions

**Core Definition**

Resource methods are the methods available on `JsonResource` instances that control transformation, conditional inclusion, relationship handling, wrapping, and pagination behaviour.

**Technical Definition**

Key methods include `toArray()` (required, defines output), `with()` (adds top-level data), `additional()` (adds metadata), `when()` / `whenLoaded()` / `whenNotNull()` / `whenHas()` / `whenAppended()` / `whenCounted()` (conditional attributes), `mergeWhen()` (merge multiple conditional attributes), `response()` / `withResponse()` (customize the HTTP response), and `paginationInformation()` (customize pagination metadata).

**Beginner-Friendly Explanation**

Resource methods are the tools you use inside a resource class to control exactly how your data is transformed. You can conditionally include fields, merge data, and customize the response.

#### Purposes

- To conditionally include or exclude attributes based on context.
- To add metadata to responses (`with()`, `additional()`).
- To customize pagination information.
- To control the HTTP response (status code, headers).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Conditional attributes
'email' => $this->when($request->user()->isAdmin(), $this->email),
'posts' => PostResource::collection($this->whenLoaded('posts')),
'name' => $this->whenNotNull($this->name),
'name' => $this->whenHas('name'),

// Merging conditional attributes
$this->mergeWhen($request->user()->isAdmin(), [
    'secret' => 'secret-value',
    'internal_id' => $this->internal_id,
]),

// Top-level data
public function with(Request $request): array
{
    return ['meta' => ['version' => '1.0']];
}

// Additional data
public function additional(Request $request): array
{
    return ['timestamp' => now()->toIso8601String()];
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `when($condition, $value)` | Include if condition is true. |
| `whenLoaded($relationship)` | Include relationship if eager-loaded. |
| `whenNotNull($value)` | Include if value is not null. |
| `whenHas($attribute)` | Include if attribute exists on model. |
| `mergeWhen($condition, $attributes)` | Merge multiple attributes conditionally. |
| `with($request)` | Add top-level data to the response. |
| `additional($request)` | Add metadata to the response. |
| `paginationInformation()` | Customize pagination links and meta. |

**Syntax Rules**

- `when()` accepts a boolean condition and a value or closure.
- `whenLoaded()` checks if a relationship has been eager-loaded.
- `mergeWhen()` accepts a condition and an array of attributes.
- `with()` and `additional()` return arrays.

**Constraints and Limitations**

- `whenLoaded()` does not trigger lazy loading; the relationship must be eager-loaded.
- `when()` with a closure defers the value computation until the condition is true.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Conditional Attributes**

**Resource**

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->when($request->user()?->isAdmin(), $this->email),
        'secret' => $this->when($request->user()?->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
    ];
}
```

**Expected Output (admin user)**

```json
{
    "data": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com",
        "secret": "secret-value",
        "created_at": "2026-10-11T12:00:00Z"
    }
}
```

**Expected Output (non-admin user)**

```json
{
    "data": {
        "id": 1,
        "name": "John Doe",
        "created_at": "2026-10-11T12:00:00Z"
    }
}
```

**Why This Output Occurs**

The `when()` method includes the `email` and `secret` attributes only if the authenticated user is an admin.

---

**Example 2: Using `with()` and `additional()`**

```php
public function with(Request $request): array
{
    return [
        'meta' => [
            'version' => '1.0',
            'generated_at' => now()->toIso8601String(),
        ],
    ];
}
```

**Expected Output**

```json
{
    "data": { ... },
    "meta": {
        "version": "1.0",
        "generated_at": "2026-10-11T12:00:00Z"
    }
}
```

**Why This Output Occurs**

The `with()` method adds top-level `meta` data to the response.

---

#### Real-World Cases with Explanation

**Case 1: Admin-Only Fields**

An API resource conditionally includes `profit_margin` and `internal_notes` only for admin users, using `when()` with the user's role.

**Explanation**: Conditional attributes enable role-based response customization.

---

### 6. Resource Wrapping

#### Definitions

**Core Definition**

Resource wrapping is the automatic enclosing of resource responses in a `data` key, which is the default behaviour of Laravel API Resources.

**Technical Definition**

By default, `JsonResource` wraps the transformed data in a `data` key. This reserves the top-level response for `links`, `meta`, and error information. Wrapping can be disabled globally via `JsonResource::withoutWrapping()` in `AppServiceProvider::boot()`. Individual resources can override the wrapper by defining `public static $wrap = 'custom_key'` or `public static $wrap = null`.

**Beginner-Friendly Explanation**

Wrapping means your data is always returned inside a `data` container. So instead of `{"name": "John"}`, you get `{"data": {"name": "John"}}`. This is a best practice because it leaves room for other information like pagination links or error messages.

#### Purposes

- To provide a consistent response structure across all API endpoints.
- To reserve top-level keys for metadata, links, and errors.
- To allow clients to distinguish between data and response metadata.
- To comply with JSON:API conventions (which require a `data` key).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Disable wrapping globally (AppServiceProvider)
use Illuminate\Http\Resources\Json\JsonResource;

public function boot(): void
{
    JsonResource::withoutWrapping();
}
```

```php
// Custom wrapper key on a resource
class UserResource extends JsonResource
{
    public static $wrap = 'user';
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Default `data` wrapper | Applied automatically to all resources. |
| `withoutWrapping()` | Disables wrapping globally. |
| `public static $wrap` | Custom wrapper key per resource. |

**Syntax Rules**

- Wrapping is enabled by default.
- Disable globally via `JsonResource::withoutWrapping()`.
- Override per-resource via `$wrap`.
- Collection responses also use the `data` wrapper.

**Constraints and Limitations**

- Disabling wrapping globally may break clients expecting the `data` key.
- Custom wrapper keys must be valid JSON keys.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Default Wrapping**

```php
return new UserResource($user);
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "name": "John Doe"
    }
}
```

**Why This Output Occurs**

The default `data` wrapper is applied automatically.

---

**Example 2: Disabling Wrapping**

```php
// AppServiceProvider.php
use Illuminate\Http\Resources\Json\JsonResource;

public function boot(): void
{
    JsonResource::withoutWrapping();
}
```

**Expected Output**

```json
{
    "id": 1,
    "name": "John Doe"
}
```

**Why This Output Occurs**

Wrapping is disabled globally, so the resource data is returned at the top level.

---

**Example 3: Custom Wrapper**

```php
class UserResource extends JsonResource
{
    public static $wrap = 'user';
}
```

**Expected Output**

```json
{
    "user": {
        "id": 1,
        "name": "John Doe"
    }
}
```

**Why This Output Occurs**

The `$wrap` property overrides the default `data` key with `user`.

---

#### Real-World Cases with Explanation

**Case 1: JSON:API Compliance**

A team building a JSON:API-compliant API uses the default `data` wrapper, which is required by the specification.

**Explanation**: Wrapping aligns Laravel's output with the JSON:API standard.

---

### 7. Resource Pagination

#### Definitions

**Core Definition**

Resource pagination is the automatic inclusion of `links` and `meta` keys in resource responses when the underlying data is a paginator, providing clients with navigation information for paginated collections.

**Technical Definition**

When a resource or resource collection wraps a `LengthAwarePaginator`, `Paginator`, or `CursorPaginator`, Laravel automatically adds `links` (first, last, prev, next) and `meta` (current_page, from, last_page, path, per_page, to, total) keys to the JSON response. The `paginationInformation()` method on a resource can customize these keys.

**Beginner-Friendly Explanation**

Pagination means your API returns data in pages instead of all at once. Resource pagination automatically adds "next page" and "previous page" links so the client knows how to navigate.

#### Purposes

- To provide clients with navigation links for paginated data.
- To include pagination metadata (total items, current page, etc.).
- To support cursor-based pagination for large datasets.
- To allow customisation of pagination output.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Controller
return UserResource::collection(User::paginate(15));
return UserResource::collection(User::simplePaginate(15));
return UserResource::collection(User::cursorPaginate(15));

// Custom pagination information
public function paginationInformation($request, $paginated, $default)
{
    $default['links']['custom'] = 'https://example.com';
    return $default;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `paginate($perPage)` | Length-aware pagination with total count. |
| `simplePaginate($perPage)` | Simple pagination (no total count). |
| `cursorPaginate($perPage)` | Cursor-based pagination. |
| `paginationInformation()` | Customizes links and meta. |

**Syntax Rules**

- Pagination metadata is added automatically for paginator instances.
- `paginate()` returns `LengthAwarePaginator`; `simplePaginate()` returns `Paginator`.
- `paginationInformation()` receives the request, paginated data, and default links/meta.

**Constraints and Limitations**

- Cursor pagination cannot jump to arbitrary pages.
- Simple pagination does not include total item count.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Length-Aware Pagination**

```php
return UserResource::collection(User::paginate(15));
```

**Expected Output**

```json
{
    "data": [ ... ],
    "links": {
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=5",
        "prev": null,
        "next": "http://example.com/users?page=2"
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 5,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 15,
        "total": 75
    }
}
```

**Why This Output Occurs**

Laravel automatically includes pagination links and metadata for `LengthAwarePaginator` instances.

---

**Example 2: Cursor Pagination**

```php
return UserResource::collection(User::cursorPaginate(15));
```

**Expected Output**

```json
{
    "data": [ ... ],
    "links": {
        "prev": null,
        "next": "http://example.com/users?cursor=eyJpZCI6MTV9"
    },
    "meta": {
        "path": "http://example.com/users",
        "per_page": 15,
        "next_cursor": "eyJpZCI6MTV9",
        "prev_cursor": null
    }
}
```

**Why This Output Occurs**

Cursor pagination provides cursor-based navigation instead of page numbers.

---

#### Real-World Cases with Explanation

**Case 1: Infinite Scroll**

A social media app uses cursor pagination to implement infinite scroll. The `next_cursor` in the response tells the app which cursor to send for the next batch of posts.

**Explanation**: Cursor pagination is ideal for large, frequently changing datasets.

---

### 8. Resource Conditional Attributes

#### Definitions

**Core Definition**

Conditional attributes are resource attributes that are included or excluded from the JSON response based on runtime conditions such as user permissions, relationship loading state, or attribute values.

**Technical Definition**

Laravel provides six conditional methods: `when($condition, $value)` (include if condition is true), `whenLoaded($relationship)` (include if eager-loaded), `whenNotNull($value)` (include if not null), `whenHas($attribute)` (include if the model has the attribute), `whenAppended($attribute)` (include if appended), and `whenCounted($relationship)` (include if count loaded). The `mergeWhen($condition, $attributes)` method merges multiple attributes conditionally.

**Beginner-Friendly Explanation**

Conditional attributes let you show or hide fields depending on who is asking. For example, an admin might see a user's email address, while a regular user does not. Or a related model is only included if it was loaded from the database.

#### Purposes

- To customize API responses based on user permissions.
- To prevent N+1 queries by including relationships only when loaded.
- To handle nullable attributes gracefully.
- To support context-dependent responses (e.g., list view vs. detail view).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
return [
    'id' => $this->id,
    'email' => $this->when($request->user()?->isAdmin(), $this->email),
    'posts' => PostResource::collection($this->whenLoaded('posts')),
    'name' => $this->whenNotNull($this->name),
    'nickname' => $this->whenHas('nickname'),
    'comments_count' => $this->whenCounted('comments'),
    $this->mergeWhen($request->user()?->isAdmin(), [
        'internal_id' => $this->internal_id,
        'secret' => 'secret-value',
    ]),
];
```

**Component Breakdown**

| Method | Condition | Use Case |
|--------|-----------|----------|
| `when($condition, $value)` | Boolean condition | Admin-only fields |
| `whenLoaded($rel)` | Relationship eager-loaded | N+1 prevention |
| `whenNotNull($value)` | Value is not null | Optional attributes |
| `whenHas($attr)` | Attribute exists on model | Dynamic attributes |
| `whenAppended($attr)` | Attribute is appended | Accessor values |
| `whenCounted($rel)` | Count is loaded | Relationship counts |
| `mergeWhen($cond, $attrs)` | Boolean condition | Multiple conditional attributes |

**Syntax Rules**

- `when()` accepts a boolean, a value, and an optional closure.
- `whenLoaded()` requires the relationship to be eager-loaded (via `with()`).
- `mergeWhen()` accepts a condition and an array of attributes.
- Conditional methods return a `MissingValue` instance when the condition is false, and Laravel removes it from the output.

**Constraints and Limitations**

- `whenLoaded()` does not trigger lazy loading; the relationship must be loaded in the controller.
- `whenHas()` checks if the attribute exists on the model, not if it is non-null.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Admin-Only Email**

**Resource**

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->when($request->user()?->isAdmin(), $this->email),
    ];
}
```

**Expected Output (admin)**

```json
{"data": {"id": 1, "name": "John", "email": "john@example.com"}}
```

**Expected Output (non-admin)**

```json
{"data": {"id": 1, "name": "John"}}
```

**Why This Output Occurs**

The `when()` method includes the `email` attribute only if the authenticated user is an admin.

---

**Example 2: Relationship Only When Loaded**

**Controller**

```php
return new UserResource(User::with('posts')->findOrFail($id));
```

**Resource**

```php
'posts' => PostResource::collection($this->whenLoaded('posts')),
```

**Expected Output (when loaded)**

```json
{
    "data": {
        "id": 1,
        "name": "John",
        "posts": [
            {"id": 1, "title": "First Post"}
        ]
    }
}
```

**Expected Output (when not loaded)**

```json
{
    "data": {
        "id": 1,
        "name": "John"
    }
}
```

**Why This Output Occurs**

`whenLoaded()` includes the `posts` relationship only if it has been eager-loaded via `with('posts')`, preventing N+1 queries.

---

**Example 3: Merging Conditional Attributes**

```php
return [
    'id' => $this->id,
    'name' => $this->name,
    $this->mergeWhen($request->user()?->isAdmin(), [
        'internal_id' => $this->internal_id,
        'profit_margin' => $this->calculateProfitMargin(),
    ]),
];
```

**Expected Output (admin)**

```json
{
    "data": {
        "id": 1,
        "name": "Product",
        "internal_id": "SKU-001",
        "profit_margin": 0.45
    }
}
```

**Why This Output Occurs**

`mergeWhen()` merges multiple attributes into the response when the condition is true.

---

#### Real-World Cases with Explanation

**Case 1: Role-Based API Responses**

An e-commerce API returns full product details (cost price, profit margin) only to admin users. Regular customers see only the retail price and description.

**Explanation**: Conditional attributes enforce role-based data visibility without separate endpoints.

---

### 9. Resource Relationships

#### Definitions

**Core Definition**

Resource relationships are the inclusion of related models (e.g., a post's author, a user's posts) within an API Resource response, either nested directly or conditionally via `whenLoaded()`.

**Technical Definition**

Related models are included by nesting their resource classes within the parent resource's `toArray()` method. The `whenLoaded()` method conditionally includes relationships only when they have been eager-loaded in the controller. `whenCounted()` includes relationship counts. Nested resources can be collections (`Resource::collection($this->whenLoaded('relationship'))`) or single resources (`new RelatedResource($this->whenLoaded('relationship'))`). API resources are often combined with query builder packages (e.g., Spatie Query Builder) to handle `?include=` parameters.

**Beginner-Friendly Explanation**

Relationships let you include related data in your API response. For example, when you fetch a blog post, you can also include the author's name and the comments on that post—all in one response.

#### Purposes

- To include related model data in API responses without multiple requests.
- To conditionally include relationships only when requested.
- To prevent N+1 queries by using `whenLoaded()`.
- To support `?include=` query parameters for client-controlled relationship inclusion.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Nested single resource
'author' => new UserResource($this->whenLoaded('author')),

// Nested collection resource
'comments' => CommentResource::collection($this->whenLoaded('comments')),

// Relationship count
'comments_count' => $this->whenCounted('comments'),

// Always-included relationship (eager-loaded in controller)
'category' => new CategoryResource($this->category),
```

**Component Breakdown**

| Pattern | Description |
|---------|-------------|
| `new UserResource($this->whenLoaded('author'))` | Nested single resource. |
| `CommentResource::collection($this->whenLoaded('comments'))` | Nested collection. |
| `$this->whenCounted('comments')` | Relationship count. |
| `new CategoryResource($this->category)` | Always included (must be eager-loaded). |

**Syntax Rules**

- Relationships must be eager-loaded in the controller (`with()`, `load()`, or `loadMissing()`).
- Use `whenLoaded()` to conditionally include relationships.
- Use `whenCounted()` for `withCount()` results.
- Nested resources can be nested to any depth.

**Constraints and Limitations**

- Lazy loading inside resources causes N+1 queries.
- Deeply nested resources can cause performance issues.
- Circular relationships can cause infinite recursion.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Nested Single Resource**

**Controller**

```php
return new PostResource(Post::with('author')->findOrFail($id));
```

**Resource**

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        'author' => new UserResource($this->whenLoaded('author')),
    ];
}
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "title": "My First Post",
        "author": {
            "id": 5,
            "name": "John Doe"
        }
    }
}
```

**Why This Output Occurs**

The `author` relationship is eager-loaded via `with('author')` and included using `whenLoaded()`.

---

**Example 2: Nested Collection Resource**

**Controller**

```php
return new UserResource(User::with('posts')->findOrFail($id));
```

**Resource**

```php
'posts' => PostResource::collection($this->whenLoaded('posts')),
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "name": "John",
        "posts": [
            {"id": 1, "title": "Post 1"},
            {"id": 2, "title": "Post 2"}
        ]
    }
}
```

**Why This Output Occurs**

The `posts` relationship is eager-loaded and transformed using `PostResource::collection()`.

---

**Example 3: Relationship Count**

**Controller**

```php
return new PostResource(Post::withCount('comments')->findOrFail($id));
```

**Resource**

```php
'comments_count' => $this->whenCounted('comments'),
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "title": "My Post",
        "comments_count": 5
    }
}
```

**Why This Output Occurs**

`whenCounted()` includes the relationship count if `withCount('comments')` was used.

---

#### Real-World Cases with Explanation

**Case 1: Blog API with Author and Comments**

A blog API returns posts with the author's name and the number of comments. The controller eager-loads both relationships, and the resource conditionally includes them using `whenLoaded()` and `whenCounted()`.

**Explanation**: Relationship handling in resources enables rich, single-request API responses.

---

### 10. Resource Best Practices

#### Definitions

**Core Definition**

Resource best practices are the recommended guidelines for designing, implementing, and maintaining API Resources to maximize security, performance, maintainability, and consistency.

**Technical Definition**

Best practices encompass resource class design, field selection, conditional attribute usage, relationship handling, eager loading, wrapping strategy, versioning, and testing.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to build clean, efficient, and secure API resources.

#### Key Best Practices

1. **Never Return Eloquent Models Directly**: Always use `JsonResource` or `ResourceCollection` to avoid leaking database schema, internal attributes, or relationship cycles.

2. **Eager Load Relationships Before Resources**: Use `with()`, `load()`, or `loadMissing()` in the controller. Never trigger lazy loading inside a resource.

3. **Use `whenLoaded()` for Relationships**: Conditionally include relationships only when eager-loaded to prevent N+1 queries.

4. **Create Separate Resources for Different Contexts**: Use different resources for list views (summary fields) and detail views (full fields).

5. **Use Conditional Attributes for Role-Based Access**: Use `when()` to include sensitive fields only for authorized users.

6. **Keep Resources Slim**: Resources should only transform data, not contain business logic.

7. **Format Dates Consistently**: Use `toIso8601String()` or `toDateTimeString()` for consistent date formatting across resources.

8. **Use `Resource::collection()` for Collections**: Prefer `Resource::collection()` over manual array mapping for consistency and pagination support.

9. **Consider Custom Collection Resources**: Use dedicated collection resource classes when you need collection-level metadata or customization.

10. **Test Resource Output**: Write tests that assert the exact JSON structure of resource responses to prevent regressions.

11. **Document Resource Fields**: Use PHPDoc annotations to document the output structure of each resource.

12. **Version Resources Alongside API Versions**: Create versioned resource classes (e.g., `V1\UserResource`, `V2\UserResource`) when the API version changes.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Well-Structured Resource with Conditional Relationships**

**Resource**

```php
<?php
declare(strict_types=1);

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

/** @mixin \App\Models\User */
final class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->when($request->user()?->isAdmin(), $this->email),
            'avatar_url' => $this->avatar_url,
            'created_at' => $this->created_at?->toIso8601String(),

            // Relationships — only included when eager-loaded
            'posts' => PostResource::collection($this->whenLoaded('posts')),
            'profile' => new ProfileResource($this->whenLoaded('profile')),

            // Relationship count
            'posts_count' => $this->whenCounted('posts'),
        ];
    }
}
```

**Controller**

```php
public function show(User $user): UserResource
{
    return new UserResource(
        $user->loadMissing(['posts', 'profile'])->loadCount('posts')
    );
}
```

**Expected Output**

```json
{
    "data": {
        "id": 1,
        "name": "John Doe",
        "avatar_url": "https://example.com/avatar.jpg",
        "created_at": "2026-10-11T12:00:00Z",
        "posts": [
            {"id": 1, "title": "First Post"}
        ],
        "profile": {
            "bio": "Laravel developer"
        },
        "posts_count": 5
    }
}
```

**Why This Output Occurs**

The resource conditionally includes `email` (admin only), includes `posts` and `profile` only when eager-loaded, and includes the `posts_count` only when counted. All relationships are eager-loaded in the controller to prevent N+1 queries.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise API Platform**

A large organization follows all best practices: separate resources for list and detail views, `whenLoaded()` for all relationships, no business logic in resources, consistent date formatting, and comprehensive resource tests.

**Explanation**: Following best practices ensures the API is secure, performant, and maintainable at scale.

---

## References

- Eloquent: API Resources – Laravel 12.x Documentation — https://laravel.com/docs/12.x/eloquent-resources
- Eloquent: API Resources – Laravel master Documentation — https://laravel.com/framework/docs/master/eloquent-resources
- Eloquent: API Resources – Laravel 10.x Documentation — https://laravel.com/docs/10.x/eloquent-resources
- Dynamic API Response Control in Laravel Resources – Laravel News — https://laravel-news.com/index.php/dynamic-api-response-control-in-laravel-resources
- API Response Transformation with JsonResource – voku/agent-skills (GitHub) — https://github.com/voku/agent-skills/blob/main/skills/laravel-best-practices/rules/controller-api-resources.md
- Laravel whenLoaded – Performance Optimization via Conditional Relationship Loading – Laravel News — https://laravel-news.com/laravel-whenloaded
- Getting Single Record and API Resources – Laravel Daily — https://laraveldaily.com/lesson/api-laravel/getting-single-record-and-api-resources
- Laravel API Resources with Relations: Methods to Avoid N+1 Query – Laravel Daily — https://laraveldaily.com/post/laravel-api-resources-with-relations-methods-to-avoid-n1-query
- JSON:API Resource in Laravel 12.45 – Laravel News — https://laravel-news.com/jsonapi-resource-laravel-12-45
- Eloquent: API Resources – Laravel 12.x (Vectorial1024 mirror) — https://github.com/Vectorial1024/docs/blob/12.x/eloquent-resources.md