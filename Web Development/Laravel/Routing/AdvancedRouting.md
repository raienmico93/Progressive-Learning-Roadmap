# Laravel Advanced Routing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Advanced Routing encompasses the framework's specialized routing features that extend basic route definition, including resource controllers, fallback handling, redirects, view routes, singleton resources, rate limiting, signed URLs, route caching, and soft-delete model binding.

**Technical Definition**

Advanced routing features are implemented across several classes in the `Illuminate\Routing` namespace. `ResourceRegistrar` compiles RESTful resource routes; `Route::fallback()` registers a catch-all route for unmatched requests; `Route::redirect()` and `Route::view()` provide shortcuts for common patterns; `Route::singleton()` registers singular resources; the `ThrottleRequests` middleware enforces rate limits; `URL::signedRoute()` and `URL::temporarySignedRoute()` generate cryptographically signed URLs; `RouteCacheCommand` compiles routes into a single cached file; and `->withTrashed()` enables model binding for soft-deleted records.

**Beginner-Friendly Explanation**

Advanced routing is the collection of powerful tools that make Laravel's routing system production-ready. Beyond defining simple URLs, you can automatically generate RESTful APIs, catch 404 errors gracefully, protect routes with rate limits, create secure temporary links, and much more—all without writing repetitive code.

---

### Key Characteristics

- **Resourceful Routing**: Automatic generation of RESTful routes via `Route::resource()` and `Route::apiResource()`, including nested and shallow nesting options.
- **Graceful Fallbacks**: `Route::fallback()` provides a catch-all handler for unmatched requests, enabling custom 404 experiences.
- **Signed URL Security**: Cryptographically signed URLs prevent tampering and enable temporary, expiring links for public access.
- **Rate Limiting**: Built-in `throttle` middleware and named limiters protect routes from abuse.
- **Singleton Resources**: `Route::singleton()` registers routes for singular resources (e.g., user profile) that have no index method.
- **Soft-Delete Binding**: `->withTrashed()` allows route model binding to resolve soft-deleted records when needed.
- **Route Caching**: `php artisan route:cache` compiles all controller-based routes into a single file for production performance.
- **Version-Specific Features**: Singleton routes were introduced in Laravel 9.42; the `scopeBindings()` method for explicit scoping is available in Laravel 8+.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Solid understanding of basic routing** (verbs, parameters, naming, groups).
- **Familiarity with controllers and Eloquent models**.
- **Knowledge of middleware** for rate limiting implementation.

---

### Related Programming Areas

- **RESTful API Design**: Resource routes are the foundation of Laravel API development.
- **Security**: Signed URLs and rate limiting protect against tampering and abuse.
- **Performance Optimization**: Route caching and singleton resources improve application efficiency.
- **Eloquent ORM**: Route model binding with soft deletes integrates routing with Eloquent.
- **Multi-Tenancy**: Domain groups and scoped bindings enable tenant-aware routing.

---

## Core Concepts

---

### 1. Route Resources

#### Definitions

**Core Definition**

Resource routes automatically generate all seven RESTful routes (index, create, store, show, edit, update, destroy) for a controller, mapping them to standard URIs and HTTP verbs.

**Technical Definition**

`Route::resource()` invokes `ResourceRegistrar::register()`, which creates routes for the resource controller. `Route::apiResource()` excludes the `create` and `edit` routes (which present HTML forms). Nested resources use dot notation (`photos.comments`), and `->shallow()` generates shallow routes where child routes omit the parent ID. Scoping with `->scoped()` enables automatic parent-child binding. Resource routes can be customized with `->names()`, `->parameters()`, `->middleware()`, and `->except()`/`->only()`.

**Beginner-Friendly Explanation**

Resource routes are a shortcut that creates all the standard CRUD routes for a controller in one line. Instead of writing seven separate routes for listing, showing, creating, editing, updating, and deleting, you write one `Route::resource()` and Laravel does the rest.

#### Purposes

- To reduce boilerplate when defining RESTful CRUD routes.
- To enforce consistent URL and HTTP verb conventions across resources.
- To automatically generate route names and controller method mappings.
- To support nested resources with automatic scoping and shallow nesting.

#### Syntax Rules and Structure

**Complete General Syntax**

Full resource (7 routes)
```php
Route::resource('photos', PhotoController::class);
```

API resource (excludes create, edit)
```php
Route::apiResource('photos', PhotoController::class);
```
Nested resource
```php
Route::resource('photos.comments', CommentController::class);
```
Shallow nesting
```php
Route::resource('photos.comments', CommentController::class)->shallow();
```
Scoped nested resource
```php
Route::resource('photos.comments', CommentController::class)->scoped([
    'comment' => 'slug',
]);
```
Partial resource
```php
Route::resource('photos', PhotoController::class)->only(['index', 'show']);
Route::resource('photos', PhotoController::class)->except(['create', 'store']);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `resource('photos', ...)` | Registers 7 RESTful routes. |
| `apiResource('photos', ...)` | Registers 5 routes (excludes create/edit). |
| `'photos.comments'` | Nested resource using dot notation. |
| `->shallow()` | Child routes omit parent ID. |
| `->scoped()` | Enables automatic parent-child scoping. |
| `->only()` / `->except()` | Include/exclude specific routes. |

**Resource Route Table (Full Resource)**

| Verb | URI | Action | Route Name |
|------|-----|--------|------------|
| GET | `/photos` | index | `photos.index` |
| GET | `/photos/create` | create | `photos.create` |
| POST | `/photos` | store | `photos.store` |
| GET | `/photos/{photo}` | show | `photos.show` |
| GET | `/photos/{photo}/edit` | edit | `photos.edit` |
| PUT/PATCH | `/photos/{photo}` | update | `photos.update` |
| DELETE | `/photos/{photo}` | destroy | `photos.destroy` |

**Syntax Rules**

- The resource name should be plural (e.g., `photos`, `users`).
- The controller must have the corresponding methods.
- `apiResource` is recommended for API endpoints.
- Nested resources use dot notation: `'parent.children'`.
- Shallow nesting is useful when child IDs are globally unique.

**Constraints and Limitations**

- Resource routes cannot be cached if they use Closure-based custom routes.
- Nested resources with more than two levels are discouraged (consider shallow nesting).
- Scoped bindings require the parent model to have a relationship matching the child parameter name.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Resource Route**

**Controller**

```php
<?php
// app/Http/Controllers/PhotoController.php
namespace App\Http\Controllers;

class PhotoController extends Controller
{
    public function index() { return 'Photo index'; }
    public function create() { return 'Photo create form'; }
    public function store() { return 'Photo stored'; }
    public function show($id) { return "Photo {$id}"; }
    public function edit($id) { return "Edit photo {$id}"; }
    public function update($id) { return "Photo {$id} updated"; }
    public function destroy($id) { return "Photo {$id} deleted"; }
}
```

**Route**

```php
// routes/web.php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```

**Expected Output**

```
// GET /photos → "Photo index"
// GET /photos/create → "Photo create form"
// POST /photos → "Photo stored"
// GET /photos/1 → "Photo 1"
// GET /photos/1/edit → "Edit photo 1"
// PUT /photos/1 → "Photo 1 updated"
// DELETE /photos/1 → "Photo 1 deleted"
```

**Why This Output Occurs**

The `resource()` method registers all seven routes, mapping each HTTP verb and URI to the corresponding controller method.

---

**Example 2: Shallow Nested Resource**

**Controller**

```php
class PhotoCommentController extends Controller
{
    public function index($photoId) { return "Comments for photo {$photoId}"; }
    public function show($commentId) { return "Comment {$commentId}"; }
    public function store($photoId) { return "Comment stored for photo {$photoId}"; }
    public function update($commentId) { return "Comment {$commentId} updated"; }
    public function destroy($commentId) { return "Comment {$commentId} deleted"; }
}
```

**Route**

```php
Route::resource('photos.comments', PhotoCommentController::class)->shallow();
```

**Expected Output**

```
// GET /photos/1/comments → index (photo 1 comments)
// POST /photos/1/comments → store (create comment for photo 1)
// GET /comments/5 → show (comment 5, no photo ID needed)
// PUT /comments/5 → update (comment 5)
// DELETE /comments/5 → destroy (comment 5)
```

**Why This Output Occurs**

Shallow nesting omits the parent ID from child routes that can be uniquely identified by the child's own ID. The `index`, `store`, and `create` routes still include the parent ID because they operate on the collection.

---

**Example 3: Scoped Nested Resource**

**Route**

```php
Route::resource('users.posts', PostController::class)->scoped([
    'post' => 'slug',
]);
```

**Expected Output**

```
// GET /users/1/posts/my-post-slug → resolves post by slug, scoped to user 1
// Returns 404 if the post does not belong to user 1
```

**Why This Output Occurs**

The `scoped()` method instructs Laravel to resolve the `Post` model using the `slug` column and automatically scope it to the `User` model's `posts` relationship.

---

#### Real-World Cases with Explanation

**Case 1: Blog with Comments**

A blog application uses `Route::resource('posts.comments', CommentController::class)->shallow()` to manage comments on posts. Comments are listed and created within a post context, but individual comments are accessed by their own ID.

**Explanation**: Shallow nesting reduces URL complexity while maintaining the logical relationship.

**Case 2: Multi-Tenant SaaS API**

A multi-tenant API uses `Route::apiResource('tenants.users', UserController::class)->scoped()` to ensure users are always scoped to their tenant.

**Explanation**: Scoped bindings prevent cross-tenant data access.

---

### 2. Route Fallback

#### Definitions

**Core Definition**

A fallback route is a catch-all route that is executed when no other route matches the incoming request, typically used to render a custom 404 page.

**Technical Definition**

`Route::fallback()` registers a route that is only matched after all other routes have been exhausted. The fallback route should always be the last route registered. It receives the request and can return a view, redirect, or any response. Middleware from the route file's group (e.g., `web`) applies to the fallback route.

**Beginner-Friendly Explanation**

A fallback route is like a safety net. If someone visits a URL that doesn't match any of your defined routes, Laravel runs the fallback route instead of showing the default 404 error page.

#### Purposes

- To provide a custom 404 page instead of Laravel's default.
- To log or track 404 errors for analytics.
- To redirect users to relevant pages when they hit a broken link.
- To handle unmatched routes gracefully in SPAs or complex applications.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::fallback(function () {
    return view('errors.404');
});

Route::fallback(function () {
    return response()->view('errors.custom-404', [], 404);
});

Route::fallback(FallbackController::class);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Route::fallback()` | Registers the fallback route. |
| Closure/Controller | The handler for unmatched requests. |
| `response()->view(..., 404)` | Returns a 404 status with a view. |

**Syntax Rules**

- The fallback route must be the last route registered.
- Middleware from the route file's group applies automatically.
- The fallback route can use any HTTP method handler.
- Only one fallback route is allowed per application.

**Constraints and Limitations**

- The fallback route cannot be cached with `route:cache` if it uses a Closure.
- Fallback routes are not included in `route:list` by default in some versions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom 404 View**

**Code**

```php
// routes/web.php
Route::fallback(function () {
    return response()->view('errors.404', [], 404);
});
```

**View**

```blade
<!-- resources/views/errors/404.blade.php -->
<h1>Page Not Found</h1>
<p>The page you are looking for does not exist.</p>
<a href="/">Go home</a>
```

**Expected Output**

```
// Visiting /non-existent-page → renders custom 404 view with 404 status
```

**Why This Output Occurs**

The fallback route catches the unmatched request and returns the custom 404 view with the appropriate HTTP status code.

---

**Example 2: Fallback Controller**

**Controller**

```php
<?php
// app/Http/Controllers/FallbackController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class FallbackController extends Controller
{
    public function __invoke(Request $request)
    {
        // Log the 404 for analytics
        logger("404: {$request->fullUrl()}");
        
        return response()->view('errors.404', [
            'attemptedUrl' => $request->fullUrl(),
        ], 404);
    }
}
```

**Route**

```php
Route::fallback(FallbackController::class);
```

**Expected Output**

```
// Visiting /broken-link → logs the URL and renders custom 404 with the attempted URL
```

**Why This Output Occurs**

The fallback controller receives the request, logs it, and returns a custom 404 view with the attempted URL passed as data.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Site with Search**

An e-commerce site uses a fallback route to display a "Page not found" page that includes a search bar and links to popular categories, improving user experience for broken links.

**Explanation**: Fallback routes turn frustrating 404 errors into opportunities for user engagement.

**Case 2: Analytics Logging**

An application logs all unmatched requests to a database for analysis, helping the team identify broken links and missing routes.

**Explanation**: Fallback routes enable tracking of 404 errors for proactive maintenance.

---

### 3. Route Redirects

#### Definitions

**Core Definition**

Route redirects are routes that automatically redirect the user to another URI, eliminating the need for a full controller or Closure.

**Technical Definition**

`Route::redirect($from, $to, $status = 302)` registers a redirect route. The `permanentRedirect()` method uses a 301 status code. Redirect routes reserve the `destination` and `status` parameter names. Parameters in the source URI are automatically appended to the destination.

**Beginner-Friendly Explanation**

A redirect route sends users from one URL to another automatically. For example, `/here` can redirect to `/there`.

#### Purposes

- To handle URL changes without breaking existing links.
- To create short URLs that redirect to longer ones.
- To enforce canonical URLs (e.g., redirecting HTTP to HTTPS).
- To simplify common redirect patterns.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::redirect('/here', '/there');
Route::redirect('/here', '/there', 301);
Route::permanentRedirect('/here', '/there');
```

**Component Breakdown**

| Method | Status Code | Use Case |
|--------|-------------|----------|
| `redirect()` | 302 (temporary) | Temporary URL changes. |
| `permanentRedirect()` | 301 (permanent) | Permanent URL changes. |

**Syntax Rules**

- The first argument is the source URI, the second is the destination.
- The third argument (optional) is the HTTP status code.
- Parameters from the source URI are appended to the destination.
- Reserved parameter names: `destination`, `status`.

**Constraints and Limitations**

- Redirect routes cannot be used with route model binding.
- The `destination` and `status` parameter names are reserved.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Redirect**

**Code**

```php
Route::redirect('/old-page', '/new-page');
```

**Expected Output**

```
// GET /old-page → 302 redirect to /new-page
```

**Why This Output Occurs**

The redirect route returns a 302 response with the `Location` header set to `/new-page`.

---

**Example 2: Permanent Redirect**

**Code**

```php
Route::permanentRedirect('/legacy-url', '/modern-url');
```

**Expected Output**

```
// GET /legacy-url → 301 permanent redirect to /modern-url
```

**Why This Output Occurs**

The `permanentRedirect()` method returns a 301 status, informing search engines and browsers that the move is permanent.

---

#### Real-World Cases with Explanation

**Case 1: Blog URL Restructuring**

A blog changes its URL structure from `/blog/{id}` to `/posts/{slug}`. Redirect routes are added for the old URLs to preserve SEO.

**Explanation**: Redirect routes maintain link equity and user experience during URL changes.

---

### 4. Route Views

#### Definitions

**Core Definition**

A view route returns a Blade view directly without requiring a controller or Closure, simplifying the definition of static pages.

**Technical Definition**

`Route::view($uri, $view, $data = [], $status = 200, $headers = [])` registers a route that renders the specified view. The `view` method accepts a URI, a view name, an optional data array, and optional status/headers. Reserved parameter names: `view`, `data`, `status`, `headers`.

**Beginner-Friendly Explanation**

A view route lets you return a Blade template directly from a route definition, without writing a controller. It is perfect for static pages like "About Us" or "Contact."

#### Purposes

- To define static pages without controllers.
- To reduce boilerplate for simple view-returning routes.
- To pass data directly to views from route definitions.
- To simplify prototyping and small applications.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::view('/welcome', 'welcome');
Route::view('/about', 'about', ['name' => 'Taylor']);
Route::view('/contact', 'contact', [], 200, ['X-Custom' => 'value']);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `view('/uri', 'view.name')` | Renders the view at the URI. |
| `['name' => 'Taylor']` | Data passed to the view. |
| `[], 200, [...]` | Optional status and headers. |

**Syntax Rules**

- The first argument is the URI, the second is the view name.
- The third argument (optional) is an array of data.
- Reserved parameter names: `view`, `data`, `status`, `headers`.

**Constraints and Limitations**

- View routes cannot use controller logic.
- For dynamic data, use a controller instead.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Static About Page**

**Code**

```php
Route::view('/about', 'pages.about');
```

**View**

```blade
<!-- resources/views/pages/about.blade.php -->
<h1>About Us</h1>
<p>We build great things.</p>
```

**Expected Output**

```
// GET /about → renders the about view
```

**Why This Output Occurs**

The view route directly renders the `pages.about` view without a controller.

---

**Example 2: View with Data**

**Code**

```php
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

**View**

```blade
<!-- resources/views/welcome.blade.php -->
<h1>Welcome, {{ $name }}!</h1>
```

**Expected Output**

```html
<h1>Welcome, Taylor!</h1>
```

**Why This Output Occurs**

The `$name` variable is passed to the view via the data array.

---

#### Real-World Cases with Explanation

**Case 1: Landing Pages**

A marketing site uses view routes for static landing pages (pricing, features, contact), avoiding unnecessary controller creation.

**Explanation**: View routes are ideal for pages that do not require dynamic logic.

---

### 5. Route Singleton

#### Definitions

**Core Definition**

Singleton routes are resource routes for singular resources that have exactly one instance (or zero or one) in the application, such as a user's profile or account settings.

**Technical Definition**

`Route::singleton($name, $controller)` registers routes for a singleton resource. Singleton resources do not have an `index` method (since there is only one) and do not include an ID in the URI. The available routes are `show`, `create`, `store`, `edit`, `update`, and `destroy`. `apiSingleton()` excludes `create` and `edit`. Singleton resources can be nested within standard resources.

**Beginner-Friendly Explanation**

A singleton route is for things that only have one instance, like your user profile. Instead of `/profile/{id}`, you just use `/profile` because there is only one profile per user.

#### Purposes

- To define routes for singular resources without unnecessary ID parameters.
- To model resources like "profile," "settings," or "account" correctly.
- To reduce URL complexity for one-of-one resources.
- To enable nested singleton resources (e.g., a user's profile).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Route::singleton('profile', ProfileController::class);
Route::apiSingleton('profile', ProfileController::class);

// Nested singleton
Route::singleton('users.profile', ProfileController::class);
```

**Component Breakdown**

| Method | Routes Generated |
|--------|-----------------|
| `singleton()` | show, create, store, edit, update, destroy |
| `apiSingleton()` | show, store, update, destroy |

**Singleton Route Table**

| Verb | URI | Action |
|------|-----|--------|
| GET | `/profile` | show |
| GET | `/profile/create` | create |
| POST | `/profile` | store |
| GET | `/profile/edit` | edit |
| PUT/PATCH | `/profile` | update |
| DELETE | `/profile` | destroy |

**Syntax Rules**

- Singleton resources do not have an `index` route.
- The URI does not contain an ID parameter.
- Nested singletons use dot notation.

**Constraints and Limitations**

- Singleton routes were introduced in Laravel 9.42.
- The controller must implement the corresponding methods.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: User Profile Singleton**

**Controller**

```php
class ProfileController extends Controller
{
    public function show() { return 'User profile'; }
    public function edit() { return 'Edit profile form'; }
    public function update() { return 'Profile updated'; }
    public function destroy() { return 'Profile deleted'; }
}
```

**Route**

```php
Route::singleton('profile', ProfileController::class);
```

**Expected Output**

```
// GET /profile → "User profile"
// GET /profile/edit → "Edit profile form"
// PUT /profile → "Profile updated"
// DELETE /profile → "Profile deleted"
```

**Why This Output Occurs**

The singleton route registers routes without an ID parameter, reflecting that there is only one profile.

---

**Example 2: Nested Singleton**

**Route**

```php
Route::singleton('users.profile', ProfileController::class);
```

**Expected Output**

```
// GET /users/1/profile → show profile for user 1
// PUT /users/1/profile → update profile for user 1
```

**Why This Output Occurs**

The singleton is scoped to the parent user resource, so the user ID is included in the URI.

---

#### Real-World Cases with Explanation

**Case 1: Account Settings**

A SaaS application uses `Route::singleton('account', AccountController::class)` for account settings, since each user has exactly one account.

**Explanation**: Singleton routes model one-to-one relationships correctly.

---

### 6. Rate Limiting

#### Definitions

**Core Definition**

Rate limiting restricts the number of requests a client can make to a route within a specified time window, protecting against abuse and ensuring fair usage.

**Technical Definition**

Laravel provides the `throttle` middleware (`Illuminate\Routing\Middleware\ThrottleRequests`) for rate limiting. The middleware accepts either a simple limit (`throttle:60,1` for 60 requests per minute) or a named limiter (`throttle:api`). Named limiters are defined in a service provider using `RateLimiter::for('name', function (Request $request) { ... })`. The limiter can return a `Limit` instance with custom response logic.

**Beginner-Friendly Explanation**

Rate limiting is like a bouncer who only lets you into a club a certain number of times per minute. It prevents people from hammering your API with too many requests.

#### Purposes

- To protect routes from brute-force attacks and abuse.
- To ensure fair resource allocation among users.
- To implement tiered rate limits (e.g., free vs. premium users).
- To return informative 429 responses with retry headers.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Simple throttle
Route::middleware('throttle:60,1')->get('/api/data', ...);

// Named limiter
Route::middleware('throttle:api')->get('/api/data', ...);
```

```php
// Defining a named limiter (in a service provider)
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `throttle:60,1` | 60 requests per 1 minute. |
| `throttle:api` | Uses the named `api` limiter. |
| `RateLimiter::for()` | Defines a named limiter. |
| `Limit::perMinute()` | Creates a per-minute limit. |

**Syntax Rules**

- The `throttle` middleware accepts `limit,decay` in minutes or a named limiter.
- Named limiters are defined in a service provider's `boot()` method.
- The `by()` method customizes the key (user ID, IP, etc.).
- A 429 response is returned when the limit is exceeded.

**Constraints and Limitations**

- Rate limiting requires a cache driver (Redis recommended for distributed systems).
- Named limiters must be registered before routes are defined.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Simple Throttle**

**Code**

```php
Route::middleware('throttle:5,1')->get('/limited', function () {
    return 'Limited route';
});
```

**Expected Output**

```
// First 5 requests: "Limited route"
// 6th request: 429 Too Many Requests
```

**Why This Output Occurs**

The `throttle:5,1` middleware allows 5 requests per minute per IP. The 6th request is rejected with a 429 status.

---

**Example 2: Named Limiter with User-Based Key**

**Service Provider**

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

**Route**

```php
Route::middleware('throttle:api')->get('/api/data', ...);
```

**Expected Output**

```
// Each authenticated user gets 60 requests per minute
// Unauthenticated users are limited by IP
```

**Why This Output Occurs**

The named limiter `api` uses the authenticated user's ID as the key, falling back to IP for guests.

---

#### Real-World Cases with Explanation

**Case 1: API Rate Limiting**

A public API uses a named limiter that allows 60 requests per minute for authenticated users and 10 for guests.

**Explanation**: Named limiters enable differentiated rate limits based on user tier.

**Case 2: Login Attempt Throttling**

A login route uses `throttle:5,1` to prevent brute-force password attacks.

**Explanation**: Rate limiting protects authentication endpoints from abuse.

---

### 7. Signed URLs

#### Definitions

**Core Definition**

Signed URLs are URLs with a cryptographic hash appended to the query string, allowing Laravel to verify that the URL has not been tampered with since it was generated.

**Technical Definition**

`URL::signedRoute($name, $parameters)` generates a URL with a signature. `URL::temporarySignedRoute($name, $expiration, $parameters)` adds an expiration timestamp. The `hasValidSignature()` method on the request validates the signature. The `ValidateSignature` middleware (`signed`) automatically rejects invalid signatures with a 403 response. The `hasValidSignatureWhileIgnoring()` method allows ignoring specific query parameters.

**Beginner-Friendly Explanation**

A signed URL is like a tamper-proof ticket. It has a special code that proves it was created by your application and hasn't been modified. You can also make it expire after a certain time.

#### Purposes

- To protect public URLs from unauthorized modification.
- To create temporary, expiring links for file downloads, password resets, or unsubscribe pages.
- To authenticate users without requiring a login for specific actions.
- To prevent URL manipulation attacks.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\URL;

// Permanent signed URL
$url = URL::signedRoute('unsubscribe', ['user' => 1]);

// Temporary signed URL (expires in 30 minutes)
$url = URL::temporarySignedRoute(
    'unsubscribe',
    now()->addMinutes(30),
    ['user' => 1]
);
```

```php
// Validating
if (! $request->hasValidSignature()) {
    abort(401);
}

// Middleware
Route::get('/unsubscribe/{user}', ...)->name('unsubscribe')->middleware('signed');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `signedRoute()` | Creates a permanent signed URL. |
| `temporarySignedRoute()` | Creates an expiring signed URL. |
| `hasValidSignature()` | Validates the signature. |
| `middleware('signed')` | Middleware for automatic validation. |

**Syntax Rules**

- Signed URLs require a named route.
- The `ValidateSignature` middleware must be registered in the HTTP kernel.
- Invalid signatures return a 403 response when using middleware.
- Expired temporary URLs return a 403 response.

**Constraints and Limitations**

- Signed URLs are not encrypted; the payload is visible.
- The signature depends on the application's `APP_KEY`; rotating the key invalidates all signed URLs.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Temporary Signed URL for Unsubscribe**

**Route**

```php
Route::get('/unsubscribe/{user}', function (Request $request, User $user) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }
    return "Unsubscribed user {$user->id}";
})->name('unsubscribe');
```

**Generating the URL**

```php
use Illuminate\Support\Facades\URL;

$url = URL::temporarySignedRoute(
    'unsubscribe',
    now()->addMinutes(30),
    ['user' => 1]
);
// https://example.com/unsubscribe/1?expires=1745000000&signature=abc123...
```

**Expected Output**

```
// Valid URL within 30 minutes → "Unsubscribed user 1"
// Expired or tampered URL → 401 Unauthorized
```

**Why This Output Occurs**

The signature is verified against the URL parameters and expiration timestamp. If valid, the route executes; otherwise, a 401 is returned.

---

**Example 2: Signed Route Middleware**

**Route**

```php
Route::post('/unsubscribe/{user}', function (Request $request, User $user) {
    return "Unsubscribed user {$user->id}";
})->name('unsubscribe')->middleware('signed');
```

**Expected Output**

```
// Valid signature → "Unsubscribed user 1"
// Invalid signature → 403 Forbidden
```

**Why This Output Occurs**

The `signed` middleware automatically validates the signature and returns a 403 response if invalid.

---

#### Real-World Cases with Explanation

**Case 1: Email Unsubscribe Links**

A newsletter application sends emails with temporary signed unsubscribe links. The links expire after 30 days and cannot be tampered with.

**Explanation**: Signed URLs enable secure, one-click unsubscribe actions without requiring login.

**Case 2: Temporary File Downloads**

A file-sharing application generates temporary signed URLs for downloads. The URLs expire after 24 hours, and the signature prevents users from accessing files they shouldn't.

**Explanation**: Signed URLs provide secure, time-limited access to private resources.

---

### 8. Route Caching

#### Definitions

**Core Definition**

Route caching is the process of compiling all application routes into a single PHP file, which is loaded instead of the individual route files on each request, improving performance.

**Technical Definition**

The `php artisan route:cache` command serializes the `RouteCollection` and stores it in `bootstrap/cache/routes-v7.php` (the version suffix may vary). On subsequent requests, Laravel loads the cached file instead of parsing the route files. The cache must be cleared (`route:clear`) after any route changes. Closure-based routes cannot be cached; all routes must use controller actions.

**Beginner-Friendly Explanation**

Route caching speeds up your application by saving all your routes in a single file. Instead of Laravel reading every route file on every request, it reads one cached file.

#### Purposes

- To reduce the overhead of parsing route definitions on each request.
- To improve application bootstrapping performance in production.
- To provide a single, optimized file for route resolution.
- To complement other caching strategies (config, view).

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Cache routes
php artisan route:cache

# Clear route cache
php artisan route:clear

# Optimize (includes route cache)
php artisan optimize
```

**Component Breakdown**

| Command | Description |
|---------|-------------|
| `route:cache` | Compiles and caches all routes. |
| `route:clear` | Removes the cached route file. |
| `optimize` | Runs `config:cache` and `route:cache`. |

**Syntax Rules**

- All routes must use controller actions; Closure routes cause an error.
- The cache file is stored in `bootstrap/cache/`.
- Route caching should only be used in production, not development.
- The cache must be cleared after route changes.

**Constraints and Limitations**

- Duplicate route names cause `route:cache` to fail.
- Closure routes cannot be cached (move to controllers).
- Route caching can mask changes during development; never cache in dev.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Caching Routes**

**Command**

```bash
php artisan route:cache
```

**Expected Output (Terminal)**

```
Route cache cleared successfully.
Routes cached successfully.
```

**Expected Output (File)**

```
// bootstrap/cache/routes-v7.php contains the compiled routes
```

**Why This Output Occurs**

The command serializes the route collection into a single PHP file.

---

**Example 2: Clearing Route Cache**

**Command**

```bash
php artisan route:clear
```

**Expected Output (Terminal)**

```
Route cache cleared successfully.
```

**Why This Output Occurs**

The cached file is deleted, forcing Laravel to re-parse the route files on the next request.

---

#### Real-World Cases with Explanation

**Case 1: Production Deployment**

A CI/CD pipeline runs `php artisan route:cache` during deployment to precompile routes.

**Explanation**: Route caching is a standard production optimization.

**Case 2: Development Workflow**

A developer runs `php artisan route:clear` after adding a new route to ensure it is recognized.

**Explanation**: Route caching must be managed carefully during development.

---

### 9. Route Model Binding with Soft Deletes

#### Definitions

**Core Definition**

Route model binding with soft deletes allows route parameters to resolve to soft-deleted Eloquent models by chaining the `withTrashed()` method onto the route definition.

**Technical Definition**

By default, implicit model binding does not retrieve soft-deleted models. The `->withTrashed()` method on a route instructs the binding resolver to include trashed models in the query. For explicit binding, the `Route::bind()` closure can use `withTrashed()` on the query. The `onlyTrashed()` method can be used to bind only trashed models.

**Beginner-Friendly Explanation**

When you soft-delete a record, it is not actually removed from the database—it is just marked as deleted. By default, Laravel's route model binding won't find soft-deleted records. Using `withTrashed()` tells Laravel to include them.

#### Purposes

- To allow access to soft-deleted records for restoration or permanent deletion.
- To implement admin panels that manage trashed records.
- To support "undo delete" functionality.
- To provide a complete audit trail of all records, including deleted ones.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Implicit binding with soft deletes
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

```php
// Explicit binding with soft deletes
Route::bind('user', function ($value) {
    return User::withTrashed()->where('id', $value)->firstOrFail();
});
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `->withTrashed()` | Includes soft-deleted models in binding. |
| `onlyTrashed()` | Binds only soft-deleted models. |

**Syntax Rules**

- `withTrashed()` is chained onto the route definition.
- For explicit binding, use `withTrashed()` in the query.
- `withTrashed()` works with both implicit and explicit binding.

**Constraints and Limitations**

- Soft deletes require the `SoftDeletes` trait on the model.
- The `withTrashed()` method must be called on the route, not the model.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Implicit Binding with Soft Deletes**

**Model**

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;
}
```

**Route**

```php
Route::get('/posts/{post}', function (Post $post) {
    return $post->title;
})->withTrashed();
```

**Expected Output**

```
// GET /posts/1 (even if soft-deleted) → returns the post title
// Without withTrashed() → 404 Not Found
```

**Why This Output Occurs**

The `withTrashed()` method instructs the implicit binding to include soft-deleted records in the query.

---

**Example 2: Explicit Binding with Soft Deletes**

**Service Provider**

```php
use App\Models\User;

public function boot(): void
{
    Route::bind('user', function ($value) {
        return User::withTrashed()->where('id', $value)->firstOrFail();
    });
}
```

**Expected Output**

```
// Any route using {user} will resolve soft-deleted users
```

**Why This Output Occurs**

The explicit binding closure uses `withTrashed()` to include soft-deleted models in the resolution.

---

#### Real-World Cases with Explanation

**Case 1: Admin Restore Panel**

An admin panel uses `Route::get('/admin/posts/{post}/restore', ...)` with `->withTrashed()` to allow administrators to restore soft-deleted posts.

**Explanation**: Soft-delete binding enables administrative recovery of deleted records.

---

### 10. Advanced Routing Best Practices

#### Definitions

**Core Definition**

Advanced routing best practices are the recommended guidelines for using Laravel's specialized routing features effectively, securely, and maintainably.

**Technical Definition**

Best practices encompass resource route design, fallback handling, signed URL usage, rate limiting strategy, route caching, and soft-delete binding. They aim to balance performance, security, and developer experience.

#### Key Best Practices

1. **Use `apiResource` for API Endpoints**: Always use `Route::apiResource()` instead of `Route::resource()` for API routes to exclude the HTML form routes (`create`, `edit`).

2. **Prefer Shallow Nesting for Resources with Unique IDs**: Use `->shallow()` when child resources have globally unique identifiers (e.g., auto-incrementing IDs), reducing URL complexity.

3. **Use `->scoped()` for Multi-Tenant Resources**: When nested resources belong to a parent, use `->scoped()` to automatically enforce parent-child relationships and prevent cross-tenant access.

4. **Always Register Fallback Routes Last**: The fallback route must be the final route registered to ensure it only catches genuinely unmatched requests.

5. **Use Named Routes with Signed URLs**: Signed URLs require named routes. Always assign names to routes that will be signed.

6. **Prefer Temporary Signed URLs Over Permanent Ones**: Use `temporarySignedRoute()` with an expiration for most use cases (downloads, unsubscribe links) to limit exposure.

7. **Apply Rate Limiting to Authentication Endpoints**: Protect login, registration, and password reset routes with `throttle:5,1` or a named limiter to prevent brute-force attacks.

8. **Define Named Rate Limiters for API Tiers**: Use `RateLimiter::for()` to define custom limiters for different user tiers (free, premium, enterprise).

9. **Cache Routes in Production Only**: Run `php artisan route:cache` during deployment, never during development. Always clear the cache after route changes.

10. **Convert Closure Routes to Controllers for Caching**: Route caching requires all routes to be controller-based. Use invokable controllers for single-action routes.

11. **Use `withTrashed()` Sparingly**: Only include soft-deleted models in binding when necessary (e.g., admin restore panels). Default binding should exclude trashed records.

12. **Document Advanced Routing Patterns**: Maintain clear documentation of resource routes, rate limiters, and signed URL usage, especially in team environments.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Well-Structured API Resource with Rate Limiting and Scoping**

**Route**

```php
// routes/api.php
use App\Http\Controllers\Api\TenantUserController;

Route::middleware(['auth:sanctum', 'throttle:api'])
    ->prefix('v1')
    ->name('api.v1.')
    ->group(function () {
        Route::apiResource('tenants.users', TenantUserController::class)
            ->scoped()
            ->names([
                'index' => 'tenants.users.index',
                'store' => 'tenants.users.store',
                'show' => 'tenants.users.show',
                'update' => 'tenants.users.update',
                'destroy' => 'tenants.users.destroy',
            ]);
    });
```

**Rate Limiter Definition**

```php
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

**Expected Output**

```
// GET /api/v1/tenants/1/users → scoped to tenant 1, rate limited to 60/min
// POST /api/v1/tenants/1/users → create user for tenant 1
// GET /api/v1/tenants/1/users/5 → show user 5 (scoped to tenant 1)
```

**Why This Output Occurs**

The resource routes are scoped, named, and rate-limited. The `scoped()` method ensures users belong to the tenant, and the named limiter enforces the rate limit.

---

**Example 2: Fallback Route with Signed URL**

**Route**

```php
// routes/web.php
Route::get('/unsubscribe/{user}', function (Request $request, User $user) {
    return "Unsubscribed user {$user->id}";
})->name('unsubscribe')->middleware('signed');

Route::fallback(function () {
    return response()->view('errors.404', [], 404);
});
```

**Expected Output**

```
// Valid signed URL → "Unsubscribed user 1"
// Invalid signed URL → 403 Forbidden
// Unmatched route → custom 404 view
```

**Why This Output Occurs**

The signed middleware validates the URL, and the fallback route catches unmatched requests.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise API with Tiered Rate Limits**

An enterprise API defines three named rate limiters: `api.free` (60/min), `api.pro` (1000/min), and `api.enterprise` (10000/min). The middleware dynamically selects the limiter based on the user's plan.

**Explanation**: Named limiters enable differentiated rate limiting without duplicating route definitions.

**Case 2: SaaS with Soft-Delete Recovery**

A SaaS application allows administrators to restore soft-deleted records via routes using `->withTrashed()`. The admin panel lists trashed records and provides restore/permanent-delete actions.

**Explanation**: Soft-delete binding enables complete lifecycle management of records.

---

## References

- Routing – Laravel 12.x Documentation — https://laravel.com/docs/12.x/routing
- Routing – Laravel 11.x Documentation — https://laravel.com/docs/11.x/routing
- Routing – Laravel 10.x Documentation — https://laravel.com/docs/10.x/routing
- Controllers – Laravel 12.x Documentation — https://laravel.com/docs/12.x/controllers
- URL Generation – Laravel 12.x Documentation — https://laravel.com/docs/12.x/urls#signed-urls
- Rate Limiting – Laravel 12.x Documentation — https://laravel.com/docs/12.x/routing#rate-limiting
- Laravel News: Laravel 9.42 Singleton Route Resources — https://laravel-news.com/laravel-9-42-singleton-route-resources
- Laravel News: Handling Unmatched Routes in Laravel — https://laravel-news.com/handling-unmatched-routes-in-laravel
- Laravel News: Route Optimization through Shallow Resource Architecture — https://laravel-news.com/laravel-shallow-resource-routing
- Laravel Daily: Route Model Binding Soft-Deleted Models — https://laraveldaily.com/post/laravel-route-model-binding-soft-deleted-models
- Web Coding Center: Special Routes – Redirect, View, Fallback — https://webcodingcenter.com/laravel/special-routes--redirect-view-fallback-routes.html
- Laravel API: ResourceRegistrar — https://api.laravel.com/docs/12.x/Illuminate/Routing/ResourceRegistrar.html
- Laravel API: PendingSingletonResourceRegistration — https://api.laravel.com/docs/12.x/Illuminate/Routing/PendingSingletonResourceRegistration.html
- Laravel API: ThrottleRequests Middleware — https://api.laravel.com/docs/12.x/Illuminate/Routing/Middleware/ThrottleRequests.html
- GitHub: Laravel Framework – CompilesStacks (Route Caching) — https://github.com/laravel/framework/blob/12.x/src/Illuminate/View/Compilers/Concerns/CompilesStacks.php

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 12.x documentation. Key version-specific features: Singleton routes were introduced in Laravel 9.42; the `scopeBindings()` method for explicit scoping is available in Laravel 8+; the `withTrashed()` method for route model binding has been available since Laravel 5.2; signed URLs require the `ValidateSignature` middleware to be registered; route caching requires all routes to be controller-based. Developers should always consult the documentation for their specific Laravel version.