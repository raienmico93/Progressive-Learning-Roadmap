# Laravel Route Model Binding: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Route model binding is a Laravel feature that automatically resolves Eloquent model instances from route parameters, eliminating the need for manual database queries in controllers and closures.

**Technical Definition**

Route model binding is implemented by the `ImplicitRouteBinding` and `RouteBinding` classes within Laravel's routing system. When a route parameter name matches a type-hinted variable in the route's closure or controller method, the router automatically queries the model's table for a record with a matching key. If no record is found, a 404 HTTP response is generated. The system supports implicit binding (convention-based), explicit binding (manually defined), custom keys (non-primary-key columns), scoped bindings (parent-child enforcement), and soft-delete handling.

**Beginner-Friendly Explanation**

Route model binding is a shortcut that turns a URL parameter like `/users/5` into a full `User` model object automatically. Instead of writing `$user = User::findOrFail(5)` in every controller, you just type-hint `User $user` in the method signature, and Laravel does the lookup for you. If the user doesn't exist, Laravel automatically returns a 404 page.

---

### Key Characteristics

- **Automatic Resolution**: Models are resolved from route parameters without manual queries.
- **Convention Over Configuration**: Implicit binding works when the parameter name matches the type-hinted variable name.
- **Custom Keys**: Models can be resolved by columns other than the primary key (e.g., slug, UUID).
- **Scoped Bindings**: Nested resources can be automatically scoped to their parent (e.g., a post must belong to a user).
- **Soft Delete Awareness**: Soft-deleted models can be included via `withTrashed()`.
- **Automatic 404 Handling**: Missing models automatically produce a 404 response.
- **Customizable Resolution**: Explicit bindings and `resolveRouteBinding()` allow full control over how models are retrieved.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Eloquent models** defined in `app/Models`.
- **Routes** defined in `routes/web.php` or `routes/api.php`.
- **Basic knowledge of Eloquent** and routing.

---

### Related Programming Areas

- **Eloquent Models**: Route model binding resolves instances of Eloquent models.
- **Controllers**: Binding is most commonly used in controller methods.
- **Middleware**: Scoped bindings can enforce tenant isolation.
- **Resource Controllers**: Route model binding works seamlessly with resourceful routing.
- **Authorization**: Bound models are used in policies and gates for permission checks.

---

## Core Concepts

---

### 1. Implicit Binding

#### Definitions

**Core Definition**

Implicit binding is the automatic resolution of Eloquent models from route parameters based on the parameter name and the type-hint in the route or controller method.

**Technical Definition**

When a route parameter name (e.g., `{user}`) matches a type-hinted variable name (e.g., `User $user`) in the route's closure or controller method, Laravel's `ImplicitRouteBinding` class queries the model's table using the model's `getRouteKeyName()` (default: `id`). If a matching record is found, it is injected; otherwise, a `ModelNotFoundException` is thrown, resulting in a 404 response.

**Beginner-Friendly Explanation**

Implicit binding is the "magic" that turns `{user}` into a `User` object automatically. You just type-hint the model in your controller, and Laravel does the rest.

#### Purposes

- To eliminate manual `findOrFail()` calls in controllers.
- To automatically return 404 responses for missing models.
- To simplify controller code and improve readability.
- To enforce consistent model resolution across the application.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});

// Controller method
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `{user}` | Route parameter name. |
| `User $user` | Type-hinted variable matching the parameter name. |
| `getRouteKeyName()` | Model method returning the column used for resolution (default: `id`). |

**Syntax Rules**

- The variable name must match the route parameter name exactly (camelCase).
- The type-hint must be an Eloquent model class.
- If no matching model is found, a 404 response is generated automatically.
- Implicit binding works in both closures and controller methods.

**Constraints and Limitations**

- The parameter name and variable name must match exactly.
- Implicit binding does not work if the parameter name is not a valid identifier.
- Soft-deleted models are not retrieved by default.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Implicit Binding**

**Setup Guide**

1. Define a route with a `{user}` parameter.
2. Type-hint `User $user` in the closure.

**Code**

```php
// routes/web.php
use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

**Expected Output**

For a request to `/users/1` where user ID 1 exists with email `john@example.com`:

```
john@example.com
```

**Why This Output Occurs**

The `{user}` parameter matches the `$user` variable. Laravel queries `User::find(1)` and injects the model. The closure returns the user's email.

---

**Example 2: Implicit Binding in a Controller**

**Code**

```php
// routes/web.php
use App\Http\Controllers\UserController;

Route::get('/users/{user}', [UserController::class, 'show']);
```

```php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    public function show(User $user)
    {
        return view('user.profile', ['user' => $user]);
    }
}
```

**Expected Output**

The `user.profile` view is rendered with the resolved `User` model.

**Why This Output Occurs**

The controller method type-hints `User $user`, and the route parameter `{user}` matches. Laravel resolves the model automatically.

---

#### Real-World Cases with Explanation

**Case 1: User Profile Pages**

A social media application uses `/users/{user}` routes to display user profiles. The controller type-hints `User $user` and receives the model directly.

**Explanation**: Implicit binding eliminates boilerplate and ensures 404s for missing users.

---

### 2. Custom Keys

#### Definitions

**Core Definition**

Custom keys allow Eloquent models to be resolved by a column other than the primary key (e.g., `slug`, `uuid`, `username`).

**Technical Definition**

Custom keys can be specified per-route using the syntax `{post:slug}` or globally on the model by overriding `getRouteKeyName()` or applying the `#[RouteKey]` attribute. The router uses the specified column to query the model's table.

**Beginner-Friendly Explanation**

Custom keys let you use a nice-looking URL like `/posts/my-first-post` instead of `/posts/1`. You tell Laravel to look up the post by its slug instead of its ID.

#### Purposes

- To create SEO-friendly URLs (e.g., `/posts/my-first-post`).
- To use UUIDs or other non-integer identifiers.
- To decouple URLs from database primary keys.
- To provide human-readable URLs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Per-route custom key
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});

// Global custom key (method override)
public function getRouteKeyName(): string
{
    return 'slug';
}

// Global custom key (attribute — Laravel 13.21+)
use Illuminate\Database\Eloquent\Attributes\RouteKey;

#[RouteKey('slug')]
class Post extends Model
{
    // ...
}
```

**Component Breakdown**

| Syntax | Description |
|--------|-------------|
| `{post:slug}` | Per-route custom key. |
| `getRouteKeyName()` | Model method override. |
| `#[RouteKey('slug')]` | PHP attribute (Laravel 13.21+). |

**Syntax Rules**

- The column must exist in the model's table.
- Custom keys work with both implicit and explicit binding.
- The column should have a unique index for performance.

**Constraints and Limitations**

- The custom key column must be unique.
- Soft-deleted models are still excluded unless `withTrashed()` is used.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Per-Route Custom Key**

**Code**

```php
// routes/web.php
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post->title;
});
```

**Expected Output**

For a request to `/posts/my-first-post` where a post with slug `my-first-post` exists:

```
My First Post
```

**Why This Output Occurs**

The `{post:slug}` syntax instructs Laravel to resolve the `Post` model by the `slug` column instead of `id`.

---

**Example 2: Global Custom Key with `getRouteKeyName()`**

**Model**

```php
// app/Models/Post.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    public function getRouteKeyName(): string
    {
        return 'slug';
    }
}
```

**Usage**

```php
Route::get('/posts/{post}', function (Post $post) {
    return $post->title;
});
```

**Expected Output**

The same as the per-route example, but all routes using `{post}` will use the slug.

**Why This Output Occurs**

The `getRouteKeyName()` method overrides the default key for all route bindings involving this model.

---

**Example 3: Global Custom Key with `#[RouteKey]` Attribute (Laravel 13.21+)**

**Model**

```php
<?php
// app/Models/Post.php
namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\RouteKey;
use Illuminate\Database\Eloquent\Model;

#[RouteKey('slug')]
class Post extends Model
{
    // ...
}
```

**Expected Output**

All route bindings for `Post` use the `slug` column.

**Why This Output Occurs**

The `#[RouteKey]` attribute provides a declarative way to customize the route key, following the pattern of `#[ObservedBy]` and `#[ScopedBy]`.

---

#### Real-World Cases with Explanation

**Case 1: Blog Posts with Slugs**

A blog uses `/posts/{post:slug}` for SEO-friendly URLs. Readers can share links like `/posts/laravel-routing-guide`.

**Explanation**: Custom keys make URLs readable and shareable.

---

### 3. Explicit Binding

#### Definitions

**Core Definition**

Explicit binding is the manual definition of how a route parameter should be resolved to a model instance, overriding Laravel's convention-based implicit binding.

**Technical Definition**

Explicit binding is registered via `Route::model('param', Model::class)` or `Route::bind('param', $callback)` in a service provider's `boot()` method. The `model` method specifies the class, while `bind` accepts a closure for custom resolution logic. The closure receives the URI segment value and must return a model instance or throw an exception.

**Beginner-Friendly Explanation**

Explicit binding lets you define your own rules for how a route parameter becomes a model. You can use a custom column or complex logic instead of relying on the default ID lookup.

#### Purposes

- To customise model resolution logic.
- To resolve models by columns other than the primary key.
- To handle complex resolution scenarios (e.g., multi-tenant, composite keys).
- To decouple route parameters from model classes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// In a service provider's boot() method
use App\Models\User;
use Illuminate\Support\Facades\Route;

Route::model('user', User::class);

Route::bind('user', function ($value) {
    return User::where('name', $value)->firstOrFail();
});
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Route::model()` | Binds a parameter to a model class. |
| `Route::bind()` | Binds a parameter to a custom closure. |
| `resolveRouteBinding()` | Model method for custom resolution. |

**Syntax Rules**

- Explicit bindings are registered in a service provider's `boot()` method.
- The closure receives the URI segment value.
- The closure must return a model instance or throw an exception.
- Explicit bindings override implicit binding for the same parameter.

**Constraints and Limitations**

- Explicit bindings are global; they apply to all routes using that parameter name.
- The closure must return a model or throw `ModelNotFoundException`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Explicit Binding with `Route::model()`**

**Service Provider**

```php
// app/Providers/RouteServiceProvider.php
public function boot(): void
{
    Route::model('user', User::class);
}
```

**Route**

```php
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

**Expected Output**

For `/users/1`, the `User` model with ID 1 is injected.

**Why This Output Occurs**

`Route::model('user', User::class)` tells Laravel that `{user}` should resolve to a `User` model by primary key.

---

**Example 2: Explicit Binding with Custom Logic**

**Service Provider**

```php
Route::bind('user', function ($value) {
    return User::where('username', $value)->firstOrFail();
});
```

**Route**

```php
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

**Expected Output**

For `/users/john`, the `User` with username `john` is injected.

**Why This Output Occurs**

The `Route::bind` closure resolves the user by the `username` column instead of the primary key.

---

**Example 3: Overriding `resolveRouteBinding()` on the Model**

**Model**

```php
// app/Models/User.php
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

**Expected Output**

All implicit bindings for `User` use the `name` column.

**Why This Output Occurs**

Overriding `resolveRouteBinding()` changes the resolution logic for all route bindings involving this model.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant Application**

A multi-tenant application resolves tenants by subdomain. An explicit binding uses the subdomain to look up the tenant model.

**Explanation**: Explicit binding handles complex resolution logic that implicit binding cannot.

---

### 4. Route Model Binding with Relationships

#### Definitions

**Core Definition**

Route model binding with relationships allows nested route parameters to be resolved as child models of a parent model, ensuring that the child belongs to the parent.

**Technical Definition**

When a route contains multiple model parameters (e.g., `/users/{user}/posts/{post}`), Laravel can automatically scope the child binding to the parent. This is called "scoped binding." The child model is resolved via the parent's relationship method, which is guessed from the parameter name (e.g., `posts` for `{post}`). The `scopeBindings()` method enables scoping explicitly, and `withoutScopedBindings()` disables it.

**Beginner-Friendly Explanation**

Scoped binding ensures that when you visit `/users/1/posts/5`, the post with ID 5 actually belongs to user 1. If it doesn't, you get a 404.

#### Purposes

- To enforce parent-child relationships in nested routes.
- To prevent unauthorized access to child resources.
- To automatically scope queries without manual `where` clauses.
- To maintain data integrity in RESTful APIs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();

// Group scoping
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        // ...
    });
});

// Disable scoping
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    // ...
})->withoutScopedBindings();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `scopeBindings()` | Enables scoped binding for the route or group. |
| `withoutScopedBindings()` | Disables scoped binding. |
| `resolveChildRouteBinding()` | Model method for custom child resolution. |

**Syntax Rules**

- Scoping is automatic when using custom keys in nested parameters.
- `scopeBindings()` explicitly enables scoping for routes without custom keys.
- `withoutScopedBindings()` disables automatic scoping.
- The parent model must have a relationship matching the child parameter name (plural).

**Constraints and Limitations**

- The parent model must define the guessed relationship method.
- Scoping does not work with `Route::bind()` (explicit bindings).
- Custom child resolution requires overriding `resolveChildRouteBinding()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Automatic Scoping with Custom Key**

**Code**

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

**Expected Output**

For `/users/1/posts/my-first-post`, the post with slug `my-first-post` is resolved only if it belongs to user 1.

**Why This Output Occurs**

Laravel automatically scopes the `Post` query to the `User` model via the `posts` relationship, because a custom key is used.

---

**Example 2: Explicit Scoping with `scopeBindings()`**

**Code**

```php
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

**Expected Output**

The post is resolved only if it belongs to the user.

**Why This Output Occurs**

The `scopeBindings()` method explicitly enables scoping even without a custom key.

---

**Example 3: Group Scoping**

**Code**

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', [PostController::class, 'show']);
    Route::get('/users/{user}/posts/{post}/comments/{comment}', [CommentController::class, 'show']);
});
```

**Expected Output**

All nested routes in the group use scoped binding.

**Why This Output Occurs**

The `scopeBindings()` method applied to the group enables scoping for all routes within it.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant Project Management**

A project management tool uses `/accounts/{account}/projects/{project}` routes. Scoped binding ensures that projects are only accessible within their account.

**Explanation**: Scoped binding prevents cross-tenant data leakage.

---

### 5. Route Model Binding with Scoped Bindings

#### Definitions

**Core Definition**

Scoped bindings are a specific form of route model binding that constrains the resolution of a child model to a parent model, ensuring the child belongs to the parent.

**Technical Definition**

Scoped bindings are implemented by the `resolveChildRouteBinding()` method on the parent model, which uses the parent's relationship to query the child. Laravel guesses the relationship name from the child parameter (plural form). The `scopeBindings()` method enables scoping explicitly, and `withoutScopedBindings()` disables it.

**Beginner-Friendly Explanation**

Scoped bindings are like a security guard that checks if a child resource actually belongs to the parent resource in the URL. If you try to access a post that doesn't belong to the user, you get a 404.

#### Purposes

- To prevent unauthorized access to child resources.
- To enforce parent-child relationships automatically.
- To simplify nested resource controllers.
- To maintain data integrity in RESTful APIs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Enable scoping
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    // ...
})->scopeBindings();

// Disable scoping
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    // ...
})->withoutScopedBindings();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `scopeBindings()` | Enables scoping. |
| `withoutScopedBindings()` | Disables scoping. |
| `resolveChildRouteBinding()` | Custom child resolution. |

**Syntax Rules**

- Scoping is automatic when using custom keys in nested parameters.
- `scopeBindings()` explicitly enables scoping for routes without custom keys.
- The parent model must have a relationship matching the child parameter name.

**Constraints and Limitations**

- Scoping requires a defined relationship on the parent model.
- Scoping does not work with `Route::bind()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Scoped Bindings in a Group**

**Code**

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', [PostController::class, 'show']);
});
```

**Expected Output**

The post is resolved only if it belongs to the user.

**Why This Output Occurs**

The `scopeBindings()` method on the group enables scoping for all nested routes.

---

**Example 2: Disabling Scoping**

**Code**

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    // ...
})->withoutScopedBindings();
```

**Expected Output**

The post is resolved by slug regardless of the user.

**Why This Output Occurs**

`withoutScopedBindings()` disables the automatic scoping that would otherwise apply with a custom key.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Items**

An e-commerce application uses `/orders/{order}/items/{item}` routes. Scoped binding ensures that items are only accessible within their order.

**Explanation**: Scoped binding prevents users from accessing items from other orders.

---

### 6. Route Model Binding with Soft Deletes

#### Definitions

**Core Definition**

Soft deletes in route model binding refer to the ability to include soft-deleted models when resolving route parameters, using the `withTrashed()` method.

**Technical Definition**

By default, implicit model binding does not retrieve models that have been soft-deleted. The `withTrashed()` method chained onto a route definition instructs Laravel to include soft-deleted models in the resolution. For explicit bindings, `withTrashed()` can also be used (Laravel 11.10+).

**Beginner-Friendly Explanation**

Soft deletes are a way of "hiding" records instead of actually deleting them. Normally, Laravel won't find hidden records when resolving routes. But you can tell Laravel to include them using `withTrashed()`.

#### Purposes

- To allow access to soft-deleted models for restoration or permanent deletion.
- To support admin panels that need to view deleted records.
- To enable undo functionality for deleted resources.
- To maintain data integrity while allowing recovery.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `withTrashed()` | Includes soft-deleted models in resolution. |

**Syntax Rules**

- `withTrashed()` is chained onto the route definition.
- Works with both implicit and explicit bindings (Laravel 11.10+).
- The model must use the `SoftDeletes` trait.

**Constraints and Limitations**

- Without `withTrashed()`, soft-deleted models are not resolved.
- `withTrashed()` applies to all model parameters in the route.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Resolving Soft-Deleted Models**

**Model**

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use SoftDeletes;
}
```

**Route**

```php
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

**Expected Output**

For a soft-deleted user with ID 1, the user is resolved and their email is returned.

**Why This Output Occurs**

The `withTrashed()` method instructs Laravel to include soft-deleted models in the query.

---

**Example 2: Soft Deletes with Explicit Binding (Laravel 11.10+)**

**Service Provider**

```php
Route::bind('user', function ($value) {
    return User::withTrashed()->where('id', $value)->firstOrFail();
});
```

**Expected Output**

Soft-deleted users are resolved.

**Why This Output Occurs**

The explicit binding uses `withTrashed()` to include soft-deleted models.

---

#### Real-World Cases with Explanation

**Case 1: Admin Restore Panel**

An admin panel allows administrators to restore soft-deleted users. The route uses `withTrashed()` to resolve the deleted user.

**Explanation**: Soft-delete awareness enables recovery workflows.

---

### 7. Route Model Binding Best Practices

#### Definitions

**Core Definition**

Route model binding best practices are the recommended guidelines for using route model binding effectively, securely, and maintainably.

**Technical Definition**

Best practices encompass choosing between implicit and explicit binding, using scoped bindings for nested resources, handling soft deletes appropriately, customizing keys for SEO, and combining binding with authorization.

**Beginner-Friendly Explanation**

Best practices are the tips that experienced developers follow to get the most out of route model binding while avoiding common pitfalls.

#### Key Best Practices

1. **Prefer Implicit Binding Over Manual Queries**: Use type-hinted models instead of `findOrFail()` for cleaner, more expressive controllers.

2. **Use Scoped Bindings for Nested Resources**: Always use `scopeBindings()` or custom keys for nested routes to enforce parent-child relationships and prevent unauthorized access.

3. **Use Custom Keys for SEO-Friendly URLs**: Prefer slugs or UUIDs over integer IDs for public-facing URLs.

4. **Use `getRouteKeyName()` or `#[RouteKey]` for Global Custom Keys**: Avoid specifying the column in every route; define it once on the model.

5. **Handle Soft Deletes Explicitly**: Use `withTrashed()` only when you need to access deleted records.

6. **Combine Binding with Authorization**: Use policies or gates to check if the authenticated user can access the bound model.

7. **Use Resource Controllers with Binding**: `Route::resource()` works seamlessly with implicit binding.

8. **Avoid `Route::bind()` When Implicit Binding Suffices**: Explicit binding adds complexity; use it only when necessary.

9. **Test Binding with Missing Models**: Ensure your application returns 404 responses for missing models.

10. **Use `missing()` for Custom 404 Behaviour**: The `missing()` method allows redirecting instead of returning a 404.

#### Real-World Cases with Explanation

**Case 1: RESTful API with Scoped Bindings**

A RESTful API uses scoped bindings for all nested resources, ensuring that endpoints like `/users/{user}/posts/{post}` only return posts belonging to the user.

**Explanation**: Scoped bindings are essential for secure, RESTful API design.

---

## References

- Laravel Routing Documentation (Laravel 13.x) — https://laravel.com/framework/docs/routing
- Laravel Routing Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/routing
- Laravel Routing Documentation (Laravel 9.x) — https://laravel.com/framework/docs/9.x/routing
- Laravel Routing Documentation (Laravel 8.x) — https://laravel.com/framework/docs/8.x/routing
- Laravel Route Key Attribute in Laravel 13.21 (Laravel News) — https://laravel-news.com/laravel-13-21-route-key
- Dutch Laravel Foundation: Use Route Model Binding — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/routing/use-route-model-binding.md
- Laravel Route Binding Behind the Curtains — https://expressive.wendelladriel.com/laravel-route-binding-behind-the-curtains
- Laravel Route Model Binding with Soft Deletes (Laravel Daily) — https://laraveldaily.com/post/laravel-route-model-binding-soft-deleted-models

---

**Version Note**: This cheat sheet is based on Laravel 8.x through 13.x documentation. Implicit and explicit binding have been available since early Laravel versions. Scoped bindings were introduced in Laravel 7. The `withTrashed()` method for explicit binding was added in Laravel 11.10. The `#[RouteKey]` attribute was introduced in Laravel 13.21. Developers should always consult the documentation for their specific Laravel version.