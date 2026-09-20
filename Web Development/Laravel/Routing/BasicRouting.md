# Laravel Basic Routing & Parameters: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel routing is the mechanism that maps incoming HTTP requests (identified by URI and HTTP verb) to specific application logic—either a Closure or a controller method—that generates a response.

**Technical Definition:** Routing in Laravel is implemented through the `Illuminate\Routing\Router` class, which registers route definitions into a `RouteCollection`. Each route encapsulates a URI pattern, an array of permitted HTTP verbs, an action (Closure, controller array, or invokable class), and optional metadata such as middleware, names, and parameter constraints. When an HTTP request enters the application, the router matches it against the compiled route collection and dispatches it to the resolved action via the route's `run()` method.

**Beginner-Friendly Explanation:** Think of routing as a receptionist at a large office building. When someone arrives (an HTTP request), the receptionist looks at where they want to go (the URI) and how they're asking to get there (the HTTP verb—GET, POST, etc.), then directs them to the right person (the controller or Closure) who can help them. Without routing, every request would hit the same wall with no direction.

### Key Characteristics

- **Declarative:** Routes are defined in dedicated files (`routes/web.php`, `routes/api.php`) using a fluent, readable syntax.
- **Verb-Aware:** Laravel distinguishes between GET, POST, PUT, PATCH, DELETE, and OPTIONS requests, allowing RESTful API design.
- **Parameterised:** URIs can contain dynamic segments (parameters) that are injected into the handling logic.
- **Nameable:** Routes can be assigned immutable names, decoupling URL generation from hardcoded paths.
- **Cacheable:** Route definitions can be compiled into a single cached file for production performance.
- **Middleware-Integrated:** Routes can be assigned middleware for authentication, rate limiting, CSRF protection, and more.
- **Model-Binding Capable:** Route parameters can automatically resolve to Eloquent model instances.

### Prerequisites

- **PHP 8.1+** (Laravel 10.x) or **PHP 8.2+** (Laravel 11.x/12.x).
- **Composer** for dependency management.
- A Laravel application with the `routes/` directory structure.
- Basic understanding of HTTP methods and status codes.
- Familiarity with PHP Closures and object-oriented PHP (for controller-based routing).

### Related Programming Areas

- **MVC Architecture:** Routing is the entry point of the Model-View-Controller pattern.
- **RESTful API Design:** HTTP verb mapping is fundamental to REST semantics.
- **Middleware Pipeline:** Routes are the primary attachment point for request/response middleware.
- **Dependency Injection Container:** Route actions resolve their dependencies through Laravel's service container.
- **Eloquent ORM:** Route model binding bridges routing parameters to database records.

### Core Concepts / Features

1. Route Definitions (Closure vs. Controller)
2. HTTP Verbs (get, post, put, patch, delete, match, any)
3. Route Names
4. Required Route Parameters
5. Optional Route Parameters
6. Regular Expression Constraints

---

## 1. Route Definitions: Closure Entry Points vs. Controller Action Mapping

### Definitions

**Core Definition:** A route definition binds a URI and HTTP verb to an executable action, which may be an inline Closure or a reference to a controller method.

**Technical Definition:** In Laravel, a route action is resolved by the `RouteAction` class. When the action is a Closure, it is stored directly and invoked during dispatch. When the action is `[Controller::class, 'method']`, the router resolves the controller instance from the service container and calls the specified method with the resolved parameters.

**Beginner-Friendly Explanation:** You can either write the response logic right inside the route file (a Closure—quick and simple) or delegate it to a dedicated controller class (organised and reusable, especially for larger applications).

### Purposes

- To define endpoints quickly for prototyping or simple responses using Closures.
- To organise request-handling logic into dedicated controller classes for maintainability.
- To enable route caching, which requires controller-based routes (Closures cannot be cached).
- To leverage dependency injection and middleware at the controller level.
- To separate concerns between routing and business logic.

### Syntax Rules and Structure

**Complete General Syntax — Closure:**

```php
Route::verb('/uri', function (Request $request) {
    // response logic
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Route::` | The `Route` facade (aliases `Illuminate\Support\Facades\Route`). |
| `verb` | The HTTP verb method (`get`, `post`, etc.). |
| `'/uri'` | The URI pattern the route responds to. |
| `function` | A PHP Closure receiving dependencies via type-hinting. |

**Complete General Syntax — Controller:**

```php
Route::verb('/uri', [ControllerName::class, 'methodName']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ControllerName::class` | Fully qualified controller class reference (resolved via `::class`). |
| `'methodName'` | The public method on the controller to invoke. |

**Syntax Rules:**

- The URI must begin with `/` (Laravel prepends it if omitted).
- Closures cannot be serialised, so they prevent `php artisan route:cache` from functioning.
- Controller methods should be public and may accept route parameters plus injected dependencies.
- Both forms can be chained with `->name()`, `->where()`, `->middleware()`, etc.

**Constraints and Limitations:**

- **Closure limitation:** Routes using Closures cannot be cached, which is a significant performance drawback in production.
- **Controller resolution:** The controller class must be autoloadable and resolvable by the service container.
- **Method visibility:** Controller methods must be `public`.
- **Serialisation:** Only controller-based routes (and invokable controllers) can be serialised for route caching.

### Multiple Annotated Complete Code Examples

**Example 1: Closure-Based Route with Dependency Injection**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

// Define a GET route at /greeting
Route::get('/greeting', function (Request $request) {
    // $request is automatically injected by Laravel's service container
    $name = $request->query('name', 'World');

    // Return a plain string response
    return "Hello, {$name}!";
});
```

**Setup:** Place this in `routes/web.php`. Ensure the Laravel application is bootstrapped.

**Expected Output:** Visiting `/greeting?name=Alice` returns `Hello, Alice!`; visiting `/greeting` returns `Hello, World!`.

**Why:** The `Request` instance is resolved from the container and injected into the Closure. The `query()` method reads the `name` query-string parameter, defaulting to `'World'`.

---

**Example 2: Controller-Based Route**

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Display the specified user.
     */
    public function show(Request $request, string $id)
    {
        // $id is the route parameter, $request is injected
        return "User ID: {$id}";
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

// Map GET /user/{id} to UserController@show
Route::get('/user/{id}', [UserController::class, 'show']);
```

**Setup:** Create the controller via `php artisan make:controller UserController`. Add the `show` method and the route.

**Expected Output:** Visiting `/user/42` returns `User ID: 42`.

**Why:** Laravel resolves `UserController` from the container, calls `show()`, and passes the `{id}` parameter (and the `Request` via injection).

---

**Example 3: Invokable Controller (Single-Action)**

```php
<?php
// app/Http/Controllers/HomeController.php

namespace App\Http\Controllers;

class HomeController extends Controller
{
    /**
     * Handle the request.
     */
    public function __invoke()
    {
        return 'Welcome to the homepage!';
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\HomeController;
use Illuminate\Support\Facades\Route;

// Invokable controllers are referenced by class name only
Route::get('/', HomeController::class);
```

**Expected Output:** Visiting `/` returns `Welcome to the homepage!`.

**Why:** When only a class name is provided (no method array), Laravel invokes the `__invoke` method on the controller.

### Real-World Cases

- **Prototyping:** Closures are ideal for quick API stubs or health-check endpoints (`/health` returning `"OK"`).
- **CRUD Applications:** Controller-based routing (`Route::resource()`) is standard for full CRUD operations on Eloquent models.
- **Single-Purpose Pages:** Invokable controllers are perfect for landing pages, dashboards, or webhook handlers where one class handles one responsibility.
- **Route Caching in Production:** All routes in a production Laravel application should be controller-based to enable `php artisan route:cache`, which can reduce request overhead by 2–5×.

### References

- Laravel Routing Documentation — https://laravel.com/docs/12.x/routing
- Laravel Controllers Documentation — https://laravel.com/docs/12.x/controllers

---

## 2. HTTP Verbs: Mapping Specific Endpoints

### Definitions

**Core Definition:** HTTP verbs are methods defined by the HTTP protocol that indicate the desired action to be performed on a resource. Laravel provides dedicated route registration methods for each verb.

**Technical Definition:** The `Registrar` interface in Laravel defines methods such as `get()`, `post()`, `put()`, `patch()`, `delete()`, and `options()`, each registering a route with the corresponding HTTP verb(s) in the `RouteCollection`. The `match()` method accepts an array of verbs, while `any()` registers a route responding to all verbs.

**Beginner-Friendly Explanation:** When you visit a website, your browser sends a GET request. When you submit a form, it sends a POST. Laravel lets you say "only respond to GET here" or "only respond to POST there," and also lets you say "respond to both GET and POST" or "respond to anything."

### Purposes

- To map CRUD operations to the correct HTTP semantics (GET for reading, POST for creating, PUT/PATCH for updating, DELETE for removing).
- To build RESTful APIs where verb + URI uniquely identifies an operation.
- To reduce the number of URIs by using different verbs on the same URI.
- To handle multi-verb endpoints with `match()` when a resource supports multiple actions at one URI.
- To create catch-all routes with `any()` for webhook receivers or fallback handlers.

### Syntax Rules and Structure

**Complete General Syntax — Single Verb:**

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Route::get` | Registers a route that only responds to HTTP GET requests. |
| `Route::post` | Registers a route that only responds to HTTP POST requests. |
| `Route::put` | Registers a route for full resource replacement (idempotent). |
| `Route::patch` | Registers a route for partial resource modification. |
| `Route::delete` | Registers a route for resource deletion. |
| `Route::options` | Registers a route for CORS preflight or capability discovery. |

**Complete General Syntax — Multi-Verb:**

```php
// Respond to specific multiple verbs
Route::match(['get', 'post'], $uri, $callback);

// Respond to all verbs
Route::any($uri, $callback);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Route::match` | Accepts an array of HTTP verb strings as the first argument. |
| `['get', 'post']` | The verbs this route responds to (case-insensitive). |
| `Route::any` | Registers the route for all standard HTTP verbs. |

**Syntax Rules:**

- Verb names in `match()` are case-insensitive but conventionally lowercase.
- Routes using `get`, `post`, `put`, `patch`, `delete`, and `options` must be defined **before** routes using `any`, `match`, or `redirect` when they share the same URI.
- `Route::any()` registers routes for GET, POST, PUT, PATCH, DELETE, and OPTIONS.

**Constraints and Limitations:**

- **`Route::any()` is discouraged** unless genuinely necessary, as it makes the route's behaviour ambiguous and can mask errors.
- **HEAD requests** are automatically handled by Laravel when GET routes are defined.
- **OPTIONS requests** are handled automatically for CORS when using `Route::options()` or the `any`/`match` methods.
- **Route ordering matters:** A wildcard `any` route defined before a specific GET route will intercept all GET requests.

### Multiple Annotated Complete Code Examples

**Example 1: RESTful Verb Mapping**

```php
<?php
// routes/api.php

use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

// GET /posts — list all posts (read)
Route::get('/posts', [PostController::class, 'index']);

// POST /posts — create a new post
Route::post('/posts', [PostController::class, 'store']);

// GET /posts/{id} — show a single post
Route::get('/posts/{id}', [PostController::class, 'show']);

// PUT /posts/{id} — replace a post entirely
Route::put('/posts/{id}', [PostController::class, 'update']);

// PATCH /posts/{id} — partially modify a post
Route::patch('/posts/{id}', [PostController::class, 'patch']);

// DELETE /posts/{id} — delete a post
Route::delete('/posts/{id}', [PostController::class, 'destroy']);
```

**Expected Output:** Each verb + URI combination maps to a distinct controller action. A GET request to `/api/posts` triggers `index()`; a DELETE to `/api/posts/5` triggers `destroy()`.

**Why:** REST semantics use the same URI with different verbs to represent different operations on the same resource. Laravel's verb-specific methods enforce this cleanly.

---

**Example 2: Multi-Verb Route with `match()`**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Respond to both GET and POST at /contact
Route::match(['get', 'post'], '/contact', function () {
    $method = request()->method();

    if ($method === 'GET') {
        return 'Please fill out the contact form.';
    }

    return 'Thank you for your message!';
});
```

**Expected Output:**
- `GET /contact` → `Please fill out the contact form.`
- `POST /contact` → `Thank you for your message!`
- `PUT /contact` → `405 Method Not Allowed`

**Why:** `match()` restricts the route to only the listed verbs, so any other verb receives a `405` response.

---

**Example 3: Catch-All Route with `any()`**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Webhook receiver — accepts any HTTP verb
Route::any('/webhook/github', function () {
    $payload = request()->all();
    // Process webhook payload...
    return response()->json(['status' => 'received'], 200);
});
```

**Expected Output:** A POST, PUT, or even DELETE request to `/webhook/github` returns `{"status":"received"}` with HTTP 200.

**Why:** Webhook providers often use different verbs for different event types. `any()` ensures the endpoint never returns 405.

### Real-World Cases

- **RESTful APIs:** Verb-based routing is the foundation of REST. Every resource (users, posts, orders) gets a standard set of verb–URI mappings.
- **Form Handling:** A single `/contact` URI can serve both the GET form display and the POST form submission.
- **Webhooks:** Third-party services (GitHub, Stripe, Slack) send webhooks using POST, but sometimes use other verbs. `any()` future-proofs the endpoint.
- **CORS Preflight:** `Route::options()` handles browser preflight requests during cross-origin API calls.

### References

- Laravel Routing: Available Router Methods — https://laravel.com/docs/12.x/routing#available-router-methods
- Laravel API: Registrar Interface — https://api.laravel.com/docs/6.x/Illuminate/Contracts/Routing/Registrar.html

---

## 3. Route Names

### Definitions

**Core Definition:** A route name is a unique, immutable string identifier assigned to a route, enabling URL generation and redirection without coupling code to the route's actual URI.

**Technical Definition:** The `name()` method on a `Route` instance sets the `name` attribute in the route's action array. The `UrlGenerator` resolves this name via the `RouteCollection`'s name lookup table, allowing the `route()` helper and `Redirect` facade to generate absolute URLs.

**Beginner-Friendly Explanation:** Instead of writing `href="/user/profile"` in your templates (which breaks if the URL changes), you give the route a name like `user.profile` and write `route('user.profile')`. If you later change the URL from `/user/profile` to `/account/profile`, you only update the route definition—every link in your app updates automatically.

### Purposes

- To decouple application code from hardcoded URLs, so URL changes require edits in only one place.
- To enable readable, self-documenting redirections (`redirect()->route('login')` instead of `redirect('/auth/login')`).
- To simplify URL generation in Blade templates and controllers.
- To support route name prefixes via route groups (`->name('admin.')`).
- To facilitate testing by allowing tests to reference routes by name rather than URI.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->name('profile')` | Chains onto any route definition to assign a name. |
| `'profile'` | The unique identifier string (commonly dot-notated: `admin.users.index`). |

**Complete General Syntax — Generating URLs:**

```php
$url = route('profile');
$url = route('profile', ['id' => 1]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `route('profile')` | The global helper resolving the named route to an absolute URL. |
| `['id' => 1]` | Associative array of route parameters to substitute into the URI. |

**Syntax Rules:**

- Route names must be unique across the entire application.
- Dot notation (`admin.users.show`) is conventional for hierarchical naming.
- The `route()` helper throws `RouteNotFoundException` if the name does not exist.
- Route name prefixes can be applied to groups: `Route::name('admin.')->group(...)`.

**Constraints and Limitations:**

- **Uniqueness:** Duplicate route names cause the last-registered route to overwrite the first in the name lookup table.
- **Closure routes can be named** but cannot be cached, so naming Closures provides no production benefit.
- **Parameter binding:** The `route()` helper requires all required parameters to be provided, or it throws `UrlGenerationException`.
- **Fallback routes** cannot be named.

### Multiple Annotated Complete Code Examples

**Example 1: Naming and Using a Route**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Define and name the route
Route::get('/user/profile', function () {
    return 'User Profile Page';
})->name('profile');

// Generate the URL from anywhere in the application
$url = route('profile');
// $url === 'http://example.com/user/profile'
```

**Expected Output:** `route('profile')` returns `http://example.com/user/profile`.

**Why:** The `name()` method registers `'profile'` in the route collection's name table. The `route()` helper looks up that name and generates the absolute URL.

---

**Example 2: Named Route with Parameters**

```php
<?php
// routes/web.php

use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

// Named route with a required parameter
Route::get('/posts/{post}', [PostController::class, 'show'])
    ->name('posts.show');

// Generate URL with parameter
$url = route('posts.show', ['post' => 42]);
// $url === 'http://example.com/posts/42'

// Generate URL with positional parameter
$url = route('posts.show', [42]);
// $url === 'http://example.com/posts/42'
```

**Expected Output:** Both `route()` calls return `http://example.com/posts/42`.

**Why:** The `route()` helper replaces the `{post}` placeholder with the provided value, whether passed as a named key or positional index.

---

**Example 3: Route Name Prefixes in Groups**

```php
<?php
// routes/web.php

use App\Http\Controllers\Admin\DashboardController;
use Illuminate\Support\Facades\Route;

Route::prefix('admin')->name('admin.')->group(function () {
    // Route name becomes 'admin.dashboard'
    Route::get('/dashboard', [DashboardController::class, 'index'])
        ->name('dashboard');

    // Route name becomes 'admin.settings'
    Route::get('/settings', [DashboardController::class, 'settings'])
        ->name('settings');
});

// Usage
$url = route('admin.dashboard'); // http://example.com/admin/dashboard
$url = route('admin.settings');  // http://example.com/admin/settings
```

**Expected Output:** `route('admin.dashboard')` returns `http://example.com/admin/dashboard`.

**Why:** The `->name('admin.')` prefix on the group prepends `admin.` to every route name defined within it, preventing naming collisions.

### Real-World Cases

- **Navigation Menus:** Blade templates use `route('home')`, `route('about')`, `route('contact')` for menu links, so changing a URL never requires touching the template.
- **Form Actions:** `<form action="{{ route('posts.store') }}" method="POST">` remains valid even if the store URI changes.
- **Email Links:** Password reset emails use `route('password.reset', ['token' => $token])` to generate secure, correct links.
- **API Versioning:** Names like `api.v1.users.index` and `api.v2.users.index` allow both API versions to coexist without URI conflicts.

### References

- Laravel Routing: Named Routes — https://laravel.com/docs/12.x/routing#named-routes
- Laravel URL Generation — https://laravel.com/docs/12.x/urls

---

## 4. Required Route Parameters

### Definitions

**Core Definition:** Required route parameters are dynamic segments within a URI pattern, enclosed in curly braces, that must be present in the incoming request for the route to match.

**Technical Definition:** The `RouteCompiler` (in `Illuminate\Routing`) compiles `{param}` placeholders into regex capture groups. During matching, the `Route` instance binds these captured values to the route's parameters array, which is then passed to the action as arguments.

**Beginner-Friendly Explanation:** If your route is `/user/{id}`, then the `{id}` part is a required parameter. The user must visit `/user/1` or `/user/42`—they cannot visit `/user/` and expect it to work. The value they put in place of `{id}` is captured and handed to your code.

### Purposes

- To capture dynamic segments from the URI (IDs, slugs, usernames) for use in querying or processing.
- To create hierarchical resource URIs such as `/posts/{post}/comments/{comment}`.
- To enable route model binding, where the parameter resolves to a database record.
- To enforce URL structure, ensuring that essential identifying information is always present.
- To support nested resources where a child resource is scoped to a parent.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
Route::get('/user/{id}', function (string $id) {
    // ...
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{id}` | The parameter placeholder (always in curly braces). |
| `$id` | The variable in the Closure/controller method that receives the value. |
| `string $id` | Optional type declaration for the parameter. |

**Complete General Syntax — Multiple Parameters:**

```php
Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{post}` | First required parameter (in URI order). |
| `{comment}` | Second required parameter (in URI order). |
| `$postId` | First closure argument, receives `{post}` value. |
| `$commentId` | Second closure argument, receives `{comment}` value. |

**Syntax Rules:**

- Parameters are enclosed in `{}` (curly braces).
- Parameters are injected into the action in the order they appear in the URI.
- Parameter names in the URI and the variable names in the action do not need to match (but matching is recommended for clarity and model binding).
- Parameters cannot contain the `-` character; use `_` instead.
- By default, parameters cannot contain `/` (slash). Use a `where` constraint with `.*` to allow slashes.

**Constraints and Limitations:**

- **Order matters:** Parameters are passed positionally. If the URI has `{a}/{b}`, the action must accept two arguments.
- **Hyphen restriction:** `{user-id}` is invalid; use `{user_id}` or `{userId}`.
- **No optional followed by required:** An optional parameter cannot precede a required parameter.
- **Maximum segments:** While Laravel supports multiple parameters, excessive nesting (more than 3–4 levels) often indicates a design smell.

### Multiple Annotated Complete Code Examples

**Example 1: Single Required Parameter**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Capture a user ID from the URI
Route::get('/user/{id}', function (string $id) {
    return "User ID: {$id}";
});
```

**Expected Output:**
- `GET /user/5` → `User ID: 5`
- `GET /user/` → `404 Not Found`

**Why:** The `{id}` placeholder requires a non-empty segment. Without it, the URI does not match the route pattern.

---

**Example 2: Multiple Required Parameters**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Nested resource: post and comment
Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    return "Post {$postId}, Comment {$commentId}";
});
```

**Expected Output:** `GET /posts/10/comments/25` returns `Post 10, Comment 25`.

**Why:** Both `{post}` and `{comment}` are required. Laravel matches the URI against the compiled regex and captures both segments in order.

---

**Example 3: Required Parameters with Route Model Binding**

```php
<?php
// routes/web.php

use App\Models\User;
use Illuminate\Support\Facades\Route;

// Implicit model binding: {user} resolves to a User model instance
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

**Expected Output:**
- `GET /users/1` → returns the email of the user with ID 1.
- `GET /users/9999` → `404 Not Found` (no matching model).

**Why:** The `{user}` parameter name matches the `$user` type-hint (`User`), triggering implicit route model binding. Laravel queries the database automatically.

### Real-World Cases

- **User Profiles:** `/users/{id}` is the canonical pattern for displaying a user's profile.
- **Blog Posts:** `/posts/{slug}` uses a required parameter with a custom key for SEO-friendly URLs.
- **E-commerce:** `/products/{product}/reviews/{review}` handles nested resources.
- **Multi-tenant Applications:** `/{tenant}/dashboard` uses a required parameter to identify the current tenant.

### References

- Laravel Routing: Required Parameters — https://laravel.com/docs/12.x/routing#required-parameters
- Laravel Route Model Binding — https://laravel.com/docs/12.x/routing#route-model-binding

---

## 5. Optional Route Parameters

### Definitions

**Core Definition:** Optional route parameters are dynamic URI segments that may or may not be present in the incoming request, indicated by a trailing `?` in the placeholder.

**Technical Definition:** Laravel's `RouteCompiler` treats a `{param?}` placeholder as optional by wrapping its regex capture group in an optional quantifier (`?`). The corresponding action argument must have a default value, otherwise a `MissingRequiredParameterException` is thrown during resolution.

**Beginner-Friendly Explanation:** If you have `/user/{name?}`, then both `/user/Alice` and `/user` are valid. If the name is omitted, your code receives the default value you specified (e.g., `'Guest'` or `null`).

### Purposes

- To provide a sensible default when a URI segment is omitted (e.g., a profile page that defaults to the authenticated user).
- To create flexible endpoints that serve both listing and detail views from the same URI pattern.
- To simplify client-side code by making some path segments non-mandatory.
- To support pagination or filtering where a page number is optional.
- To maintain backward compatibility when adding new optional segments to existing URIs.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
Route::get('/user/{name?}', function (?string $name = null) {
    return $name ?? 'Guest';
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{name?}` | The `?` inside the braces marks the parameter as optional. |
| `?string $name` | Nullable type-hint for the parameter. |
| `= null` | Default value — **required** when the parameter is optional. |

**Complete General Syntax — Optional with Explicit Default:**

```php
Route::get('/user/{name?}', function (string $name = 'John') {
    return $name;
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{name?}` | Optional parameter placeholder. |
| `string $name = 'John'` | Default value `'John'` is used when the parameter is absent. |

**Syntax Rules:**

- The `?` must be **inside** the curly braces: `{name?}`, not `{name}?`.
- The corresponding function parameter **must** have a default value (`= null`, `= 'John'`, etc.).
- Optional parameters must be at the **trailing position** of the URI. `/user/{name?}/edit` is invalid.
- Only one optional parameter is conventionally used per route, though multiple trailing optional parameters are technically possible (each with a default).

**Constraints and Limitations:**

- **Trailing only:** An optional parameter cannot be followed by a required parameter or a static segment.
- **Default required:** Omitting the default value in the action causes a runtime exception.
- **Ambiguity:** An optional parameter at the root level (`/{page?}`) can accidentally match other routes.
- **Route caching:** Optional parameters work with route caching, but the default value must be serialisable.

### Multiple Annotated Complete Code Examples

**Example 1: Optional Parameter with `null` Default**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Optional name parameter
Route::get('/user/{name?}', function (?string $name = null) {
    // if ($name === null) {
    //     return 'Hello, Guest!';
    // }

    return "Hello, {$name ?? 'Guest'}!";
});
```

**Expected Output:**
- `GET /user` → `Hello, Guest!`
- `GET /user/Alice` → `Hello, Alice!`

**Why:** When the `{name?}` segment is absent, Laravel calls the Closure without that argument, so the default `null` is used.

---

**Example 2: Optional Parameter with Explicit Default**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Optional page number for pagination
Route::get('/posts/page/{page?}', function (int $page = 1) {
    return "Showing page {$page} of posts.";
});
```

**Expected Output:**
- `GET /posts/page` → `Showing page 1 of posts.`
- `GET /posts/page/3` → `Showing page 3 of posts.`

**Why:** The default value `1` is used when no page is specified, providing a sensible fallback for the first page.

---

**Example 3: Combining Optional and Required Parameters**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Required category, optional subcategory
Route::get('/shop/{category}/{subcategory?}', function (string $category, ?string $subcategory = null) {
    if ($subcategory) {
        return "Category: {$category}, Subcategory: {$subcategory}";
    }

    return "Category: {$category}";
});
```

**Expected Output:**
- `GET /shop/electronics` → `Category: electronics`
- `GET /shop/electronics/laptops` → `Category: electronics, Subcategory: laptops`

**Why:** The required `{category}` must always be present. The optional `{subcategory?}` is only matched if an additional segment exists.

### Real-World Cases

- **User Profiles:** `/profile/{tab?}` defaults to the "overview" tab when no tab is specified.
- **Search Pagination:** `/search/{query}/{page?}` defaults to page 1.
- **Language Selection:** `/blog/{locale?}` defaults to the application's default locale.
- **API Filtering:** `/api/products/{category?}` lists all products when no category is given, or filters by category when provided.

### References

- Laravel Routing: Optional Parameters — https://laravel.com/docs/12.x/routing#optional-parameters
- Laravel Lumen Routing (Optional Parameters) — https://lumen.laravel.com/docs/routing

---

## 6. Regular Expression Constraints

### Definitions

**Core Definition:** Regular expression constraints are patterns applied to route parameters that restrict which URI values match the route, using the `where()` method or its convenience helpers.

**Technical Definition:** The `where()` method on a `Route` instance sets a key–value pair in the route's `wheres` array. During `RouteCompiler::compile()`, these patterns are appended to the parameter's capture group as a regex sub-pattern, filtering matches before the route is dispatched.

**Beginner-Friendly Explanation:** You can tell Laravel "only match this route if the `{id}` part is a number" or "only if the `{name}` part contains letters." If the incoming URI doesn't fit the pattern, Laravel skips that route and tries the next one—or returns a 404 if none match.

### Purposes

- To ensure that route parameters contain only the expected type of data (numeric IDs, alphabetic slugs, UUIDs).
- To prevent invalid or malicious input from reaching route handlers.
- To differentiate between routes that share a similar URI structure but differ in parameter format.
- To provide an early, cheap validation layer before controller logic runs.
- To improve performance by failing fast on non-matching URIs.

### Syntax Rules and Structure

**Complete General Syntax — `where()` with Custom Regex:**

```php
Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->where('id', '[0-9]+')` | Constrains the `{id}` parameter to one or more digits. |
| `'id'` | The parameter name (without braces). |
| `'[0-9]+'` | The regular expression pattern (no delimiters). |

**Complete General Syntax — Convenience Helpers:**

```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->whereNumber('id')` | Shorthand for `where('id', '[0-9]+')`. |
| `->whereAlpha('name')` | Shorthand for `where('name', '[a-zA-Z]+')`. |
| `->whereAlphaNumeric('code')` | Shorthand for `where('code', '[a-zA-Z0-9]+')`. |
| `->whereUuid('uuid')` | Shorthand for UUID pattern (Laravel 9+). |

**Complete General Syntax — Global Pattern (in `RouteServiceProvider`):**

```php
// In RouteServiceProvider::boot()
Route::pattern('id', '[0-9]+');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Route::pattern('id', '[0-9]+')` | Applies the constraint globally to all `{id}` parameters. |

**Syntax Rules:**

- The regex is provided **without delimiters** (no `/.../`).
- Multiple constraints can be chained: `->where(...)->where(...)`.
- An array can constrain multiple parameters at once: `->where(['id' => '[0-9]+', 'name' => '[a-z]+'])`.
- Global patterns defined via `Route::pattern()` apply to all routes, but per-route `where()` calls override them.
- To allow `/` in a parameter, use `->where('path', '.*')`.

**Constraints and Limitations:**

- **No delimiters:** Including delimiters (`/[0-9]+/`) causes the pattern to be misinterpreted.
- **Global patterns are overridden** by route-specific `where()` constraints on the same parameter name.
- **`whereNumber()` and `whereAlpha()`** are available from Laravel 8.x onward. In older versions, use `where()` with explicit regex.
- **Complex regex** can impact performance; simple character-class constraints are preferred.
- **Regex anchors** (`^` and `$`) are added automatically; do not include them.

### Multiple Annotated Complete Code Examples

**Example 1: Numeric ID Constraint**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Only match if {id} is one or more digits
Route::get('/user/{id}', function (string $id) {
    return "User ID: {$id}";
})->whereNumber('id');
```

**Expected Output:**
- `GET /user/42` → `User ID: 42`
- `GET /user/abc` → `404 Not Found`

**Why:** The `whereNumber` helper applies the regex `[0-9]+` to the `{id}` parameter. The URI `/user/abc` fails the pattern, so the route is skipped.

---

**Example 2: Alphabetic and Alphanumeric Constraints**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Constrain multiple parameters
Route::get('/category/{name}/product/{sku}', function (string $name, string $sku) {
    return "Category: {$name}, SKU: {$sku}";
})
    ->whereAlpha('name')        // Only letters: [a-zA-Z]+
    ->whereAlphaNumeric('sku'); // Letters and numbers: [a-zA-Z0-9]+
```

**Expected Output:**
- `GET /category/electronics/product/ABC123` → `Category: electronics, SKU: ABC123`
- `GET /category/123/product/ABC123` → `404 Not Found` (name is not alphabetic)
- `GET /category/electronics/product/AB!@#` → `404 Not Found` (sku contains special chars)

**Why:** Each `where*` method applies a specific character-class constraint to its corresponding parameter.

---

**Example 3: Custom Regex with Multiple Parameters**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Constrain a date parameter
Route::get('/archive/{year}/{month}', function (string $year, string $month) {
    return "Archive for {$year}-{$month}";
})->where([
    'year'  => '[0-9]{4}',  // Exactly 4 digits
    'month' => '0[1-9]|1[0-2]', // 01-12
]);
```

**Expected Output:**
- `GET /archive/2024/06` → `Archive for 2024-06`
- `GET /archive/24/6` → `404 Not Found` (year must be 4 digits)
- `GET /archive/2024/13` → `404 Not Found` (month must be 01-12)

**Why:** The array form of `where()` applies multiple constraints in a single call, with each regex validated independently.

### Real-World Cases

- **SEO Slugs:** `/blog/{slug}` constrained to `[a-z0-9-]+` ensures slugs are URL-safe.
- **User IDs:** `/users/{id}` with `whereNumber` prevents non-numeric IDs from reaching the controller.
- **Date Archives:** `/archive/{year}/{month}` with numeric constraints ensures valid date segments.
- **Localisation:** `/{locale}` constrained to `en|fr|de|es` restricts the route to supported languages.
- **API Versioning:** `/api/v{version}` with `whereNumber` ensures the version is a number.

### References

- Laravel Routing: Regular Expression Constraints — https://laravel.com/docs/12.x/routing#parameters-regular-expression-constraints
- Laravel API: RouteRegistrar (whereAlpha, whereNumber) — https://api.laravel.com/docs/6.x/Illuminate/Routing/RouteRegistrar.html

---

## References

- Laravel Routing Documentation — https://laravel.com/docs/12.x/routing
- Laravel 10.x Routing Documentation — https://laravel.com/framework/docs/10.x/routing
- Laravel 13.x Routing Documentation (Traditional Chinese) — https://laravel.com.tw/docs/13.x/routing
- Laravel API: Registrar Interface — https://api.laravel.com/docs/6.x/Illuminate/Contracts/Routing/Registrar.html
- Laravel API: RouteRegistrar (whereAlpha, whereNumber) — https://api.laravel.com/docs/6.x/Illuminate/Routing/RouteRegistrar.html
- Laravel Controllers Documentation — https://laravel.com/docs/12.x/controllers
- Laravel URL Generation Documentation — https://laravel.com/docs/12.x/urls
- Laravel Route Model Binding — https://laravel.com/docs/12.x/routing#route-model-binding
- Laravel Lumen Routing (Optional Parameters) — https://lumen.laravel.com/docs/routing
- Laravel 5.2 HTTP Routing (Legacy) — https://laravel.com/index.php/docs/5.2/routing