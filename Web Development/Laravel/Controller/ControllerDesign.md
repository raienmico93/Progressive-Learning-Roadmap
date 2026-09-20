# Laravel Controller Design: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Controller Design is the practice of structuring controller classes to handle HTTP requests efficiently while maintaining a strict separation of concerns, keeping controllers thin, delegating business logic to dedicated service classes, validating input through form requests, and transforming responses into consistent output formats.

**Technical Definition:** Controller design in Laravel involves adhering to the Model-View-Controller (MVC) architectural pattern, where controllers act as the orchestration layer between incoming HTTP requests and the application's domain logic. A well-designed controller is responsible only for HTTP concerns: reading request data, delegating to services or actions, and returning a response. Business logic resides in service classes or action classes, validation rules are encapsulated in Form Request classes, and response transformation is handled by Eloquent API Resources or similar transformers.

**Beginner-Friendly Explanation:** A controller is like a restaurant waiter. The waiter takes your order (the HTTP request), tells the kitchen what to make (delegates to a service class), and brings you your food (returns a response). The waiter shouldn't be cooking the food—that's the kitchen's job. Keeping the waiter's role focused on taking orders and delivering food makes the restaurant run smoothly. Similarly, keeping controllers thin makes your application easier to maintain, test, and grow.

### Key Characteristics

- **Separation of Concerns:** Controllers handle HTTP concerns; services handle business logic; models handle data.
- **Thin Controllers:** Controller methods are short, readable, and focused on orchestration, not implementation.
- **Delegated Business Logic:** Complex operations are moved to service classes, action classes, or model methods.
- **Centralised Validation:** Validation rules live in Form Request classes, not inline in controllers.
- **Transformed Responses:** API responses are shaped by resource classes, not raw model output.
- **Testability:** Each layer can be tested in isolation—controllers with HTTP tests, services with unit tests.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (the concepts are stable across versions; Form Requests, API Resources, and service container features are available across these versions).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with the `app/Http/Controllers` directory.
- Basic understanding of routing, Eloquent models, and the service container.

### Related Programming Areas

- **MVC Architecture:** Controller design is the practical application of MVC principles in Laravel.
- **Service Container & Dependency Injection:** Controllers resolve dependencies through Laravel's IoC container.
- **Form Request Validation:** A dedicated validation layer that keeps controllers clean.
- **Eloquent API Resources:** A response transformation layer for JSON APIs.
- **Service Layer / Repository Pattern:** Architectural patterns for organising business logic and data access.

### Core Concepts / Features

1. Separation of Concerns
2. Thin Controllers
3. Delegating Business Logic
4. Service Classes
5. Request Validation
6. Response Transformation

---

## 1. Separation of Concerns

### Definitions

**Core Definition:** Separation of concerns is the design principle that each component of an application should have a single, well-defined responsibility, with clear boundaries between the HTTP layer, the business logic layer, and the data access layer.

**Technical Definition:** In Laravel, separation of concerns is implemented through the Model-View-Controller (MVC) pattern. Controllers handle HTTP requests and orchestrate responses. Models manage data and persistence. Views render presentation. Service classes encapsulate reusable business logic that spans multiple models or external services. Form Requests handle validation. API Resources handle response transformation. Each layer communicates through well-defined interfaces, and dependencies flow in one direction: HTTP → Service → Repository → Model.

**Beginner-Friendly Explanation:** Imagine a restaurant. The waiter takes orders (controller), the kitchen cooks the food (model/services), and the presentation on the plate is the view. Each person has a specific job. If the waiter tried to cook, or the chef tried to serve, the restaurant would be chaos. Separation of concerns means each part of your application does one thing well.

### Purposes

- To keep controllers focused on HTTP orchestration, not business logic.
- To make business logic reusable across multiple controllers, jobs, and CLI commands.
- To improve testability by allowing each layer to be tested in isolation.
- To reduce the risk of code duplication and inconsistencies.
- To enable the application to scale by allowing teams to work on different layers independently.
- To provide a clear mental model for where new code should be placed.

### Syntax Rules and Structure

**Complete General Syntax — Layer Responsibilities:**

```
HTTP Layer (Controllers):
  - Read request data
  - Delegate to services/actions
  - Return responses (views, JSON, redirects)

Business Logic Layer (Services/Actions):
  - Encapsulate business rules
  - Coordinate between multiple models
  - Interface with external services

Data Access Layer (Models/Repositories):
  - Query the database
  - Define relationships
  - Manage persistence

Validation Layer (Form Requests):
  - Define validation rules
  - Authorize requests
  - Prepare input data

Transformation Layer (API Resources):
  - Shape JSON output
  - Hide sensitive attributes
  - Format dates, numbers, relationships
```

**Component Breakdown:**

| Layer | Responsibility | Example Class |
|-------|---------------|---------------|
| Controller | HTTP orchestration | `PostController` |
| Form Request | Input validation | `StorePostRequest` |
| Service | Business logic | `PostService` |
| Action | Single business operation | `CreatePostAction` |
| Model | Data & persistence | `Post` |
| API Resource | Response transformation | `PostResource` |

**Syntax Rules:**

- Controllers should never contain complex business logic—delegate to services or actions.
- Services should never depend on HTTP-specific classes (e.g., `Request`)—accept plain arrays or DTOs.
- Form Requests should handle all validation; controllers call `$request->validated()`.
- API Resources should be the only place where response shaping logic lives.
- Models should contain relationships, scopes, accessors, and mutators—but not complex workflows.

**Constraints and Limitations:**

- **Over-engineering:** Not every application needs a full service layer. Small applications may be fine with thin controllers and model methods.
- **Performance:** Additional layers introduce slight overhead, though this is negligible in practice.
- **Team conventions:** The exact boundaries between services, actions, and repositories are team decisions, not framework mandates.

### Multiple Annotated Complete Code Examples

**Example 1: Poor Separation of Concerns (Fat Controller)**

```php
<?php
// BAD: All logic crammed into the controller

namespace App\Http\Controllers;

use App\Models\Order;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        // Validation inline
        $validated = $request->validate([
            'items' => 'required|array',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
        ]);

        // Business logic inline
        $order = Order::create(['user_id' => auth()->id()]);
        $total = 0;

        foreach ($validated['items'] as $item) {
            $product = \App\Models\Product::find($item['product_id']);
            $order->items()->create([
                'product_id' => $product->id,
                'quantity' => $item['quantity'],
                'price' => $product->price,
            ]);
            $total += $product->price * $item['quantity'];
            $product->decrement('stock', $item['quantity']);
        }

        $order->update(['total' => $total]);

        // Notification logic inline
        Mail::to($order->user)->send(new \App\Mail\OrderConfirmation($order));

        return response()->json($order, 201);
    }
}
```

**Expected Output:** The order is created, but the controller is doing five different jobs: validation, order creation, inventory management, total calculation, and notification. This is difficult to test, maintain, and reuse.

**Why:** This violates separation of concerns. If the notification logic changes, or if you need to create orders from a CLI command, you'd have to duplicate all this logic.

---

**Example 2: Proper Separation of Concerns**

```php
<?php
// GOOD: Each layer has its own responsibility

// app/Http/Requests/StoreOrderRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreOrderRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'items' => 'required|array',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
        ];
    }
}
```

```php
<?php
// app/Services/OrderService.php
namespace App\Services;

use App\Mail\OrderConfirmation;
use App\Models\Order;
use App\Models\Product;
use Illuminate\Support\Facades\Mail;

class OrderService
{
    public function createOrder(int $userId, array $items): Order
    {
        $order = Order::create(['user_id' => $userId]);
        $total = 0;

        foreach ($items as $item) {
            $product = Product::findOrFail($item['product_id']);
            $order->items()->create([
                'product_id' => $product->id,
                'quantity' => $item['quantity'],
                'price' => $product->price,
            ]);
            $total += $product->price * $item['quantity'];
            $product->decrement('stock', $item['quantity']);
        }

        $order->update(['total' => $total]);

        Mail::to($order->user)->send(new OrderConfirmation($order));

        return $order;
    }
}
```

```php
<?php
// app/Http/Controllers/OrderController.php
namespace App\Http\Controllers;

use App\Http\Requests\StoreOrderRequest;
use App\Services\OrderService;

class OrderController extends Controller
{
    public function __construct(
        private OrderService $orderService
    ) {}

    public function store(StoreOrderRequest $request)
    {
        $order = $this->orderService->createOrder(
            auth()->id(),
            $request->validated()['items']
        );

        return response()->json($order, 201);
    }
}
```

**Expected Output:** The controller is thin, readable, and focused on HTTP orchestration. The service handles business logic. The Form Request handles validation.

**Why:** Each layer has a single responsibility. The service can be reused from a CLI command or queue job. The controller is easy to test because it only orchestrates. The validation rules are reusable across multiple endpoints.

---

**Example 3: Dependency Flow in a Layered Architecture**

```php
<?php
// The dependency flow: HTTP → Service → Repository → Model

// 1. Controller (HTTP Layer)
namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Services\PostService;

class PostController extends Controller
{
    public function __construct(
        private PostService $postService
    ) {}

    public function store(StorePostRequest $request)
    {
        $post = $this->postService->create($request->validated());
        return new \App\Http\Resources\PostResource($post);
    }
}
```

```php
<?php
// 2. Service (Business Logic Layer)
namespace App\Services;

use App\Repositories\PostRepositoryInterface;

class PostService
{
    public function __construct(
        private PostRepositoryInterface $posts
    ) {}

    public function create(array $data)
    {
        // Business logic: e.g., slug generation, event dispatch
        $data['slug'] = \Str::slug($data['title']);
        return $this->posts->create($data);
    }
}
```

```php
<?php
// 3. Repository (Data Access Layer)
namespace App\Repositories;

use App\Models\Post;

class EloquentPostRepository implements PostRepositoryInterface
{
    public function create(array $data): Post
    {
        return Post::create($data);
    }
}
```

**Expected Output:** The controller depends on the service, the service depends on the repository interface, and the repository depends on the Eloquent model. Each layer knows only about the layer directly below it.

**Why:** This follows the Dependency Inversion Principle. The repository interface allows the data source to be swapped (e.g., for an API or in-memory store) without changing the service or controller.

### Real-World Cases

- **E-commerce:** `OrderController` delegates to `OrderService`, which coordinates payment, inventory, and notifications.
- **SaaS Multi-Tenancy:** `TenantController` delegates to `TenantService`, which handles provisioning, database creation, and configuration.
- **Blog Platform:** `PostController` delegates to `PostService`, which handles slug generation, publishing, and tagging.
- **API Development:** Controllers delegate to services, which return DTOs or models that are transformed by API Resources.

### References

- Laravel: What is MVC? — https://laravel.com/learn/getting-started-with-laravel/what-is-mvc
- Clean Code in Laravel: Organizing Your Application — https://github.com/ahmadmayahi/clean-code-in-laravel/blob/main/02-write-clean/004-organizing-your-application.md
- Laravel Service Container — https://laravel.com/docs/master/container

---

## 2. Thin Controllers

### Definitions

**Core Definition:** A thin controller is a controller whose methods contain minimal code—typically only reading request data, delegating to a service or action, and returning a response—with no business logic, complex conditionals, or database queries.

**Technical Definition:** The "thin controller" principle states that controllers should be limited to HTTP concerns: receiving a request, passing data to a service or action, and returning a response. The controller's method body should be short enough to read in a single glance, typically under 10 lines. Business logic, database queries, and complex operations are delegated to service classes, action classes, or model methods. Taylor Otwell, the creator of Laravel, has stated his preference for "thin controllers" with "fat models" that expose a rich API of public methods, even if those methods delegate to other services behind the scenes.

**Beginner-Friendly Explanation:** A thin controller is like a receptionist who takes your message and passes it to the right person. The receptionist doesn't try to solve your problem—they just route you to someone who can. If your controller method is more than a few lines long, it's probably doing too much.

### Purposes

- To make controller methods readable and easy to understand at a glance.
- To reduce the cognitive load when navigating the codebase.
- To make controllers easier to test by isolating HTTP behaviour from business logic.
- To prevent code duplication by centralising logic in reusable services.
- To improve maintainability by separating concerns and reducing coupling.
- To align with the Single Responsibility Principle (SRP).

### Syntax Rules and Structure

**Complete General Syntax — Thin Controller Method:**

```php
public function store(StorePostRequest $request)
{
    $post = $this->postService->create($request->validated());
    return new PostResource($post);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `StorePostRequest $request` | Form Request handles validation automatically. |
| `$this->postService->create(...)` | Delegates business logic to a service. |
| `new PostResource($post)` | Transforms the response via an API Resource. |
| Return statement | Returns the response—no complex logic. |

**Complete General Syntax — Fat vs. Thin Comparison:**

```php
// FAT CONTROLLER (BAD)
public function store(Request $request)
{
    $validated = $request->validate([...]);
    $post = Post::create($validated);
    $post->tags()->sync($request->tags);
    Cache::forget('posts.all');
    Mail::to($post->author)->send(new PostPublished($post));
    event(new PostCreated($post));
    return response()->json($post, 201);
}

// THIN CONTROLLER (GOOD)
public function store(StorePostRequest $request)
{
    $post = $this->postService->create($request->validated());
    return new PostResource($post);
}
```

**Syntax Rules:**

- A controller method should ideally be 3–10 lines long.
- Validation should be handled by Form Requests, not inline `$request->validate()`.
- Business logic should be delegated to services or actions.
- Response transformation should be handled by API Resources or view composers.
- Database queries should not appear in controllers—use services or repositories.

**Constraints and Limitations:**

- **Thin vs. anemic:** A controller can be too thin—if it just passes data through without any orchestration, it may be an unnecessary layer. Thin controllers should still orchestrate the request flow.
- **Service proliferation:** Over-delegation can lead to a proliferation of tiny service classes. Group related operations into cohesive services.
- **Team conventions:** There is no universally agreed-upon "maximum lines" for a controller method. The goal is readability, not an arbitrary line count.

### Multiple Annotated Complete Code Examples

**Example 1: Thin Controller with Service Delegation**

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Http\Resources\PostResource;
use App\Services\PostService;

class PostController extends Controller
{
    public function __construct(
        private PostService $postService
    ) {}

    /**
     * Store a newly created post.
     */
    public function store(StorePostRequest $request)
    {
        // 1. Validation is handled by StorePostRequest
        // 2. Business logic is delegated to PostService
        // 3. Response transformation is handled by PostResource
        $post = $this->postService->create($request->validated());

        return new PostResource($post);
    }

    /**
     * Display the specified post.
     */
    public function show(Post $post)
    {
        return new PostResource($post);
    }

    /**
     * Update the specified post.
     */
    public function update(UpdatePostRequest $request, Post $post)
    {
        $post = $this->postService->update($post, $request->validated());

        return new PostResource($post);
    }
}
```

**Expected Output:** Each controller method is 1–3 lines of actual logic. Validation, business logic, and response transformation are handled by dedicated classes.

**Why:** The controller is thin and readable. It orchestrates the request flow without implementing any of the underlying logic. This makes the controller easy to test (mock the service) and easy to modify (change the service without touching the controller).

---

**Example 2: Thin Controller with Action Class**

```php
<?php
// app/Actions/CreatePostAction.php

namespace App\Actions;

use App\Models\Post;

class CreatePostAction
{
    public function execute(array $data): Post
    {
        $post = Post::create($data);
        $post->tags()->sync($data['tags'] ?? []);
        event(new \App\Events\PostCreated($post));
        return $post;
    }
}
```

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Actions\CreatePostAction;
use App\Http\Requests\StorePostRequest;
use App\Http\Resources\PostResource;

class PostController extends Controller
{
    public function store(StorePostRequest $request, CreatePostAction $action)
    {
        $post = $action->execute($request->validated());
        return new PostResource($post);
    }
}
```

**Expected Output:** The controller method is two lines. The `CreatePostAction` encapsulates a single business operation (creating a post with tags and dispatching an event).

**Why:** Action classes are ideal for single-purpose business operations. They have a single public method (`execute`) and are easy to test in isolation. The controller remains thin.

---

**Example 3: Thin Controller with Model Method Delegation**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    /**
     * Suspend the user account.
     */
    public function suspend(string $reason): void
    {
        $this->update(['suspended_at' => now(), 'suspension_reason' => $reason]);
        $this->tokens()->delete(); // Revoke API tokens
        event(new \App\Events\UserSuspended($this));
    }
}
```

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Http\Requests\SuspendUserRequest;
use App\Models\User;

class UserController extends Controller
{
    public function suspend(SuspendUserRequest $request, User $user)
    {
        $user->suspend($request->validated('reason'));
        return response()->noContent();
    }
}
```

**Expected Output:** The controller method is two lines. The `suspend` method on the `User` model encapsulates the business logic. This follows Taylor Otwell's "fat model" approach, where models expose a rich API of public methods.

**Why:** The model provides a fluent, readable interface (`$user->suspend($reason)`) while delegating the actual work to other services or events behind the scenes. The controller remains thin.

### Real-World Cases

- **API Controllers:** Thin controllers that delegate to services and return API Resources are the standard for modern Laravel APIs.
- **Admin Panels:** Admin controllers delegate to services for user management, content moderation, and configuration.
- **Webhook Handlers:** Thin controllers that pass webhook payloads to action classes for processing.
- **Form Submissions:** Thin controllers that validate via Form Requests, delegate to services, and redirect with flash messages.

### References

- Taylor Otwell: "Thin" Controllers, "Fat" Models Approach — https://laraveldaily.com/post/taylor-otwell-thin-controllers-fat-models-approach
- Clean Code in Laravel: Organizing Your Application — https://github.com/ahmadmayahi/clean-code-in-laravel/blob/main/02-write-clean/004-organizing-your-application.md
- Laravel Daily: Laravel Service Classes — https://laraveldaily.com/post/laravel-service-classes

---

## 3. Delegating Business Logic

### Definitions

**Core Definition:** Delegating business logic is the practice of moving complex operations—such as calculations, multi-step workflows, external API calls, and event dispatching—out of controllers and into dedicated classes (services, actions, or model methods).

**Technical Definition:** Delegation in Laravel controller design means that controllers do not implement business rules. Instead, they pass validated data to a service class, action class, or model method, which encapsulates the logic and returns a result. The controller then transforms the result into a response. This follows the principle of "thin controllers, fat services/models"—the controller owns HTTP concerns, while the service or model owns domain logic. The delegation can be to a service class (for reusable, multi-method logic), an action class (for single-purpose operations), or a model method (for logic that naturally belongs to the model).

**Beginner-Friendly Explanation:** If a controller is a waiter, delegating business logic means the waiter doesn't cook the food—they pass the order to the kitchen. The kitchen (service or model) knows how to prepare the dish. The waiter just takes the order and delivers the result.

### Purposes

- To keep controllers focused on HTTP orchestration, not implementation details.
- To make business logic reusable across controllers, queue jobs, CLI commands, and tests.
- To improve testability by isolating business rules in testable units.
- To reduce the risk of code duplication and inconsistencies.
- To allow business logic to be changed without modifying the HTTP layer.
- To follow the Single Responsibility Principle by giving each class one reason to change.

### Syntax Rules and Structure

**Complete General Syntax — Delegation to a Service:**

```php
// Controller
public function store(StorePostRequest $request)
{
    $post = $this->postService->create($request->validated());
    return new PostResource($post);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$this->postService` | The injected service dependency. |
| `->create(...)` | The service method that encapsulates the business logic. |
| `$request->validated()` | Validated data passed to the service. |

**Complete General Syntax — Delegation to an Action:**

```php
// Controller
public function store(StorePostRequest $request, CreatePostAction $action)
{
    $post = $action->execute($request->validated());
    return new PostResource($post);
}
```

**Complete General Syntax — Delegation to a Model Method:**

```php
// Controller
public function suspend(SuspendUserRequest $request, User $user)
{
    $user->suspend($request->validated('reason'));
    return response()->noContent();
}
```

**Syntax Rules:**

- The controller should pass **validated data** (not raw request data) to the service or action.
- Services should accept plain arrays or DTOs, **not** the `Request` object, to remain decoupled from HTTP.
- Action classes should have a single public method (typically `execute` or `__invoke`).
- Model methods should be well-named and readable (e.g., `$user->suspend($reason)`).
- The controller should not perform any database queries directly—delegate to the service or model.

**Constraints and Limitations:**

- **Service vs. Action:** Services are for reusable, multi-method logic; actions are for single-purpose operations. Choose based on the scope of the operation.
- **Request coupling:** Passing the entire `Request` object to a service couples the service to HTTP, making it harder to use from CLI or queue jobs.
- **Over-delegation:** Delegating every single line of code to a separate class can lead to an explosion of tiny classes. Group related logic into cohesive services.

### Multiple Annotated Complete Code Examples

**Example 1: Delegating to a Service Class**

```php
<?php
// app/Services/PostService.php

namespace App\Services;

use App\Models\Post;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Mail;

class PostService
{
    /**
     * Create a new post with tags, cache invalidation, and notification.
     */
    public function create(array $data): Post
    {
        // Business logic: create the post
        $post = Post::create([
            'title' => $data['title'],
            'body' => $data['body'],
            'slug' => \Str::slug($data['title']),
            'user_id' => auth()->id(),
        ]);

        // Business logic: sync tags
        if (!empty($data['tags'])) {
            $post->tags()->sync($data['tags']);
        }

        // Business logic: invalidate cache
        Cache::forget('posts.all');

        // Business logic: notify subscribers
        Mail::to($post->user)->send(new \App\Mail\PostPublished($post));

        return $post;
    }
}
```

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Http\Resources\PostResource;
use App\Services\PostService;

class PostController extends Controller
{
    public function __construct(
        private PostService $postService
    ) {}

    public function store(StorePostRequest $request)
    {
        // Delegate all business logic to the service
        $post = $this->postService->create($request->validated());

        // Return the transformed response
        return new PostResource($post);
    }
}
```

**Expected Output:** The controller delegates the entire post creation workflow—including slug generation, tag syncing, cache invalidation, and email notification—to the `PostService`. The controller is two lines.

**Why:** The service encapsulates a reusable business operation. The controller is thin and focused on HTTP concerns. The service can be reused from a CLI command or queue job.

---

**Example 2: Delegating to an Action Class**

```php
<?php
// app/Actions/ProcessPaymentAction.php

namespace App\Actions;

use App\Models\Order;
use App\Services\PaymentGateway;

class ProcessPaymentAction
{
    public function __construct(
        private PaymentGateway $gateway
    ) {}

    public function execute(Order $order, string $token): bool
    {
        // Business logic: charge the payment
        $result = $this->gateway->charge($order->total, $token);

        if ($result->success) {
            $order->update([
                'payment_id' => $result->transactionId,
                'paid_at' => now(),
            ]);
            return true;
        }

        return false;
    }
}
```

```php
<?php
// app/Http/Controllers/PaymentController.php

namespace App\Http\Controllers;

use App\Actions\ProcessPaymentAction;
use App\Http\Requests\ProcessPaymentRequest;
use App\Models\Order;

class PaymentController extends Controller
{
    public function store(
        ProcessPaymentRequest $request,
        Order $order,
        ProcessPaymentAction $action
    ) {
        $success = $action->execute($order, $request->validated('token'));

        if (!$success) {
            return response()->json(['error' => 'Payment failed'], 422);
        }

        return response()->json(['status' => 'paid']);
    }
}
```

**Expected Output:** The controller delegates the payment processing to the `ProcessPaymentAction`. The action encapsulates the gateway interaction and order update.

**Why:** Action classes are ideal for single-purpose operations like payment processing. They are easy to test (mock the gateway) and easy to reuse from a queue job or webhook handler.

---

**Example 3: Delegating to a Model Method**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    /**
     * Suspend the user account.
     */
    public function suspend(string $reason): void
    {
        $this->update([
            'suspended_at' => now(),
            'suspension_reason' => $reason,
        ]);

        // Revoke all API tokens
        $this->tokens()->delete();

        // Dispatch an event
        event(new \App\Events\UserSuspended($this));
    }

    /**
     * Check if the user is suspended.
     */
    public function isSuspended(): bool
    {
        return $this->suspended_at !== null;
    }
}
```

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Http\Requests\SuspendUserRequest;
use App\Models\User;

class UserController extends Controller
{
    public function suspend(SuspendUserRequest $request, User $user)
    {
        // Delegate to the model method
        $user->suspend($request->validated('reason'));

        return response()->noContent();
    }
}
```

**Expected Output:** The controller calls `$user->suspend($reason)`, which encapsulates the suspension logic. The controller method is two lines.

**Why:** The model method provides a fluent, readable interface. The logic naturally belongs to the `User` model. This follows Taylor Otwell's "fat model" approach, where models expose rich public APIs.

### Real-World Cases

- **Order Processing:** `OrderController` delegates to `OrderService` for creating orders, processing payments, and sending confirmations.
- **User Registration:** `RegisterController` delegates to `RegisterUserAction` for creating users, sending welcome emails, and setting up profiles.
- **Content Publishing:** `PostController` delegates to `PostService` for creating posts, syncing tags, and invalidating caches.
- **Subscription Management:** `SubscriptionController` delegates to `SubscriptionService` for upgrading, downgrading, and cancelling subscriptions.

### References

- Clean Code in Laravel: Organizing Your Application — https://github.com/ahmadmayahi/clean-code-in-laravel/blob/main/02-write-clean/004-organizing-your-application.md
- Laravel Daily: Service Classes — https://laraveldaily.com/post/laravel-service-classes
- Laravel Daily: Actions vs Services — https://laraveldaily.com/post/laravel-actions-services

---

## 4. Service Classes

### Definitions

**Core Definition:** A service class is a stateless, reusable class that encapsulates a specific capability or set of related business operations, providing a clean API for controllers, actions, jobs, and other parts of the application to use.

**Technical Definition:** A service class in Laravel is a plain PHP class (typically stored in `app/Services`) that contains business logic and is resolved through the service container. Services are stateless—they do not store request-specific data between method calls. They accept plain data (arrays, DTOs, or models) and return results (models, DTOs, booleans, or void). Services are distinct from actions: an action is something the application does (a single operation), while a service is something the application uses (a reusable capability). A service can have multiple public methods, while an action typically has one.

**Beginner-Friendly Explanation:** A service class is like a toolbox. It contains tools (methods) for doing specific things—like calculating taxes, formatting money, or processing payments. Any part of your application can grab the toolbox and use its tools. The toolbox doesn't remember what you used it for last time (it's stateless), and it doesn't care who's using it. It just does its job when asked.

### Purposes

- To encapsulate complex business logic in a reusable, testable class.
- To keep controllers thin by moving business operations out of the HTTP layer.
- To provide a single source of truth for a specific capability (e.g., payment processing, tax calculation).
- To allow business logic to be reused across controllers, actions, jobs, and CLI commands.
- To improve testability by isolating business logic from HTTP concerns.
- To provide a clean, well-named interface for domain operations.

### Syntax Rules and Structure

**Complete General Syntax — Basic Service Class:**

```php
<?php

namespace App\Services;

class TaxCalculator
{
    public function calculate(int $subtotalInCents, string $country): int
    {
        $rate = match ($country) {
            'NL' => 0.21,
            'DE' => 0.19,
            'FR' => 0.20,
            default => 0.0,
        };

        return (int) round($subtotalInCents * $rate);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `namespace App\Services;` | Services are typically stored in `app/Services`. |
| `class TaxCalculator` | A well-named class that describes its capability. |
| `public function calculate(...)` | A public method that performs the service's job. |
| No constructor | Services are often stateless; no state to initialize. |

**Complete General Syntax — Service with Dependencies:**

```php
<?php

namespace App\Services;

use App\Repositories\OrderRepositoryInterface;
use App\Services\PaymentGateway;

class OrderService
{
    public function __construct(
        private OrderRepositoryInterface $orders,
        private PaymentGateway $gateway
    ) {}

    public function createOrder(int $userId, array $items): Order
    {
        $order = $this->orders->create(['user_id' => $userId]);
        $total = 0;

        foreach ($items as $item) {
            // ... business logic
        }

        $this->gateway->charge($total, $order->payment_token);

        return $order;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `__construct(...)` | Services declare their dependencies via constructor injection. |
| `private OrderRepositoryInterface $orders` | The repository dependency (PHP 8 property promotion). |
| `private PaymentGateway $gateway` | The payment gateway dependency. |

**Syntax Rules:**

- Services should be **stateless**: no request-specific data stored as properties.
- Services should accept **plain data** (arrays, DTOs, models), not the `Request` object.
- Services should return **results** (models, DTOs, booleans, void), not HTTP responses.
- Services should have **focused interfaces**: well-named methods that describe what they do (e.g., `calculateTax`, `formatMoney`, `processPayment`).
- Services should be **unit-testable in isolation**: dependencies are injected, not instantiated.
- Whether to use a `Service` suffix is a team decision—a well-named class like `TaxCalculator` may not need it.

**Constraints and Limitations:**

- **Statelessness:** Services should not hold request state. If you need to pass data between methods, use a DTO or pass parameters.
- **Request coupling:** Avoid injecting or passing the `Request` object to services—extract the relevant data and pass it as an array or DTO.
- **Service explosion:** Avoid creating a service for every tiny operation. Group related operations into cohesive services.
- **Naming:** Services should be named after their capability (e.g., `PaymentService`, `TaxCalculator`), not after the entity they operate on (e.g., `UserService` may become a catch-all).

### Multiple Annotated Complete Code Examples

**Example 1: Basic Stateless Service**

```php
<?php
// app/Services/MoneyFormatter.php

namespace App\Services;

class MoneyFormatter
{
    /**
     * Format a money value in cents to a human-readable string.
     */
    public function format(int $cents, string $currency = 'USD'): string
    {
        return match ($currency) {
            'USD' => '$' . number_format($cents / 100, 2),
            'EUR' => number_format($cents / 100, 2, ',', '.') . ' €',
            default => number_format($cents / 100, 2) . ' ' . $currency,
        };
    }

    /**
     * Format a money value in a compact form (e.g., $1.2k).
     */
    public function formatCompact(int $cents, string $currency = 'USD'): string
    {
        if ($cents >= 100_00) {
            return match ($currency) {
                'USD' => '$' . round($cents / 100_00, 1) . 'k',
                default => round($cents / 100_00, 1) . 'k ' . $currency,
            };
        }

        return $this->format($cents, $currency);
    }
}
```

```php
<?php
// app/Http/Controllers/InvoiceController.php

namespace App\Http\Controllers;

use App\Services\MoneyFormatter;

class InvoiceController extends Controller
{
    public function __construct(
        private MoneyFormatter $formatter
    ) {}

    public function show(Invoice $invoice)
    {
        return view('invoices.show', [
            'total' => $this->formatter->format($invoice->total_in_cents),
            'compact' => $this->formatter->formatCompact($invoice->total_in_cents),
        ]);
    }
}
```

**Expected Output:** The `MoneyFormatter` service is stateless and reusable. The controller injects it and uses its methods to format money values.

**Why:** The service encapsulates the formatting logic. It has no state, no side effects, and can be injected anywhere. The controller is thin and delegates the formatting work.

---

**Example 2: Service with Dependencies (Order Processing)**

```php
<?php
// app/Services/OrderService.php

namespace App\Services;

use App\Mail\OrderConfirmation;
use App\Models\Order;
use App\Models\Product;
use Illuminate\Support\Facades\Mail;

class OrderService
{
    public function __construct(
        private PaymentGateway $gateway
    ) {}

    /**
     * Create an order, process payment, and send confirmation.
     */
    public function createOrder(int $userId, array $items, string $paymentToken): Order
    {
        // Create the order
        $order = Order::create([
            'user_id' => $userId,
            'status' => 'pending',
        ]);

        // Add items and calculate total
        $total = 0;
        foreach ($items as $item) {
            $product = Product::findOrFail($item['product_id']);
            $order->items()->create([
                'product_id' => $product->id,
                'quantity' => $item['quantity'],
                'price' => $product->price,
            ]);
            $total += $product->price * $item['quantity'];
            $product->decrement('stock', $item['quantity']);
        }

        $order->update(['total' => $total]);

        // Process payment
        $result = $this->gateway->charge($total, $paymentToken);
        $order->update([
            'payment_id' => $result->transactionId,
            'status' => $result->success ? 'paid' : 'failed',
        ]);

        // Send confirmation if payment succeeded
        if ($result->success) {
            Mail::to($order->user)->send(new OrderConfirmation($order));
        }

        return $order;
    }
}
```

```php
<?php
// app/Http/Controllers/OrderController.php

namespace App\Http\Controllers;

use App\Http\Requests\StoreOrderRequest;
use App\Services\OrderService;

class OrderController extends Controller
{
    public function __construct(
        private OrderService $orderService
    ) {}

    public function store(StoreOrderRequest $request)
    {
        $order = $this->orderService->createOrder(
            auth()->id(),
            $request->validated('items'),
            $request->validated('payment_token')
        );

        return response()->json($order, 201);
    }
}
```

**Expected Output:** The `OrderService` encapsulates the entire order creation workflow—creating the order, adding items, calculating totals, processing payment, and sending confirmation. The controller is two lines.

**Why:** The service is stateless (dependencies are injected, no request state stored). It accepts plain data (user ID, items array, payment token) and returns an `Order` model. The controller remains thin and focused on HTTP concerns.

---

**Example 3: Service for External API Integration**

```php
<?php
// app/Services/StripePaymentGateway.php

namespace App\Services;

use Stripe\StripeClient;

class StripePaymentGateway implements PaymentGateway
{
    private StripeClient $stripe;

    public function __construct(string $secretKey)
    {
        $this->stripe = new StripeClient($secretKey);
    }

    public function charge(int $amountInCents, string $token): PaymentResult
    {
        try {
            $charge = $this->stripe->charges->create([
                'amount' => $amountInCents,
                'currency' => 'usd',
                'source' => $token,
            ]);

            return new PaymentResult(
                success: true,
                transactionId: $charge->id
            );
        } catch (\Stripe\Exception\CardException $e) {
            return new PaymentResult(
                success: false,
                error: $e->getMessage()
            );
        }
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\PaymentGateway;
use App\Services\StripePaymentGateway;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(PaymentGateway::class, function ($app) {
            return new StripePaymentGateway(
                config('services.stripe.secret')
            );
        });
    }
}
```

**Expected Output:** The `StripePaymentGateway` service wraps the Stripe API. It implements the `PaymentGateway` interface, allowing it to be swapped for a different gateway (e.g., PayPal) without changing the consuming code.

**Why:** This is a common use case for services: wrapping third-party APIs. The service is stateless (the Stripe client is configured once), has a focused interface (`charge`), and can be injected anywhere via the container.

### Real-World Cases

- **Payment Processing:** `StripePaymentGateway`, `PayPalPaymentGateway` services wrap payment provider APIs.
- **Tax Calculation:** `TaxCalculator` service computes taxes based on country and product type.
- **Shipping Rate Calculation:** `ShippingRateCalculator` service fetches rates from carriers.
- **Notification Services:** `NotificationService` sends emails, SMS, and push notifications.
- **Third-Party API Wrappers:** `GitHubService`, `SlackService`, `AWS S3Service`—services that wrap external APIs.

### References

- Clean Code in Laravel: Organizing Your Application — https://github.com/ahmadmayahi/clean-code-in-laravel/blob/main/02-write-clean/004-organizing-your-application.md
- Laravel Daily: Service Classes — https://laraveldaily.com/post/laravel-service-classes
- Laravel Daily: Services vs Actions — https://laraveldaily.com/post/laravel-services-vs-actions
- Laravel Service Container — https://laravel.com/docs/master/container

---

## 5. Request Validation

### Definitions

**Core Definition:** Request validation in Laravel is the process of verifying that incoming HTTP request data meets a set of defined rules before it is used by the application, with Form Request classes providing a dedicated, reusable location for validation logic.

**Technical Definition:** Laravel provides multiple validation approaches: inline `$request->validate()` in controllers, the `Validator` facade for manual validation, and Form Request classes for complex or reusable validation scenarios. A Form Request is a custom class that extends `Illuminate\Foundation\Http\FormRequest` and defines `rules()` (validation rules), `authorize()` (authorization logic), and optionally `messages()` (custom error messages) and `attributes()` (custom attribute names). When a Form Request is type-hinted in a controller method, Laravel automatically validates the request before the controller method executes. If validation fails, the user is redirected back with errors (for web requests) or a JSON error response is returned (for API requests).

**Beginner-Friendly Explanation:** Validation is like a bouncer at a club. Before anyone gets in, the bouncer checks their ID and makes sure they meet the requirements. Form Requests are like having a dedicated bouncer for each entrance—you define the rules once, and the bouncer enforces them automatically. If someone doesn't meet the rules, they're turned away with a clear message.

### Purposes

- To ensure that incoming data meets the application's requirements before it is processed.
- To keep controllers thin by moving validation rules out of controller methods.
- To centralise validation logic for a given endpoint, making it reusable and testable.
- To provide automatic, consistent error responses for both web and API requests.
- To separate authorization logic from validation logic using the `authorize()` method.
- To allow custom error messages and attribute names for better user experience.

### Syntax Rules and Structure

**Complete General Syntax — Form Request Class:**

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     */
    public function rules(): array
    {
        return [
            'title' => 'required|string|min:3|max:255',
            'body' => 'required|string',
            'tags' => 'nullable|array',
            'tags.*' => 'exists:tags,id',
        ];
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends FormRequest` | The Form Request base class. |
| `authorize()` | Returns whether the user is allowed to make the request. |
| `rules()` | Returns an array of validation rules. |
| `'title' => 'required\|string\|min:3\|max:255'` | Validation rules for the `title` field. |

**Complete General Syntax — Using a Form Request in a Controller:**

```php
use App\Http\Requests\StorePostRequest;

public function store(StorePostRequest $request)
{
    // Validation has already passed
    $validated = $request->validated();
    // ...
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `StorePostRequest $request` | Type-hint the Form Request—Laravel validates automatically. |
| `$request->validated()` | Returns only the fields that passed validation. |

**Complete General Syntax — Custom Error Messages:**

```php
public function messages(): array
{
    return [
        'title.required' => 'A title is required.',
        'body.required' => 'A message is required.',
    ];
}
```

**Complete General Syntax — Custom Attribute Names:**

```php
public function attributes(): array
{
    return [
        'email' => 'email address',
    ];
}
```

**Complete General Syntax — Preparing Input for Validation:**

```php
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => \Str::slug($this->title),
    ]);
}
```

**Syntax Rules:**

- Form Requests are generated via `php artisan make:request StorePostRequest`.
- The `authorize()` method must return `true` for the validation to proceed; returning `false` results in a 403 response.
- The `rules()` method returns an array of `field => rules` pairs. Rules can be strings (`'required|string'`) or arrays (`['required', 'string']`).
- Use `$request->validated()` instead of `$request->all()` to ensure only validated fields are used.
- Custom messages are defined in the `messages()` method; custom attribute names in `attributes()`.
- The `prepareForValidation()` method allows sanitising input before validation (e.g., slug generation).

**Constraints and Limitations:**

- **Form Request proliferation:** Creating a Form Request for every tiny form can lead to many small classes. For simple forms, inline validation may be acceptable.
- **`authorize()` requirement:** In older Laravel versions, `authorize()` was required; in newer versions, it can be omitted (defaults to `true`).
- **Array validation:** Validating nested arrays requires careful use of `*` wildcards (e.g., `'items.*.product_id'`).
- **Performance:** Form Requests add a slight overhead due to class resolution, though this is negligible.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Form Request**

```php
<?php
// app/Http/Requests/StorePostRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|min:3|max:255',
            'body' => 'required|string',
            'published_at' => 'nullable|date|after:now',
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'A title is required.',
            'title.min' => 'The title must be at least 3 characters.',
            'body.required' => 'A message is required.',
            'published_at.after' => 'The publish date must be in the future.',
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Models\Post;

class PostController extends Controller
{
    public function store(StorePostRequest $request)
    {
        // Validation has already passed
        $post = Post::create($request->validated());

        return redirect()->route('posts.show', $post);
    }
}
```

**Expected Output:**
- If validation passes: the post is created and the user is redirected.
- If validation fails: the user is redirected back with errors and old input.
- For API requests: a 422 JSON response is returned with error messages.

**Why:** The Form Request encapsulates all validation logic. The controller is thin and only handles the successful case. The custom messages provide user-friendly error feedback.

---

**Example 2: Form Request with Authorization**

```php
<?php
// app/Http/Requests/UpdatePostRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UpdatePostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to update this post.
     */
    public function authorize(): bool
    {
        // Only the post's author can update it
        return $this->user()->can('update', $this->route('post'));
    }

    public function rules(): array
    {
        return [
            'title' => 'sometimes|string|min:3|max:255',
            'body' => 'sometimes|string',
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/PostController.php

namespace App\Http\Controllers;

use App\Http\Requests\UpdatePostRequest;
use App\Models\Post;

class PostController extends Controller
{
    public function update(UpdatePostRequest $request, Post $post)
    {
        $post->update($request->validated());

        return new \App\Http\Resources\PostResource($post);
    }
}
```

**Expected Output:**
- If the user is the post's author: validation passes and the post is updated.
- If the user is not the author: a 403 Forbidden response is returned.

**Why:** The `authorize()` method integrates with Laravel's authorization system (Policies and Gates). The controller does not need to check authorization—the Form Request handles it before the controller method executes.

---

**Example 3: Form Request with `prepareForValidation`**

```php
<?php
// app/Http/Requests/StorePostRequest.php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Support\Str;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    protected function prepareForValidation(): void
    {
        // Automatically generate a slug from the title
        $this->merge([
            'slug' => Str::slug($this->title),
        ]);
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|min:3|max:255',
            'slug' => 'required|string|unique:posts,slug',
            'body' => 'required|string',
        ];
    }
}
```

**Expected Output:** The `slug` is automatically generated from the `title` before validation. If the generated slug already exists, validation fails with a unique constraint error.

**Why:** The `prepareForValidation()` method allows input sanitisation before validation rules are applied. This is useful for generating slugs, trimming whitespace, or normalising data.

### Real-World Cases

- **User Registration:** `RegisterRequest` validates name, email, password, and password confirmation.
- **Order Creation:** `StoreOrderRequest` validates items array, quantities, and payment token.
- **Profile Updates:** `UpdateProfileRequest` validates name, email, and avatar upload.
- **API Endpoints:** `StorePostRequest` validates JSON payloads for API consumers.
- **Admin Panels:** `UpdateUserRequest` validates user data with authorization checks.

### References

- Laravel Validation Documentation — https://laravel.com/framework/docs/master/validation
- Laravel Form Request Validation — https://laravel.com/docs/12.x/validation#form-request-validation
- Krayin CRM Developer Portal: Validation — https://devdocs.krayincrm.com/2.2/packages/validation.html

---

## 6. Response Transformation

### Definitions

**Core Definition:** Response transformation is the process of converting raw data (typically Eloquent models or collections) into a structured JSON or view response, ensuring that only the intended fields are exposed and that the output format is consistent across all endpoints.

**Technical Definition:** Laravel provides Eloquent API Resources (classes extending `Illuminate\Http\Resources\Json\JsonResource`) as a transformation layer between models and JSON responses. A resource class defines a `toArray()` method that returns the array of attributes to be serialised. Resources can transform individual models or collections (`Resource::collection()`). They support conditional attributes (`when()`, `whenHas()`), relationships (`whenLoaded()`), pagination, and metadata. When a resource is returned from a controller, Laravel automatically converts it to a `JsonResponse` with a `data` wrapper by default. This provides granular control over the JSON serialization of models and their relationships, hiding sensitive attributes and formatting values consistently.

**Beginner-Friendly Explanation:** When you return a model directly from a controller, Laravel dumps all its database columns into JSON—including sensitive ones like passwords. Response transformation is like putting a filter on the output. You decide exactly which fields are included, how dates are formatted, and what relationships are embedded. API Resources are Laravel's built-in tool for this filtering.

### Purposes

- To shape JSON responses consistently across all API endpoints.
- To hide sensitive attributes (passwords, tokens, internal IDs) from API consumers.
- To format dates, numbers, and relationships into a consistent structure.
- To include only the relationships that are loaded (avoiding N+1 query leaks).
- To add computed attributes (e.g., `full_name`, `excerpt`) without modifying the model.
- To wrap responses in a consistent `data` envelope for easier client-side parsing.

### Syntax Rules and Structure

**Complete General Syntax — API Resource Class:**

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'excerpt' => \Str::limit($this->body, 100),
            'author' => new UserResource($this->whenLoaded('user')),
            'created_at' => $this->created_at->toDateString(),
        ];
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends JsonResource` | The base class for API resources. |
| `toArray(Request $request)` | Returns the array of attributes to serialise. |
| `$this->id` | Accesses the underlying model's attributes. |
| `whenLoaded('user')` | Conditionally includes a relationship if it's loaded. |
| `\Str::limit(...)` | Computes a truncated excerpt. |

**Complete General Syntax — Using a Resource in a Controller:**

```php
use App\Http\Resources\PostResource;
use App\Models\Post;

public function show(Post $post)
{
    return new PostResource($post);
}

public function index()
{
    return PostResource::collection(Post::with('user')->paginate(20));
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `new PostResource($post)` | Transforms a single model. |
| `PostResource::collection(...)` | Transforms a collection of models. |

**Complete General Syntax — Resource Collection:**

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class PostCollection extends ResourceCollection
{
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'meta' => [
                'total' => $this->collection->count(),
            ],
        ];
    }
}
```

**Syntax Rules:**

- API Resources are generated via `php artisan make:resource PostResource`.
- Resources are stored in `app/Http/Resources` by default.
- The `toArray()` method receives the current `Request` instance for conditional logic.
- `whenLoaded()` conditionally includes a relationship only if it has been eager-loaded.
- `when()` conditionally includes an attribute based on a boolean condition.
- Collection resources are generated via `php artisan make:resource PostCollection --collection`.
- By default, resources wrap the output in a `data` key. This can be disabled via `JsonResource::withoutWrapping()`.

**Constraints and Limitations:**

- **`data` wrapper:** API Resources automatically wrap responses in a `data` key. Client-side code must access `response.data.data` for the actual payload.
- **Performance:** Resources add a transformation layer. For simple endpoints, returning the model directly may be sufficient.
- **Relationship loading:** `whenLoaded()` only includes a relationship if it's already loaded. You must eager-load relationships in the controller to avoid N+1 queries.
- **Nested resources:** Transforming nested relationships can become complex. Consider using separate resource classes for each model.

### Multiple Annotated Complete Code Examples

**Example 1: Basic API Resource**

```php
<?php
// app/Http/Resources/PostResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'excerpt' => \Str::limit($this->body, 100),
            'created_at' => $this->created_at->toDateString(),
            'updated_at' => $this->updated_at->toDateString(),
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\PostResource;
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        return PostResource::collection(Post::all());
    }

    public function show(Post $post)
    {
        return new PostResource($post);
    }
}
```

**Expected Output:**
- `GET /api/posts` → `{"data": [{"id": 1, "title": "...", "excerpt": "...", "created_at": "2026-01-15"}, ...]}`
- `GET /api/posts/1` → `{"data": {"id": 1, "title": "...", ...}}`

**Why:** The `PostResource` controls exactly which fields are exposed. The `excerpt` is computed from the `body` (truncated to 100 characters). The `created_at` and `updated_at` dates are formatted as date strings instead of full timestamps. The response is wrapped in a `data` key.

---

**Example 2: Resource with Conditional Relationships**

```php
<?php
// app/Http/Resources/PostResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'excerpt' => \Str::limit($this->body, 100),
            // Include author only if the user relationship is loaded
            'author' => new UserResource($this->whenLoaded('user')),
            // Include comments count only if the comments relationship is loaded
            'comments_count' => $this->whenLoaded('comments', fn() => $this->comments->count()),
            // Include tags only if the tags relationship is loaded
            'tags' => TagResource::collection($this->whenLoaded('tags')),
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\PostResource;
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        // Eager-load relationships to include them in the response
        $posts = Post::with(['user', 'tags'])->paginate(20);
        return PostResource::collection($posts);
    }

    public function show(Post $post)
    {
        // Load relationships conditionally
        $post->load(['user', 'comments', 'tags']);
        return new PostResource($post);
    }
}
```

**Expected Output:**
- `GET /api/posts` → includes `author` and `tags` (because they are eager-loaded), but not `comments_count` (not loaded).
- `GET /api/posts/1` → includes `author`, `comments_count`, and `tags` (all loaded).

**Why:** The `whenLoaded()` method conditionally includes relationships based on whether they have been eager-loaded. This prevents N+1 query problems and ensures that only the necessary data is included in each response.

---

**Example 3: Resource Collection with Metadata**

```php
<?php
// app/Http/Resources/PostCollection.php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class PostCollection extends ResourceCollection
{
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'meta' => [
                'total' => $this->collection->count(),
                'api_version' => '1.0',
            ],
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Resources\PostCollection;
use App\Models\Post;

class PostController extends Controller
{
    public function index()
    {
        return new PostCollection(Post::paginate(20));
    }
}
```

**Expected Output:**
```json
{
    "data": [...],
    "meta": {
        "total": 42,
        "api_version": "1.0"
    },
    "links": {
        "first": "...",
        "last": "...",
        "prev": null,
        "next": "..."
    }
}
```

**Why:** The `PostCollection` resource adds custom metadata to the response. Pagination links are automatically added when paginated data is passed to a resource collection.

### Real-World Cases

- **API Development:** API Resources are the standard way to shape JSON responses in Laravel APIs.
- **Mobile App Backends:** Resources ensure consistent data structures for iOS and Android clients.
- **SPA Backends:** Vue and React front-ends consume resource-transformed data with the `data` wrapper.
- **Multi-Version APIs:** Different resource classes for `v1` and `v2` allow API versioning without changing models.
- **Public vs. Admin APIs:** Different resources can expose different fields for public and admin consumers.

### References

- Laravel Eloquent: API Resources — https://laravel.com/framework/docs/13.x/eloquent-resources
- Laravel Daily: Eloquent API Resources to Transform Fields — https://laraveldaily.com/lesson/vue-laravel-vite-spa-crud/eloquent-api-resources-transform-fields
- Laravel News: Streamline API Resources with Fluent Methods — https://laravel-news.com/streamline-api-resources-with-laravel-fluent-methods

---

## References

- Laravel: What is MVC? — https://laravel.com/learn/getting-started-with-laravel/what-is-mvc
- Laravel Service Container — https://laravel.com/docs/master/container
- Laravel Validation Documentation — https://laravel.com/framework/docs/master/validation
- Laravel Eloquent: API Resources — https://laravel.com/framework/docs/13.x/eloquent-resources
- Taylor Otwell: "Thin" Controllers, "Fat" Models Approach — https://laraveldaily.com/post/taylor-otwell-thin-controllers-fat-models-approach
- Clean Code in Laravel: Organizing Your Application — https://github.com/ahmadmayahi/clean-code-in-laravel/blob/main/02-write-clean/004-organizing-your-application.md
- Krayin CRM Developer Portal: Validation — https://devdocs.krayincrm.com/2.2/packages/validation.html
- Laravel Daily: Eloquent API Resources to Transform Fields — https://laraveldaily.com/lesson/vue-laravel-vite-spa-crud/eloquent-api-resources-transform-fields
- Laravel Daily: Service Classes — https://laraveldaily.com/post/laravel-service-classes
- Laravel Daily: Services vs Actions — https://laraveldaily.com/post/laravel-services-vs-actions
- Laravel News: Streamline API Resources with Fluent Methods — https://laravel-news.com/streamline-api-resources-with-laravel-fluent-methods