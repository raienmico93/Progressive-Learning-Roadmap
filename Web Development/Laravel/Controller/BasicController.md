# Laravel Controllers: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A Laravel controller is a PHP class that groups related request-handling logic, allowing you to organize the behaviour of your application's routes into dedicated, reusable methods rather than defining everything as closures in route files.

**Technical Definition**

Controllers are classes stored in the `app/Http/Controllers` directory. They are resolved by Laravel's service container, which enables constructor and method dependency injection. A controller may contain any number of public methods that respond to HTTP requests. When a route matches, the router resolves the controller via the service container, calls the specified method, and passes route parameters as arguments.

**Beginner-Friendly Explanation**

A controller is a class that holds the code for your routes. Instead of writing all your logic in route files, you organize related logic into methods on a controller. For example, a `UserController` might have methods for showing, creating, updating, and deleting users. When someone visits a URL, Laravel finds the matching controller method and runs it.

---

### Key Characteristics

- **Class-Based Organization**: Related request handling logic is grouped into a single class, improving code organization and maintainability.
- **Service Container Resolution**: Controllers are resolved by Laravel's service container, enabling automatic dependency injection in constructors and methods.
- **No Base Class Required**: Controllers are not required to extend a base class, though extending a base controller is convenient for sharing common methods.
- **Multiple Action Types**: Controllers support basic methods, single-action (`__invoke`) controllers, and resource controllers with pre-defined CRUD methods.
- **Middleware Integration**: Middleware can be assigned to controller routes, the controller class, or individual methods via the `HasMiddleware` interface or PHP attributes.
- **Dependency Injection**: Constructor and method injection allow controllers to declare their dependencies, which are automatically resolved.
- **Testing Support**: Controllers can be tested using Laravel's HTTP testing API, which simulates requests and provides assertions on responses.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Basic knowledge of PHP classes, methods, and namespaces**.
- **Familiarity with Laravel routing** (defining routes, HTTP verbs, route parameters).
- **Understanding of the service container** is helpful for dependency injection.

---

### Related Programming Areas

- **Routing**: Controllers are the action handlers for routes. Route definitions point to controller methods.
- **Middleware**: Middleware filters requests before they reach controller methods, handling authentication, authorization, rate limiting, and more.
- **Eloquent ORM**: Controllers often interact with Eloquent models to retrieve and persist data.
- **Blade Views**: Controllers return views rendered by the Blade templating engine.
- **Form Requests**: Controller validation is commonly delegated to form request classes.
- **Service Classes**: Business logic is often extracted from controllers into dedicated service classes to keep controllers thin.

---

## Core Concepts

---

### 1. Controllers

#### Definitions

**Core Definition**

A controller is a PHP class that groups related request-handling logic into a single class, with each public method corresponding to a route action.

**Technical Definition**

Controllers extend `App\Http\Controllers\Controller` (optional) and are stored in `app/Http/Controllers`. They are resolved by the service container, enabling dependency injection. Routes reference controller methods using the `[ControllerClass::class, 'method']` syntax. Controllers can have any number of public methods, and route parameters are passed to the method as arguments.

**Beginner-Friendly Explanation**

A controller is like a folder for your route logic. Instead of having all your code scattered in route files, you put related code together in a class. For example, everything related to users goes in `UserController`.

#### Purposes

- To organize request-handling logic into dedicated, reusable classes.
- To separate routing configuration from request-handling logic.
- To enable dependency injection for cleaner, more testable code.
- To provide a natural place for grouping related actions (e.g., all user-related operations).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

```php
// Route definition
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `namespace App\Http\Controllers;` | Default namespace for controllers. |
| `class UserController extends Controller` | Controller class, optionally extending a base controller. |
| `public function show(string $id): View` | Public method responding to a route. |
| `[UserController::class, 'show']` | Route action syntax referencing the controller method. |

**Syntax Rules**

- Controllers are stored in `app/Http/Controllers`.
- Public methods respond to HTTP requests.
- The `Controller` base class is optional but convenient.
- Routes reference controller methods using the `[Class::class, 'method']` array syntax.
- Route parameters are passed to the controller method as arguments.

**Constraints and Limitations**

- Controllers should remain thin; complex business logic should be delegated to service classes.
- Controllers are resolved by the service container on each request.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Controller**

**Controller**

```php
<?php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    public function index()
    {
        return view('users.index', ['users' => User::all()]);
    }

    public function show(string $id)
    {
        return view('users.show', ['user' => User::findOrFail($id)]);
    }
}
```

**Route**

```php
// routes/web.php
use App\Http\Controllers\UserController;

Route::get('/users', [UserController::class, 'index']);
Route::get('/users/{id}', [UserController::class, 'show']);
```

**Expected Output**

```
// GET /users → renders users index view with all users
// GET /users/1 → renders users show view for user 1
```

**Why This Output Occurs**

The router resolves `UserController` from the service container, calls the appropriate method, and passes the route parameter `{id}` as an argument. The method returns a view with data.

---

#### Real-World Cases with Explanation

**Case 1: Blog Application**

A blog uses `PostController` for listing posts, showing individual posts, creating, editing, and deleting. Each action is a method on the controller, keeping the route file clean.

**Explanation**: Controllers centralize related logic, making the application easier to navigate and maintain.

---

### 2. Controller Creation

#### Definitions

**Core Definition**

Controller creation is the process of generating a new controller class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:controller` command generates a PHP class file in `app/Http/Controllers`. The command supports options for creating resource controllers (`--resource`), invokable controllers (`--invokable`), API resource controllers (`--api`), and generating associated form requests and models.

**Beginner-Friendly Explanation**

Creating a controller is easy—just run a command in your terminal, and Laravel makes the file for you.

#### Purposes

- To automate the creation of controller boilerplate.
- To ensure controllers are placed in the correct namespace and directory.
- To support various controller types (basic, resource, invokable) with appropriate method stubs.
- To reduce manual file creation errors.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:controller UserController
php artisan make:controller PhotoController --resource
php artisan make:controller ProvisionServer --invokable
php artisan make:controller PhotoController --api
php artisan make:controller PhotoController --resource --model=Photo
php artisan make:controller PhotoController --resource --requests
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:controller UserController` | Creates a basic controller. |
| `--resource` | Generates a resource controller with CRUD method stubs. |
| `--invokable` | Generates a single-action controller with `__invoke`. |
| `--api` | Generates an API resource controller (excludes `create`, `edit`). |
| `--model=Photo` | Type-hints the model in resource methods. |
| `--requests` | Generates form request classes for store/update. |

**Syntax Rules**

- The controller name should be PascalCase.
- Options can be combined.
- Generated files must be edited to add logic.

**Constraints and Limitations**

- The command does not register routes automatically.
- Generated stubs may need customization.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Creating a Resource Controller**

**Command**

```bash
php artisan make:controller PhotoController --resource
```

**Generated File**

```php
<?php
// app/Http/Controllers/PhotoController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PhotoController extends Controller
{
    public function index() { }
    public function create() { }
    public function store(Request $request) { }
    public function show(string $id) { }
    public function edit(string $id) { }
    public function update(Request $request, string $id) { }
    public function destroy(string $id) { }
}
```

**Expected Output (Terminal)**

```
INFO  Controller [app/Http/Controllers/PhotoController.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the controller with all seven resource method stubs.

---

#### Real-World Cases with Explanation

**Case 1: Rapid API Development**

A developer runs `php artisan make:controller Api/UserController --api --model=User` to generate an API controller with model type-hints.

**Explanation**: Artisan accelerates development by generating appropriate stubs.

---

### 3. `artisan make:controller`

#### Definitions

**Core Definition**

`artisan make:controller` is the Artisan command used to generate new controller classes.

**Technical Definition**

The command is implemented by `Illuminate\Foundation\Console\ControllerMakeCommand`. It generates a PHP class file and optionally invokes other make commands for form requests. The `--resource` option generates method stubs for all seven RESTful actions; `--api` excludes `create` and `edit`; `--invokable` generates only the `__invoke` method.

**Beginner-Friendly Explanation**

`artisan make:controller` is the command you type in your terminal to create a new controller file.

#### Purposes

- To generate controller classes with minimal effort.
- To provide consistent structure for controller files.
- To combine controller generation with related class generation (form requests, models).
- To enforce Laravel's naming and directory conventions.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:controller <Name> [options]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `make:controller` | The Artisan command. |
| `<Name>` | The controller class name (PascalCase). |
| `[options]` | Flags for generating related classes. |

**Syntax Rules**

- The controller name is required.
- Options are prefixed with `--`.
- Generated files are placed in `app/Http/Controllers`.

**Constraints and Limitations**

- Routes are not registered automatically.
- Generated stubs must be filled in.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Invokable Controller**

**Command**

```bash
php artisan make:controller ProvisionServer --invokable
```

**Generated File**

```php
<?php
// app/Http/Controllers/ProvisionServer.php
namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    public function __invoke()
    {
        // ...
    }
}
```

**Expected Output (Terminal)**

```
INFO  Controller [app/Http/Controllers/ProvisionServer.php] created successfully.
```

**Why This Output Occurs**

The `--invokable` flag generates a controller with only the `__invoke` method, suitable for single-action controllers.

---

#### Real-World Cases with Explanation

**Case 1: Single-Action Controllers for Complex Actions**

A payment processing action is complex enough to warrant its own controller. Running `php artisan make:controller ProcessPaymentController --invokable` generates the appropriate stub.

**Explanation**: Invokable controllers keep focused logic isolated.

---

### 4. Single-Action Controllers

#### Definitions

**Core Definition**

A single-action controller is a controller class that handles a single action, defined by the `__invoke` method.

**Technical Definition**

When a controller has only one action, you may define a single `__invoke` method. The router resolves the controller and calls `__invoke` without needing to specify a method name. Invokable controllers are generated with the `--invokable` option of `make:controller`. The `__invoke` method is a PHP magic method that allows the object to be called like a function.

**Beginner-Friendly Explanation**

Sometimes you have a controller that does just one thing. Instead of giving that action a name, you can use a special `__invoke` method that Laravel calls automatically.

#### Purposes

- To dedicate an entire controller to a single, complex action.
- To simplify route definitions by omitting the method name.
- To isolate focused logic for better testability and clarity.
- To reduce the number of methods in large controllers.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    public function __invoke()
    {
        // ...
    }
}
```

```php
// Route definition
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `__invoke()` | The magic method called when the controller is invoked. |
| `ProvisionServer::class` | Route reference without a method name. |

**Syntax Rules**

- The controller must define the `__invoke` method.
- Routes reference the controller class directly, without a method name.
- The `--invokable` option generates the stub.

**Constraints and Limitations**

- An invokable controller can only handle one action.
- For multiple actions, use a standard controller.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Invokable Controller**

**Controller**

```php
<?php
// app/Http/Controllers/ShowProfile.php
namespace App\Http\Controllers;

use App\Models\User;

class ShowProfile extends Controller
{
    public function __invoke(string $id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

**Route**

```php
Route::get('/user/{id}', ShowProfile::class);
```

**Expected Output**

```
// GET /user/1 → renders user profile for user 1
```

**Why This Output Occurs**

The router resolves `ShowProfile` and calls `__invoke` with the route parameter.

---

#### Real-World Cases with Explanation

**Case 1: Webhook Handler**

A webhook endpoint receives POST requests from a third-party service. An invokable controller handles the entire webhook processing logic.

**Explanation**: Invokable controllers are perfect for single-purpose endpoints.

---

### 5. Resource Controllers

#### Definitions

**Core Definition**

A resource controller is a controller that handles the standard CRUD operations (create, read, update, delete) for a resource, with methods automatically mapped to RESTful routes.

**Technical Definition**

Resource controllers are generated with the `--resource` option. The controller contains stubs for `index`, `create`, `store`, `show`, `edit`, `update`, and `destroy`. The `Route::resource()` method registers all seven routes in a single line. The `--api` option excludes `create` and `edit` (HTML form routes). The `--model` option type-hints the model in resource methods.

**Beginner-Friendly Explanation**

A resource controller is a controller that already has all the standard methods for managing a resource, like listing, showing, creating, updating, and deleting.

#### Purposes

- To reduce boilerplate when defining CRUD routes.
- To enforce RESTful conventions across resources.
- To automatically generate route names and controller method mappings.
- To support nested and shallow resource routing.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Controller generation
php artisan make:controller PhotoController --resource

// Route registration
Route::resource('photos', PhotoController::class);

// API resource (excludes create, edit)
Route::apiResource('photos', PhotoController::class);
```

**Resource Route Table**

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

- The resource name should be plural.
- The controller must implement the corresponding methods.
- `apiResource` excludes `create` and `edit`.
- `only()` and `except()` can limit generated routes.

**Constraints and Limitations**

- Resource routes cannot be cached if they use Closure-based custom routes.
- Nested resources with more than two levels are discouraged.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Resource Controller**

**Controller**

```php
<?php
// app/Http/Controllers/PhotoController.php
namespace App\Http\Controllers;

use App\Models\Photo;
use Illuminate\Http\Request;

class PhotoController extends Controller
{
    public function index()
    {
        return view('photos.index', ['photos' => Photo::all()]);
    }

    public function show(string $id)
    {
        return view('photos.show', ['photo' => Photo::findOrFail($id)]);
    }

    public function store(Request $request)
    {
        Photo::create($request->validated());
        return redirect()->route('photos.index');
    }
}
```

**Route**

```php
Route::resource('photos', PhotoController::class);
```

**Expected Output**

```
// GET /photos → photo index
// GET /photos/1 → photo show
// POST /photos → store and redirect
```

**Why This Output Occurs**

`Route::resource()` registers all seven routes, mapping them to the controller's methods.

---

**Example 2: Partial Resource Routes**

```php
Route::resource('photos', PhotoController::class)->only(['index', 'show']);
Route::resource('photos', PhotoController::class)->except(['create', 'store', 'update', 'destroy']);
```

**Expected Output**

```
// Only index and show routes are registered for the first.
// All except create, store, update, destroy for the second.
```

**Why This Output Occurs**

`only()` and `except()` filter which resource routes are registered.

---

#### Real-World Cases with Explanation

**Case 1: Blog Post Management**

A blog uses `Route::resource('posts', PostController::class)` to manage posts, with `PostController` handling all CRUD operations.

**Explanation**: Resource controllers drastically reduce route definition boilerplate.

**Case 2: API Endpoints**

An API uses `Route::apiResource('users', UserController::class)` to expose user endpoints without HTML form routes.

**Explanation**: API resource routes are tailored for stateless API design.

---

### 6. Controller Middleware

#### Definitions

**Core Definition**

Controller middleware are filters that run before or after controller actions, used for authentication, authorization, logging, rate limiting, and other cross-cutting concerns.

**Technical Definition**

Middleware can be assigned to controller routes in route files, within the controller class via the `HasMiddleware` interface, or using PHP attributes. The `HasMiddleware` interface requires a static `middleware()` method returning an array of middleware. The `Middleware` attribute can be applied at the class or method level.

**Beginner-Friendly Explanation**

Middleware are like security guards that check requests before they reach your controller methods.

#### Purposes

- To protect controller actions from unauthorized access.
- To apply consistent request filtering across controller methods.
- To perform actions before or after controller methods.
- To organize cross-cutting concerns outside of controllers.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Route-level middleware
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```

```php
// Controller-level middleware (HasMiddleware)
use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController implements HasMiddleware
{
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('log', only: ['index']),
            new Middleware('subscribed', except: ['store']),
        ];
    }
}
```

```php
// Middleware attributes (PHP 8+)
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
#[Middleware('log', only: ['index'])]
class UserController
{
    // ...
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| Route `->middleware()` | Applies middleware to a single route. |
| `HasMiddleware` interface | Applies middleware to all controller actions. |
| `Middleware` attribute | PHP attribute syntax for middleware. |

**Syntax Rules**

- Middleware are executed in the order listed.
- `only` and `except` control which methods the middleware applies to.
- Middleware can be strings (aliases) or class names.

**Constraints and Limitations**

- Middleware order matters for authentication dependencies.
- Middleware must be registered in the application's middleware stack.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: HasMiddleware Implementation**

**Controller**

```php
<?php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController implements HasMiddleware
{
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('admin', only: ['destroy']),
        ];
    }

    public function index() { return 'User index'; }
    public function destroy($id) { return "User {$id} deleted"; }
}
```

**Expected Output**

```
// GET /users → requires auth
// DELETE /users/1 → requires auth + admin
```

**Why This Output Occurs**

The `auth` middleware applies to all methods, while `admin` only applies to `destroy`.

---

**Example 2: Middleware Attributes**

```php
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class UserController
{
    #[Middleware('log')]
    public function index() { }

    #[Middleware('subscribed')]
    public function store() { }
}
```

**Expected Output**

```
// All methods require auth; index also requires log; store also requires subscribed.
```

**Why This Output Occurs**

Class-level middleware applies to all methods, and method-level middleware is merged.

---

#### Real-World Cases with Explanation

**Case 1: Admin Panel Protection**

An admin controller uses the `HasMiddleware` interface to apply `auth` and `admin` middleware to all actions.

**Explanation**: Middleware on the controller ensures consistent protection without repeating it on each route.

---

### 7. Controller Dependency Injection

#### Definitions

**Core Definition**

Controller dependency injection is the process of automatically resolving and injecting a controller's dependencies (services, repositories, etc.) through constructor or method parameters.

**Technical Definition**

Laravel's service container resolves controllers, enabling type-hinted dependencies in the constructor to be automatically injected. Method injection allows dependencies to be type-hinted on individual controller methods, with route parameters listed after other dependencies.

**Beginner-Friendly Explanation**

Dependency injection means you don't have to create objects manually. You just declare what you need, and Laravel gives it to you.

#### Purposes

- To decouple controllers from the concrete implementations of their dependencies.
- To enable testability by allowing dependencies to be mocked.
- To reduce boilerplate by automatically resolving dependencies.
- To support clean, service-oriented architecture.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Constructor injection
class UserController extends Controller
{
    public function __construct(
        protected UserRepository $users,
    ) {}
}
```

```php
// Method injection
public function store(Request $request): RedirectResponse
{
    $name = $request->name;
    // ...
    return redirect('/users');
}
```

**Component Breakdown**

| Injection Type | Description |
|----------------|-------------|
| Constructor | Dependencies resolved when the controller is instantiated. |
| Method | Dependencies resolved when the method is called. |

**Syntax Rules**

- Type-hinted dependencies are automatically resolved.
- Route parameters must be listed after injected dependencies.
- Constructor injection is preferred over method injection for required dependencies.

**Constraints and Limitations**

- Only type-hinted dependencies are resolved; scalar parameters require defaults or explicit binding.
- Method injection is resolved per-call, which may have performance implications.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Constructor Injection**

**Controller**

```php
<?php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    public function __construct(
        protected UserRepository $users,
    ) {}

    public function index()
    {
        return view('users.index', ['users' => $this->users->all()]);
    }
}
```

**Expected Output**

```
// The UserRepository is automatically injected when the controller is instantiated
```

**Why This Output Occurs**

The service container resolves `UserRepository` and passes it to the constructor.

---

**Example 2: Method Injection with Route Parameter**

**Controller**

```php
public function update(Request $request, string $id): RedirectResponse
{
    // $request is injected, $id is the route parameter
    return redirect('/users');
}
```

**Route**

```php
Route::put('/user/{id}', [UserController::class, 'update']);
```

**Expected Output**

```
// The Request instance is injected, and the route parameter {id} is passed as the second argument
```

**Why This Output Occurs**

Laravel resolves the `Request` dependency and passes the route parameter after it.

---

#### Real-World Cases with Explanation

**Case 1: Repository Pattern**

A controller injects a `UserRepository` to interact with the data layer, keeping database logic out of the controller.

**Explanation**: Dependency injection enables clean separation of concerns.

---

### 8. Controller Method Injection

#### Definitions

**Core Definition**

Method injection is the practice of type-hinting dependencies on individual controller methods, allowing Laravel to resolve and inject them when the method is called.

**Technical Definition**

When a controller method is invoked, Laravel's service container resolves any type-hinted dependencies and passes them as arguments. The `Illuminate\Http\Request` instance is the most common method-injected dependency. Route parameters are passed after injected dependencies.

**Beginner-Friendly Explanation**

Method injection lets you ask for dependencies directly in the method where you need them, rather than in the constructor.

#### Purposes

- To inject dependencies only where they are needed.
- To avoid constructor bloat when a dependency is used by only one method.
- To provide access to the current request and other contextual services.
- To support route model binding alongside other dependencies.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function store(Request $request): RedirectResponse
{
    $name = $request->name;
    // ...
    return redirect('/users');
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Request $request` | The injected dependency. |
| `string $id` | Route parameter (listed after dependencies). |

**Syntax Rules**

- Type-hinted parameters are resolved by the container.
- Route parameters must come after injected dependencies.
- Multiple dependencies can be injected.

**Constraints and Limitations**

- Method injection is resolved per-call.
- Scalar parameters cannot be auto-resolved without a default or binding.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Request Injection**

**Controller**

```php
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
    ]);

    User::create($validated);

    return redirect()->route('users.index');
}
```

**Expected Output**

```
// The Request instance is injected, providing access to input and validation
```

**Why This Output Occurs**

Laravel resolves `Request` from the container and passes it to the `store` method.

---

**Example 2: Multiple Dependencies with Route Parameter**

**Controller**

```php
public function update(Request $request, UserService $service, string $id)
{
    $service->update($id, $request->validated());
    return redirect()->route('users.index');
}
```

**Expected Output**

```
// Request and UserService are injected; $id is the route parameter
```

**Why This Output Occurs**

The container resolves both type-hinted dependencies, and the route parameter is passed last.

---

#### Real-World Cases with Explanation

**Case 1: Accessing the Request in a Controller Method**

A controller method injects `Request` to access input data, headers, and session state.

**Explanation**: Method injection is the standard way to access the current request in controllers.

---

### 9. Controller Traits

#### Definitions

**Core Definition**

Controller traits are reusable PHP traits that can be included in controller classes to share common methods and behaviour across multiple controllers.

**Technical Definition**

Traits are a PHP language feature that allows methods to be reused in multiple classes. In Laravel, controller traits are commonly used to share helper methods (e.g., API response formatting, authorization helpers) across controllers. The base `Controller` class in Laravel uses several traits: `AuthorizesRequests`, `DispatchesJobs`, and `ValidatesRequests`.

**Beginner-Friendly Explanation**

A controller trait is a collection of methods that you can "mix into" any controller. It is like a shared toolbox that multiple controllers can use.

#### Purposes

- To share common methods across multiple controllers without inheritance.
- To keep controllers thin by extracting reusable logic.
- To provide API response helpers, authorization methods, or validation shortcuts.
- To avoid duplication when multiple controllers need the same behaviour.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Http/Controllers/Concerns/ApiResponder.php
namespace App\Http\Controllers\Concerns;

trait ApiResponder
{
    protected function respondSuccess($data, int $status = 200)
    {
        return response()->json(['data' => $data], $status);
    }

    protected function respondError(string $message, int $status = 400)
    {
        return response()->json(['error' => $message], $status);
    }
}
```

```php
// Usage in a controller
use App\Http\Controllers\Concerns\ApiResponder;

class UserController extends Controller
{
    use ApiResponder;

    public function index()
    {
        return $this->respondSuccess(User::all());
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `trait ApiResponder` | The trait definition. |
| `use ApiResponder;` | Includes the trait in a controller. |
| `$this->respondSuccess(...)` | Calls a trait method. |

**Syntax Rules**

- Traits are defined with the `trait` keyword.
- The `use` statement includes the trait in a class.
- Trait methods are accessed as if they were defined on the class.
- Traits can be composed of multiple other traits.

**Constraints and Limitations**

- Traits cannot be instantiated independently.
- Method name conflicts between traits and the class must be resolved.
- Traits are not a substitute for proper service classes for complex logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: API Response Trait**

**Trait**

```php
<?php
// app/Http/Controllers/Concerns/ApiResponder.php
namespace App\Http\Controllers\Concerns;

trait ApiResponder
{
    protected function success($data, int $status = 200)
    {
        return response()->json(['data' => $data], $status);
    }

    protected function error(string $message, int $status = 400)
    {
        return response()->json(['error' => $message], $status);
    }
}
```

**Controller**

```php
<?php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Http\Controllers\Concerns\ApiResponder;
use App\Models\User;

class UserController extends Controller
{
    use ApiResponder;

    public function index()
    {
        return $this->success(User::all());
    }

    public function show(string $id)
    {
        $user = User::find($id);
        if (! $user) {
            return $this->error('User not found', 404);
        }
        return $this->success($user);
    }
}
```

**Expected Output**

```
// GET /users → {"data":[...]}
// GET /users/999 → {"error":"User not found"} with 404 status
```

**Why This Output Occurs**

The `ApiResponder` trait provides the `success` and `error` methods, which are used by the controller to format JSON responses.

---

#### Real-World Cases with Explanation

**Case 1: Shared API Response Formatting**

Multiple API controllers use a trait to ensure consistent JSON response formatting (e.g., `{"data": ...}` for success, `{"error": ...}` for failures).

**Explanation**: Traits promote consistency across controllers without forcing inheritance.

---

### 10. Controller Testing

#### Definitions

**Core Definition**

Controller testing is the practice of writing automated tests that verify controller methods return the expected responses for given requests.

**Technical Definition**

Laravel provides a fluent HTTP testing API. Tests make simulated requests to the application using methods like `get()`, `post()`, `put()`, `patch()`, and `delete()`, which return a `TestResponse` instance with assertions such as `assertStatus()`, `assertSee()`, `assertJson()`, and more. CSRF middleware is automatically disabled during tests.

**Beginner-Friendly Explanation**

Testing a controller means writing code that sends a fake request and checks if the response is what you expect.

#### Purposes

- To ensure controller actions return the correct responses.
- To catch regressions when controller code changes.
- To verify that validation, authorization, and middleware behave correctly.
- To document expected controller behaviour through executable tests.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// tests/Feature/UserControllerTest.php
namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    public function test_index_returns_users(): void
    {
        User::factory()->count(3)->create();

        $response = $this->get('/users');

        $response->assertStatus(200);
        $response->assertSee('User 1');
    }

    public function test_store_creates_user(): void
    {
        $response = $this->post('/users', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ]);

        $response->assertRedirect('/users');
        $this->assertDatabaseHas('users', ['email' => 'john@example.com']);
    }
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `$this->get()` | Makes a GET request. |
| `$this->post()` | Makes a POST request. |
| `$response->assertStatus()` | Asserts the HTTP status code. |
| `$response->assertSee()` | Asserts a string appears in the response. |
| `$response->assertRedirect()` | Asserts a redirect to a URI. |

**Syntax Rules**

- Tests extend `Tests\TestCase`.
- The `get`, `post`, etc. methods simulate requests internally.
- Assertions are chainable.
- Database transactions can be used to isolate tests.

**Constraints and Limitations**

- Each test should make only one request.
- CSRF middleware is disabled in tests.
- Authentication can be simulated with `actingAs()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Testing a Controller Index Method**

**Test**

```php
public function test_index_displays_users(): void
{
    User::factory()->count(5)->create();

    $response = $this->get('/users');

    $response->assertStatus(200);
    $response->assertViewHas('users');
}
```

**Expected Output (Test Result)**

```
PASS  Tests\Feature\UserControllerTest
✓ index displays users
```

**Why This Output Occurs**

The test creates 5 users, makes a GET request to `/users`, and asserts the response status and view data.

---

**Example 2: Testing Validation**

**Test**

```php
public function test_store_validates_required_fields(): void
{
    $response = $this->post('/users', []);

    $response->assertSessionHasErrors(['name', 'email']);
}
```

**Expected Output (Test Result)**

```
PASS  Tests\Feature\UserControllerTest
✓ store validates required fields
```

**Why This Output Occurs**

The POST request with empty data triggers validation errors, which are asserted.

---

#### Real-World Cases with Explanation

**Case 1: Feature Testing a REST API**

A team writes feature tests for each API endpoint, verifying status codes, JSON structure, and database state.

**Explanation**: Controller testing ensures API contracts are maintained.

---

### 11. Controller Best Practices

#### Definitions

**Core Definition**

Controller best practices are the recommended guidelines for designing, implementing, and maintaining controllers to maximize maintainability, testability, and clarity.

**Technical Definition**

Best practices encompass keeping controllers thin, delegating business logic to services, using form requests for validation, leveraging resource controllers, applying middleware appropriately, and writing comprehensive tests.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their controllers clean and easy to work with.

#### Key Best Practices

1. **Keep Controllers Thin**: Controllers should orchestrate requests, not contain business logic. Delegate complex operations to service classes or model methods.

2. **Use Form Requests for Validation**: Always use form request classes for validation instead of validating in the controller.

3. **Use Resource Controllers for CRUD**: Leverage `Route::resource()` and `--resource` generation for standard CRUD operations.

4. **Prefer Single-Action Controllers for Complex Actions**: If a controller action is particularly complex, dedicate an entire controller to it using `__invoke`.

5. **Inject Dependencies**: Use constructor and method injection instead of instantiating dependencies directly.

6. **Apply Middleware Properly**: Use the `HasMiddleware` interface or route-level middleware, avoiding unnecessary middleware on all actions.

7. **Return Consistent Responses**: Use API resources or response macros to ensure consistent response formats.

8. **Use Route Model Binding**: Leverage implicit or explicit route model binding to automatically resolve models.

9. **Keep Methods Short**: If a method is growing too long, extract logic to a service class.

10. **Write Tests**: Write feature tests for each controller action to ensure correctness and prevent regressions.

11. **Organize with Namespaces**: Use subdirectories and namespaces (e.g., `Admin`, `Api`) to organize controllers logically.

12. **Document Controller Actions**: Add PHPDoc comments for complex methods, especially those with non-obvious behaviour.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Thin Controller with Service Delegation**

**Controller**

```php
<?php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Http\Requests\StoreUserRequest;
use App\Services\UserService;

class UserController extends Controller
{
    public function __construct(
        protected UserService $userService,
    ) {}

    public function store(StoreUserRequest $request)
    {
        $user = $this->userService->createUser($request->validated());

        return redirect()
            ->route('users.show', $user)
            ->with('success', 'User created successfully.');
    }
}
```

**Service**

```php
<?php
// app/Services/UserService.php
namespace App\Services;

use App\Models\User;

class UserService
{
    public function createUser(array $data): User
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
        ]);
    }
}
```

**Expected Output**

```
// POST /users with valid data → user created, redirect to show page
```

**Why This Output Occurs**

The controller delegates user creation to the `UserService`, keeping the controller thin. Validation is handled by the `StoreUserRequest`.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise SaaS Application**

A large SaaS application follows best practices: thin controllers, service classes for business logic, form requests for validation, resource controllers for CRUD, and comprehensive feature tests.

**Explanation**: Following best practices ensures the codebase remains maintainable as it grows.

---

## References

- Controllers – Laravel 13.x Documentation — https://laravel.com/docs/13.x/controllers
- Controllers – Laravel 12.x Documentation — https://laravel.com/docs/12.x/controllers
- Controllers – Laravel 11.x Documentation — https://laravel.com/docs/11.x/controllers
- Controllers – Laravel 10.x Documentation — https://laravel.com/docs/10.x/controllers
- HTTP Tests – Laravel 13.x Documentation — https://laravel.com/docs/13.x/http-tests
- Service Container – Laravel 13.x Documentation — https://laravel.com/docs/13.x/container
- Single Action Controllers – Laravel Daily — https://laraveldaily.com/post/single-action-controllers
- Controller Best Practices – ButterCMS — https://buttercms.com/blog/laravel-best-practices/
- Laravel Controller Testing – Zizaco/TestCases-Laravel — https://packagist.org/packages/zizaco/testcases-laravel

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific changes include: the `HasMiddleware` interface was introduced in Laravel 11 for controller middleware; PHP attributes for middleware are available in Laravel 11+; the `--api` option for `make:controller` generates API resource controllers. Developers should always consult the documentation for their specific Laravel version.