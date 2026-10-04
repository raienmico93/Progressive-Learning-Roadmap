# Laravel API Design — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel API Design refers to the architectural patterns, conventions, and framework tools used to shape API endpoints — including pagination, filtering, sorting, versioning, idempotency, error formatting, and rate limiting — so that APIs are performant, predictable, and maintainable.

**Technical Definition:** Laravel API Design is the application of REST principles within the Laravel framework using its pagination classes (`LengthAwarePaginator`, `CursorPaginator`), Eloquent query scopes and query builder packages (`spatie/laravel-query-builder`), full-text search integration (`Laravel Scout`), versioning middleware (URI path, header, query parameter, or `Accept` header strategies), idempotency middleware (cache or Redis-backed request key storage), standardized error formats (`RFC 7807 Problem Details`), and advanced rate limiting via the `RateLimiter` facade with custom `Retry-After` headers.

**Beginner-Friendly Explanation:** When you build an API, it is not enough for it to simply work — it needs to work well at scale. This cheat sheet covers the "design" decisions that make an API production-ready: how to handle millions of records without slowing down (pagination), how to let clients filter and sort data (query scopes), how to search large text fields (Scout), how to change your API without breaking existing clients (versioning), how to prevent accidental duplicate operations (idempotency), how to return errors that clients can parse (error conventions), and how to protect your server from too many requests (rate limiting). Each of these is a practical tool that separates a hobby project from a professional API.

### Key Characteristics

- **Performance-aware pagination:** Offset pagination is simple but degrades on deep pages; cursor pagination provides constant-time performance for large datasets.
- **Dynamic query building:** Filtering, sorting, and searching are driven by request parameters rather than hard-coded controller logic.
- **Lifecycle management:** API versions move through active, deprecated, and sunset states, with standard HTTP headers communicating these states to clients.
- **Safety mechanisms:** Idempotency keys prevent duplicate mutations from retried requests; rate limiting prevents abuse.
- **Standardized error communication:** Error responses follow a consistent structure (e.g., RFC 7807) so clients can handle them programmatically.
- **Framework-native tooling:** Laravel provides built-in support for most of these patterns, with community packages filling gaps where needed.

### Prerequisites

- PHP 8.1+ (Laravel 10+) or PHP 8.2+ (Laravel 11+).
- Composer dependency manager.
- A Laravel application with Eloquent models, controllers, and `routes/api.php` configured.
- Basic understanding of HTTP, REST, and JSON.
- (For Scout) A supported search driver (database, Algolia, Meilisearch, or Typesense).
- (For Redis-backed idempotency and rate limiting) A Redis server.

### Related Programming Areas

- **Eloquent ORM** — Query scopes, relationships, and pagination are built on Eloquent.
- **HTTP Protocol** — Status codes, headers, and methods are central to API design.
- **REST Architecture** — The design principles underlying resource-oriented APIs.
- **Caching** — Used for idempotency keys, rate limiting counters, and response caching.
- **Full-Text Search** — Scout integrates with external search engines.
- **Middleware** — Versioning, idempotency, and rate limiting are implemented as middleware.

### Core Concepts / Features

1. **Pagination:** Length-aware offset pagination vs. high-performance cursor pagination.
2. **Filtering & Query Scopes:** Local scopes and dedicated query builder packages.
3. **Sorting & Searching:** Dynamic multi-column sorting and full-text search via Laravel Scout.
4. **Versioning Strategies:** Maintenance lifecycles, deprecation headers, and backward compatibility.
5. **Idempotency:** Unique request keys and caching to prevent duplicate mutations.
6. **Error Conventions:** Standardized error formats (RFC 7807 problem details, JSON:API error objects).
7. **Advanced Rate Limiting:** Handling `429 Too Many Requests` with precise `Retry-After` headers.

---

## 1. Pagination

### Definitions

**Core Definition:** Pagination is the technique of dividing a large dataset into smaller, manageable chunks (pages) that can be requested individually, improving response times and reducing payload sizes.

**Technical Definition:** Laravel provides two primary pagination strategies: **Length-aware offset pagination** (`paginate()`), which uses `LIMIT` and `OFFSET` clauses and returns a `LengthAwarePaginator` with total counts and page numbers, and **Cursor pagination** (`cursorPaginate()`), which uses a `WHERE` clause with a cursor value (typically the last seen ID) and returns a `CursorPaginator` with `next_cursor` and `prev_cursor` links. Offset pagination executes `SELECT * FROM table LIMIT 15 OFFSET 149985` for deep pages, forcing the database to scan and discard rows. Cursor pagination executes `SELECT * FROM table WHERE id > 149985 ORDER BY id LIMIT 15`, using an index seek for constant-time performance.

**Beginner-Friendly Explanation:** Imagine a library with a million books. If you ask for "books 999,991 through 1,000,005," the librarian has to count through 999,990 books to find your starting point — that takes forever. That is offset pagination. Cursor pagination is like saying "give me the next 15 books after this specific one I'm holding" — the librarian can go straight to the shelf and grab them. For small datasets, both work fine. For large datasets, cursor pagination is dramatically faster.

### Purposes

- To improve API response times by limiting the amount of data returned per request.
- To reduce memory consumption on both the server and the client by avoiding loading entire datasets.
- To provide clients with navigation links (next, previous, first, last) for traversing large result sets.
- To enable infinite scroll and "load more" user interfaces.
- To prevent timeouts and database overload when querying tables with millions of rows.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Offset pagination (LengthAware) — includes total count and page numbers
$users = User::paginate(15);
// Generates: SELECT * FROM users LIMIT 15 OFFSET 0
// Response includes: data, links (first, last, prev, next), meta (current_page, total, per_page, last_page)

// Simple pagination — no total count, faster for large datasets
$users = User::simplePaginate(15);
// Response includes: data, links (prev, next) — no meta totals

// Cursor pagination — constant-time performance for large datasets
$users = User::orderBy('id')->cursorPaginate(15);
// Generates: SELECT * FROM users WHERE id > ? ORDER BY id LIMIT 15
// Response includes: data, links (prev, next) with cursor strings, meta (next_cursor, prev_cursor)

// Cursor pagination with multiple order columns
$users = User::orderBy('created_at')->orderBy('id')->cursorPaginate(15);

// Using pagination with API Resources
return UserResource::collection(User::paginate(15));
return UserResource::collection(User::orderBy('id')->cursorPaginate(15));
```

**Component Breakdown:**

- `paginate($perPage)` — Returns a `LengthAwarePaginator` with total count, page numbers, and full navigation links.
- `simplePaginate($perPage)` — Returns a `Paginator` with only next/previous links, no total count. Faster than `paginate()` because it does not execute a `COUNT(*)` query.
- `cursorPaginate($perPage)` — Returns a `CursorPaginator` with cursor-based navigation. Requires an `orderBy` clause on a unique, indexed column.
- `UserResource::collection($paginator)` — Wraps the paginator in a resource collection, automatically including `links` and `meta` in the JSON response.

**Syntax Rules:**

- Cursor pagination **requires** an `orderBy` clause on a unique column (typically `id`). Without it, Laravel throws an exception.
- Offset pagination generates `LIMIT` and `OFFSET` clauses; cursor pagination generates a `WHERE` clause with the cursor value.
- Paginated resource collections automatically include `links` and `meta` keys in the JSON response.
- `simplePaginate()` does not include a `last_page` or `total` in the response, reducing query overhead.

**Constraints and Limitations:**

- **Offset pagination degrades linearly with page depth.** On a table with millions of rows, navigating to page 10,000 forces the database to scan and discard 150,000 rows.
- **Cursor pagination cannot jump to arbitrary pages.** Clients can only navigate sequentially (next/previous). This makes it unsuitable for UIs that require page number navigation.
- **Cursor pagination requires a unique, indexed order column.** Without an index, the `WHERE` clause degrades to a full table scan.
- **Cursor pagination may skip or duplicate records in datasets with frequent writes.** Because the cursor is based on a value snapshot, insertions or deletions between requests can shift the result set.
- **`simplePaginate()` does not provide total counts**, which may be required for certain UI elements (e.g., "Page 1 of 500").

### Annotated Code Examples

**Example 1: Offset vs. Cursor Pagination in a Controller**

```php
<?php
// File: app/Http/Controllers/Api/UserController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    // Offset pagination — suitable for small datasets (< 10,000 rows)
    public function indexOffset(Request $request)
    {
        $users = User::query()
            ->when($request->has('role'), fn($q) => $q->where('role', $request->role))
            ->latest()
            ->paginate(15); // Includes total count and page numbers

        return UserResource::collection($users);
    }

    // Cursor pagination — suitable for large datasets and infinite scroll
    public function indexCursor(Request $request)
    {
        $users = User::query()
            ->when($request->has('role'), fn($q) => $q->where('role', $request->role))
            ->orderBy('id') // REQUIRED: unique, indexed column
            ->cursorPaginate(15); // Constant-time performance

        return UserResource::collection($users);
    }
}
```

**Step-by-Step Setup:**

1. Define routes: `Route::get('/users/offset', [UserController::class, 'indexOffset']);` and `Route::get('/users/cursor', [UserController::class, 'indexCursor']);`.
2. Ensure the `users` table has an indexed `id` column (default).
3. Seed the table with a large dataset (e.g., 100,000 rows) for performance comparison.

**Expected Output (Offset Pagination):**

```json
{
    "data": [ { "id": 1, "name": "Alice" }, ... ],
    "links": {
        "first": "https://example.com/api/users/offset?page=1",
        "last": "https://example.com/api/users/offset?page=6667",
        "prev": null,
        "next": "https://example.com/api/users/offset?page=2"
    },
    "meta": {
        "current_page": 1,
        "per_page": 15,
        "total": 100000,
        "last_page": 6667
    }
}
```

**Expected Output (Cursor Pagination):**

```json
{
    "data": [ { "id": 1, "name": "Alice" }, ... ],
    "links": {
        "prev": null,
        "next": "https://example.com/api/users/cursor?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0"
    },
    "meta": {
        "next_cursor": "eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0",
        "prev_cursor": null
    }
}
```

**Why This Output Occurs:** The offset endpoint returns page numbers and total counts because `paginate()` executes a `COUNT(*)` query alongside the data query. The cursor endpoint returns opaque cursor strings instead of page numbers; `cursorPaginate()` does not execute a `COUNT(*)` query, making it faster on large tables. The cursor is a base64-encoded JSON object containing the last row's ID and ordering direction.

---

**Example 2: Custom Pagination Meta with a Resource Collection**

```php
<?php
// File: app/Http/Resources/UserCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    public $collects = UserResource::class;

    public function toArray(Request $request): array
    {
        return ['data' => $this->collection];
    }

    public function with(Request $request): array
    {
        return [
            'meta' => [
                'total_users' => $this->collection->total(), // Only for LengthAwarePaginator
                'api_version' => '1.0',
                'pagination_type' => $this->collection instanceof \Illuminate\Pagination\CursorPaginator
                    ? 'cursor'
                    : 'offset',
            ],
        ];
    }
}
```

**Expected Output:**

```json
{
    "data": [ ... ],
    "links": { ... },
    "meta": {
        "current_page": 1,
        "per_page": 15,
        "total": 100000,
        "total_users": 100000,
        "api_version": "1.0",
        "pagination_type": "offset"
    }
}
```

**Why This Output Occurs:** The `with()` method adds custom metadata to the response. The `pagination_type` key dynamically reports which pagination strategy was used, which is useful for clients that support both. Note that `total_users` is only available when using `LengthAwarePaginator`; cursor paginators do not have a `total()` method.

### Real-World Cases

- **Social media feeds:** Infinite scroll uses cursor pagination because users never jump to arbitrary pages, and new content is constantly added.
- **Admin data tables:** Offset pagination with page numbers is preferred because administrators need to navigate to specific pages and see total record counts.
- **Public API endpoints:** Cursor pagination is recommended for endpoints that may return millions of records (e.g., `/api/logs`, `/api/events`).
- **Search results:** Offset pagination is often used because search result sets are typically bounded and users want to see "Page 1 of 50."

---

## 2. Filtering & Query Scopes

### Definitions

**Core Definition:** Filtering is the process of narrowing a dataset based on client-specified criteria (e.g., `?status=active&role=admin`), implemented in Laravel through Eloquent local scopes and dedicated query builder packages.

**Technical Definition:** Laravel's **local scopes** are methods on an Eloquent model that encapsulate reusable query constraints, defined with the `#[Scope]` attribute and prefixed with `scope` (or using the attribute in Laravel 12+). Local scopes are called statically on the model (e.g., `User::popular()->active()->get()`). For API-driven filtering, packages such as `spatie/laravel-query-builder` provide a declarative way to build queries from request parameters, supporting exact filters, partial filters, scope filters, custom filters, and relationship filters.

**Beginner-Friendly Explanation:** Filtering is like using the search filters on an e-commerce site: you select "brand: Nike," "price: under $100," and "size: 10," and the site shows only products that match. In your API, clients send these filters as query parameters (`?brand=nike&max_price=100&size=10`), and your Laravel code translates them into database queries. Local scopes are reusable filter snippets defined on your model; query builder packages handle the translation from URL parameters to database queries automatically.

### Purposes

- To allow clients to retrieve only the data they need, reducing payload size and processing time.
- To encapsulate common query logic in reusable methods (local scopes) rather than duplicating it across controllers.
- To provide a consistent, documented filter interface for API consumers.
- To enable complex filtering (relationships, date ranges, partial matches) without exposing raw database queries to clients.
- To improve API performance by allowing the database to filter data instead of the application.

### Syntax Rules and Structure

#### Complete General Syntax (Local Scopes)

```php
<?php
// File: app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    #[Scope]
    protected function popular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }

    #[Scope]
    protected function active(Builder $query): void
    {
        $query->where('active', 1);
    }

    #[Scope]
    protected function ofType(Builder $query, string $type): void
    {
        $query->where('type', $type);
    }
}

// Usage
$users = User::popular()->active()->orderBy('created_at')->get();
$admins = User::ofType('admin')->get();
```

#### Complete General Syntax (Spatie Query Builder)

```php
// Installation: composer require spatie/laravel-query-builder

// Basic usage in a controller
use Spatie\QueryBuilder\QueryBuilder;

$users = QueryBuilder::for(User::class)
    ->allowedFilters(['name', 'email', 'role'])
    ->allowedSorts(['name', 'created_at'])
    ->allowedIncludes(['posts', 'comments'])
    ->paginate(15);

// API request:
// GET /users?filter[name]=John&filter[role]=admin&sort=-created_at&include=posts

// Advanced filtering with exact and partial matches
QueryBuilder::for(User::class)
    ->allowedFilters([
        'name', // Partial match by default
        AllowedFilter::exact('email'), // Exact match
        AllowedFilter::scope('popular'), // Uses the 'popular' local scope
        AllowedFilter::scope('created_after'), // Custom scope with parameter
    ])
    ->allowedSorts(['name', 'created_at', 'votes'])
    ->paginate(15);
```

**Component Breakdown:**

- `#[Scope]` attribute — Laravel 12+ attribute that marks a method as a local scope. In earlier versions, the method name must be prefixed with `scope` (e.g., `scopePopular`).
- `Builder $query` — The query builder instance passed to the scope. The scope mutates it by adding `where` clauses.
- `QueryBuilder::for(User::class)` — Creates a query builder instance for the `User` model.
- `allowedFilters(['name', 'email'])` — Declares which fields clients are allowed to filter on. Only whitelisted filters are applied, preventing SQL injection and unintended queries.
- `AllowedFilter::exact('email')` — Creates an exact-match filter (as opposed to the default partial match).
- `AllowedFilter::scope('popular')` — Uses a local scope as a filter. The scope is called when the client sends `?filter[popular]=1`.
- `allowedSorts(['name', 'created_at'])` — Declares which columns clients can sort by.
- `allowedIncludes(['posts'])` — Declares which relationships clients can eager-load.

**Syntax Rules:**

- Local scopes **must** accept a `Builder` instance as their first parameter and return either the builder or `void`.
- In Laravel 12+, scopes are marked with the `#[Scope]` attribute and should be `protected`. In earlier versions, they are public methods prefixed with `scope`.
- The Spatie Query Builder **must** be given an explicit whitelist of filters, sorts, and includes. Requests attempting to use unlisted parameters are ignored.
- Filter parameters use array notation: `?filter[name]=John&filter[role]=admin`.
- Sort parameters use a minus sign for descending: `?sort=-created_at` (descending) or `?sort=name` (ascending).

**Constraints and Limitations:**

- **Local scopes are not automatically applied to API requests.** They must be called explicitly in controllers or used via `AllowedFilter::scope()`.
- **Spatie Query Builder's default filter is a partial match (LIKE `%value%`).** For exact matches, `AllowedFilter::exact()` must be used. This is a common source of confusion.
- **Filtering on relationship columns requires explicit configuration.** Use `AllowedFilter::scope()` with a custom scope that performs a `whereHas()` query.
- **Over-filtering can lead to slow queries.** Without proper database indexes, complex filters can degrade performance.

### Annotated Code Examples

**Example 1: Local Scopes for Reusable Filtering**

```php
<?php
// File: app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    #[Scope]
    protected function published(Builder $query): void
    {
        $query->where('status', 'published');
    }

    #[Scope]
    protected function byAuthor(Builder $query, int $authorId): void
    {
        $query->where('user_id', $authorId);
    }

    #[Scope]
    protected function createdAfter(Builder $query, string $date): void
    {
        $query->where('created_at', '>=', $date);
    }
}
```

```php
// Controller
public function index(Request $request)
{
    $posts = Post::query()
        ->published()
        ->when($request->has('author'), fn($q) => $q->byAuthor($request->author))
        ->when($request->has('after'), fn($q) => $q->createdAfter($request->after))
        ->latest()
        ->paginate(15);

    return PostResource::collection($posts);
}
```

**Step-by-Step Setup:**

1. Define the scopes on the `Post` model as shown.
2. Call the scopes in the controller using `when()` to conditionally apply them.
3. Test with `GET /api/posts?author=5&after=2025-01-01`.

**Expected Output:**

```json
{
    "data": [
        { "id": 1, "title": "Published Post", "author_id": 5, "status": "published" }
    ],
    "links": { ... },
    "meta": { ... }
}
```

**Why This Output Occurs:** The `published()` scope adds `WHERE status = 'published'`. The `byAuthor(5)` scope adds `WHERE user_id = 5`. The `createdAfter('2025-01-01')` scope adds `WHERE created_at >= '2025-01-01'`. These constraints are combined into a single query with AND operators. The `when()` method ensures that each scope is only applied if the corresponding request parameter is present.

---

**Example 2: Spatie Query Builder for Declarative API Filtering**

```php
<?php
// File: app/Http/Controllers/Api/UserController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\UserResource;
use App\Models\User;
use Spatie\QueryBuilder\AllowedFilter;
use Spatie\QueryBuilder\QueryBuilder;

class UserController extends Controller
{
    public function index()
    {
        $users = QueryBuilder::for(User::class)
            ->allowedFilters([
                'name',                    // Partial match
                AllowedFilter::exact('email'), // Exact match
                AllowedFilter::scope('popular'), // Uses local scope
                AllowedFilter::scope('created_after'), // Custom scope with parameter
                AllowedFilter::exact('role'),
            ])
            ->allowedSorts(['name', 'email', 'created_at', 'votes'])
            ->allowedIncludes(['posts', 'comments'])
            ->defaultSort('-created_at') // Default sort if none provided
            ->paginate(20);

        return UserResource::collection($users);
    }
}
```

**Expected API Requests and Outputs:**

```bash
# Filter by name (partial match)
GET /api/users?filter[name]=john

# Filter by exact email
GET /api/users?filter[email]=john@example.com

# Filter by role (exact match)
GET /api/users?filter[role]=admin

# Sort by name ascending, then created_at descending
GET /api/users?sort=name,-created_at

# Include relationships
GET /api/users?include=posts,comments

# Combined filters
GET /api/users?filter[role]=admin&filter[popular]=1&sort=-created_at&include=posts&page=2
```

**Expected Output:**

```json
{
    "data": [
        {
            "id": 1,
            "name": "John Doe",
            "email": "john@example.com",
            "posts": { "data": [ { "id": 10, "title": "First Post" } ] }
        }
    ],
    "links": { ... },
    "meta": { ... }
}
```

**Why This Output Occurs:** The `allowedFilters()` method whitelists the filters that clients can use. The `QueryBuilder` reads the `filter` query parameters and applies the corresponding constraints. `AllowedFilter::exact('email')` generates `WHERE email = 'john@example.com'`, while the default `'name'` filter generates `WHERE name LIKE '%john%'`. The `allowedSorts()` method enables sorting, and `allowedIncludes()` enables relationship eager-loading. All of this is done without exposing raw SQL to the client.

### Real-World Cases

- **E-commerce product filtering:** Clients filter products by category, price range, brand, rating, and availability using query parameters.
- **User management dashboards:** Administrators filter users by role, status, registration date, and activity level.
- **Log and event APIs:** Clients filter logs by severity, source, timestamp range, and correlation ID.
- **Content management systems:** Editors filter posts by status, author, category, and publication date.

---

## 3. Sorting & Searching

### Definitions

**Core Definition:** Sorting is the arrangement of query results in a specified order (ascending or descending) based on one or more columns; searching is the retrieval of records matching a text query, typically across multiple columns.

**Technical Definition:** In Laravel, sorting is implemented via `orderBy()` on the query builder, and dynamic sorting is handled by packages like `spatie/laravel-query-builder` (via `allowedSorts()`) or custom controller logic. Searching is implemented through **Laravel Scout**, a driver-based full-text search package that syncs Eloquent models with search indexes (database, Algolia, Meilisearch, Typesense) and provides a `search()` method on models using the `Searchable` trait. Scout's `search()` method returns a `Builder` instance that can be further constrained with `where()` clauses and paginated.

**Beginner-Friendly Explanation:** Sorting is like arranging a deck of cards — you can sort by suit, by number, or by both. In an API, clients say `?sort=name` or `?sort=-created_at` to control the order of results. Searching is like using the search bar on a website — it looks through all the text in your records and returns the ones that match. Laravel Scout makes this powerful by using specialized search engines that can handle typos, synonyms, and ranking, far beyond what a simple SQL `LIKE` query can do.

### Purposes

- To allow clients to control the order of results (e.g., newest first, most popular first, alphabetical).
- To enable multi-column sorting for complex ordering requirements (e.g., sort by category, then by price).
- To provide fast, relevant full-text search across large text fields without scanning every row.
- To support typo tolerance, faceted filtering, and ranking in search results.
- To decouple search indexing from the primary database, improving query performance.

### Syntax Rules and Structure

#### Complete General Syntax (Sorting with Spatie Query Builder)

```php
// Controller
use Spatie\QueryBuilder\QueryBuilder;

$posts = QueryBuilder::for(Post::class)
    ->allowedSorts(['title', 'created_at', 'votes'])
    ->defaultSort('-created_at')
    ->paginate(15);

// API requests:
// GET /posts?sort=title          → ORDER BY title ASC
// GET /posts?sort=-created_at     → ORDER BY created_at DESC
// GET /posts?sort=title,-created_at → ORDER BY title ASC, created_at DESC
```

#### Complete General Syntax (Searching with Laravel Scout)

```php
// Installation: composer require laravel/scout
// php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

// Add the Searchable trait to the model
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    // Optional: Customise which fields are indexed
    public function toSearchableArray(): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
        ];
    }
}

// Searching
$posts = Post::search('laravel')->get();
$posts = Post::search('laravel')->where('status', 'published')->paginate(15);

// Import existing records into the search index
// php artisan scout:import "App\Models\Post"
```

**Component Breakdown:**

- `allowedSorts(['title', 'created_at'])` — Declares which columns clients can sort by. Requests attempting to sort by unlisted columns are ignored.
- `defaultSort('-created_at')` — Sets a default sort order when the client does not specify one.
- `use Searchable` — Trait that adds Scout's search capabilities to an Eloquent model and registers observers to keep the search index in sync.
- `toSearchableArray()` — Optional method that defines which model attributes are indexed for searching. If omitted, the model's `toArray()` result is indexed.
- `Post::search('query')` — Returns a Scout `Builder` instance that can be chained with `where()`, `orderBy()`, and `paginate()`.
- `php artisan scout:import` — Imports all existing records into the search index.

**Syntax Rules:**

- Scout **must** be configured with a driver in `config/scout.php` (`database`, `algolia`, `meilisearch`, `typesense`, or `collection`).
- The `Searchable` trait **must** be added to any model that should be searchable.
- Scout's `search()` method returns a `Builder` instance, not a standard Eloquent builder. Some Eloquent methods (e.g., `with()`) are not available; use `where()` and `orderBy()` instead.
- Scout automatically syncs index updates when models are created, updated, or deleted, using model observers.
- For production use with external engines, a queue driver should be configured to handle indexing asynchronously.

**Constraints and Limitations:**

- **Scout's `database` driver is limited.** It uses MySQL/PostgreSQL full-text indexes and `LIKE` clauses, which are not as powerful as dedicated search engines (no typo tolerance, limited ranking).
- **Scout does not support complex relationship filtering by default.** Only simple `AND` filters are supported on indexed fields.
- **External search engines (Algolia, Meilisearch) incur additional infrastructure costs and complexity.** The `database` driver is sufficient for small to medium datasets.
- **Scout's `search()` method cannot be combined with arbitrary Eloquent scopes.** Only constraints on indexed fields are supported.
- **Sorting with Scout is limited to the search engine's supported sort fields**, which may differ from the database columns.

### Annotated Code Examples

**Example 1: Dynamic Multi-Column Sorting with Spatie Query Builder**

```php
<?php
// File: app/Http/Controllers/Api/ProductController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\ProductResource;
use App\Models\Product;
use Spatie\QueryBuilder\QueryBuilder;

class ProductController extends Controller
{
    public function index()
    {
        $products = QueryBuilder::for(Product::class)
            ->allowedFilters(['name', 'category_id', 'price'])
            ->allowedSorts(['name', 'price', 'created_at', 'popularity'])
            ->defaultSort('-created_at')
            ->paginate(20);

        return ProductResource::collection($products);
    }
}
```

**Expected API Requests and Outputs:**

```bash
# Sort by name ascending
GET /api/products?sort=name
# SQL: ORDER BY name ASC

# Sort by price descending
GET /api/products?sort=-price
# SQL: ORDER BY price DESC

# Multi-column sort: category ascending, then price descending
GET /api/products?sort=category_id,-price
# SQL: ORDER BY category_id ASC, price DESC

# Default sort (if none provided)
GET /api/products
# SQL: ORDER BY created_at DESC
```

**Expected Output:**

```json
{
    "data": [
        { "id": 1, "name": "Widget", "price": 9.99, "category_id": 1 },
        { "id": 2, "name": "Gadget", "price": 19.99, "category_id": 1 },
        { "id": 3, "name": "Gizmo", "price": 29.99, "category_id": 2 }
    ],
    "links": { ... },
    "meta": { ... }
}
```

**Why This Output Occurs:** The `allowedSorts()` whitelist restricts which columns can be used for sorting. The `-` prefix indicates descending order. Multiple sort parameters are comma-separated, and the Query Builder applies them in order. The `defaultSort()` method ensures a consistent order when no sort is specified.

---

**Example 2: Full-Text Search with Laravel Scout and Meilisearch**

```php
<?php
// File: app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    public function toSearchableArray(): array
    {
        return [
            'id'      => $this->id,
            'title'   => $this->title,
            'body'    => $this->body,
            'author'  => $this->user->name,
            'tags'    => $this->tags->pluck('name')->join(', '),
        ];
    }
}
```

```php
// Controller
public function search(Request $request)
{
    $request->validate(['q' => 'required|string|min:2']);

    $posts = Post::search($request->q)
        ->where('status', 'published') // Scout supports simple where clauses
        ->paginate(15);

    return PostResource::collection($posts);
}
```

```bash
# Import existing records
php artisan scout:import "App\Models\Post"

# API request
GET /api/posts/search?q=laravel
```

**Expected Output:**

```json
{
    "data": [
        {
            "id": 1,
            "title": "Getting Started with Laravel",
            "body": "Laravel is a PHP framework...",
            "author": "Alice Johnson"
        }
    ],
    "links": { ... },
    "meta": { ... }
}
```

**Why This Output Occurs:** The `Searchable` trait registers model observers that keep the search index in sync. The `toSearchableArray()` method defines which fields are indexed — here, the post's title, body, author name, and tags. When `Post::search('laravel')` is called, Scout queries the configured search engine (Meilisearch in this example) and returns matching post IDs. Scout then hydrates the Eloquent models from the database and applies any additional `where()` constraints.

### Real-World Cases

- **E-commerce product search:** Customers search for products by name, description, and tags; results are ranked by relevance and can be sorted by price or popularity.
- **Documentation sites:** Users search across all documentation pages, with results ranked by relevance and filtered by version.
- **Job boards:** Candidates search for jobs by title, company, and location; results are sorted by posting date or relevance.
- **CRM systems:** Sales representatives search across contacts, companies, and deals, with results filtered by ownership and stage.

---

## 4. Versioning Strategies

### Definitions

**Core Definition:** API versioning is the practice of maintaining multiple versions of an API simultaneously, allowing clients to continue using older versions while newer versions are introduced with breaking changes.

**Technical Definition:** Laravel API versioning can be implemented through four primary strategies: **URI path versioning** (e.g., `/api/v1/`, `/api/v2/`), **header versioning** (e.g., `X-API-Version: 2`), **query parameter versioning** (e.g., `?api_version=2`), and **Accept header versioning** (e.g., `Accept: application/vnd.api.v2+json`). Version lifecycle management involves marking versions as active, deprecated, or sunset, and communicating these states to clients via standard HTTP headers: `Deprecation` (RFC 9745 / draft-dalal-deprecation-header) and `Sunset` (RFC 8594). Laravel packages such as `grazulex/laravel-apiroute` provide automatic lifecycle management with these headers.

**Beginner-Friendly Explanation:** Imagine you have a mobile app that talks to your API. You want to change how the API works, but you cannot force all users to update their app immediately. Versioning lets you keep the old API working for old app versions while new app versions use the new API. You might use `/api/v1/` for the old one and `/api/v2/` for the new one. When you are ready to shut down v1, you tell clients in advance by sending a "Sunset" header that says "this version will stop working on this date."

### Purposes

- To introduce breaking changes to an API without disrupting existing clients.
- To allow clients to migrate to new versions at their own pace.
- To provide a clear lifecycle for API versions (active → deprecated → sunset → removed).
- To communicate deprecation and sunset dates to clients via standard HTTP headers.
- To maintain backward compatibility for critical integrations while evolving the API.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel API Route Package)

```php
// Installation: composer require grazulex/laravel-apiroute

// In routes/api.php
use Grazulex\ApiRoute\Facades\ApiRoute;

// Define version 2 as the current stable version
ApiRoute::version('v2', function () {
    Route::apiResource('orders', App\Http\Controllers\Api\V2\OrderController::class);
})->current();

// Define version 1 as deprecated with a sunset date
ApiRoute::version('v1', function () {
    Route::apiResource('orders', App\Http\Controllers\Api\V1\OrderController::class);
})->deprecated('2025-08-01')->sunset('2025-12-31');
```

**Expected Response Headers for Deprecated Version:**

```
HTTP/1.1 200 OK
X-API-Version: v1
X-API-Version-Status: deprecated
Deprecation: Fri, 01 Aug 2025 00:00:00 GMT
Sunset: Wed, 31 Dec 2025 00:00:00 GMT
```

**Component Breakdown:**

- `ApiRoute::version('v2', ...)` — Defines a versioned route group. The version identifier (`v2`) is used in the URI path, header, or query parameter, depending on configuration.
- `->current()` — Marks the version as the current stable version.
- `->deprecated('2025-08-01')` — Marks the version as deprecated and sets the deprecation date. The `Deprecation` header is added to responses.
- `->sunset('2025-12-31')` — Sets the planned end-of-life date. The `Sunset` header is added to responses.
- `X-API-Version` — Response header indicating which version handled the request.
- `X-API-Version-Status` — Response header indicating the version's lifecycle state (`active`, `deprecated`, `sunset`).

**Syntax Rules:**

- Versioning packages typically support multiple resolution strategies: URI path (default), header (`X-API-Version`), query parameter (`?api_version=2`), and Accept header (`Accept: application/vnd.api.v2+json`).
- Deprecation and Sunset headers follow RFC 8594 and the IETF Deprecation header draft.
- The `Sunset` header **must** contain an HTTP-date (RFC 7231 format), not a relative time.
- Deprecated versions continue to function until the sunset date, after which they may return `410 Gone` or `404 Not Found`.

**Constraints and Limitations:**

- **URI path versioning pollutes the URL space** but is the most explicit and easiest to debug. Header and query parameter versioning keep URLs clean but are less discoverable.
- **Version lifecycle management requires discipline.** Deprecation dates must be communicated to clients well in advance; short deprecation windows are hostile to API consumers.
- **Multiple versions increase maintenance burden.** Each version requires its own controllers, resources, and tests. Code sharing via base classes or traits can reduce duplication.
- **Sunset headers are not universally respected by clients.** Some HTTP clients ignore them entirely; additional communication (documentation, emails) is required.

### Annotated Code Examples

**Example 1: URI Path Versioning with Deprecation Headers**

```php
<?php
// File: routes/api.php

use Grazulex\ApiRoute\Facades\ApiRoute;
use Illuminate\Support\Facades\Route;

// Version 2 — Current stable version
ApiRoute::version('v2', function () {
    Route::apiResource('orders', App\Http\Controllers\Api\V2\OrderController::class);
    Route::apiResource('products', App\Http\Controllers\Api\V2\ProductController::class);
})->current();

// Version 1 — Deprecated, sunset planned for end of year
ApiRoute::version('v1', function () {
    Route::apiResource('orders', App\Http\Controllers\Api\V1\OrderController::class);
    Route::apiResource('products', App\Http\Controllers\Api\V1\ProductController::class);
})->deprecated('2025-06-01')->sunset('2025-12-31');
```

```php
// config/apiroute.php — Configure versioning strategy
return [
    'strategy' => 'uri', // 'uri', 'header', 'query', or 'accept'
    'response_headers' => true, // Include X-API-Version and deprecation headers
];
```

**Step-by-Step Setup:**

1. Install the package: `composer require grazulex/laravel-apiroute`.
2. Publish the config: `php artisan vendor:publish --provider="Grazulex\ApiRoute\ApiRouteServiceProvider"`.
3. Define versioned routes as shown.
4. Test with `GET /api/v1/orders` — the response includes `Deprecation` and `Sunset` headers.
5. Test with `GET /api/v2/orders` — the response includes `X-API-Version: v2` but no deprecation headers.

**Expected Output (v1 Request):**

```
HTTP/1.1 200 OK
X-API-Version: v1
X-API-Version-Status: deprecated
Deprecation: Sun, 01 Jun 2025 00:00:00 GMT
Sunset: Wed, 31 Dec 2025 00:00:00 GMT
Content-Type: application/json

{ "data": [ { "id": 1, "status": "pending" } ] }
```

**Expected Output (v2 Request):**

```
HTTP/1.1 200 OK
X-API-Version: v2
X-API-Version-Status: active
Content-Type: application/json

{ "data": [ { "id": 1, "status": "pending", "tracking_url": "https://..." } ] }
```

**Why This Output Occurs:** The `ApiRoute` facade registers versioned route groups with the specified lifecycle states. When a request is made to `/api/v1/`, the middleware resolves the version, checks its status (`deprecated`), and adds the `Deprecation` and `Sunset` headers to the response. The response body still contains the v1 data structure, but clients are informed that the version is being retired. The v2 response does not include deprecation headers because it is the current active version.

---

**Example 2: Header-Based Versioning with Custom Middleware**

```php
<?php
// File: app/Http/Middleware/ApiVersionMiddleware.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ApiVersionMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $version = $request->header('X-API-Version', '1');

        if (!in_array($version, ['1', '2'])) {
            return response()->json([
                'error' => 'Unsupported API version.',
                'supported' => ['1', '2'],
            ], 400);
        }

        $request->attributes->set('api_version', $version);

        $response = $next($request);
        $response->headers->set('X-API-Version', $version);

        if ($version === '1') {
            $response->headers->set('Deprecation', 'true');
            $response->headers->set('Sunset', 'Wed, 31 Dec 2025 00:00:00 GMT');
        }

        return $response;
    }
}
```

```php
// Controller
public function index(Request $request)
{
    $version = $request->attributes->get('api_version');

    if ($version === '1') {
        return response()->json(Order::all()); // Flat structure
    }

    return response()->json([
        'data' => Order::all(),
        'meta' => ['version' => '2'],
    ]);
}
```

**Expected Output (Header `X-API-Version: 2`):**

```json
{
    "data": [ { "id": 1, "status": "pending" } ],
    "meta": { "version": "2" }
}
```

**Expected Output (Header `X-API-Version: 1`):**

```json
[ { "id": 1, "status": "pending" } ]
```

**Response Headers (v1):**

```
X-API-Version: 1
Deprecation: true
Sunset: Wed, 31 Dec 2025 00:00:00 GMT
```

**Why This Output Occurs:** The middleware reads the `X-API-Version` header, validates it, and stores it on the request attributes. The controller reads the version and returns the appropriate response structure. The middleware adds version and deprecation headers to the response. Header-based versioning keeps the URL clean but requires clients to send a custom header, which is less discoverable than URI path versioning.

### Real-World Cases

- **Public APIs (Stripe, GitHub, Twilio):** URI path versioning (`/v1/`, `/v2/`) is the industry standard for public APIs, providing clear, explicit version identification.
- **Internal microservices:** Header-based versioning is often used when an API gateway manages routing and version resolution.
- **Mobile app backends:** URI versioning allows old app versions to continue functioning while new versions use the latest API.
- **Enterprise integrations:** Sunset headers provide enterprise clients with clear timelines for migration, satisfying compliance and planning requirements.

---

## 5. Idempotency

### Definitions

**Core Definition:** Idempotency is the property of an API operation where performing the same request multiple times produces the same result as performing it once, preventing duplicate side effects such as double charges or duplicate orders.

**Technical Definition:** Idempotency in Laravel APIs is implemented via middleware that requires clients to send a unique `Idempotency-Key` header with mutating requests (POST, PATCH, PUT). The middleware stores the response associated with the key in a cache store (Redis, database, or file). When a request with the same key arrives again, the middleware returns the cached response without executing the controller. Concurrent duplicate requests (arriving while the original is still processing) receive a `409 Conflict` response with a `Retry-After: 1` header.

**Beginner-Friendly Explanation:** Imagine you are ordering a pizza online. You click "Place Order," but the internet hiccups and you are not sure if the order went through. You click again. Without idempotency, you might end up with two pizzas and two charges. With idempotency, your app sends a unique "order key" with the request. If the same key arrives twice, the server knows it is the same order and returns the original confirmation without placing a second order. This is especially important for payments, bookings, and any operation where duplicates are costly.

### Purposes

- To prevent duplicate mutations from network retries, timeouts, or user double-clicks.
- To ensure that critical operations (payments, orders, bookings) are processed exactly once.
- To provide a safe mechanism for clients to retry requests without fear of side effects.
- To handle concurrent duplicate requests gracefully with a `409 Conflict` response.
- To improve reliability in distributed systems where network failures are common.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Installation: composer require dabergut/laravel-idempotency

// Apply middleware to routes that should be idempotent
Route::post('/orders', CreateOrderController::class)->middleware('idempotent');
Route::patch('/orders/{order}', UpdateOrderController::class)->middleware('idempotent');

// Or apply to a group
Route::middleware('idempotent')->group(function () {
    Route::post('/orders', CreateOrderController::class);
    Route::post('/payments', ProcessPaymentController::class);
    Route::patch('/orders/{order}', UpdateOrderController::class);
});
```

```bash
# Client sends the Idempotency-Key header
POST /api/orders HTTP/1.1
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{"product_id": 42, "quantity": 1}
```

**Expected Response Headers:**

```
Idempotent-Replayed: false
```

**Component Breakdown:**

- `middleware('idempotent')` — Applies the idempotency middleware to the route. The middleware checks for the `Idempotency-Key` header.
- `Idempotency-Key` — A client-generated unique identifier (typically a UUID) for the request. The same key must be used for retries of the same operation.
- `Idempotent-Replayed: false` — Response header indicating the request was processed fresh (not a replay).
- `Idempotent-Replayed: true` — Response header indicating the response was served from cache (the request was a duplicate).
- `409 Conflict` — Returned when a concurrent duplicate request arrives while the original is still processing. Includes a `Retry-After: 1` header.

**Syntax Rules:**

- The `Idempotency-Key` header **must** be a unique value per logical operation. Clients should generate a UUID for each operation and reuse it on retries.
- The middleware **only** applies to mutating HTTP methods (POST, PATCH, PUT, DELETE). GET requests are inherently idempotent and do not require the header.
- The cached response includes the status code, headers, and body. Subsequent requests with the same key receive the exact same response.
- Idempotency keys are scoped per user or per route, depending on configuration. This prevents key collisions between users.
- The cache duration (TTL) is configurable; after expiry, the same key can be reused.

**Constraints and Limitations:**

- **Idempotency keys are the client's responsibility.** If the client does not send a key, the middleware does nothing. Well-behaved clients must generate and reuse keys.
- **Caching responses consumes storage.** For high-volume APIs, the cache store (Redis) must be sized appropriately.
- **The same key with a different payload is a conflict.** The middleware typically fingerprints the request body; if the same key is used with different data, a `422 Unprocessable Entity` or `409 Conflict` is returned.
- **Idempotency does not guarantee exactly-once delivery.** It guarantees that duplicate requests do not cause duplicate side effects, but the original request may still fail.
- **Concurrent requests require atomic locking.** Without atomic locks (e.g., Redis `SET NX`), two simultaneous requests with the same key could both execute.

### Annotated Code Examples

**Example 1: Idempotent Order Creation Endpoint**

```php
<?php
// File: routes/api.php

use App\Http\Controllers\Api\CreateOrderController;
use App\Http\Controllers\Api\ProcessPaymentController;
use Illuminate\Support\Facades\Route;

// Apply idempotency middleware to critical mutation endpoints
Route::middleware('auth:sanctum')->group(function () {
    Route::post('/orders', CreateOrderController::class)
        ->middleware('idempotent');

    Route::post('/payments', ProcessPaymentController::class)
        ->middleware('idempotent');
});
```

```php
<?php
// File: app/Http/Controllers/Api/CreateOrderController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Order;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class CreateOrderController extends Controller
{
    public function __invoke(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'product_id' => 'required|exists:products,id',
            'quantity'   => 'required|integer|min:1',
        ]);

        // This code runs only ONCE per unique Idempotency-Key.
        // If the client retries with the same key, the cached
        // response is returned without executing this logic.
        $order = Order::create([
            'user_id'    => $request->user()->id,
            'product_id' => $validated['product_id'],
            'quantity'   => $validated['quantity'],
            'status'     => 'pending',
        ]);

        return response()->json($order, 201);
    }
}
```

**Step-by-Step Setup:**

1. Install the package: `composer require dabergut/laravel-idempotency`.
2. Apply the `idempotent` middleware to mutating routes.
3. Test: Send `POST /api/orders` with an `Idempotency-Key` header. The order is created.
4. Send the same request again with the same key. The response is returned from cache; no duplicate order is created.

**Expected Output (First Request):**

```json
{
    "id": 101,
    "user_id": 1,
    "product_id": 42,
    "quantity": 1,
    "status": "pending"
}
```

**Response Header:** `Idempotent-Replayed: false`

**Expected Output (Second Request with Same Key):**

```json
{
    "id": 101,
    "user_id": 1,
    "product_id": 42,
    "quantity": 1,
    "status": "pending"
}
```

**Response Header:** `Idempotent-Replayed: true`

**Why This Output Occurs:** The middleware reads the `Idempotency-Key` header and checks the cache for a stored response. On the first request, no cached response exists, so the middleware allows the request to proceed, captures the response, and stores it in the cache with the key. On the second request with the same key, the middleware finds the cached response and returns it directly, without executing the controller. The `Idempotent-Replayed` header tells the client whether the response was fresh or cached.

---

**Example 2: Redis-Backed Idempotency with Atomic Locking**

```php
<?php
// File: config/idempotency.php

return [
    'driver' => 'redis', // 'cache', 'redis', 'database', 'dynamodb'
    'ttl' => 86400, // Cache responses for 24 hours
    'prefix' => 'idempotency:',
    'scope' => 'user', // 'user', 'route', or 'global'
];
```

```php
// The middleware automatically handles:
// 1. Atomic locking (Redis SET NX) to prevent concurrent duplicates
// 2. Body fingerprinting to detect same key with different payload
// 3. Response caching with the specified TTL

// If a concurrent duplicate request arrives while the original is processing:
// - The second request receives 409 Conflict with Retry-After: 1 header
```

**Expected Output (Concurrent Duplicate):**

```
HTTP/1.1 409 Conflict
Retry-After: 1
Content-Type: application/json

{
    "error": "A request with this idempotency key is already in progress.",
    "retry_after": 1
}
```

**Why This Output Occurs:** Redis-backed idempotency uses atomic `SET NX` (set if not exists) operations to acquire a lock on the idempotency key. If a second request arrives while the first is still processing, the lock acquisition fails, and the middleware returns a `409 Conflict` with a `Retry-After: 1` header. This prevents race conditions where two identical requests could both execute simultaneously.

### Real-World Cases

- **Payment processing:** Stripe and other payment providers require idempotency keys for all charge creation requests to prevent double charges.
- **Order creation:** E-commerce APIs use idempotency to prevent duplicate orders when clients retry after network timeouts.
- **Booking systems:** Hotel and flight booking APIs use idempotency to prevent duplicate reservations.
- **Bank transfers:** Financial APIs use idempotency keys to ensure that a transfer is not executed twice due to retry logic.
- **Webhook delivery:** Idempotency keys can be used to prevent duplicate processing of webhook payloads.

---

## 6. Error Conventions

### Definitions

**Core Definition:** Error conventions are standardized formats and structures for API error responses, enabling clients to parse and handle errors programmatically.

**Technical Definition:** Two prominent standards for API error responses are **RFC 7807 (Problem Details for HTTP APIs)**, which defines a JSON object with fields `type`, `title`, `status`, `detail`, and `instance`, and is served with the `application/problem+json` media type, and the **JSON:API error object**, which defines an `errors` array with objects containing `id`, `links`, `status`, `code`, `title`, `detail`, `source`, and `meta`. In Laravel, RFC 7807 can be implemented using packages like `api-skeletons/laravel-api-problem`, while JSON:API error formatting requires custom resource classes or dedicated packages.

**Beginner-Friendly Explanation:** When something goes wrong with an API request, the client needs to understand what happened so it can show the user a helpful message or retry the request. If every API returns errors in a different format, clients have to write custom parsing logic for each one. Standardized error formats like RFC 7807 solve this: they define exactly what fields an error response should contain (`title`, `detail`, `status`), so any client that understands the standard can handle errors from any API that follows it.

### Purposes

- To provide a consistent, machine-readable error format across all API endpoints.
- To enable clients to programmatically distinguish between different error types using the `type` or `code` field.
- To include human-readable error messages (`title`, `detail`) for display to end users.
- To allow additional error details (validation errors, context) to be included in a standardised way.
- To comply with industry standards, reducing integration friction for third-party developers.

### Syntax Rules and Structure

#### Complete General Syntax (RFC 7807 Problem Details)

```json
{
    "type": "https://example.com/probs/out-of-credit",
    "title": "You do not have enough credit.",
    "status": 403,
    "detail": "Your current balance is 30, but that costs 50.",
    "instance": "/account/12345/msgs/abc",
    "balance": 30,
    "accounts": ["/account/12345", "/account/67890"]
}
```

**Content-Type:** `application/problem+json`

**Component Breakdown:**

- `type` (string, URI) — A URI reference identifying the problem type. When dereferenced, it should provide human-readable documentation for the problem type.
- `title` (string) — A short, human-readable summary of the problem type. It should not change from occurrence to occurrence of the problem.
- `status` (integer) — The HTTP status code generated by the origin server for this occurrence of the problem.
- `detail` (string) — A human-readable explanation specific to this occurrence of the problem.
- `instance` (string, URI) — A URI reference that identifies the specific occurrence of the problem.
- Additional fields — The problem object may contain additional fields (e.g., `balance`, `accounts`) that provide context about the error.

#### Complete General Syntax (JSON:API Error Object)

```json
{
    "errors": [
        {
            "id": "abc123",
            "status": "422",
            "code": "validation_error",
            "title": "Validation Error",
            "detail": "The title field is required.",
            "source": {
                "pointer": "/data/attributes/title"
            },
            "meta": {
                "field": "title",
                "rule": "required"
            }
        }
    ]
}
```

**Component Breakdown:**

- `errors` (array) — An array of error objects. JSON:API allows multiple errors to be returned in a single response.
- `id` (string) — A unique identifier for this specific occurrence of the problem.
- `status` (string) — The HTTP status code applicable to this problem, expressed as a string.
- `code` (string) — An application-specific error code.
- `title` (string) — A short, human-readable summary of the problem.
- `detail` (string) — A human-readable explanation specific to this occurrence.
- `source` (object) — An object containing references to the source of the error. For validation errors, `pointer` references the JSON pointer to the invalid field.
- `meta` (object) — Non-standard metadata about the error.

**Syntax Rules:**

- RFC 7807 responses **must** use the `application/problem+json` media type, not `application/json`.
- The `type` field **should** be a URI that resolves to human-readable documentation. If it is `about:blank`, the `title` field is assumed to describe the problem.
- JSON:API error objects **must** be returned in an `errors` array, even if there is only one error.
- The `status` field in JSON:API errors is a string, not an integer.
- Both standards allow additional fields beyond the standard ones, providing flexibility for application-specific error details.

**Constraints and Limitations:**

- **RFC 7807 does not define a standard way to return validation errors for multiple fields.** Additional fields (like `errors`) must be added ad hoc.
- **JSON:API error objects are verbose.** The nested structure and required fields increase payload size compared to simpler error formats.
- **Neither standard is natively supported by Laravel's default exception handler.** Custom exception rendering or third-party packages are required.
- **Clients must be written to expect the chosen format.** Switching error formats between API versions is a breaking change.

### Annotated Code Examples

**Example 1: RFC 7807 Problem Details with `api-skeletons/laravel-api-problem`**

```php
<?php
// Installation: composer require api-skeletons/laravel-api-problem

// File: app/Exceptions/Handler.php (Laravel 10-) or bootstrap/app.php (Laravel 11+)

// Laravel 11+ — bootstrap/app.php
use ApiSkeletons\Laravel\ApiProblem\Facades\ApiProblem;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Http\Request;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (Throwable $e, Request $request) {
        if ($request->expectsJson()) {
            if ($e instanceof \Illuminate\Database\Eloquent\ModelNotFoundException) {
                return ApiProblem::response(
                    'The requested resource was not found.',
                    404
                );
            }

            if ($e instanceof \Illuminate\Validation\ValidationException) {
                return ApiProblem::response(
                    $e->getMessage(),
                    422,
                    null,
                    null,
                    ['errors' => $e->errors()]
                );
            }

            // Fallback for all other exceptions
            $status = method_exists($e, 'getStatusCode') ? $e->getStatusCode() : 500;
            return ApiProblem::response(
                app()->isProduction() ? 'An unexpected error occurred.' : $e->getMessage(),
                $status
            );
        }
    });
})
```

```php
// Controller — Throw exceptions normally; the handler formats them
public function show(Post $post)
{
    // Route model binding throws ModelNotFoundException if not found
    return new PostResource($post);
}
```

**Expected Output (404 Not Found):**

```
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{
    "type": "http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html",
    "title": "Not Found",
    "status": 404,
    "detail": "The requested resource was not found."
}
```

**Expected Output (422 Validation Error):**

```
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/problem+json

{
    "type": "http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html",
    "title": "Unprocessable Entity",
    "status": 422,
    "detail": "The given data was invalid.",
    "errors": {
        "title": ["The title field is required."],
        "body": ["The body field is required."]
    }
}
```

**Why This Output Occurs:** The `ApiProblem::response()` method constructs an RFC 7807 problem detail response with the correct media type (`application/problem+json`). The `type` field defaults to the HTTP status code documentation URI. The `title` field is derived from the HTTP status text. The `detail` field contains the exception message or a custom message. Additional fields (like `errors` for validation failures) are merged into the problem object.

---

**Example 2: JSON:API Error Object with a Custom Exception Handler**

```php
<?php
// File: bootstrap/app.php (Laravel 11+)

use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Http\Request;
use Illuminate\Validation\ValidationException;
use Illuminate\Database\Eloquent\ModelNotFoundException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (ValidationException $e, Request $request) {
        if ($request->expectsJson()) {
            $errors = [];
            foreach ($e->errors() as $field => $messages) {
                foreach ($messages as $message) {
                    $errors[] = [
                        'status' => '422',
                        'code'   => 'validation_error',
                        'title'  => 'Validation Error',
                        'detail' => $message,
                        'source' => ['pointer' => "/data/attributes/{$field}"],
                        'meta'   => ['field' => $field],
                    ];
                }
            }

            return response()->json(['errors' => $errors], 422);
        }
    });

    $exceptions->render(function (ModelNotFoundException $e, Request $request) {
        if ($request->expectsJson()) {
            $model = class_basename($e->getModel());
            return response()->json([
                'errors' => [[
                    'status' => '404',
                    'code'   => 'resource_not_found',
                    'title'  => 'Resource Not Found',
                    'detail' => "{$model} not found.",
                ]],
            ], 404);
        }
    });
})
```

**Expected Output (Validation Error):**

```json
{
    "errors": [
        {
            "status": "422",
            "code": "validation_error",
            "title": "Validation Error",
            "detail": "The title field is required.",
            "source": { "pointer": "/data/attributes/title" },
            "meta": { "field": "title" }
        },
        {
            "status": "422",
            "code": "validation_error",
            "title": "Validation Error",
            "detail": "The body field is required.",
            "source": { "pointer": "/data/attributes/body" },
            "meta": { "field": "body" }
        }
    ]
}
```

**Why This Output Occurs:** The custom exception handler iterates over the validation errors and constructs a JSON:API-compliant error object for each field. The `errors` array contains one object per validation failure. The `source.pointer` field uses JSON Pointer syntax to identify the invalid field in the request payload. This format is consistent with the JSON:API specification, allowing clients that understand JSON:API to parse the errors automatically.

### Real-World Cases

- **Public APIs with third-party developers:** RFC 7807 provides a standard error format that developers can document and rely on, reducing integration friction.
- **SPAs and mobile apps:** Standardised error formats allow frontend code to handle errors generically (e.g., display `detail` in a toast notification).
- **API gateways and proxies:** Gateways can parse and transform RFC 7807 error responses for logging, monitoring, or client-specific formatting.
- **Compliance and auditing:** Standardised error formats make it easier to log and audit API errors for security and compliance purposes.

---

## 7. Advanced Rate Limiting

### Definitions

**Core Definition:** Advanced rate limiting is the practice of restricting the number of requests a client can make within a time window, with customisable limits, dynamic throttling based on user attributes, and precise `Retry-After` headers when limits are exceeded.

**Technical Definition:** Laravel's rate limiting is implemented through the `RateLimiter` facade and the `throttle` middleware. Named rate limiters are defined in `AppServiceProvider::boot()` using `RateLimiter::for('name', function (Request $request) { return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip()); })`. When a limit is exceeded, Laravel returns a `429 Too Many Requests` response with `Retry-After`, `X-RateLimit-Limit`, and `X-RateLimit-Remaining` headers. Custom responses can be configured via the `response()` method on the `Limit` class, and the `Retry-After` header can be customised to provide precise retry timing.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer at a club who lets in a certain number of people per hour. If you try to enter too many times, the bouncer says "wait 30 seconds and try again." The `Retry-After` header is that "wait 30 seconds" message. Advanced rate limiting lets you set different limits for different users (e.g., premium users get more requests), customise the wait time, and return a helpful JSON response instead of a generic error page.

### Purposes

- To protect the API from abuse, brute-force attacks, and denial-of-service attempts.
- To ensure fair resource allocation by preventing any single client from monopolising server capacity.
- To enforce tiered access with different rate limits for free, premium, and enterprise users.
- To provide clients with clear, actionable information about when they can retry (`Retry-After` header).
- To dynamically adjust limits based on user attributes, endpoint sensitivity, or time of day.

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
        // Basic rate limiter — 60 requests per minute per user/IP
        RateLimiter::for('api', function (Request $request) {
            return Limit::perMinute(60)->by(
                $request->user()?->id ?: $request->ip()
            );
        });

        // Tiered rate limiter — different limits for free vs. premium users
        RateLimiter::for('premium-api', function (Request $request) {
            if ($request->user()?->isPremium()) {
                return Limit::perMinute(1000)->by($request->user()->id);
            }
            return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
        });

        // Login rate limiter — 5 attempts per minute per email+IP
        RateLimiter::for('login', function (Request $request) {
            return [
                Limit::perMinute(5)->by($request->input('email') . '|' . $request->ip()),
                Limit::perMinute(20)->by($request->ip()),
            ];
        });

        // Custom response with Retry-After header
        RateLimiter::for('custom', function (Request $request) {
            return Limit::perMinute(10)
                ->by($request->ip())
                ->response(function (Request $request, array $headers) {
                    return response()->json([
                        'error' => 'Rate limit exceeded. Please try again later.',
                        'retry_after' => $headers['Retry-After'] ?? null,
                    ], 429, $headers);
                });
        });
    }
}
```

```php
// File: routes/api.php — Apply rate limiters to routes

use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('posts', PostController::class);
});

Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:login');

Route::middleware(['auth:sanctum', 'throttle:premium-api'])->group(function () {
    Route::get('/premium/data', [PremiumController::class, 'index']);
});
```

**Component Breakdown:**

- `RateLimiter::for('api', ...)` — Defines a named rate limiter called `api`.
- `Limit::perMinute(60)` — Creates a limit of 60 requests per minute.
- `->by($request->user()?->id ?: $request->ip())` — Segments the limit by user ID (if authenticated) or IP address (if guest).
- `->response(function (Request $request, array $headers) { ... })` — Customises the `429` response. The `$headers` array contains `Retry-After`, `X-RateLimit-Limit`, and `X-RateLimit-Remaining`.
- `throttle:api` — Applies the `api` rate limiter to the route or group.
- `Limit::perMinute(5)->by($request->input('email') . '|' . $request->ip())` — Segments the login limit by email+IP combination, preventing both brute-force on a single account and distributed attacks from a single IP.

**Syntax Rules:**

- Named rate limiters are defined in a service provider's `boot()` method.
- The `by()` method segments the limit. Without it, the limit is applied globally across all requests.
- Multiple limits can be returned as an array: `return [Limit::perMinute(10), Limit::perDay(1000)]`. The most restrictive limit applies.
- The `response()` method customises the 429 response and receives the `$headers` array containing `Retry-After`.
- Rate limiting requires a cache store (Redis recommended for distributed applications).

**Constraints and Limitations:**

- **Rate limiting counters are stored in the cache.** If the cache is cleared, all limits are reset.
- **The `Retry-After` header value is in seconds.** Clients should parse it as an integer and wait that many seconds before retrying.
- **Rate limiters defined in a service provider are resolved at boot time.** Dynamic limits based on request attributes are evaluated per request, but the limiter definition itself is static.
- **Custom responses must return a response with a 429 status code.** Returning a different status code may confuse clients.
- **The `X-RateLimit-Remaining` header may not be present in all responses.** It is added only when the `throttle` middleware is applied and the limit has not been exceeded.

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

```php
// routes/api.php
Route::middleware('throttle:api')->group(function () {
    Route::get('/data', [DataController::class, 'index']);
    Route::get('/user', function (Request $request) {
        return $request->user();
    })->middleware('auth:sanctum');
});
```

**Step-by-Step Setup:**

1. Define the tiered rate limiter in `AppServiceProvider::boot()`.
2. Apply `throttle:api` to the API routes.
3. Test with a free user, premium user, and unauthenticated request.

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

**Why This Output Occurs:** The rate limiter checks the authenticated user's tier and returns the corresponding `Limit` object. The `response()` method customises the 429 response with tier-specific messaging. The `Retry-After` header is automatically added by Laravel's `ThrottleRequests` middleware and is included in the `$headers` array passed to the response callback. The `X-RateLimit-Limit` and `X-RateLimit-Remaining` headers provide clients with visibility into their current rate limit status.

---

**Example 2: Rate Limiting with `Retry-After` and `X-RateLimit-*` Headers**

```php
// The following headers are automatically added to every response
// when the throttle middleware is applied:

// Successful request (under limit):
// X-RateLimit-Limit: 60
// X-RateLimit-Remaining: 45

// Rate-limited request (over limit):
// HTTP/1.1 429 Too Many Requests
// Retry-After: 30
// X-RateLimit-Limit: 60
// X-RateLimit-Remaining: 0

// Client-side handling example (JavaScript):
async function fetchWithRetry(url, options = {}) {
    const response = await fetch(url, options);

    if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After');
        const remaining = response.headers.get('X-RateLimit-Remaining');

        console.warn(`Rate limited. Retry after ${retryAfter} seconds.`);

        // Wait and retry
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
        return fetchWithRetry(url, options);
    }

    return response;
}
```

**Expected Output:**

- Under limit: `X-RateLimit-Remaining` decreases with each request.
- Over limit: `429 Too Many Requests` with `Retry-After: 30` and `X-RateLimit-Remaining: 0`.
- After waiting 30 seconds: the request succeeds and `X-RateLimit-Remaining` is reset to the limit value.

**Why This Output Occurs:** Laravel's `ThrottleRequests` middleware increments a counter in the cache for each request. The `X-RateLimit-Limit` header reports the maximum number of requests allowed in the current window, and `X-RateLimit-Remaining` reports how many requests are left. When the limit is exceeded, the middleware returns a `429` response with a `Retry-After` header indicating how many seconds the client should wait before retrying. The client can use this information to implement exponential backoff or simply wait the specified time.

### Real-World Cases

- **Public API tiers:** Free users get 60 requests/minute, premium users get 1,000 requests/minute, enterprise users get custom limits.
- **Authentication endpoints:** Login and registration endpoints are throttled to 5–10 attempts per minute per IP to prevent brute-force attacks.
- **AI and ML APIs:** Expensive endpoints (e.g., image generation, large language model inference) are throttled aggressively to control costs.
- **Webhook delivery:** Outgoing webhooks are rate-limited to prevent overwhelming the receiving service.
- **Third-party API proxies:** Your application acts as a proxy to an external API with its own rate limits; you throttle your own users to stay within those limits.

---

## References

- Laravel Pagination Documentation — https://laravel.com/docs/pagination
- Laravel Cursor Pagination (Laravel News) — https://laravel-news.com/cursor-pagination
- Eloquent: Getting Started (Local Scopes) — https://laravel.com/docs/eloquent#local-scopes
- Spatie Laravel Query Builder — https://github.com/spatie/laravel-query-builder
- Laravel Scout Documentation — https://laravel.com/docs/scout
- Meilisearch Laravel Scout Guide — https://www.meilisearch.com/docs/learn/integrations/laravel_scout
- Laravel API Route (Grazulex) — https://laravel-news.com/laravel-api-route
- RFC 8594: The Sunset HTTP Header Field — https://www.rfc-editor.org/rfc/rfc8594
- RFC 9745: The Deprecation HTTP Header Field — https://www.rfc-editor.org/rfc/rfc9745
- Laravel Idempotency (dabergut) — https://packagist.org/packages/dabergut/laravel-idempotency
- Laravel Idempotency with Redis (OthmanCharai) — https://github.com/OthmanCharai/Idempotency
- RFC 7807: Problem Details for HTTP APIs — https://www.rfc-editor.org/rfc/rfc7807
- JSON:API Error Objects — https://jsonapi.org/format/#errors
- api-skeletons/laravel-api-problem — https://packagist.org/packages/api-skeletons/laravel-api-problem
- Laravel Rate Limiting Documentation — https://laravel.com/docs/routing#rate-limiting
- Laravel Rate Limiting Guide (OneUptime) — https://github.com/OneUptime/blog/blob/master/posts/2026-02-03-laravel-rate-limiting/README.md
- RFC 7231: Retry-After Header — https://www.rfc-editor.org/rfc/rfc7231#section-7.1.3