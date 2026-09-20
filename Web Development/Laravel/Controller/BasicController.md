# Laravel Controller Fundamentals & Lifecycle: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A Laravel controller is a PHP class that organises related HTTP request-handling logic into a single, cohesive unit, acting as the intermediary between route definitions and the application's business logic.

**Technical Definition:** In Laravel's MVC architecture, controllers are classes stored in `app/Http/Controllers` that extend `Illuminate\Routing\Controller` (optionally) and receive resolved dependencies through the service container. When the router matches an incoming request to a controller action, it resolves the controller instance from the IoC container, injects dependencies into the constructor, then invokes the action method—injecting route parameters and method-level dependencies. The action returns a `Response` instance (or a value convertible to one), which passes back through the middleware stack before being sent to the client.

**Beginner-Friendly Explanation:** Think of a controller as a traffic director at a busy intersection. When a request arrives (a car approaching), the router (the traffic signal) tells it which controller method (which lane) to go to. The controller method then does the work—gathering data, talking to the database, and deciding what to send back (the response). Instead of cramming all this logic into route files, controllers keep everything organised by topic (users, posts, orders).

### Key Characteristics

- **Organised Request Handling:** Groups related request logic into a single class (e.g., `UserController` handles all user-related requests).
- **Service Container Integration:** Controllers are resolved through Laravel's IoC container, enabling automatic dependency injection.
- **Method Flexibility:** A controller can have any number of public methods, each responding to a specific route.
- **Base Class Optional:** Controllers are not required to extend a base class, though extending `App\Http\Controllers\Controller` is conventional.
- **Middleware Support:** Middleware can be assigned at the route level or within the controller via the `HasMiddleware` interface.
- **Response Versatility:** Controller actions can return views, JSON, redirects, files, and more.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (controller fundamentals are stable across these versions; some features like `HasMiddleware` interface are available from Laravel 10+).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `app/Http/Controllers` directory.
- Basic understanding of routing, HTTP verbs, and the service container.

### Related Programming Areas

- **MVC Architecture:** Controllers are the "C" in Model-View-Controller, mediating between models and views.
- **Routing:** Routes map URIs and HTTP verbs to controller actions.
- **Middleware:** Controllers integrate with middleware for authentication, rate limiting, and request filtering.
- **Service Container:** Dependency injection into controllers is powered by Laravel's IoC container.
- **Request Lifecycle:** Controllers occupy a specific phase in the HTTP request lifecycle, after middleware and before response generation.

### Core Concepts / Features

1. The Controller Purpose
2. CLI Code Generation
3. Dependency Injection (DI)
4. Unified Response Objects

---

## 1. The Controller Purpose

### Definitions

**Core Definition:** The controller's purpose is to act as the traffic director of the application—intercepting incoming HTTP requests, gathering data from models or services, and returning appropriate responses.

**Technical Definition:** In the request lifecycle, the router matches an incoming request to a controller action. The `Route` instance's `run()` method invokes the controller via `ControllerDispatcher`, which resolves the controller from the container and calls the specified method with resolved parameters. The controller method performs business logic and returns a `Response`, which the kernel then sends back through the middleware stack to the client.

**Beginner-Friendly Explanation:** When someone visits a URL, Laravel doesn't just dump everything into one giant file. Instead, the router says "this is a user-related request, so I'll send it to the UserController." The controller method then does the actual work—fetching data, validating input, and deciding what the user sees next.

### Purposes

- To organise related request-handling logic into a single class instead of scattering it across route files.
- To provide a clear separation of concerns between routing, request handling, and response generation.
- To enable dependency injection and testability by making controllers resolvable from the service container.
- To support middleware assignment at both the route and controller level.
- To facilitate the organisation of large applications by grouping endpoints by domain area (users, posts, orders).

### Syntax Rules and Structure

**Complete General Syntax — Basic Controller:**

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `namespace App\Http\Controllers;` | The default namespace for all controllers. |
| `class UserController extends Controller` | The controller class, extending the base controller (optional). |
| `public function show(string $id): View` | A public action method that accepts route parameters. |
| `return view(...)` | Returns a `View` instance as the response. |

**Complete General Syntax — Routing to a Controller:**

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `UserController::class` | References the controller class. |
| `'show'` | The method name to invoke. |
| `{id}` | The route parameter passed to the method. |

**Syntax Rules:**

- Controllers are stored in `app/Http/Controllers` by default.
- Controller methods should be `public` to be routable.
- Controllers may extend a base controller class for shared functionality.
- Route parameters are injected after any method dependencies (e.g., `Request`).
- Single-action controllers use the `__invoke` method and are routed by class name only.

**Constraints and Limitations:**

- **Method visibility:** Only `public` methods can be routed to.
- **Route parameter order:** Route parameters must come after injected dependencies in the method signature.
- **Namespace:** Controllers must be in the correct namespace to be resolvable.
- **Route caching:** Controller-based routes can be cached; Closure routes cannot.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Controller with Route Parameter**

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): View
    {
        // Find the user or throw a 404
        $user = User::findOrFail($id);

        // Return the view with the user data
        return view('user.profile', ['user' => $user]);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::get('/user/{id}', [UserController::class, 'show']);
```

**Setup:** Create the controller via `php artisan make:controller UserController`. Create a `user/profile.blade.php` view.

**Expected Output:** `GET /user/1` renders the profile view with user 1's data. `GET /user/9999` returns a 404 error.

**Why:** The router matches `/user/{id}` to `UserController@show`. The `{id}` parameter is injected into the `show` method, which queries the database and returns a view.

---

**Example 2: Single-Action Controller**

```php
<?php
// app/Http/Controllers/ProvisionServer.php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     */
    public function __invoke()
    {
        // Provisioning logic...
        return response()->json(['status' => 'provisioned']);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\ProvisionServer;
use Illuminate\Support\Facades\Route;

// No method specified — invokes __invoke
Route::post('/server', ProvisionServer::class);
```

**Setup:** Generate via `php artisan make:controller ProvisionServer --invokable`.

**Expected Output:** `POST /server` invokes the `__invoke` method and returns `{"status":"provisioned"}`.

**Why:** Single-action controllers dedicate the entire class to one action. The router automatically invokes `__invoke` when only the class name is provided.

---

**Example 3: Controller with Middleware**

```php
<?php
// app/Http/Controllers/DashboardController.php

namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class DashboardController extends Controller implements HasMiddleware
{
    /**
     * Get the middleware that should be assigned to the controller.
     */
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('verified', only: ['index']),
        ];
    }

    public function index()
    {
        return view('dashboard');
    }

    public function settings()
    {
        return view('settings');
    }
}
```

**Expected Output:** `GET /dashboard` requires both `auth` and `verified` middleware. `GET /settings` requires only `auth`.

**Why:** The `HasMiddleware` interface allows controllers to declare their middleware statically. The `Middleware` class supports `only` and `except` for fine-grained control.

### Real-World Cases

- **User Management:** `UserController` handles all user-related requests (index, show, store, update, destroy).
- **Order Processing:** `OrderController` manages order creation, payment processing, and history.
- **API Endpoints:** `Api\UserController` returns JSON responses for mobile and SPA clients.
- **Admin Panels:** `Admin\DashboardController` aggregates statistics and renders the admin dashboard.
- **Webhook Handlers:** `WebhookController` receives and processes third-party webhook payloads.

### References

- Laravel Controllers Documentation — https://laravel.com/docs/12.x/controllers
- Laravel Request Lifecycle — https://laravel.com/docs/12.x/lifecycle

---

## 2. CLI Code Generation

### Definitions

**Core Definition:** Laravel's Artisan CLI provides the `make:controller` command to scaffold controller classes with optional flags for resource methods, API-only methods, and model binding.

**Technical Definition:** The `make:controller` command invokes the `ControllerMakeCommand` class, which uses stub files to generate controller code. The `--resource` flag includes the seven RESTful methods; `--api` excludes `create` and `edit`; `--model` binds the controller to an Eloquent model, enabling route model binding and injecting the model into methods; `--invokable` generates a single `__invoke` method; `--singleton` generates a singleton resource controller.

**Beginner-Friendly Explanation:** Instead of writing boilerplate controller code by hand, you can run a terminal command and Laravel generates the file for you—with all the methods you need already stubbed out. It's like having a template that fills in the blanks for you.

### Purposes

- To scaffold controllers quickly without writing boilerplate code.
- To generate resource controllers with all seven RESTful methods pre-defined.
- To generate API controllers with only the five stateless methods.
- To bind a controller to an Eloquent model for automatic route model binding.
- To generate single-action (invokable) controllers for focused functionality.
- To generate singleton controllers for single-record resources.

### Syntax Rules and Structure

**Complete General Syntax — Basic Controller:**

```bash
php artisan make:controller UserController
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `php artisan` | Laravel's CLI entry point. |
| `make:controller` | The controller generation command. |
| `UserController` | The class name (creates `app/Http/Controllers/UserController.php`). |

**Complete General Syntax — Resource Controller:**

```bash
php artisan make:controller PostController --resource
# or shorthand: -r
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--resource` / `-r` | Generates the seven RESTful methods: `index`, `create`, `store`, `show`, `edit`, `update`, `destroy`. |

**Complete General Syntax — API Controller:**

```bash
php artisan make:controller Api/PostController --api
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--api` | Generates five methods (excludes `create` and `edit`). |
| `Api/` | Creates the controller in a subdirectory namespace. |

**Complete General Syntax — Controller with Model:**

```bash
php artisan make:controller PostController --resource --model=Post
# or shorthand: -r -m Post
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--model=Post` / `-m Post` | Binds the controller to the `Post` model, injecting it into methods. |

**Complete General Syntax — Invokable Controller:**

```bash
php artisan make:controller ProvisionServer --invokable
# or shorthand: -i
```

**Complete General Syntax — Singleton Controller:**

```bash
php artisan make:controller ProfileController --singleton
```

**Syntax Rules:**

- The controller name should follow PascalCase (e.g., `UserController`).
- Subdirectories can be specified with forward slashes (e.g., `Admin/DashboardController`).
- The `--resource` flag generates the full CRUD method set.
- The `--api` flag is equivalent to `--resource` minus `create` and `edit`.
- The `--model` flag injects the model into methods via implicit route model binding.
- The `--invokable` flag creates a single-action controller with only `__invoke`.
- The `--singleton` flag creates a controller for single-record resources.

**Constraints and Limitations:**

- **Model binding:** The `--model` flag requires the model to exist before generation.
- **Controller placement:** By default, controllers are created in `app/Http/Controllers`; subdirectories create nested namespaces.
- **Resource methods:** The `--resource` flag generates all seven methods, even if you only need a few. Use `->only()` or `->except()` on the route to limit registered routes.
- **Singleton issue:** A known bug exists where `make:controller --singleton --model` does not correctly bind the model.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Controller Generation**

```bash
php artisan make:controller UserController
```

**Expected Output:** Creates `app/Http/Controllers/UserController.php` with a basic class extending `Controller`.

**Why:** This is the simplest form of controller generation. The generated file contains a namespace declaration, imports, and an empty class extending the base controller.

---

**Example 2: Resource Controller with Model Binding**

```bash
php artisan make:controller PostController --resource --model=Post
```

**Expected Output:** Creates `app/Http/Controllers/PostController.php` with seven methods, each type-hinting `Post $post` where applicable.

**Generated Method Signatures:**

```php
public function index() { }
public function create() { }
public function store(Request $request) { }
public function show(Post $post) { }
public function edit(Post $post) { }
public function update(Request $request, Post $post) { }
public function destroy(Post $post) { }
```

**Why:** The `--model` flag binds the controller to the `Post` model. Methods `show`, `edit`, `update`, and `destroy` receive the model instance through route model binding.

---

**Example 3: API Controller in a Subdirectory**

```bash
php artisan make:controller Api/V1/UserController --api
```

**Expected Output:** Creates `app/Http/Controllers/Api/V1/UserController.php` with namespace `App\Http\Controllers\Api\V1` and five methods (`index`, `store`, `show`, `update`, `destroy`).

**Why:** The subdirectory path creates a nested namespace, keeping API controllers organised and separated from web controllers. The `--api` flag excludes the form-display methods.

### Real-World Cases

- **Rapid Prototyping:** Generate resource controllers for all CRUD entities during initial development.
- **API Development:** Use `--api` to generate JSON-only controllers for mobile and SPA clients.
- **Admin Panels:** Generate controllers in an `Admin/` subdirectory for administrative endpoints.
- **Single-Purpose Endpoints:** Use `--invokable` for webhook handlers, health checks, or one-off actions.
- **Singleton Resources:** Use `--singleton` for user profiles, application settings, and other single-record resources.

### References

- Laravel Artisan: make:controller — https://laravel.com/docs/12.x/artisan#make-controller
- Laravel Controllers: Resource Controllers — https://laravel.com/docs/12.x/controllers#resource-controllers

---

## 3. Dependency Injection (DI)

### Definitions

**Core Definition:** Dependency Injection in controllers is the practice of having Laravel's service container automatically resolve and provide dependencies—such as the `Request` object, repositories, or SDK clients—either through the constructor or action method parameters.

**Technical Definition:** Laravel's IoC container resolves all controllers. When a controller is instantiated, the container reads the constructor's type-hints, resolves each dependency recursively, and passes them to the constructor. For method injection, the `RouteDependencyResolverTrait` inspects the action method's signature, resolves type-hinted dependencies from the container, and passes route parameters as positional arguments after resolved dependencies.

**Beginner-Friendly Explanation:** Instead of manually creating objects inside your controller (like `new UserRepository()`), you just type-hint what you need in the constructor or method, and Laravel creates it for you and hands it over. This makes your code easier to test because you can swap out dependencies with mock objects.

### Purposes

- To automatically capture the active HTTP request data via the `Request` type-hint.
- To bind broad utilities (repositories, SDK clients, payment gateways) into the controller instance via the constructor.
- To improve testability by allowing dependencies to be mocked or swapped.
- To reduce boilerplate code by eliminating manual instantiation.
- To leverage Laravel's service container for recursive dependency resolution.

### Syntax Rules and Structure

**Complete General Syntax — Constructor Injection:**

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `__construct(UserRepository $users)` | The constructor type-hints the dependency. |
| `$this->users = $users;` | The dependency is stored for use in action methods. |

**Complete General Syntax — Method Injection:**

```php
use Illuminate\Http\Request;

public function store(Request $request)
{
    $name = $request->input('name');
    // ...
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Request $request` | The `Request` instance is injected automatically. |
| `$request->input('name')` | The request data is accessed via the injected instance. |

**Complete General Syntax — Method Injection with Route Parameters:**

```php
public function update(Request $request, string $id)
{
    // $request is injected, $id is the route parameter
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Request $request` | Injected dependency (comes first). |
| `string $id` | Route parameter (comes after dependencies). |

**Syntax Rules:**

- Constructor injection resolves dependencies when the controller is instantiated.
- Method injection resolves dependencies per action invocation.
- Route parameters must be listed **after** injected dependencies in the method signature.
- The `Request` instance is the most commonly method-injected dependency.
- Any class resolvable by the container can be type-hinted.

**Constraints and Limitations:**

- **Constructor limitation:** Constructor-injected dependencies are resolved when the controller is instantiated, which may be inefficient if only some actions use them.
- **Method injection scope:** Method injection works for controller actions and route closures, but not for arbitrary methods.
- **Route parameter order:** If route parameters are listed before dependencies, the container may attempt to resolve them as dependencies and fail.
- **Singleton behaviour:** Constructor-injected dependencies are shared across all actions of the controller instance (per request).

### Multiple Annotated Complete Code Examples

**Example 1: Constructor Injection with a Repository**

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * The user repository instance.
     */
    protected $users;

    /**
     * Create a new controller instance.
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * Show a list of all users.
     */
    public function index(): View
    {
        $users = $this->users->all();
        return view('users.index', compact('users'));
    }
}
```

**Setup:** Create a `UserRepository` class with an `all()` method that returns all users.

**Expected Output:** `GET /users` returns the `users.index` view with all users from the repository.

**Why:** The constructor type-hints `UserRepository`. Laravel's container resolves it automatically and injects it. The repository is stored as a property and used in the `index` method.

---

**Example 2: Method Injection with Request and Route Parameter**

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Update the specified post.
     */
    public function update(Request $request, Post $post)
    {
        // $request is injected, $post is resolved via route model binding
        $post->update($request->validated());
        return redirect()->route('posts.show', $post);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

Route::put('/posts/{post}', [PostController::class, 'update']);
```

**Expected Output:** `PUT /posts/5` updates post 5 with the request data and redirects to the post's show page.

**Why:** The `Request` dependency is injected first, then the `{post}` route parameter is resolved via route model binding. The order (dependency first, parameter second) is required.

---

**Example 3: Method Injection with a Custom Service**

```php
<?php
// app/Http/Controllers/PaymentController.php

namespace App\Http\Controllers;

use App\Services\PaymentGateway;
use Illuminate\Http\Request;

class PaymentController extends Controller
{
    /**
     * Process a payment.
     */
    public function process(Request $request, PaymentGateway $gateway)
    {
        $result = $gateway->charge(
            $request->input('amount'),
            $request->input('token')
        );

        return response()->json(['status' => $result]);
    }
}
```

**Expected Output:** `POST /payment` processes the payment through the injected `PaymentGateway` service and returns the result as JSON.

**Why:** The `PaymentGateway` service is type-hinted on the method. Laravel's container resolves it and injects it alongside the `Request` instance. The gateway is used only in this action, making method injection more appropriate than constructor injection.

### Real-World Cases

- **Repository Pattern:** Inject repositories into controllers for data access abstraction.
- **Payment Gateways:** Inject Stripe, PayPal, or custom payment service clients.
- **SDK Clients:** Inject AWS, Google Cloud, or third-party API clients.
- **Request Validation:** Inject `Request` to access input, files, and headers.
- **Service Layer:** Inject domain services that encapsulate complex business logic.

### References

- Laravel Controllers: Dependency Injection — https://laravel.com/docs/12.x/controllers#dependency-injection-and-controllers
- Laravel Service Container — https://laravel.com/docs/12.x/container
- Laravel HTTP Requests — https://laravel.com/docs/12.x/requests

---

## 4. Unified Response Objects

### Definitions

**Core Definition:** Unified response objects are the various return types a controller action can produce—views, JSON, redirects, files, and raw responses—all of which implement or are convertible to `Illuminate\Http\Response`, providing a consistent interface for the HTTP layer.

**Technical Definition:** Laravel's `ResponseFactory` (accessed via the `response()` helper) creates `Illuminate\Http\Response`, `Illuminate\Http\JsonResponse`, and `Illuminate\Http\RedirectResponse` instances. The `view()` helper returns an `Illuminate\View\View` which is rendered and wrapped in a `Response` by the kernel. The `redirect()` helper returns a `RedirectResponse`. Each response type carries status codes, headers, and content, and all pass through the middleware stack before being sent to the client.

**Beginner-Friendly Explanation:** A controller can respond in many different ways—showing a web page, returning data as JSON, or redirecting the user to another page. Laravel wraps all of these in a consistent "response" object, so the rest of the framework knows how to handle them.

### Purposes

- To render HTML templates via the `view()` helper or `response()->view()`.
- To force structural API payloads via `response()->json()`.
- To construct user redirection workflows via `redirect()->route()`.
- To return raw strings, arrays, or `Response` instances with custom status codes and headers.
- To download or display files via `response()->download()` and `response()->file()`.

### Syntax Rules and Structure

**Complete General Syntax — View Response:**

```php
return view('user.profile', ['user' => $user]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `view()` | Global helper for rendering Blade templates. |
| `'user.profile'` | The view path (dot notation). |
| `['user' => $user]` | Data passed to the view. |

**Complete General Syntax — JSON Response:**

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `response()->json()` | Creates a `JsonResponse` with `Content-Type: application/json`. |
| `[...]` | The data array to be JSON-encoded. |
| Optional status | `response()->json($data, 201)` sets the HTTP status code. |

**Complete General Syntax — Redirect Response:**

```php
return redirect()->route('posts.show', ['post' => $post->id]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `redirect()->route()` | Creates a `RedirectResponse` to a named route. |
| `'posts.show'` | The route name. |
| `['post' => $post->id]` | Route parameters. |

**Complete General Syntax — Redirect with Flash Data:**

```php
return redirect()->route('dashboard')->with('status', 'Profile updated!');
```

**Complete General Syntax — Custom View Response with Headers:**

```php
return response()->view('hello', $data, 200)
    ->header('Content-Type', 'text/html');
```

**Complete General Syntax — Download Response:**

```php
return response()->download($pathToFile, $name, $headers);
```

**Syntax Rules:**

- The `view()` helper returns a `View` instance; the kernel converts it to a `Response`.
- The `response()->json()` method sets `Content-Type: application/json` automatically.
- The `redirect()->route()` method requires the route name and any required parameters.
- The `->with()` method flashes data to the session for the next request.
- Raw strings and arrays returned from controllers are automatically converted to `Response` instances.
- The `response()` helper without arguments returns a `ResponseFactory` instance.

**Constraints and Limitations:**

- **View existence:** The view must exist, or a `InvalidArgumentException` is thrown.
- **JSON encoding:** Complex objects may fail to encode; use `JsonSerializable` or arrays.
- **Redirect loops:** Careless redirect logic can cause infinite redirect loops.
- **Status codes:** By default, views return 200, JSON returns 200, and redirects return 302. Custom status codes must be specified explicitly.

### Multiple Annotated Complete Code Examples

**Example 1: View Response with Data**

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    /**
     * Show the user's profile.
     */
    public function show(string $id)
    {
        $user = User::findOrFail($id);

        // Render the 'user.profile' Blade view with user data
        return view('user.profile', ['user' => $user]);
    }
}
```

**Setup:** Create `resources/views/user/profile.blade.php` with `{{ $user->name }}`.

**Expected Output:** `GET /user/1` renders the profile view showing the user's name.

**Why:** The `view()` helper renders the Blade template, passing the `$user` variable. The kernel wraps the rendered HTML in a `Response` with a 200 status code.

---

**Example 2: JSON Response for an API**

```php
<?php
// app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Post;

class PostController extends Controller
{
    /**
     * Return a list of all posts as JSON.
     */
    public function index()
    {
        $posts = Post::all();

        // Return JSON with a 200 status code
        return response()->json([
            'data' => $posts,
            'count' => $posts->count(),
        ]);
    }
}
```

**Expected Output:** `GET /api/posts` returns:
```json
{
    "data": [...],
    "count": 42
}
```

**Why:** The `response()->json()` method encodes the array as JSON and sets the `Content-Type: application/json` header automatically.

---

**Example 3: Redirect with Flash Message**

```php
<?php
// app/Http/Controllers/ProfileController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class ProfileController extends Controller
{
    /**
     * Update the authenticated user's profile.
     */
    public function update(Request $request)
    {
        $user = Auth::user();
        $user->update($request->validated());

        // Redirect to the profile show page with a flash message
        return redirect()
            ->route('profile.show')
            ->with('status', 'Profile updated successfully!');
    }
}
```

```blade
{{-- resources/views/profile/show.blade.php --}}
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

**Expected Output:** After updating, the user is redirected to `/profile`, where the flash message `Profile updated successfully!` is displayed.

**Why:** The `redirect()->route()` method creates a redirect response. The `->with()` method flashes data to the session, which is available on the next request and displayed in the view.

### Real-World Cases

- **Web Pages:** `view()` renders HTML pages for browser users.
- **API Endpoints:** `response()->json()` returns structured data for mobile and SPA clients.
- **Form Submissions:** `redirect()->route()` sends users back to a list or detail page after form submission.
- **File Downloads:** `response()->download()` serves files (invoices, reports) to users.
- **Flash Messages:** `->with()` displays success or error messages after redirects.

### References

- Laravel Responses Documentation — https://laravel.com/docs/12.x/responses
- Laravel Views Documentation — https://laravel.com/docs/12.x/views
- Laravel Redirects Documentation — https://laravel.com/docs/12.x/responses#redirects

---

## References

- Laravel Controllers Documentation — https://laravel.com/docs/12.x/controllers
- Laravel Routing Documentation — https://laravel.com/docs/12.x/routing
- Laravel HTTP Requests Documentation — https://laravel.com/docs/12.x/requests
- Laravel HTTP Responses Documentation — https://laravel.com/docs/12.x/responses
- Laravel Service Container Documentation — https://laravel.com/docs/12.x/container
- Laravel Request Lifecycle Documentation — https://laravel.com/docs/12.x/lifecycle
- Laravel Artisan Console Documentation — https://laravel.com/docs/12.x/artisan
- Laravel Views Documentation — https://laravel.com/docs/12.x/views
- Laravel Middleware Documentation — https://laravel.com/docs/12.x/middleware
- Laravel News: Laravel VS Code Extension Artisan Commands — https://laravel-news.com/laravel-vs-code-extension-artisan-commands
- Laravel 12.x Controllers (BookStack Mirror) — https://www.bookstack.cn/read/laravel-12.x-en/a48e3de1eb4c90a6.md
- Laravel 13.x Controllers (Laravel.net.cn) — https://laravel.net.cn/docs/13.x/controllers
- Understanding the Laravel Request Lifecycle (Laravel 11) — https://tsecurity.de/de/2525601/IT+Programmierung/Understanding+the+Laravel+Request+Lifecycle+(Laravel+11)/
- Laravel Request Lifecycle Reference (fusengine/agents) — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-architecture/references/lifecycle.md