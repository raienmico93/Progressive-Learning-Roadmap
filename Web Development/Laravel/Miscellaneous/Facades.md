# Laravel Facades: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Facades in Laravel provide a "static" interface to classes that are available in the application's service container, serving as "static proxies" to underlying classes while maintaining more testability and flexibility than traditional static methods.

**Technical Definition:** A Laravel Facade is a class that extends `Illuminate\Support\Facades\Facade` and implements a single method, `getFacadeAccessor()`, which returns a string key representing a service container binding. The base `Facade` class uses PHP's `__callStatic()` magic method to intercept static method calls, resolve the underlying service from the container, and forward the call to the resolved object instance. All facades are defined in the `Illuminate\Support\Facades` namespace and are resolved through the application's service container.

**Beginner-Friendly Explanation:** A Facade is like a TV remote control. Instead of walking to the TV and pressing buttons on the back panel (using the actual service class), you use the remote (the Facade) to change channels, adjust volume, and turn it on. The remote doesn't contain the TV's electronics—it just sends signals to the TV. Similarly, `Cache::get('key')` looks like a static method call, but behind the scenes, Laravel finds the actual cache service in the container and calls `get()` on it. It's a shorthand that makes code cleaner and easier to read.

### Key Characteristics

1. **Static-Looking Syntax:** Facades provide a terse, memorable syntax for accessing services without remembering long class names.
2. **Proxy Pattern:** Facades are static proxies that forward calls to underlying service instances resolved from the container.
3. **Single Method Implementation:** A facade class only needs to implement `getFacadeAccessor()` to define what to resolve from the container.
4. **PHP Magic Methods:** The base `Facade` class uses `__callStatic()` to defer calls to the resolved object.
5. **Service Container Integration:** Facades resolve their underlying instances through Laravel's service container.
6. **Testability:** Facades can be mocked and swapped for testing using `shouldReceive()`, `spy()`, `partialMock()`, and `swap()`.
7. **Real-Time Facades:** Any class can be used as a facade by prefixing its namespace with `Facades\`.
8. **Facade Root Caching:** Resolved instances are cached in the facade base to minimize container lookups.

### Prerequisites

- Laravel 10.x or higher (12.x recommended)
- PHP 8.1 or higher
- Composer package manager
- Familiarity with the Laravel Service Container
- Understanding of dependency injection and interfaces
- Basic knowledge of PHP magic methods (`__callStatic`)
- A working Laravel application

### Related Programming Areas

- **Service Container:** The DI container where facade-resolved services live.
- **Dependency Injection:** The alternative approach to accessing services.
- **Service Providers:** Where services are bound into the container.
- **Contracts:** The interfaces that facades proxy to.
- **Real-Time Facades:** A feature for using any class as a facade.
- **Testing and Mocking:** Facades support mocking for isolated tests.
- **Helper Functions:** Global functions that complement facades.

### Core Concepts / Features

1. Facade Concept (Providing a static proxy interface to underlying classes inside the service container)
2. Static-Looking Interfaces (Utilizing PHP's `__callStatic()` magic method to proxy calls dynamically)
3. Underlying Container Resolution (Overriding `getFacadeAccessor()` to return a string container key)
4. Common Laravel Facades (Deep diving into `Log`, `Cache`, `DB`, and `Route` usage)
5. Enhanced: Real-Time Facades (Prefixing class namespaces dynamically with `Facades\` to instantly generate a facade on the fly)
6. Enhanced: Facade Root Caching (Understanding how Laravel caches resolved instances inside the facade base to minimize container lookup overhead)


## 1. Facade Concept (Providing a Static Proxy Interface to Underlying Classes Inside the Service Container)

### Definitions

**Core Definition:** The facade concept is a design pattern in which a class provides a simplified, static-looking interface to a more complex underlying subsystem or service, hiding the complexity of instantiation, configuration, and interaction.

**Technical Definition:** In Laravel, a facade is a class that extends `Illuminate\Support\Facades\Facade` and acts as a "static proxy" to a service bound in the container. The facade class defines `getFacadeAccessor()`, which returns the container binding key. When a static method is called on the facade, the base `Facade` class's `__callStatic()` method resolves the underlying service from the container and forwards the call to it. In technical terms, Laravel Facades are a convenient syntax for using the Laravel service container as a service locator.

**Beginner-Friendly Explanation:** Think of a facade as a restaurant menu. The menu lists dishes (methods) you can order, but you don't need to know how the kitchen (underlying service) prepares them. You just say "I'll have the pasta" (call `Cache::get('key')`), and the waiter (facade) relays your order to the kitchen. The menu is a simple, static-looking interface to a complex behind-the-scenes operation.

### Purposes

- To provide a terse, memorable syntax for accessing complex services.
- To hide the complexity of service instantiation and configuration.
- To serve as a convenient syntax for using the service container as a service locator.
- To maintain testability through mockable static interfaces.
- To reduce boilerplate code compared to manual container resolution.
- To provide a consistent API across different service implementations.
- To complement dependency injection when constructor injection is impractical.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// 1. The facade class (e.g., Cache)

namespace Illuminate\Support\Facades;

class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache'; // The container binding key
    }
}
```

```php
<?php
// 2. Using the facade

use Illuminate\Support\Facades\Cache;

$value = Cache::get('key'); // Static-looking call
```

```php
<?php
// 3. What's really happening behind the scenes

$value = app('cache')->get('key'); // Equivalent container resolution
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `extends Facade` | Inherits `__callStatic()` proxy behavior |
| `getFacadeAccessor()` | Returns the container binding key |
| `return 'cache'` | The binding key for the cache service |
| `Cache::get()` | Static-looking method call |
| `app('cache')->get()` | Equivalent manual resolution |

#### Syntax Rules

1. A facade class **must** extend `Illuminate\Support\Facades\Facade`.
2. A facade class **must** implement `getFacadeAccessor()`, returning a string container binding key.
3. Facade methods **are not** defined on the facade class; they are proxied to the underlying service.
4. Facade calls **must** be imported via `use` statements in namespaced files.
5. The binding key returned by `getFacadeAccessor()` **must** match a binding registered in a service provider.
6. Facades **may** be used without dependency injection, though injection is often preferred for testability.

#### Constraints and Limitations

- **Scope Creep:** Facades are easy to use, which can lead to class bloat; keep classes focused.
- **Service Locator Anti-Pattern:** Facades use the container as a service locator, which can obscure dependencies.
- **Testing Complexity:** Facades require `shouldReceive()` for mocking, which is different from constructor injection mocks.
- **No IDE Autocompletion (Without Helpers):** Static calls on facades may not provide full IDE autocompletion without Laravel Idea or similar tools.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Basic Facade Usage with Cache

**Step-by-Step Setup Guide:**

1. Import the `Cache` facade.
2. Call static methods on it as if it were a static class.
3. Verify the result is the same as manual container resolution.

**Complete Executable Code:**

```php
<?php
// routes/web.php

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    // Store a value in the cache
    Cache::put('greeting', 'Hello, world!', 3600);

    // Retrieve the value
    $greeting = Cache::get('greeting');

    return response()->json(['greeting' => $greeting]);
});
```

```php
<?php
// Equivalent manual resolution

Route::get('/cache-manual', function () {
    $cache = app('cache');
    $cache->put('greeting', 'Hello, world!', 3600);
    $greeting = $cache->get('greeting');

    return response()->json(['greeting' => $greeting]);
});
```

**Expected Output:**

- Both routes return `{"greeting":"Hello, world!"}`.
- The facade and manual resolution produce identical results.

**Why This Code Produces That Result:**

- `Cache::put()` triggers `__callStatic()`, which resolves the `cache` binding and calls `put()` on the resolved instance.
- The manual version does the same thing explicitly.
- The facade provides a cleaner syntax for the same operation.

### Real-World Cases

**Case 1: Logging with the Log Facade**

A developer uses `Log::info('User logged in')` to write log messages without importing the underlying `Logger` class or resolving it from the container.

**Case 2: Database Queries with the DB Facade**

A controller uses `DB::table('users')->get()` for a quick database query without defining an Eloquent model.

**Case 3: Route Definition with the Route Facade**

A routes file uses `Route::get('/home', ...)` to define routes, leveraging the facade for a clean, expressive syntax.

### References

- Laravel Facades Documentation (12.x) - https://laravel.com/framework/docs/12.x/facades
- Laravel API: Facade Class - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html
- Laravel API: Cache Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Cache.html


## 2. Static-Looking Interfaces (Utilizing PHP's `__callStatic()` Magic Method to Proxy Calls Dynamically)

### Definitions

**Core Definition:** The static-looking interface of a facade is an illusion created by PHP's `__callStatic()` magic method, which intercepts calls to undefined static methods and forwards them to an underlying object instance.

**Technical Definition:** PHP's `__callStatic()` magic method is invoked when an inaccessible (protected or private) or non-existent static method is called on a class. Laravel's base `Facade` class implements `__callStatic()`, which resolves the facade's underlying instance from the container (using the key from `getFacadeAccessor()`) and then calls the requested method on that instance with the provided arguments. This mechanism allows facades to appear as static classes while actually delegating to instance methods on container-resolved objects.

**Beginner-Friendly Explanation:** The `__callStatic()` magic method is like a receptionist at a company. When you call the company's main number and ask for "John in accounting," the receptionist (the magic method) finds John (the underlying service) and transfers your call. You dialed a static number (the facade), but you're actually talking to a specific person (the instance). Laravel's facades use this trick to make instance methods look static.

### Purposes

- To enable static-looking syntax for instance-based services.
- To intercept undefined static method calls and route them to the correct instance.
- To resolve the underlying service from the container at call time.
- To forward method arguments and return values transparently.
- To maintain the illusion of a static API while using dependency injection under the hood.
- To allow facades to work with any service that implements the expected methods.
- To provide a consistent calling convention across all facades.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// The base Facade class's __callStatic method (simplified)

namespace Illuminate\Support\Facades;

abstract class Facade
{
    /**
     * Handle dynamic, static calls to the object.
     *
     * @param  string  $method
     * @param  array  $args
     * @return mixed
     */
    public static function __callStatic($method, $args)
    {
        // Resolve the facade root instance from the container
        $instance = static::getFacadeRoot();

        // If no instance, throw an exception
        if (! $instance) {
            throw new \RuntimeException('A facade root has not been set.');
        }

        // Call the method on the instance
        return $instance->$method(...$args);
    }
}
```

```php
<?php
// A concrete facade using the magic method

namespace Illuminate\Support\Facades;

class Cache extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

```php
<?php
// When you call Cache::get('key'), this is what happens:

// 1. PHP sees that Cache::get() doesn't exist as a static method
// 2. PHP invokes Cache::__callStatic('get', ['key'])
// 3. __callStatic() resolves the 'cache' binding from the container
// 4. It calls ->get('key') on the resolved instance
// 5. It returns the result
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `__callStatic($method, $args)` | PHP magic method for undefined static calls |
| `static::getFacadeRoot()` | Resolves the underlying instance |
| `$instance->$method(...$args)` | Forwards the call to the instance |
| `getFacadeAccessor()` | Returns the container binding key |

#### Syntax Rules

1. `__callStatic()` **must** be defined as a public static method.
2. It receives the method name as a string and the arguments as an array.
3. It **must** return the result of the forwarded call.
4. The facade root **must** be resolvable from the container; otherwise, a `RuntimeException` is thrown.
5. The underlying instance **must** have the called method; otherwise, a `BadMethodCallException` is thrown by PHP.
6. `__callStatic()` is only invoked for **inaccessible or non-existent** static methods.
7. Facades **may** also implement `__call()` for instance-level proxying (rarely used).

#### Constraints and Limitations

- **No Method Existence Check:** PHP does not verify that the method exists on the instance before calling; a `BadMethodCallException` occurs at runtime.
- **Performance Overhead:** Each static call resolves the facade root (though caching mitigates this).
- **IDE Support:** Static analysis tools may not recognize facade methods without additional helpers.
- **Debugging Difficulty:** Stack traces through `__callStatic()` can be harder to read.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Tracing a Facade Call Through `__callStatic()`

```php
<?php
// A simplified demonstration of how __callStatic works

class SimpleFacade
{
    protected static ?object $instance = null;

    public static function setInstance(object $instance): void
    {
        static::$instance = $instance;
    }

    public static function __callStatic(string $method, array $args): mixed
    {
        echo "Intercepted: {$method}(" . implode(', ', $args) . ")\n";
        return static::$instance->$method(...$args);
    }
}

class GreetingService
{
    public function greet(string $name): string
    {
        return "Hello, {$name}!";
    }
}

// Set the underlying instance
SimpleFacade::setInstance(new GreetingService());

// Call a non-existent static method
$result = SimpleFacade::greet('World');
echo $result . "\n";
```

**Expected Output:**

```
Intercepted: greet(World)
Hello, World!
```

**Why This Code Produces That Result:**

- `SimpleFacade::greet('World')` is not a real static method.
- PHP invokes `__callStatic('greet', ['World'])`.
- The magic method forwards the call to the `GreetingService` instance's `greet()` method.
- The result is returned transparently.

### Real-World Cases

**Case 1: Cache Facade with `__callStatic()`**

`Cache::get('key')` is intercepted by `__callStatic()`, which resolves the `cache` binding and calls `get('key')` on the cache repository.

**Case 2: Log Facade with `__callStatic()`**

`Log::info('message')` is intercepted and forwarded to the `Logger` instance resolved from the `log` binding.

**Case 3: Custom Facade with `__callStatic()`**

A developer creates a custom facade for a `PaymentService`, allowing calls like `Payment::charge(99.99)` while the actual payment logic lives in an injectable service.

### References

- PHP Manual: `__callStatic()` - https://www.php.net/manual/en/language.oop5.overloading.php#object.callstatic
- Laravel Facades: How Facades Work - https://laravel.com/framework/docs/12.x/facades#how-facades-work
- Laravel API: Facade::__callStatic - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html#method___callStatic


## 3. Underlying Container Resolution (Overriding `getFacadeAccessor()` to Return a String Container Key)

### Definitions

**Core Definition:** `getFacadeAccessor()` is the single method a facade class must implement, returning the string key that identifies the service binding in the Laravel service container that the facade should proxy to.

**Technical Definition:** The `getFacadeAccessor()` method is a protected static method declared on the base `Facade` class as abstract or requiring implementation. It must return a string representing the container binding key (e.g., `'cache'`, `'db'`, `'log'`, `'router'`). When a static method is called on the facade, the base `Facade` class uses this key to resolve the underlying service from the container via `app($key)`. If the key is not bound in the container, a `BindingResolutionException` is thrown.

**Beginner-Friendly Explanation:** `getFacadeAccessor()` is like the address on a package. When you use a facade, Laravel needs to know which service in the container to deliver your method call to. The `getFacadeAccessor()` method provides that address. For example, the `Cache` facade's accessor returns `'cache'`, which tells Laravel "look for the service registered under the key `cache`."

### Purposes

- To define which container binding the facade should resolve.
- To decouple the facade class from the underlying service implementation.
- To allow the same facade to work with different implementations bound to the same key.
- To provide a centralized point for changing the resolved service.
- To enable swapping implementations in tests by rebinding the key.
- To support aliased bindings (e.g., `'cache'` binding to `CacheManager`).
- To document the relationship between facade and service.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// The facade class with getFacadeAccessor()

namespace Illuminate\Support\Facades;

class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache'; // The container binding key
    }
}
```

```php
<?php
// The corresponding binding in a service provider

namespace App\Providers;

use Illuminate\Cache\CacheManager;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind the 'cache' key to the CacheManager
        $this->app->singleton('cache', function ($app) {
            return new CacheManager($app);
        });

        // Alias the binding so it can be resolved by class name
        $this->app->alias('cache', CacheManager::class);
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `getFacadeAccessor(): string` | Returns the container binding key |
| `return 'cache'` | The key registered in the container |
| `$this->app->singleton('cache', ...)` | Registers the service under the key |
| `$this->app->alias('cache', CacheManager::class)` | Allows resolution by class name |

#### Syntax Rules

1. `getFacadeAccessor()` **must** be declared as `protected static` and return a `string`.
2. The returned string **must** match a binding registered in a service provider.
3. If the returned string is a class name, the container will attempt to resolve it directly.
4. The method **should** return a simple, unique key (e.g., `'cache'`, `'db'`, `'log'`).
5. The method **must not** resolve the service itself; resolution happens in `__callStatic()`.
6. For real-time facades, `getFacadeAccessor()` returns the full class name of the proxied class.
7. The binding key **should** be registered in a service provider's `register()` method.

#### Constraints and Limitations

- **Key Must Exist:** If the key is not bound, a `BindingResolutionException` is thrown at call time.
- **String Only:** The method must return a string; returning an object or closure will cause errors.
- **No Dynamic Resolution:** The key is static; it cannot vary based on runtime conditions (for that, use a different facade or real-time facades).
- **Aliasing Required:** To resolve the facade by class name (e.g., in constructor injection), the key must be aliased.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Custom Facade with `getFacadeAccessor()`

**Step-by-Step Setup Guide:**

1. Create a service class.
2. Create a facade class extending `Facade`.
3. Implement `getFacadeAccessor()` returning the binding key.
4. Register the binding in a service provider.
5. Use the facade in a controller.

**Complete Executable Code:**

```php
<?php
// app/Services/PaymentService.php

namespace App\Services;

class PaymentService
{
    public function charge(float $amount): array
    {
        return ['status' => 'succeeded', 'amount' => $amount];
    }

    public function refund(string $chargeId): array
    {
        return ['status' => 'refunded', 'charge_id' => $chargeId];
    }
}
```

```php
<?php
// app/Facades/Payment.php

namespace App\Facades;

use Illuminate\Support\Facades\Facade;

class Payment extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'payment';
    }
}
```

```php
<?php
// app/Providers/PaymentServiceProvider.php

namespace App\Providers;

use App\Services\PaymentService;
use Illuminate\Support\ServiceProvider;

class PaymentServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('payment', function ($app) {
            return new PaymentService();
        });

        // Alias for constructor injection
        $this->app->alias('payment', PaymentService::class);
    }
}
```

```php
<?php
// app/Http/Controllers/CheckoutController.php

namespace App\Http\Controllers;

use App\Facades\Payment;

class CheckoutController extends Controller
{
    public function process()
    {
        // Static-looking facade call
        $result = Payment::charge(99.99);

        return response()->json($result);
    }
}
```

**Expected Output:**

- `GET /checkout` returns `{"status":"succeeded","amount":99.99}`.
- The `Payment` facade proxies to the `PaymentService` instance.

**Why This Code Produces That Result:**

- `getFacadeAccessor()` returns `'payment'`, which matches the registered binding.
- `__callStatic()` resolves `app('payment')` and calls `charge(99.99)` on it.
- The facade provides a clean, static-looking API for the payment service.

#### Example 2: Real-Time Facade Accessor

```php
<?php
// Real-time facades use the full class name as the accessor

namespace App\Http\Controllers;

use Facades\App\Services\PaymentService;

class CheckoutController extends Controller
{
    public function process()
    {
        // The accessor is App\Services\PaymentService::class
        $result = PaymentService::charge(99.99);

        return response()->json($result);
    }
}
```

**Expected Output:**

- The `PaymentService` real-time facade resolves the `App\Services\PaymentService` binding (or auto-resolves the class) and calls `charge(99.99)`.

**Why This Code Produces That Result:**

- `Facades\App\Services\PaymentService` triggers the real-time facade system.
- The accessor is the fully qualified class name.
- The container resolves the class automatically (no explicit binding needed).

### Real-World Cases

**Case 1: Custom Logging Facade**

A team creates a `CustomLog` facade with `getFacadeAccessor()` returning `'custom.log'`, binding their own logger implementation.

**Case 2: Multi-Tenant Payment Facade**

A multi-tenant SaaS uses a `Payment` facade whose accessor returns `'payment'`, with the binding resolving to different gateways per tenant.

**Case 3: Feature-Flagged Service Facade**

A facade's accessor returns a key that is bound to different implementations based on feature flags, allowing seamless swapping.

### References

- Laravel Facades: Creating Facades - https://laravel.com/framework/docs/12.x/facades#creating-facades
- Laravel API: Facade::getFacadeAccessor - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html#method_getFacadeAccessor
- Laravel News: Learn How to Create Custom Facades in Laravel - https://laravel-news.com/how-to-create-custom-facades


## 4. Common Laravel Facades (Deep Diving into `Log`, `Cache`, `DB`, and `Route` Usage)

### Definitions

**Core Definition:** Laravel ships with dozens of built-in facades that provide access to almost all of the framework's features, with the most commonly used being `Log`, `Cache`, `DB`, and `Route`.

**Technical Definition:** Each built-in facade follows the same pattern: a class extending `Facade` with `getFacadeAccessor()` returning a specific container binding key. `Log` proxies to `Illuminate\Log\Logger` via the `'log'` binding. `Cache` proxies to `Illuminate\Cache\Repository` via the `'cache'` binding. `DB` proxies to `Illuminate\Database\DatabaseManager` via the `'db'` binding. `Route` proxies to `Illuminate\Routing\Router` via the `'router'` binding.

**Beginner-Friendly Explanation:** Laravel gives you a toolbox full of facades. `Log` is your diary—write messages at different levels. `Cache` is your short-term memory—store and retrieve data quickly. `DB` is your direct line to the database—run queries without models. `Route` is your map—define the paths users can take through your application. Each one has a simple, static-looking API that hides the complexity underneath.

### Purposes

- To provide convenient access to Laravel's core services.
- To reduce boilerplate when using logging, caching, database, and routing.
- To offer a consistent API across different drivers and implementations.
- To enable quick prototyping without full dependency injection setup.
- To serve as documentation of the framework's capabilities.
- To allow testing through facade mocking.
- To provide terse, readable code in controllers, routes, and services.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Log Facade — Illuminate\Support\Facades\Log

use Illuminate\Support\Facades\Log;

Log::debug('Debug message');
Log::info('User logged in', ['user_id' => 1]);
Log::warning('Disk space low');
Log::error('Payment failed', ['order_id' => 123]);
Log::critical('System down');
```

```php
<?php
// Cache Facade — Illuminate\Support\Facades\Cache

use Illuminate\Support\Facades\Cache;

Cache::put('key', 'value', 3600);           // Store for 1 hour
$value = Cache::get('key');                   // Retrieve
$value = Cache::remember('key', 3600, fn () => 'computed'); // Remember
Cache::forget('key');                         // Remove
Cache::flush();                               // Clear all
```

```php
<?php
// DB Facade — Illuminate\Support\Facades\DB

use Illuminate\Support\Facades\DB;

$users = DB::table('users')->where('active', true)->get();
$user = DB::table('users')->where('email', 'john@example.com')->first();
DB::table('users')->insert(['name' => 'John', 'email' => 'john@example.com']);
DB::table('users')->where('id', 1)->update(['name' => 'John Doe']);
DB::table('users')->where('id', 1)->delete();

// Raw queries
$results = DB::select('SELECT * FROM users WHERE active = ?', [true]);
```

```php
<?php
// Route Facade — Illuminate\Support\Facades\Route

use Illuminate\Support\Facades\Route;

Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
Route::put('/users/{id}', [UserController::class, 'update']);
Route::delete('/users/{id}', [UserController::class, 'destroy']);

// Named routes
Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');

// Route groups with middleware
Route::middleware(['auth'])->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
});
```

**Component Breakdown:**

| Facade | Underlying Class | Binding Key | Primary Use |
|--------|-----------------|-------------|-------------|
| `Log` | `Illuminate\Log\Logger` | `'log'` | Application logging |
| `Cache` | `Illuminate\Cache\Repository` | `'cache'` | Caching data |
| `DB` | `Illuminate\Database\DatabaseManager` | `'db'` | Database queries |
| `Route` | `Illuminate\Routing\Router` | `'router'` | Route definition |

#### Syntax Rules

1. Each facade **must** be imported from `Illuminate\Support\Facades\{Name}`.
2. Facade methods **correspond** to methods on the underlying service.
3. `Log` methods include `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.
4. `Cache` methods include `get`, `put`, `remember`, `forget`, `flush`, `has`, `increment`, `decrement`.
5. `DB` methods include `table`, `select`, `insert`, `update`, `delete`, `statement`, `transaction`.
6. `Route` methods include `get`, `post`, `put`, `patch`, `delete`, `any`, `match`, `group`, `middleware`.
7. Facades **may** be used in routes, controllers, services, and anywhere outside of `config` files.

#### Constraints and Limitations

- **No Facades in Config Files:** Using facades in `config/*.php` files causes "A facade root has not been set" errors during `config:cache`.
- **Scope Creep:** Using many facades in one class can obscure dependencies and violate single responsibility.
- **Testing:** Facades require `shouldReceive()` mocks, which are less explicit than constructor injection mocks.
- **Static Analysis:** Some static analysis tools struggle with facade method resolution.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Log Facade in a Service

```php
<?php
// app/Services/OrderService.php

namespace App\Services;

use App\Models\Order;
use Illuminate\Support\Facades\Log;

class OrderService
{
    public function ship(Order $order): void
    {
        Log::info('Shipping order', ['order_id' => $order->id]);

        try {
            // Shipping logic...
            $order->update(['status' => 'shipped']);

            Log::info('Order shipped successfully', ['order_id' => $order->id]);
        } catch (\Exception $e) {
            Log::error('Order shipping failed', [
                'order_id' => $order->id,
                'error' => $e->getMessage(),
            ]);

            throw $e;
        }
    }
}
```

**Expected Output:**

- Log entries are written to the configured log channel (e.g., `storage/logs/laravel.log`).
- Each entry includes the message and context data.

**Why This Code Produces That Result:**

- `Log::info()` proxies to the `Logger` instance resolved from the `'log'` binding.
- The `Logger` writes to the configured channel.
- Context data is included in the log entry.

#### Example 2: Cache Facade with Remember Pattern

```php
<?php
// app/Services/ProductService.php

namespace App\Services;

use Illuminate\Support\Facades\Cache;

class ProductService
{
    public function getFeaturedProducts(): array
    {
        // Cache the result for 1 hour
        return Cache::remember('products.featured', 3600, function () {
            return \App\Models\Product::where('featured', true)
                ->orderBy('created_at', 'desc')
                ->take(10)
                ->get()
                ->toArray();
        });
    }

    public function clearFeaturedCache(): void
    {
        Cache::forget('products.featured');
    }
}
```

**Expected Output:**

- The first call queries the database and caches the result.
- Subsequent calls within the hour return the cached result without querying the database.
- `clearFeaturedCache()` removes the cached data, forcing a fresh query.

**Why This Code Produces That Result:**

- `Cache::remember()` checks the cache, and if the key is missing, executes the closure, stores the result, and returns it.
- The cache key is `'products.featured'`.
- The TTL is 3600 seconds (1 hour).

### Real-World Cases

**Case 1: Logging in a Payment Controller**

A payment controller uses `Log::error()` to record failed payment attempts with order IDs and error messages.

**Case 2: Caching Expensive Queries**

A dashboard service uses `Cache::remember()` to cache aggregated statistics for 5 minutes, reducing database load.

**Case 3: Database Transactions with DB Facade**

A service uses `DB::transaction()` to wrap multiple database operations in a single atomic transaction.

**Case 4: Route Groups with Middleware**

A routes file uses `Route::middleware(['auth', 'verified'])->group()` to apply middleware to a group of admin routes.

### References

- Laravel Logging Documentation - https://laravel.com/docs/12.x/logging
- Laravel Cache Documentation - https://laravel.com/docs/12.x/cache
- Laravel Database: Query Builder - https://laravel.com/docs/12.x/queries
- Laravel Routing Documentation - https://laravel.com/docs/12.x/routing
- Laravel API: Log Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Log.html
- Laravel API: DB Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/DB.html
- Laravel API: Route Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Route.html


## 5. Enhanced: Real-Time Facades (Prefixing Class Namespaces Dynamically with `Facades\` to Instantly Generate a Facade on the Fly)

### Definitions

**Core Definition:** Real-time facades allow you to treat any class in your application as if it were a facade, without creating a dedicated facade class, by simply prefixing the imported class's namespace with `Facades\`.

**Technical Definition:** When you import a class using `use Facades\App\Services\Publisher;`, Laravel's `AliasLoader` dynamically generates a facade class for `App\Services\Publisher` at runtime. The generated facade extends `Facade` and implements `getFacadeAccessor()` returning the full class name (`App\Services\Publisher::class`). The container then resolves the class (or its interface binding) and forwards static calls to it. This feature was introduced in Laravel 5.5 and eliminates the boilerplate of creating dedicated facade classes for every service.

**Beginner-Friendly Explanation:** Normally, to use a facade, you have to create a facade class, define `getFacadeAccessor()`, and register a binding. Real-time facades let you skip all that. Just write `use Facades\App\Services\Publisher;` and suddenly `Publisher::publish()` works as a facade. Laravel creates the facade for you on the fly. It's like having a magic wand that turns any class into a facade instantly.

### Purposes

- To eliminate the boilerplate of creating dedicated facade classes.
- To quickly convert any service into a facade for convenient access.
- To enable static-like access to classes that weren't designed as facades.
- To reduce the number of files in a project by avoiding one facade per service.
- To simplify prototyping and rapid development.
- To allow interfaces to be used as facades (resolving the bound implementation).
- To provide the same testability as traditional facades.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Traditional facade: requires a dedicated facade class

namespace App\Facades;

use Illuminate\Support\Facades\Facade;

class Publisher extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'publisher';
    }
}

// Plus a binding in a service provider:
// $this->app->bind('publisher', fn () => new \App\Services\Publisher());
```

```php
<?php
// Real-time facade: no dedicated class needed

namespace App\Http\Controllers;

use Facades\App\Services\Publisher;

class ArticleController extends Controller
{
    public function publish(int $articleId)
    {
        // Publisher is resolved from the container (or auto-resolved)
        $result = Publisher::publish($articleId);

        return response()->json($result);
    }
}
```

```php
<?php
// Real-time facade with an interface

namespace App\Http\Controllers;

use Facades\App\Contracts\Publisher;

class ArticleController extends Controller
{
    public function publish(int $articleId)
    {
        // The bound implementation of the interface is resolved
        $result = Publisher::publish($articleId);

        return response()->json($result);
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `use Facades\App\Services\Publisher;` | Imports the real-time facade |
| `Publisher::publish()` | Static-looking call on the real-time facade |
| `AliasLoader` | Dynamically generates the facade class |
| `getFacadeAccessor()` | Returns the full class name (`App\Services\Publisher::class`) |
| `app(Publisher::class)` | Resolves the class from the container |

#### Syntax Rules

1. The class **must** be imported with the `Facades\` prefix before its full namespace.
2. The underlying class **must** be resolvable from the service container (either auto-resolvable or bound).
3. Real-time facades **work with interfaces**; the container resolves the bound implementation.
4. The generated facade **must** have access to the container; it does not work in config files.
5. Real-time facades **can be tested** using the same `shouldReceive()` methods as traditional facades.
6. The `Facades\` prefix **must** be at the beginning of the `use` statement.
7. Real-time facades **do not** require a service provider registration for the facade itself (only for the underlying class if it's an interface).

#### Constraints and Limitations

- **No Boot-Time Use:** Real-time facades cannot be used in `config` files or during the application's bootstrap phase.
- **Slight Overhead:** The dynamic facade generation adds a small runtime overhead compared to pre-defined facades.
- **IDE Support:** Some IDEs may not fully recognize real-time facade methods.
- **Debugging:** Generated facade classes can make stack traces more complex.
- **Not for Every Class:** Classes with constructors requiring primitive parameters may not auto-resolve.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Real-Time Facade for a Service Class

**Step-by-Step Setup Guide:**

1. Create a service class with a public method.
2. Import it using `Facades\` in a controller.
3. Call the method statically.
4. Verify the service is resolved and the method executes.

**Complete Executable Code:**

```php
<?php
// app/Services/Publisher.php

namespace App\Services;

class Publisher
{
    public function publish(int $articleId): array
    {
        // Simulate publishing logic
        return [
            'article_id' => $articleId,
            'status' => 'published',
            'published_at' => now()->toIso8601String(),
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/ArticleController.php

namespace App\Http\Controllers;

use Facades\App\Services\Publisher;

class ArticleController extends Controller
{
    public function publish(int $articleId)
    {
        // Real-time facade — no dedicated facade class needed
        $result = Publisher::publish($articleId);

        return response()->json($result);
    }
}
```

**Expected Output:**

- `POST /articles/{id}/publish` returns `{"article_id":1,"status":"published","published_at":"2026-10-06T12:00:00Z"}`.
- The `Publisher` service is auto-resolved from the container.

**Why This Code Produces That Result:**

- `use Facades\App\Services\Publisher;` tells Laravel to generate a real-time facade.
- `Publisher::publish($articleId)` is intercepted by the generated facade's `__callStatic()`.
- The facade resolves `App\Services\Publisher` from the container (auto-resolution, since it has no dependencies) and calls `publish()`.

#### Example 2: Real-Time Facade with an Interface

```php
<?php
// app/Contracts/Publisher.php

namespace App\Contracts;

interface Publisher
{
    public function publish(int $articleId): array;
}
```

```php
<?php
// app/Services/EloquentPublisher.php

namespace App\Services;

use App\Contracts\Publisher;

class EloquentPublisher implements Publisher
{
    public function publish(int $articleId): array
    {
        return ['article_id' => $articleId, 'status' => 'published'];
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\Publisher;
use App\Services\EloquentPublisher;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(Publisher::class, EloquentPublisher::class);
    }
}
```

```php
<?php
// app/Http/Controllers/ArticleController.php

namespace App\Http\Controllers;

use Facades\App\Contracts\Publisher;

class ArticleController extends Controller
{
    public function publish(int $articleId)
    {
        // The bound implementation (EloquentPublisher) is resolved
        $result = Publisher::publish($articleId);

        return response()->json($result);
    }
}
```

**Expected Output:**

- The real-time facade resolves `App\Contracts\Publisher` to `EloquentPublisher` and calls `publish()`.

**Why This Code Produces That Result:**

- The interface is bound to `EloquentPublisher` in the service provider.
- The real-time facade's accessor returns `App\Contracts\Publisher::class`.
- The container resolves the bound implementation.
- This allows swapping implementations without changing the facade usage.

### Real-World Cases

**Case 1: Rapid Prototyping**

A developer uses real-time facades during prototyping to avoid creating facade classes for every new service.

**Case 2: Testing Interface Implementations**

A test uses `Facades\App\Contracts\PaymentGateway::shouldReceive('charge')` to mock the payment gateway without creating a dedicated facade.

**Case 3: Third-Party Service Access**

A package consumer uses `Facades\Vendor\Package\Service::method()` to access a package service statically without modifying the package.

### References

- Laravel Facades: Real-Time Facades - https://laravel.com/framework/docs/12.x/facades#real-time-facades
- Laravel API: AliasLoader - https://api.laravel.com/docs/11.x/Illuminate/Foundation/AliasLoader.html
- Medium: Laravel Facades: Traditional and Real-Time - https://medium.com/@sitemap/laravel-facades-traditional-and-real-time


## 6. Enhanced: Facade Root Caching (Understanding How Laravel Caches Resolved Instances Inside the Facade Base to Minimize Container Lookup Overhead)

### Definitions

**Core Definition:** Facade root caching is the mechanism by which Laravel stores the resolved instance of a facade's underlying service in a static array on the base `Facade` class, so subsequent calls to the same facade do not need to re-resolve the service from the container.

**Technical Definition:** The base `Facade` class maintains a static `$resolvedInstance` array (keyed by the facade accessor) and a static `$cached` boolean property. When `getFacadeRoot()` is called for the first time, the container resolves the service and stores it in `$resolvedInstance`. Subsequent calls to the same facade return the cached instance directly, bypassing the container lookup. The `$cached` property (default `true`) determines whether instances should be cached. The cache can be cleared per-facade via `Facade::clearResolvedInstance()` or globally via `Facade::clearResolvedInstances()`.

**Beginner-Friendly Explanation:** Imagine you call a restaurant to order food. The first time, the receptionist looks up the kitchen's phone number, calls them, and takes your order. The second time you call, the receptionist already has the kitchen's number on speed dial (the cached instance), so they skip the lookup and call immediately. Facade root caching is Laravel's "speed dial"—it remembers the resolved service so it doesn't have to ask the container every time.

### Purposes

- To minimize container lookup overhead on repeated facade calls.
- To improve performance in loops or high-frequency facade usage.
- To ensure the same instance is used across multiple calls (singleton-like behavior).
- To allow clearing the cache when the underlying service needs to be re-resolved.
- To support testing by swapping the cached instance with a mock.
- To provide a consistent instance across the request lifecycle.
- To reduce the performance impact of `__callStatic()` resolution.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// The base Facade class's caching mechanism (simplified)

namespace Illuminate\Support\Facades;

abstract class Facade
{
    /**
     * The resolved object instances.
     *
     * @var array
     */
    protected static $resolvedInstance = [];

    /**
     * Indicates if the resolved instance should be cached.
     *
     * @var bool
     */
    protected static $cached = true;

    /**
     * Get the root object behind the facade.
     *
     * @return mixed
     */
    public static function getFacadeRoot()
    {
        return static::resolveFacadeInstance(static::getFacadeAccessor());
    }

    /**
     * Resolve the facade root instance from the container.
     *
     * @param  string  $name
     * @return mixed
     */
    protected static function resolveFacadeInstance($name)
    {
        // Return cached instance if available
        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        // Resolve from container and cache
        if (static::$app) {
            return static::$resolvedInstance[$name] = static::$app[$name];
        }
    }

    /**
     * Clear a resolved facade instance.
     *
     * @param  string  $name
     * @return void
     */
    public static function clearResolvedInstance($name)
    {
        unset(static::$resolvedInstance[$name]);
    }

    /**
     * Clear all of the resolved instances.
     *
     * @return void
     */
    public static function clearResolvedInstances()
    {
        static::$resolvedInstance = [];
    }
}
```

```php
<?php
// Swapping the cached instance for testing

use Illuminate\Support\Facades\Cache;

// Replace the cached instance with a mock
Cache::swap($mock);

// Or using shouldReceive (which internally swaps)
Cache::shouldReceive('get')->andReturn('mocked');
```

```php
<?php
// Clearing a cached instance

use Illuminate\Support\Facades\Facade;

// Clear a specific facade
Facade::clearResolvedInstance('cache');

// Clear all facades
Facade::clearResolvedInstances();
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `$resolvedInstance` | Static array storing cached instances keyed by accessor |
| `$cached` | Boolean flag indicating if caching is enabled |
| `resolveFacadeInstance()` | Checks cache, resolves from container if needed |
| `clearResolvedInstance()` | Clears a specific facade's cached instance |
| `clearResolvedInstances()` | Clears all cached facade instances |
| `swap()` | Replaces the cached instance with a mock |

#### Syntax Rules

1. Facade instances are cached in the static `$resolvedInstance` array, keyed by the accessor string.
2. Caching occurs on the **first** call to `getFacadeRoot()` for each facade.
3. Subsequent calls return the cached instance without container lookup.
4. The `$cached` property **must** be `true` (default) for caching to occur.
5. `clearResolvedInstance()` **must** be called to remove a specific cached instance.
6. `clearResolvedInstances()` **must** be called to clear all cached instances.
7. `swap()` **must** be used to replace a cached instance with a mock or alternative implementation.
8. In Octane, the cache is cleared between requests to prevent state leakage.

#### Constraints and Limitations

- **Stale Instances:** If the underlying binding changes after the facade is cached, the facade continues using the old instance until cleared.
- **State Leakage in Octane:** Singletons persist across requests in Octane; facades must be cleared between requests.
- **Testing:** Failing to clear cached instances between tests can cause test pollution.
- **Memory:** Cached instances persist for the request lifecycle; long-running processes may accumulate memory.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Facade Root Caching in Action

**Step-by-Step Setup Guide:**

1. Use a facade multiple times in a single request.
2. Observe that the underlying instance is only resolved once.
3. Clear the cache and observe re-resolution.

**Complete Executable Code:**

```php
<?php
// app/Http/Controllers/CacheController.php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Facade;

class CacheController extends Controller
{
    public function index()
    {
        // First call: resolves and caches the cache instance
        Cache::put('key1', 'value1', 3600);

        // Second call: uses the cached instance (no container lookup)
        Cache::put('key2', 'value2', 3600);

        // Third call: still using the cached instance
        $value = Cache::get('key1');

        // Clear the cache for this facade
        Facade::clearResolvedInstance('cache');

        // Next call: re-resolves the cache instance from the container
        Cache::put('key3', 'value3', 3600);

        return response()->json([
            'value' => $value,
            'all_keys' => [Cache::get('key1'), Cache::get('key2'), Cache::get('key3')],
        ]);
    }
}
```

**Expected Output:**

- The first three `Cache` calls use the same cached instance.
- After `clearResolvedInstance('cache')`, the next call re-resolves the instance.
- All keys are stored and retrieved successfully.

**Why This Code Produces That Result:**

- `Cache::put()` triggers `__callStatic()`, which calls `getFacadeRoot()`.
- `getFacadeRoot()` checks `$resolvedInstance['cache']`; if absent, resolves from the container and caches.
- After clearing, the next call re-resolves from the container.

#### Example 2: Swapping a Facade for Testing

```php
<?php
// tests/Feature/PaymentTest.php

namespace Tests\Feature;

use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class PaymentTest extends TestCase
{
    public function test_payment_uses_cache()
    {
        // Swap the cached Cache instance with a mock
        Cache::shouldReceive('remember')
            ->once()
            ->with('payment.config', 3600, \Mockery::any())
            ->andReturn(['driver' => 'stripe']);

        // Resolve the service — it uses the mocked Cache
        $service = app(\App\Services\PaymentService::class);
        $result = $service->process(99.99);

        $this->assertEquals('stripe', $result['driver']);
    }
}
```

**Expected Output:**

- The test passes, verifying the service uses `Cache::remember()` with the correct arguments.
- No real cache interaction occurs.

**Why This Code Produces That Result:**

- `Cache::shouldReceive()` internally calls `swap()` to replace the cached instance with a mock.
- The service resolves the `Cache` facade, which returns the mock.
- The mock records and verifies the call.

### Real-World Cases

**Case 1: High-Frequency Logging**

A service logs thousands of messages in a loop. Facade root caching ensures the `Log` instance is resolved only once, reducing container overhead.

**Case 2: Octane Request Isolation**

Laravel Octane clears resolved facade instances between requests to prevent state leakage from one request to the next.

**Case 3: Testing with Mocked Facades**

Tests use `Cache::shouldReceive()` to swap the cached instance with a mock, isolating the unit under test.

### References

- Laravel API: Facade::$resolvedInstance - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html#property_resolvedInstance
- Laravel API: Facade::clearResolvedInstance - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html#method_clearResolvedInstance
- Laravel API: Facade::clearResolvedInstances - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html#method_clearResolvedInstances
- Laravel Facades: Facade Class Reference - https://laravel.com/framework/docs/12.x/facades#facade-class-reference


## Summary Table of Facade Concepts

| Concept | Mechanism | Key Method/Property | Purpose |
|---------|-----------|-------------------|---------|
| Facade Concept | Static proxy to container service | `extends Facade` | Clean syntax for complex services |
| Static-Looking Interfaces | PHP `__callStatic()` | `__callStatic()` | Intercept static calls, forward to instance |
| Container Resolution | Container binding key | `getFacadeAccessor()` | Define which service to resolve |
| Common Facades | Built-in facades | `Log`, `Cache`, `DB`, `Route` | Logging, caching, database, routing |
| Real-Time Facades | Dynamic facade generation | `use Facades\...` | Instant facade for any class |
| Facade Root Caching | Static instance cache | `$resolvedInstance` | Minimize container lookup overhead |


## References

- Laravel Facades Documentation (12.x) - https://laravel.com/framework/docs/12.x/facades
- Laravel Facades Documentation (10.x) - https://laravel.com/framework/docs/10.x/facades
- Laravel Facades Documentation (13.x) - https://laravel.com/framework/docs/13.x/facades
- Laravel Facades: How Facades Work - https://laravel.com/framework/docs/12.x/facades#how-facades-work
- Laravel Facades: Real-Time Facades - https://laravel.com/framework/docs/12.x/facades#real-time-facades
- Laravel Facades: Facade Class Reference - https://laravel.com/framework/docs/12.x/facades#facade-class-reference
- Laravel API: Facade Class - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Facade.html
- Laravel API: Cache Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Cache.html
- Laravel API: DB Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/DB.html
- Laravel API: Log Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Log.html
- Laravel API: Route Facade - https://api.laravel.com/docs/11.x/Illuminate/Support/Facades/Route.html
- Laravel API: AliasLoader - https://api.laravel.com/docs/11.x/Illuminate/Foundation/AliasLoader.html
- Laravel News: Learn How to Create Custom Facades in Laravel - https://laravel-news.com/how-to-create-custom-facades
- PHP Manual: `__callStatic()` - https://www.php.net/manual/en/language.oop5.overloading.php#object.callstatic
- DeepWiki: Service Providers and Facades - https://deepwiki.com/unicodeveloper/laravel-exam/2.3-service-providers-and-facades
- Medium: Laravel Facades: Traditional and Real-Time - https://medium.com/@sitemap/laravel-facades-traditional-and-real-time
- Laravel Logging Documentation - https://laravel.com/docs/12.x/logging
- Laravel Cache Documentation - https://laravel.com/docs/12.x/cache
- Laravel Database: Query Builder - https://laravel.com/docs/12.x/queries
- Laravel Routing Documentation - https://laravel.com/docs/12.x/routing