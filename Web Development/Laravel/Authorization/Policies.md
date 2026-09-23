# Resource Authorization with Policies: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Policies are dedicated authorization classes that organize permission logic around a specific Eloquent model, providing methods like `view`, `create`, `update`, and `delete` that determine whether a user can perform actions on model instances.

**Technical Definition:** Policies extend `Illuminate\Auth\Access\HandlesAuthorization` and are stored in `App\Policies`. They are mapped to models in `AuthServiceProvider::$policies` or auto-discovered by convention. Each policy method receives the authenticated `User` as its first argument, followed by the model instance (except for `create` and `viewAny`). The `Gate` class resolves policy methods when a model class or instance is passed to `can()`, `authorize()`, or Blade directives.

**Beginner-Friendly Explanation:** A policy is a rulebook for a specific type of model. For your `Post` model, the `PostPolicy` says who can view, create, edit, and delete posts. Instead of scattering permission checks throughout your controllers, you write them once in the policy and Laravel consults it automatically when you ask "Can this user edit this post?"

### Key Characteristics

- **Model-Centric:** One policy class per model, grouping all authorization logic for that model.
- **Convention-Based Discovery:** `Post` model maps to `PostPolicy` automatically in `App\Policies`.
- **Standard CRUD Methods:** `viewAny`, `view`, `create`, `update`, `delete`, plus soft-delete methods `restore` and `forceDelete`.
- **Framework Integration:** Policies work seamlessly with Blade directives, controller helpers, and route middleware.
- **Additional Context:** Extra arguments can be passed to policy methods for context-sensitive decisions.
- **Graceful Denial:** Return `Response::deny()` for custom messages or `denyAsNotFound()` to hide resources.

### Prerequisites

- A Laravel application with authentication configured.
- At least one Eloquent model and its corresponding database table.
- Understanding of Gates and the `Gate` facade.
- The `AuthServiceProvider` or `AppServiceProvider` for registering policies.

### Related Programming Areas

- **Gates:** Closure-based authorization for model-less abilities.
- **Middleware:** The `can` middleware enforces policies on routes.
- **Blade Templates:** `@can`, `@cannot`, `@canany` directives conditionally render UI.
- **Controllers:** `$this->authorize()` and `authorizeResource()` enforce policies in controller methods.

### Core Concepts / Features

1. Generating and Mapping Policies
2. Naming Conventions and Standard CRUD Methods
3. Blade Integration (`@can`, `@cannot`, `@canany`)
4. Controller Integration (`$this->authorize()`, `authorizeResource()`)
5. Route Middleware (`can:`)
6. Guest Users and Nullable Model Arguments

---

## 1. Generating Policy Classes and Mapping Them to Eloquent Models

### Definitions

**Core Definition:** Policy generation creates a dedicated authorization class for a specific Eloquent model, and mapping associates that policy with the model so Laravel knows which policy to consult for authorization checks.

**Technical Definition:** The `php artisan make:policy PostPolicy --model=Post` command generates a policy class with stub methods for standard CRUD abilities. Policies are registered via the `$policies` array in `AuthServiceProvider` or auto-discovered by convention (`App\Models\Post` → `App\Policies\PostPolicy`). The `Gate` class uses the `PolicyFinder` to resolve the policy for a given model class.

**Beginner-Friendly Explanation:** You generate a policy file for your model with one Artisan command. Laravel automatically knows to use `PostPolicy` for the `Post` model because of naming conventions. If your models and policies follow the standard structure, you don't need to configure anything.

### Purposes

- To create a dedicated authorization class for a specific model.
- To organize all permission logic for a model in one place.
- To leverage Laravel's convention-based policy discovery.
- To provide standard CRUD method stubs for immediate implementation.

### Syntax Rules and Structure

**Complete General Syntax — Generation:**

```bash
# Generate with model binding (creates CRUD method stubs)
php artisan make:policy PostPolicy --model=Post

# Generate without model binding
php artisan make:policy AdminPolicy
```

**Complete General Syntax — Registration:**

```php
// app/Providers/AuthServiceProvider.php

use App\Models\Post;
use App\Policies\PostPolicy;

protected $policies = [
    Post::class => PostPolicy::class,
];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `--model=Post` | Generates CRUD method stubs in the policy. |
| `$policies` array | Maps model classes to policy classes. |
| `Post::class => PostPolicy::class` | The mapping entry. |

**Syntax Rules:**
- Policy classes are stored in `App\Policies`.
- The `--model` flag generates stub methods for `viewAny`, `view`, `create`, `update`, `delete`, `restore`, `forceDelete`.
- Auto-discovery works when the policy name matches `{Model}Policy` in `App\Policies`.
- Custom discovery requires `Gate::guessPolicyNamesUsing()`.

**Constraints and Limitations:**
- **Class name matching:** Auto-discovery requires the model class basename to match the policy basename (e.g., `Post` → `PostPolicy`).
- **Namespace:** Policies must be in `App\Policies` for auto-discovery unless custom discovery is configured.

### Multiple Annotated Complete Code Examples

**Example 1: Generate and Register a Policy**

```bash
php artisan make:policy PostPolicy --model=Post
```

```php
<?php
// app/Policies/PostPolicy.php (generated)

namespace App\Policies;

use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\HandlesAuthorization;

class PostPolicy
{
    use HandlesAuthorization;

    public function viewAny(User $user): bool
    {
        return true;
    }

    public function view(User $user, Post $post): bool
    {
        return true;
    }

    public function create(User $user): bool
    {
        return $user->hasVerifiedEmail();
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

```php
<?php
// app/Providers/AuthServiceProvider.php

protected $policies = [
    Post::class => PostPolicy::class,
];
```

**Expected Output:** The `Post` model now uses `PostPolicy` for all authorization checks. Laravel auto-discovers the mapping if conventions are followed.

**Why:** The `--model` flag generates CRUD stubs. The `$policies` array registers the mapping with the Gate .

### Real-World Cases

- **Blog Platform:** `PostPolicy` governs who can view, create, edit, and delete posts.
- **E-commerce:** `OrderPolicy` controls order visibility, cancellation, and refunds.
- **Team Management:** `TeamPolicy` manages invitations, member removal, and settings.

### References

- How to Use Laravel Policies for Authorization — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-07-02-laravel-policies-authorization/README.md
- Laravel 10.x Authorization: Generating Policies — https://github.com/DevStorm-Team/laravel-book/blob/7a980beccf7933f873e383c4dbc4aaa8e9319b6f/laravel-docs-10.x.pdf

---

## 2. Policy Naming Conventions and Standard CRUD Methods

### Definitions

**Core Definition:** Laravel's policy naming conventions map controller actions and standard CRUD operations to specific policy methods, enabling automatic authorization with minimal configuration.

**Technical Definition:** The standard policy methods are `viewAny`, `view`, `create`, `update`, `delete`, `restore`, and `forceDelete`. Resource controllers map their actions to these methods: `index` → `viewAny`, `show` → `view`, `create`/`store` → `create`, `edit`/`update` → `update`, `destroy` → `delete`. The `restore` and `forceDelete` methods are used with soft-deleted models.

**Beginner-Friendly Explanation:** Laravel has a naming system. If your controller has an `index` method, Laravel knows to check the policy's `viewAny` method. If your controller has a `show` method, it checks `view`. You don't need to write extra configuration—just name your methods correctly.

### Purposes

- To provide a standard, predictable mapping between controller actions and policy methods.
- To enable automatic authorization via `authorizeResource()`.
- To cover the full lifecycle of a model, including soft deletes.
- To maintain consistency across the application.

### Syntax Rules and Structure

**Standard Method Mapping:**

| Controller Method | Policy Method | Model Required |
|-------------------|---------------|----------------|
| `index` | `viewAny` | No |
| `show` | `view` | Yes |
| `create` / `store` | `create` | No (class name) |
| `edit` / `update` | `update` | Yes |
| `destroy` | `delete` | Yes |
| — | `restore` | Yes (soft-deleted) |
| — | `forceDelete` | Yes (soft-deleted) |

**Complete General Syntax — Policy Methods:**

```php
public function viewAny(User $user): bool
{
    return true;
}

public function view(User $user, Post $post): bool
{
    return $post->isPublished() || $user->id === $post->user_id;
}

public function create(User $user): bool
{
    return $user->hasVerifiedEmail();
}

public function update(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}

public function delete(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}

public function restore(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}

public function forceDelete(User $user, Post $post): bool
{
    return $user->id === $post->user_id && $post->trashed();
}
```

**Component Breakdown:**

| Method | Receives Model? | Purpose |
|--------|-----------------|---------|
| `viewAny` | No | List all resources. |
| `view` | Yes | View a specific resource. |
| `create` | No | Create a new resource. |
| `update` | Yes | Update an existing resource. |
| `delete` | Yes | Delete an existing resource. |
| `restore` | Yes | Restore a soft-deleted resource. |
| `forceDelete` | Yes | Permanently delete a soft-deleted resource. |

**Syntax Rules:**
- `viewAny` and `create` receive only the `User`; they do not require a model instance.
- `view`, `update`, `delete`, `restore`, and `forceDelete` receive the `User` and the model instance.
- For `create`, pass the class name (`Post::class`) when authorizing, not an instance.
- The `before()` method can override all methods for specific users (e.g., admins).

**Constraints and Limitations:**
- **`before()` scope:** The policy's `before()` method is not called if the ability method does not exist on the policy.
- **Custom methods:** Non-CRUD methods (e.g., `publish`) follow the same signature pattern but are not automatically mapped by `authorizeResource()`.

### Multiple Annotated Complete Code Examples

**Example 1: Full Policy with CRUD Methods**

```php
<?php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\HandlesAuthorization;

class PostPolicy
{
    use HandlesAuthorization;

    public function before(User $user, string $ability): ?bool
    {
        if ($user->isAdmin()) {
            return true;
        }

        return null;
    }

    public function viewAny(User $user): bool
    {
        return true;
    }

    public function view(User $user, Post $post): bool
    {
        return $post->isPublished() || $user->id === $post->user_id;
    }

    public function create(User $user): bool
    {
        return $user->hasVerifiedEmail();
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function restore(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function forceDelete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id && $post->trashed();
    }
}
```

**Expected Output:** Admins can do anything (via `before`). Regular users can view published posts, create posts with verified emails, and update/delete/restore/force-delete their own posts.

**Why:** The `before()` method grants admins full access. The CRUD methods implement specific rules for each action .

### Real-World Cases

- **Content Management:** `viewAny` for index pages, `view` for detail pages, `create`/`update`/`delete` for CRUD operations.
- **Soft-Deletable Models:** `restore` and `forceDelete` manage the trash lifecycle.
- **Multi-Tenant:** Policy methods check tenant membership before allowing access.

### References

- Laravel 9.x Authorization — https://laravel.com/framework/docs/9.x/authorization
- Authorization | Laravel 13.x — https://laravel.com/framework/docs/authorization
- Laravel 10.x Authorization PDF — https://github.com/DevStorm-Team/laravel-book/blob/7a980beccf7933f873e383c4dbc4aaa8e9319b6f/laravel-docs-10.x.pdf

---

## 3. Blade: Protecting UI Components

### Definitions

**Core Definition:** Blade authorization directives conditionally render UI elements based on whether the current user is authorized to perform a given action, keeping the frontend consistent with backend permissions.

**Technical Definition:** The `@can`, `@cannot`, and `@canany` directives are Blade shortcuts for `Auth::user()->can()` checks. `@can('update', $post)` renders its block only if the user passes the `update` policy method. `@cannot` renders when denied. `@canany(['publish', 'schedule'], $article)` renders when the user passes at least one of the listed abilities. Class names can be passed for model-less actions (`@can('create', Post::class)`).

**Beginner-Friendly Explanation:** These directives hide buttons and links that users aren't allowed to use. If a user can't edit a post, the Edit button simply doesn't appear. This prevents confusing errors and keeps the interface clean.

### Purposes

- To conditionally display UI elements based on user permissions.
- To maintain consistency between backend authorization and frontend display.
- To prevent users from seeing actions they cannot perform.
- To support complex permission scenarios with `@canany`.

### Syntax Rules and Structure

**Complete General Syntax — `@can`:**

```blade
@can('update', $post)
    <button>Edit Post</button>
@endcan
```

**Complete General Syntax — `@cannot`:**

```blade
@cannot('delete', $post)
    <span>You cannot delete this post.</span>
@endcannot
```

**Complete General Syntax — `@canany`:**

```blade
@canany(['publish', 'schedule'], $article)
    <div>Publishing controls</div>
@endcanany
```

**Complete General Syntax — Class Name for Model-Less Actions:**

```blade
@can('create', App\Models\Post::class)
    <a href="/posts/create">Create Post</a>
@endcan
```

**Component Breakdown:**

| Directive | Behaviour |
|-----------|-----------|
| `@can('ability', $model)` | Renders if authorized. |
| `@cannot('ability', $model)` | Renders if denied. |
| `@canany(['ability1', 'ability2'], $model)` | Renders if authorized for any. |
| `@elsecan` / `@elsecannot` | Chained alternatives. |

**Syntax Rules:**
- `@can` and `@cannot` accept the ability name as the first argument.
- The second argument is the model instance (or class name for model-less actions).
- `@canany` accepts an array of ability names.
- `@elsecan` and `@elsecannot` provide chained alternatives.

**Constraints and Limitations:**
- **Multiple guards:** Blade directives use the default guard. To specify a guard, use `Auth::guard('admin')->user()->can()` in an `@if`.
- **Performance:** Each directive triggers an authorization check. Avoid excessive nesting.

### Multiple Annotated Complete Code Examples

**Example 1: Basic UI Protection**

```blade
{{-- resources/views/posts/show.blade.php --}}

<div class="post-actions">
    @can('update', $post)
        <a href="{{ route('posts.edit', $post) }}" class="btn">Edit</a>
    @endcan

    @can('delete', $post)
        <form method="POST" action="{{ route('posts.destroy', $post) }}">
            @csrf
            @method('DELETE')
            <button type="submit" class="btn btn-danger">Delete</button>
        </form>
    @endcan
</div>
```

**Expected Output:** Edit and Delete buttons appear only for users authorized to update or delete the post.

**Why:** `@can` checks the policy method and renders the block only if authorized .

---

**Example 2: `@canany` for Multiple Permissions**

```blade
{{-- resources/views/articles/show.blade.php --}}

@canany(['publish', 'schedule'], $article)
    <div class="publishing-controls">
        <h3>Publishing Options</h3>
        @can('publish', $article)
            <button>Publish Now</button>
        @endcan
        @can('schedule', $article)
            <button>Schedule</button>
        @endcan
    </div>
@endcanany

@cannot('publish', $article)
    <p>You do not have permission to publish this article.</p>
@endcannot
```

**Expected Output:** Publishing controls appear only if the user can either publish or schedule the article. The specific buttons are further gated by individual `@can` checks.

**Why:** `@canany` checks multiple abilities at once, useful for grouping related controls .

### Real-World Cases

- **Admin Panels:** `@can('viewAny', User::class)` gates admin navigation.
- **Blog Editing:** `@can('update', $post)` shows edit links only to authors.
- **E-commerce:** `@can('refund', $order)` displays refund options to authorized staff.

### References

- Blade Authorization Directives for View Security — https://laravel-news.com/index.php/blade-authorization-can-cannot
- Laravel Blade Directives Reference — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-blade/references/directives.md

---

## 4. Controllers: `$this->authorize()` and `authorizeResource()`

### Definitions

**Core Definition:** Controller authorization helpers provide methods to check policies before executing controller actions, either individually per method or automatically across all resource controller methods.

**Technical Definition:** The `$this->authorize('ability', $model)` method (available via the `AuthorizesRequests` trait) throws `AuthorizationException` (converted to 403) on failure. The `$this->authorizeResource(Model::class, 'param')` method in the controller constructor attaches `can` middleware to all resource controller methods based on the model's policy. In Laravel 11+, `authorizeResource()` works only with classic controllers extending the base `Controller` class; invokable/slim controllers must use route middleware instead.

**Beginner-Friendly Explanation:** In your controller, you can check permissions one method at a time with `$this->authorize()`, or you can set up automatic checks for all CRUD methods with `authorizeResource()`. The latter is like telling Laravel: "For this controller, automatically check the policy before every action."

### Purposes

- To authorize actions within controller methods before executing logic.
- To automatically map resource controller methods to policy methods.
- To throw `AuthorizationException` for automatic 403 responses.
- To keep authorization logic close to the controller action it protects.

### Syntax Rules and Structure

**Complete General Syntax — `$this->authorize()`:**

```php
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);

    // Authorized, proceed with update
}
```

**Complete General Syntax — Model-Less Action:**

```php
public function create()
{
    $this->authorize('create', Post::class);

    // Authorized, show create form
}
```

**Complete General Syntax — `authorizeResource()`:**

```php
public function __construct()
{
    $this->authorizeResource(Post::class, 'post');
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `$this->authorize('ability', $model)` | Checks policy; throws 403 on denial. |
| `$this->authorize('create', Post::class)` | Checks model-less action. |
| `authorizeResource(Post::class, 'post')` | Auto-maps all resource methods. |

**Method Mapping with `authorizeResource()`:**

| Controller Method | Policy Method |
|-------------------|---------------|
| `index` | `viewAny` |
| `show` | `view` |
| `create` | `create` |
| `store` | `create` |
| `edit` | `update` |
| `update` | `update` |
| `destroy` | `delete` |

**Syntax Rules:**
- `$this->authorize()` must be called before the protected action.
- `authorizeResource()` is called in the controller's `__construct()` method.
- The second argument to `authorizeResource()` is the route parameter name (e.g., `'post'` for `{post}`).
- The controller must extend `App\Http\Controllers\Controller` for `authorizeResource()` to work.

**Constraints and Limitations:**
- **Laravel 11+ slim controllers:** `authorizeResource()` is not available in invokable/slim controllers that do not extend the base `Controller` class.
- **`authorize()` in Laravel 11+:** The `AuthorizesRequests` trait must be explicitly added if the base controller does not include it.
- **Route middleware alternative:** Use `->middleware('can:update,post')` for slim controllers.

### Multiple Annotated Complete Code Examples

**Example 1: Manual `$this->authorize()`**

```php
<?php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function edit(Post $post)
    {
        $this->authorize('update', $post);

        return view('posts.edit', compact('post'));
    }

    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);

        $post->update($request->validated());

        return redirect()->route('posts.show', $post);
    }

    public function create()
    {
        $this->authorize('create', Post::class);

        return view('posts.create');
    }
}
```

**Expected Output:** Unauthorized users receive a 403 response before the controller method body executes. Authorized users proceed normally.

**Why:** `$this->authorize()` checks the policy and throws `AuthorizationException` on denial, which Laravel converts to 403 .

---

**Example 2: Automatic `authorizeResource()`**

```php
<?php
namespace App\Http\Controllers;

use App\Models\Post;

class PostController extends Controller
{
    public function __construct()
    {
        // Automatically authorize all resource methods
        $this->authorizeResource(Post::class, 'post');
    }

    public function index()
    {
        // Authorized via PostPolicy@viewAny
        return view('posts.index', ['posts' => Post::all()]);
    }

    public function show(Post $post)
    {
        // Authorized via PostPolicy@view
        return view('posts.show', compact('post'));
    }

    public function edit(Post $post)
    {
        // Authorized via PostPolicy@update
        return view('posts.edit', compact('post'));
    }

    public function update(Request $request, Post $post)
    {
        // Authorized via PostPolicy@update
        $post->update($request->validated());
        return redirect()->route('posts.show', $post);
    }

    public function destroy(Post $post)
    {
        // Authorized via PostPolicy@delete
        $post->delete();
        return redirect()->route('posts.index');
    }
}
```

**Expected Output:** Each resource method is automatically authorized via the corresponding policy method. Unauthorized requests receive 403 before the method body runs.

**Why:** `authorizeResource()` attaches the `can` middleware to each resource controller method, mapping controller actions to policy methods .

### Real-World Cases

- **Resource Controllers:** `authorizeResource()` for standard CRUD controllers.
- **Custom Actions:** `$this->authorize()` for non-resource actions like `publish` or `archive`.
- **API Controllers:** `$this->authorize()` in API controllers for JSON 403 responses.

### References

- Laravel 10.x Authorization: Via Controller Helpers — https://github.com/DevStorm-Team/laravel-book/blob/7a980beccf7933f873e383c4dbc4aaa8e9319b6f/laravel-docs-10.x.pdf
- Laravel 13.x Authorization: Via Controller Helpers — https://laravel.com/framework/docs/authorization
- Policies not blocking edits — https://laracasts.com/index.php/discuss/channels/laravel/policies-not-blocking-edits

---

## 5. Routing: Applying `can:` Middleware

### Definitions

**Core Definition:** The `can` middleware authorizes actions at the route level, checking the policy before the request reaches the controller or closure.

**Technical Definition:** The `can:` middleware alias maps to `Illuminate\Auth\Middleware\Authorize`. It accepts the ability name and optional model parameter: `can:update,post`. For model-less actions, pass the class name: `can:create,App\Models\Post`. When authorization fails, the middleware returns a 403 response. In Laravel 11+, the `can()` method on routes works for individual routes but not for route groups (use `->middleware('can:...')` instead).

**Beginner-Friendly Explanation:** Instead of checking permissions inside your controller, you can attach a `can` middleware to the route. Laravel checks the policy before the controller runs. If denied, the user gets a 403 and the controller never executes.

### Purposes

- To authorize actions before the request reaches the controller.
- To keep authorization logic declarative in route definitions.
- To protect routes without modifying controller code.
- To support both model-specific and model-less authorization.

### Syntax Rules and Structure

**Complete General Syntax — Model-Specific:**

```php
Route::put('/post/{post}', [PostController::class, 'update'])
    ->middleware('can:update,post');
```

**Complete General Syntax — Model-Less:**

```php
Route::post('/post', [PostController::class, 'store'])
    ->middleware('can:create,App\Models\Post');
```

**Complete General Syntax — Individual Route with `->can()`:**

```php
Route::put('/post/{post}', [PostController::class, 'update'])
    ->can('update', 'post');
```

**Component Breakdown:**

| Syntax | Description |
|--------|-------------|
| `->middleware('can:update,post')` | Route middleware (works everywhere). |
| `->can('update', 'post')` | Route method (single routes only). |
| `can:update,post` | Ability and route parameter. |
| `can:create,App\Models\Post` | Model-less action (class name). |

**Syntax Rules:**
- The middleware accepts the ability and route parameter separated by a comma.
- The route parameter must match the parameter name in the route definition (e.g., `{post}` → `post`).
- For model-less actions, pass the fully qualified class name.
- `->can()` works on individual routes; `->middleware('can:...')` works everywhere.

**Constraints and Limitations:**
- **Route groups:** `->can()` on a route group does not register the middleware; use `->middleware('can:...')` instead .
- **Multiple guards:** The `can` middleware uses the default guard.
- **No custom messages:** The middleware returns a generic 403; use controller authorization for custom messages.

### Multiple Annotated Complete Code Examples

**Example 1: Route Middleware for Model-Specific Action**

```php
<?php
// routes/web.php

use App\Http\Controllers\PostController;

Route::put('/post/{post}', [PostController::class, 'update'])
    ->middleware('can:update,post');

Route::delete('/post/{post}', [PostController::class, 'destroy'])
    ->middleware('can:delete,post');
```

**Expected Output:** Only users authorized to update or delete the specific post can access these routes. Others receive 403.

**Why:** The `can:update,post` middleware checks `PostPolicy@update` with the route-bound `$post` model before the controller method runs .

---

**Example 2: Model-Less Route Authorization**

```php
<?php
Route::post('/post', [PostController::class, 'store'])
    ->middleware('can:create,App\Models\Post');
```

**Expected Output:** Only users authorized to create posts can access the route. Others receive 403.

**Why:** Passing the class name (`App\Models\Post`) tells Laravel to use `PostPolicy@create` without requiring a model instance .

---

**Example 3: Route Group with Middleware**

```php
<?php
Route::prefix('admin')
    ->middleware('can:access-admin')
    ->group(function () {
        Route::get('/dashboard', [AdminController::class, 'dashboard']);
        Route::get('/users', [AdminController::class, 'users']);
    });
```

**Expected Output:** All routes in the group require the `access-admin` ability. Unauthorized users receive 403.

**Why:** `->middleware('can:...')` works on route groups. The `->can()` method does not work on groups in Laravel 11+ .

### Real-World Cases

- **RESTful APIs:** `can:update,post` on PUT routes for clean, declarative authorization.
- **Admin Sections:** `can:access-admin` on admin route groups.
- **Model-Less Actions:** `can:create,App\Models\Post` on store routes.

### References

- Laravel 13.x Authorization: Via Middleware — https://laravel.com/framework/docs/authorization
- can() method does not work for route groups · Issue #55114 — https://github.com/laravel/framework/issues/55114
- How is the 'can' attribute processed in Route Groups? — https://github.com/laravel/framework/discussions/56663

---

## 6. Handling Guest/Unauthenticated Users with Nullable Model Arguments

### Definitions

**Core Definition:** Policy methods can accept nullable user arguments to handle guest (unauthenticated) users, allowing public access to certain actions while restricting others.

**Technical Definition:** By default, Laravel's `Gate` returns `false` for all policy checks when no user is authenticated. To allow guests, the policy method's `User` parameter can be made nullable (`?User $user`), and the method logic explicitly handles the `null` case. For model-less actions, the same pattern applies. The `before()` method can also be used to handle guest logic globally.

**Beginner-Friendly Explanation:** Sometimes you want guests (people who aren't logged in) to be able to do certain things, like view published blog posts. By making the user parameter nullable in your policy, you can say "if the user is null (a guest), allow this action" or "if the user is null, deny this action."

### Purposes

- To allow public access to specific actions (e.g., viewing published content).
- To restrict other actions to authenticated users.
- To handle guest authorization checks gracefully without errors.
- To provide a consistent authorization API for both authenticated and guest users.

### Syntax Rules and Structure

**Complete General Syntax — Nullable User:**

```php
public function view(?User $user, Post $post): bool
{
    // Guests can view published posts
    if ($post->isPublished()) {
        return true;
    }

    // Guests cannot view unpublished posts
    if ($user === null) {
        return false;
    }

    // Authenticated users can view their own unpublished posts
    return $user->id === $post->user_id;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `?User $user` | Nullable user parameter. |
| `$user === null` | Check for guest. |
| Return logic | Allow/deny based on guest status. |

**Complete General Syntax — Model-Less Guest Access:**

```php
public function viewAny(?User $user): bool
{
    // Everyone (guests and users) can list published posts
    return true;
}
```

**Syntax Rules:**
- Making the `User` parameter nullable allows the policy method to be called for guests.
- Without nullable, Laravel returns `false` for guests without calling the method.
- The `before()` method can handle guest logic globally.
- For model-less actions (`create`, `viewAny`), the same nullable pattern applies.

**Constraints and Limitations:**
- **Default denial:** Without nullable, all policy checks return `false` for guests.
- **`before()` and guests:** The `before()` method receives `null` for the user parameter if the signature allows it.
- **Route middleware:** The `can` middleware still returns 403 for guests unless the policy explicitly allows them.

### Multiple Annotated Complete Code Examples

**Example 1: Allow Guests to View Published Posts**

```php
<?php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function view(?User $user, Post $post): bool
    {
        // Published posts are viewable by anyone (including guests)
        if ($post->isPublished()) {
            return true;
        }

        // Guests cannot view unpublished posts
        if ($user === null) {
            return false;
        }

        // Authors can view their own unpublished posts
        return $user->id === $post->user_id;
    }
}
```

```php
// Controller
public function show(Post $post)
{
    $this->authorize('view', $post);
    return view('posts.show', compact('post'));
}
```

**Expected Output:** Guests can view published posts. Guests attempting to view unpublished posts receive 403. Authenticated authors can view their own unpublished posts.

**Why:** The nullable `?User $user` allows the policy method to be called for guests. The logic explicitly handles the `null` case .

---

**Example 2: Allow Guests to List All Posts**

```php
<?php
public function viewAny(?User $user): bool
{
    // Everyone can list posts (the query filters by visibility)
    return true;
}
```

**Expected Output:** Guests and authenticated users can access the post index page. The controller filters the query to show only publicly visible posts to guests.

**Why:** `viewAny` with a nullable user allows the index action for everyone. The controller is responsible for filtering the data .

### Real-World Cases

- **Blog Platform:** Guests can view published posts; only authors can edit drafts.
- **E-commerce:** Guests can browse products; only authenticated users can add to cart.
- **Documentation Sites:** Guests can read public docs; only team members can edit.

### References

- Laravel 9.x Authorization: Guest Users — https://laravel.com/framework/docs/9.x/authorization
- Laravel 13.x Authorization — https://laravel.com/framework/docs/authorization

---

## References

- How to Use Laravel Policies for Authorization — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-07-02-laravel-policies-authorization/README.md
- Blade Authorization Directives for View Security — https://laravel-news.com/index.php/blade-authorization-can-cannot
- Policies not blocking edits — https://laracasts.com/index.php/discuss/channels/laravel/policies-not-blocking-edits
- Laravel 13.x Authorization — https://laravel.com/framework/docs/authorization
- Laravel 10.x Authorization PDF — https://github.com/DevStorm-Team/laravel-book/blob/7a980beccf7933f873e383c4dbc4aaa8e9319b6f/laravel-docs-10.x.pdf
- Laravel 9.x Authorization — https://laravel.com/framework/docs/9.x/authorization
- can() method does not work for route groups · Issue #55114 — https://github.com/laravel/framework/issues/55114
- How is the 'can' attribute processed in Route Groups? — https://github.com/laravel/framework/discussions/56663
- Spatie laravel-permission Blade Directives — https://spatie.be/docs/laravel-permission/v5/basic-usage/blade-directives
- Laravel Nova Authorization — https://nova.laravel.com/docs/v4/resources/authorization
- ktourvas/rolesandperms Package — https://packagist.org/packages/ktourvas/rolesandperms
- triotechnology/laravel-permission Package — https://packagist.org/packages/triotechnology/laravel-permission