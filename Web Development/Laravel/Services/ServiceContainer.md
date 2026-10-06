# Laravel Service Container: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Laravel Service Container is a powerful dependency injection (DI) container that manages class dependencies and performs dependency injection throughout the application. It is the fundamental mechanism by which Laravel resolves and injects dependencies into controllers, event listeners, middleware, queued jobs, and more.

**Technical Definition:** The service container (`Illuminate\Container\Container`) is an implementation of the Inversion of Control (IoC) pattern that uses PHP's Reflection API to inspect class constructors, automatically resolve type-hinted dependencies recursively, and instantiate objects with all required dependencies injected. It maintains a registry of bindings (abstract-to-concrete mappings), shared instances (singletons), aliases, contextual bindings, tags, and extenders, enabling flexible, testable, and decoupled application architecture.

**Beginner-Friendly Explanation:** Think of the service container as a smart factory. When you need an object—say, a `PodcastController` that requires an `AppleMusic` service—you don't build it yourself. Instead, you ask the factory (the container), and it looks at the blueprint (the constructor's type-hints), builds all the parts it needs, and hands you the finished product. If you need the same part later, it can remember it (singleton), or build a fresh one each time. You can also tell the factory: "When building this specific product, use this special part" (contextual binding).

### Key Characteristics

1. **Automatic Resolution:** Resolves class dependencies automatically via PHP Reflection without configuration.
2. **Binding Registry:** Maps interfaces to concrete implementations, closures, or instances.
3. **Singleton Management:** Ensures a class is resolved only once per application lifecycle.
4. **Contextual Binding:** Injects different implementations into different classes that share the same dependency.
5. **Tagging:** Groups related bindings for collective resolution.
6. **Extension:** Decorates or modifies resolved services after instantiation.
7. **Scoped Bindings:** Creates per-request/per-job singletons for safe Octane and queue worker compatibility.
8. **Zero-Configuration DI:** Most classes require no explicit container interaction.

### Prerequisites

- Laravel 9.x or higher (12.x recommended)
- PHP 8.0 or higher
- Composer package manager
- Familiarity with PHP type-hinting and interfaces
- Understanding of Laravel Service Providers
- Basic knowledge of the Inversion of Control pattern

### Related Programming Areas

- **Dependency Injection:** The broader design pattern the container implements.
- **Inversion of Control:** The architectural principle behind the container.
- **Service Providers:** The registration point for container bindings.
- **PHP Reflection API:** The mechanism for automatic resolution.
- **Facades:** The static interface to container-managed services.
- **Testing and Mocking:** The container enables easy test doubles.

### Core Concepts / Features

1. Dependency Resolution (`app()`, `resolve()`, `App::make()`)
2. Bindings (`$this->app->bind()`)
3. Singletons (`$this->app->singleton()`)
4. Contextual Bindings (`$this->app->when()->needs()->give()`)
5. Automatic Resolution (PHP Reflection API)
6. Enhanced: Container Tagging (`$this->app->tag()` and `$this->app->tagged()`)
7. Enhanced: Extending Bindings (`$this->app->extend()`)
8. Enhanced: Scoped Bindings (`$this->app->scoped()`)


## 1. Dependency Resolution (Resolving Instances Explicitly Using `app()`, `resolve()`, or `App::make()`)

### Definitions

**Core Definition:** Dependency resolution is the process of asking the service container to build and return an instance of a class or interface, automatically injecting all required dependencies.

**Technical Definition:** The container's `make` method (exposed via `app()`, `resolve()`, and `App::make()`) accepts an abstract type (class name, interface name, or alias), inspects the concrete class's constructor using Reflection, recursively resolves each type-hinted dependency, and returns a fully constructed instance. If the type is bound, the bound resolver is executed; otherwise, the container attempts automatic resolution.

**Beginner-Friendly Explanation:** Resolution is like ordering a custom sandwich from a deli. You say "I want a `PodcastController`." The deli (container) checks the recipe (constructor), sees it needs `AppleMusic`, prepares that first, then assembles the whole sandwich and hands it to you. You can order by name (`app(Transistor::class)`), by the global helper (`resolve()`), or via the facade (`App::make()`).

### Purposes

- To explicitly obtain an instance of a class from the container.
- To resolve dependencies outside of constructor injection (e.g., in routes or commands).
- To pass runtime parameters to a class that also has container-resolved dependencies.
- To resolve interface bindings to their concrete implementations.
- To enable dependency injection in contexts that aren't automatically resolved.
- To facilitate testing by resolving mock implementations.
- To provide a consistent API for object construction across the application.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php

use App\Services\Transistor;
use Illuminate\Support\Facades\App;

// Using the app() helper
$transistor = app(Transistor::class);

// Using the resolve() helper
$transistor = resolve(Transistor::class);

// Using the App facade
$transistor = App::make(Transistor::class);

// Using the container instance directly
$transistor = $this->app->make(Transistor::class);

// Resolving with runtime parameters
$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);

// Using the app() helper with parameters
$transistor = app()->make(Transistor::class, ['id' => 1]);

// Checking if a binding exists before resolving
if (app()->bound(Transistor::class)) {
    $transistor = app(Transistor::class);
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `app()` | Global helper to resolve from the container |
| `resolve()` | Global helper that delegates to `app()` |
| `App::make()` | Facade method for resolution |
| `$this->app->make()` | Container instance method (in service providers) |
| `makeWith()` | Resolves with manually supplied parameters |
| `bound()` | Checks if a binding exists |
| `$parameters` | Associative array of primitive constructor values |

#### Syntax Rules

1. `app()` and `resolve()` are functionally equivalent; `resolve()` calls `app()` internally.
2. `App::make()` is a facade proxy to the container's `make` method.
3. `make()` accepts an optional second argument: an associative array of parameters for unresolvable dependencies.
4. `makeWith()` is used when some dependencies cannot be resolved by the container (e.g., primitive values like `$id`).
5. The container resolves dependencies recursively until the full dependency tree is built.
6. If the class cannot be resolved, a `BindingResolutionException` is thrown.
7. `bound()` should be used to check for a binding before resolving, especially for optional services.

#### Constraints and Limitations

- **Primitive Dependencies:** The container cannot resolve primitive values (strings, ints) without explicit parameters via `makeWith()` or `['param' => $value]`.
- **Circular Dependencies:** Circular dependencies cause infinite recursion and should be avoided or broken with setter injection.
- **Interface Without Binding:** Resolving an interface without a registered binding throws an exception; you must bind it first.
- **Performance:** Resolution uses Reflection, which has overhead; excessive runtime resolution (e.g., in loops) should be avoided.
- **Non-Instantiable Classes:** Abstract classes and interfaces cannot be resolved unless bound to a concrete implementation.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Resolving a Concrete Class

**Step-by-Step Setup Guide:**

1. Create a simple service class.
2. Resolve it using `app()` in a route closure.
3. Verify the instance is created and injected.

**Complete Executable Code:**

```php
<?php
// app/Services/Transistor.php

namespace App\Services;

class Transistor
{
    public function __construct(
        public ?int $id = null
    ) {}

    public function describe(): string
    {
        return "Transistor #{$this->id}";
    }
}
```

```php
<?php
// routes/web.php

use App\Services\Transistor;

Route::get('/transistor', function () {
    // Resolve without parameters — id defaults to null
    $transistor = app(Transistor::class);
    return $transistor->describe(); // "Transistor #"
});

Route::get('/transistor/{id}', function (int $id) {
    // Resolve with a runtime parameter
    $transistor = app()->make(Transistor::class, ['id' => $id]);
    return $transistor->describe(); // "Transistor #42"
});
```

**Expected Output:**

- `GET /transistor` returns `"Transistor #"`.
- `GET /transistor/42` returns `"Transistor #42"`.

**Why This Code Produces That Result:**

- `app(Transistor::class)` inspects the constructor, sees `?int $id` with a default, and instantiates the class with `id = null`.
- `makeWith()` (or `make()` with parameters) supplies the `id` value explicitly.
- The container automatically injects the parameter when provided.

#### Example 2: Resolving an Interface Binding

```php
<?php
// app/Contracts/PodcastSource.php

namespace App\Contracts;

interface PodcastSource
{
    public function findPodcast(string $id): array;
}
```

```php
<?php
// app/Services/AppleMusic.php

namespace App\Services;

use App\Contracts\PodcastSource;

class AppleMusic implements PodcastSource
{
    public function findPodcast(string $id): array
    {
        return ['id' => $id, 'title' => 'Example Podcast'];
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\PodcastSource;
use App\Services\AppleMusic;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind the interface to the concrete implementation
        $this->app->bind(PodcastSource::class, AppleMusic::class);
    }
}
```

```php
<?php
// routes/web.php

use App\Contracts\PodcastSource;

Route::get('/podcast/{id}', function (string $id) {
    // Resolve the interface — the container returns AppleMusic
    $source = app(PodcastSource::class);
    return $source->findPodcast($id);
});
```

**Expected Output:**

- `GET /podcast/123` returns `{"id":"123","title":"Example Podcast"}`.
- The container resolves `PodcastSource` to `AppleMusic` and injects it.

**Why This Code Produces That Result:**

- The `bind()` call in `AppServiceProvider` maps the interface to the concrete class.
- When `app(PodcastSource::class)` is called, the container looks up the binding and instantiates `AppleMusic`.
- Without the binding, resolving the interface would throw a `BindingResolutionException`.

### Real-World Cases

**Case 1: Resolving Services in Artisan Commands**

An Artisan command resolves a `ReportGenerator` service using `app(ReportGenerator::class)` to generate reports outside the HTTP request lifecycle.

**Case 2: Resolving in Route Closures**

A route closure type-hints a `PaymentGateway` interface; the container resolves the bound implementation automatically.

**Case 3: Resolving in Tests**

A test resolves a `UserRepository` from the container and replaces it with a mock to assert behavior.

### References

- Laravel Service Container: Resolving - https://laravel.com/docs/12.x/container#resolving
- Laravel Service Container: The make Method - https://laravel.com/docs/12.x/container#the-make-method
- Laravel API: Container - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html


## 2. Bindings (Registering Basic Resolver Closures via `$this->app->bind()`)

### Definitions

**Core Definition:** A binding is a registration in the service container that maps an abstract type (interface or class name) to a concrete implementation, resolver closure, or instance, telling the container how to build that dependency.

**Technical Definition:** The `bind()` method registers an entry in the container's `$bindings` array. When the abstract type is resolved, the container executes the concrete resolver (a Closure or class name) to produce an instance. If `$shared` is `true`, the binding behaves as a singleton. Bindings are typically registered in the `register()` method of a Service Provider.

**Beginner-Friendly Explanation:** A binding is like a recipe card in a recipe box. You write: "When someone asks for a `PaymentGateway`, make them a `StripeGateway`." The container looks at the card whenever someone needs a `PaymentGateway` and follows the instructions.

### Purposes

- To map interfaces to concrete implementations.
- To register custom factory logic for complex object creation.
- To decouple application code from specific implementations.
- To enable swapping implementations for testing or different environments.
- To configure objects with runtime-dependent parameters.
- To provide a central registration point for all application dependencies.
- To support lazy loading of expensive services.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// In a Service Provider's register() method

namespace App\Providers;

use App\Contracts\PaymentGateway;
use App\Services\StripeGateway;
use App\Services\PayPalGateway;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind interface to concrete class
        $this->app->bind(PaymentGateway::class, StripeGateway::class);

        // Bind with a closure
        $this->app->bind(PaymentGateway::class, function ($app) {
            return new StripeGateway(
                config('services.stripe.key'),
                config('services.stripe.secret')
            );
        });

        // Bind with a closure that receives the container
        $this->app->bind('ReportGenerator', function ($app) {
            return new \App\Services\ReportGenerator(
                $app->make(\App\Contracts\ReportRepository::class)
            );
        });

        // Bind only if not already bound
        $this->app->bindIf(PaymentGateway::class, StripeGateway::class);

        // Bind an existing instance
        $this->app->instance(PaymentGateway::class, new StripeGateway('key', 'secret'));
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `bind($abstract, $concrete)` | Registers a binding |
| `$abstract` | The interface or class name to bind |
| `$concrete` | A Closure, class name, or null (self-binding) |
| `$shared` (optional) | If `true`, behaves as a singleton |
| `bindIf()` | Registers only if not already bound |
| `instance()` | Registers an existing object instance |
| `$app` (closure param) | The container instance, injected automatically |

#### Syntax Rules

1. `bind()` **must** be called within a Service Provider's `register()` method (or bootstrapped code).
2. The `$abstract` parameter is the key used for resolution (usually an interface or class FQCN).
3. The `$concrete` parameter can be a Closure, a class FQCN string, or `null` (self-binding).
4. Closure-based bindings receive the container instance as their only argument.
5. Bindings are **not** shared by default; each resolution creates a new instance.
6. `bindIf()` prevents overriding existing bindings (useful for package development).
7. `instance()` registers an already-constructed object as a shared instance.

#### Constraints and Limitations

- **Not Shared by Default:** `bind()` creates a new instance on each resolution; use `singleton()` for shared instances.
- **Registration Timing:** Bindings must be registered before they are resolved; late registration causes a `BindingResolutionException`.
- **Closure Complexity:** Complex closures should be extracted to factory classes for testability.
- **No Automatic Interface Binding:** Interfaces must be explicitly bound; the container cannot resolve them automatically.
- **Deferred Providers:** Bindings in deferred providers are only registered when the service is first resolved.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Binding an Interface to a Concrete Implementation

**Step-by-Step Setup Guide:**

1. Define an interface and a concrete implementation.
2. Register the binding in `AppServiceProvider`.
3. Type-hint the interface in a controller.
4. The container injects the concrete implementation automatically.

**Complete Executable Code:**

```php
<?php
// app/Contracts/Logger.php

namespace App\Contracts;

interface Logger
{
    public function log(string $message): void;
}
```

```php
<?php
// app/Services/DatabaseLogger.php

namespace App\Services;

use App\Contracts\Logger;

class DatabaseLogger implements Logger
{
    public function log(string $message): void
    {
        \DB::table('logs')->insert([
            'message' => $message,
            'created_at' => now(),
        ]);
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\Logger;
use App\Services\DatabaseLogger;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Map the Logger interface to DatabaseLogger
        $this->app->bind(Logger::class, DatabaseLogger::class);
    }
}
```

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Contracts\Logger;

class UserController extends Controller
{
    // The container injects DatabaseLogger automatically
    public function __construct(
        protected Logger $logger
    ) {}

    public function store()
    {
        $this->logger->log('User created');
        return response()->json(['status' => 'ok']);
    }
}
```

**Expected Output:**

- A POST request to the user store route inserts a log entry into the `logs` table.
- The controller receives a `DatabaseLogger` instance without explicit instantiation.

**Why This Code Produces That Result:**

- The `bind()` call maps `Logger` to `DatabaseLogger`.
- When the container resolves `UserController`, it inspects the constructor, sees `Logger`, and looks up the binding.
- The bound concrete class is instantiated and injected.

#### Example 2: Binding with a Closure for Complex Configuration

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\PaymentGateway;
use App\Services\StripeGateway;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(PaymentGateway::class, function ($app) {
            // Complex factory logic based on config
            $driver = config('services.payment.driver', 'stripe');

            return match ($driver) {
                'stripe' => new StripeGateway(
                    config('services.stripe.key'),
                    config('services.stripe.secret')
                ),
                'paypal' => new \App\Services\PayPalGateway(
                    config('services.paypal.client_id'),
                    config('services.paypal.secret')
                ),
                default => throw new \InvalidArgumentException("Unknown payment driver: {$driver}"),
            };
        });
    }
}
```

**Expected Output:**

- The container returns the appropriate gateway based on the configured driver.
- Switching the driver in `.env` changes the implementation without code changes.

**Why This Code Produces That Result:**

- The closure is executed on each resolution, reading the current config.
- `match` selects the concrete class based on the driver value.
- This pattern supports environment-specific implementations.

### Real-World Cases

**Case 1: Payment Gateway Abstraction**

A SaaS platform binds a `PaymentGateway` interface to either Stripe or PayPal based on the tenant's configuration, allowing per-tenant payment processing.

**Case 2: Storage Abstraction**

An application binds a `FileStorage` interface to either local storage or S3 based on environment configuration.

**Case 3: Notification Channels**

A notification service binds different channel implementations (email, SMS, push) to a common `NotificationChannel` interface.

### References

- Laravel Service Container: Binding - https://laravel.com/docs/12.x/container#binding
- Laravel Service Container: Binding Basics - https://laravel.com/docs/12.x/container#binding-basics
- Laravel API: Container::bind - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html#method_bind


## 3. Singletons (Ensuring a Class is Resolved Exactly Once per Application Lifecycle via `$this->app->singleton()`)

### Definitions

**Core Definition:** A singleton binding ensures that the container resolves and returns the same instance of a class every time it is requested, for the duration of the application lifecycle.

**Technical Definition:** The `singleton()` method registers a binding with the `$shared` flag set to `true`. On the first resolution, the container instantiates the concrete and stores the instance in its `$instances` array. Subsequent resolutions return the cached instance without re-executing the resolver. This is ideal for stateful services, database connections, and configuration objects that should not be duplicated.

**Beginner-Friendly Explanation:** A singleton is like a company's CEO. No matter how many times you ask "Who's the CEO?", you get the same person. The container builds the instance once and reuses it everywhere. This is useful for things like a database connection pool or a configuration manager.

### Purposes

- To ensure a single, shared instance of a service across the application.
- To maintain state consistency (e.g., a shopping cart or session manager).
- To avoid the overhead of repeatedly constructing expensive objects.
- To share configuration or connection pools.
- To implement the Singleton design pattern within the container's DI framework.
- To provide a single point of access for shared resources.
- To support features like caching and logging where a single instance is preferred.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// In a Service Provider's register() method

namespace App\Providers;

use App\Services\AnalyticsService;
use App\Contracts\CacheManager;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Simple singleton binding
        $this->app->singleton(AnalyticsService::class);

        // Singleton with a closure
        $this->app->singleton(CacheManager::class, function ($app) {
            return new \App\Services\RedisCacheManager(
                $app->make('redis')
            );
        });

        // Singleton with a class name
        $this->app->singleton(CacheManager::class, \App\Services\RedisCacheManager::class);

        // Register only if not already bound
        $this->app->singletonIf(AnalyticsService::class);

        // Bind an existing instance as a singleton
        $this->app->instance(AnalyticsService::class, new AnalyticsService());
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `singleton($abstract, $concrete)` | Registers a shared binding |
| `singletonIf()` | Registers only if not already bound |
| `instance()` | Registers an existing object as shared |
| Closure `$app` param | Container instance for resolving dependencies |
| `$instances` array | Internal storage for shared instances |

#### Syntax Rules

1. `singleton()` registers a binding with the shared flag set to `true`.
2. The first resolution instantiates and caches the instance; subsequent resolutions return the cached instance.
3. `singleton()` can be used with a Closure, class name, or no concrete (self-binding).
4. `singletonIf()` prevents overriding existing bindings.
5. `instance()` registers an already-constructed object, which is immediately shared.
6. Singletons are reset between requests in traditional PHP-FPM but **persist across requests** in Laravel Octane.

#### Constraints and Limitations

- **State Leakage in Octane:** Singletons persist across requests in Octane, so mutable state can leak between requests. Use `scoped()` instead.
- **Testing Complexity:** Singletons can retain state between tests, causing flaky tests. Use `$this->app->forgetInstance()` in test teardown.
- **Memory:** Long-lived singletons hold memory for the entire application lifecycle.
- **Circular Dependencies:** Singletons with circular dependencies can cause deadlocks.
- **Not a Replacement for Static:** Singletons are container-managed; they are not the same as PHP static properties.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Singleton Analytics Service

**Step-by-Step Setup Guide:**

1. Create an `AnalyticsService` that accumulates events.
2. Register it as a singleton.
3. Inject it into multiple controllers.
4. Verify that events accumulate across controllers.

**Complete Executable Code:**

```php
<?php
// app/Services/AnalyticsService.php

namespace App\Services;

class AnalyticsService
{
    protected array $events = [];

    public function track(string $event, array $data = []): void
    {
        $this->events[] = ['event' => $event, 'data' => $data, 'at' => now()];
    }

    public function getEvents(): array
    {
        return $this->events;
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\AnalyticsService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Register as singleton — one shared instance
        $this->app->singleton(AnalyticsService::class);
    }
}
```

```php
<?php
// app/Http/Controllers/PageController.php

namespace App\Http\Controllers;

use App\Services\AnalyticsService;

class PageController extends Controller
{
    public function __construct(
        protected AnalyticsService $analytics
    ) {}

    public function home()
    {
        $this->analytics->track('page_view', ['page' => 'home']);
        return view('home');
    }

    public function about()
    {
        $this->analytics->track('page_view', ['page' => 'about']);
        // The same singleton instance — events accumulate
        return response()->json($this->analytics->getEvents());
    }
}
```

**Expected Output:**

- Visiting `/home` tracks a `page_view` event.
- Visiting `/about` tracks another event; `getEvents()` returns **both** events, proving the same instance was used.

**Why This Code Produces That Result:**

- `singleton()` caches the first `AnalyticsService` instance.
- Both controllers receive the same instance, so `$events` accumulates.
- If `bind()` were used instead, each controller would get a fresh instance with an empty `$events` array.

#### Example 2: Singleton with Closure for Database Connection

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\DatabaseConnection;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(DatabaseConnection::class, function ($app) {
            // Expensive connection setup — done only once
            $dsn = config('database.connections.mysql');
            $connection = new \PDO(
                "mysql:host={$dsn['host']};dbname={$dsn['database']}",
                $dsn['username'],
                $dsn['password'],
                [\PDO::ATTR_ERRMODE => \PDO::ERRMODE_EXCEPTION]
            );

            return new DatabaseConnection($connection);
        });
    }
}
```

**Expected Output:**

- The PDO connection is established only once, regardless of how many times `DatabaseConnection` is resolved.
- All consumers share the same connection.

**Why This Code Produces That Result:**

- The closure is executed only on the first resolution.
- The resulting instance is cached and returned for all subsequent resolutions.
- This avoids the overhead of reconnecting.

### Real-World Cases

**Case 1: Configuration Manager**

A `ConfigManager` singleton holds application configuration loaded from multiple sources (files, database, external API) and is shared across all services.

**Case 2: Shopping Cart**

An e-commerce application uses a singleton `CartService` to maintain the user's cart state across requests within the same session.

**Case 3: Logger**

A logging service registered as a singleton buffers log entries and flushes them at the end of the request.

### References

- Laravel Service Container: Binding Singletons - https://laravel.com/docs/12.x/container#binding-singletons
- Laravel API: Container::singleton - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html#method_singleton
- Laravel Octane: Scoped Bindings - https://laravel.com/docs/12.x/octane#scoped-bindings


## 4. Contextual Bindings (Injecting Different Implementations into Specific Classes Using `$this->app->when()->needs()->give()`)

### Definitions

**Core Definition:** Contextual binding allows the container to inject different implementations of the same interface into different classes, based on the consuming class's identity.

**Technical Definition:** The `when()` method defines the context (the consuming class or classes), `needs()` specifies the abstract dependency, and `give()` provides the concrete implementation (a Closure, class name, or array). The container stores this in its `$contextual` map and consults it during resolution before falling back to global bindings.

**Beginner-Friendly Explanation:** Imagine two chefs who both need "oil." The Italian chef needs olive oil; the pastry chef needs butter. Contextual binding lets you say: "When the Italian chef asks for oil, give olive oil. When the pastry chef asks for oil, give butter." Same dependency, different implementations, based on who's asking.

### Purposes

- To inject different implementations of the same interface into different classes.
- To override a global binding for a specific consumer.
- To provide specialized configurations for different parts of the application.
- To support multiple storage backends (e.g., local vs. S3) for different controllers.
- To customize third-party package dependencies for specific classes.
- To avoid creating wrapper classes just to vary a dependency.
- To support the Strategy pattern at the container level.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// In a Service Provider's register() method

namespace App\Providers;

use App\Http\Controllers\PhotoController;
use App\Http\Controllers\VideoController;
use App\Http\Controllers\UploadController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // When PhotoController needs Filesystem, give local disk
        $this->app->when(PhotoController::class)
            ->needs(Filesystem::class)
            ->give(function () {
                return Storage::disk('local');
            });

        // When VideoController or UploadController needs Filesystem, give S3
        $this->app->when([VideoController::class, UploadController::class])
            ->needs(Filesystem::class)
            ->give(function () {
                return Storage::disk('s3');
            });

        // Give an array of implementations (for variadic dependencies)
        $this->app->when(Firewall::class)
            ->needs(Filter::class)
            ->give([
                NullFilter::class,
                ProfanityFilter::class,
                TooLongFilter::class,
            ]);

        // Give a closure that returns an array
        $this->app->when(Firewall::class)
            ->needs(Filter::class)
            ->give(function ($app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(ProfanityFilter::class),
                    $app->make(TooLongFilter::class),
                ];
            });

        // Give tagged bindings
        $this->app->when(ReportAggregator::class)
            ->needs(Report::class)
            ->giveTagged('reports');
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `when($concrete)` | Defines the consuming class(es) |
| `needs($abstract)` | The dependency to customize |
| `give($implementation)` | The implementation (Closure, class, array) |
| `giveTagged($tag)` | Injects all bindings with a tag |
| `when([...])` | Accepts an array of consuming classes |

#### Syntax Rules

1. `when()` **must** be called before `needs()`, which **must** be called before `give()`.
2. `when()` accepts a class name or an array of class names.
3. `needs()` accepts the abstract type (interface or class name) being customized.
4. `give()` accepts a Closure, a class name string, or an array of class names.
5. For variadic dependencies, `give()` can return an array of implementations.
6. `giveTagged()` injects all container bindings with the specified tag.
7. Contextual bindings take precedence over global bindings for the specified consumer.

#### Constraints and Limitations

- **Consumer-Specific:** Contextual bindings only apply to the specified consuming class; other consumers use the global binding.
- **No Automatic Inheritance:** Contextual bindings do not automatically apply to subclasses of the specified consumer.
- **Registration Order:** Contextual bindings must be registered before the consumer is resolved.
- **Complexity:** Overuse of contextual bindings can make the dependency graph hard to reason about.
- **No Primitive Values:** Contextual bindings cannot inject primitive values (use `makeWith()` for those).

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Different Storage Disks for Different Controllers

**Step-by-Step Setup Guide:**

1. Define a `Filesystem` interface dependency in two controllers.
2. Register contextual bindings for each controller.
3. Resolve the controllers and verify different disks are injected.

**Complete Executable Code:**

```php
<?php
// app/Http/Controllers/PhotoController.php

namespace App\Http\Controllers;

use Illuminate\Contracts\Filesystem\Filesystem;

class PhotoController extends Controller
{
    public function __construct(
        protected Filesystem $disk
    ) {}

    public function store()
    {
        // Uses local disk (contextual binding)
        $this->disk->put('photo.jpg', 'data');
        return response()->json(['disk' => $this->disk->getDriver()->getAdapter()]);
    }
}
```

```php
<?php
// app/Http/Controllers/VideoController.php

namespace App\Http\Controllers;

use Illuminate\Contracts\Filesystem\Filesystem;

class VideoController extends Controller
{
    public function __construct(
        protected Filesystem $disk
    ) {}

    public function store()
    {
        // Uses S3 disk (contextual binding)
        $this->disk->put('video.mp4', 'data');
        return response()->json(['disk' => $this->disk->getDriver()->getAdapter()]);
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Http\Controllers\PhotoController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // PhotoController gets the local disk
        $this->app->when(PhotoController::class)
            ->needs(Filesystem::class)
            ->give(fn () => Storage::disk('local'));

        // VideoController gets the S3 disk
        $this->app->when(VideoController::class)
            ->needs(Filesystem::class)
            ->give(fn () => Storage::disk('s3'));
    }
}
```

**Expected Output:**

- `PhotoController` receives the local disk; `VideoController` receives S3.
- Uploading a photo stores it locally; uploading a video stores it on S3.
- Both controllers share the same `Filesystem` interface.

**Why This Code Produces That Result:**

- The contextual bindings map each controller to a different disk.
- When the container resolves `PhotoController`, it consults the contextual map and uses the local disk.
- When resolving `VideoController`, it uses S3.

#### Example 2: Variadic Dependencies with Contextual Binding

```php
<?php
// app/Services/Firewall.php

namespace App\Services;

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
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
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\Firewall;
use App\Models\Filter;
use App\Filters\NullFilter;
use App\Filters\ProfanityFilter;
use App\Filters\TooLongFilter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Inject an array of Filter implementations into Firewall
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

- `Firewall` receives three `Filter` instances, each resolved by the container.
- The firewall applies all filters sequentially.

**Why This Code Produces That Result:**

- `give()` with an array tells the container to resolve each class and inject them as variadic arguments.
- This eliminates the need to manually construct the filter array.

### Real-World Cases

**Case 1: Multi-Cloud Storage**

A SaaS application stores user avatars locally but backups on S3. Contextual binding injects the appropriate disk into each controller.

**Case 2: Different Payment Processors**

A marketplace uses Stripe for buyers and PayPal for seller payouts. Contextual binding injects the correct gateway into each service.

**Case 3: Testing with Mocks**

In tests, a specific controller can be given a mock implementation of a dependency while other controllers use the real one.

### References

- Laravel Service Container: Contextual Binding - https://laravel.com/docs/12.x/container#contextual-binding
- Laravel Service Container: Variadic Tag Dependencies - https://laravel.com/docs/12.x/container#variadic-tag-dependencies
- Laravel API: ContextualBindingBuilder - https://api.laravel.com/docs/11.x/Illuminate/Container/ContextualBindingBuilder.html


## 5. Automatic Resolution (Zero-Configuration Dependency Injection via PHP Reflection API)

### Definitions

**Core Definition:** Automatic resolution is the container's ability to instantiate a class and inject its dependencies without any explicit binding, by inspecting the class's constructor type-hints using PHP's Reflection API.

**Technical Definition:** When `make()` is called for an unbound concrete class, the container uses `ReflectionClass` to inspect the constructor. For each parameter with a class type-hint, it recursively resolves that dependency. For parameters with default values, it uses the default. For untyped or primitive parameters without defaults, it throws a `BindingResolutionException`. This process is recursive, building the entire dependency tree.

**Beginner-Friendly Explanation:** Automatic resolution is like a self-assembling Lego set. You say "build me a `PodcastController`," and the container reads the instructions (constructor), sees it needs an `AppleMusic` service, builds that first (which might need its own parts), and assembles everything. You don't have to write a single binding.

### Purposes

- To eliminate the need for explicit bindings for concrete classes.
- To reduce configuration boilerplate in service providers.
- To enable rapid development with minimal setup.
- To support constructor injection throughout the application.
- To automatically resolve dependencies in controllers, commands, jobs, and listeners.
- To recursively build complex object graphs.
- To make dependency injection the default, not the exception.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// No binding required — the container resolves automatically

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use App\Services\PodcastParser;

class PodcastController extends Controller
{
    /**
     * The container inspects this constructor via Reflection,
     * sees AppleMusic and PodcastParser type-hints, resolves
     * each recursively, and injects them.
     */
    public function __construct(
        protected AppleMusic $apple,
        protected PodcastParser $parser
    ) {}

    public function show(string $id)
    {
        $podcast = $this->apple->findPodcast($id);
        return view('podcasts.show', [
            'podcast' => $this->parser->parse($podcast),
        ]);
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `ReflectionClass` | Inspects the class constructor |
| `ReflectionParameter` | Examines each constructor parameter |
| `getType()` | Returns the type-hint (class name) |
| Recursive resolution | Resolves each dependency's dependencies |
| Default values | Used for optional parameters |
| `BindingResolutionException` | Thrown for unresolvable primitives |

#### Syntax Rules

1. Automatic resolution works only for **concrete classes** (not interfaces or abstract classes).
2. The container inspects the **constructor's type-hints** using PHP Reflection.
3. Each type-hinted class dependency is recursively resolved.
4. Parameters with default values are optional and use the default if not resolved.
5. Primitive parameters (string, int, bool) without defaults require explicit values via `makeWith()`.
6. Interfaces **must** be bound; they cannot be auto-resolved.
7. Automatic resolution applies to controllers, event listeners, middleware, queued jobs, and route closures.

#### Constraints and Limitations

- **Interfaces and Abstracts:** Cannot be auto-resolved; require explicit bindings.
- **Primitive Parameters:** Cannot be auto-resolved without defaults; require `makeWith()`.
- **Performance:** Reflection has overhead; caching (e.g., Octane) mitigates this.
- **Circular Dependencies:** Cause infinite recursion; must be broken manually.
- **No Setter Injection:** Automatic resolution only inspects constructors, not setters.
- **Variadic Parameters:** Supported but require contextual binding or explicit arrays.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Automatic Resolution in a Route Closure

**Step-by-Step Setup Guide:**

1. Create a service class with no dependencies.
2. Type-hint it in a route closure.
3. The container resolves it automatically.

**Complete Executable Code:**

```php
<?php
// app/Services/ReportGenerator.php

namespace App\Services;

class ReportGenerator
{
    public function generate(): string
    {
        return 'Report generated at ' . now()->toDateTimeString();
    }
}
```

```php
<?php
// routes/web.php

use App\Services\ReportGenerator;

Route::get('/report', function (ReportGenerator $generator) {
    // The container automatically resolves ReportGenerator
    return $generator->generate();
});
```

**Expected Output:**

- `GET /report` returns `"Report generated at 2026-10-06 12:00:00"`.
- No binding was registered; the container used Reflection.

**Why This Code Produces That Result:**

- The route closure type-hints `ReportGenerator`.
- The container inspects its constructor (no dependencies) and instantiates it.
- The instance is injected into the closure.

#### Example 2: Recursive Automatic Resolution

```php
<?php
// app/Services/DatabaseConnection.php

namespace App\Services;

class DatabaseConnection
{
    public function __construct(
        protected string $host = 'localhost',
        protected string $database = 'app'
    ) {}

    public function query(string $sql): array
    {
        return ['result' => "Queried {$this->database} on {$this->host}"];
    }
}
```

```php
<?php
// app/Services/UserRepository.php

namespace App\Services;

class UserRepository
{
    public function __construct(
        protected DatabaseConnection $connection
    ) {}

    public function findAll(): array
    {
        return $this->connection->query('SELECT * FROM users');
    }
}
```

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Services\UserRepository;

class UserController extends Controller
{
    /**
     * The container resolves UserRepository, which requires
     * DatabaseConnection. Both are auto-resolved recursively.
     */
    public function __construct(
        protected UserRepository $users
    ) {}

    public function index()
    {
        return response()->json($this->users->findAll());
    }
}
```

**Expected Output:**

- `GET /users` returns `{"result": "Queried app on localhost"}`.
- The container recursively resolved `DatabaseConnection` to build `UserRepository`.

**Why This Code Produces That Result:**

- The container inspects `UserController` → needs `UserRepository` → needs `DatabaseConnection`.
- `DatabaseConnection` has primitive parameters with defaults, so the container uses the defaults.
- The entire tree is built automatically.

### Real-World Cases

**Case 1: Rapid Prototyping**

A developer scaffolds controllers and services without writing a single binding; the container auto-resolves everything.

**Case 2: Package Development**

A package provides concrete classes that consumers can type-hint without additional configuration.

**Case 3: Testing**

Tests type-hint concrete classes in method signatures; the container resolves them automatically.

### References

- Laravel Service Container: Zero Configuration Resolution - https://laravel.com/docs/12.x/container#zero-configuration-resolution
- Laravel Service Container: Automatic Injection - https://laravel.com/docs/12.x/container#automatic-injection
- PHP Reflection API - https://www.php.net/manual/en/book.reflection.php


## 6. Enhanced: Container Tagging (`$this->app->tag()` and `$this->app->tagged()` for Resolving Collections of Related Instances)

### Definitions

**Core Definition:** Container tagging is a mechanism for assigning one or more string labels (tags) to a group of bindings, allowing them to be resolved collectively as an array via the `tagged()` method.

**Technical Definition:** The `tag()` method associates an array of abstract types with one or more tag names, stored in the container's `$tags` array. The `tagged()` method resolves all bindings associated with a tag and returns an iterable collection of instances. This is useful for implementing plugin architectures, report aggregators, and strategy collections.

**Beginner-Friendly Explanation:** Tagging is like putting colored stickers on items in a warehouse. You stick a "fragile" tag on several boxes, then later say "bring me everything tagged fragile." The container collects all tagged items and returns them as a group.

### Purposes

- To resolve all implementations of a common interface as a collection.
- To implement plugin or extension architectures.
- To build report aggregators, validators, or filter chains.
- To inject multiple related services into a single consumer.
- To avoid hard-coding a list of implementations.
- To support modular, extensible application design.
- To group services for batch processing.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// In a Service Provider's register() method

namespace App\Providers;

use App\Reports\CpuReport;
use App\Reports\MemoryReport;
use App\Reports\DiskReport;
use App\Services\ReportAggregator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Register individual bindings
        $this->app->bind(CpuReport::class, function () {
            return new CpuReport();
        });

        $this->app->bind(MemoryReport::class, function () {
            return new MemoryReport();
        });

        $this->app->bind(DiskReport::class, function () {
            return new DiskReport();
        });

        // Tag them with 'reports'
        $this->app->tag([
            CpuReport::class,
            MemoryReport::class,
            DiskReport::class,
        ], 'reports');

        // Resolve all tagged bindings
        $this->app->bind(ReportAggregator::class, function ($app) {
            return new ReportAggregator($app->tagged('reports'));
        });
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `tag($abstracts, $tags)` | Assigns tags to bindings |
| `$abstracts` | Array or string of abstract types |
| `$tags` | Array or string of tag names |
| `tagged($tag)` | Resolves all bindings with the tag |
| `giveTagged($tag)` | Contextual injection of tagged bindings |

#### Syntax Rules

1. `tag()` accepts an array of abstracts and an array (or string) of tags.
2. A binding can have multiple tags.
3. `tagged()` returns an iterable collection of resolved instances.
4. `tagged()` resolves each binding fresh (unless they are singletons).
5. `giveTagged()` can be used in contextual bindings to inject tagged services.
6. Tags are stored in the container's `$tags` property.
7. Tagging does **not** instantiate the services; resolution happens on `tagged()`.

#### Constraints and Limitations

- **Resolution Order:** `tagged()` resolves in the order the bindings were tagged, not necessarily registration order.
- **No Lazy Resolution:** All tagged bindings are resolved when `tagged()` is called, which can be expensive.
- **Tag Name Collisions:** Using the same tag name across different packages can cause unexpected inclusions.
- **No Type Safety:** `tagged()` returns a mixed collection; consumers must filter by type if needed.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Report Aggregator with Tagged Reports

**Step-by-Step Setup Guide:**

1. Create multiple report classes implementing a common interface.
2. Register and tag them.
3. Inject the tagged collection into an aggregator.
4. Resolve the aggregator and verify all reports are included.

**Complete Executable Code:**

```php
<?php
// app/Contracts/Report.php

namespace App\Contracts;

interface Report
{
    public function generate(): array;
}
```

```php
<?php
// app/Reports/CpuReport.php
namespace App\Reports;
use App\Contracts\Report;

class CpuReport implements Report
{
    public function generate(): array
    {
        return ['cpu' => '45%'];
    }
}
```

```php
<?php
// app/Reports/MemoryReport.php
namespace App\Reports;
use App\Contracts\Report;

class MemoryReport implements Report
{
    public function generate(): array
    {
        return ['memory' => '62%'];
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
    public function __construct(
        protected iterable $reports
    ) {}

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

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\Report;
use App\Reports\CpuReport;
use App\Reports\MemoryReport;
use App\Services\ReportAggregator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind implementations to the interface
        $this->app->bind(Report::class, CpuReport::class);
        $this->app->bind(Report::class, MemoryReport::class);

        // Tag them
        $this->app->tag([CpuReport::class, MemoryReport::class], 'reports');

        // Inject tagged bindings into the aggregator
        $this->app->bind(ReportAggregator::class, function ($app) {
            return new ReportAggregator($app->tagged('reports'));
        });
    }
}
```

**Expected Output:**

- Resolving `ReportAggregator` yields an aggregator with both `CpuReport` and `MemoryReport`.
- `all()` returns `['cpu' => '45%', 'memory' => '62%']`.

**Why This Code Produces That Result:**

- `tag()` associates both report classes with the `reports` tag.
- `tagged('reports')` resolves both and returns them as an iterable.
- The aggregator receives both and merges their results.

#### Example 2: Plugin Architecture with Tags

```php
<?php
// app/Providers/PluginServiceProvider.php

namespace App\Providers;

use App\Plugins\AnalyticsPlugin;
use App\Plugins\SeoPlugin;
use App\Plugins\CachePlugin;
use Illuminate\Support\ServiceProvider;

class PluginServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(AnalyticsPlugin::class);
        $this->app->singleton(SeoPlugin::class);
        $this->app->singleton(CachePlugin::class);

        $this->app->tag([
            AnalyticsPlugin::class,
            SeoPlugin::class,
            CachePlugin::class,
        ], 'plugins');
    }
}
```

```php
<?php
// app/Services/PluginManager.php

namespace App\Services;

class PluginManager
{
    public function __construct(
        protected iterable $plugins
    ) {}

    public function boot(): void
    {
        foreach ($this->plugins as $plugin) {
            $plugin->register();
        }
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\PluginManager;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(PluginManager::class, function ($app) {
            return new PluginManager($app->tagged('plugins'));
        });
    }
}
```

**Expected Output:**

- `PluginManager` receives all three plugins and calls `register()` on each.
- Adding a new plugin requires only tagging it—no changes to `PluginManager`.

**Why This Code Produces That Result:**

- Tagging decouples the plugin list from the manager.
- New plugins are added by tagging them in the service provider.
- The manager receives the full collection automatically.

### Real-World Cases

**Case 1: Report Aggregator**

A monitoring dashboard aggregates CPU, memory, disk, and network reports, each tagged `reports`, into a single view.

**Case 2: Validation Pipeline**

Multiple validators are tagged `validators` and injected into a validation service that runs them sequentially.

**Case 3: Plugin System**

A CMS allows plugins to register themselves by tagging their bindings `plugins`; the plugin manager boots all tagged plugins.

### References

- Laravel Service Container: Tagging - https://laravel.com/docs/12.x/container#tagging
- Laravel Service Container: Variadic Tag Dependencies - https://laravel.com/docs/12.x/container#variadic-tag-dependencies
- Laravel API: Container::tag - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html#method_tag


## 7. Enhanced: Extending Bindings (`$this->app->extend()` to Modify, Wrap, or Decorate Resolved Service Instances)

### Definitions

**Core Definition:** Extending bindings allows you to intercept a service after it is resolved and modify, wrap, or decorate it before it is returned to the consumer, without changing the original binding.

**Technical Definition:** The `extend()` method registers a closure in the container's `$extenders` array. When the specified abstract is resolved, the container calls each registered extender closure with the resolved instance and the container. The closure returns the modified instance, which is then cached (if the binding is shared) and returned.

**Beginner-Friendly Explanation:** Extending a binding is like gift-wrapping a present after it's been bought. The original item (service) is unchanged, but you add a bow (decoration), a card (extra configuration), or put it in a nicer box (wrapper). The recipient (consumer) sees the wrapped version, but the original is intact underneath.

### Purposes

- To decorate a service with additional behavior.
- To wrap a service with a proxy or adapter.
- To configure a service after instantiation.
- To modify third-party package services without forking them.
- To add cross-cutting concerns (logging, caching) to services.
- To replace a service's implementation conditionally.
- To compose services with additional functionality.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// In a Service Provider's register() or boot() method

namespace App\Providers;

use App\Services\Service;
use App\Services\DecoratedService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Register the base service
        $this->app->bind(Service::class, function () {
            return new Service();
        });

        // Extend it with a decorator
        $this->app->extend(Service::class, function (Service $service, $app) {
            return new DecoratedService($service);
        });

        // Multiple extenders are applied in registration order
        $this->app->extend(Service::class, function (Service $service, $app) {
            $service->setExtraConfig(config('service.extra'));
            return $service;
        });
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `extend($abstract, $closure)` | Registers an extender |
| `$service` | The resolved instance being extended |
| `$app` | The container instance |
| Return value | The modified instance to be used |
| Multiple extenders | Applied in registration order |

#### Syntax Rules

1. `extend()` must be called after the binding is registered.
2. The closure receives the resolved instance as its first argument and the container as its second.
3. The closure **must** return the modified instance.
4. Multiple extenders can be registered for the same abstract; they are applied in order.
5. Extenders are called only once per resolution (for shared bindings).
6. Extenders work with `bind()`, `singleton()`, and `instance()` bindings.
7. Extenders are stored in the container's `$extenders` array.

#### Constraints and Limitations

- **Order Matters:** Extenders are applied in the order they are registered.
- **Shared Bindings:** For singletons, extenders are applied only on the first resolution.
- **No Un-Extend:** There is no built-in way to remove an extender once registered.
- **Type Safety:** The extender must return an instance compatible with the original binding.
- **Performance:** Each extender adds a closure call during resolution.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Decorating a Service with Logging

**Step-by-Step Setup Guide:**

1. Create a base service.
2. Create a decorator that wraps the base service and adds logging.
3. Register the base binding and extend it.
4. Resolve the service and verify the decorated behavior.

**Complete Executable Code:**

```php
<?php
// app/Contracts/Notifier.php

namespace App\Contracts;

interface Notifier
{
    public function send(string $message): void;
}
```

```php
<?php
// app/Services/EmailNotifier.php

namespace App\Services;

use App\Contracts\Notifier;

class EmailNotifier implements Notifier
{
    public function send(string $message): void
    {
        \Log::info("Email sent: {$message}");
    }
}
```

```php
<?php
// app/Services/LoggedNotifier.php

namespace App\Services;

use App\Contracts\Notifier;

class LoggedNotifier implements Notifier
{
    public function __construct(
        protected Notifier $inner
    ) {}

    public function send(string $message): void
    {
        \Log::info("Notifier called with: {$message}");
        $this->inner->send($message);
        \Log::info("Notifier completed");
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Contracts\Notifier;
use App\Services\EmailNotifier;
use App\Services\LoggedNotifier;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Register the base notifier
        $this->app->bind(Notifier::class, EmailNotifier::class);

        // Extend it with the logging decorator
        $this->app->extend(Notifier::class, function (Notifier $notifier, $app) {
            return new LoggedNotifier($notifier);
        });
    }
}
```

```php
<?php
// Usage

$notifier = app(Notifier::class);
$notifier->send('Hello, world!');

// Log output:
// [info] Notifier called with: Hello, world!
// [info] Email sent: Hello, world!
// [info] Notifier completed
```

**Expected Output:**

- The `LoggedNotifier` wraps `EmailNotifier`.
- Two additional log entries are produced (before and after).
- The original `EmailNotifier` behavior is preserved.

**Why This Code Produces That Result:**

- `extend()` intercepts the resolution of `Notifier`.
- The resolved `EmailNotifier` is passed to the extender closure.
- The extender returns a `LoggedNotifier` that delegates to the original.
- The consumer receives the decorated instance.

#### Example 2: Configuring a Third-Party Service

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Cashier\Cashier;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Extend Cashier's configuration after resolution
        $this->app->extend(Cashier::class, function (Cashier $cashier, $app) {
            $cashier->setCurrency('EUR');
            $cashier->setTaxPercentage(21.0);
            return $cashier;
        });
    }
}
```

**Expected Output:**

- Cashier is configured with EUR currency and 21% tax after resolution.
- All consumers of Cashier receive the configured instance.

**Why This Code Produces That Result:**

- `extend()` allows post-resolution configuration without modifying the package.
- The extender runs after Cashier is instantiated.
- The configured instance is cached (if shared) and returned.

### Real-World Cases

**Case 1: Caching Decorator**

A repository binding is extended with a `CachedRepository` decorator that checks a cache before querying the database.

**Case 2: Logging Decorator**

A service binding is extended with a `LoggingService` that logs method calls for debugging.

**Case 3: Third-Party Configuration**

A package service is extended to apply application-specific configuration without editing vendor files.

### References

- Laravel Service Container: Extending Bindings - https://laravel.com/docs/12.x/container#extending-bindings
- Laravel API: Container::extend - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html#method_extend
- Laravel News: Decorating Services in Laravel - https://laravel-news.com/decorating-services


## 8. Enhanced: Scoped Bindings (Registering Objects That Should Be Singletons Only for the Duration of a Specific Request/Job Lifecycle)

### Definitions

**Core Definition:** Scoped bindings are a hybrid between `bind()` and `singleton()`: the container resolves and caches an instance once per "scope" (a request or job lifecycle), and the instance is flushed when the scope ends, ensuring fresh instances in subsequent scopes.

**Technical Definition:** The `scoped()` method registers a binding with a `scoped` flag. The instance is stored in the container's `$scopedInstances` array, keyed by the abstract name. When the scope is flushed (via `$app->forgetScopedInstances()` or automatically by Laravel Octane between requests), the instance is removed, and the next resolution creates a new one. This is essential for Octane compatibility, where singletons persist across requests and can leak state.

**Beginner-Friendly Explanation:** A scoped binding is like a whiteboard in a meeting room. During the meeting (request), everyone writes on the same whiteboard. When the meeting ends (request ends), the whiteboard is erased. The next meeting gets a clean whiteboard. This prevents one meeting's notes from confusing the next.

### Purposes

- To share instances within a single request or job without leaking state across requests.
- To ensure Octane compatibility by resetting state between requests.
- To provide request-scoped caching or context objects.
- To avoid the pitfalls of singletons in long-lived workers.
- To support per-request configuration that shouldn't persist.
- To safely share database transactions or session data within a request.
- To replace singletons in Octane environments without code changes to consumers.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// In a Service Provider's register() method

namespace App\Providers;

use App\Services\RequestContext;
use App\Services\ReportGenerator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Simple scoped binding
        $this->app->scoped(RequestContext::class);

        // Scoped binding with a closure
        $this->app->scoped(ReportGenerator::class, function ($app) {
            return new ReportGenerator(
                $app->make(RequestContext::class)
            );
        });

        // Scoped binding with a class name
        $this->app->scoped(ReportGenerator::class, \App\Services\ReportGenerator::class);

        // Register only if not already bound
        $this->app->scopedIf(RequestContext::class);
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `scoped($abstract, $concrete)` | Registers a per-scope shared binding |
| `scopedIf()` | Registers only if not already bound |
| `$scopedInstances` | Internal array for scoped instances |
| `forgetScopedInstances()` | Flushes scoped instances (called by Octane) |

#### Syntax Rules

1. `scoped()` registers a binding that is shared **within** a scope but **not across** scopes.
2. The instance is cached in `$scopedInstances` on first resolution within a scope.
3. At the end of the scope, `forgetScopedInstances()` is called automatically (by Octane or manually).
4. `scopedIf()` prevents overriding existing bindings.
5. Scoped bindings can use Closures, class names, or self-binding.
6. In non-Octane applications, scoped bindings behave like singletons within a single request (since each request is a new PHP process).
7. In Octane, scoped bindings are **essential** for any stateful service.

#### Constraints and Limitations

- **Octane-Specific Benefit:** In traditional PHP-FPM, scoped bindings behave like singletons per request; the benefit is primarily in Octane.
- **Manual Flush Required Outside Octane:** If you need to flush scoped instances manually (e.g., in a long-running command), you must call `$app->forgetScopedInstances()`.
- **No Cross-Request Sharing:** Scoped instances cannot be shared across requests; use `singleton()` for that.
- **Testing:** Tests may need to flush scoped instances between test cases.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Request Context in an Octane Application

**Step-by-Step Setup Guide:**

1. Create a `RequestContext` service that holds request-specific data.
2. Register it as scoped.
3. Inject it into multiple services within the same request.
4. Verify it resets between requests in Octane.

**Complete Executable Code:**

```php
<?php
// app/Services/RequestContext.php

namespace App\Services;

class RequestContext
{
    protected array $data = [];

    public function set(string $key, mixed $value): void
    {
        $this->data[$key] = $value;
    }

    public function get(string $key, mixed $default = null): mixed
    {
        return $this->data[$key] ?? $default;
    }

    public function all(): array
    {
        return $this->data;
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\RequestContext;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Scoped: one instance per request, reset between requests
        $this->app->scoped(RequestContext::class);
    }
}
```

```php
<?php
// app/Http/Controllers/DashboardController.php

namespace App\Http\Controllers;

use App\Services\RequestContext;

class DashboardController extends Controller
{
    public function __construct(
        protected RequestContext $context
    ) {}

    public function index()
    {
        $this->context->set('user_id', auth()->id());
        $this->context->set('request_time', now()->toIso8601String());

        return response()->json($this->context->all());
    }
}
```

**Expected Output:**

- Each request gets a fresh `RequestContext` with empty data.
- In Octane, the context from request 1 does **not** leak into request 2.

**Why This Code Produces That Result:**

- `scoped()` caches the instance within the request.
- Octane calls `forgetScopedInstances()` between requests, clearing the cache.
- The next request resolves a new `RequestContext`.

#### Example 2: Scoped Database Transaction

```php
<?php
// app/Services/TransactionManager.php

namespace App\Services;

use Illuminate\Support\Facades\DB;

class TransactionManager
{
    protected bool $inTransaction = false;

    public function begin(): void
    {
        if (!$this->inTransaction) {
            DB::beginTransaction();
            $this->inTransaction = true;
        }
    }

    public function commit(): void
    {
        if ($this->inTransaction) {
            DB::commit();
            $this->inTransaction = false;
        }
    }

    public function rollback(): void
    {
        if ($this->inTransaction) {
            DB::rollBack();
            $this->inTransaction = false;
        }
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Services\TransactionManager;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Scoped: transaction state is per-request, reset between requests
        $this->app->scoped(TransactionManager::class);
    }
}
```

**Expected Output:**

- Within a request, multiple services share the same `TransactionManager`.
- Between requests, the transaction state is reset.
- In Octane, no transaction state leaks from one request to the next.

**Why This Code Produces That Result:**

- `scoped()` ensures the transaction manager is shared within the request but not across requests.
- Octane's scope flushing resets `$inTransaction` between requests.

### Real-World Cases

**Case 1: Octane-Powered API**

An API running on Laravel Octane uses scoped bindings for request-specific services like `RequestContext`, `AuthContext`, and `TenantResolver`, ensuring no state leaks between requests.

**Case 2: Multi-Tenant SaaS**

A multi-tenant SaaS uses a scoped `TenantContext` to hold the current tenant for the duration of a request, preventing tenant data from leaking across requests in Octane.

**Case 3: Background Job Processing**

A queue worker uses scoped bindings to ensure job-specific state is reset between jobs.

### References

- Laravel Service Container: Scoped Bindings - https://laravel.com/docs/12.x/container#scoped-bindings
- Laravel Octane: Scoped Bindings - https://laravel.com/docs/12.x/octane#scoped-bindings
- Laravel API: Container::scoped - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html#method_scoped


## Summary Table of Service Container Features

| Feature | Method | Lifetime | Use Case |
|---------|--------|----------|----------|
| Binding | `bind()` | New instance per resolution | Interface-to-concrete mapping |
| Singleton | `singleton()` | One instance per application | Stateful services, connections |
| Contextual Binding | `when()->needs()->give()` | Per-consumer | Different implementations per class |
| Automatic Resolution | (none) | New instance per resolution | Concrete classes with type-hints |
| Tagging | `tag()` / `tagged()` | Per resolution | Plugin architectures, collections |
| Extending | `extend()` | Post-resolution | Decorators, configuration |
| Scoped | `scoped()` | Per request/job | Octane, request context |


## References

- Laravel Service Container Documentation (12.x) - https://laravel.com/docs/12.x/container
- Laravel Service Container: Binding - https://laravel.com/docs/12.x/container#binding
- Laravel Service Container: Binding Singletons - https://laravel.com/docs/12.x/container#binding-singletons
- Laravel Service Container: Contextual Binding - https://laravel.com/docs/12.x/container#contextual-binding
- Laravel Service Container: Tagging - https://laravel.com/docs/12.x/container#tagging
- Laravel Service Container: Extending Bindings - https://laravel.com/docs/12.x/container#extending-bindings
- Laravel Service Container: Scoped Bindings - https://laravel.com/docs/12.x/container#scoped-bindings
- Laravel Service Container: Resolving - https://laravel.com/docs/12.x/container#resolving
- Laravel API: Container - https://api.laravel.com/docs/11.x/Illuminate/Container/Container.html
- Laravel API: Application Contract - https://api.laravel.com/docs/11.x/Illuminate/Contracts/Foundation/Application.html
- Laravel Octane Documentation - https://laravel.com/docs/12.x/octane
- Laravel Octane: Scoped Bindings - https://laravel.com/docs/12.x/octane#scoped-bindings
- Laravel News: Advanced Application Architecture through Laravel's Service Container Management - https://laravel-news.com/advanced-service-container
- PHP Reflection API - https://www.php.net/manual/en/book.reflection.php
- Laravel: Service Container (5.4) - https://laravel.com/docs/5.4/container