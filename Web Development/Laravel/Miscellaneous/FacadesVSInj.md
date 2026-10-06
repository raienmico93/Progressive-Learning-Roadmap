# Laravel Facades versus Dependency Injection: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Facades and Dependency Injection (DI) are two distinct approaches to accessing services in a Laravel application. Facades provide a static, global-looking interface to services in the container, while Dependency Injection provides services through explicit constructor or method parameters, making dependencies visible and swappable.

**Technical Definition:** Facades (`Illuminate\Support\Facades\Facade`) use PHP's `__callStatic()` magic method to proxy static calls to underlying services resolved from the service container. Dependency Injection, in contrast, uses PHP's Reflection API to inspect type-hinted constructor or method parameters and inject the resolved dependencies at instantiation or invocation time. Both ultimately resolve services from the same container, but they differ in explicitness, coupling, testability, and static analysis compatibility. Laravel Facades are technically service locators—a design pattern where a class asks a global registry for its dependencies rather than receiving them explicitly.

**Beginner-Friendly Explanation:** Imagine you work in an office. With Dependency Injection, your boss hands you the exact tools you need before you start working—"Here's your laptop, here's your phone, here's your coffee." With Facades, you walk to a central supply closet whenever you need something—"I need a laptop" (and the closet hands you one). Both approaches get the job done. Facades are quicker in the moment, but Dependency Injection makes it clearer what you actually need. The best teams use each where it makes sense.

### Key Characteristics

1. **Explicitness:** DI makes dependencies visible in the constructor signature; facades hide them behind static calls.
2. **Convenience:** Facades offer terse, memorable syntax; DI requires more boilerplate but is more explicit.
3. **Testability:** Both are testable—facades via `shouldReceive()`, DI via container mocks or constructor injection.
4. **Coupling:** Facades couple classes to Laravel's global state; DI keeps classes decoupled and portable.
5. **Static Analysis:** DI is fully supported by PHPStan/Larastan; facades require Laravel-specific extensions.
6. **Hidden Dependencies:** Facades can obscure a class's actual dependency footprint (the "API lie").
7. **Global State:** Facades resolve from the global container; DI passes dependencies explicitly.
8. **Team Guardrails:** Clear conventions are needed to decide when to use each approach.

### Prerequisites

- Laravel 10.x or higher (12.x recommended)
- PHP 8.1 or higher
- Composer package manager
- Familiarity with the Laravel Service Container
- Understanding of the Facade pattern and `__callStatic()`
- Basic knowledge of PHP type-hinting and interfaces
- PHPStan or Larastan for static analysis (optional but recommended)
- A working Laravel application

### Related Programming Areas

- **Service Container:** The underlying DI container both approaches use.
- **Service Providers:** Where services are bound and registered.
- **Contracts:** The interfaces that DI often type-hints.
- **Testing and Mocking:** Both approaches support mocking, but differently.
- **Static Analysis:** PHPStan, Larastan, and IDE helpers.
- **SOLID Principles:** DI promotes Dependency Inversion; facades can violate it.
- **Service Locator Pattern:** Facades are a form of service locator.

### Core Concepts / Features

1. Convenience (Rapid prototyping and readable, expressive syntax)
2. Testability (Leveraging static mocking frameworks without traditional dependency mocking overhead)
3. Coupling (Balancing tight coupling to global state against constructor parameter abstraction)
4. Architectural Considerations (Team guardrails on facades in microservices vs. clean DDD layers)
5. Enhanced: Hidden Dependencies (Identifying the "API lie" where facades obscure actual dependency footprints)
6. Enhanced: Static Analysis Integration (Configuring PHPStan, Larastan, and IDE helpers)


## 1. Convenience (Rapid Prototyping and Readable, Expressive Syntax)

### Definitions

**Core Definition:** Convenience in the context of facades refers to the terse, expressive syntax that facades provide, allowing developers to access services with minimal boilerplate compared to constructor injection.

**Technical Definition:** Facades eliminate the need to import, type-hint, and store service dependencies in a class's constructor. A single `use Illuminate\Support\Facades\Cache;` statement enables static-like calls (`Cache::get('key')`) anywhere in the file, without requiring the class to declare `Cache` as a dependency. This reduces the lines of code per class and accelerates development, particularly during prototyping or for utility operations. However, the convenience comes at the cost of explicitness—the dependency is not visible in the class's API.

**Beginner-Friendly Explanation:** Facades are like using a microwave—you press a button and it works. Dependency Injection is like cooking from scratch—you gather ingredients, measure them, and combine them. The microwave is faster and simpler for quick tasks, but cooking from scratch gives you more control and makes it clear what's in your meal. For quick prototyping or small scripts, facades are a huge time-saver.

### Purposes

- To reduce boilerplate code when accessing framework services.
- To accelerate prototyping and rapid development.
- To provide a readable, expressive syntax for common operations.
- To allow quick access to services without full DI setup.
- To improve code density for utility operations.
- To lower the barrier to entry for developers new to Laravel.
- To enable one-liner service access in routes and closures.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Approach 1: Facade (convenient, terse)

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;

class ProductController extends Controller
{
    public function index()
    {
        // One-line cache and log operations
        $products = Cache::remember('products.all', 3600, fn () => \App\Models\Product::all()->toArray());
        Log::info('Products retrieved', ['count' => count($products)]);

        return response()->json($products);
    }
}
```

```php
<?php
// Approach 2: Dependency Injection (explicit, more verbose)

namespace App\Http\Controllers;

use Illuminate\Contracts\Cache\Repository as Cache;
use Psr\Log\LoggerInterface;

class ProductController extends Controller
{
    public function __construct(
        protected Cache $cache,
        protected LoggerInterface $logger
    ) {}

    public function index()
    {
        $products = $this->cache->remember('products.all', 3600, fn () => \App\Models\Product::all()->toArray());
        $this->logger->info('Products retrieved', ['count' => count($products)]);

        return response()->json($products);
    }
}
```

**Component Breakdown:**

| Aspect | Facade | Dependency Injection |
|--------|--------|----------------------|
| Imports | `use Illuminate\Support\Facades\Cache;` | `use Illuminate\Contracts\Cache\Repository;` |
| Class Signature | No constructor changes | Constructor type-hints |
| Usage | `Cache::get()` | `$this->cache->get()` |
| Lines of Code | Fewer | More |
| Explicitness | Hidden | Visible |

#### Syntax Rules

1. Facades **must** be imported via `use` statements in namespaced files.
2. Facades **may** be used without modifying the class constructor.
3. DI **requires** constructor or method type-hints and assignment to properties.
4. Facades **should not** be used in `config` files or during bootstrap.
5. DI **should** be preferred for classes with complex dependencies or business logic.
6. Facades **may** be used in routes, closures, and one-off operations.
7. Both approaches resolve the same service from the container.

#### Constraints and Limitations

- **Hidden Dependencies:** Facades make it harder to see what a class actually depends on.
- **Global State:** Facades rely on the global container, which can complicate testing and reuse.
- **Overuse:** Relying too heavily on facades can lead to "facade soup"—classes with many hidden dependencies.
- **Portability:** Facade-based code is harder to reuse outside of Laravel.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Prototyping with Facades

**Step-by-Step Setup Guide:**

1. Create a route closure that uses facades.
2. Cache a value, log a message, and query the database—all with facades.
3. Compare with the DI version.

**Complete Executable Code:**

```php
<?php
// routes/web.php — Facade approach (concise)

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/stats', function () {
    $stats = Cache::remember('stats', 600, function () {
        return DB::table('orders')->selectRaw('COUNT(*) as count, SUM(total) as revenue')->first();
    });

    Log::info('Stats retrieved', ['count' => $stats->count]);

    return response()->json($stats);
});
```

```php
<?php
// routes/web.php — DI approach (more verbose, requires a controller)

namespace App\Http\Controllers;

use Illuminate\Contracts\Cache\Repository as Cache;
use Illuminate\Database\DatabaseManager;
use Psr\Log\LoggerInterface;

class StatsController extends Controller
{
    public function __construct(
        protected Cache $cache,
        protected DatabaseManager $db,
        protected LoggerInterface $logger
    ) {}

    public function index()
    {
        $stats = $this->cache->remember('stats', 600, function () {
            return $this->db->table('orders')
                ->selectRaw('COUNT(*) as count, SUM(total) as revenue')
                ->first();
        });

        $this->logger->info('Stats retrieved', ['count' => $stats->count]);

        return response()->json($stats);
    }
}
```

**Expected Output:**

- Both approaches return `{"count": 100, "revenue": 5000.00}`.
- The facade version is a single closure; the DI version requires a dedicated controller class.

**Why This Code Produces That Result:**

- Facades resolve services from the container on first call.
- DI injects the same services but through explicit constructor parameters.
- Both produce identical results; the difference is in code structure and explicitness.

### Real-World Cases

**Case 1: Prototyping an MVP**

A startup uses facades extensively during the first sprint to move quickly, then refactors to DI as the codebase stabilizes.

**Case 2: Route Closures**

Simple routes use facades in closures, avoiding the overhead of creating a controller.

**Case 3: Artisan Commands**

A quick Artisan command uses facades for logging and caching without the ceremony of DI.

### References

- Laravel Facades: Facades vs. Dependency Injection - https://laravel.com/framework/docs/12.x/facades#facades-vs-dependency-injection
- Laravel Facades: Facades vs. Helper Functions - https://laravel.com/docs/5.x/facades#facades-vs-helper-functions
- Stack Overflow: What are the benefits of dependency injection over facades? - https://stackoverflow.com/questions/30344364/what-are-the-benefits-of-using-dependency-injection-instead-of-facades-in-laravel


## 2. Testability (Leveraging Built-In Static Mocking Frameworks Without Traditional Dependency Mocking Overhead)

### Definitions

**Core Definition:** Testability refers to how easily a class or component can be isolated and tested. Facades provide a built-in mocking mechanism via `shouldReceive()`, while DI relies on the service container or constructor injection to swap dependencies with mocks.

**Technical Definition:** Laravel's `Facade::shouldReceive()` method internally calls `swap()` to replace the cached facade instance with a Mockery mock. This allows tests to define expectations without constructing the class under test with mocked dependencies. In DI, tests use `$this->mock(Interface::class)` to bind a mock into the container, which is then injected into the class under test. Both approaches use Mockery, but facades integrate mocking more directly because of the static nature of the call. Laravel's `TestCase` provides `mock()`, `spy()`, and `instance()` for DI-based mocking, and `Facade::shouldReceive()` for facade-based mocking.

**Beginner-Friendly Explanation:** Testing with facades is like using a universal remote with programmable buttons—you press "Mock" and the facade starts pretending to be the real service. Testing with DI is like swapping a real battery for a fake one—you take the real one out and put the fake one in its place. Both work, but the facade approach is often quicker to set up because you don't need to rebuild the object.

### Purposes

- To isolate the unit under test from external dependencies.
- To verify that dependencies are called with the correct arguments.
- To simulate edge cases (errors, timeouts) without real services.
- To speed up tests by replacing slow operations with mocks.
- To ensure tests are deterministic and repeatable.
- To avoid hitting external APIs, databases, or file systems.
- To test error handling without triggering real failures.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Approach 1: Testing with facade mocking

namespace Tests\Feature;

use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class ProductControllerTest extends TestCase
{
    public function test_index_uses_cache()
    {
        // Configure the facade mock
        Cache::shouldReceive('remember')
            ->once()
            ->with('products.all', 3600, \Mockery::any())
            ->andReturn([['id' => 1, 'name' => 'Product']]);

        $response = $this->get('/products');

        $response->assertJson([['id' => 1, 'name' => 'Product']]);
    }
}
```

```php
<?php
// Approach 2: Testing with DI mocking

namespace Tests\Feature;

use Illuminate\Contracts\Cache\Repository as Cache;
use Tests\TestCase;

class ProductControllerTest extends TestCase
{
    public function test_index_uses_cache()
    {
        // Mock the interface and bind it into the container
        $cache = $this->mock(Cache::class);
        $cache->shouldReceive('remember')
            ->once()
            ->with('products.all', 3600, \Mockery::any())
            ->andReturn([['id' => 1, 'name' => 'Product']]);

        $response = $this->get('/products');

        $response->assertJson([['id' => 1, 'name' => 'Product']]);
    }
}
```

**Component Breakdown:**

| Aspect | Facade Mocking | DI Mocking |
|--------|---------------|------------|
| Setup | `Cache::shouldReceive()` | `$this->mock(Cache::class)` |
| Binding | Implicit (facade root swap) | Explicit (container binding) |
| Reset | Automatic per test | Automatic per test |
| Coupling | Coupled to facade | Coupled to interface |
| Complexity | Lower (fewer lines) | Higher (more explicit) |

#### Syntax Rules

1. Facade mocks **must** be configured with `shouldReceive()` before the code under test runs.
2. `shouldReceive()` **must** be called on the facade class.
3. DI mocks **must** be bound into the container via `$this->mock()` or `$this->instance()`.
4. Facade mocks **automatically** replace the cached instance for the test.
5. DI mocks **must** be type-compatible with the interface the class expects.
6. Both approaches **must** reset between tests; Laravel's `TestCase` handles this automatically.
7. `shouldReceive()` **may** chain `once()`, `with()`, `andReturn()`, `andThrow()`.

#### Constraints and Limitations

- **Facade Mock Scope:** Facade mocks apply globally for the test; mocking the same facade with different expectations in one test is complex.
- **DI Mock Explicit:** DI mocks require explicit binding and are more verbose.
- **Facade Coupling:** Facade mocking couples tests to the facade, not the interface; changing to DI requires test changes.
- **Mockery Dependency:** Both approaches use Mockery; developers must understand Mockery's API.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Facade Mocking in a Feature Test

```php
<?php
// tests/Feature/PaymentTest.php

namespace Tests\Feature;

use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class PaymentTest extends TestCase
{
    public function test_payment_logs_success()
    {
        // Mock the Log facade
        Log::shouldReceive('info')
            ->once()
            ->with('Payment succeeded', ['amount' => 99.99]);

        // Mock the Cache facade
        Cache::shouldReceive('put')
            ->once()
            ->with('last_payment', 99.99, 3600);

        // Execute the code under test
        $response = $this->post('/checkout', ['amount' => 99.99]);

        $response->assertStatus(200);
    }
}
```

**Expected Output:**

- The test passes, verifying that `Log::info()` and `Cache::put()` were called with the correct arguments.
- No real logging or caching occurs.

**Why This Code Produces That Result:**

- `Log::shouldReceive()` and `Cache::shouldReceive()` swap the cached facade instances with mocks.
- The mocks record and verify the calls.
- The test is isolated from real logging and caching.

#### Example 2: DI Mocking in a Unit Test

```php
<?php
// tests/Unit/ProductServiceTest.php

namespace Tests\Unit;

use App\Contracts\ProductRepository;
use App\Services\ProductService;
use Tests\TestCase;

class ProductServiceTest extends TestCase
{
    public function test_get_featured_products()
    {
        // Mock the repository interface
        $repo = $this->mock(ProductRepository::class);
        $repo->shouldReceive('featured')
            ->once()
            ->andReturn([['id' => 1, 'name' => 'Featured Product']]);

        // Resolve the service — the container injects the mock
        $service = app(ProductService::class);
        $result = $service->getFeatured();

        $this->assertCount(1, $result);
        $this->assertEquals('Featured Product', $result[0]['name']);
    }
}
```

**Expected Output:**

- The test passes, verifying the service uses the repository as expected.
- No database access occurs.

**Why This Code Produces That Result:**

- `$this->mock(ProductRepository::class)` binds a mock into the container.
- The service resolves the mock via constructor injection.
- The service's logic is tested in isolation.

### Real-World Cases

**Case 1: Testing Payment Flows**

A checkout test mocks the `PaymentGateway` facade to simulate successful and failed charges without hitting Stripe.

**Case 2: Testing Notification Logic**

A notification test mocks the `Notification` facade to verify the correct message is sent.

**Case 3: Testing Repository Queries**

A repository test mocks the `DB` facade to verify query construction without executing SQL.

### References

- Laravel Facades: Testing Facades - https://laravel.com/docs/12.x/facades#testing-facades
- Laravel Testing: Mocking - https://laravel.com/docs/12.x/mocking
- Laravel API: Facade::shouldReceive - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html#method_shouldReceive


## 3. Coupling (Balancing Tight Coupling to Global State Against the Flexibility of Constructor Parameter Abstraction)

### Definitions

**Core Definition:** Coupling in the facades vs. DI debate refers to the degree to which a class depends on external systems. Facades create tight coupling to Laravel's global container and facade system; DI creates loose coupling through abstract interfaces passed as constructor parameters.

**Technical Definition:** Facade-based code depends on the global `Facade` class and the service container's static `$app` instance. This coupling means the class cannot be instantiated or tested outside a Laravel application without significant setup. DI-based code depends only on its declared interfaces, making it portable across frameworks and testable in isolation. The tradeoff is convenience vs. portability—facades are convenient but coupled; DI is verbose but decoupled. The SOLID Dependency Inversion Principle favors DI because high-level modules should not depend on low-level details (the container) but on abstractions (interfaces).

**Beginner-Friendly Explanation:** Coupling is like the difference between living in a company town (facades) and owning your own car (DI). In a company town, everything is convenient—the store, the school, and the doctor are all right there. But if the company closes, you're stuck. With your own car, you can drive anywhere, but you have to maintain it yourself. Facades are convenient but tie you to Laravel. DI gives you freedom but requires more setup.

### Purposes

- To understand the tradeoff between convenience and portability.
- To make informed architectural decisions about when to use facades vs. DI.
- To minimize coupling in domain-critical code.
- To maximize convenience in framework-specific code.
- To ensure code is testable in isolation when needed.
- To follow the Dependency Inversion Principle in domain layers.
- To document coupling decisions for team alignment.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Tight coupling: Facade-based code

namespace App\Services;

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;

class OrderService
{
    public function place(array $data): array
    {
        // Depends on the global Cache and Log facades
        Cache::put('last_order', $data, 3600);
        Log::info('Order placed', $data);

        return $data;
    }
}
```

```php
<?php
// Loose coupling: DI-based code

namespace App\Services;

use Psr\Log\LoggerInterface;
use Illuminate\Contracts\Cache\Repository as Cache;

class OrderService
{
    public function __construct(
        protected Cache $cache,
        protected LoggerInterface $logger
    ) {}

    public function place(array $data): array
    {
        // Depends only on abstractions
        $this->cache->put('last_order', $data, 3600);
        $this->logger->info('Order placed', $data);

        return $data;
    }
}
```

**Component Breakdown:**

| Aspect | Facade (Tight Coupling) | DI (Loose Coupling) |
|--------|------------------------|---------------------|
| Dependency | Global container | Constructor parameters |
| Portability | Laravel-only | Any framework |
| Testing | Requires Laravel app | Requires only mocks |
| Setup | Minimal | More verbose |
| SOLID Compliance | Violates DIP | Follows DIP |

#### Syntax Rules

1. Facade-based classes **must** be run within a Laravel application to function.
2. DI-based classes **may** be instantiated outside Laravel with manual dependency injection.
3. Facades **must** be resolved from the global container; there is no alternative.
4. DI **allows** alternative containers (e.g., PHP-DI, Symfony DI) to be used.
5. Coupling **should** be minimized in domain layers and maximized in infrastructure layers.
6. Facades **should** be avoided in reusable packages; DI is preferred.
7. Coupling decisions **should** be documented in the codebase's architecture guide.

#### Constraints and Limitations

- **Global State:** Facades rely on the static `$app` instance, which complicates parallel testing.
- **Portability:** Facade-based code cannot be reused outside Laravel without refactoring.
- **Test Isolation:** DI-based classes can be tested without booting Laravel; facade-based classes cannot.
- **Overhead:** DI adds boilerplate, which some teams find excessive for small applications.
- **Team Preferences:** Coupling decisions often depend on team culture and project size.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Comparing Coupling in a Domain Service

```php
<?php
// Tight coupling: Facade-based domain service (NOT ideal for domain layer)

namespace App\Domain\Orders;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

class OrderProcessor
{
    public function process(int $orderId): void
    {
        DB::transaction(function () use ($orderId) {
            Log::info("Processing order {$orderId}");
            // Business logic...
        });
    }
}
```

```php
<?php
// Loose coupling: DI-based domain service (ideal for domain layer)

namespace App\Domain\Orders;

use Psr\Log\LoggerInterface;
use App\Contracts\TransactionManager;

class OrderProcessor
{
    public function __construct(
        protected TransactionManager $transactions,
        protected LoggerInterface $logger
    ) {}

    public function process(int $orderId): void
    {
        $this->transactions->run(function () use ($orderId) {
            $this->logger->info("Processing order {$orderId}");
            // Business logic...
        });
    }
}
```

**Expected Output:**

- Both classes process orders.
- The facade version cannot be tested outside Laravel.
- The DI version can be unit tested with mock dependencies.

**Why This Code Produces That Result:**

- The facade version depends on Laravel's global facades.
- The DI version depends only on interfaces, making it portable.
- The DI version follows the Dependency Inversion Principle.

### Real-World Cases

**Case 1: Domain-Driven Design**

A DDD application uses DI for domain services (portable, testable) and facades for infrastructure concerns (logging, caching in controllers).

**Case 2: Reusable Packages**

A package author uses DI to ensure the package works across frameworks, avoiding facades.

**Case 3: Monolithic Applications**

A Laravel-only monolith uses facades throughout, accepting the coupling tradeoff for convenience.

### References

- Laravel Facades: Facades vs. Dependency Injection - https://laravel.com/framework/docs/12.x/facades#facades-vs-dependency-injection
- SOLID Principles: Dependency Inversion - https://en.wikipedia.org/wiki/Dependency_inversion_principle
- Laravel News: Advanced Application Architecture - https://laravel-news.com/service-container-management
- Matheus Lima: Service Container & Dependency Injection - https://matheuslima.medium.com/service-container-dependency-injection-55321c5dfb91


## 4. Architectural Considerations (Establishing Team Guardrails on When to Use Facades in Microservices Versus Clean Domain-Driven Architecture Layers)

### Definitions

**Core Definition:** Architectural considerations in the facades vs. DI debate refer to the conventions, rules, and boundaries teams establish to decide when each approach is appropriate, balancing convenience against maintainability across different architectural styles.

**Technical Definition:** In a microservices architecture, each service is independently deployable and may use different frameworks or languages. Facades are Laravel-specific and hinder portability, so DI is preferred for inter-service communication, domain logic, and adapters. In a clean architecture (Domain-Driven Design, Hexagonal Architecture), the domain layer must remain framework-agnostic, so DI is mandatory in the domain and application layers. Facades are acceptable in the infrastructure and presentation layers, where Laravel is already a hard dependency. Teams codify these decisions in architecture decision records (ADRs), static analysis rules, and code review guidelines.

**Beginner-Friendly Explanation:** Think of your application as a house with different rooms. The domain layer (living room) is where the family spends time—it should be comfortable and not depend on any specific furniture brand. The infrastructure layer (utility room) is where the plumbing and electrical systems live—it's fine for those to be specific to the house. Facades are like built-in appliances (convenient but hard to move); DI is like furniture (portable and replaceable). You put built-in appliances in the utility room, not the living room.

### Purposes

- To establish clear rules for when facades are acceptable.
- To keep domain logic framework-agnostic.
- To ensure microservices are independently deployable.
- To make codebases maintainable as they grow.
- To facilitate onboarding by documenting conventions.
- To enable testing at every layer.
- To prevent architectural drift over time.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Domain layer — DI ONLY (no facades)

namespace App\Domain\Orders;

use App\Domain\Contracts\OrderRepository;
use Psr\Log\LoggerInterface;

class OrderService
{
    public function __construct(
        protected OrderRepository $orders,
        protected LoggerInterface $logger
    ) {}

    public function place(array $data): Order
    {
        $order = $this->orders->create($data);
        $this->logger->info('Order placed', ['id' => $order->id]);
        return $order;
    }
}
```

```php
<?php
// Application layer — DI PREFERRED, facades acceptable for framework glue

namespace App\Application\Orders;

use App\Domain\Orders\OrderService;
use Illuminate\Support\Facades\DB;

class PlaceOrderHandler
{
    public function __construct(
        protected OrderService $orders
    ) {}

    public function handle(array $data): array
    {
        return DB::transaction(function () use ($data) {
            $order = $this->orders->place($data);
            return ['id' => $order->id];
        });
    }
}
```

```php
<?php
// Infrastructure layer — facades ACCEPTABLE

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;

class OrderController extends Controller
{
    public function index()
    {
        $orders = Cache::remember('orders.all', 3600, fn () => Order::all()->toArray());
        Log::info('Orders listed');
        return response()->json($orders);
    }
}
```

**Component Breakdown:**

| Layer | Facades | DI | Rationale |
|-------|---------|-----|-----------|
| Domain | ❌ Forbidden | ✅ Required | Framework-agnostic |
| Application | ⚠️ Limited | ✅ Preferred | Some framework glue OK |
| Infrastructure | ✅ Acceptable | ✅ Acceptable | Laravel is a hard dependency |
| Presentation (HTTP) | ✅ Acceptable | ✅ Acceptable | Controller-specific concerns |

#### Syntax Rules

1. Domain layer classes **must not** use facades; they must use DI.
2. Application layer classes **may** use facades for framework glue (transactions, events).
3. Infrastructure layer classes **may** use facades freely.
4. Presentation layer (controllers, middleware) **may** use facades freely.
5. Teams **must** document their conventions in an architecture guide or ADR.
6. Static analysis tools **should** enforce the rules (e.g., PHPStan rules).
7. Code reviews **should** verify adherence to the conventions.

#### Constraints and Limitations

- **Overhead:** Strict rules add friction to development; teams must balance rigor with velocity.
- **Learning Curve:** New developers must learn the conventions.
- **Tooling:** Enforcing rules requires static analysis or custom linters.
- **Legacy Code:** Existing codebases may violate the rules, requiring gradual refactoring.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Clean Architecture Layers with Facade Boundaries

```php
<?php
// Domain layer: no facades

namespace App\Domain\Billing;

interface PaymentGateway
{
    public function charge(float $amount): bool;
}

class Invoice
{
    public function __construct(
        public int $id,
        public float $amount,
        public string $status = 'pending'
    ) {}
}

class BillingService
{
    public function __construct(
        protected PaymentGateway $gateway
    ) {}

    public function pay(Invoice $invoice): bool
    {
        if ($this->gateway->charge($invoice->amount)) {
            $invoice->status = 'paid';
            return true;
        }
        return false;
    }
}
```

```php
<?php
// Infrastructure layer: facades acceptable

namespace App\Infrastructure\Billing;

use App\Domain\Billing\PaymentGateway;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class StripeGateway implements PaymentGateway
{
    public function charge(float $amount): bool
    {
        Log::info('Charging via Stripe', ['amount' => $amount]);

        $response = Http::post('https://api.stripe.com/v1/charges', [
            'amount' => $amount * 100,
            'currency' => 'usd',
        ]);

        return $response->successful();
    }
}
```

```php
<?php
// Application layer: facades acceptable for transactions

namespace App\Application\Billing;

use App\Domain\Billing\BillingService;
use App\Domain\Billing\Invoice;
use Illuminate\Support\Facades\DB;

class PayInvoiceHandler
{
    public function __construct(
        protected BillingService $billing
    ) {}

    public function handle(Invoice $invoice): bool
    {
        return DB::transaction(function () use ($invoice) {
            return $this->billing->pay($invoice);
        });
    }
}
```

**Expected Output:**

- The domain layer has no Laravel dependencies.
- The infrastructure layer uses facades for HTTP and logging.
- The application layer uses the `DB` facade for transactions.
- Each layer respects its boundary.

**Why This Code Produces That Result:**

- The domain layer depends only on interfaces.
- The infrastructure layer implements domain interfaces using Laravel features.
- The application layer orchestrates domain and infrastructure with some framework glue.

### Real-World Cases

**Case 1: Microservices Architecture**

A microservices platform uses DI in domain services and facades only in the HTTP layer, keeping services portable.

**Case 2: Domain-Driven Design Monolith**

A DDD monolith uses DI throughout the domain and application layers, with facades allowed only in controllers and infrastructure.

**Case 3: Laravel-Only Application**

A Laravel-only application uses facades throughout, accepting the coupling for maximum convenience.

### References

- Laravel Facades: Facades vs. Dependency Injection - https://laravel.com/framework/docs/12.x/facades#facades-vs-dependency-injection
- GitHub: Laravel Axiom — Architectural Constraints - https://github.com/laravel-axiom/laravel-axiom
- Spatie: Laravel Beyond CRUD - https://spatie.be/products/laravel-beyond-crud
- Matheus Lima: Service Container & Dependency Injection - https://matheuslima.medium.com/service-container-dependency-injection-55321c5dfb91


## 5. Enhanced: Hidden Dependencies (Identifying the "API Lie" Where Facades Obscure a Class's Actual Dependency Footprint)

### Definitions

**Core Definition:** Hidden dependencies occur when a class uses facades internally, so its constructor signature does not reveal what services it actually depends on. This is sometimes called the "API lie" because the class's public API misrepresents its true dependencies.

**Technical Definition:** When a class uses `Cache::get()`, `Log::info()`, or `DB::table()` internally, those dependencies are not declared in the constructor. A developer reading the constructor sees no dependencies, but the class actually requires the cache, logger, and database to function. This violates the Dependency Inversion Principle and makes the class harder to test, refactor, and reason about. The "API lie" is that the constructor's signature suggests the class is dependency-free, when it is not. DI solves this by requiring every dependency to be declared explicitly in the constructor or method signature.

**Beginner-Friendly Explanation:** Imagine ordering a pizza and the menu says "cheese pizza" (no mention of anchovies). When it arrives, it's covered in anchovies. That's the "API lie"—the menu (constructor) didn't tell you what was actually in the pizza (class). Facades are like secret ingredients that don't appear on the menu. DI is like a menu that lists every ingredient up front. You always know what you're getting.

### Purposes

- To identify classes with hidden dependencies.
- To make dependencies explicit in the constructor signature.
- To improve testability by making mocks obvious.
- To facilitate refactoring by revealing what a class actually needs.
- To follow the Dependency Inversion Principle.
- To improve code readability and maintainability.
- To prevent "surprise" dependencies in domain logic.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Hidden dependencies (the "API lie")

namespace App\Services;

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\DB;

class ReportService
{
    /**
     * The constructor reveals NO dependencies.
     * But the class actually needs Cache, Log, and DB.
     */
    public function __construct() {}

    public function generate(int $reportId): array
    {
        Log::info("Generating report {$reportId}");

        $data = Cache::remember("report.{$reportId}", 3600, function () use ($reportId) {
            return DB::table('reports')->find($reportId);
        });

        return (array) $data;
    }
}
```

```php
<?php
// Explicit dependencies (no API lie)

namespace App\Services;

use Illuminate\Contracts\Cache\Repository as Cache;
use Illuminate\Database\DatabaseManager;
use Psr\Log\LoggerInterface;

class ReportService
{
    /**
     * The constructor reveals ALL dependencies.
     * A developer immediately knows what this class needs.
     */
    public function __construct(
        protected Cache $cache,
        protected DatabaseManager $db,
        protected LoggerInterface $logger
    ) {}

    public function generate(int $reportId): array
    {
        $this->logger->info("Generating report {$reportId}");

        $data = $this->cache->remember("report.{$reportId}", 3600, function () use ($reportId) {
            return $this->db->table('reports')->find($reportId);
        });

        return (array) $data;
    }
}
```

**Component Breakdown:**

| Aspect | Hidden Dependencies | Explicit Dependencies |
|--------|--------------------|-----------------------|
| Constructor | Empty | Reveals all dependencies |
| API Honesty | Lies about dependencies | Truthful |
| Testability | Hard to know what to mock | Obvious what to mock |
| Refactoring | Risky (unknown dependencies) | Safe (known dependencies) |
| SOLID Compliance | Violates DIP | Follows DIP |

#### Syntax Rules

1. A class's constructor **should** declare every service it depends on.
2. Facades used inside methods **should** be replaced with constructor-injected interfaces.
3. Dependencies **should** be declared as interfaces, not concrete classes.
4. The "API lie" **should** be avoided in domain and application layers.
5. Facades **may** be used in controllers and infrastructure, where the lie is acceptable.
6. Static analysis tools **may** detect hidden dependencies via custom rules.
7. Code reviews **should** flag classes with empty constructors that use facades.

#### Constraints and Limitations

- **Refactoring Cost:** Refactoring facade-based classes to DI requires significant changes.
- **Verbosity:** Explicit dependencies make constructors longer.
- **Legacy Code:** Existing codebases may have many "API lies" that are expensive to fix.
- **Team Buy-In:** Developers used to facades may resist the extra boilerplate.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Identifying Hidden Dependencies

```php
<?php
// Before: hidden dependencies

class NotificationService
{
    public function send(string $message, string $recipient): void
    {
        \Log::info("Sending notification", ['to' => $recipient]);
        \Mail::raw($message, fn ($m) => $m->to($recipient));
        \Cache::put("last_notification.{$recipient}", now(), 3600);
    }
}

// The constructor lies: it shows no dependencies,
// but the class needs Log, Mail, and Cache.
```

```php
<?php
// After: explicit dependencies

use Illuminate\Contracts\Cache\Repository as Cache;
use Illuminate\Contracts\Mail\Mailer;
use Psr\Log\LoggerInterface;

class NotificationService
{
    public function __construct(
        protected Mailer $mailer,
        protected Cache $cache,
        protected LoggerInterface $logger
    ) {}

    public function send(string $message, string $recipient): void
    {
        $this->logger->info("Sending notification", ['to' => $recipient]);
        $this->mailer->raw($message, fn ($m) => $m->to($recipient));
        $this->cache->put("last_notification.{$recipient}", now(), 3600);
    }
}

// The constructor now honestly reveals all dependencies.
```

**Expected Output:**

- Both classes send notifications.
- The refactored class's constructor reveals all three dependencies.
- Tests can now mock all three dependencies explicitly.

**Why This Code Produces That Result:**

- The refactored class replaces facades with DI.
- The constructor signature honestly reflects the class's needs.
- Tests are more explicit and isolated.

### Real-World Cases

**Case 1: Refactoring Legacy Code**

A team refactors a legacy service that used facades internally; the new version uses DI, making tests faster and more isolated.

**Case 2: Onboarding New Developers**

A new developer reads a class's constructor and immediately understands what it needs, thanks to explicit dependencies.

**Case 3: Static Analysis Enforcement**

A team configures PHPStan to flag classes with empty constructors that use facades, catching hidden dependencies early.

### References

- Laravel Facades: Facades vs. Dependency Injection - https://laravel.com/framework/docs/12.x/facades#facades-vs-dependency-injection
- Laravel Daily: Hidden Dependencies - https://laraveldaily.com/lesson/api-laravel-hidden-dependencies
- GitHub: Laravel Axiom - https://github.com/laravel-axiom/laravel-axiom
- Matheus Lima: Service Container & Dependency Injection - https://matheuslima.medium.com/service-container-dependency-injection-55321c5dfb91


## 6. Enhanced: Static Analysis Integration (Configuring Tools Like PHPStan or Larastan Alongside IDE Helpers to Handle Static Facade Signature Validation Accurately)

### Definitions

**Core Definition:** Static analysis integration refers to configuring tools like PHPStan (with Larastan) and IDE helpers (e.g., Laravel Idea, `barryvdh/laravel-ide-helper`) to correctly recognize facade method signatures, return types, and static calls, enabling accurate type checking and autocompletion.

**Technical Definition:** PHPStan is a static analysis tool for PHP that detects type errors, undefined methods, and other issues without running the code. Larastan is a PHPStan extension that understands Laravel's magic, including facades, service container resolution, and Eloquent models. Without Larastan, PHPStan reports false positives for facade calls (e.g., "Call to an undefined static method Cache::get()"). Larastan resolves the facade's underlying class and its method signatures, enabling accurate analysis. IDE helpers generate PHPDoc annotations for facades and models, providing autocompletion in editors.

**Beginner-Friendly Explanation:** Static analysis is like a spell-checker for code. PHPStan and Larastan check your code for errors before you run it. But facades look "magic" to PHPStan—it doesn't know that `Cache::get()` is real. Larastan is like a Laravel dictionary that teaches PHPStan all the Laravel-specific words. IDE helpers do the same for your editor, so you get autocompletion and type hints when using facades.

### Purposes

- To catch type errors and undefined methods before runtime.
- To enable accurate static analysis of facade calls.
- To provide IDE autocompletion and type hints for facades.
- To enforce architectural rules (e.g., no facades in domain layer).
- To improve code quality and reduce bugs.
- To speed up development with better tooling.
- To integrate with CI/CD pipelines for automated checks.

### Syntax Rules and Structure

#### Complete General Syntax

```json
// composer.json — Install Larastan and IDE Helper

{
    "require-dev": {
        "larastan/larastan": "^2.0",
        "barryvdh/laravel-ide-helper": "^3.0",
        "nunomaduro/larastan": "^2.0"
    }
}
```

```yaml
# phpstan.neon — PHPStan configuration with Larastan

includes:
    - vendor/larastan/larastan/extension.neon

parameters:
    paths:
        - app/
    level: 6
    excludePaths:
        - app/Console/Kernel.php
    checkMissingIterableValueType: false
    checkGenericClassInNonGenericObjectType: false
    ignoreErrors:
        - '#Call to an undefined method Illuminate\\Database\\Eloquent\\Builder::#'

    # Larastan-specific settings
    universal_object_crates:
        - Carbon\Carbon
        - Illuminate\Support\Collection
```

```bash
# Generate IDE helper files
php artisan ide-helper:generate
php artisan ide-helper:models
php artisan ide-helper:meta

# Run PHPStan with Larastan
./vendor/bin/phpstan analyse
```

```php
<?php
// app/Services/ReportService.php — Static analysis-friendly code

namespace App\Services;

use Illuminate\Contracts\Cache\Repository as Cache;
use Psr\Log\LoggerInterface;

class ReportService
{
    public function __construct(
        protected Cache $cache,
        protected LoggerInterface $logger
    ) {}

    /**
     * @return array<string, mixed>
     */
    public function generate(int $reportId): array
    {
        $this->logger->info("Generating report {$reportId}");

        /** @var array<string, mixed> $data */
        $data = $this->cache->remember("report.{$reportId}", 3600, function () use ($reportId) {
            return ['id' => $reportId, 'data' => 'example'];
        });

        return $data;
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `phpstan.neon` | PHPStan configuration file |
| `larastan/extension.neon` | Larastan extension for PHPStan |
| `level: 6` | Strictness level (0-9) |
| `universal_object_crates` | Classes treated as universally compatible |
| `ide-helper:generate` | Generates facade autocompletion |
| `ide-helper:meta` | Generates container binding metadata |
| `@var` annotations | Explicit type hints for facades |

#### Syntax Rules

1. Larastan **must** be included in `phpstan.neon` via `extension.neon`.
2. The `paths` parameter **must** list the directories to analyze.
3. The `level` parameter **must** be set between 0 and 9 (higher = stricter).
4. IDE helper **must** be run after adding new facades or models.
5. `_ide_helper.php` **should** be added to `.gitignore` (regenerated per developer).
6. PHPStan **should** be run in CI/CD pipelines for automated checks.
7. Custom rules **may** be added to enforce architectural conventions (e.g., no facades in domain layer).

#### Constraints and Limitations

- **Performance:** PHPStan can be slow on large codebases; caching helps.
- **False Positives:** Larastan reduces but does not eliminate false positives.
- **Version Compatibility:** Larastan versions must match PHPStan and Laravel versions.
- **IDE Helper Staleness:** `_ide_helper.php` must be regenerated after adding facades or models.
- **Learning Curve:** Configuring PHPStan at higher levels requires understanding of type theory.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Configuring PHPStan with Larastan

**Step-by-Step Setup Guide:**

1. Install Larastan via Composer.
2. Create `phpstan.neon` with the Larastan extension.
3. Run PHPStan on the `app/` directory.
4. Review and fix reported errors.

**Complete Executable Code:**

```bash
# Install Larastan
composer require --dev larastan/larastan

# Create phpstan.neon
cat > phpstan.neon << 'EOF'
includes:
    - vendor/larastan/larastan/extension.neon

parameters:
    paths:
        - app/
    level: 6
EOF

# Run PHPStan
./vendor/bin/phpstan analyse
```

**Expected Output:**

- PHPStan analyzes the `app/` directory at level 6.
- Facade calls are recognized (no false positives).
- Type errors, undefined methods, and other issues are reported.

**Why This Code Produces That Result:**

- Larastan teaches PHPStan about Laravel's magic.
- Facade calls like `Cache::get()` are resolved to their underlying methods.
- The analysis catches real issues without false positives.

#### Example 2: IDE Helper for Facades

```bash
# Install IDE Helper
composer require --dev barryvdh/laravel-ide-helper

# Generate facade autocompletion
php artisan ide-helper:generate

# Generate model metadata
php artisan ide-helper:models --write

# Generate container metadata
php artisan ide-helper:meta
```

**Expected Output:**

- `_ide_helper.php` is generated with PHPDoc annotations for all facades.
- `_ide_helper_models.php` is generated with model property annotations.
- `.phpstorm.meta.php` is generated with container binding metadata.
- IDEs (PhpStorm, VS Code with Intelephense) provide autocompletion for facade methods.

**Why This Code Produces That Result:**

- IDE Helper generates PHPDoc blocks that describe facade methods and their signatures.
- The IDE reads these annotations and provides autocompletion and type hints.
- The `_ide_helper.php` file should be added to `.gitignore`.

### Real-World Cases

**Case 1: CI/CD Pipeline**

A team runs PHPStan at level 8 in their CI pipeline, catching type errors before merging PRs.

**Case 2: Large Codebase Refactoring**

A team refactoring a large Laravel application uses Larastan to identify hidden dependencies and type issues.

**Case 3: Developer Onboarding**

New developers use IDE Helper to get autocompletion for facades and models, reducing the learning curve.

### References

- Larastan GitHub Repository - https://github.com/larastan/larastan
- PHPStan Documentation - https://phpstan.org/user-guide/getting-started
- Laravel IDE Helper GitHub Repository - https://github.com/barryvdh/laravel-ide-helper
- Laravel News: Larastan 3.0 - https://laravel-news.com/larastan-3-0
- PHPStan: Configuring Larastan - https://github.com/larastan/larastan


## Summary Table of Facades vs. Dependency Injection

| Aspect | Facades | Dependency Injection |
|--------|---------|---------------------|
| Syntax | `Cache::get()` | `$this->cache->get()` |
| Convenience | High | Moderate |
| Testability | `Cache::shouldReceive()` | `$this->mock(Cache::class)` |
| Coupling | Tight (global container) | Loose (interface) |
| Portability | Laravel-only | Framework-agnostic |
| Hidden Dependencies | Yes (API lie) | No (explicit) |
| Static Analysis | Requires Larastan | Native support |
| SOLID Compliance | Violates DIP | Follows DIP |
| Best For | Controllers, infrastructure, prototyping | Domain, application, packages |


## References

- Laravel Facades Documentation (12.x) - https://laravel.com/framework/docs/12.x/facades
- Laravel Facades: Facades vs. Dependency Injection - https://laravel.com/framework/docs/12.x/facades#facades-vs-dependency-injection
- Laravel Facades: Testing Facades - https://laravel.com/docs/12.x/facades#testing-facades
- Laravel Service Container Documentation - https://laravel.com/docs/12.x/container
- Laravel API: Facade Class - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html
- Laravel API: Facade::shouldReceive - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html#method_shouldReceive
- Laravel Daily: API — Laravel Hidden Dependencies - https://laraveldaily.com/lesson/api-laravel-hidden-dependencies
- Larastan GitHub Repository - https://github.com/larastan/larastan
- Laravel IDE Helper GitHub Repository - https://github.com/barryvdh/laravel-ide-helper
- PHPStan Documentation - https://phpstan.org/user-guide/getting-started
- Laravel News: Larastan 3.0 - https://laravel-news.com/larastan-3-0
- Spatie: Laravel Beyond CRUD - https://spatie.be/products/laravel-beyond-crud
- Matheus Lima: Service Container & Dependency Injection - https://matheuslima.medium.com/service-container-dependency-injection-55321c5dfb91
- GitHub: Laravel Axiom — Architectural Constraints - https://github.com/laravel-axiom/laravel-axiom
- Stack Overflow: What are the benefits of dependency injection over facades? - https://stackoverflow.com/questions/30344364/what-are-the-benefits-of-using-dependency-injection-instead-of-facades-in-laravel
- SOLID Principles: Dependency Inversion - https://en.wikipedia.org/wiki/Dependency_inversion_principle