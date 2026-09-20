# Laravel Route Model Binding: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Route Model Binding is a feature that automatically resolves Eloquent model instances from route parameters, eliminating the need for manual database queries inside route handlers or controller methods.

**Technical Definition:** Route model binding leverages Laravel's service container and the `Illuminate\Contracts\Routing\UrlRoutable` interface. During request dispatching, the `SubstituteBindings` middleware inspects the route action's signature for parameters that implement `UrlRoutable`. For implicit binding, the router matches the type-hinted model class with the corresponding URI segment by variable name, then calls `resolveRouteBinding()` on the model to retrieve the instance. For explicit binding, a resolver closure registered via `Route::model()` or `Route::bind()` is invoked instead. If resolution fails, the router generates a `ModelNotFoundException`, which Laravel converts into a 404 HTTP response by default.

**Beginner-Friendly Explanation:** Normally, if you want to fetch a user from the database based on an ID in the URL, you'd write `User::find($id)` inside your controller. Route model binding lets Laravel do that work for you automatically. If you type-hint `User $user` in your controller and your route has `{user}`, Laravel will fetch the matching user and hand it to you—or return a 404 if no such user exists.

### Key Characteristics

- **Convention-Based:** Implicit binding requires only a matching type-hint and variable name—no configuration needed.
- **Automatic 404 Generation:** Missing models trigger a `ModelNotFoundException`, automatically converted to a 404 response.
- **Customisable Resolution:** Keys, scoping, and missing-model behaviour can all be overridden.
- **Parent-Child Integrity:** Scoped bindings enforce that nested resources belong to their parent models.
- **Soft-Delete Aware:** Soft-deleted models are excluded by default, but can be included with `->withTrashed()`.
- **Cache Compatible:** Works with route caching since resolution happens at runtime, not definition time.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (route model binding is stable across these versions; `missing()` since 8.x, `scopeBindings()` since 8.x, `#[RouteKey]` attribute since Laravel 13.21).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- Eloquent models configured with primary keys and relationships.
- The `SubstituteBindings` middleware active in the route's middleware stack (included by default in `web` and `api` groups).

### Related Programming Areas

- **Eloquent ORM:** Route model binding is tightly coupled to Eloquent's query builder and relationship system.
- **Service Container:** Models are resolved through Laravel's IoC container for dependency injection.
- **Middleware Pipeline:** The `SubstituteBindings` middleware executes the binding resolution.
- **Exception Handling:** Missing models integrate with Laravel's exception handler for 404 responses.
- **Multi-Tenancy:** Scoped bindings enforce tenant isolation at the routing layer.

### Core Concepts / Features

1. Implicit Binding
2. Explicit Binding
3. Custom Keys & Scoping
4. Nested Bindings & Child Scoping
5. Missing-Model Customization

---

## 1. Implicit Binding

### Definitions

**Core Definition:** Implicit binding is Laravel's automatic mechanism for resolving Eloquent models based on the type-hint and variable name of a route parameter matching a URI segment.

**Technical Definition:** When the `SubstituteBindings` middleware processes a route, it reflects the route action's parameters. For each parameter whose type-hint is a subclass of `Illuminate\Database\Eloquent\Model` (and therefore implements `UrlRoutable`), it compares the parameter's variable name to the URI segment names. If a match is found, it calls `resolveRouteBinding($value)` on the model class, which by default queries the database using the model's primary key.

**Beginner-Friendly Explanation:** If your route is `/users/{user}` and your controller method is `show(User $user)`, Laravel notices that `{user}` and `$user` match, and that `$user` is type-hinted as the `User` model. It automatically fetches the user with that ID from the database and injects it into your method. If no user exists, Laravel shows a 404 page.

### Purposes

- To eliminate boilerplate `findOrFail()` calls from controllers and route closures.
- To centralise model resolution logic in a single, testable location.
- To automatically generate 404 responses when models are not found.
- To enable seamless dependency injection of fully hydrated model instances.
- To support rapid prototyping by reducing the code required for CRUD operations.

### Syntax Rules and Structure

**Complete General Syntax — Closure:**

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{user}` | The URI segment parameter. |
| `User $user` | Type-hinted parameter whose variable name matches the URI segment. |
| `$user->email` | The injected model instance, ready for use. |

**Complete General Syntax — Controller:**

```php
// Route definition
Route::get('/users/{user}', [UserController::class, 'show']);

// Controller method
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `User $user` | The type-hint and variable name that trigger binding. |
| `view(...)` | The resolved model is passed to the view. |

**Syntax Rules:**

- The variable name in the route parameter (`{user}`) must match the parameter name in the closure/controller method (`$user`).
- The type-hint must be a concrete Eloquent model class or an interface resolved by the container.
- The model must have a primary key column (default: `id`) unless customised.
- Binding resolution occurs automatically when the `SubstituteBindings` middleware is active.

**Constraints and Limitations:**

- **Name matching:** If `{user}` and `$user` do not match, implicit binding will not trigger—the raw string value is injected instead.
- **Soft deletes:** Models with `SoftDeletes` are excluded by default. Use `->withTrashed()` to include them.
- **Performance:** Each bound model issues a separate database query; eager loading is not automatically applied.
- **Order of resolution:** Parameters are resolved in the order they appear in the route action signature.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Implicit Binding in a Closure**

```php
<?php
// routes/web.php

use App\Models\User;
use Illuminate\Support\Facades\Route;

// {user} matches $user — implicit binding triggers
Route::get('/users/{user}', function (User $user) {
    // $user is a fully hydrated User model instance
    return "User: {$user->name} (Email: {$user->email})";
});
```

**Setup:** Create a `User` model and seed a record with `id = 1`, `name = 'Alice'`, `email = 'alice@example.com'`.

**Expected Output:** `GET /users/1` → `User: Alice (Email: alice@example.com)`

**Why:** The `SubstituteBindings` middleware detects that the `$user` parameter is type-hinted as `User` and that `{user}` matches. It queries `User::find(1)` and injects the result. `GET /users/9999` returns 404.

---

**Example 2: Implicit Binding in a Controller**

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Models\Post;

class PostController extends Controller
{
    /**
     * Display the specified post.
     */
    public function show(Post $post)
    {
        // $post is resolved automatically from the {post} URI segment
        return view('posts.show', compact('post'));
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

Route::get('/posts/{post}', [PostController::class, 'show']);
```

**Expected Output:** `GET /posts/5` renders `posts.show` with the `Post` model having ID 5. `GET /posts/abc` returns 404 if no post with key `abc` exists.

**Why:** The controller method's `Post $post` type-hint and variable name match the `{post}` URI segment. Laravel resolves the model before invoking the controller.

---

**Example 3: Soft-Deleted Models with `withTrashed()`**

```php
<?php
// routes/web.php

use App\Models\User;
use Illuminate\Support\Facades\Route;

// Include soft-deleted users in binding resolution
Route::get('/users/{user}', function (User $user) {
    return "User: {$user->name} (deleted: " . ($user->trashed() ? 'yes' : 'no') . ")";
})->withTrashed();
```

**Expected Output:**
- `GET /users/1` (active user) → `User: Alice (deleted: no)`
- `GET /users/2` (soft-deleted user) → `User: Bob (deleted: yes)`

**Why:** By default, implicit binding excludes soft-deleted models. The `->withTrashed()` modifier instructs the binding resolver to include them, calling `resolveSoftDeletableRouteBinding()` instead of `resolveRouteBinding()`.

### Real-World Cases

- **User Profiles:** `/users/{user}` resolves the user model for profile pages without any manual query.
- **Blog Posts:** `/posts/{post}` resolves the post model for detail views, automatically 404ing for missing posts.
- **E-commerce Products:** `/products/{product}` resolves the product model for product detail pages.
- **Admin Panels:** `/admin/users/{user}/edit` resolves the user model for editing forms.
- **API Resources:** `/api/v1/posts/{post}` resolves the post model for JSON API responses.

### References

- Laravel Routing: Implicit Binding — https://laravel.com/docs/12.x/routing#implicit-binding
- Laravel Routing: Soft Deleted Models — https://laravel.com/docs/12.x/routing#implicit-soft-deleted-models
- Laravel API: UrlRoutable Interface — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Routing/UrlRoutable.html

---

## 2. Explicit Binding

### Definitions

**Core Definition:** Explicit binding is the manual registration of a model association for a specific route parameter name, defined centrally in a service provider rather than relying on type-hint conventions.

**Technical Definition:** The `Route::model()` method registers a binding in the router's `$binders` array, mapping a parameter name (e.g., `'user'`) to a model class. When the `SubstituteBindings` middleware encounters a parameter with that name, it invokes the registered binder's `resolveRouteBinding()` method on the specified model class. The more flexible `Route::bind()` method accepts a closure that receives the URI segment value and returns the model instance, allowing arbitrary resolution logic.

**Beginner-Friendly Explanation:** Instead of relying on Laravel to guess which model a parameter corresponds to, you explicitly say "whenever you see `{user}` in any route, resolve it to a `User` model." This is useful when your parameter names don't match your model names, or when you need custom resolution logic (e.g., resolving by username instead of ID).

### Purposes

- To decouple route parameter names from model class names (e.g., `{article}` → `Post` model).
- To define custom resolution logic that cannot be expressed through implicit binding.
- To centralise binding configuration in a service provider for maintainability.
- To enforce consistent resolution across all routes sharing a parameter name.
- To resolve models from non-Eloquent sources (e.g., APIs, configuration files) via closures.

### Syntax Rules and Structure

**Complete General Syntax — `Route::model()`:**

```php
// In RouteServiceProvider::boot()
use App\Models\User;
use Illuminate\Support\Facades\Route;

public function boot(): void
{
    Route::model('user', User::class);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'user'` | The route parameter name (without braces). |
| `User::class` | The Eloquent model class to resolve. |

**Complete General Syntax — `Route::bind()`:**

```php
// In RouteServiceProvider::boot()
use App\Models\User;
use Illuminate\Support\Facades\Route;

public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('username', $value)->firstOrFail();
    });
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'user'` | The route parameter name. |
| `function (string $value)` | Closure receiving the raw URI segment value. |
| `firstOrFail()` | Throws `ModelNotFoundException` if no match—converted to 404. |

**Syntax Rules:**

- Bindings must be registered in the `boot()` method of `RouteServiceProvider` (Laravel 10 and earlier) or in `AppServiceProvider::boot()` (Laravel 11+).
- `Route::model()` binds a parameter name to a model class using the model's default resolution (primary key).
- `Route::bind()` accepts a closure for custom resolution logic.
- Explicit bindings take precedence over implicit binding for the same parameter name.
- Bindings are global—they apply to all routes using that parameter name.

**Constraints and Limitations:**

- **Global scope:** An explicit binding for `'user'` affects every route using `{user}`, which may be undesirable if different routes need different resolution logic.
- **Closure serialisation:** `Route::bind()` closures cannot be serialised for route caching. Use `Route::model()` or a dedicated resolver class for cache compatibility.
- **No per-route override:** You cannot override an explicit binding for a single route; you must use a different parameter name or custom resolution in the controller.
- **Laravel 11+:** The `RouteServiceProvider` is no longer included by default. Register bindings in `AppServiceProvider::boot()` or a dedicated service provider.

### Multiple Annotated Complete Code Examples

**Example 1: Explicit Binding with `Route::model()`**

```php
<?php
// app/Providers/AppServiceProvider.php (Laravel 11+)
// or app/Providers/RouteServiceProvider.php (Laravel 10)

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Bind every {user} parameter to the User model
        Route::model('user', User::class);
    }
}
```

```php
<?php
// routes/web.php

use App\Models\User;
use Illuminate\Support\Facades\Route;

// {user} is now resolved to a User model instance
Route::get('/users/{user}', function (User $user) {
    return "User: {$user->name}";
});
```

**Expected Output:** `GET /users/1` → `User: Alice`. `GET /users/9999` → 404.

**Why:** The `Route::model()` call registers a binder that resolves `{user}` to a `User` model using its primary key. Even if the variable name didn't match, the binding would still apply.

---

**Example 2: Explicit Binding with Custom Resolution Logic**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Resolve {user} by username instead of ID
        Route::bind('user', function (string $value) {
            return User::where('username', $value)->firstOrFail();
        });
    }
}
```

```php
<?php
// routes/web.php

use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::get('/users/{user}', function (User $user) {
    return "User: {$user->name} (username: {$user->username})";
});
```

**Expected Output:** `GET /users/alice` → `User: Alice (username: alice)`. `GET /users/1` → 404 unless a user with username `'1'` exists.

**Why:** The `Route::bind()` closure receives the raw URI value (`'alice'`) and queries the database by `username` instead of `id`. The `firstOrFail()` call ensures a 404 if no match is found.

---

**Example 3: Overriding `resolveRouteBinding()` on the Model**

```php
<?php
// app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * Retrieve the model for a bound value.
     * Resolve by slug instead of ID.
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('slug', $value)->firstOrFail();
    }
}
```

```php
<?php
// routes/web.php

use App\Models\Post;
use Illuminate\Support\Facades\Route;

Route::get('/posts/{post}', function (Post $post) {
    return $post->title;
});
```

**Expected Output:** `GET /posts/my-first-post` → returns the title of the post with slug `my-first-post`. `GET /posts/5` → 404 unless a post with slug `'5'` exists.

**Why:** Overriding `resolveRouteBinding()` on the model applies the custom resolution to **all** bindings for that model class—implicit and explicit—providing a single point of customisation.

### Real-World Cases

- **Legacy Systems:** Mapping old parameter names (`{userId}`) to new models (`User`) without changing route definitions.
- **Username-Based Profiles:** Resolving users by `username` globally via `Route::bind()`.
- **Multi-Source Resolution:** Resolving models from external APIs or configuration files via custom closures.
- **Model-Specific Logic:** Overriding `resolveRouteBinding()` on a model to enforce global query constraints (e.g., only published posts).

### References

- Laravel Routing: Explicit Binding — https://laravel.com/docs/12.x/routing#explicit-binding
- Laravel Routing: Customizing The Resolution Logic — https://laravel.com/docs/12.x/routing#customizing-the-resolution-logic
- Laravel API: UrlRoutable::resolveRouteBinding() — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Routing/UrlRoutable.html

---

## 3. Custom Keys & Scoping

### Definitions

**Core Definition:** Custom keys allow route model binding to resolve models using a database column other than the primary key (e.g., `slug`, `uuid`), specified either per-route or globally on the model.

**Technical Definition:** Laravel supports two mechanisms for custom keys: (1) inline specification via the route parameter syntax `{post:slug}`, which passes the column name as the `$field` argument to `resolveRouteBinding()`; (2) overriding the `getRouteKeyName()` method on the Eloquent model, which changes the default route key for all bindings of that model. When a custom key is used in a nested binding, Laravel automatically scopes the child query to the parent using relationship name conventions.

**Beginner-Friendly Explanation:** By default, Laravel looks up models by their `id` column. But sometimes you want URLs like `/posts/my-first-post` (using a slug) instead of `/posts/1`. You can tell Laravel to use the `slug` column by writing `{post:slug}` in your route, or by adding a `getRouteKeyName()` method to your model.

### Purposes

- To create SEO-friendly URLs using slugs instead of numeric IDs.
- To obscure sequential IDs for security through UUIDs or tokens.
- To resolve models by natural keys (e.g., username, email) without custom controllers.
- To maintain consistent lookup behaviour across all routes for a given model.
- To enable nested scoping where child models are resolved relative to their parent.

### Syntax Rules and Structure

**Complete General Syntax — Per-Route Custom Key:**

```php
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{post:slug}` | The `:slug` suffix specifies the column to use for resolution. |
| `Post $post` | Type-hinted parameter receives the resolved model. |

**Complete General Syntax — Global Custom Key (Method):**

```php
// In the Eloquent model
public function getRouteKeyName(): string
{
    return 'slug';
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `getRouteKeyName()` | Overrides the default route key (`id`). |
| `'slug'` | The column name to use for all bindings of this model. |

**Complete General Syntax — Global Custom Key (Attribute, Laravel 13.21+):**

```php
use Illuminate\Database\Eloquent\Attributes\RouteKey;

#[RouteKey('slug')]
class Post extends Model
{
    // ...
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `#[RouteKey('slug')]` | PHP attribute declaring the route key at the class level. |
| `class Post` | The Eloquent model class. |

**Syntax Rules:**

- The `:{column}` syntax is specified **inside** the route parameter braces: `{post:slug}`, not `{post}:slug`.
- The column must be unique and indexed for performance.
- When a custom key is used in a nested binding, Laravel automatically scopes the child query to the parent using the pluralised route parameter name as the relationship name.
- `getRouteKeyName()` applies globally to all bindings for that model; `{post:slug}` applies only to that route.
- The `#[RouteKey]` attribute (Laravel 13.21+) takes precedence over `getRouteKeyName()`.

**Constraints and Limitations:**

- **Uniqueness:** The custom key column must contain unique values, or `firstOrFail()` may throw a `ModelNotFoundException` or return an unexpected record.
- **Performance:** Ensure the custom key column is indexed. Unindexed lookups on large tables will be slow.
- **Scoping side effects:** Automatic scoping for custom-keyed nested bindings may produce unexpected queries if the parent model does not have a relationship matching the pluralised parameter name.
- **Laravel 13.21+:** The `#[RouteKey]` attribute is only available in Laravel 13.21 and later; earlier versions require `getRouteKeyName()`.

### Multiple Annotated Complete Code Examples

**Example 1: Per-Route Custom Key with Slug**

```php
<?php
// routes/web.php

use App\Models\Post;
use Illuminate\Support\Facades\Route;

// Resolve {post} by the slug column
Route::get('/posts/{post:slug}', function (Post $post) {
    return "Title: {$post->title} (slug: {$post->slug})";
});
```

**Setup:** Create a `Post` model with columns `id`, `slug`, `title`. Seed a record with `slug = 'my-first-post'`, `title = 'My First Post'`.

**Expected Output:** `GET /posts/my-first-post` → `Title: My First Post (slug: my-first-post)`. `GET /posts/1` → 404 (no post with slug `'1'`).

**Why:** The `:slug` suffix tells Laravel to pass `'slug'` as the `$field` argument to `resolveRouteBinding()`, which queries `where('slug', $value)` instead of `where('id', $value)`.

---

**Example 2: Global Custom Key with `getRouteKeyName()`**

```php
<?php
// app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * Use the slug column for route model binding.
     */
    public function getRouteKeyName(): string
    {
        return 'slug';
    }
}
```

```php
<?php
// routes/web.php

use App\Models\Post;
use Illuminate\Support\Facades\Route;

// {post} resolves by slug automatically
Route::get('/posts/{post}', function (Post $post) {
    return $post->title;
});
```

**Expected Output:** `GET /posts/my-first-post` → `My First Post`.

**Why:** The `getRouteKeyName()` override changes the default resolution column for all bindings of the `Post` model. Even without `:slug` in the route, Laravel uses the `slug` column.

---

**Example 3: Custom Key with Automatic Nested Scoping**

```php
<?php
// routes/web.php

use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Route;

// Nested binding with custom key — automatically scoped to parent
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return "User: {$user->name}, Post: {$post->title}";
});
```

**Setup:** Ensure the `User` model has a `posts()` relationship (hasMany). Seed a user with ID 1 and a post with slug `'hello'` belonging to that user.

**Expected Output:** `GET /users/1/posts/hello` → `User: Alice, Post: Hello`. `GET /users/2/posts/hello` → 404 if the post does not belong to user 2.

**Why:** When a custom key is used in a nested parameter, Laravel automatically scopes the child query using the pluralised parameter name (`posts`) as the relationship name on the parent. It calls `$user->posts()->where('slug', 'hello')->firstOrFail()`.

### Real-World Cases

- **Blog Platforms:** `/posts/{post:slug}` creates SEO-friendly URLs like `/posts/laravel-routing-guide`.
- **E-commerce:** `/products/{product:sku}` resolves products by stock-keeping unit.
- **User Profiles:** `/users/{user:username}` uses usernames instead of IDs in profile URLs.
- **API Resources:** `/api/users/{user:uuid}` uses UUIDs for external API consumers, obscuring internal IDs.
- **Multi-Tenant SaaS:** `/tenants/{tenant:subdomain}/dashboard` resolves tenants by subdomain.

### References

- Laravel Routing: Customizing the Key — https://laravel.com/docs/12.x/routing#customizing-the-key
- Laravel Routing: Custom Keys and Scoping — https://laravel.com/docs/12.x/routing#custom-keys-and-scoping
- Laravel News: RouteKey Model Attribute in Laravel 13.21 — https://laravel-news.com/route-key-model-attribute
- Laravel API: getRouteKeyName() — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Model.html

---

## 4. Nested Bindings & Child Scoping

### Definitions

**Core Definition:** Nested bindings resolve multiple models from a single route, with the `scopeBindings()` modifier enforcing that child models belong to their parent models.

**Technical Definition:** When a route contains multiple model-bound parameters (e.g., `{user}` and `{post}`), Laravel resolves each independently by default. The `scopeBindings()` method (or `Route::scopeBindings()->group(...)`) sets the `scope_bindings` attribute on the route, causing the `SubstituteBindings` middleware to call `resolveChildRouteBinding()` on the parent model instead of resolving the child globally. This method uses the pluralised child parameter name as the relationship name to constrain the query.

**Beginner-Friendly Explanation:** If you have a route like `/users/{user}/posts/{post}`, you want to make sure the post actually belongs to that user. Without scoping, Laravel would fetch any post with that ID, even if it belongs to a different user. With `->scopeBindings()`, Laravel checks the relationship and returns a 404 if the post doesn't belong to the user.

### Purposes

- To enforce parent-child data integrity at the routing layer, preventing cross-tenant data access.
- To eliminate manual ownership checks from controller actions.
- To provide automatic 404 responses for invalid nested resource URLs.
- To simplify nested resource controllers by removing repetitive `whereBelongsTo()` queries.
- To support multi-level nesting (e.g., user → post → comment) with automatic scoping at each level.

### Syntax Rules and Structure

**Complete General Syntax — Per-Route Scoping:**

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{user}` | Parent parameter. |
| `{post}` | Child parameter. |
| `->scopeBindings()` | Enables parent-child scoping for this route. |

**Complete General Syntax — Group Scoping:**

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });

    Route::get('/users/{user}/posts/{post}/comments/{comment}', function (
        User $user, Post $post, Comment $comment
    ) {
        return $comment;
    });
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Route::scopeBindings()->group(...)` | Applies scoping to all routes in the group. |
| Multiple nested parameters | Each level is scoped to its parent. |

**Complete General Syntax — Disabling Scoping:**

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->withoutScopedBindings()` | Explicitly disables scoping for this route, even if the group has scoping enabled. |

**Syntax Rules:**

- `scopeBindings()` is available as a route method and a group method (Laravel 8+).
- Scoping requires the parent model to have a relationship whose name matches the pluralised child parameter name (e.g., `{post}` → `posts` relationship).
- The parent parameter must be resolved **before** the child parameter for scoping to work.
- `withoutScopedBindings()` overrides group-level scoping for individual routes.
- Custom-keyed nested bindings (e.g., `{post:slug}`) are **automatically scoped** even without `scopeBindings()`.

**Constraints and Limitations:**

- **Relationship naming:** The parent model must have a relationship named after the plural of the child parameter (e.g., `posts` for `{post}`). Override via `resolveChildRouteBinding()` if the relationship name differs.
- **Order dependency:** If the parent parameter is resolved after the child, scoping will fail.
- **Performance:** Scoping adds a `whereHas` or relationship constraint to the child query. Ensure appropriate database indexes exist.
- **Polymorphic relationships:** Scoping with polymorphic relationships requires custom `resolveChildRouteBinding()` logic.
- **Laravel version:** `scopeBindings()` was introduced in Laravel 8.0.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Parent-Child Scoping**

```php
<?php
// routes/web.php

use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Route;

// Scope {post} to {user}
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return "User: {$user->name}, Post: {$post->title}";
})->scopeBindings();
```

**Setup:** Seed two users, each with one post. User 1 (Alice) has post ID 1; user 2 (Bob) has post ID 2.

**Expected Output:**
- `GET /users/1/posts/1` → `User: Alice, Post: Alice's Post`
- `GET /users/2/posts/1` → 404 (post 1 does not belong to user 2)

**Why:** The `scopeBindings()` modifier causes Laravel to resolve `{post}` through the `$user->posts()` relationship instead of globally. If the post does not belong to the user, `firstOrFail()` throws a `ModelNotFoundException`.

---

**Example 2: Multi-Level Nested Scoping with Group**

```php
<?php
// routes/web.php

use App\Models\Comment;
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Route;

// Apply scoping to an entire group of nested routes
Route::scopeBindings()->group(function () {
    // Two-level nesting: user → post
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post->title;
    });

    // Three-level nesting: user → post → comment
    Route::get('/users/{user}/posts/{post}/comments/{comment}', function (
        User $user, Post $post, Comment $comment
    ) {
        return $comment->body;
    });
});
```

**Expected Output:**
- `GET /users/1/posts/1/comments/5` → returns comment body if comment 5 belongs to post 1, which belongs to user 1.
- `GET /users/2/posts/1/comments/5` → 404 if any link in the chain is invalid.

**Why:** The group-level `scopeBindings()` applies to all routes within it. Each nested parameter is resolved relative to its immediate parent, enforcing the full chain of ownership.

---

**Example 3: Custom Relationship Name with `resolveChildRouteBinding()`**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Resolve the child binding with a custom relationship name.
     */
    public function resolveChildRouteBinding($childType, $value, $field)
    {
        // Use 'articles' relationship instead of default 'posts'
        if ($childType === 'post') {
            return $this->articles()->where($field ?? 'id', $value)->firstOrFail();
        }

        return parent::resolveChildRouteBinding($childType, $value, $field);
    }

    public function articles()
    {
        return $this->hasMany(Post::class);
    }
}
```

```php
<?php
// routes/web.php

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post->title;
})->scopeBindings();
```

**Expected Output:** `GET /users/1/posts/5` resolves post 5 through the `articles()` relationship instead of the default `posts()` relationship.

**Why:** Overriding `resolveChildRouteBinding()` on the parent model allows you to specify a custom relationship name when the conventional pluralised parameter name does not match the actual relationship method.

### Real-World Cases

- **Multi-Tenant SaaS:** `/tenants/{tenant}/users/{user}/posts/{post}` ensures posts are only accessible within the correct tenant.
- **E-commerce Order Management:** `/customers/{customer}/orders/{order}` prevents accessing orders belonging to other customers.
- **Project Management:** `/projects/{project}/tasks/{task}` enforces that tasks belong to the specified project.
- **Social Media:** `/users/{user}/posts/{post}/comments/{comment}` ensures comments are nested correctly under their post and author.
- **Educational Platforms:** `/courses/{course}/lessons/{lesson}` prevents accessing lessons from other courses.

### References

- Laravel Routing: Custom Keys and Scoping — https://laravel.com/docs/12.x/routing#custom-keys-and-scoping
- Laravel Routing: Scoped Bindings — https://laravel.com/docs/12.x/routing#scoped-bindings
- Laravel News: Scoped Route Binding for Nested Resource Management — https://laravel-news.com/scoped-route-binding
- Laravel API: resolveChildRouteBinding() — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Model.html

---

## 5. Missing-Model Customization

### Definitions

**Core Definition:** Missing-model customization allows developers to override the default 404 response generated when an implicitly bound model cannot be found, using the `->missing()` closure modifier.

**Technical Definition:** The `missing()` method on a `Route` instance registers a closure in the route's `missing` action. During binding resolution, if `resolveRouteBinding()` returns `null` or throws a `ModelNotFoundException`, the `SubstituteBindings` middleware checks for a registered missing handler. If present, it invokes the closure with the current `Request` instance and uses the returned response instead of the default 404. The closure can return a redirect, a custom view, a JSON response, or any `Response` instance.

**Beginner-Friendly Explanation:** Normally, if someone visits `/users/9999` and no such user exists, Laravel shows a generic 404 page. With `->missing()`, you can instead redirect them to the user list, show a custom "User not found" page, or return a JSON error—all without writing any controller code.

### Purposes

- To provide custom 404 experiences that match the application's design and user flow.
- To redirect missing resources to relevant index pages instead of showing a dead end.
- To return structured JSON errors for API clients when a bound model is missing.
- To log missing-model events for analytics and monitoring.
- To implement soft-delete recovery flows (e.g., redirecting to a restore page).
- To maintain consistent error handling across all model-bound routes.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
use Illuminate\Support\Facades\Route;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->missing(function (Request $request) { ... })` | Registers the missing-model handler. |
| `Request $request` | The current request instance, providing context (URL, headers). |
| `Redirect::route('locations.index')` | The custom response returned instead of 404. |

**Complete General Syntax — JSON API Response:**

```php
Route::get('/api/posts/{post}', function (Post $post) {
    return $post;
})->missing(function (Request $request) {
    return response()->json([
        'error' => 'Post not found',
        'path'  => $request->path(),
    ], 404);
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `response()->json(...)` | Returns a structured JSON response with HTTP 404. |
| `$request->path()` | Includes the requested path for debugging. |

**Syntax Rules:**

- The `missing()` method is called on the route definition, after the action and any other modifiers.
- The closure receives the `Request` instance as its only argument.
- The closure must return a `Response` instance (view, redirect, JSON, etc.).
- The HTTP status code is determined by the returned response—if you return a view, you must explicitly set the status code to 404 if desired.
- The `missing()` method works with both implicit and explicit bindings.

**Constraints and Limitations:**

- **Single handler:** Only one `missing()` handler can be registered per route. Calling it multiple times overwrites the previous handler.
- **No route parameters:** The closure does not receive the unresolved parameter value directly—use `$request->route()` to access it.
- **Serialisation:** The closure cannot be serialised for route caching. Use a controller-based approach for cache compatibility.
- **Implicit binding only:** The `missing()` method applies to binding resolution failures. It does not handle 404s from non-binding sources (e.g., manual `abort(404)`).

### Multiple Annotated Complete Code Examples

**Example 1: Redirect to Index on Missing Model**

```php
<?php
// routes/web.php

use App\Http\Controllers\LocationController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
use Illuminate\Support\Facades\Route;

Route::get('/locations/{location:slug}', [LocationController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        // Redirect to the locations index instead of showing 404
        return Redirect::route('locations.index')
            ->with('error', 'Location not found.');
    });

// Define the index route
Route::get('/locations', [LocationController::class, 'index'])
    ->name('locations.index');
```

**Expected Output:** `GET /locations/nonexistent` → 302 redirect to `/locations` with a flash message `'Location not found.'`

**Why:** When the `{location:slug}` binding fails, the `missing()` closure is invoked. It returns a `RedirectResponse`, which Laravel uses instead of the default 404. The flash message is available on the next request.

---

**Example 2: JSON Response for API Clients**

```php
<?php
// routes/api.php

use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/posts/{post}', function (Post $post) {
    return $post;
})->missing(function (Request $request) {
    // Return structured JSON error for API consumers
    return response()->json([
        'error'   => 'Resource not found',
        'message' => "Post with identifier '{$request->route('post')}' does not exist.",
        'path'    => $request->path(),
    ], 404);
});
```

**Expected Output:** `GET /api/posts/9999` → HTTP 404 with body:
```json
{
    "error": "Resource not found",
    "message": "Post with identifier '9999' does not exist.",
    "path": "api/posts/9999"
}
```

**Why:** The `missing()` closure returns a `JsonResponse` with a custom message and the unresolved parameter value (accessed via `$request->route('post')`). This provides API consumers with actionable error information.

---

**Example 3: Custom 404 View with Logging**

```php
<?php
// routes/web.php

use App\Models\Article;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/articles/{article:slug}', function (Article $article) {
    return view('articles.show', compact('article'));
})->missing(function (Request $request) {
    // Log the missing article for analytics
    Log::info('Article not found', [
        'slug'      => $request->route('article'),
        'url'       => $request->fullUrl(),
        'referer'   => $request->header('referer'),
        'user_agent'=> $request->userAgent(),
    ]);

    // Return a custom 404 view with the missing slug
    return response()->view('errors.article-not-found', [
        'slug' => $request->route('article'),
    ], 404);
});
```

**Expected Output:** `GET /articles/missing-article` → HTTP 404 with the `errors.article-not-found` view, and a log entry recording the missing slug, URL, referer, and user agent.

**Why:** The `missing()` closure performs logging for analytics and returns a custom view with HTTP 404 status. This provides a branded 404 experience while collecting data for content strategy.

### Real-World Cases

- **Blog Platforms:** Redirect missing article slugs to the blog index with a "Article not found" flash message.
- **E-commerce:** Return a JSON 404 for missing product APIs while redirecting browser users to a search page.
- **Documentation Sites:** Log missing page requests to identify broken internal links and update them.
- **Multi-Tenant SaaS:** Redirect missing tenant subdomains to a "Tenant not found" landing page with a signup call-to-action.
- **Legacy URL Migration:** Check a legacy URL mapping table in the `missing()` closure and 301-redirect to the new URL if a match is found.

### References

- Laravel Routing: Customizing Missing Model Behavior — https://laravel.com/docs/12.x/routing#customizing-missing-model-behavior
- Laravel API: Route::missing() — https://api.laravel.com/docs/12.x/Illuminate/Routing/Route.html
- Laravel Error Handling — https://laravel.com/docs/12.x/errors

---

## References

- Laravel Routing Documentation (12.x) — https://laravel.com/docs/12.x/routing
- Laravel Routing: Route Model Binding — https://laravel.com/docs/12.x/routing#route-model-binding
- Laravel Routing: Implicit Binding — https://laravel.com/docs/12.x/routing#implicit-binding
- Laravel Routing: Explicit Binding — https://laravel.com/docs/12.x/routing#explicit-binding
- Laravel Routing: Customizing the Key — https://laravel.com/docs/12.x/routing#customizing-the-key
- Laravel Routing: Custom Keys and Scoping — https://laravel.com/docs/12.x/routing#custom-keys-and-scoping
- Laravel Routing: Customizing Missing Model Behavior — https://laravel.com/docs/12.x/routing#customizing-missing-model-behavior
- Laravel Routing: Scoped Bindings — https://laravel.com/docs/12.x/routing#scoped-bindings
- Laravel API: UrlRoutable Interface — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Routing/UrlRoutable.html
- Laravel API: resolveRouteBinding() — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Model.html#method_resolveRouteBinding
- Laravel API: resolveChildRouteBinding() — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Model.html#method_resolveChildRouteBinding
- Laravel API: Route::missing() — https://api.laravel.com/docs/12.x/Illuminate/Routing/Route.html
- Laravel News: RouteKey Model Attribute in Laravel 13.21 — https://laravel-news.com/route-key-model-attribute
- Laravel News: Scoped Route Binding for Nested Resource Management — https://laravel-news.com/scoped-route-binding
- Laravel News: Customizing Route Model Binding Keys — https://laravel-news.com/customizing-route-model-binding-keys
- Laravel Daily: Route Model Binding Tips — https://laraveldaily.com/post/laravel-route-model-binding-tips
- Expressive Wendell Adriel: Laravel Route Binding Behind the Curtains — https://expressive.wendelladriel.com/laravel-route-binding-behind-the-curtains
- Laravel Error Handling — https://laravel.com/docs/12.x/errors