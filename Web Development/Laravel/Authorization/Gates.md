# Procedural Authorization with Gates: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Gates are closure-based authorization callbacks that define standalone abilities for checking whether an authenticated user is permitted to perform a specific action, independent of any model or policy class.

**Technical Definition:** The `Illuminate\Auth\Access\Gate` class (accessible via the `Gate` facade) provides a fluent API for defining and evaluating authorization abilities. Gates are registered via `Gate::define()` and evaluated through `Gate::allows()`, `Gate::denies()`, `Gate::authorize()`, or `Gate::inspect()`. Global hooks registered via `Gate::before()` and `Gate::after()` intercept all authorization checks, enabling administrative bypass logic and audit logging .

**Beginner-Friendly Explanation:** Think of Gates as simple permission rules you write inline. Instead of creating a whole policy class for every permission check, you can define a closure like "Admins can edit settings" and call `Gate::allows('edit-settings')` anywhere in your application. Gates are perfect for model-less permissions—actions that don't relate to a specific database record.

### Key Characteristics

- **Closure-Based:** Gates are defined as simple PHP closures, not classes.
- **Model-Less:** Ideal for actions that don't require a specific model instance (e.g., "view admin dashboard").
- **Fluent Evaluation:** Methods like `allows()`, `denies()`, and `inspect()` provide flexible checking options.
- **Global Interception:** `before()` and `after()` hooks run for every gate check, enabling cross-cutting concerns.
- **Context Passing:** Additional arguments can be passed into gate closures for context-sensitive decisions.

### Prerequisites

- A Laravel application with authentication configured.
- The `Gate` facade imported (`use Illuminate\Support\Facades\Gate;`).
- A service provider (typically `AuthServiceProvider` or `AppServiceProvider`) for registering gates.
- Understanding of closures and basic authorization concepts.

### Related Programming Areas

- **Policies:** Class-based authorization for model-specific actions.
- **Middleware:** The `can` middleware uses gates for route protection.
- **Blade Templates:** `@can` and `@cannot` directives evaluate gates in views.
- **Controllers:** `$this->authorize()` and `Gate::authorize()` enforce gates in controller methods.

### Core Concepts / Features

1. Defining Standalone Closure-Based Gates
2. Evaluating Authorization Requests (`allows()`, `denies()`, `inspect()`)
3. Global Overrides (`before()` and `after()` Hooks)
4. Passing Additional Context and Data Arguments

---

## 1. Defining Standalone Closure-Based Gates

### Definitions

**Core Definition:** A Gate definition is a named closure registered with the `Gate` facade that receives the authenticated user (and optional arguments) and returns `true` or `false` to grant or deny the action.

**Technical Definition:** `Gate::define(string $ability, callable|string $callback)` registers an ability in the Gate's `$abilities` array. The closure receives the authenticated user as its first argument, followed by any additional arguments passed during evaluation. The closure's return value is normalised to a boolean (or a `Response` instance).

**Beginner-Friendly Explanation:** Defining a gate is like writing a rule in plain PHP: "If the user is an admin, return true." You give the rule a name (like `edit-settings`), and later you ask Laravel "Can this user edit settings?" Laravel runs your rule and tells you the answer.

### Purposes

- To define permissions for actions that don't relate to a specific model.
- To keep simple authorization logic out of policy classes.
- To provide a named, reusable permission check.
- To allow closure-based logic with access to the authenticated user.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
use Illuminate\Support\Facades\Gate;

Gate::define('ability-name', function (User $user, $additionalArgument = null) {
    return $user->condition; // true or false
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'ability-name'` | The unique name of the ability (e.g., `'edit-settings'`). |
| `function (User $user)` | Closure receiving the authenticated user. |
| `$additionalArgument` | Optional additional data passed during evaluation. |
| Return value | `true` to allow, `false` to deny. |

**Syntax Rules:**
- Gates are typically registered in `AuthServiceProvider::boot()` or `AppServiceProvider::boot()`.
- The closure must type-hint the `User` model as its first parameter.
- Gates can be defined anywhere, but central registration is recommended.
- Gate names should be descriptive and use kebab-case (e.g., `'manage-billing'`).

**Constraints and Limitations:**
- **Guest users:** By default, gates return `false` for unauthenticated users. To allow guests, make the user parameter nullable .
- **No automatic HTTP response:** A denied gate returns `false`; it does not automatically redirect or return an error unless you use `Gate::authorize()`.
- **No model binding:** Gates do not automatically resolve models; pass model instances as additional arguments if needed.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Gate Definition**

```php
<?php
// app/Providers/AuthServiceProvider.php

namespace App\Providers;

use App\Models\User;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;

class AuthServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Define a simple gate for admin settings access
        Gate::define('edit-settings', function (User $user) {
            return $user->isAdmin;
        });

        // Define a gate with additional logic
        Gate::define('view-reports', function (User $user) {
            return $user->isAdmin || $user->isManager;
        });
    }
}
```

```php
// Usage in a controller
if (Gate::allows('edit-settings')) {
    // Show settings form
}
```

**Expected Output:** Admins can edit settings; managers and regular users cannot. The `view-reports` gate allows both admins and managers.

**Why:** `Gate::define()` registers the closure under the ability name. When `Gate::allows()` is called, the closure receives the authenticated user and returns the boolean result .

---

**Example 2: Gate with Optional Guest Access**

```php
<?php
// Allow guests to view public content
Gate::define('view-public-content', function (?User $user) {
    // If user is null (guest), still allow
    if ($user === null) {
        return true;
    }

    // Authenticated users can always view
    return true;
});
```

**Expected Output:** Both guests and authenticated users can view public content.

**Why:** Making the `$user` parameter nullable (`?User $user`) allows the gate to be called for unauthenticated requests. Without the nullable hint, the gate returns `false` for guests .

---

**Example 3: Gate for Model-Less Create Action**

```php
<?php
Gate::define('create-post', function (User $user) {
    return $user->hasVerifiedEmail() && $user->subscription->active;
});
```

```php
// Controller usage
if (Gate::denies('create-post')) {
    abort(403, 'You must have a verified email and active subscription to create posts.');
}
```

**Expected Output:** Only users with verified emails and active subscriptions can create posts. Others receive a 403 error with a custom message.

**Why:** The `create-post` gate checks multiple conditions without requiring a specific model instance. `Gate::denies()` returns the inverse of `Gate::allows()` .

### Real-World Cases

- **Admin Dashboards:** `Gate::define('view-admin', fn($user) => $user->isAdmin)` protects admin panels.
- **Feature Flags:** `Gate::define('use-beta-features', fn($user) => $user->isBetaTester)` gates access to experimental features.
- **Subscription Tiers:** `Gate::define('access-premium-content', fn($user) => $user->subscribed('premium'))` controls premium access.
- **Team Management:** `Gate::define('manage-team', fn($user) => $user->isTeamOwner())` gates team administration.

### References

- Gate Interface API (Laravel 7.x) — https://api.laravel.com/docs/7.x/Illuminate/Contracts/Auth/Access/Gate.html
- Gate Class API (Laravel 8.x) — https://api.laravel.com/docs/8.x/Illuminate/Auth/Access/Gate.html

---

## 2. Evaluating Authorization Requests via the Gate Facade

### Definitions

**Core Definition:** Gate evaluation methods are the static facade methods used to check whether the current user is authorized to perform a given ability, returning either a boolean or a full authorization response object.

**Technical Definition:** The `Gate` facade provides four primary evaluation methods: `allows()` returns a boolean indicating permission, `denies()` returns the inverse boolean, `authorize()` throws an `AuthorizationException` on failure, and `inspect()` returns the full `Response` object (allowing access to the message and status code). These methods resolve the ability to a closure or policy method and invoke it with the authenticated user and provided arguments .

**Beginner-Friendly Explanation:** These are the "questions" you ask about permissions. `allows()` asks "Can they do this?" and gives a yes/no answer. `authorize()` asks the same question but throws an error if the answer is no (useful in controllers). `inspect()` gives you the full answer including any custom error message.

### Purposes

- To check permissions and branch logic based on the result (`allows`, `denies`).
- To enforce authorization by throwing an exception on failure (`authorize`).
- To retrieve detailed authorization responses including messages and status codes (`inspect`).
- To provide a consistent API for authorization checks across controllers, middleware, and services.

### Syntax Rules and Structure

**Complete General Syntax — `allows()` and `denies()`:**

```php
use Illuminate\Support\Facades\Gate;

// Boolean checks
if (Gate::allows('update-post', $post)) {
    // Authorized
}

if (Gate::denies('delete-post', $post)) {
    // Denied
}
```

**Complete General Syntax — `authorize()`:**

```php
// Throws AuthorizationException (403) on failure
Gate::authorize('update-post', $post);
```

**Complete General Syntax — `inspect()`:**

```php
use Illuminate\Auth\Access\Response;

$response = Gate::inspect('update-post', $post);

if ($response->allowed()) {
    // Authorized
} else {
    echo $response->message(); // Custom denial message
    echo $response->status();  // HTTP status code
}
```

**Component Breakdown:**

| Method | Return Type | Behaviour |
|--------|-------------|-----------|
| `allows($ability, $args)` | `bool` | Returns `true` if allowed. |
| `denies($ability, $args)` | `bool` | Returns `true` if denied. |
| `authorize($ability, $args)` | `Response` | Throws `AuthorizationException` on denial. |
| `inspect($ability, $args)` | `Response` | Returns full response object. |

**Syntax Rules:**
- `allows()` and `denies()` are inverses of each other.
- `authorize()` is preferred in controllers for automatic 403 responses.
- `inspect()` is used when you need the denial message or status code.
- Additional arguments are passed as the second parameter (array or single value).

**Constraints and Limitations:**
- **`allows()` vs `authorize()`:** `allows()` returns `false` on denial; `authorize()` throws an exception. Choose based on whether you want to handle the failure or let Laravel handle it.
- **Response messages:** Custom denial messages are only available via `inspect()` or `authorize()`; `allows()` returns only a boolean.
- **Guest users:** By default, all methods return `false` (or throw) for unauthenticated users unless the gate allows guests.

### Multiple Annotated Complete Code Examples

**Example 1: Using `allows()` for Conditional Logic**

```php
<?php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::all();

        return view('posts.index', [
            'posts' => $posts,
            'canCreate' => Gate::allows('create-post'),
        ]);
    }

    public function edit(Post $post)
    {
        if (Gate::denies('update-post', $post)) {
            abort(403, 'You cannot edit this post.');
        }

        return view('posts.edit', compact('post'));
    }
}
```

**Expected Output:** The index view receives a `$canCreate` boolean to conditionally show the "Create Post" button. The edit method aborts with 403 if the user cannot update the post.

**Why:** `Gate::allows()` returns a boolean for conditional display logic. `Gate::denies()` is used for early abort with a custom message .

---

**Example 2: Using `authorize()` for Automatic 403**

```php
<?php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    public function update(Request $request, Post $post)
    {
        // Throws AuthorizationException (403) if denied
        Gate::authorize('update-post', $post);

        $post->update($request->validated());

        return redirect()->route('posts.show', $post);
    }
}
```

**Expected Output:** Unauthorized users receive a 403 Forbidden response automatically. Authorized users proceed to update the post.

**Why:** `Gate::authorize()` throws `Illuminate\Auth\Access\AuthorizationException` on denial. Laravel's exception handler automatically converts this to a 403 HTTP response .

---

**Example 3: Using `inspect()` for Custom Error Messages**

```php
<?php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    public function update(Request $request, Post $post)
    {
        $response = Gate::inspect('update-post', $post);

        if ($response->denied()) {
            return back()->with('error', $response->message());
        }

        $post->update($request->validated());

        return redirect()->route('posts.show', $post);
    }
}
```

**Expected Output:** If the gate denial includes a custom message (via `Response::deny('You do not own this post.')`), that message is displayed to the user. Otherwise, a generic error is shown.

**Why:** `Gate::inspect()` returns the full `Response` object, providing access to `message()` and `status()` for custom error handling .

### Real-World Cases

- **Blade Templates:** `@can('update-post', $post)` uses `allows()` under the hood.
- **API Controllers:** `Gate::authorize()` in API controllers returns 403 JSON responses automatically.
- **Conditional UI:** `Gate::allows()` controls whether to show edit/delete buttons.
- **Audit Logging:** `Gate::inspect()` captures denial messages for logging.

### References

- Gate Facade API (Laravel 8.x) — https://api.laravel.com/docs/8.x/Illuminate/Support/Facades/Gate.html
- Laravel Authorization Documentation (12.x) — https://github.com/Vectorial1024/docs/blob/12.x/authorization.md

---

## 3. Global Overrides: Gate::before() and Gate::after() Hooks

### Definitions

**Core Definition:** `Gate::before()` and `Gate::after()` are global hooks that run before or after **every** gate and policy check, enabling cross-cutting authorization logic such as administrative bypass or audit logging.

**Technical Definition:** The `Gate` class maintains `$beforeCallbacks` and `$afterCallbacks` arrays. Before callbacks are invoked by `callBeforeCallbacks()` before resolving the ability; if a callback returns a non-null value, that value is used as the authorization result and the normal check is skipped. After callbacks are invoked by `callAfterCallbacks()` after the ability is resolved, receiving the user, ability, arguments, and result; they can modify the result only if the original result was `null` .

**Beginner-Friendly Explanation:** Think of `before()` as a "super-admin override"—it runs before any other permission check and can say "yes, allow this no matter what." Think of `after()` as a "notebook"—it runs after every check and can record what happened (for audit logs). The `before()` hook can also deny everything (useful for maintenance mode).

### Purposes

- To grant super-administrators access to all actions without defining explicit permissions (`before`).
- To deny all actions during maintenance or in specific states (`before`).
- To log every authorization check for auditing and debugging (`after`).
- To apply conditional overrides based on user roles or application state (`before`).
- To modify authorization results after normal evaluation (`after`).

### Syntax Rules and Structure

**Complete General Syntax — `before()`:**

```php
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    // Super admin bypass
    if ($user->isSuperAdmin()) {
        return true; // Allow everything
    }

    // Return null to fall through to normal checks
    return null;
});
```

**Complete General Syntax — `after()`:**

```php
Gate::after(function (User $user, string $ability, $result, array $arguments) {
    // Log every authorization check
    logger()->info("Authorization check: {$ability}", [
        'user_id' => $user->id,
        'result' => $result,
    ]);

    // Return result to keep it unchanged
    return $result;
});
```

**Component Breakdown:**

| Hook | Parameters | Return Behaviour |
|------|------------|------------------|
| `before` | `User $user, string $ability` | Return `true`/`false` to override; `null` to continue. |
| `after` | `User $user, string $ability, $result, array $arguments` | Return the result (modified or not). |

**Syntax Rules:**
- `before()` callbacks run in registration order. The first non-null return short-circuits the check.
- `after()` callbacks run after the normal check. They receive the resolved result and can return a modified value.
- Both hooks apply to **all** gates and policies.
- In Laravel 11+, these can be registered in `AppServiceProvider::boot()`.
- Returning `false` from `before()` denies all abilities for matching users .

**Constraints and Limitations:**
- **`before()` returning `false`:** This denies everything, including actions that might otherwise be allowed. Use carefully.
- **`after()` result modification:** After callbacks can only modify the result if the original was `null`; otherwise, the result is already determined.
- **Policy `before` vs. Gate `before`:** The global Gate `before()` runs before any policy's own `before()` method.
- **Guest users:** `before()` and `after()` receive `null` for the user parameter when the request is unauthenticated (unless the callback signature allows it).

### Multiple Annotated Complete Code Examples

**Example 1: Super Admin Bypass with `before()`**

```php
<?php
// app/Providers/AuthServiceProvider.php

namespace App\Providers;

use App\Models\User;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;

class AuthServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Super admins can do everything
        Gate::before(function (User $user, string $ability) {
            if ($user->hasRole('Super Admin')) {
                return true; // Allow all abilities
            }

            // Return null to continue with normal checks
            return null;
        });
    }
}
```

**Expected Output:** Users with the "Super Admin" role pass **all** authorization checks (gates and policies) without needing explicit permissions. Other users fall through to normal checks.

**Why:** The `before()` callback returns `true` for super admins, short-circuiting all other authorization logic. Returning `null` allows normal processing for other users .

---

**Example 2: Audit Logging with `after()`**

```php
<?php
// app/Providers/AuthServiceProvider.php

use Illuminate\Support\Facades\Gate;
use Illuminate\Support\Facades\Log;

Gate::after(function (User $user, string $ability, $result, array $arguments) {
    // Log every authorization check
    Log::info('Authorization check', [
        'user_id' => $user->id,
        'ability' => $ability,
        'result' => $result,
        'arguments' => collect($arguments)->map(fn($arg) => 
            is_object($arg) ? get_class($arg) : $arg
        )->toArray(),
    ]);

    // Return the result unchanged
    return $result;
});
```

**Expected Output:** Every `Gate::allows()`, `Gate::denies()`, and `$user->can()` call produces a log entry with the user, ability, result, and argument types.

**Why:** The `after()` callback runs after every authorization check, providing comprehensive audit logging without modifying the authorization result .

---

**Example 3: Maintenance Mode Denial with `before()`**

```php
<?php
Gate::before(function (User $user, string $ability) {
    // During maintenance, deny all actions except for admins
    if (app()->isDownForMaintenance() && ! $user->isAdmin()) {
        return false; // Deny everything
    }

    return null; // Allow normal checks
});
```

**Expected Output:** During maintenance mode, non-admin users are denied all authorization checks. Admins continue to work normally.

**Why:** Returning `false` from `before()` denies the ability immediately. The check for `isAdmin()` ensures admins retain access .

### Real-World Cases

- **Super Admin Systems:** Global `before()` hook grants full access to super admins (Spatie recommends this approach) .
- **Audit Trails:** `after()` hooks log all authorization decisions for compliance.
- **Maintenance Mode:** `before()` hook denies all actions except for administrators.
- **Multi-Tenant:** `before()` hook denies access if the user's tenant is suspended.

### References

- Spatie Laravel Permission: Defining a Super-Admin — https://spatie.be/docs/laravel-permission/v3/basic-usage/super-admin
- Gate Class API (Laravel 7.x) — https://api.laravel.com/docs/7.x/Illuminate/Auth/Access/Gate.html
- Gate Contract API (Laravel 7.x) — https://api.laravel.com/docs/7.x/Illuminate/Contracts/Auth/Access/Gate.html

---

## 4. Passing Additional Context and Data Arguments into Gate Closures

### Definitions

**Core Definition:** Additional context and data arguments allow passing extra values (model instances, scalar values, arrays) into gate closures beyond the authenticated user, enabling context-sensitive authorization decisions.

**Technical Definition:** The second argument to `Gate::allows()`, `Gate::denies()`, `Gate::authorize()`, and `Gate::inspect()` can be a single value or an array. When an array is passed, the first element is used to determine the appropriate policy (if any), and the remaining elements are passed as additional parameters to the gate closure or policy method. For gate closures, all arguments after the user are passed directly .

**Beginner-Friendly Explanation:** Sometimes a permission check needs more than just "who is the user?" It might need to know "which post are they trying to edit?" or "what category is this?" You can pass these extra pieces of information as additional arguments, and your gate closure receives them so it can make a more informed decision.

### Purposes

- To pass model instances into gates for resource-specific checks.
- To provide scalar context (e.g., a category ID, a flag) to gate closures.
- To enable conditional logic based on multiple pieces of data.
- To support policy resolution when passing model instances to gates.

### Syntax Rules and Structure

**Complete General Syntax — Single Argument:**

```php
Gate::define('update-post', function (User $user, Post $post) {
    return $user->id === $post->user_id;
});

// Usage
Gate::allows('update-post', $post);
```

**Complete General Syntax — Multiple Arguments (Array):**

```php
Gate::define('update-post', function (User $user, Post $post, int $category) {
    return $user->id === $post->user_id && $category > 3;
});

// Usage — pass an array with all arguments
Gate::allows('update-post', [$post, $categoryId]);
```

**Complete General Syntax — Policy with Additional Arguments:**

```php
// PostPolicy
public function update(User $user, Post $post, int $category): bool
{
    return $user->id === $post->user_id && $category > 3;
}

// Controller — first element determines policy, rest passed as arguments
$this->authorize('update', [$post, $request->input('category')]);
```

**Component Breakdown:**

| Argument Position | Purpose |
|-------------------|---------|
| First argument (after user) | Model instance for policy resolution or primary context. |
| Second argument onward | Additional context passed to closure/policy method. |
| Array format | `[$model, $extra1, $extra2]` — all elements passed. |

**Syntax Rules:**
- For gates, all arguments after the user are passed directly to the closure.
- For policies, the first element of the array determines which policy to use; remaining elements are passed as method parameters.
- When passing a single model, it is used both for policy resolution and as the first additional argument.
- When passing an array, the first element is used for policy resolution.

**Constraints and Limitations:**
- **Policy resolution:** The first element of the array is used to determine the policy class. If you pass `[$post, $category]`, the `PostPolicy` is used.
- **Gate closures:** Gates do not have automatic model resolution; all arguments are passed directly to the closure.
- **`can()` method:** The `User::can()` method accepts the same arguments as the Gate facade.

### Multiple Annotated Complete Code Examples

**Example 1: Gate with Additional Scalar Argument**

```php
<?php
// Define a gate that checks category access level
Gate::define('edit-post', function (User $user, Post $post, int $requiredLevel) {
    return $user->id === $post->user_id && $user->access_level >= $requiredLevel;
});

// Usage — pass post and required level
$canEdit = Gate::allows('edit-post', [$post, 5]);
```

**Expected Output:** The user can edit the post only if they own it and their access level is 5 or higher.

**Why:** The array `[$post, 5]` passes both the post instance and the required access level to the gate closure. The closure receives `$user`, `$post`, and `$requiredLevel` .

---

**Example 2: Policy with Additional Context**

```php
<?php
// app/Policies/PostPolicy.php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function update(User $user, Post $post, int $category): bool
    {
        return $user->id === $post->user_id && $category > 3;
    }
}
```

```php
// Controller — pass array with post and category
public function update(Request $request, Post $post)
{
    $this->authorize('update', [$post, $request->input('category')]);

    $post->update($request->validated());

    return redirect()->route('posts.show', $post);
}
```

**Expected Output:** The post can be updated only if the user owns it and the category value is greater than 3.

**Why:** The array `[$post, $category]` tells Laravel to use `PostPolicy` (from the first element) and pass both the post and category to the `update` method. The policy method receives `$user`, `$post`, and `$category` .

---

**Example 3: Gate with Multiple Additional Arguments**

```php
<?php
Gate::define('assign-task', function (User $user, Project $project, User $assignee, string $priority) {
    // User must be project manager
    if ($user->id !== $project->manager_id) {
        return false;
    }

    // Assignee must be a project member
    if (! $project->members->contains($assignee)) {
        return false;
    }

    // High priority tasks require admin approval
    if ($priority === 'high' && ! $user->isAdmin()) {
        return false;
    }

    return true;
});

// Usage
$canAssign = Gate::allows('assign-task', [$project, $assignee, 'high']);
```

**Expected Output:** The task can be assigned only if the user is the project manager, the assignee is a project member, and (for high-priority tasks) the user is an admin.

**Why:** The array `[$project, $assignee, 'high']` passes three additional arguments to the gate closure. The closure receives `$user`, `$project`, `$assignee`, and `$priority` .

### Real-World Cases

- **Blog Editing:** Pass the post and the requested category to determine edit permissions.
- **Task Assignment:** Pass project, assignee, and priority to gate task assignment.
- **Multi-Tenant:** Pass tenant ID as additional context for tenant-scoped permissions.
- **E-commerce:** Pass order and discount code to gate discount application.

### References

- Laravel Authorization: Supplying Additional Context — https://github.com/driade/laravel-book/blob/6fe1474eee83cd64f124c2a1abca04323268f855/laravel-docs-6.x.pdf
- Laravel Authorization Documentation (12.x) — https://github.com/Vectorial1024/docs/blob/12.x/authorization.md

---

## References

- Gate Interface API (Laravel 7.x) — https://api.laravel.com/docs/7.x/Illuminate/Contracts/Auth/Access/Gate.html
- Gate Class API (Laravel 8.x) — https://api.laravel.com/docs/8.x/Illuminate/Auth/Access/Gate.html
- Gate Class API (Laravel 7.x) — https://api.laravel.com/docs/7.x/Illuminate/Auth/Access/Gate.html
- Gate Facade API (Laravel 8.x) — https://api.laravel.com/docs/8.x/Illuminate/Support/Facades/Gate.html
- Spatie Laravel Permission: Defining a Super-Admin — https://spatie.be/docs/laravel-permission/v3/basic-usage/super-admin
- Laravel Authorization Documentation (12.x) — https://github.com/Vectorial1024/docs/blob/12.x/authorization.md
- Laravel Authorization: Additional Context (Laravel 6.x) — https://github.com/driade/laravel-book/blob/6fe1474eee83cd64f124c2a1abca04323268f855/laravel-docs-6.x.pdf
- Laravel 13.x Authorization (Chinese) — https://laravel.net.cn/docs/13.x/authorization
- Laravel 12.x Authorization (Chinese) — https://laravel.net.cn/docs/12.x/authorization