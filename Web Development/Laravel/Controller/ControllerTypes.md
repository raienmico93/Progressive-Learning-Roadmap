# Laravel Controller Specializations & Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel controller specializations are distinct architectural patterns for organising controller classes, each tailored to a specific request-handling scenario: standard controllers for custom non-RESTful logic, resource/API controllers for conventional CRUD operations, invokable controllers for single-purpose actions, and nested resource controllers for hierarchically related data.

**Technical Definition:** Laravel provides four primary controller specializations, each generated via dedicated Artisan flags (`make:controller`, `--resource`, `--api`, `--invokable`) and routed through distinct registration patterns (`[Controller::class, 'method']`, `Route::resource()`, `Route::apiResource()`, `Controller::class`). The controller type determines the generated method stubs, the routing registration method, and the structural conventions for request handling. Standard controllers extend `App\Http\Controllers\Controller`; resource and API controllers additionally implement conventional CRUD methods; invokable controllers implement the PHP `__invoke()` magic method; nested resource controllers extend resource controllers with dot-notation routing for parent-child relationships.

**Beginner-Friendly Explanation:** Not all controllers are the same. Some handle custom, one-off logic (standard controllers). Some follow a predictable pattern for managing things like posts or users (resource and API controllers). Some do exactly one thing (invokable controllers). And some manage data that belongs to other data—like comments on a blog post (nested resource controllers). Laravel gives you a different "template" for each situation.

### Key Characteristics

- **Purpose-Driven Selection:** Each specialization exists because a particular request-handling pattern is common enough to warrant dedicated tooling.
- **Artisan Scaffolding:** Each type can be generated via a specific `make:controller` flag, pre-populating the appropriate method stubs.
- **Routing Synergy:** The controller type and the routing registration method are designed to work together (e.g., `--resource` pairs with `Route::resource()`).
- **Middleware Compatibility:** All controller types support middleware through the `HasMiddleware` interface (Laravel 11+) or constructor-based middleware (Laravel 10 and earlier).
- **Dependency Injection:** All controller types leverage Laravel's service container for constructor and method injection.
- **Response Versatility:** All controller types can return views, JSON, redirects, and other response types.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (controller specializations are stable across these versions; the `HasMiddleware` interface is available from Laravel 10+; invokable controllers from Laravel 5.6.28+; singleton resources from Laravel 9.42+).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `app/Http/Controllers` directory.
- Basic understanding of routing, HTTP verbs, and Eloquent models.

### Related Programming Areas

- **RESTful API Design:** Resource and API controllers are the primary implementation of REST in Laravel.
- **Routing:** Each controller type pairs with a specific route registration method.
- **Service Container:** All controller types resolve dependencies through Laravel's IoC container.
- **Middleware Pipeline:** Controllers integrate with middleware for authentication, rate limiting, and request filtering.
- **Eloquent ORM:** Resource and nested resource controllers leverage route model binding.

### Core Concepts / Features

1. Standard Controllers
2. Resource & API Controllers
3. Invokable (Single-Action) Controllers
4. Nested Resource Controllers

---

## 1. Standard Controllers

### Definitions

**Core Definition:** A standard controller is a flexible PHP class that houses custom, non-RESTful action methods, providing a general-purpose container for request-handling logic that does not fit the conventional CRUD pattern.

**Technical Definition:** A standard controller extends `App\Http\Controllers\Controller` (optional but conventional) and contains any number of public methods. Each method corresponds to a route action and can accept dependency-injected parameters (e.g., `Request`, services) and route parameters. The controller is resolved from the service container by `ControllerDispatcher` and invoked with resolved parameters. Standard controllers are generated via `php artisan make:controller ControllerName` without additional flags.

**Beginner-Friendly Explanation:** A standard controller is a blank canvas. You create it, add whatever methods you need—like `dashboard()`, `search()`, or `exportReport()`—and route to them. Unlike resource controllers, there's no preset structure. You decide what methods exist and what they do.

### Purposes

- To organise custom, non-RESTful request-handling logic into a cohesive class.
- To provide a home for actions that don't fit the seven conventional CRUD methods (e.g., `search`, `export`, `approve`).
- To group related custom actions under a single controller for discoverability.
- To serve as the base for all other controller specializations.
- To support rapid prototyping where the full resource structure is unnecessary.

### Syntax Rules and Structure

**Complete General Syntax — Standard Controller:**

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ReportController extends Controller
{
    public function dashboard()
    {
        return view('reports.dashboard');
    }

    public function export(Request $request)
    {
        // Export logic...
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `class ReportController extends Controller` | The controller class extending the base controller. |
| `public function dashboard()` | A custom action method. |
| `public function export(Request $request)` | A custom action with method injection. |

**Complete General Syntax — Routing to Standard Controller Actions:**

```php
use App\Http\Controllers\ReportController;

Route::get('/reports/dashboard', [ReportController::class, 'dashboard']);
Route::post('/reports/export', [ReportController::class, 'export']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[ReportController::class, 'dashboard']` | Array syntax mapping the class and method. |
| `Route::get(...)` | The HTTP verb and URI. |

**Syntax Rules:**

- Controller methods must be `public` to be routable.
- Methods can accept any combination of injected dependencies and route parameters.
- Route parameters must be listed after injected dependencies in the method signature.
- The controller name should be descriptive of the domain area (e.g., `ReportController`, `DashboardController`).
- No specific methods are required; the structure is entirely developer-defined.

**Constraints and Limitations:**

- **No automatic route generation:** Unlike resource controllers, standard controllers require manual route definitions for each action.
- **Route caching:** Standard controllers are cacheable as long as no Closure routes are used.
- **Method organisation:** Without the resource convention, developers must manually organise methods to avoid overly large controllers.
- **Naming consistency:** Unlike resource controllers, there is no enforced naming convention, which can lead to inconsistency across a codebase.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Standard Controller**

```php
<?php
// app/Http/Controllers/DashboardController.php

namespace App\Http\Controllers;

use App\Models\Order;
use App\Models\User;
use Illuminate\View\View;

class DashboardController extends Controller
{
    /**
     * Show the application dashboard.
     */
    public function index(): View
    {
        return view('dashboard', [
            'userCount' => User::count(),
            'orderCount' => Order::count(),
        ]);
    }

    /**
     * Show the reports page.
     */
    public function reports(): View
    {
        return view('reports.index');
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\DashboardController;
use Illuminate\Support\Facades\Route;

Route::get('/dashboard', [DashboardController::class, 'index']);
Route::get('/dashboard/reports', [DashboardController::class, 'reports']);
```

**Setup:** Generate via `php artisan make:controller DashboardController`. Create the corresponding views.

**Expected Output:**
- `GET /dashboard` → renders `dashboard` view with user and order counts.
- `GET /dashboard/reports` → renders `reports.index` view.

**Why:** The standard controller groups related dashboard actions. Each method is custom—`index` and `reports` are not part of the resource convention. Routes are defined manually.

---

**Example 2: Standard Controller with Method Injection**

```php
<?php
// app/Http/Controllers/SearchController.php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class SearchController extends Controller
{
    /**
     * Search for posts.
     */
    public function index(Request $request)
    {
        $query = $request->input('q');

        $posts = Post::where('title', 'like', "%{$query}%")
            ->orWhere('body', 'like', "%{$query}%")
            ->paginate(20);

        return view('search.results', compact('posts', 'query'));
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\SearchController;
use Illuminate\Support\Facades\Route;

Route::get('/search', [SearchController::class, 'index']);
```

**Expected Output:** `GET /search?q=laravel` returns paginated posts matching "laravel" in the title or body.

**Why:** The `Request` instance is method-injected, providing access to the query string. The controller method is custom (`index` is used, but the logic is not a conventional resource list—it's a search with query-string filtering).

---

**Example 3: Standard Controller with Multiple Custom Actions**

```php
<?php
// app/Http/Controllers/InvoiceController.php

namespace App\Http\Controllers;

use App\Models\Invoice;
use Illuminate\Http\Request;

class InvoiceController extends Controller
{
    /**
     * Send an invoice to the customer.
     */
    public function send(Invoice $invoice)
    {
        // Send email...
        return redirect()->back()->with('status', 'Invoice sent!');
    }

    /**
     * Mark an invoice as paid.
     */
    public function markPaid(Invoice $invoice)
    {
        $invoice->update(['paid_at' => now()]);
        return redirect()->route('invoices.index');
    }

    /**
     * Download an invoice as PDF.
     */
    public function download(Invoice $invoice)
    {
        return response()->download(
            storage_path("app/invoices/{$invoice->id}.pdf")
        );
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\InvoiceController;
use Illuminate\Support\Facades\Route;

Route::post('/invoices/{invoice}/send', [InvoiceController::class, 'send']);
Route::patch('/invoices/{invoice}/mark-paid', [InvoiceController::class, 'markPaid']);
Route::get('/invoices/{invoice}/download', [InvoiceController::class, 'download']);
```

**Expected Output:**
- `POST /invoices/1/send` sends invoice 1 and redirects back with a flash message.
- `PATCH /invoices/1/mark-paid` marks invoice 1 as paid and redirects to the index.
- `GET /invoices/1/download` downloads the invoice PDF.

**Why:** These actions are not part of the conventional CRUD set. `send`, `markPaid`, and `download` are custom operations that belong together in a controller focused on invoice management.

### Real-World Cases

- **Dashboard Aggregation:** `DashboardController@index` aggregates statistics from multiple models.
- **Search Functionality:** `SearchController@index` handles full-text search across multiple entities.
- **Report Generation:** `ReportController@export` generates CSV, PDF, or Excel reports.
- **Workflow Actions:** `InvoiceController@send`, `OrderController@approve`, `UserController@suspend`—custom state transitions.
- **Admin Utilities:** `MaintenanceController@clearCache`, `SystemController@health`—administrative operations.

### References

- Laravel Controllers: Basic Controllers — https://laravel.com/docs/12.x/controllers#basic-controllers
- Laravel Controllers: Single Action Controllers — https://laravel.com/docs/12.x/controllers#single-action-controllers
- Laravel Artisan: make:controller — https://laravel.com/docs/12.x/artisan#make-controller

---

## 2. Resource & API Controllers

### Definitions

**Core Definition:** Resource controllers implement the seven conventional RESTful actions (index, create, store, show, edit, update, destroy) for a given resource, while API controllers implement the five stateless actions (index, store, show, update, destroy) suitable for JSON APIs.

**Technical Definition:** A resource controller is generated via `php artisan make:controller ControllerName --resource` and paired with `Route::resource()`. An API controller is generated via `--api` (or `--resource --api`) and paired with `Route::apiResource()`. The `ResourceRegistrar` class maps each action to a specific HTTP verb, URI pattern, and route name. API controllers exclude the `create` and `edit` actions because JSON APIs do not require HTML form-display endpoints.

**Beginner-Friendly Explanation:** A resource controller is a ready-made template for managing one type of thing. It has seven methods—one for listing, one for showing a create form, one for saving, one for viewing a single item, one for showing an edit form, one for updating, and one for deleting. An API controller is the same but drops the two form-related methods because APIs don't return HTML forms.

### Purposes

- To scaffold full multi-action endpoint structures with a single line of code.
- To implement standard architectural actions (index, create, store, show, edit, update, destroy) for unified web screens or stateless JSON APIs.
- To enforce RESTful conventions consistently across an application.
- To reduce boilerplate and potential for routing errors.
- To enable rapid prototyping of CRUD applications.
- To provide predictable URLs and route names for front-end developers.

### Syntax Rules and Structure

**Complete General Syntax — Resource Controller Generation:**

```bash
php artisan make:controller PostController --resource
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--resource` / `-r` | Generates the seven RESTful methods: `index`, `create`, `store`, `show`, `edit`, `update`, `destroy`. |

**Complete General Syntax — API Controller Generation:**

```bash
php artisan make:controller Api/PostController --api
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--api` | Generates five methods (excludes `create` and `edit`). |
| `Api/` | Creates the controller in a subdirectory namespace. |

**Complete General Syntax — Resource Route Registration:**

```php
use App\Http\Controllers\PostController;

Route::resource('posts', PostController::class);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'posts'` | The resource name (plural, kebab-case by convention). |
| `PostController::class` | The controller class handling the resource. |

**Generated Routes:**

| Verb | URI | Action | Route Name |
|------|-----|--------|------------|
| GET | `/posts` | index | posts.index |
| GET | `/posts/create` | create | posts.create |
| POST | `/posts` | store | posts.store |
| GET | `/posts/{post}` | show | posts.show |
| GET | `/posts/{post}/edit` | edit | posts.edit |
| PUT/PATCH | `/posts/{post}` | update | posts.update |
| DELETE | `/posts/{post}` | destroy | posts.destroy |

**Complete General Syntax — API Resource Route Registration:**

```php
Route::apiResource('posts', PostController::class);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Route::apiResource()` | Registers five API routes (excludes `create`, `edit`). |

**Generated API Routes:**

| Verb | URI | Action | Route Name |
|------|-----|--------|------------|
| GET | `/api/posts` | index | posts.index |
| POST | `/api/posts` | store | posts.store |
| GET | `/api/posts/{post}` | show | posts.show |
| PUT/PATCH | `/api/posts/{post}` | update | posts.update |
| DELETE | `/api/posts/{post}` | destroy | posts.destroy |

**Syntax Rules:**

- The resource name is conventionally plural and kebab-case (e.g., `blog-posts`).
- Route parameters are automatically singularised: `posts` → `{post}`.
- Additional routes should be defined **before** `Route::resource()` to avoid wildcard capture.
- API routes defined in `routes/api.php` receive the `/api` prefix and `api` middleware group automatically.
- The `--model` flag binds the controller to an Eloquent model, injecting it into methods via route model binding: `php artisan make:controller PostController --resource --model=Post`.

**Constraints and Limitations:**

- **Wildcard capture:** The `{post}` parameter can capture unintended URIs if additional routes are defined after the resource route.
- **Controller method requirements:** All seven methods must exist on a resource controller, or use `->only()`/`->except()` to limit routes.
- **API statelessness:** API routes lack session and CSRF middleware. Authentication must be handled via tokens (Sanctum, Passport).
- **No built-in shallow nesting:** Use `->shallow()` to avoid requiring parent IDs in child operations.

### Multiple Annotated Complete Code Examples

**Example 1: Resource Controller with All Seven Routes**

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function index() { return Post::all(); }
    public function create() { return view('posts.create'); }
    public function store(Request $request)
    {
        $post = Post::create($request->validated());
        return redirect()->route('posts.show', $post);
    }
    public function show(Post $post) { return view('posts.show', compact('post')); }
    public function edit(Post $post) { return view('posts.edit', compact('post')); }
    public function update(Request $request, Post $post)
    {
        $post->update($request->validated());
        return redirect()->route('posts.show', $post);
    }
    public function destroy(Post $post)
    {
        $post->delete();
        return redirect()->route('posts.index');
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

Route::resource('posts', PostController::class);
```

**Setup:** Generate the controller via `php artisan make:controller PostController --resource`.

**Expected Output:** All seven routes are registered and named automatically. `GET /posts` lists posts; `GET /posts/1` shows post 1; `POST /posts` creates a post.

**Why:** `Route::resource()` registers the seven conventional routes. Each route uses implicit model binding: `{post}` automatically resolves to a `Post` model instance in `show`, `edit`, `update`, and `destroy`.

---

**Example 2: API Controller with Five Routes**

```php
<?php
// app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function index() { return Post::all(); }
    public function store(Request $request)
    {
        return Post::create($request->validated());
    }
    public function show(Post $post) { return $post; }
    public function update(Request $request, Post $post)
    {
        $post->update($request->validated());
        return $post->fresh();
    }
    public function destroy(Post $post)
    {
        $post->delete();
        return response()->noContent();
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Controllers\Api\PostController;
use Illuminate\Support\Facades\Route;

Route::apiResource('posts', PostController::class);
```

**Expected Output:**
- `GET /api/posts` → JSON array of posts.
- `POST /api/posts` → JSON object of the created post.
- `GET /api/posts/1` → JSON object of post 1.
- `PUT /api/posts/1` → JSON object of the updated post.
- `DELETE /api/posts/1` → 204 No Content.

**Why:** `apiResource()` registers only the five stateless routes. The `/api` prefix and `api` middleware group are applied automatically because the route is defined in `routes/api.php`.

---

**Example 3: Resource Controller with Partial Mapping**

```php
<?php
// routes/web.php

use App\Http\Controllers\CategoryController;
use Illuminate\Support\Facades\Route;

// Only register index and show — no create, edit, or delete
Route::resource('categories', CategoryController::class)
    ->only(['index', 'show']);
```

**Expected Output:** Only `GET /categories` and `GET /categories/{category}` are registered. `POST /categories` returns 405 Method Not Allowed.

**Why:** The `only()` method acts as a whitelist, registering only the two specified actions. This reduces the attack surface and route count.

### Real-World Cases

- **Blog Platforms:** `Route::resource('posts', PostController::class)` handles all blog post CRUD operations.
- **E-commerce:** `Route::apiResource('products', ProductController::class)` manages the product catalogue via JSON API.
- **User Management:** `Route::resource('users', UserController::class)` handles user CRUD in admin panels.
- **Mobile App Backends:** `Route::apiResource('posts', Api\PostController::class)` serves JSON to iOS/Android apps.
- **Content Management:** `Route::resource('articles', ArticleController::class)->only(['index', 'show'])` for a read-only public article directory.

### References

- Laravel Controllers: Resource Controllers — https://laravel.com/docs/12.x/controllers#resource-controllers
- Laravel Controllers: API Resource Routes — https://laravel.com/docs/12.x/controllers#api-resource-routes
- Laravel Controllers: Partial Resource Routes — https://laravel.com/docs/12.x/controllers#restful-partial-resource-routes
- Laravel Routing: Resource Routes — https://laravel.com/docs/12.x/routing#resource-routes

---

## 3. Invokable (Single-Action) Controllers

### Definitions

**Core Definition:** An invokable controller is a controller class that implements the PHP `__invoke()` magic method, dedicating the entire class to a single operational task.

**Technical Definition:** The `__invoke()` magic method allows a PHP object to be called as a function. When a route references an invokable controller by class name only (e.g., `Route::post('/server', ProvisionServer::class)`), Laravel's `ControllerDispatcher` invokes the `__invoke()` method on the resolved controller instance. Invokable controllers are generated via `php artisan make:controller ProvisionServer --invokable`. This pattern was introduced in Laravel 5.6.28 and is ideal for actions that do not share state or methods with other actions.

**Beginner-Friendly Explanation:** Sometimes a controller only needs to do one thing—like process a payment, provision a server, or send a notification. Instead of creating a full controller with multiple methods, you create a single-action controller with just one method: `__invoke()`. It's like a specialised tool for one job.

### Purposes

- To create clean, highly isolated controllers dedicated to a single operational task.
- To reduce controller bloat by separating one-off actions from multi-action controllers.
- To enforce the single responsibility principle at the controller level.
- To simplify route definitions: `Route::post('/server', ProvisionServer::class)` requires no method name.
- To improve code organisation by naming controllers after the action they perform (e.g., `ProcessPaymentController`, `SendInvoiceController`).
- To provide a dedicated home for webhook handlers, health checks, and one-off administrative actions.

### Syntax Rules and Structure

**Complete General Syntax — Invokable Controller:**

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     */
    public function __invoke(Request $request)
    {
        // Provisioning logic...
        return response()->json(['status' => 'provisioned']);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `class ProvisionServer extends Controller` | The controller class name should describe the action. |
| `public function __invoke(Request $request)` | The single action method, invoked automatically. |
| `return response()->json(...)` | The response returned to the client. |

**Complete General Syntax — Routing to an Invokable Controller:**

```php
use App\Http\Controllers\ProvisionServer;

// Only the class name is required — no method array
Route::post('/server', ProvisionServer::class);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Route::post('/server', ProvisionServer::class)` | The router automatically invokes `__invoke()`. |
| No method name | Invokable controllers are referenced by class name only. |

**Complete General Syntax — Artisan Generation:**

```bash
php artisan make:controller ProvisionServer --invokable
# or shorthand: -i
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--invokable` / `-i` | Generates a controller with only the `__invoke()` method. |

**Syntax Rules:**

- The controller must define a `public function __invoke()` method.
- The route references the controller by class name only (no array with method name).
- The `__invoke()` method can accept any dependencies and route parameters, just like any other action method.
- The controller name should describe the action (e.g., `ProcessPaymentController`, `SendWelcomeEmailController`).
- Invokable controllers can be generated via `--invokable` or by manually creating the `__invoke()` method.

**Constraints and Limitations:**

- **Single action only:** Invokable controllers cannot handle multiple routes with different logic. For multiple actions, use a standard or resource controller.
- **Naming convention:** Controllers should be named after the action they perform. Generic names like `ActionController` reduce clarity.
- **Route caching:** Invokable controllers are cacheable as long as no Closure routes are used.
- **Laravel version:** The `--invokable` flag and `__invoke` pattern are available from Laravel 5.6.28+. In older versions, use a standard controller with a single method.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Invokable Controller**

```php
<?php
// app/Http/Controllers/ProcessPaymentController.php

namespace App\Http\Controllers;

use App\Services\PaymentGateway;
use Illuminate\Http\Request;

class ProcessPaymentController extends Controller
{
    /**
     * Process a payment.
     */
    public function __invoke(Request $request, PaymentGateway $gateway)
    {
        $result = $gateway->charge(
            $request->input('amount'),
            $request->input('token')
        );

        return response()->json(['status' => $result]);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\ProcessPaymentController;
use Illuminate\Support\Facades\Route;

Route::post('/payment', ProcessPaymentController::class);
```

**Setup:** Generate via `php artisan make:controller ProcessPaymentController --invokable`. Create a `PaymentGateway` service.

**Expected Output:** `POST /payment` processes the payment through the injected gateway and returns `{"status":"success"}` (or the gateway's result).

**Why:** The `__invoke()` method is the sole entry point. The `PaymentGateway` service is method-injected alongside the `Request` instance. The route references only the class name.

---

**Example 2: Invokable Controller for Webhook Handling**

```php
<?php
// app/Http/Controllers/StripeWebhookController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;

class StripeWebhookController extends Controller
{
    /**
     * Handle a Stripe webhook.
     */
    public function __invoke(Request $request)
    {
        $payload = $request->all();
        $event = $payload['type'] ?? 'unknown';

        Log::info('Stripe webhook received', ['event' => $event]);

        // Process the event...
        return response()->json(['status' => 'received']);
    }
}
```

```php
<?php
// routes/api.php

use App\Http\Controllers\StripeWebhookController;
use Illuminate\Support\Facades\Route;

Route::post('/webhooks/stripe', StripeWebhookController::class);
```

**Expected Output:** `POST /api/webhooks/stripe` logs the webhook event and returns `{"status":"received"}`.

**Why:** Webhook handlers are single-purpose: receive a payload, process it, acknowledge receipt. An invokable controller keeps this logic isolated and clearly named.

---

**Example 3: Invokable Controller with Route Parameter**

```php
<?php
// app/Http/Controllers/ShowInvoiceController.php

namespace App\Http\Controllers;

use App\Models\Invoice;

class ShowInvoiceController extends Controller
{
    /**
     * Display a single invoice.
     */
    public function __invoke(Invoice $invoice)
    {
        return view('invoices.show', compact('invoice'));
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\ShowInvoiceController;
use Illuminate\Support\Facades\Route;

Route::get('/invoices/{invoice}', ShowInvoiceController::class);
```

**Expected Output:** `GET /invoices/5` renders the `invoices.show` view with invoice 5.

**Why:** Even with route model binding, the invokable pattern works seamlessly. The `{invoice}` parameter is resolved to an `Invoice` model and injected into `__invoke()`.

### Real-World Cases

- **Payment Processing:** `ProcessPaymentController` handles a single payment transaction.
- **Webhook Receivers:** `StripeWebhookController`, `GitHubWebhookController`—each receives and processes a specific webhook type.
- **Server Provisioning:** `ProvisionServerController` handles a single provisioning request.
- **User Registration:** `RegisterUserController` handles a single registration flow.
- **Email Sending:** `SendWelcomeEmailController` sends a welcome email.
- **Password Reset:** `SendPasswordResetLinkController` and `ResetPasswordController` each handle one step of the password reset flow.

### References

- Laravel Controllers: Single Action Controllers — https://laravel.com/docs/12.x/controllers#single-action-controllers
- Laravel Artisan: make:controller --invokable — https://laravel.com/docs/12.x/artisan#make-controller
- Laravel Daily: Invokable Controllers — https://laraveldaily.com/post/laravel-invokable-controllers

---

## 4. Nested Resource Controllers

### Definitions

**Core Definition:** A nested resource controller manages a child resource that belongs exclusively to a parent resource, using dot-notation routing to generate URIs that include both parent and child identifiers.

**Technical Definition:** Nested resource controllers are registered via `Route::resource('posts.comments', CommentController::class)`, which generates routes with URIs like `/posts/{post}/comments` and `/posts/{post}/comments/{comment}`. The `ResourceRegistrar` handles parameter naming for both parent and child resources, and Laravel's implicit model binding automatically resolves both models. The `->scoped()` modifier (Laravel 8+) enforces that the child model belongs to the parent by using the parent's relationship. The `->shallow()` modifier (Laravel 9+) removes the parent parameter from child-specific routes (show, edit, update, destroy) where the child can be uniquely identified by its own ID.

**Beginner-Friendly Explanation:** Some data only makes sense in the context of its parent. A comment belongs to a specific blog post. A task belongs to a specific project. Nested resource controllers let you express this relationship directly in your URLs: `/posts/5/comments/3` means "comment 3 on post 5." Laravel automatically checks that the comment actually belongs to that post.

### Purposes

- To handle deeply coupled data relationships where child resources cannot exist without their parent.
- To enforce parent-child integrity at the routing layer through scoped bindings.
- To provide clean, descriptive URLs that reflect the data hierarchy (e.g., `/posts/5/comments`).
- To eliminate manual ownership checks from controller methods.
- To enable shallow nesting where child operations don't require the parent identifier in the URL.
- To support multi-level nesting (e.g., user → post → comment).

### Syntax Rules and Structure

**Complete General Syntax — Nested Resource Registration:**

```php
use App\Http\Controllers\CommentController;
use App\Http\Controllers\PostController;

Route::resource('posts', PostController::class);
Route::resource('posts.comments', CommentController::class);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'posts.comments'` | Dot notation creates nested routes. |
| Generated URIs | `/posts/{post}/comments`, `/posts/{post}/comments/{comment}`, etc. |

**Generated Nested Routes:**

| Verb | URI | Action |
|------|-----|--------|
| GET | `/posts/{post}/comments` | index |
| GET | `/posts/{post}/comments/create` | create |
| POST | `/posts/{post}/comments` | store |
| GET | `/posts/{post}/comments/{comment}` | show |
| GET | `/posts/{post}/comments/{comment}/edit` | edit |
| PUT/PATCH | `/posts/{post}/comments/{comment}` | update |
| DELETE | `/posts/{post}/comments/{comment}` | destroy |

**Complete General Syntax — Scoped Nested Resources:**

```php
use App\Http\Controllers\CommentController;

Route::resource('posts.comments', CommentController::class)
    ->scoped(['comment' => 'slug']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->scoped(['comment' => 'slug'])` | Enables automatic scoping and specifies the child's lookup key. |
| `'comment' => 'slug'` | Resolve comments by their `slug` column instead of `id`. |

**Complete General Syntax — Shallow Nested Resources:**

```php
use App\Http\Controllers\CommentController;

Route::resource('posts.comments', CommentController::class)->shallow();
```

**Generated Shallow Routes:**

| Verb | URI | Action |
|------|-----|--------|
| GET | `/posts/{post}/comments` | index |
| GET | `/posts/{post}/comments/create` | create |
| POST | `/posts/{post}/comments` | store |
| GET | `/comments/{comment}` | show |
| GET | `/comments/{comment}/edit` | edit |
| PUT/PATCH | `/comments/{comment}` | update |
| DELETE | `/comments/{comment}` | destroy |

**Complete General Syntax — API Nested Resources:**

```php
use App\Http\Controllers\Api\CommentController;

Route::apiResource('posts.comments', CommentController::class);
```

**Syntax Rules:**

- Nested resources use dot notation: `'posts.comments'`.
- The parent resource route must be defined before or alongside the nested route.
- Scoped bindings (`->scoped()`) require the parent model to have a relationship whose name matches the pluralised child parameter (e.g., `comments` for `{comment}`).
- Shallow nesting (`->shallow()`) removes the parent parameter from `show`, `edit`, `update`, and `destroy` routes.
- Custom keys in nested bindings (e.g., `{post:slug}`) automatically enable scoping.

**Constraints and Limitations:**

- **Relationship naming:** Scoped bindings assume the parent model has a relationship named after the plural of the child parameter. Override via `resolveChildRouteBinding()` if the relationship name differs.
- **Performance:** Scoped bindings add a `whereHas` or relationship constraint to the child query. Ensure appropriate database indexes exist.
- **Depth:** Nesting beyond two or three levels (e.g., `users.posts.comments.replies`) becomes difficult to manage and is generally discouraged.
- **Shallow nesting:** Shallow nesting is only appropriate when the child model has a unique identifier (e.g., auto-incrementing ID, UUID) that is globally unique.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Nested Resource Controller**

```php
<?php
// app/Http/Controllers/CommentController.php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Http\Request;

class CommentController extends Controller
{
    // GET /posts/{post}/comments
    public function index(Post $post)
    {
        return $post->comments;
    }

    // GET /posts/{post}/comments/{comment}
    public function show(Post $post, Comment $comment)
    {
        return $comment;
    }

    // POST /posts/{post}/comments
    public function store(Request $request, Post $post)
    {
        $comment = $post->comments()->create($request->validated());
        return redirect()->route('posts.comments.show', [$post, $comment]);
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\CommentController;
use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

Route::resource('posts', PostController::class);
Route::resource('posts.comments', CommentController::class);
```

**Expected Output:**
- `GET /posts/1/comments` → returns comments for post 1.
- `GET /posts/1/comments/5` → returns comment 5 (if it belongs to post 1).
- `POST /posts/1/comments` → creates a comment on post 1.

**Why:** The dot-notation `'posts.comments'` generates nested routes. Both `{post}` and `{comment}` parameters are injected into the controller methods in order. The `store` method uses the `$post->comments()` relationship to create the comment, ensuring the parent-child link.

---

**Example 2: Scoped Nested Resource with Custom Key**

```php
<?php
// routes/web.php

use App\Http\Controllers\CommentController;
use Illuminate\Support\Facades\Route;

// Scoped binding: comment is resolved by slug AND must belong to the post
Route::resource('posts.comments', CommentController::class)
    ->scoped(['comment' => 'slug']);
```

```php
<?php
// app/Http/Controllers/CommentController.php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;

class CommentController extends Controller
{
    public function show(Post $post, Comment $comment)
    {
        // Laravel has already verified that $comment belongs to $post
        return view('comments.show', compact('post', 'comment'));
    }
}
```

**Setup:** Ensure the `Post` model has a `comments()` relationship (`hasMany(Comment::class)`).

**Expected Output:** `GET /posts/1/comments/my-comment` resolves the comment with slug `my-comment` **only if** it belongs to post 1. If the comment belongs to a different post, a 404 is returned.

**Why:** The `->scoped(['comment' => 'slug'])` modifier instructs Laravel to resolve the comment by slug **and** verify ownership through the `comments` relationship on the `Post` model. This eliminates manual ownership checks.

---

**Example 3: Shallow Nested Resource for Simplified Child URLs**

```php
<?php
// routes/web.php

use App\Http\Controllers\CommentController;
use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

Route::resource('posts', PostController::class);

// Shallow nesting: only index, create, store require parent ID
Route::resource('posts.comments', CommentController::class)->shallow();
```

**Expected Output:**
- `GET /posts/1/comments` → list comments for post 1 (parent required).
- `POST /posts/1/comments` → create a comment on post 1 (parent required).
- `GET /comments/5` → show comment 5 (parent not required).
- `PUT /comments/5` → update comment 5 (parent not required).
- `DELETE /comments/5` → delete comment 5 (parent not required).

**Why:** The `->shallow()` modifier removes the parent parameter from routes where the child can be uniquely identified by its own ID. This simplifies child-specific URLs while preserving the parent context for collection-level actions.

### Real-World Cases

- **Blog Comments:** `/posts/{post}/comments/{comment}` manages comments belonging to a specific blog post.
- **E-commerce Order Items:** `/orders/{order}/items/{item}` manages line items within an order.
- **Project Tasks:** `/projects/{project}/tasks/{task}` manages tasks within a project.
- **User Posts:** `/users/{user}/posts/{post}` manages posts authored by a specific user.
- **Course Lessons:** `/courses/{course}/lessons/{lesson}` manages lessons within a course.
- **Warehouse Products:** `/warehouses/{warehouse}/products/{product}` manages inventory within a warehouse.

### References

- Laravel Controllers: Nested Resources — https://laravel.com/docs/12.x/controllers#restful-nested-resources
- Laravel Controllers: Scoping Nested Resources — https://laravel.com/docs/12.x/controllers#restful-scoping-nested-resources
- Laravel News: Scoped Route Binding for Nested Resource Management — https://laravel-news.com/route-resource-scoped
- Laravel News: Route Optimization through Shallow Resource Architecture — https://laravel-news.com/route-shallow-resource
- Laravel Routing: Shallow Nesting — https://laravel.com/docs/12.x/controllers#restful-shallow-nesting

---

## References

- Laravel Controllers Documentation (12.x) — https://laravel.com/docs/12.x/controllers
- Laravel Controllers: Resource Controllers — https://laravel.com/docs/12.x/controllers#resource-controllers
- Laravel Controllers: API Resource Routes — https://laravel.com/docs/12.x/controllers#api-resource-routes
- Laravel Controllers: Single Action Controllers — https://laravel.com/docs/12.x/controllers#single-action-controllers
- Laravel Controllers: Nested Resources — https://laravel.com/docs/12.x/controllers#restful-nested-resources
- Laravel Controllers: Scoping Nested Resources — https://laravel.com/docs/12.x/controllers#restful-scoping-nested-resources
- Laravel Controllers: Shallow Nesting — https://laravel.com/docs/12.x/controllers#restful-shallow-nesting
- Laravel Controllers: Partial Resource Routes — https://laravel.com/docs/12.x/controllers#restful-partial-resource-routes
- Laravel Routing Documentation — https://laravel.com/docs/12.x/routing
- Laravel Artisan: make:controller — https://laravel.com/docs/12.x/artisan#make-controller
- Laravel API: HasMiddleware Interface — https://api.laravel.com/docs/10.x/Illuminate/Routing/Controllers/HasMiddleware.html
- Laravel API: Middleware Class — https://api.laravel.com/docs/10.x/Illuminate/Routing/Controllers/Middleware.html
- Laravel News: Scoped Route Binding for Nested Resource Management — https://laravel-news.com/route-resource-scoped
- Laravel News: Route Optimization through Shallow Resource Architecture — https://laravel-news.com/route-shallow-resource
- Laravel Daily: 60 Laravel Quick Tips (Invokable Controllers) — https://laraveldaily.com/wp-content/uploads/2019/04/laravel-tips-2019-04.pdf
- Generalist Programmer: Laravel Controllers: Complete Guide to Resource, Invokable & More — https://generalistprogrammer.com/laravel-controllers
- GitHub: Laravel Docs Controllers.md (12.x) — https://github.com/laravel/docs/blob/12.x/controllers.md