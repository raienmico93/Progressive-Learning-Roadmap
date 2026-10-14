# Laravel Authorization: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Authorization is the framework's system for determining whether an authenticated user is permitted to perform a specific action or access a specific resource within an application.

**Technical Definition**

Laravel provides two primary authorization mechanisms: **Gates** (closure-based authorization for actions unrelated to a specific model) and **Policies** (class-based authorization organized around a particular model or resource). Both are built on the `Illuminate\Contracts\Auth\Access\Gate` contract and the `Illuminate\Auth\Access\Gate` implementation. Gates and policies can be defined, registered, and checked via the `Gate` facade, the `Authorizable` trait on the `User` model, middleware, controllers, and Blade directives. Authorization checks return boolean values or `Illuminate\Auth\Access\Response` instances, and failed checks throw `Illuminate\Auth\Access\AuthorizationException` (converted to 403 HTTP responses).

**Beginner-Friendly Explanation**

Authorization is about what you are allowed to do after you have logged in. Authentication tells the application who you are; authorization tells it what you can do. For example, you might be logged in as a regular user, but that does not mean you can delete another user's post. Gates and policies are the rules that decide these things. Think of gates as simple yes/no questions ("Can this user view the admin dashboard?") and policies as more organized rulebooks for specific things ("Here are all the rules for what a user can do with a blog post").

---

### Key Characteristics

- **Two Complementary Mechanisms**: Gates (closure-based, for model-agnostic actions) and Policies (class-based, for model-specific actions).
- **Facade-Based API**: The `Gate` facade provides a unified interface for defining and checking abilities.
- **User Model Integration**: The `Authorizable` trait adds `can()`, `cannot()`, and `cannotAny()` methods to the `User` model.
- **Blade Directives**: `@can`, `@cannot`, and `@canany` provide view-level authorization checks.
- **Before/After Hooks**: `Gate::before()` and `Gate::after()` allow global authorization logic (e.g., super-admin bypass, audit logging).
- **Policy Auto-Discovery**: Policies are automatically discovered based on naming conventions without manual registration.
- **Response Objects**: Gates and policies can return `Response` instances with custom denial messages.
- **Middleware Integration**: The `can` middleware can authorize routes without controller logic.
- **Controller Helpers**: The `AuthorizesRequests` trait provides `authorize()` and `authorizeResource()` methods.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A working authentication system** (session, Sanctum, or Passport) to provide an authenticated user.
- **Eloquent models** for policy-based authorization.
- **Basic understanding of PHP closures and classes**.

---

### Related Programming Areas

- **Authentication**: Authorization operates on the authenticated user provided by the authentication system.
- **Middleware**: The `can` middleware enforces authorization at the route level.
- **Blade Templating**: `@can`, `@cannot`, and `@canany` directives conditionally render content based on permissions.
- **Eloquent Models**: Policies are organized around Eloquent models.
- **Service Providers**: Gates and policy mappings are typically registered in `AppServiceProvider` or `AuthServiceProvider`.
- **API Resources**: Authorization determines which fields are included in API responses (via conditional attributes).

---

## Core Concepts

---

### 1. Authorization

#### Definitions

**Core Definition**

Authorization is the process of determining whether an authenticated user has permission to perform a specific action or access a specific resource.

**Technical Definition**

Laravel's authorization system is built on the `Gate` contract, which manages abilities (named actions) and their associated callbacks. Abilities can be defined as closures (gates) or as methods on policy classes. The `Gate::allows()`, `Gate::denies()`, and `Gate::authorize()` methods evaluate abilities for the currently authenticated user. The `Authorizable` trait on the `User` model provides `can()` and `cannot()` methods that delegate to the Gate.

**Beginner-Friendly Explanation**

Authorization is the set of rules that decides what a logged-in user is allowed to do. For example, a user might be allowed to edit their own posts but not someone else's. Laravel gives you tools to define these rules and check them easily.

#### Purposes

- To control access to application features and data based on user permissions.
- To prevent unauthorized users from performing sensitive actions.
- To centralize authorization logic in a maintainable, testable way.
- To provide consistent authorization checks across controllers, views, and APIs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Gate;

// Define a gate
Gate::define('update-post', function (User $user, Post $post) {
    return $user->id === $post->user_id;
});

// Check authorization
if (Gate::allows('update-post', $post)) {
    // Authorized
}

if (Gate::denies('update-post', $post)) {
    // Denied
}

Gate::authorize('update-post', $post); // Throws AuthorizationException if denied
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Gate::define()` | Defines an ability with a closure. |
| `Gate::allows()` | Checks if the current user is authorized. |
| `Gate::denies()` | Checks if the current user is denied. |
| `Gate::authorize()` | Authorizes or throws an exception. |

**Syntax Rules**

- Gates are typically defined in `AppServiceProvider::boot()`.
- Gates receive the authenticated user as the first argument.
- Additional arguments (e.g., a model) are passed after the user.
- `Gate::authorize()` throws `AuthorizationException` if denied.

**Constraints and Limitations**

- Gates are best for actions not tied to a specific model.
- For model-specific actions, use policies instead.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Gate Definition and Check**

**Service Provider**

```php
<?php
// app/Providers/AppServiceProvider.php
namespace App\Providers;

use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });
    }
}
```

**Controller**

```php
public function update(Request $request, Post $post)
{
    Gate::authorize('update-post', $post);
    // Update the post...
    return redirect('/posts');
}
```

**Expected Output**

```
// If user owns post → update proceeds
// If user does not own post → 403 Forbidden
```

**Why This Output Occurs**

`Gate::authorize()` evaluates the `update-post` gate with the authenticated user and the post. If the user's ID matches the post's `user_id`, the action is authorized; otherwise, a 403 response is returned.

---

#### Real-World Cases with Explanation

**Case 1: Admin Dashboard Access**

A gate `access-admin` checks if the user has an `is_admin` flag. Only administrators can view the admin dashboard.

**Explanation**: Gates are ideal for simple, model-agnostic checks.

---

### 2. Gates

#### Definitions

**Core Definition**

A gate is a closure-based authorization check that determines if a user is authorized to perform a given action.

**Technical Definition**

Gates are defined via `Gate::define($ability, $callback)` where `$callback` receives the authenticated user and optional additional arguments. Gates are stored in the `Gate` instance's `$abilities` array. When checked, the gate callback is invoked, and its boolean (or `Response`) result determines authorization.

**Beginner-Friendly Explanation**

A gate is a simple yes/no question you define. For example, "Can this user view the admin panel?" You write the rule, and Laravel checks it whenever you ask.

#### Purposes

- To define authorization rules for actions not tied to a specific model.
- To provide a simple, closure-based approach to authorization.
- To support quick, ad-hoc authorization checks.
- To serve as a learning tool before adopting policies.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Gate::define('ability-name', function (User $user, $additional = null) {
    return true; // or false
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `'ability-name'` | The name of the ability. |
| `$user` | The authenticated user (injected automatically). |
| `$additional` | Optional additional arguments (e.g., a model). |

**Syntax Rules**

- Gates are defined in a service provider's `boot()` method.
- The callback must return a boolean or `Response`.
- Additional arguments can be passed as an array.
- Gates can also be defined with `Class@method` strings.

**Constraints and Limitations**

- Gates cannot be auto-discovered; they must be defined explicitly.
- Gates are best for model-agnostic actions; use policies for model-specific logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Gate with Additional Context**

**Step 1: Define the Gate**
`app/Providers/AuthServiceProvider.php`
```php
// AppServiceProvider.php

use App\Models\User;
use App\Models\Category;
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    // Define the gate with two extra arguments: $category and $extraFlag
    Gate::define('create-post', function (User $user, Category $category, bool $extraFlag) {
        return $user->hasRole('editor') && $category->is_active && $extraFlag;
    });
}
```
\
`app/Http/Controllers/PostController.php`
**Step 2: Prepare Your Controller**
```php
namespace App\Http\Controllers;

use App\Models\Category;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    public function store(Request $request, Category $category)
    {
        // 1. Define your extra context flag
        $extraFlag = $request->input('agree_to_terms', false); 

        // 2. Check the gate by passing context variables inside an array
        if (Gate::allows('create-post', [$category, $extraFlag])) {
            // The user is authorized, Put your post creation logic here...
            
            return response()->json(['message' => 'Post created successfully!']);
        }

        // 3. Handle unauthorized actions
        abort(403, 'You are not authorized to create a post in this category.');
    }
}
```
\
`resources/views/posts/create.blade.php`
**Step 3: (Optional) Use it in Blade Templates**
```blade
@can('create-post', [$category, true])
    <!-- This button only shows if the user is an editor, the category is active, and the flag is true -->
    <button type="submit">Publish Post</button>
@else
    <p>You do not have permission to publish here.</p>
@endcan
```

**Expected Output**

- Authorized only if the user is an editor, the category is active, and $extraFlag is true

**Why This Output Occurs**

The gate receives the user, the category, and the extra flag. All conditions must be true for authorization.

---

**Example 2: Gate Returning a Response Object**

```php
use Illuminate\Auth\Access\Response;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('You must be a super administrator.');
});
```

**Expected Output**

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // Authorized
} else {
    echo $response->message(); // "You must be a super administrator."
}
```

**Why This Output Occurs**

The gate returns a `Response` object, which `Gate::inspect()` can interrogate for both the authorization result and a custom denial message.

---

#### Real-World Cases with Explanation

**Case 1: Feature Flag Access**

A gate `view-beta-feature` checks if the user is on a whitelist of beta testers.

**Explanation**: Gates provide a lightweight way to implement feature flags and access controls.

---

### 3. Policies

#### Definitions

**Core Definition**

A policy is a class that organizes authorization logic around a particular model or resource, with methods corresponding to actions (view, create, update, delete, etc.).

**Technical Definition**

Policies are classes typically placed in `app/Policies` that define methods for each action they authorize. Each method receives the authenticated user and optionally a model instance, and returns a boolean or `Response`. Policies are auto-discovered based on naming conventions (e.g., `User` model → `UserPolicy`) or explicitly registered via `Gate::policy()` or the `$policies` array.

**Beginner-Friendly Explanation**

A policy is like a rulebook for a specific thing in your application. For example, a `PostPolicy` contains all the rules about what users can do with blog posts: who can view them, who can edit them, who can delete them.

#### Purposes

- To group authorization logic for a specific model in one place.
- To provide a structured, class-based approach to authorization.
- To enable auto-discovery and reduce registration boilerplate.
- To integrate seamlessly with controllers, Blade, and middleware.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Policies/PostPolicy.php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
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
        return $user->hasRole('editor');
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `viewAny(User $user)` | Can the user list all models? |
| `view(User $user, Post $post)` | Can the user view a specific model? |
| `create(User $user)` | Can the user create a new model? |
| `update(User $user, Post $post)` | Can the user update a specific model? |
| `delete(User $user, Post $post)` | Can the user delete a specific model? |

**Syntax Rules**

- Policy methods must return a boolean or `Response`.
- Methods receive the authenticated user as the first argument.
- Model-specific methods receive the model instance as the second argument.
- `viewAny` and `create` do not receive a model instance.

**Constraints and Limitations**

- Policy methods should be simple and delegate complex logic to services or the model.
- Policies cannot be used for model-agnostic actions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Policy**

**Policy**

```php
<?php
// app/Policies/PostPolicy.php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

**Controller**

```php
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);
    // Update the post...
}
```

**Expected Output**

```
// If user owns post → update proceeds
// If user does not own post → 403 Forbidden
```

**Why This Output Occurs**

The `authorize()` method resolves the `PostPolicy` and calls its `update` method with the user and post. If the user's ID matches the post's `user_id`, authorization succeeds.

---

#### Real-World Cases with Explanation

**Case 1: Blog Post Management**

A `PostPolicy` defines who can create, edit, and delete posts. Editors can create posts; authors can edit their own posts; admins can delete any post.

**Explanation**: Policies centralize model-specific authorization logic, making it easy to maintain and test.

---

### 4. Policy Creation

#### Definitions

**Core Definition**

Policy creation is the process of generating a new policy class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:policy` command generates a PHP class in `app/Policies`. The `--model=Post` option generates example CRUD method stubs (`viewAny`, `view`, `create`, `update`, `delete`). If the `app/Policies` directory does not exist, Laravel creates it automatically.

**Beginner-Friendly Explanation**

Creating a policy is a one-command process. Laravel makes the file, and you fill in the rules.

#### Purposes

- To automate the generation of policy class boilerplate.
- To ensure policies are placed in the correct directory and namespace.
- To generate example CRUD methods for common actions.
- To reduce manual file creation errors.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:policy PostPolicy
php artisan make:policy PostPolicy --model=Post
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:policy PostPolicy` | Creates an empty policy class. |
| `--model=Post` | Generates CRUD method stubs for the `Post` model. |

**Syntax Rules**

- The policy name should be PascalCase and end with `Policy`.
- The `--model` option is optional but recommended.
- Generated files must be edited to add logic.

**Constraints and Limitations**

- The command does not register the policy automatically.
- Generated stubs must be customized.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Policy with Model**

**Command**

```bash
php artisan make:policy PostPolicy --model=Post
```

**Generated File**

```php
<?php
// app/Policies/PostPolicy.php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function viewAny(User $user): bool { }
    public function view(User $user, Post $post): bool { }
    public function create(User $user): bool { }
    public function update(User $user, Post $post): bool { }
    public function delete(User $user, Post $post): bool { }
    public function restore(User $user, Post $post): bool { }
    public function forceDelete(User $user, Post $post): bool { }
}
```

**Expected Output (Terminal)**

```
INFO  Policy [app/Policies/PostPolicy.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the policy with example method stubs based on the `Post` model.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Policy Generation**

A developer runs `php artisan make:policy UserPolicy --model=User` and immediately fills in the authorization rules for viewing, creating, updating, and deleting users.

**Explanation**: Artisan accelerates development by generating appropriate policy stubs.

---

### 5. Policy Registration

#### Definitions

**Core Definition**

Policy registration is the process of informing Laravel which policy class should be used when authorizing actions against a given model type.

**Technical Definition**

Policies can be registered in three ways: (1) auto-discovery based on naming conventions (the default in Laravel 11+), (2) the `$policies` array in `AuthServiceProvider` (Laravel 10 and earlier), or (3) explicit registration via `Gate::policy(Post::class, PostPolicy::class)`. Auto-discovery requires the policy to be in a `Policies` directory at or above the directory containing the model, with a name matching `{Model}Policy`.

**Beginner-Friendly Explanation**

Registering a policy tells Laravel "when I ask about permissions for a Post, use the PostPolicy class." In modern Laravel, this usually happens automatically.

#### Purposes

- To map a model to its corresponding policy.
- To enable Laravel to resolve the correct policy during authorization checks.
- To support custom policy naming or locations.
- To override auto-discovery when conventions do not fit.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Auto-discovery (no registration needed)
// Model: app/Models/Post.php
// Policy: app/Policies/PostPolicy.php

// Explicit registration (AppServiceProvider or AuthServiceProvider)
use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

Gate::policy(Post::class, PostPolicy::class);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| Auto-discovery | Convention-based, no registration required. |
| `$policies` array | Maps models to policies in `AuthServiceProvider`. |
| `Gate::policy()` | Explicitly registers a policy. |

**Syntax Rules**

- Auto-discovery requires `app/Policies` at or above the model directory.
- The policy name must be `{ModelName}Policy`.
- Explicit registration overrides auto-discovery.
- `$policies` array is used in Laravel 10 and earlier.

**Constraints and Limitations**

- Auto-discovery fails if the policy is in a non-standard location.
- Explicit registration is required for custom naming.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Auto-Discovery**

**Model:** `app/Models/Post.php`
**Policy:** `app/Policies/PostPolicy.php`

**Expected Output**

```
// Laravel automatically discovers PostPolicy for Post model
// No registration needed
```

**Why This Output Occurs**

The policy follows the standard naming convention and location.

---

**Example 2: Explicit Registration**

```php
// app/Providers/AppServiceProvider.php
use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::policy(Post::class, PostPolicy::class);
}
```

**Expected Output**

```
// PostPolicy is explicitly registered for the Post model
```

**Why This Output Occurs**

`Gate::policy()` manually maps the model to the policy, overriding auto-discovery.

---

#### Real-World Cases with Explanation

**Case 1: Non-Standard Policy Location**

A project stores policies in `app/Authorization/Policies`. Auto-discovery fails, so the developer explicitly registers each policy via `Gate::policy()`.

**Explanation**: Explicit registration is necessary when conventions are not followed.

---

### 6. Policy Methods

#### Definitions

**Core Definition**

Policy methods are the public methods on a policy class that correspond to specific actions (viewAny, view, create, update, delete, etc.) and return whether the user is authorized.

**Technical Definition**

Laravel recognizes several conventional method names: `viewAny`, `view`, `create`, `update`, `delete`, `restore`, and `forceDelete`. Custom methods can also be defined. Each method receives the authenticated user as the first argument; model-specific methods receive the model instance as the second argument. Methods must return a boolean or `Response`.

**Beginner-Friendly Explanation**

Policy methods are the individual rules inside a policy. For example, the `update` method answers "Can this user update this post?"

#### Purposes

- To define authorization rules for each action on a model.
- To provide a clear, readable mapping between actions and permissions.
- To enable fine-grained control over who can do what.
- To support custom actions beyond CRUD.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
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
    return $user->hasRole('editor');
}

public function update(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}

public function delete(User $user, Post $post): bool
{
    return $user->id === $post->user_id || $user->isAdmin();
}
```

**Component Breakdown**

| Method | Model Argument? | Typical Use |
|--------|----------------|-------------|
| `viewAny` | No | List all models |
| `view` | Yes | View a specific model |
| `create` | No | Create a new model |
| `update` | Yes | Update a specific model |
| `delete` | Yes | Delete a specific model |
| `restore` | Yes | Restore a soft-deleted model |
| `forceDelete` | Yes | Permanently delete a model |

**Syntax Rules**

- Methods must return `bool` or `Response`.
- `viewAny` and `create` receive only the user.
- Other methods receive the user and the model.
- Custom method names can be used for non-CRUD actions.

**Constraints and Limitations**

- Policy methods should not contain complex business logic; delegate to services.
- Methods should not throw exceptions; return `false` instead.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Policy Method**

```php
public function publish(User $user, Post $post): bool
{
    return $user->hasRole('editor') && $post->is_draft;
}
```

**Controller**

```php
$this->authorize('publish', $post);
```

**Expected Output**

```
// Authorized only if user is an editor and post is a draft
```

**Why This Output Occurs**

The `publish` method checks both the user's role and the post's status.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Management**

An `OrderPolicy` defines `view`, `update`, and `cancel` methods. Customers can view and cancel their own orders; admins can update any order.

**Explanation**: Policy methods map directly to the actions available in the application.

---

### 7. Policy Filters

#### Definitions

**Core Definition**

Policy filters are special `before` methods on a policy class that run before any other policy method, allowing certain users (e.g., super-admins) to bypass normal authorization checks.

**Technical Definition**

If a policy defines a `before` method, it is called before the intended policy method. The `before` method receives the authenticated user and the ability name, and can return `true` (authorize), `false` (deny), or `null` (fall through to the normal policy method). This is commonly used for super-admin bypass.

**Beginner-Friendly Explanation**

A policy filter is like a VIP pass. If you are a super-admin, the `before` method lets you skip all the normal rules and do anything.

#### Purposes

- To allow super-admins or specific roles to bypass all policy checks.
- To implement global denial rules for certain users.
- To centralize cross-cutting authorization logic.
- To avoid repeating super-admin checks in every policy method.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function before(User $user, string $ability): ?bool
{
    if ($user->isSuperAdmin()) {
        return true;
    }

    return null; // Fall through to the policy method
}
```

**Component Breakdown**

| Return Value | Effect |
|-------------|--------|
| `true` | Authorize the action. |
| `false` | Deny the action. |
| `null` | Fall through to the normal policy method. |

**Syntax Rules**

- The `before` method receives the user and the ability name.
- It can be defined on individual policies or globally via `Gate::before()`.
- Returning `null` allows the normal policy method to run.

**Constraints and Limitations**

- The `before` method is not called if the policy class does not contain a method matching the ability name.
- Global `Gate::before()` callbacks affect all policies.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Super-Admin Bypass**

```php
public function before(User $user, string $ability): ?bool
{
    if ($user->hasRole('Super Admin')) {
        return true;
    }

    return null;
}
```

**Expected Output**

```
// Super Admin → all actions authorized
// Other users → normal policy checks apply
```

**Why This Output Occurs**

The `before` method returns `true` for super-admins, bypassing the normal policy methods.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant SaaS Admin**

A SaaS platform has a super-admin role that can manage all tenants. The `before` method grants the super-admin full access to all policies.

**Explanation**: Policy filters centralize super-admin logic, avoiding repetition across dozens of policy methods.

---

### 8. Policy Auto-Discovery

#### Definitions

**Core Definition**

Policy auto-discovery is Laravel's automatic resolution of policy classes based on naming conventions and directory structure, eliminating the need for manual registration.

**Technical Definition**

Laravel auto-discovers policies when the policy class is in a `Policies` directory at or above the directory containing the model, and the policy name matches `{ModelName}Policy`. For example, a `User` model in `app/Models` is matched with a `UserPolicy` in `app/Policies`. Auto-discovery can be customized via `Gate::guessPolicyNamesUsing()`.

**Beginner-Friendly Explanation**

Policy auto-discovery means you do not have to tell Laravel which policy goes with which model. If you follow the naming rules, Laravel figures it out automatically.

#### Purposes

- To reduce boilerplate by eliminating manual policy registration.
- To enforce consistent naming and directory conventions.
- To simplify project setup and maintenance.
- To allow customization when conventions do not fit.

#### Syntax Rules and Structure

**Complete General Syntax**

```
app/
├── Models/
│   └── Post.php
└── Policies/
    └── PostPolicy.php
```

**Component Breakdown**

| Requirement | Description |
|-------------|-------------|
| Directory | `Policies` at or above the model directory. |
| Naming | `{ModelName}Policy`. |
| Namespace | Policy must be in the correct namespace. |

**Syntax Rules**

- Auto-discovery works out of the box for standard structures.
- Explicit registration overrides auto-discovery.
- `Gate::guessPolicyNamesUsing()` customizes discovery logic.

**Constraints and Limitations**

- Auto-discovery fails if the policy is in a non-standard location.
- Nested model directories require policies in matching nested `Policies` directories.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Standard Auto-Discovery**

**Model:** `app/Models/User.php`
**Policy:** `app/Policies/UserPolicy.php`

**Expected Output**

```
// Laravel automatically uses UserPolicy for User model
// No registration needed
```

**Why This Output Occurs**

The policy follows the naming convention and is in the standard location.

---

**Example 2: Custom Discovery Logic**

```php
// AppServiceProvider.php
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::guessPolicyNamesUsing(function (string $modelClass) {
        return str_replace('Models', 'Policies', $modelClass) . 'Policy';
    });
}
```

**Expected Output**

```
// Custom mapping: App\Models\Post → App\Policies\PostPolicy
```

**Why This Output Occurs**

`guessPolicyNamesUsing()` overrides the default discovery logic.

---

#### Real-World Cases with Explanation

**Case 1: Modular Application**

A modular application places models and policies in domain-specific directories. Custom discovery logic ensures policies are found.

**Explanation**: Auto-discovery with customization accommodates non-standard project structures.

---

### 9. `authorize()`

#### Definitions

**Core Definition**

The `authorize()` method is a controller helper (provided by the `AuthorizesRequests` trait) that authorizes an action and throws an `AuthorizationException` (rendering a 403 response) if the user is not authorized.

**Technical Definition**

`$this->authorize($ability, $arguments)` resolves the appropriate gate or policy and throws `Illuminate\Auth\Access\AuthorizationException` if denied. The exception is automatically converted to a 403 HTTP response by Laravel's exception handler. The `authorizeResource()` method authorizes all actions of a resource controller in a single call.

**Beginner-Friendly Explanation**

`authorize()` is a shortcut that checks permission and automatically throws an error if the user is not allowed. It keeps your controller code clean.

#### Purposes

- To enforce authorization before performing an action.
- To throw a consistent 403 error when authorization fails.
- To reduce boilerplate by automatically handling denial responses.
- To integrate with resource controllers via `authorizeResource()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Authorize a single action
$this->authorize('update', $post);

// Authorize a resource controller
public function __construct()
{
    $this->authorizeResource(Post::class, 'post');
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `authorize($ability, $arguments)` | Authorizes a single action. |
| `authorizeResource($model, $parameter)` | Authorizes all resource actions. |

**Syntax Rules**

- The `AuthorizesRequests` trait must be used by the controller.
- The second argument can be a model instance or an array.
- `authorizeResource()` maps controller methods to policy methods.

**Constraints and Limitations**

- `authorize()` throws an exception; use `Gate::allows()` for boolean checks.
- The `AuthorizesRequests` trait is included in the base `Controller` class by default.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Authorizing a Single Action**

```php
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);
    // Update the post...
}
```

**Expected Output**

```
// If authorized → update proceeds
// If denied → 403 Forbidden
```

**Why This Output Occurs**

`authorize()` resolves the `PostPolicy` and calls `update`. If denied, it throws `AuthorizationException`, which Laravel converts to a 403 response.

---

**Example 2: Resource Controller Authorization**

```php
public function __construct()
{
    $this->authorizeResource(Post::class, 'post');
}
```

**Expected Output**

```
// index → viewAny, show → view, store → create, update → update, destroy → delete
```

**Why This Output Occurs**

`authorizeResource()` automatically maps each resource controller method to the corresponding policy method.

---

#### Real-World Cases with Explanation

**Case 1: Blog Post CRUD**

A `PostController` uses `authorizeResource(Post::class, 'post')` in its constructor. Every CRUD action is automatically authorized against the `PostPolicy`.

**Explanation**: `authorizeResource()` eliminates repetitive authorization code in every controller method.

---

### 10. `can()`

#### Definitions

**Core Definition**

The `can()` method is available on the `User` model (via the `Authorizable` trait) and checks if the user is authorized to perform a given action, returning a boolean.

**Technical Definition**

`$user->can($ability, $arguments)` delegates to the `Gate::allows()` method with the user specified via `Gate::forUser()`. If a policy is registered for the model, `can()` calls the appropriate policy method; otherwise, it falls back to closure-based gates.

**Beginner-Friendly Explanation**

`can()` is a simple way to check if a user is allowed to do something. It returns `true` or `false`.

#### Purposes

- To check authorization in a boolean context (without throwing exceptions).
- To conditionally display UI elements based on permissions.
- To perform authorization checks in controllers, views, or other services.
- To provide a user-centric API for authorization.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
if ($user->can('update', $post)) {
    // Authorized
}

if ($user->cannot('delete', $post)) {
    // Denied
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `can($ability, $arguments)` | Returns `true` if authorized. |
| `cannot($ability, $arguments)` | Returns `true` if denied. |

**Syntax Rules**

- `can()` is provided by the `Authorizable` trait.
- The second argument can be a model instance or an array.
- `can()` returns a boolean.

**Constraints and Limitations**

- `can()` does not throw exceptions; use `authorize()` for automatic 403 responses.
- `can()` always returns `false` for unauthenticated users.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Checking in a Controller**

```php
public function index(Request $request)
{
    if ($request->user()->can('viewAny', Post::class)) {
        return view('posts.index', ['posts' => Post::all()]);
    }

    abort(403);
}
```

**Expected Output**

```
// If authorized → posts index view
// If denied → 403 Forbidden
```

**Why This Output Occurs**

`can()` checks the `viewAny` policy method for the `Post` model. If authorized, the view is returned; otherwise, a 403 is thrown.

---

#### Real-World Cases with Explanation

**Case 1: Conditional Navigation**

A Blade navigation menu uses `@can` (which delegates to `can()`) to show the "Admin Panel" link only to users with administrative privileges.

**Explanation**: `can()` enables clean, conditional UI rendering based on permissions.

---

### 11. `cannot()`

#### Definitions

**Core Definition**

The `cannot()` method is the inverse of `can()`, returning `true` if the user is denied the given action.

**Technical Definition**

`$user->cannot($ability, $arguments)` returns the negation of `$user->can($ability, $arguments)`. It is provided by the `Authorizable` trait and delegates to `Gate::denies()`.

**Beginner-Friendly Explanation**

`cannot()` checks if a user is not allowed to do something. It is the opposite of `can()`.

#### Purposes

- To check denial in a boolean context.
- To conditionally display content when a user lacks permission.
- To provide a readable alternative to `!$user->can(...)`.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
if ($user->cannot('delete', $post)) {
    // User is denied
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `cannot($ability, $arguments)` | Returns `true` if denied. |

**Syntax Rules**

- `cannot()` is the inverse of `can()`.
- Accepts the same arguments as `can()`.

**Constraints and Limitations**

- Same as `can()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Denial Check**

```php
if ($user->cannot('delete', $post)) {
    return response()->json(['error' => 'You cannot delete this post.'], 403);
}
```

**Expected Output**

```
// If denied → 403 with error message
// If authorized → continues
```

**Why This Output Occurs**

`cannot()` returns `true` when the user lacks permission, allowing conditional denial handling.

---

#### Real-World Cases with Explanation

**Case 1: Upgrade Notice**

A Blade view uses `@cannot('create-posts')` to display an "Upgrade your account" notice to users who do not have permission to create posts.

**Explanation**: `cannot()` enables targeted messaging for users lacking permissions.

---

### 12. `Gate::allows()`

#### Definitions

**Core Definition**

`Gate::allows()` checks if the currently authenticated user is authorized to perform a given ability, returning a boolean.

**Technical Definition**

`Gate::allows($ability, $arguments)` evaluates the ability against the currently authenticated user. It returns `true` if authorized, `false` otherwise. If no user is authenticated, it returns `false`.

**Beginner-Friendly Explanation**

`Gate::allows()` asks "Is the current user allowed to do this?" and gives a yes/no answer.

#### Purposes

- To check authorization in a boolean context using the Gate facade.
- To perform authorization checks without a `User` instance.
- To integrate with controllers, middleware, and services.
- To provide a consistent API for both gates and policies.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
if (Gate::allows('update-post', $post)) {
    // Authorized
}

if (Gate::allows('access-admin')) {
    // Authorized
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Gate::allows($ability, $arguments)` | Returns `true` if authorized. |

**Syntax Rules**

- The currently authenticated user is used automatically.
- Arguments can be a model instance or an array.
- Returns `false` if no user is authenticated.

**Constraints and Limitations**

- `Gate::allows()` does not throw exceptions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Gate Check in a Controller**

```php
if (Gate::allows('update-post', $post)) {
    // Update the post...
} else {
    abort(403);
}
```

**Expected Output**

```
// If authorized → update proceeds
// If denied → 403 Forbidden
```

**Why This Output Occurs**

`Gate::allows()` evaluates the `update-post` gate with the authenticated user and the post.

---

#### Real-World Cases with Explanation

**Case 1: Feature Gating**

A middleware checks `Gate::allows('access-beta')` before allowing access to a beta feature route.

**Explanation**: `Gate::allows()` provides a clean way to enforce feature flags.

---

### 13. `Gate::denies()`

#### Definitions

**Core Definition**

`Gate::denies()` is the inverse of `Gate::allows()`, returning `true` if the currently authenticated user is denied the given ability.

**Technical Definition**

`Gate::denies($ability, $arguments)` returns the negation of `Gate::allows()`. It is provided by the `Gate` facade and is useful for readable denial checks.

**Beginner-Friendly Explanation**

`Gate::denies()` asks "Is the current user not allowed to do this?"

#### Purposes

- To check denial in a boolean context.
- To provide a readable alternative to `!Gate::allows(...)`.
- To conditionally handle unauthorized access.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
if (Gate::denies('update-post', $post)) {
    abort(403);
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Gate::denies($ability, $arguments)` | Returns `true` if denied. |

**Syntax Rules**

- Same arguments as `Gate::allows()`.
- Returns `true` if no user is authenticated.

**Constraints and Limitations**

- Same as `Gate::allows()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Denial Check**

```php
if (Gate::denies('update-post', $post)) {
    return redirect()->back()->with('error', 'Unauthorized');
}
```

**Expected Output**

```
// If denied → redirect with error
// If authorized → continues
```

**Why This Output Occurs**

`Gate::denies()` returns `true` when the user lacks permission, triggering the redirect.

---

#### Real-World Cases with Explanation

**Case 1: Middleware Authorization**

A custom middleware uses `Gate::denies()` to check if a user can access a route, redirecting unauthorized users.

**Explanation**: `Gate::denies()` provides a clean denial-check pattern.

---

### 14. `Gate::authorize()`

#### Definitions

**Core Definition**

`Gate::authorize()` authorizes an action and throws an `AuthorizationException` if the user is not authorized.

**Technical Definition**

`Gate::authorize($ability, $arguments)` evaluates the ability and throws `Illuminate\Auth\Access\AuthorizationException` if denied. Laravel's exception handler converts this to a 403 HTTP response. It is the facade equivalent of the controller's `authorize()` method.

**Beginner-Friendly Explanation**

`Gate::authorize()` checks permission and automatically throws an error if the user is not allowed. It keeps your code clean by handling denial automatically.

#### Purposes

- To enforce authorization with automatic 403 responses.
- To reduce boilerplate by eliminating manual denial handling.
- To provide a consistent authorization pattern across the application.
- To integrate with Laravel's exception handling system.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Gate::authorize('update-post', $post);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Gate::authorize($ability, $arguments)` | Authorizes or throws. |

**Syntax Rules**

- Throws `AuthorizationException` if denied.
- The exception is automatically converted to a 403 response.
- Accepts the same arguments as `Gate::allows()`.

**Constraints and Limitations**

- `Gate::authorize()` always throws on denial; use `Gate::allows()` for boolean checks.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Authorizing in a Controller**

```php
public function update(Request $request, Post $post)
{
    Gate::authorize('update-post', $post);
    // Update the post...
}
```

**Expected Output**

```
// If authorized → update proceeds
// If denied → 403 Forbidden
```

**Why This Output Occurs**

`Gate::authorize()` throws if the user is not authorized, and Laravel converts the exception to a 403 response.

---

#### Real-World Cases with Explanation

**Case 1: API Authorization**

An API controller uses `Gate::authorize()` to ensure only authorized users can update resources. Failed checks return 403 JSON responses.

**Explanation**: `Gate::authorize()` provides consistent authorization for API endpoints.

---

### 15. `Gate::forUser()`

#### Definitions

**Core Definition**

`Gate::forUser()` allows authorization checks to be performed for a specific user other than the currently authenticated user.

**Technical Definition**

`Gate::forUser($user)` returns a new `Gate` instance bound to the given user. Subsequent calls to `allows()`, `denies()`, or `authorize()` on that instance use the specified user instead of the authenticated user.

**Beginner-Friendly Explanation**

`Gate::forUser()` lets you check permissions for someone else. For example, an admin might want to check if another user can edit a post.

#### Purposes

- To check authorization for a user other than the currently authenticated one.
- To support administrative tools that display or modify other users' permissions.
- To enable testing of authorization logic with specific users.
- To provide flexibility in authorization checks.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // The specified user can update the post
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // The specified user cannot update the post
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Gate::forUser($user)` | Returns a Gate instance for the specified user. |
| `->allows()` | Checks authorization for that user. |

**Syntax Rules**

- `forUser()` accepts a `User` model instance.
- The returned Gate instance is scoped to that user.
- Can be chained with `allows()`, `denies()`, or `authorize()`.

**Constraints and Limitations**

- `forUser()` does not change the authenticated user for the current request.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Admin Checking Another User's Permissions**

```php
$otherUser = User::find(5);

if (Gate::forUser($otherUser)->allows('update-post', $post)) {
    // User 5 can update the post
}
```

**Expected Output**

```
// Returns true if user 5 is authorized, false otherwise
```

**Why This Output Occurs**

`forUser()` binds the Gate to user 5, and `allows()` evaluates the `update-post` gate for that user.

---

#### Real-World Cases with Explanation

**Case 1: Admin User Management**

An admin panel allows administrators to impersonate other users and view the application as they would see it. `Gate::forUser()` is used to check permissions for the impersonated user.

**Explanation**: `forUser()` enables authorization checks for any user, not just the authenticated one.

---

### 16. `Gate::before()`

#### Definitions

**Core Definition**

`Gate::before()` registers a callback that runs before all other authorization checks, allowing global authorization logic such as super-admin bypass.

**Technical Definition**

`Gate::before(callable $callback)` adds a callback to the Gate's `beforeCallbacks` array. The callback receives the authenticated user and the ability name. If it returns a non-null value, that value is used as the authorization result, and the normal gate/policy check is skipped. If it returns `null`, authorization proceeds normally.

**Beginner-Friendly Explanation**

`Gate::before()` is a global "first check" that runs before any other permission check. It is commonly used to let super-admins do anything.

#### Purposes

- To implement super-admin bypass globally.
- To centralize global authorization rules.
- To avoid repeating the same check in every gate and policy.
- To implement global denial rules (e.g., banned users).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// AppServiceProvider::boot()
Gate::before(function (User $user, string $ability) {
    if ($user->isSuperAdmin()) {
        return true;
    }

    return null; // Fall through
});
```

**Component Breakdown**

| Return Value | Effect |
|-------------|--------|
| `true` | Authorize the action (skip normal checks). |
| `false` | Deny the action (skip normal checks). |
| `null` | Proceed with normal checks. |

**Syntax Rules**

- Registered via `Gate::before()` in a service provider.
- Receives the authenticated user and the ability name.
- Returning non-null short-circuits the authorization process.

**Constraints and Limitations**

- Global `before` callbacks affect all gates and policies.
- Returning `false` denies all actions for matching users.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Super-Admin Bypass**

```php
// AppServiceProvider.php
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::before(function (User $user, string $ability) {
        if ($user->hasRole('Super Admin')) {
            return true;
        }

        return null;
    });
}
```

**Expected Output**

```
// Super Admin → all actions authorized
// Other users → normal authorization checks apply
```

**Why This Output Occurs**

The `before` callback returns `true` for super-admins, bypassing all other authorization checks.

---

#### Real-World Cases with Explanation

**Case 1: SaaS Super-Admin**

A multi-tenant SaaS platform has a super-admin role that can manage all tenants. A global `before` callback grants the super-admin full access to all gates and policies.

**Explanation**: `Gate::before()` centralizes super-admin bypass logic, keeping individual policies clean.

---

### 17. `Gate::after()`

#### Definitions

**Core Definition**

`Gate::after()` registers a callback that runs after all other authorization checks, allowing post-authorization logic such as audit logging.

**Technical Definition**

`Gate::after(callable $callback)` adds a callback to the Gate's `afterCallbacks` array. The callback receives the authenticated user, the ability name, the result, and the arguments. It **cannot modify** the authorization result; it is used for side effects such as logging.

**Beginner-Friendly Explanation**

`Gate::after()` is a "last step" that runs after a permission check. It cannot change the result, but it can do things like log who was authorized or denied.

#### Purposes

- To log authorization decisions for audit purposes.
- To perform side effects after authorization checks.
- To monitor authorization patterns for security analysis.
- To integrate authorization with logging or analytics systems.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Gate::after(function (User $user, string $ability, bool $result, array $arguments) {
    Log::info("User {$user->id} checked {$ability}: " . ($result ? 'allowed' : 'denied'));
});
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `$user` | The authenticated user. |
| `$ability` | The ability name. |
| `$result` | The authorization result (`true` or `false`). |
| `$arguments` | The arguments passed to the check. |

**Syntax Rules**

- Registered via `Gate::after()` in a service provider.
- Cannot modify the authorization result.
- Used for side effects only.

**Constraints and Limitations**

- The result is read-only.
- After callbacks run for every authorization check, which may impact performance.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Authorization Audit Logging**

```php
// AppServiceProvider.php
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\Facades\Log;

public function boot(): void
{
    Gate::after(function (User $user, string $ability, bool $result, array $arguments) {
        Log::info('Authorization check', [
            'user_id' => $user->id,
            'ability' => $ability,
            'result' => $result,
        ]);
    });
}
```

**Expected Output (log file)**

```
[2026-10-11 12:00:00] local.INFO: Authorization check {"user_id":1,"ability":"update-post","result":true}
```

**Why This Output Occurs**

The `after` callback logs every authorization check with the user ID, ability, and result.

---

#### Real-World Cases with Explanation

**Case 1: Compliance Audit Trail**

A regulated application logs every authorization decision to maintain a compliance audit trail. `Gate::after()` captures all checks without modifying authorization logic.

**Explanation**: `Gate::after()` is ideal for audit logging and monitoring.

---

### 18. `Gate::define()`

#### Definitions

**Core Definition**

`Gate::define()` registers a new ability (gate) with a callback that determines authorization.

**Technical Definition**

`Gate::define(string $ability, callable|string $callback)` adds the ability to the Gate's `$abilities` array. The callback can be a closure, a `Class@method` string, or an array `[Class::class, 'method']`. When the ability is checked, the callback receives the authenticated user and optional additional arguments.

**Beginner-Friendly Explanation**

`Gate::define()` is how you create a new gate. You give it a name and a rule, and Laravel uses that rule whenever you check that ability.

#### Purposes

- To create new authorization abilities.
- To define simple, closure-based authorization rules.
- To map abilities to policy methods via class callbacks.
- To provide a flexible API for authorization definition.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Closure definition
Gate::define('update-post', function (User $user, Post $post) {
    return $user->id === $post->user_id;
});

// Class@method definition
Gate::define('update-post', [PostPolicy::class, 'update']);

// String callback
Gate::define('update-post', 'App\Policies\PostPolicy@update');
```

**Component Breakdown**

| Callback Type | Example |
|--------------|---------|
| Closure | `function (User $user) { ... }` |
| Array | `[PostPolicy::class, 'update']` |
| String | `'App\Policies\PostPolicy@update'` |

**Syntax Rules**

- Defined in a service provider's `boot()` method.
- The callback receives the user as the first argument.
- Additional arguments are passed after the user.

**Constraints and Limitations**

- Gates are not auto-discovered; they must be defined explicitly.
- For model-specific logic, prefer policies.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Closure Gate**

```php
Gate::define('access-admin', function (User $user) {
    return $user->is_admin;
});
```

**Expected Output**

```
// Returns true if user is an admin
```

**Why This Output Occurs**

The gate checks the `is_admin` attribute of the authenticated user.

---

**Example 2: Policy Method Gate**

```php
use App\Policies\PostPolicy;

Gate::define('update-post', [PostPolicy::class, 'update']);
```

**Expected Output**

```
// Delegates the authorization check to PostPolicy::update
```

**Why This Output Occurs**

The gate callback resolves the `PostPolicy` class and calls its `update` method.

---

#### Real-World Cases with Explanation

**Case 1: Feature Flag Gates**

A SaaS application defines gates for each premium feature (e.g., `access-analytics`, `access-api`). Each gate checks the user's subscription plan.

**Explanation**: Gates provide a lightweight way to implement feature flags.

---

### 19. `Gate::resource()`

#### Definitions

**Core Definition**

`Gate::resource()` defines multiple gates at once for a resource, automatically mapping standard CRUD abilities to policy methods.

**Technical Definition**

`Gate::resource(string $name, string $class, array $abilities = null)` defines gates for `view`, `create`, `update`, and `delete` (by default) using the given policy class. The ability names are prefixed with the resource name (e.g., `posts.view`). Custom abilities can be specified via the third argument.

**Beginner-Friendly Explanation**

`Gate::resource()` is a shortcut that creates four gates at once: view, create, update, and delete. It is useful when you want to use gates with a policy-like structure.

#### Purposes

- To define multiple gates for a resource in one line.
- To reduce boilerplate when defining CRUD gates.
- To map gates to policy methods automatically.
- To provide a resource-oriented API for gate definition.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Gate::resource('posts', 'App\Policies\PostPolicy');

// With custom abilities
Gate::resource('posts', 'App\Policies\PostPolicy', [
    'image' => 'updateImage',
]);
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `$name` | Resource name (prefix for abilities). |
| `$class` | The policy class. |
| `$abilities` | Optional custom ability mappings. |

**Syntax Rules**

- Default abilities: `view`, `create`, `update`, `delete`.
- Ability names are prefixed: `posts.view`, `posts.create`, etc.
- Custom abilities can be added or overridden.

**Constraints and Limitations**

- `Gate::resource()` is less commonly used in modern Laravel; policies are preferred.
- The generated ability names use dot notation.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Resource Gates**

```php
Gate::resource('posts', 'App\Policies\PostPolicy');
```

**Expected Output**

```
// Defines gates: posts.view, posts.create, posts.update, posts.delete
// Each delegates to the corresponding PostPolicy method
```

**Why This Output Occurs**

`Gate::resource()` creates four gates that map to `view`, `create`, `update`, and `delete` methods on `PostPolicy`.

---

#### Real-World Cases with Explanation

**Case 1: Legacy Gate-Based Authorization**

A legacy application uses gates instead of policies. `Gate::resource()` provides a way to define CRUD gates consistently.

**Explanation**: `Gate::resource()` bridges gate-based and policy-based authorization patterns.

---

### 20. `@can`

#### Definitions

**Core Definition**

The `@can` Blade directive conditionally renders a block of template content based on whether the authenticated user is authorized for a given ability.

**Technical Definition**

`@can($ability, $arguments)` compiles to a call to `Gate::check()` (or the equivalent policy check). If the check passes, the content between `@can` and `@endcan` is rendered. The `@else` directive can be used for fallback content.

**Beginner-Friendly Explanation**

`@can` lets you show or hide parts of a page based on permissions. For example, show the "Edit" button only if the user can edit the post.

#### Purposes

- To conditionally display UI elements based on user permissions.
- To maintain consistency between backend authorization and frontend display.
- To prevent unauthorized users from seeing actions they cannot perform.
- To improve user experience by hiding irrelevant controls.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan

@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@else
    <span>You cannot edit this post.</span>
@endcan
```

**Component Breakdown**

| Directive | Description |
|-----------|-------------|
| `@can($ability, $arguments)` | Renders if authorized. |
| `@else` | Renders if not authorized. |
| `@endcan` | Ends the block. |

**Syntax Rules**

- Accepts an ability name and optional model arguments.
- Can be combined with `@else`.
- Uses the currently authenticated user.

**Constraints and Limitations**

- `@can` does not replace backend authorization; always authorize in controllers too.
- `@can` is for display purposes only.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Conditional Edit Button**

```blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}" class="btn">Edit Post</a>
@endcan
```

**Expected Output (if authorized)**

```html
<a href="/posts/1/edit" class="btn">Edit Post</a>
```

**Expected Output (if denied)**

```html
<!-- No edit button rendered -->
```

**Why This Output Occurs**

`@can` checks the `update` ability for the post. If authorized, the edit button is rendered; otherwise, it is omitted.

---

#### Real-World Cases with Explanation

**Case 1: Admin Panel Navigation**

A Blade layout uses `@can('access-admin')` to show the "Admin Panel" link only to administrators.

**Explanation**: `@can` ensures that unauthorized users do not see links to actions they cannot perform.

---

### 21. `@cannot`

#### Definitions

**Core Definition**

The `@cannot` Blade directive renders content when the authenticated user is **not** authorized for a given ability.

**Technical Definition**

`@cannot` is the inverse of `@can`. It compiles to a negated authorization check. Content between `@cannot` and `@endcannot` is rendered when the user is denied.

**Beginner-Friendly Explanation**

`@cannot` lets you show content when a user is not allowed to do something. For example, show an "Upgrade" notice when the user cannot access a premium feature.

#### Purposes

- To display fallback content when a user lacks permission.
- To show upgrade prompts or alternative messaging.
- To provide a clear denial experience in the UI.
- To complement `@can` for complete conditional rendering.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@cannot('delete', $post)
    <span class="text-muted">Deletion not permitted</span>
@endcannot
```

**Component Breakdown**

| Directive | Description |
|-----------|-------------|
| `@cannot($ability, $arguments)` | Renders if denied. |
| `@endcannot` | Ends the block. |

**Syntax Rules**

- Accepts the same arguments as `@can`.
- Renders when the authorization check fails.

**Constraints and Limitations**

- Same as `@can`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Upgrade Notice**

```blade
@cannot('create-posts')
    <div class="upgrade-notice">
        <p>Upgrade your account to create posts.</p>
        <a href="{{ route('plans.index') }}">View Plans</a>
    </div>
@endcannot
```

**Expected Output (if denied)**

```html
<div class="upgrade-notice">
    <p>Upgrade your account to create posts.</p>
    <a href="/plans">View Plans</a>
</div>
```

**Why This Output Occurs**

`@cannot` renders the upgrade notice when the user lacks the `create-posts` ability.

---

#### Real-World Cases with Explanation

**Case 1: Feature Upgrade Prompts**

A SaaS application uses `@cannot` to display upgrade prompts for features the user's plan does not include.

**Explanation**: `@cannot` enables targeted messaging for users lacking permissions.

---

### 22. `@canany`

#### Definitions

**Core Definition**

The `@canany` Blade directive renders content if the authenticated user is authorized for **any** of the specified abilities.

**Technical Definition**

`@canany($abilities, $arguments)` compiles to a check that returns `true` if at least one of the listed abilities is authorized. It accepts an array of ability names as the first argument.

**Beginner-Friendly Explanation**

`@canany` lets you check multiple permissions at once. If the user has **any** of them, the content is shown.

#### Purposes

- To conditionally render content when a user has any of several permissions.
- To simplify complex authorization checks in views.
- To provide a convenient alternative to chaining multiple `@can` directives.
- To improve readability for multi-permission scenarios.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
@canany(['publish', 'schedule'], $article)
    <div class="publishing-controls">
        Publishing options available
    </div>
@endcanany
```

**Component Breakdown**

| Directive | Description |
|-----------|-------------|
| `@canany($abilities, $arguments)` | Renders if any ability is authorized. |
| `@endcanany` | Ends the block. |

**Syntax Rules**

- The first argument is an array of ability names.
- Optional model arguments follow.
- Renders if at least one ability is authorized.

**Constraints and Limitations**

- Does not provide information about which ability was authorized.
- For "all abilities" checks, use `Gate::check()` instead.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Multi-Permission Admin Panel**

```blade
@canany(['view-analytics', 'manage-users', 'system-settings'])
    <div class="admin-panel">
        <h3>Administration</h3>
        <ul>
            @can('view-analytics')
                <li><a href="/analytics">Analytics</a></li>
            @endcan
            @can('manage-users')
                <li><a href="/users">User Management</a></li>
            @endcan
            @can('system-settings')
                <li><a href="/settings">System Settings</a></li>
            @endcan
        </ul>
    </div>
@endcanany
```

**Expected Output (if any admin permission)**

```html
<div class="admin-panel">
    <h3>Administration</h3>
    <ul>
        <li><a href="/analytics">Analytics</a></li>
        <!-- Other links based on individual @can checks -->
    </ul>
</div>
```

**Why This Output Occurs**

`@canany` renders the admin panel if the user has any of the listed administrative permissions. Individual `@can` checks then determine which links appear.

---

#### Real-World Cases with Explanation

**Case 1: Dashboard Widgets**

A dashboard uses `@canany` to show a "Management" section if the user has any administrative permission, with individual links controlled by `@can`.

**Explanation**: `@canany` provides a clean way to conditionally render sections based on multiple permissions.

---

### 23. Authorization Best Practices

#### Definitions

**Core Definition**

Authorization best practices are the recommended guidelines for implementing, organizing, and maintaining authorization logic in Laravel applications.

**Technical Definition**

Best practices encompass the choice between gates and policies, policy organization, controller authorization, Blade directives, testing, performance, and security. They aim to balance security, maintainability, and developer experience.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep authorization secure, organized, and easy to maintain.

#### Key Best Practices

1. **Use Policies for Model-Specific Actions**: Policies group authorization logic around models, making it easier to find and maintain. Use gates for model-agnostic actions like accessing an admin dashboard.

2. **Keep Policy Methods Simple**: Policy methods should return boolean values and delegate complex business logic to services or the model. Avoid database queries in policies where possible.

3. **Use `authorize()` in Controllers**: Use `$this->authorize()` or `Gate::authorize()` to enforce authorization and automatically throw 403 exceptions. Avoid manual `if (! Gate::allows(...)) abort(403);` patterns.

4. **Use `authorizeResource()` for Resource Controllers**: Register resource authorization in the controller constructor to automatically map CRUD actions to policy methods.

5. **Use Blade Directives for UI Only**: `@can`, `@cannot`, and `@canany` should only control display logic. Always enforce authorization in controllers or middleware.

6. **Implement Super-Admin Bypass with `Gate::before()`**: Centralize super-admin logic in a global `before` callback rather than repeating it in every policy method.

7. **Use `Gate::after()` for Audit Logging**: Log authorization decisions for security monitoring and compliance without modifying authorization results.

8. **Test Authorization Logic**: Write tests for gates and policies, including edge cases (unauthenticated users, users with different roles, resource ownership).

9. **Use Consistent Naming**: Follow Laravel conventions for policy method names (`viewAny`, `view`, `create`, `update`, `delete`) to enable auto-discovery and resource controller integration.

10. **Separate Authorization from Authentication**: Authorization should only run after authentication. Use the `auth` middleware before the `can` middleware.

11. **Use `Gate::forUser()` for Administrative Tools**: When building admin panels that manage other users, use `forUser()` to check permissions for the target user.

12. **Avoid Hardcoding Role Checks in Policies**: Instead of checking `$user->role === 'admin'`, use abilities or gates that can be redefined without changing policy code.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Well-Structured Authorization**

**Policy**

```php
<?php
// app/Policies/PostPolicy.php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function viewAny(User $user): bool
    {
        return true;
    }

    public function view(User $user, Post $post): bool
    {
        return $post->is_published || $user->id === $post->user_id;
    }

    public function create(User $user): bool
    {
        return $user->hasRole('editor');
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->hasRole('admin');
    }

    public function before(User $user, string $ability): ?bool
    {
        if ($user->hasRole('super-admin')) {
            return true;
        }

        return null;
    }
}
```

**Controller**

```php
public function __construct()
{
    $this->authorizeResource(Post::class, 'post');
}
```

**Blade**

```blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan
```

**Expected Output**

```
// Viewing: published posts visible to all; drafts visible only to author
// Creating: only editors
// Updating: only author
// Deleting: author or admin
// Super-admin: bypasses all checks
```

**Why This Output Occurs**

The policy encapsulates all post-related authorization logic, the controller uses `authorizeResource()` for automatic enforcement, and Blade directives conditionally render UI controls.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise SaaS Application**

A large SaaS platform uses policies for all model-specific authorization, gates for feature flags, `Gate::before()` for super-admin bypass, `Gate::after()` for audit logging, and comprehensive authorization tests.

**Explanation**: Following best practices ensures authorization is secure, maintainable, and auditable at scale.

---

## References

- Authorization – Laravel 12.x Documentation — https://laravel.com/docs/12.x/authorization
- Authorization – Laravel master Documentation — https://laravel.com/framework/docs/master/authorization
- Authorization – Laravel 10.x Documentation — https://laravel.com/docs/10.x/authorization
- Gate Interface – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Auth/Access/Gate.html
- Gate Class – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Auth/Access/Gate.html
- Blade Authorization Directives for View Security – Laravel News — https://laravel-news.com/blade-authorization-can-cannot
- Laravel's UsePolicy Attribute: Explicit Authorization Control – Laravel News — https://laravel-news.com/laravel-usepolicy-attribute
- Security Best Practices – Laravel Permission (Spatie) — https://mintlify.wiki/spatie/laravel-permission/security-best-practices
- Best Practices: Use Policies and Gates for Authorization – Dutch Laravel Foundation — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/project-structure-and-code-architecture/use-policies-and-gates-for-authorization.md
- Spatie Permissions + Laravel Policies: Best Practice for Permission/Role Controllers – Laracasts — https://laracasts.com/discuss/channels/laravel/spatie-permissions-laravel-policies-best-practice-for-permissionrole-controllers
- Laravel Authorization with Gates and Policies – GitHub (voku/agent-skills) — https://github.com/voku/agent-skills/blob/main/skills/laravel-owasp-security/rules/sec-authentication-rate-limiting.md

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific changes include: Laravel 11 removed the `AuthServiceProvider` from the default application skeleton, making policy auto-discovery the default; the `UsePolicy` attribute (Laravel 12.30+) provides explicit policy mapping; `Gate::resource()` is less commonly used in modern Laravel in favor of policies. Developers should always consult the documentation for their specific Laravel version.