# Laravel Dependency Injection: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Dependency Injection (DI) is a design pattern in which an object receives its dependencies from an external source rather than creating them internally, and Laravel implements this pattern through its Service Container to automatically resolve and inject class dependencies.

**Technical Definition:** Dependency Injection is a form of Inversion of Control (IoC) where the responsibility of instantiating and providing dependencies is inverted from the consuming class to an external injector—in Laravel's case, the Service Container (`Illuminate\Container\Container`). The container uses PHP's Reflection API to inspect constructor and method signatures, resolves type-hinted dependencies recursively, and injects them at runtime. Laravel supports constructor injection (via `__construct`), method injection (via controller actions, job `handle` methods, and closures), and interface-based injection (by binding abstractions to concrete implementations in service providers).

**Beginner-Friendly Explanation:** Imagine you're building a car. Instead of forging your own engine, tires, and radio from scratch, you order them from suppliers and bolt them in. Dependency Injection is the principle that your car (class) shouldn't build its own parts (dependencies); it should receive them from a factory (the Service Container). This makes the car easier to test (swap in a fake engine), easier to change (switch from petrol to electric), and less coupled to any single supplier.

### Key Characteristics

1. **Automatic Resolution:** The container inspects type-hints via PHP Reflection and resolves dependencies without configuration.
2. **Constructor Injection:** Dependencies are declared in the constructor, the primary and recommended injection method.
3. **Method Injection:** Dependencies can be injected directly into controller actions, job handlers, and closures.
4. **Interface-Based Decoupling:** Classes depend on interfaces, and the container binds interfaces to concrete implementations.
5. **Dependency Inversion:** Follows the SOLID "D" principle—high-level modules depend on abstractions, not concretions.
6. **Zero-Configuration Default:** Concrete classes require no explicit bindings; the container resolves them automatically.
7. **Testability:** Dependencies can be swapped with mocks or stubs without modifying the consuming class.
8. **Recursive Resolution:** The container builds the entire dependency graph, resolving sub-dependencies of dependencies.

### Prerequisites

- Laravel 9.x or higher (12.x recommended)
- PHP 8.0 or higher
- Composer package manager
- Familiarity with PHP type-hinting, interfaces, and object-oriented programming
- Understanding of Laravel Service Providers (the registration point for bindings)
- Basic knowledge of the Inversion of Control pattern

### Related Programming Areas

- **Service Container:** The Laravel implementation of the DI container.
- **SOLID Principles:** Dependency Inversion is the "D" in SOLID.
- **Service Providers:** Where interface bindings are registered.
- **PHP Reflection API:** The mechanism enabling automatic resolution.
- **Testing and Mocking:** DI enables easy test doubles.
- **Repository Pattern:** A common architectural pattern that leverages DI.
- **Facades:** Laravel's static interface to container-managed services.

### Core Concepts / Features

1. Constructor Injection (Declaring dependencies in object class constructors for automatic instantiation)
2. Method Injection (Type-hinting dependencies directly in Controller actions, Jobs, or Console commands)
3. Interface-Based Dependencies (Decoupling code by binding an abstract interface to a concrete class implementation)
4. Dependency Inversion (Adhering to SOLID principles by depending upon abstractions rather than concretions)
5. Enhanced: Container Call Invocation (Using `$this->app->call()` to execute any PHP callable while automatically injecting its dependencies at runtime)
6. Enhanced: Variadic Dependency Resolution (Injecting dynamic arrays of objects or configuration primitives into constructors using the container)


## 1. Constructor Injection

### Definitions

**Core Definition:** Constructor injection is a form of dependency injection in which a class declares its dependencies as parameters in its constructor, and the container automatically provides those dependencies when instantiating the class.

**Technical Definition:** Constructor injection is the primary DI mechanism in Laravel. When the container resolves a class, it uses `ReflectionClass` to inspect the constructor's parameters. For each parameter with a class type-hint, the container recursively resolves that dependency. For parameters with default values, it uses the default. For primitive parameters without defaults, it throws a `BindingResolutionException` unless parameters are supplied via `makeWith()`. This mechanism applies to controllers, event listeners, middleware, queued jobs, and any class resolved from the container.

**Beginner-Friendly Explanation:** Constructor injection is like ordering a custom sandwich. You tell the deli (container): "To make me, you need bread, cheese, and lettuce." The deli gathers those ingredients (dependencies) and hands you the finished sandwich. You never have to go to the store yourself—you just declare what you need, and it appears.

### Purposes

- To declare a class's dependencies explicitly in its constructor.
- To enable automatic dependency resolution without manual instantiation.
- To make dependencies visible and testable by exposing them in the constructor signature.
- To decouple a class from its dependencies, allowing implementations to be swapped.
- To follow the single responsibility principle by delegating object construction to the container.
- To enable mocking in tests by injecting test doubles through the constructor.
- To provide a standard, consistent pattern for dependency management across the application.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php

namespace App\Services;

use App\Contracts\PaymentGateway;
use App\Contracts\Logger;

class OrderService
{
    /**
     * Create a new OrderService instance.
     *
     * The container inspects these type-hints and injects
     * the resolved PaymentGateway and Logger automatically.
     */
    public function __construct(
        protected PaymentGateway $gateway,
        protected Logger $logger
    ) {}
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `__construct()` | The constructor method; the container inspects it |
| `PaymentGateway $gateway` | Type-hinted dependency; resolved from container |
| `Logger $logger` | Another type-hinted dependency |
| `protected` | Visibility modifier; stores the dependency |
| `ReflectionClass` | PHP API used by the container to inspect the constructor |

#### Syntax Rules

1. Dependencies **must** be type-hinted (class or interface names) for automatic resolution.
2. The container resolves dependencies **recursively**; a dependency's dependencies are also resolved.
3. Primitive parameters (string, int, bool) **must** have default values or be supplied via `makeWith()`.
4. Interfaces **must** be bound to concrete implementations in a Service Provider.
5. Constructor injection works automatically for controllers, event listeners, middleware, and queued jobs.
6. The container caches resolved instances for singletons; non-singletons are resolved fresh each time.

#### Constraints and Limitations

- **Primitive Dependencies:** The container cannot resolve primitive values without defaults; use `makeWith()` to supply them.
- **Circular Dependencies:** A → B → A causes infinite recursion; break with setter injection or `Lazy` resolution.
- **Non-Instantiable Types:** Abstract classes and interfaces require explicit bindings.
- **Performance:** Reflection has overhead; cache resolved classes in Octane or use compiled container.
- **No Setter Injection:** Constructor injection only inspects the constructor, not setter methods.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Basic Constructor Injection in a Controller

**Step-by-Step Setup Guide:**

1. Create a service class with no dependencies.
2. Type-hint it in a controller's constructor.
3. The container automatically resolves and injects it when the controller is instantiated.

**Complete Executable Code:**

```php
<?php
// app/Services/AppleMusic.php

namespace App\Services;

class AppleMusic
{
    public function findPodcast(string $id): array
    {
        return ['id' => $id, 'title' => 'Example Podcast'];
    }
}
```

```php
<?php
// app/Http/Controllers/PodcastController.php

namespace App\Http\Controllers;

use App\Services\AppleMusic;

class PodcastController extends Controller
{
    /**
     * The container inspects this constructor, sees AppleMusic,
     * resolves it automatically (no binding needed), and injects it.
     */
    public function __construct(
        protected AppleMusic $apple
    ) {}

    public function show(string $id)
    {
        return $this->apple->findPodcast($id);
    }
}
```

```php
// routes/web.php

Route::get('/podcasts/{id}', [PodcastController::class, 'show']);
```

**Expected Output:**

- `GET /podcasts/42` returns `{"id":"42","title":"Example Podcast"}`.
- The `PodcastController` receives an `AppleMusic` instance without any explicit binding.

**Why This Code Produces That Result:**

- The container resolves `PodcastController` and inspects its constructor.
- It sees the `AppleMusic` type-hint, resolves `AppleMusic` (which has no dependencies), and injects it.
- No binding was needed because `AppleMusic` is a concrete class with no interface dependencies.

#### Example 2: Constructor Injection with Interface Binding

```php
<?php
// app/Contracts/PaymentGateway.php

namespace App\Contracts;

interface PaymentGateway
{
    public function charge(float $amount): bool;
}
```

```php
<?php
// app/Services/StripeGateway.php

namespace App\Services;

use App\Contracts\PaymentGateway;

class StripeGateway implements PaymentGateway
{
    public function charge(float $amount): bool
    {
        // Stripe API call...
        return true;
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\PaymentGateway;
use App\Services\StripeGateway;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind the interface to the concrete implementation
        $this->app->bind(PaymentGateway::class, StripeGateway::class);
    }
}
```

```php
<?php
// app/Http/Controllers/CheckoutController.php

namespace App\Http\Controllers;

use App\Contracts\PaymentGateway;

class CheckoutController extends Controller
{
    public function __construct(
        protected PaymentGateway $gateway
    ) {}

    public function process()
    {
        return $this->gateway->charge(99.99);
    }
}
```

**Expected Output:**

- The container resolves `PaymentGateway` to `StripeGateway` and injects it.
- `process()` charges $99.99 via Stripe.

**Why This Code Produces That Result:**

- The `bind()` call in `AppServiceProvider` maps the interface to the concrete class.
- When `CheckoutController` is resolved, the container sees `PaymentGateway`, looks up the binding, and instantiates `StripeGateway`.
- Swapping to a different gateway requires only changing the binding—no controller code changes.

### Real-World Cases

**Case 1: E-Commerce Checkout**

An `OrderService` constructor-injects a `PaymentGateway` interface and a `Logger`. The container resolves the bound `StripeGateway` and `DatabaseLogger`.

**Case 2: Multi-Tenant SaaS**

A `TenantContext` service is constructor-injected into controllers and services. The container resolves it per-request (using scoped bindings).

**Case 3: API Client Wrapper**

An `ApiClient` service constructor-injects an `HttpClient` and a `Logger`. The container resolves both automatically.

### References

- Laravel Service Container: Automatic Injection - https://laravel.com/docs/12.x/container#automatic-injection
- Laravel Service Container: Constructor Injection - https://laravel.com/docs/12.x/container
- Laravel Service Container: Zero Configuration Resolution - https://laravel.com/docs/12.x/container#zero-configuration-resolution
- CodeMag: Dependency Injection and Service Container in Laravel - https://www.codemag.com/Article/2212041/Dependency-Injection-and-Service-Container-in-Laravel


## 2. Method Injection

### Definitions

**Core Definition:** Method injection is a form of dependency injection in which dependencies are type-hinted directly in a method's parameter list (typically controller actions, job `handle` methods, or closures), and the container resolves and injects them when the method is invoked.

**Technical Definition:** Laravel's container resolves method dependencies when invoking controller actions, queued job `handle` methods, event listener `handle` methods, middleware `handle` methods, and closures passed to `app()->call()`. The container inspects the method's parameters using Reflection, resolves type-hinted dependencies, and passes them as arguments. Primitive parameters (like route parameters) are matched by name or passed via the route. This mechanism is distinct from constructor injection, which resolves dependencies at object instantiation time.

**Beginner-Friendly Explanation:** Method injection is like a waiter bringing you a glass of water when you sit down at a restaurant—you didn't have to build the restaurant or hire the waiter; the water just appears when you need it. In Laravel, when a controller action runs, the container hands it the dependencies it declares in its method signature, like a `Request` object or a service.

### Purposes

- To inject dependencies into controller actions without polluting the constructor.
- To receive the current `Request` object in controller methods.
- To inject services into queued job `handle` methods.
- To inject dependencies into event listener `handle` methods.
- To invoke closures with automatically injected dependencies via `app()->call()`.
- To combine route parameters with injected dependencies in a single method signature.
- To keep controllers lean by injecting dependencies only where needed.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Method injection: The container resolves AppleMusic and Request
     * and injects them when this action is invoked.
     * The $id parameter is a route parameter (primitive).
     */
    public function show(Request $request, AppleMusic $apple, string $id)
    {
        return $apple->findPodcast($id);
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `Request $request` | Injected HTTP request |
| `AppleMusic $apple` | Injected service |
| `string $id` | Route parameter (primitive) |
| `app()->call([$obj, 'method'])` | Manual method invocation with DI |
| `handle()` | Job/listener method resolved by container |

#### Syntax Rules

1. Method injection works automatically for controller actions, job `handle` methods, listener `handle` methods, and middleware.
2. The container resolves type-hinted dependencies and passes them as arguments.
3. Primitive parameters (route parameters) are matched by name or position.
4. Method injection can be used in conjunction with constructor injection.
5. `app()->call()` manually invokes any PHP callable with DI.
6. The `Request` object is always available via method injection in controllers.
7. Method injection does **not** work for arbitrary methods on arbitrary classes; it is limited to container-invoked methods.

#### Constraints and Limitations

- **Limited Scope:** Method injection only works for methods invoked by the container (controllers, jobs, listeners, middleware, `app()->call()`).
- **No Automatic Resolution for Model Methods:** Calling `$model->someMethod()` does not trigger DI; use constructor injection or `app()->call()`.
- **Route Parameter Conflicts:** If a route parameter name matches a type-hinted dependency name, Laravel may inject the wrong value.
- **Performance:** Reflection on each method invocation adds overhead; negligible for most applications.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Controller Action with Request and Service Injection

**Step-by-Step Setup Guide:**

1. Define a route with a parameter.
2. Create a controller action that type-hints `Request` and a service.
3. The container resolves both and injects them when the route is hit.

**Complete Executable Code:**

```php
<?php
// app/Services/PodcastParser.php

namespace App\Services;

class PodcastParser
{
    public function parse(array $podcast): array
    {
        return array_merge($podcast, ['parsed' => true]);
    }
}
```

```php
<?php
// app/Http/Controllers/PodcastController.php

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use App\Services\PodcastParser;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Method injection: Request, AppleMusic, and PodcastParser
     * are resolved by the container. $id is a route parameter.
     */
    public function show(
        Request $request,
        AppleMusic $apple,
        PodcastParser $parser,
        string $id
    ) {
        $podcast = $apple->findPodcast($id);
        return $parser->parse($podcast);
    }
}
```

```php
// routes/web.php

Route::get('/podcasts/{id}', [PodcastController::class, 'show']);
```

**Expected Output:**

- `GET /podcasts/42` returns `{"id":"42","title":"Example Podcast","parsed":true}`.
- The `Request` object is available if needed (e.g., for query parameters).
- The `AppleMusic` and `PodcastParser` services are injected automatically.

**Why This Code Produces That Result:**

- The container inspects the `show` method's parameters.
- It resolves `Request`, `AppleMusic`, and `PodcastParser` from the container.
- The `$id` parameter is supplied by the route.
- All parameters are passed to the method in the correct order.

#### Example 2: Queued Job with Method Injection

```php
<?php
// app/Jobs/ProcessPodcast.php

namespace App\Jobs;

use App\Services\AppleMusic;
use App\Services\PodcastParser;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        protected string $podcastId
    ) {}

    /**
     * The container resolves AppleMusic and PodcastParser
     * and injects them when the job is processed.
     */
    public function handle(AppleMusic $apple, PodcastParser $parser): void
    {
        $podcast = $apple->findPodcast($this->podcastId);
        $parsed = $parser->parse($podcast);
        // ... store parsed podcast
    }
}
```

**Expected Output:**

- When the queued job runs, the container injects `AppleMusic` and `PodcastParser` into `handle()`.
- The job processes the podcast with the injected services.

**Why This Code Produces That Result:**

- Laravel's queue worker invokes `handle()` through the container.
- The container inspects the method signature and resolves the type-hinted dependencies.
- This allows jobs to use services without storing them as properties (which would complicate serialization).

### Real-World Cases

**Case 1: API Controller with Request Injection**

A REST API controller injects the `Request` object into each action to access query parameters, headers, and the authenticated user.

**Case 2: Event Listener with Service Injection**

An event listener for `OrderShipped` injects a `NotificationService` into its `handle()` method to send shipping notifications.

**Case 3: Middleware with Dependency Injection**

A middleware injects a `RateLimiter` service into its `handle()` method to enforce per-user rate limits.

### References

- Laravel Controllers: Dependency Injection - https://laravel.com/docs/12.x/controllers#dependency-injection-and-controllers
- Laravel Queues: Job Middleware and Injection - https://laravel.com/docs/12.x/queues#job-middleware
- Laravel Service Container: Method Invocation - https://laravel.com/docs/12.x/container#method-invocation-and-injection
- Laravel API: Container::call - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html#method_call


## 3. Interface-Based Dependencies

### Definitions

**Core Definition:** Interface-based dependencies are a design approach in which a class depends on an interface (abstraction) rather than a concrete class, and the Laravel Service Container binds that interface to a specific concrete implementation at runtime.

**Technical Definition:** The container's `bind()` method registers a mapping between an abstract type (usually an interface FQCN) and a concrete type (a class FQCN or Closure). When the abstract is resolved, the container instantiates the concrete and returns it. This decouples the consuming class from the implementation, enabling swapping of implementations without modifying the consumer. The binding is typically registered in a Service Provider's `register()` method.

**Beginner-Friendly Explanation:** Imagine you need a "payment processor" in your app. Instead of hard-coding "Stripe," you write your code to depend on a "PaymentProcessor" interface. The container then decides which concrete processor to use—Stripe, PayPal, or a fake one for testing. Your code never knows or cares which one it's using; it just calls `charge()` and the container handles the rest.

### Purposes

- To decouple high-level modules from low-level implementation details.
- To allow swapping implementations without modifying consuming code.
- To enable mocking in tests by binding a fake implementation.
- To support multiple implementations (e.g., Stripe, PayPal, Square) selected by configuration.
- To follow the Dependency Inversion Principle (the "D" in SOLID).
- To make the application more maintainable and extensible.
- To centralize implementation choices in Service Providers.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// 1. Define the interface

namespace App\Contracts;

interface EventPusher
{
    public function push(string $event, array $data): void;
}
```

```php
<?php
// 2. Implement the interface

namespace App\Services;

use App\Contracts\EventPusher;

class RedisEventPusher implements EventPusher
{
    public function push(string $event, array $data): void
    {
        \Redis::publish($event, json_encode($data));
    }
}
```

```php
<?php
// 3. Bind the interface to the implementation in a Service Provider

namespace App\Providers;

use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(EventPusher::class, RedisEventPusher::class);
    }
}
```

```php
<?php
// 4. Depend on the interface, not the concrete class

namespace App\Http\Controllers;

use App\Contracts\EventPusher;

class EventController extends Controller
{
    public function __construct(
        protected EventPusher $pusher
    ) {}

    public function push()
    {
        $this->pusher->push('user.created', ['id' => 1]);
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `interface EventPusher` | The abstraction |
| `class RedisEventPusher implements EventPusher` | The concrete implementation |
| `$this->app->bind(EventPusher::class, RedisEventPusher::class)` | The binding |
| `__construct(EventPusher $pusher)` | Consumer depends on abstraction |
| `AppServiceProvider` | Registration point |

#### Syntax Rules

1. The interface **must** be defined and the concrete class **must** implement it.
2. The binding **must** be registered before the consumer is resolved.
3. `bind()` maps the interface to the concrete; `singleton()` makes it shared.
4. The consumer type-hints the interface, not the concrete class.
5. The container resolves the interface by looking up the binding.
6. Bindings can be closures for complex instantiation logic.
7. The binding is typically registered in a Service Provider's `register()` method.

#### Constraints and Limitations

- **Registration Required:** Interfaces cannot be auto-resolved; a binding is mandatory.
- **Binding Timing:** Bindings must be registered before resolution; late registration causes `BindingResolutionException`.
- **Single Binding:** A single interface can only have one default binding; contextual bindings are needed for multiple consumers.
- **No Type Safety at Binding Time:** The container does not verify that the concrete implements the interface at binding time (errors occur at resolution).
- **Overhead:** Resolution requires a lookup in the bindings array; negligible in practice.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Repository Interface Binding

**Step-by-Step Setup Guide:**

1. Define a `UserRepository` interface.
2. Implement `EloquentUserRepository`.
3. Bind the interface to the implementation.
4. Inject the interface into a controller.

**Complete Executable Code:**

```php
<?php
// app/Contracts/UserRepository.php

namespace App\Contracts;

interface UserRepository
{
    public function find(int $id): ?array;
    public function all(): array;
}
```

```php
<?php
// app/Repositories/EloquentUserRepository.php

namespace App\Repositories;

use App\Contracts\UserRepository;
use App\Models\User;

class EloquentUserRepository implements UserRepository
{
    public function find(int $id): ?array
    {
        return User::find($id)?->toArray();
    }

    public function all(): array
    {
        return User::all()->toArray();
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\UserRepository;
use App\Repositories\EloquentUserRepository;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(UserRepository::class, EloquentUserRepository::class);
    }
}
```

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Contracts\UserRepository;

class UserController extends Controller
{
    public function __construct(
        protected UserRepository $users
    ) {}

    public function index()
    {
        return response()->json($this->users->all());
    }

    public function show(int $id)
    {
        return response()->json($this->users->find($id));
    }
}
```

**Expected Output:**

- The controller receives an `EloquentUserRepository` instance.
- Swapping to a `CacheUserRepository` requires only changing the binding—no controller changes.

**Why This Code Produces That Result:**

- The `bind()` call maps the interface to the concrete repository.
- The container resolves the interface when instantiating the controller.
- The controller is decoupled from the concrete implementation.

#### Example 2: Testing with a Mock Implementation

```php
<?php
// tests/Feature/UserControllerTest.php

namespace Tests\Feature;

use App\Contracts\UserRepository;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    public function test_index_returns_users()
    {
        // Bind a fake repository for the test
        $this->app->bind(UserRepository::class, function () {
            return new class implements UserRepository {
                public function find(int $id): ?array
                {
                    return ['id' => $id, 'name' => 'Test User'];
                }

                public function all(): array
                {
                    return [['id' => 1, 'name' => 'Test User']];
                }
            };
        });

        $response = $this->get('/users');

        $response->assertJson([['id' => 1, 'name' => 'Test User']]);
    }
}
```

**Expected Output:**

- The test binds a fake implementation and asserts the controller returns the fake data.
- No database access occurs.

**Why This Code Produces That Result:**

- The test overrides the binding in the container.
- The controller receives the fake implementation.
- This proves the controller is decoupled from the real repository.

### Real-World Cases

**Case 1: Payment Gateway Abstraction**

An application binds `PaymentGateway` to `StripeGateway` in production and `FakeGateway` in tests, enabling safe testing of payment flows.

**Case 2: Storage Abstraction**

An application binds `FileStorage` to `S3Storage` in production and `LocalStorage` in development.

**Case 3: Notification Channels**

A notification service binds `NotificationChannel` to `EmailChannel` or `SmsChannel` based on user preferences.

### References

- Laravel Service Container: Binding Interfaces to Implementations - https://laravel.com/docs/12.x/container#binding-interfaces-to-implementations
- Laravel Service Container: Binding - https://laravel.com/docs/12.x/container#binding
- Laravel News: Advanced Application Architecture - https://laravel-news.com/service-container-management
- SOLID Principles: Dependency Inversion - https://en.wikipedia.org/wiki/Dependency_inversion_principle


## 4. Dependency Inversion (Adhering to SOLID Principles by Depending upon Abstractions Rather than Concretions)

### Definitions

**Core Definition:** The Dependency Inversion Principle (DIP) is the "D" in the SOLID principles, stating that high-level modules should not depend on low-level modules; both should depend on abstractions, and abstractions should not depend on details—details should depend on abstractions.

**Technical Definition:** DIP is implemented in Laravel through the Service Container's interface binding mechanism. A high-level class (e.g., `OrderService`) depends on an interface (`PaymentGateway`), not a concrete class (`StripeGateway`). The container injects the concrete implementation at runtime. This inverts the traditional dependency direction: instead of `OrderService` depending on `StripeGateway` (a low-level detail), both depend on the `PaymentGateway` abstraction. The concrete implementation also depends on the abstraction (by implementing it), not the other way around.

**Beginner-Friendly Explanation:** Normally, a manager (high-level) might depend on a specific worker (low-level). If the worker quits, the manager is stuck. Dependency Inversion says: the manager should depend on a job description (interface), and any qualified worker can fill it. The manager doesn't care who the worker is—just that they can do the job. In Laravel, the container is the hiring agency that matches the job description to a worker.

### Purposes

- To decouple high-level business logic from low-level implementation details.
- To make the application resilient to changes in external services or libraries.
- To enable swapping implementations without modifying high-level code.
- To improve testability by allowing mocks to be injected.
- To follow the SOLID principles for maintainable, extensible code.
- To reduce the ripple effect of changes across the codebase.
- To establish clear contracts between modules.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// The abstraction (interface)

namespace App\Contracts;

interface PaymentGateway
{
    public function charge(float $amount): bool;
}
```

```php
<?php
// A low-level module depends on the abstraction

namespace App\Services;

use App\Contracts\PaymentGateway;

class StripeGateway implements PaymentGateway
{
    public function charge(float $amount): bool
    {
        // Stripe-specific implementation
        return true;
    }
}
```

```php
<?php
// A high-level module also depends on the abstraction

namespace App\Services;

use App\Contracts\PaymentGateway;

class OrderService
{
    public function __construct(
        protected PaymentGateway $gateway
    ) {}

    public function checkout(float $amount): bool
    {
        return $this->gateway->charge($amount);
    }
}
```

```php
<?php
// The container wires the abstraction to a concrete implementation

namespace App\Providers;

use App\Contracts\PaymentGateway;
use App\Services\StripeGateway;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(PaymentGateway::class, StripeGateway::class);
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `PaymentGateway` (interface) | The abstraction both modules depend on |
| `StripeGateway` (concrete) | Low-level implementation; depends on the abstraction |
| `OrderService` (high-level) | Depends on the abstraction, not the concrete |
| `bind()` | Wires the abstraction to the concrete |

#### Syntax Rules

1. Define interfaces for all external dependencies of high-level modules.
2. High-level modules **must** type-hint interfaces, not concrete classes.
3. Low-level modules **must** implement the interfaces.
4. The container **must** be told which concrete to use via `bind()`.
5. The binding is registered in a Service Provider.
6. The concrete implementation can be swapped without changing high-level code.
7. The abstraction (interface) should be owned by the high-level module, not the low-level one.

#### Constraints and Limitations

- **Over-Abstraction:** Not every class needs an interface; premature abstraction adds complexity.
- **Binding Overhead:** Each interface requires a binding; too many bindings can bloat service providers.
- **Naming Confusion:** Interfaces and implementations should be named clearly (e.g., `PaymentGateway` → `StripeGateway`).
- **Learning Curve:** Developers new to DI may struggle with the inversion concept.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: DIP in a Notification System

**Step-by-Step Setup Guide:**

1. Define a `NotificationService` interface.
2. Implement `EmailNotificationService` and `SlackNotificationService`.
3. Bind the interface to `EmailNotificationService` by default.
4. A high-level `AlertManager` depends on the interface.
5. Swap implementations via configuration.

**Complete Executable Code:**

```php
<?php
// app/Contracts/NotificationService.php

namespace App\Contracts;

interface NotificationService
{
    public function send(string $message, string $recipient): void;
}
```

```php
<?php
// app/Services/EmailNotificationService.php

namespace App\Services;

use App\Contracts\NotificationService;

class EmailNotificationService implements NotificationService
{
    public function send(string $message, string $recipient): void
    {
        \Mail::raw($message, fn ($mail) => $mail->to($recipient));
    }
}
```

```php
<?php
// app/Services/SlackNotificationService.php

namespace App\Services;

use App\Contracts\NotificationService;

class SlackNotificationService implements NotificationService
{
    public function send(string $message, string $recipient): void
    {
        \Http::post('https://slack.com/api/chat.postMessage', [
            'channel' => $recipient,
            'text' => $message,
        ]);
    }
}
```

```php
<?php
// app/Services/AlertManager.php

namespace App\Services;

use App\Contracts\NotificationService;

class AlertManager
{
    public function __construct(
        protected NotificationService $notifier
    ) {}

    public function alert(string $message): void
    {
        $this->notifier->send($message, 'admin@example.com');
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\NotificationService;
use App\Services\EmailNotificationService;
use App\Services\SlackNotificationService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind based on environment or config
        $this->app->bind(NotificationService::class, function () {
            return app()->environment('production')
                ? new SlackNotificationService()
                : new EmailNotificationService();
        });
    }
}
```

**Expected Output:**

- In production, `AlertManager` uses `SlackNotificationService`.
- In development, it uses `EmailNotificationService`.
- No changes to `AlertManager` are needed.

**Why This Code Produces That Result:**

- `AlertManager` depends on the `NotificationService` abstraction.
- The binding closure selects the implementation based on environment.
- DIP decouples the high-level alerting logic from the low-level notification method.

#### Example 2: Testing with DIP

```php
<?php
// tests/Unit/AlertManagerTest.php

namespace Tests\Unit;

use App\Contracts\NotificationService;
use App\Services\AlertManager;
use PHPUnit\Framework\TestCase;

class AlertManagerTest extends TestCase
{
    public function test_alert_sends_notification()
    {
        // Create a mock implementation
        $mock = $this->createMock(NotificationService::class);
        $mock->expects($this->once())
            ->method('send')
            ->with('Test alert', 'admin@example.com');

        $manager = new AlertManager($mock);
        $manager->alert('Test alert');
    }
}
```

**Expected Output:**

- The test passes, verifying that `alert()` calls `send()` with the correct arguments.
- No real notification is sent.

**Why This Code Produces That Result:**

- `AlertManager` depends on the interface, so a mock can be injected.
- DIP enables unit testing without external dependencies.

### Real-World Cases

**Case 1: Multi-Cloud Abstraction**

A SaaS platform defines a `CloudStorage` interface and binds it to `S3Storage`, `GCSStorage`, or `AzureStorage` based on tenant configuration.

**Case 2: Payment Provider Rotation**

An e-commerce platform binds `PaymentGateway` to different providers per region, following DIP.

**Case 3: Feature Flag Swapping**

A feature-flagged application binds `SearchService` to `ElasticSearchService` or `DatabaseSearchService` based on a feature flag.

### References

- Laravel Service Container: Binding Interfaces to Implementations - https://laravel.com/docs/12.x/container#binding-interfaces-to-implementations
- SOLID Principles: Dependency Inversion - https://en.wikipedia.org/wiki/Dependency_inversion_principle
- Laravel News: Advanced Application Architecture - https://laravel-news.com/service-container-management
- CodeMag: Dependency Injection and Service Container in Laravel - https://www.codemag.com/Article/2212041/Dependency-Injection-and-Service-Container-in-Laravel


## 5. Enhanced: Container Call Invocation (`$this->app->call()` to Execute Any PHP Callable with Automatic DI)

### Definitions

**Core Definition:** The container's `call()` method invokes any PHP callable (closure, `[object, method]` array, or `Class@method` string) while automatically resolving and injecting its type-hinted dependencies.

**Technical Definition:** `Container::call()` accepts a callable and an optional array of parameters. It uses Reflection to inspect the callable's parameters, resolves type-hinted dependencies from the container, merges any explicitly provided parameters, and invokes the callable. This is useful when you need to execute a method or closure that has dependencies but is not automatically resolved by the framework (e.g., arbitrary methods on objects, custom commands, or dynamic callbacks).

**Beginner-Friendly Explanation:** `call()` is like a universal remote for your code. You point it at any function or method and say "run this," and it automatically gathers everything that function needs—dependencies, parameters, and all—before running it. It's useful when you want the container's magic outside of the usual controller/job flow.

### Purposes

- To invoke arbitrary methods on objects with automatic dependency injection.
- To execute closures with container-resolved dependencies.
- To call methods that are not automatically resolved by the framework (e.g., service methods, custom callbacks).
- To combine explicitly passed parameters with container-resolved dependencies.
- To enable dependency injection in contexts that Laravel doesn't automatically resolve.
- To test methods in isolation by invoking them through the container.
- To dynamically execute callbacks in event-driven or plugin architectures.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php

use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;

// Invoke a closure with injected dependencies
$result = App::call(function (AppleMusic $apple) {
    return $apple->findPodcast('123');
});

// Invoke an object method
$stats = App::call([new PodcastStats, 'generate']);

// Invoke a class@method string
$result = App::call('App\PodcastStats@generate');

// Invoke with explicit parameters merged with DI
$result = App::call([$service, 'process'], ['orderId' => 123]);

// Using the container instance
$result = $this->app->call([$service, 'process'], ['orderId' => 123]);
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `App::call($callable)` | Facade method for container invocation |
| `$this->app->call($callable)` | Container instance method |
| `[$object, 'method']` | Array callable syntax |
| `'Class@method'` | String callable syntax |
| `function (Dep $dep) { }` | Closure with type-hinted dependency |
| `$parameters` | Associative array of explicit parameters |

#### Syntax Rules

1. `call()` accepts any PHP callable: Closure, `[object, 'method']`, `'Class@method'`, or invokable object.
2. Type-hinted dependencies in the callable's signature are resolved from the container.
3. Explicit parameters are merged with resolved dependencies.
4. Explicit parameters **must** be an associative array keyed by parameter name.
5. If a parameter is both type-hinted and explicitly provided, the explicit value takes precedence.
6. `call()` uses Reflection to inspect the callable's parameters.
7. Primitive parameters without explicit values and without defaults cause a `BindingResolutionException`.

#### Constraints and Limitations

- **Reflection Overhead:** Each `call()` invocation performs Reflection; avoid in tight loops.
- **Parameter Name Matching:** Explicit parameters must match the parameter name exactly.
- **No Automatic Resolution for `__invoke` Objects:** Invokable objects are supported, but dependencies are resolved from the `__invoke` method.
- **Error Handling:** Exceptions thrown by the callable propagate normally.
- **No Route Model Binding:** `call()` does not perform route model binding; use explicit parameters.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Invoking a Service Method with DI

**Step-by-Step Setup Guide:**

1. Create a service with a method that has dependencies.
2. Use `App::call()` to invoke the method.
3. The container resolves the dependencies and passes them.

**Complete Executable Code:**

```php
<?php
// app/Services/PodcastStats.php

namespace App\Services;

class PodcastStats
{
    /**
     * Generate stats for a podcast. The AppleMusic dependency
     * will be resolved by the container when called via App::call().
     */
    public function generate(AppleMusic $apple): array
    {
        return [
            'podcast' => $apple->findPodcast('123'),
            'generated_at' => now()->toIso8601String(),
        ];
    }
}
```

```php
<?php
// routes/web.php

use App\Services\PodcastStats;
use Illuminate\Support\Facades\App;

Route::get('/stats', function () {
    // App::call() resolves AppleMusic and injects it into generate()
    $stats = App::call([new PodcastStats, 'generate']);
    return response()->json($stats);
});
```

**Expected Output:**

- `GET /stats` returns the podcast stats with the injected `AppleMusic` data.
- The `AppleMusic` dependency is resolved automatically.

**Why This Code Produces That Result:**

- `App::call([new PodcastStats, 'generate'])` inspects the `generate` method.
- It sees the `AppleMusic` type-hint and resolves it from the container.
- The resolved instance is passed as the `$apple` argument.

#### Example 2: Invoking a Closure with Mixed Parameters

```php
<?php
// routes/web.php

use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;

Route::get('/podcast/{id}', function (string $id) {
    // App::call() resolves AppleMusic and merges the explicit 'id'
    $result = App::call(function (AppleMusic $apple, string $id) {
        return $apple->findPodcast($id);
    }, ['id' => $id]);

    return response()->json($result);
});
```

**Expected Output:**

- `GET /podcast/42` returns the podcast with ID 42.
- `AppleMusic` is injected; `$id` is supplied explicitly.

**Why This Code Produces That Result:**

- The closure type-hints `AppleMusic` (resolved) and `string $id` (explicit).
- `App::call()` resolves `AppleMusic`, merges the `id` parameter, and invokes the closure.
- The `$id` parameter is passed by name.

### Real-World Cases

**Case 1: Dynamic Report Generation**

A reporting system uses `App::call()` to invoke different report generator methods based on user selection, injecting shared dependencies.

**Case 2: Plugin Execution**

A plugin system invokes plugin methods via `App::call()`, allowing plugins to declare dependencies that the container resolves.

**Case 3: Testing Service Methods**

Tests use `App::call()` to invoke service methods with dependencies, verifying behavior without full application bootstrapping.

### References

- Laravel Service Container: Method Invocation and Injection - https://laravel.com/docs/12.x/container#method-invocation-and-injection
- Laravel API: Container::call - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html#method_call
- GitHub: Laravel Documentation (13.x) - Method Invocation - https://github.com/driade/laravel-book


## 6. Enhanced: Variadic Dependency Resolution (Injecting Dynamic Arrays of Objects or Configuration Primitives into Constructors)

### Definitions

**Core Definition:** Variadic dependency resolution is the container's ability to inject a dynamic array of objects (or primitive values) into a constructor's variadic parameter (`Type ...$items`), using contextual binding with `give()` or `giveTagged()`.

**Technical Definition:** When a class's constructor has a variadic parameter type-hinted as a class (e.g., `Filter ...$filters`), the container cannot resolve it automatically because variadic parameters expect multiple instances. Laravel provides two mechanisms: (1) contextual binding with `$this->app->when(Consumer::class)->needs(Filter::class)->give([...])` to supply an array of concrete classes, or (2) `giveTagged('tag')` to inject all bindings associated with a tag. The container resolves each class in the array and passes them as variadic arguments.

**Beginner-Friendly Explanation:** A variadic parameter is like a shopping bag that can hold any number of items. Instead of saying "I need exactly one filter," you say "I need zero or more filters." Laravel lets you fill that bag with a list of filter classes (or tagged services) that the container resolves and drops into the constructor as individual arguments.

### Purposes

- To inject a dynamic collection of services into a class.
- To support plugin architectures where the number of plugins varies.
- To implement filter chains, validator pipelines, and middleware stacks.
- To inject multiple implementations of an interface as a variadic array.
- To use tagged bindings for variadic injection without hard-coding class lists.
- To decouple the consumer from the specific list of implementations.
- To support report aggregators that combine multiple report generators.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Consumer class with a variadic dependency

namespace App\Services;

use App\Contracts\Filter;
use App\Contracts\Logger;

class Firewall
{
    protected array $filters;

    /**
     * The container will inject multiple Filter instances
     * into the variadic $filters parameter.
     */
    public function __construct(
        protected Logger $logger,
        Filter ...$filters
    ) {
        $this->filters = $filters;
    }

    public function inspect(string $input): bool
    {
        foreach ($this->filters as $filter) {
            if (!$filter->passes($input)) {
                return false;
            }
        }
        return true;
    }
}
```

```php
<?php
// Contextual binding to supply the variadic array

namespace App\Providers;

use App\Services\Firewall;
use App\Contracts\Filter;
use App\Filters\NullFilter;
use App\Filters\ProfanityFilter;
use App\Filters\TooLongFilter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Option 1: Give an array of class names
        $this->app->when(Firewall::class)
            ->needs(Filter::class)
            ->give([
                NullFilter::class,
                ProfanityFilter::class,
                TooLongFilter::class,
            ]);

        // Option 2: Give a closure returning resolved instances
        $this->app->when(Firewall::class)
            ->needs(Filter::class)
            ->give(function ($app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(ProfanityFilter::class),
                    $app->make(TooLongFilter::class),
                ];
            });

        // Option 3: Give tagged bindings
        $this->app->tag([NullFilter::class, ProfanityFilter::class], 'filters');
        $this->app->when(Firewall::class)
            ->needs(Filter::class)
            ->giveTagged('filters');
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `Filter ...$filters` | Variadic parameter type-hinted as interface |
| `when(Firewall::class)` | Contextual binding context |
| `needs(Filter::class)` | The variadic dependency type |
| `give([...])` | Array of concrete classes |
| `giveTagged('filters')` | Inject all tagged bindings |
| `tag([...], 'filters')` | Tag bindings for variadic injection |

#### Syntax Rules

1. The variadic parameter **must** be type-hinted as the interface or class being injected.
2. Variadic dependencies **cannot** be auto-resolved; contextual binding is required.
3. `give()` accepts an array of class names, a Closure returning an array, or `giveTagged()`.
4. Each class in the array is resolved individually by the container.
5. `giveTagged()` injects all bindings associated with the tag.
6. Tagged bindings can be registered anywhere; the order of injection follows registration/tag order.
7. The consumer receives the variadic instances as individual constructor arguments.

#### Constraints and Limitations

- **No Auto-Resolution:** Variadic parameters require explicit contextual binding.
- **Order Dependency:** The order of instances follows the array order or tag registration order.
- **Type Safety:** All injected instances **must** implement the type-hinted interface.
- **Tag Collisions:** Using the same tag across packages can cause unexpected inclusions.
- **Performance:** Resolving many tagged bindings can be expensive if the list is large.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Filter Chain with Variadic Injection

**Step-by-Step Setup Guide:**

1. Define a `Filter` interface and three implementations.
2. Create a `Firewall` class with a variadic `Filter ...$filters` parameter.
3. Register a contextual binding with an array of filter classes.
4. Resolve `Firewall` and verify all filters are injected.

**Complete Executable Code:**

```php
<?php
// app/Contracts/Filter.php

namespace App\Contracts;

interface Filter
{
    public function passes(string $input): bool;
}
```

```php
<?php
// app/Filters/NullFilter.php
namespace App\Filters;
use App\Contracts\Filter;

class NullFilter implements Filter
{
    public function passes(string $input): bool
    {
        return $input !== null;
    }
}
```

```php
<?php
// app/Filters/ProfanityFilter.php
namespace App\Filters;
use App\Contracts\Filter;

class ProfanityFilter implements Filter
{
    public function passes(string $input): bool
    {
        return !str_contains($input, 'badword');
    }
}
```

```php
<?php
// app/Filters/TooLongFilter.php
namespace App\Filters;
use App\Contracts\Filter;

class TooLongFilter implements Filter
{
    public function passes(string $input): bool
    {
        return strlen($input) <= 255;
    }
}
```

```php
<?php
// app/Services/Firewall.php

namespace App\Services;

use App\Contracts\Filter;
use App\Contracts\Logger;

class Firewall
{
    protected array $filters;

    public function __construct(
        protected Logger $logger,
        Filter ...$filters
    ) {
        $this->filters = $filters;
    }

    public function inspect(string $input): bool
    {
        foreach ($this->filters as $filter) {
            if (!$filter->passes($input)) {
                $this->logger->log('Filter rejected: ' . get_class($filter));
                return false;
            }
        }
        return true;
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\Firewall;
use App\Contracts\Filter;
use App\Filters\NullFilter;
use App\Filters\ProfanityFilter;
use App\Filters\TooLongFilter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->when(Firewall::class)
            ->needs(Filter::class)
            ->give([
                NullFilter::class,
                ProfanityFilter::class,
                TooLongFilter::class,
            ]);
    }
}
```

**Expected Output:**

- Resolving `Firewall` injects three `Filter` instances.
- `inspect('hello')` returns `true` (passes all filters).
- `inspect('badword')` returns `false` (fails the profanity filter).

**Why This Code Produces That Result:**

- The contextual binding tells the container to inject the three filter classes.
- Each class is resolved individually and passed as variadic arguments.
- The firewall iterates over the filters and applies each one.

#### Example 2: Variadic Injection with Tagged Bindings

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\ReportAggregator;
use App\Contracts\Report;
use App\Reports\CpuReport;
use App\Reports\MemoryReport;
use App\Reports\DiskReport;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind report implementations
        $this->app->bind(CpuReport::class, fn () => new CpuReport());
        $this->app->bind(MemoryReport::class, fn () => new MemoryReport());
        $this->app->bind(DiskReport::class, fn () => new DiskReport());

        // Tag them
        $this->app->tag([
            CpuReport::class,
            MemoryReport::class,
            DiskReport::class,
        ], 'reports');

        // Inject all tagged reports into ReportAggregator's variadic parameter
        $this->app->when(ReportAggregator::class)
            ->needs(Report::class)
            ->giveTagged('reports');
    }
}
```

```php
<?php
// app/Services/ReportAggregator.php

namespace App\Services;

use App\Contracts\Report;

class ReportAggregator
{
    protected array $reports;

    public function __construct(
        Report ...$reports
    ) {
        $this->reports = $reports;
    }

    public function all(): array
    {
        $results = [];
        foreach ($this->reports as $report) {
            $results = array_merge($results, $report->generate());
        }
        return $results;
    }
}
```

**Expected Output:**

- `ReportAggregator` receives all three reports via variadic injection.
- `all()` returns the merged results of all reports.

**Why This Code Produces That Result:**

- `giveTagged('reports')` resolves all bindings tagged `reports`.
- Each is passed as a variadic argument to the constructor.
- Adding a new report requires only tagging it—no changes to the aggregator.

### Real-World Cases

**Case 1: Validation Pipeline**

A validation service receives multiple `Validator` implementations via variadic injection, allowing dynamic validation rules.

**Case 2: Report Aggregator**

A monitoring dashboard aggregates CPU, memory, disk, and network reports, each tagged `reports`, via variadic injection.

**Case 3: Middleware Stack**

A custom middleware stack receives multiple `Middleware` instances via variadic injection, processed in order.

### References

- Laravel Service Container: Binding Typed Variadics - https://laravel.com/docs/12.x/container#binding-typed-variadics
- Laravel Service Container: Variadic Tag Dependencies - https://laravel.com/docs/12.x/container#variadic-tag-dependencies
- Laravel Service Container: Tagging - https://laravel.com/docs/12.x/container#tagging
- Laravel API: ContextualBindingBuilder - https://api.laravel.com/docs/11.x/Illuminate/Container/ContextualBindingBuilder.html


## Summary Table of Dependency Injection Features

| Feature | Mechanism | Container Method | Use Case |
|---------|-----------|-----------------|----------|
| Constructor Injection | `__construct(Type $dep)` | Automatic (Reflection) | Primary DI for controllers, services |
| Method Injection | `method(Type $dep)` | Automatic (Reflection) | Controller actions, job handlers |
| Interface Binding | `bind(Interface, Concrete)` | `$this->app->bind()` | Decoupling, swappable implementations |
| Dependency Inversion | Interface + binding | `bind()` + type-hint | SOLID "D" principle |
| Container Call | `App::call($callable)` | `Container::call()` | Arbitrary callable execution with DI |
| Variadic Resolution | `Type ...$items` | `give()` / `giveTagged()` | Plugin architectures, filter chains |


## References

- Laravel Service Container Documentation (12.x) - https://laravel.com/docs/12.x/container
- Laravel Service Container: Automatic Injection - https://laravel.com/docs/12.x/container#automatic-injection
- Laravel Service Container: Binding Interfaces to Implementations - https://laravel.com/docs/12.x/container#binding-interfaces-to-implementations
- Laravel Service Container: Method Invocation and Injection - https://laravel.com/docs/12.x/container#method-invocation-and-injection
- Laravel Service Container: Binding Typed Variadics - https://laravel.com/docs/12.x/container#binding-typed-variadics
- Laravel Service Container: Variadic Tag Dependencies - https://laravel.com/docs/12.x/container#variadic-tag-dependencies
- Laravel Controllers: Dependency Injection - https://laravel.com/docs/12.x/controllers#dependency-injection-and-controllers
- Laravel Queues: Job Middleware and Injection - https://laravel.com/docs/12.x/queues#job-middleware
- Laravel API: Container - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html
- Laravel API: ContextualBindingBuilder - https://api.laravel.com/docs/11.x/Illuminate/Container/ContextualBindingBuilder.html
- Laravel News: Advanced Application Architecture through Laravel's Service Container Management - https://laravel-news.com/service-container-management
- CodeMag: Dependency Injection and Service Container in Laravel - https://www.codemag.com/Article/2212041/Dependency-Injection-and-Service-Container-in-Laravel
- SOLID Principles: Dependency Inversion - https://en.wikipedia.org/wiki/Dependency_inversion_principle
- PHP Reflection API - https://www.php.net/manual/en/book.reflection.php