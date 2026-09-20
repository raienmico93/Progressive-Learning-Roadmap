# Laravel Response Architecture & Consistency: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Response Architecture & Consistency refers to the framework's unified system for constructing, transforming, and delivering HTTP responses, ensuring that headers, cookies, JSON payloads, pagination metadata, and SPA hydration data all follow consistent, reusable patterns across an application.

**Technical Definition:** All Laravel responses extend `Symfony\Component\HttpFoundation\Response` through `Illuminate\Http\Response` and its subclasses (`JsonResponse`, `RedirectResponse`, `BinaryFileResponse`, `StreamedResponse`). The `ResponseFactory` (accessed via the `response()` helper) creates these instances, while `JsonResource` and `ResourceCollection` provide a transformation layer between Eloquent models and JSON output. Response macros (registered via `Response::macro()`) allow custom, reusable response formats to be defined in service providers. For SPA architectures, Inertia.js and Livewire provide specialised response types that hydrate frontend components with server-side data.

**Beginner-Friendly Explanation:** When your application sends something back to the user—whether it's a web page, JSON data, a file download, or the data needed for a React or Vue component—Laravel provides a consistent set of tools for building that response. You can add custom headers, attach cookies, transform your database models into clean JSON, paginate large datasets with automatic metadata, and even build reusable response templates. This consistency means you write less code and get more predictable results.

### Key Characteristics

- **Unified Response Abstraction:** All response types share a common base class, ensuring consistent handling throughout the framework.
- **Fluent Interface:** Response methods are chainable, allowing headers, cookies, status codes, and content to be configured in a single expression.
- **Transformation Layer:** Eloquent API Resources provide granular control over JSON serialization, hiding sensitive attributes and shaping nested relationships.
- **Automatic Pagination Metadata:** Length-aware paginators automatically include `meta` and `links` keys in JSON responses.
- **Extensibility via Macros:** Custom response formats can be registered globally using `Response::macro()`.
- **SPA-Ready:** Inertia.js and Livewire provide dedicated response types for modern frontend architectures.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (the Response API is stable across versions; Inertia.js and Livewire are separate packages with their own version requirements).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with routes, controllers, and Eloquent models configured.
- For Inertia: `inertiajs/inertia-laravel` installed and configured.
- For Livewire: `livewire/livewire` installed and configured.

### Related Programming Areas

- **HTTP Responses:** The foundational layer for all response types.
- **Eloquent API Resources:** The transformation layer between models and JSON.
- **Pagination:** Length-aware paginators integrate with API Resources to provide automatic metadata.
- **Service Providers:** Response macros are registered in service providers.
- **SPA Frameworks:** Inertia.js and Livewire bridge Laravel's backend with modern frontend frameworks.

### Core Concepts / Features

1. Custom Headers & Cookies
2. Eloquent API Resources
3. Pagination Payload Formats
4. Global Response Abstractions (Response Macros)
5. SPA Adaptations (Inertia & Livewire)

---

## 1. Custom Headers & Cookies

### Definitions

**Core Definition:** Custom headers and cookies are transport-layer metadata that can be attached to any Laravel response, allowing developers to mutate HTTP headers for caching, CORS, rate limiting, and other purposes, and to append cookies for user tracking, session management, and personalisation.

**Technical Definition:** The `Illuminate\Http\Response` class provides `header()`, `withHeaders()`, and `withoutHeader()` methods for manipulating the response's `HeaderBag`, and `cookie()` and `withCookie()` methods for attaching `Symfony\Component\HttpFoundation\Cookie` instances. The `header()` method sets a single header (replacing by default), `withHeaders()` merges an array of headers, and `withCookie()` appends a cookie to the response's cookie bag. Cookies can be created using the `cookie()` helper, which returns a `Cookie` instance with configurable name, value, expiration, path, domain, secure flag, and HTTP-only flag.

**Beginner-Friendly Explanation:** When your application sends a response, it can include extra information called "headers" that tell the browser how to handle the response—like whether to cache it, what content type it is, or CORS permissions. Cookies are small pieces of data stored in the user's browser that can remember things like login sessions or preferences. Laravel makes it easy to attach both to any response with simple, chainable methods.

### Purposes

- To mutate transport channels using `->header()` arrays for caching, CORS, and security headers.
- To append user tracking states via `->withCookie()` for session management and personalisation.
- To remove headers that are no longer needed using `->withoutHeader()`.
- To set cookies with custom expiration, path, domain, and security flags.
- To expire cookies using `->withoutCookie()` for logout functionality.

### Syntax Rules and Structure

**Complete General Syntax — Attaching Headers:**

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Custom-Header', 'value');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->header('Name', 'Value')` | Sets a single header on the response. |
| `->withHeaders(['Name' => 'Value'])` | Sets multiple headers at once. |
| `->withoutHeader('Name')` | Removes a header from the response. |

**Complete General Syntax — Attaching Cookies:**

```php
return response($content)
    ->withCookie('name', 'value', $minutes);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->withCookie('name', 'value', $minutes)` | Attaches a cookie with the given name, value, and expiration. |
| `->cookie(Cookie $cookie)` | Attaches a pre-built `Cookie` instance. |
| `->withoutCookie('name')` | Expires the named cookie. |

**Complete General Syntax — Cookie Helper:**

```php
$cookie = cookie('name', 'value', $minutes, $path, $domain, $secure, $httpOnly);
return response('Hello')->cookie($cookie);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `$minutes` | Expiration time in minutes. |
| `$path` | Cookie path (default `/`). |
| `$domain` | Cookie domain. |
| `$secure` | Whether the cookie is only sent over HTTPS. |
| `$httpOnly` | Whether the cookie is inaccessible to JavaScript. |

**Syntax Rules:**

- `header()` replaces existing headers by default; pass `false` as the third argument to append.
- `withHeaders()` merges the given array with existing headers, overriding duplicates.
- `withCookie()` accepts either a name-value-minutes combination or a `Cookie` instance.
- Cookies are encrypted and signed by Laravel's `EncryptCookies` middleware when using the `web` middleware group.
- API routes (without the `web` middleware group) do not encrypt cookies by default.

**Constraints and Limitations:**

- **Header size:** HTTP headers have size limits (typically 8KB per header, 16KB total). Avoid storing large amounts of data in headers.
- **Cookie size:** Cookies are limited to 4KB each. Use session storage for larger data.
- **Cookie encryption:** Cookies are encrypted in the `web` middleware group, but not in API routes unless explicitly added.
- **HTTP-only flag:** For security, sensitive cookies (like session IDs) should have the `httpOnly` flag set to prevent XSS access.

### Multiple Annotated Complete Code Examples

**Example 1: Attaching Custom Headers**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/api/data', function () {
    $data = ['id' => 1, 'name' => 'Example'];

    // Attach multiple headers in a fluent chain
    return response()->json($data)
        ->header('X-RateLimit-Limit', '60')
        ->header('X-RateLimit-Remaining', '59')
        ->header('X-Cache-Status', 'HIT');
});
```

**Expected Output:** `GET /api/data` returns the JSON payload with the custom headers `X-RateLimit-Limit: 60`, `X-RateLimit-Remaining: 59`, and `X-Cache-Status: HIT`.

**Why:** The `header()` method sets each header individually. Multiple calls can be chained. The `response()->json()` method creates the base `JsonResponse`, and the chained `header()` calls add the custom headers before the response is sent.

---

**Example 2: Attaching a Cookie**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/set-theme', function () {
    // Create a cookie that lasts for 1 day (1440 minutes)
    $cookie = cookie('theme', 'dark', 1440, '/', null, false, true);

    return response('Theme preference saved.')
        ->withCookie($cookie);
});
```

**Expected Output:** `GET /set-theme` returns the string `Theme preference saved.` and sets a cookie named `theme` with value `dark`, expiring in 1 day.

**Why:** The `cookie()` helper creates a `Cookie` instance with the specified parameters. The `withCookie()` method attaches it to the response. The browser stores the cookie and sends it back on subsequent requests.

---

**Example 3: Removing a Header**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/clean-response', function () {
    return response('Hello')
        ->header('X-Powered-By', 'Laravel')
        ->withoutHeader('X-Powered-By');
});
```

**Expected Output:** `GET /clean-response` returns `Hello` without the `X-Powered-By` header.

**Why:** The `withoutHeader()` method removes the specified header from the response, which is useful for security hardening (e.g., removing `X-Powered-By` to avoid revealing the framework).

### Real-World Cases

- **Rate Limiting:** API responses include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `Retry-After` headers.
- **CORS:** Cross-origin API responses include `Access-Control-Allow-Origin` and related headers.
- **Caching:** Responses include `Cache-Control`, `ETag`, and `Last-Modified` headers for HTTP caching.
- **User Preferences:** Cookies store theme settings, language preferences, and dismissed notifications.
- **Security:** `Strict-Transport-Security`, `X-Content-Type-Options`, and `X-Frame-Options` headers are set for security hardening.

### References

- Laravel HTTP Responses: Attaching Headers — https://laravel.com/docs/12.x/responses#attaching-headers-to-responses
- Laravel HTTP Responses: Attaching Cookies — https://laravel.com/docs/12.x/responses#attaching-cookies-to-responses
- Laravel API: Response — https://api.laravel.com/docs/12.x/Illuminate/Http/Response.html
- Laravel API: Cookie — https://api.laravel.com/docs/12.x/Illuminate/Http/Cookie.html

---

## 2. Eloquent API Resources

### Definitions

**Core Definition:** Eloquent API Resources are transformation classes that sit between Eloquent models and JSON responses, providing granular control over which attributes and relationships are included in API output.

**Technical Definition:** A resource class extends `Illuminate\Http\Resources\Json\JsonResource` and defines a `toArray(Request $request)` method that returns the array of attributes to be serialised. A collection resource extends `Illuminate\Http\Resources\Json\ResourceCollection` and handles collections of models. Resources proxy property and method access to the underlying model via the `$this` variable. Conditional attributes are included using `when()`, `whenHas()`, and `whenNotNull()`, and relationships are conditionally included using `whenLoaded()`.

**Beginner-Friendly Explanation:** When you return a model directly, Laravel dumps all its database columns into JSON—including sensitive ones like passwords. API Resources are like a filter: you decide exactly which fields are included, how they're formatted, and which relationships are embedded. This gives you full control over your API's output shape.

### Purposes

- To standardise the transformation, nesting, and structure of JSON API models via `JsonResource` and `ResourceCollection`.
- To hide sensitive attributes (passwords, tokens, internal IDs) from API consumers.
- To include only the relationships that are loaded, avoiding N+1 query leaks.
- To add computed attributes (e.g., `full_name`, `excerpt`) without modifying the model.
- To conditionally include attributes based on user permissions or request context.
- To wrap responses in a consistent `data` envelope for easier client-side parsing.

### Syntax Rules and Structure

**Complete General Syntax — Resource Class:**

```php
<?php

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
            'created_at' => $this->created_at->toDateString(),
        ];
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends JsonResource` | The base class for API resources. |
| `toArray(Request $request)` | Returns the array of attributes to serialise. |
| `$this->id` | Accesses the underlying model's attributes. |

**Complete General Syntax — Using a Resource:**

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `new UserResource($user)` | Transforms a single model. |
| `UserResource::collection($users)` | Transforms a collection of models. |

**Complete General Syntax — Conditional Attributes:**

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
        'email' => $this->whenHas('email'),
        'nickname' => $this->whenNotNull($this->nickname),
    ];
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `when($condition, $value)` | Includes the attribute only if the condition is true. |
| `whenHas('email')` | Includes the attribute if it is present on the model. |
| `whenNotNull($value)` | Includes the attribute if it is not null. |

**Complete General Syntax — Conditional Relationships:**

```php
'author' => new UserResource($this->whenLoaded('user')),
'comments' => CommentResource::collection($this->whenLoaded('comments')),
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `whenLoaded('user')` | Includes the relationship only if it has been eager-loaded. |

**Syntax Rules:**

- Resource classes are generated via `php artisan make:resource UserResource`.
- Collection resources are generated via `php artisan make:resource UserCollection --collection` or by including `Collection` in the name.
- The `toArray()` method receives the current `Request` instance for conditional logic.
- `whenLoaded()` only includes a relationship if it has been eager-loaded—you must eager-load relationships in the controller to include them.
- By default, resources wrap the output in a `data` key. Disable this via `JsonResource::withoutWrapping()`.

**Constraints and Limitations:**

- **`data` wrapper:** API Resources automatically wrap responses in a `data` key. Client-side code must access `response.data.data` for the actual payload.
- **Relationship loading:** `whenLoaded()` does not eager-load relationships—it only checks if they are already loaded. Eager-load in the controller or query.
- **Performance:** Resources add a transformation layer. For simple endpoints, returning the model directly may be sufficient.
- **Nested resources:** Transforming deeply nested relationships can become complex. Consider using separate resource classes for each model.

### Multiple Annotated Complete Code Examples

**Example 1: Basic API Resource**

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
            'created_at' => $this->created_at->toDateString(),
        ];
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

**Expected Output:**
- `GET /api/users` → `{"data": [{"id": 1, "name": "Alice", "email": "alice@example.com", "created_at": "2026-01-15"}, ...]}`
- `GET /api/user/1` → `{"data": {"id": 1, "name": "Alice", "email": "alice@example.com", "created_at": "2026-01-15"}}`

**Why:** The `UserResource` controls exactly which fields are exposed. The `created_at` date is formatted as a date string instead of a full timestamp. The response is wrapped in a `data` key.

---

**Example 2: Resource with Conditional Relationships**

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
            'excerpt' => \Str::limit($this->body, 100),
            'author' => new UserResource($this->whenLoaded('user')),
            'comments_count' => $this->whenLoaded('comments', fn() => $this->comments->count()),
            'tags' => TagResource::collection($this->whenLoaded('tags')),
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\PostResource;
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with(['user', 'tags'])->paginate(20);
        return PostResource::collection($posts);
    }

    public function show(Post $post)
    {
        $post->load(['user', 'comments', 'tags']);
        return new PostResource($post);
    }
}
```

**Expected Output:**
- `GET /api/posts` → includes `author` and `tags` (eager-loaded), but not `comments_count` (not loaded).
- `GET /api/posts/1` → includes `author`, `comments_count`, and `tags` (all loaded).

**Why:** `whenLoaded()` conditionally includes relationships based on whether they have been eager-loaded. This prevents N+1 query problems and ensures that only the necessary data is included in each response.

---

**Example 3: Customising Pagination Information**

```php
<?php
// app/Http/Resources/PostCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class PostCollection extends ResourceCollection
{
    public function toArray($request): array
    {
        return [
            'data' => $this->collection,
            'meta' => [
                'total' => $this->total(),
                'per_page' => $this->perPage(),
                'current_page' => $this->currentPage(),
            ],
        ];
    }

    public function paginationInformation($request, $paginated, $default)
    {
        $default['links']['custom'] = 'https://example.com/docs';
        return $default;
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Resources\PostCollection;
use App\Models\Post;
use Illuminate\Support\Facades\Route;

Route::get('/posts', function () {
    return new PostCollection(Post::paginate(15));
});
```

**Expected Output:** `GET /api/posts` returns a paginated JSON response with `data`, `links`, and `meta` keys, including a custom `links.custom` entry.

**Why:** The `paginationInformation()` method receives the paginator data and the default `links` and `meta` arrays, allowing customisation of the pagination metadata.

### Real-World Cases

- **API Development:** API Resources are the standard way to shape JSON responses in Laravel APIs.
- **Mobile App Backends:** Resources ensure consistent data structures for iOS and Android clients.
- **SPA Backends:** Vue and React front-ends consume resource-transformed data with the `data` wrapper.
- **Multi-Version APIs:** Different resource classes for `v1` and `v2` allow API versioning without changing models.
- **Public vs. Admin APIs:** Different resources can expose different fields for public and admin consumers.

### References

- Laravel Eloquent: API Resources — https://laravel.com/docs/12.x/eloquent-resources
- Laravel API: JsonResource — https://api.laravel.com/docs/12.x/Illuminate/Http/Resources/Json/JsonResource.html
- Laravel API: ResourceCollection — https://api.laravel.com/docs/12.x/Illuminate/Http/Resources/Json/ResourceCollection.html

---

## 3. Pagination Payload Formats

### Definitions

**Core Definition:** Laravel's pagination payload formats are the standardised JSON structures emitted by length-aware paginators, containing `data`, `links`, and `meta` keys that provide navigational and statistical information about the paginated dataset.

**Technical Definition:** When a `LengthAwarePaginator` or `CursorPaginator` instance is returned from a route or controller, Laravel automatically serialises it to JSON with a `data` array containing the paginated items, a `links` object with `first`, `last`, `prev`, and `next` URLs, and a `meta` object with `current_page`, `from`, `last_page`, `path`, `per_page`, `to`, and `total` fields. When used with API Resources, the pagination structure is preserved, and the `paginationInformation()` method on the resource can customise the `links` and `meta` keys.

**Beginner-Friendly Explanation:** When you have a lot of data—like hundreds of blog posts—you don't want to send all of it at once. Pagination breaks the data into pages. Laravel automatically includes information about which page you're on, how many total items there are, and links to the next and previous pages. This metadata helps front-end developers build pagination controls without any extra work.

### Purposes

- To handle paginated metadata (e.g., `meta`, `links`, `total` count arrays) emitted automatically by Laravel's length-aware query engines.
- To provide consistent pagination structure across all API endpoints.
- To enable front-end pagination controls with minimal server-side configuration.
- To support cursor-based pagination for real-time datasets.
- To allow customisation of pagination metadata via the `paginationInformation()` method.

### Syntax Rules and Structure

**Complete General Syntax — Paginating a Query:**

```php
$users = User::paginate(15);
return response()->json($users);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `User::paginate(15)` | Returns a `LengthAwarePaginator` with 15 items per page. |
| JSON structure | Includes `data`, `links`, and `meta` keys. |

**Complete General Syntax — Paginated JSON Structure:**

```json
{
    "data": [...],
    "links": {
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=3",
        "prev": null,
        "next": "http://example.com/users?page=2"
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 3,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 15,
        "total": 42
    }
}
```

**Component Breakdown:**

| Key | Description |
|-----|-------------|
| `data` | The array of paginated items. |
| `links.first` | URL to the first page. |
| `links.last` | URL to the last page. |
| `links.prev` | URL to the previous page (null on first page). |
| `links.next` | URL to the next page (null on last page). |
| `meta.current_page` | The current page number. |
| `meta.last_page` | The total number of pages. |
| `meta.per_page` | Items per page. |
| `meta.total` | Total number of items. |

**Complete General Syntax — Customising Pagination Metadata:**

```php
public function paginationInformation($request, $paginated, $default)
{
    $default['links']['custom'] = 'https://example.com/docs';
    return $default;
}
```

**Complete General Syntax — Cursor Pagination:**

```php
$users = User::cursorPaginate(15);
return response()->json($users);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `cursorPaginate(15)` | Uses cursor-based pagination (no total count). |
| JSON structure | Includes `data`, `links`, and `meta` with `next_cursor` and `prev_cursor`. |

**Syntax Rules:**

- `paginate()` uses offset-based pagination with a total count.
- `simplePaginate()` uses offset-based pagination without a total count (faster for large datasets).
- `cursorPaginate()` uses cursor-based pagination, ideal for real-time or frequently changing datasets.
- When used with API Resources, the pagination structure is preserved and can be customised via `paginationInformation()`.
- The `links` and `meta` keys are automatically included in JSON responses.

**Constraints and Limitations:**

- **Performance:** `paginate()` performs a `COUNT(*)` query for the total; for large tables, consider `simplePaginate()` or `cursorPaginate()`.
- **Cursor pagination:** Cursor-based pagination does not include `total` or `last_page` because the total count is not known without an expensive query.
- **Customisation:** The `paginationInformation()` method must be defined on the resource class, not the controller.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Paginated Response**

```php
<?php
// routes/api.php

use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::get('/users', function () {
    // Paginate users with 15 per page
    return User::paginate(15);
});
```

**Expected Output:** `GET /api/users` returns:
```json
{
    "data": [
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        ...
    ],
    "links": {
        "first": "http://example.com/api/users?page=1",
        "last": "http://example.com/api/users?page=3",
        "prev": null,
        "next": "http://example.com/api/users?page=2"
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 3,
        "path": "http://example.com/api/users",
        "per_page": 15,
        "to": 15,
        "total": 42
    }
}
```

**Why:** Laravel's `LengthAwarePaginator` automatically serialises to this JSON structure, including navigation links and statistical metadata.

---

**Example 2: Paginated API Resource**

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
        ];
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::get('/users', function () {
    // Return a paginated resource collection
    return UserResource::collection(User::paginate(15));
});
```

**Expected Output:** `GET /api/users` returns the same pagination structure, but with each user transformed by the `UserResource` class (only `id`, `name`, and `email` are included).

**Why:** When a resource collection wraps a paginator, the pagination metadata is preserved and the `data` array contains the transformed resources.

---

**Example 3: Customising Pagination Metadata**

```php
<?php
// app/Http/Resources/UserCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    public function paginationInformation($request, $paginated, $default)
    {
        // Add a custom link to the pagination metadata
        $default['links']['custom'] = 'https://example.com/docs/pagination';
        return $default;
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Resources\UserCollection;
use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::get('/users', function () {
    return new UserCollection(User::paginate(15));
});
```

**Expected Output:** `GET /api/users` returns the standard pagination structure with an additional `links.custom` entry pointing to `https://example.com/docs/pagination`.

**Why:** The `paginationInformation()` method allows customising the `links` and `meta` keys. The `$default` array contains the standard `links` and `meta` structures, which can be modified before being returned.

### Real-World Cases

- **API List Endpoints:** All list endpoints return paginated responses with `meta` and `links` for front-end pagination controls.
- **Admin Data Tables:** Paginated responses populate data tables with page numbers and total counts.
- **Mobile App Feeds:** Infinite scroll implementations use `links.next` to load the next page of data.
- **Real-Time Feeds:** Cursor-based pagination is used for real-time feeds where the total count is not known.
- **Search Results:** Paginated search results include `meta.total` for displaying "X results found".

### References

- Laravel Eloquent: API Resources — Pagination — https://laravel.com/docs/12.x/eloquent-resources#pagination
- Laravel Database: Pagination — https://laravel.com/docs/12.x/pagination
- Laravel API: LengthAwarePaginator — https://api.laravel.com/docs/12.x/Illuminate/Pagination/LengthAwarePaginator.html
- Laravel API: CursorPaginator — https://api.laravel.com/docs/12.x/Illuminate/Pagination/CursorPaginator.html

---

## 4. Global Response Abstractions (Response Macros)

### Definitions

**Core Definition:** Response macros are custom, reusable response formats registered on the `Response` facade using `Response::macro()`, allowing developers to define application-wide response patterns (e.g., success, error, paginated) that can be invoked from any controller or route.

**Technical Definition:** The `Illuminate\Support\Facades\Response` facade uses the `Macroable` trait, which provides the `macro()` static method for registering custom methods. Macros are registered in a service provider's `boot()` method and are invoked as static calls on the `Response` facade or as instance methods on response objects. Each macro receives the arguments passed to it and returns a `Response` instance. Macros are resolved through the `__callStatic` magic method, which checks the registered macros array.

**Beginner-Friendly Explanation:** If you find yourself writing the same JSON response structure over and over—like `response()->json(['success' => true, 'data' => $data, 'message' => $message])`—you can create a macro to encapsulate that pattern. Once registered, you can call `Response::success($data, $message)` from anywhere in your application. It's like creating your own custom response shortcuts.

### Purposes

- To register custom, highly reusable response formats inside service providers using `Response::macro()`.
- To ensure consistency across all API responses by centralising response structure.
- To reduce boilerplate code in controllers by encapsulating common response patterns.
- To provide a single point of change for response format updates.
- To make response code more readable and expressive.

### Syntax Rules and Structure

**Complete General Syntax — Registering a Macro:**

```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\Response;

Response::macro('success', function (string $message, array $data = [], int $status = 200) {
    return response()->json([
        'success' => true,
        'message' => $message,
        'data' => $data,
    ], $status);
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Response::macro('success', ...)` | Registers a macro named `success`. |
| `function (string $message, array $data = [], int $status = 200)` | The macro's signature. |
| `return response()->json(...)` | The macro returns a `JsonResponse`. |

**Complete General Syntax — Using a Macro:**

```php
// In a controller or route
return Response::success('User created', $user, 201);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Response::success(...)` | Invokes the registered macro. |
| `'User created'` | The message argument. |
| `$user` | The data argument. |
| `201` | The status code argument. |

**Complete General Syntax — Error Macro:**

```php
Response::macro('error', function (string $message, array $errors = [], int $status = 400) {
    return response()->json([
        'success' => false,
        'message' => $message,
        'errors' => $errors,
    ], $status);
});
```

**Syntax Rules:**

- Macros must be registered before they are used—typically in a service provider's `boot()` method.
- Macros are invoked as static calls on the `Response` facade: `Response::macroName(...)`.
- Macros can accept any number of arguments and return any type of response.
- Macros can be chained with other response methods: `Response::success(...)->header(...)`.
- The `Response` facade and the `response()` helper share the same macro registry.

**Constraints and Limitations:**

- **Registration timing:** Macros must be registered before the response is constructed. Registering in `boot()` ensures availability.
- **Closure serialisation:** Macros using Closures cannot be serialised for route caching. Use class methods for cache compatibility.
- **Naming collisions:** Macro names must not conflict with existing `Response` methods (e.g., `header`, `cookie`).
- **Laravel 12 compatibility:** There are reports of `ResponseFactory` macros breaking in Laravel 12; test thoroughly after upgrading.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Success and Error Macros**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Register a success response macro
        Response::macro('success', function (string $message, mixed $data = null, int $status = 200) {
            return response()->json([
                'success' => true,
                'message' => $message,
                'data' => $data,
            ], $status);
        });

        // Register an error response macro
        Response::macro('error', function (string $message, array $errors = [], int $status = 400) {
            return response()->json([
                'success' => false,
                'message' => $message,
                'errors' => $errors,
            ], $status);
        });
    }
}
```

```php
<?php
// app/Http/Controllers/Api/UserController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Support\Facades\Response;

class UserController extends Controller
{
    public function show(User $user)
    {
        return Response::success('User retrieved successfully', $user);
    }

    public function store(Request $request)
    {
        $validator = \Validator::make($request->all(), [
            'email' => 'required|email|unique:users',
        ]);

        if ($validator->fails()) {
            return Response::error('Validation failed', $validator->errors()->toArray(), 422);
        }

        $user = User::create($request->all());
        return Response::success('User created successfully', $user, 201);
    }
}
```

**Expected Output:**
- `GET /api/user/1` → `{"success": true, "message": "User retrieved successfully", "data": {...}}`
- `POST /api/users` with invalid email → `{"success": false, "message": "Validation failed", "errors": {"email": ["..."]}}` with status 422.

**Why:** The `success` and `error` macros encapsulate the common response structure. Controllers use them to return consistent responses without repeating the JSON structure.

---

**Example 2: Paginated Response Macro**

```php
<?php
// app/Providers/AppServiceProvider.php

use Illuminate\Support\Facades\Response;

Response::macro('paginated', function ($paginator, string $message = 'Data retrieved') {
    return response()->json([
        'success' => true,
        'message' => $message,
        'data' => $paginator->items(),
        'meta' => [
            'current_page' => $paginator->currentPage(),
            'last_page' => $paginator->lastPage(),
            'per_page' => $paginator->perPage(),
            'total' => $paginator->total(),
        ],
        'links' => [
            'first' => $paginator->url(1),
            'last' => $paginator->url($paginator->lastPage()),
            'prev' => $paginator->previousPageUrl(),
            'next' => $paginator->nextPageUrl(),
        ],
    ]);
});
```

```php
<?php
// routes/api.php

use App\Models\User;
use Illuminate\Support\Facades\Response;
use Illuminate\Support\Facades\Route;

Route::get('/users', function () {
    $users = User::paginate(15);
    return Response::paginated($users, 'Users retrieved successfully');
});
```

**Expected Output:** `GET /api/users` returns a JSON response with `success`, `message`, `data`, `meta`, and `links` keys, all in a consistent structure.

**Why:** The `paginated` macro wraps the paginator's data and metadata into a custom response structure, providing a consistent format across all paginated endpoints.

---

**Example 3: Chaining Macros with Other Methods**

```php
<?php
// routes/api.php

use App\Models\User;
use Illuminate\Support\Facades\Response;
use Illuminate\Support\Facades\Route;

Route::get('/user/{user}', function (User $user) {
    // Chain the macro with additional response methods
    return Response::success('User found', $user)
        ->header('X-User-ID', $user->id)
        ->header('X-Cache-Status', 'HIT');
});
```

**Expected Output:** `GET /api/user/1` returns the success response with the custom headers `X-User-ID: 1` and `X-Cache-Status: HIT`.

**Why:** The `Response::success()` macro returns a `JsonResponse` instance, which can be chained with `header()` and other response methods, just like any other response.

### Real-World Cases

- **API Consistency:** Macros ensure all API responses follow the same `success`/`message`/`data` structure.
- **Error Handling:** An `error` macro standardises validation error responses across all endpoints.
- **Pagination:** A `paginated` macro standardises paginated responses.
- **Authentication:** A `token` macro returns JWT tokens in a consistent structure.
- **Webhook Responses:** A `webhook` macro acknowledges webhook receipt with a standard payload.

### References

- Laravel API: Response Macro — https://api.laravel.com/docs/12.x/Illuminate/Http/Response.html#method_macro
- Laravel Macros Documentation (Medium) — https://medium.com/@laravelprotips/a-guide-to-laravel-macros-59c8cde1e6c8
- Laracasts: Response Macros for Pagination — https://laracasts.com/discuss/channels/laravel/response-macros-for-pagination

---

## 5. SPA Adaptations (Inertia & Livewire)

### Definitions

**Core Definition:** SPA adaptations in Laravel are response mechanisms that bridge the backend framework with modern frontend frameworks—Inertia.js renders JavaScript page components with server-side data, while Livewire renders dynamic components with server-side hydration streams.

**Technical Definition:** Inertia.js uses `Inertia::render()` to return a response containing the JavaScript page component name and a props array. On initial page load, an HTML skeleton with a `data-page` attribute (containing the JSON-serialised page object) is returned; on subsequent navigations, a JSON response containing the page component and props is returned, which the Inertia client uses to swap components without a full page reload. Livewire uses server-side components that render to HTML and include a `wire:snapshot` attribute containing the component's state. On user interaction, an AJAX request is sent to the server with the snapshot; Livewire "hydrates" the component from the snapshot, executes the method, "dehydrates" the new state, and returns the updated HTML and snapshot.

**Beginner-Friendly Explanation:** Building a single-page application (SPA) with React or Vue normally means creating a separate API and dealing with complicated data hydration. Inertia.js lets you use Laravel routes and controllers, but instead of returning Blade views, you return JavaScript page components with data. Livewire takes a different approach: you write PHP components that render to HTML, and Livewire handles the JavaScript communication behind the scenes, updating only the parts of the page that change.

### Purposes

- To manage reactive frontend components using `Inertia::render()` data boundaries.
- To provide partial hydration streams via Livewire's snapshot-based hydration system.
- To build modern SPAs without creating a separate API.
- To leverage Laravel's routing, controllers, and middleware for SPA navigation.
- To maintain a single code repository for both backend and frontend logic.
- To enable server-side rendering (SSR) with Inertia for improved SEO and initial load performance.

### Syntax Rules and Structure

**Complete General Syntax — Inertia::render():**

```php
use Inertia\Inertia;

return Inertia::render('Event/Show', [
    'event' => $event->only('id', 'title', 'start_date', 'description'),
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Inertia::render()` | Returns an Inertia response. |
| `'Event/Show'` | The JavaScript page component path. |
| `['event' => ...]` | The props (data) passed to the component. |

**Complete General Syntax — Inertia Helper:**

```php
return inertia('Event/Show', [
    'event' => $event,
]);
```

**Complete General Syntax — Livewire Hydration:**

```php
// Livewire component
class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            Count: {{ $count }}
            <button wire:click="increment">+</button>
        </div>
        HTML;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `public $count = 1` | Public property stored in the snapshot. |
| `wire:click="increment"` | Livewire directive that triggers an AJAX request. |
| Snapshot | JSON containing `state` and `memo` stored in `wire:snapshot` attribute. |

**Syntax Rules:**

- Inertia pages are stored in `resources/js/Pages` and correspond to Vue, React, or Svelte components.
- Inertia props are automatically converted from Laravel types (models, collections, API resources) to JSON.
- Livewire components are PHP classes that extend `Livewire\Component`.
- Livewire snapshots are stored in the `wire:snapshot` HTML attribute and are JSON-serialised.
- Inertia uses a `data-page` attribute on the root element for initial hydration.
- Livewire uses `wire:id` and `wire:snapshot` attributes for component identification and state.

**Constraints and Limitations:**

- **Inertia:** Requires a JavaScript build pipeline (Vite, Webpack) and a frontend framework (Vue, React, Svelte).
- **Inertia:** All data returned from controllers is visible client-side—omit sensitive information.
- **Livewire:** Each interaction requires a server round-trip, which can introduce latency compared to pure JavaScript interactions.
- **Livewire:** Snapshots can become large for components with many public properties, increasing payload size.
- **Version compatibility:** Inertia.js v2 and v3 have different APIs; check the version you are using.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Inertia Response**

```php
<?php
// app/Http/Controllers/EventController.php

namespace App\Http\Controllers;

use App\Models\Event;
use Inertia\Inertia;

class EventController extends Controller
{
    public function show(Event $event)
    {
        // Return an Inertia page component with event data
        return Inertia::render('Event/Show', [
            'event' => $event->only('id', 'title', 'start_date', 'description'),
        ]);
    }
}
```

```vue
<!-- resources/js/Pages/Event/Show.vue -->
<script setup>
const props = defineProps(['event']);
</script>

<template>
    <h1>{{ event.title }}</h1>
    <p>{{ event.description }}</p>
</template>
```

**Expected Output:** `GET /events/1` returns an Inertia response. On initial load, an HTML skeleton is returned; the Inertia client hydrates the `Event/Show` component with the `event` prop. On subsequent navigation, a JSON response is returned and the component is swapped without a full page reload.

**Why:** `Inertia::render()` returns a response containing the page component name and props. The Inertia client handles the hydration and component swapping on the frontend.

---

**Example 2: Livewire Component with Hydration**

```php
<?php
// app/Livewire/Counter.php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

```blade
{{-- resources/views/livewire/counter.blade.php --}}
<div>
    <h1>Count: {{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

**Expected Output:** The initial page load renders `<div wire:id="..." wire:snapshot="{...}">` with `Count: 1`. Clicking the "+" button sends an AJAX request with the snapshot, Livewire hydrates the component, increments the count, dehydrates the new state, and returns the updated HTML with `Count: 2`.

**Why:** Livewire stores the component's state in a JSON snapshot embedded in the HTML. On interaction, the snapshot is sent to the server, the component is re-created ("hydrated"), the method is executed, and the updated state is returned ("dehydrated").

---

**Example 3: Inertia with API Resources**

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
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/DashboardController.php

namespace App\Http\Controllers;

use App\Http\Resources\UserResource;
use App\Models\User;
use Inertia\Inertia;

class DashboardController extends Controller
{
    public function index()
    {
        return Inertia::render('Dashboard', [
            'user' => new UserResource(auth()->user()),
            'users' => UserResource::collection(User::paginate(10)),
        ]);
    }
}
```

**Expected Output:** The `Dashboard` page component receives the authenticated user and a paginated collection of users, both transformed by the `UserResource` class.

**Why:** Inertia automatically resolves API Resources and paginators, converting them to JSON props for the page component. The `data` wrapper from the resource collection is preserved in the props.

### Real-World Cases

- **SaaS Dashboards:** Inertia renders dashboard components with data from Laravel controllers.
- **CRUD Applications:** Livewire handles form submissions and table updates with minimal JavaScript.
- **Real-Time Feeds:** Livewire's polling and event system updates feeds without page reloads.
- **Multi-Step Forms:** Inertia preserves form state across page navigations.
- **Admin Panels:** Both Inertia and Livewire are used in Laravel starter kits (Breeze, Jetstream) for admin interfaces.

### References

- Inertia.js Documentation: Responses — https://inertiajs.com/docs/v2/the-basics/responses
- Laravel Livewire Documentation: Hydration — https://livewire.laravel.com/docs/3.x/hydration
- Laravel Frontend Documentation — https://laravel.com/docs/12.x/frontend
- Inertia.js Protocol — https://inertiajs.com/the-protocol

---

## References

- Laravel HTTP Responses Documentation (12.x) — https://laravel.com/docs/12.x/responses
- Laravel HTTP Responses: Attaching Headers — https://laravel.com/docs/12.x/responses#attaching-headers-to-responses
- Laravel HTTP Responses: Attaching Cookies — https://laravel.com/docs/12.x/responses#attaching-cookies-to-responses
- Laravel Eloquent: API Resources — https://laravel.com/docs/12.x/eloquent-resources
- Laravel Eloquent: API Resources — Pagination — https://laravel.com/docs/12.x/eloquent-resources#pagination
- Laravel Database: Pagination — https://laravel.com/docs/12.x/pagination
- Laravel API: Response — https://api.laravel.com/docs/12.x/Illuminate/Http/Response.html
- Laravel API: JsonResource — https://api.laravel.com/docs/12.x/Illuminate/Http/Resources/Json/JsonResource.html
- Laravel API: ResourceCollection — https://api.laravel.com/docs/12.x/Illuminate/Http/Resources/Json/ResourceCollection.html
- Laravel API: LengthAwarePaginator — https://api.laravel.com/docs/12.x/Illuminate/Pagination/LengthAwarePaginator.html
- Inertia.js Documentation: Responses — https://inertiajs.com/docs/v2/the-basics/responses
- Laravel Livewire Documentation: Hydration — https://livewire.laravel.com/docs/3.x/hydration
- Laravel Frontend Documentation — https://laravel.com/docs/12.x/frontend
- Laravel API: Response Macro — https://api.laravel.com/docs/12.x/Illuminate/Http/Response.html#method_macro
- Laravel Macros Guide (Medium) — https://medium.com/@laravelprotips/a-guide-to-laravel-macros-59c8cde1e6c8
- Laracasts: Response Macros for Pagination — https://laracasts.com/discuss/channels/laravel/response-macros-for-pagination