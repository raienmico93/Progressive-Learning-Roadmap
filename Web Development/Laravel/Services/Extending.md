# Laravel Extending: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Extending Laravel is the practice of adding new functionality, customizing existing behavior, or integrating third-party services into a Laravel application through well-defined extension points such as service providers, manager classes, and package auto-discovery.

**Technical Definition:** Laravel provides a layered extension architecture built on the Service Container and Service Provider system. Extensions can bind custom services into the container via `register()`, configure application behavior via `boot()`, register custom drivers into Manager classes (Cache, Auth, Storage, Broadcast) using the `extend()` method, and distribute reusable functionality as Composer packages that leverage auto-discovery via `composer.json` metadata. The framework's Manager pattern—implemented by `CacheManager`, `SessionManager`, `AuthManager`, `FilesystemManager`, and `BroadcastManager`—exposes an `extend(string $driver, Closure $callback)` method for injecting custom driver resolution logic.

**Beginner-Friendly Explanation:** Laravel is like a house with standard electrical outlets, plumbing, and fixtures. Extending Laravel is like adding new rooms, upgrading appliances, or connecting to a new utility provider. You don't tear down the house; you use the standard connections (service providers, manager classes) to add what you need. Whether you're adding a custom payment gateway, wrapping an external API, or creating a reusable package for the community, Laravel provides clear extension points for doing it cleanly.

### Key Characteristics

1. **Service Provider Centric:** Service providers are the primary extension mechanism, responsible for binding services and bootstrapping functionality.
2. **Manager Pattern:** Driver-based subsystems use Manager classes with an `extend()` method for registering custom drivers.
3. **Package Auto-Discovery:** Laravel automatically registers service providers and facades listed in a package's `composer.json` under `extra.laravel`.
4. **Asset Publishing:** Packages can publish configuration, views, translations, migrations, and assets to the host application via the `publishes()` method.
5. **Container Bindings:** Custom services and helper singletons are registered via `bind()` and `singleton()` in service providers.
6. **Deferred Loading:** Heavy providers can implement `DeferrableProvider` to load lazily.
7. **Domain Separation:** Custom providers can be organized by domain (API, console, payments) for clarity.
8. **Framework Extension Points:** Auth, Cache, Storage, Session, and Broadcast all support custom driver injection.

### Prerequisites

- Laravel 10.x or higher (12.x recommended)
- PHP 8.1 or higher
- Composer package manager
- Familiarity with the Service Container and Service Providers
- Understanding of the Manager pattern and driver-based architecture
- Basic knowledge of Composer package structure
- A working Laravel application with `bootstrap/providers.php`

### Related Programming Areas

- **Service Container:** The DI container where custom services are bound.
- **Service Providers:** The registration point for extensions.
- **Package Development:** Distributing extensions as Composer packages.
- **Manager Pattern:** Laravel's driver-based architecture for Cache, Auth, Storage, etc.
- **Facades:** Static proxies for container-managed services.
- **Composer:** PHP dependency manager with auto-discovery support.
- **Artisan Commands:** Console utilities registered via providers.

### Core Concepts / Features

1. Custom Services (Building customized helper singletons and isolated business abstractions)
2. Package Integration (Automating service setups via package auto-discovery pipelines)
3. Custom Providers (Creating distinct domain providers for API, console, and payment processing)
4. Framework Extension Points (Using standard managers to introduce unique drivers)
5. Enhanced: Custom Driver Injection (`Auth::extend()`, `Cache::extend()`, `Storage::extend()`)
6. Enhanced: Publishing Assets (Configuring the `publishes()` pipeline for views, language files, migrations)


## 1. Custom Services (Building Customized Helper Singletons and Isolated Business Abstractions)

### Definitions

**Core Definition:** Custom services are user-defined classes registered in the service container as singletons or regular bindings, encapsulating reusable business logic, helper methods, or abstractions that are shared across the application.

**Technical Definition:** A custom service is any class bound into the service container via `$this->app->singleton()` or `$this->app->bind()` in a service provider's `register()` method. Singletons ensure a single shared instance per application lifecycle, ideal for services that hold state, manage connections, or provide expensive-to-construct functionality. The service container resolves these services automatically when type-hinted in constructors, enabling dependency injection. The `$singletons` array property on a Service Provider also accepts array-based singleton registration.

**Beginner-Friendly Explanation:** A custom service is like a specialized tool you build for your workshop. Instead of rebuilding it every time you need it, you store it in a shared toolbox (the container) and grab the same one whenever you need it. Helper singletons are particularly useful for things like API clients, configuration managers, or analytics trackers—you want one shared instance, not a new one every time.

### Purposes

- To encapsulate reusable business logic in a dedicated, testable class.
- To share a single instance of a service across the application via singleton binding.
- To abstract complex operations behind a simple, injectable API.
- To provide helper methods that are accessible throughout the application.
- To manage stateful services (e.g., connection pools, counters) consistently.
- To reduce code duplication by centralizing shared functionality.
- To enable dependency injection for services that require configuration or other dependencies.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// 1. Define the custom service

namespace App\Services;

use App\Contracts\Logger;

class AnalyticsService
{
    protected array $events = [];

    public function __construct(
        protected Logger $logger
    ) {}

    public function track(string $event, array $data = []): void
    {
        $this->events[] = ['event' => $event, 'data' => $data, 'at' => now()];
        $this->logger->log("Tracked: {$event}");
    }

    public function getEvents(): array
    {
        return $this->events;
    }
}
```

```php
<?php
// 2. Register the singleton in a service provider

namespace App\Providers;

use App\Services\AnalyticsService;
use App\Services\FileLogger;
use App\Contracts\Logger;
use Illuminate\Support\ServiceProvider;

class AnalyticsServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind the logger dependency
        $this->app->singleton(Logger::class, FileLogger::class);

        // Register the analytics service as a singleton
        $this->app->singleton(AnalyticsService::class, function ($app) {
            return new AnalyticsService(
                $app->make(Logger::class)
            );
        });
    }
}
```

```php
<?php
// 3. Use the service via dependency injection

namespace App\Http\Controllers;

use App\Services\AnalyticsService;

class DashboardController extends Controller
{
    public function __construct(
        protected AnalyticsService $analytics
    ) {}

    public function index()
    {
        $this->analytics->track('dashboard_view');
        return response()->json($this->analytics->getEvents());
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `AnalyticsService` | The custom service class |
| `__construct(Logger $logger)` | Dependency injection via constructor |
| `singleton(AnalyticsService::class, ...)` | Registers a shared instance |
| `$app->make(Logger::class)` | Resolves the logger dependency |
| `$this->analytics->track()` | Uses the injected service |

#### Syntax Rules

1. Custom services **must** be registered in a service provider's `register()` method.
2. Singletons **must** be registered with `singleton()`, not `bind()`.
3. The `$singletons` array property can be used for simple singleton registration.
4. The service's dependencies **must** be resolvable from the container.
5. Closures receive the container instance as their only argument.
6. Services **should** be type-hinted in constructors for automatic injection.
7. Services **should not** be resolved in `register()`; only bind them.

#### Constraints and Limitations

- **Singleton State Leakage:** In Octane, singletons persist across requests; use `scoped()` for request-specific state.
- **No Resolution in Register:** Resolving services in `register()` can cause errors if the dependency's provider hasn't been registered.
- **Binding Order:** Bindings that depend on other bindings must be registered after those dependencies.
- **Testing:** Singletons can retain state between tests; use `$this->app->forgetInstance()` in test teardown.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Settings Helper Singleton

**Step-by-Step Setup Guide:**

1. Create a `SettingsHelper` class.
2. Register it as a singleton in `AppServiceProvider`.
3. Inject it into a controller or another service.
4. Verify the same instance is used.

**Complete Executable Code:**

```php
<?php
// app/Helpers/SettingsHelper.php

namespace App\Helpers;

class SettingsHelper
{
    protected array $settings = [];

    public function __construct()
    {
        $this->settings = [
            'app_name' => config('app.name'),
            'timezone' => config('app.timezone'),
        ];
    }

    public function get(string $key, mixed $default = null): mixed
    {
        return $this->settings[$key] ?? $default;
    }

    public function set(string $key, mixed $value): void
    {
        $this->settings[$key] = $value;
    }

    public function all(): array
    {
        return $this->settings;
    }
}
```

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Helpers\SettingsHelper;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Register as a singleton — same instance everywhere
        $this->app->singleton(SettingsHelper::class, function ($app) {
            return new SettingsHelper();
        });
    }
}
```

```php
<?php
// app/Http/Controllers/SettingsController.php

namespace App\Http\Controllers;

use App\Helpers\SettingsHelper;

class SettingsController extends Controller
{
    public function __construct(
        protected SettingsHelper $settings
    ) {}

    public function update()
    {
        // This modifies the shared singleton instance
        $this->settings->set('theme', 'dark');

        return response()->json($this->settings->all());
    }
}
```

**Expected Output:**

- The `SettingsHelper` singleton is shared across the request.
- Modifying it in one controller action affects the same instance elsewhere.

**Why This Code Produces That Result:**

- `singleton()` caches the instance on first resolution.
- All subsequent resolutions return the same object.
- State (the `$settings` array) is preserved across the request.

#### Example 2: Custom Service with Interface Binding

```php
<?php
// app/Contracts/ExchangeRateService.php

namespace App\Contracts;

interface ExchangeRateService
{
    public function rate(string $from, string $to): float;
}
```

```php
<?php
// app/Services/LiveExchangeRateService.php

namespace App\Services;

use App\Contracts\ExchangeRateService;
use Illuminate\Support\Facades\Http;

class LiveExchangeRateService implements ExchangeRateService
{
    public function __construct(
        protected string $apiKey
    ) {}

    public function rate(string $from, string $to): float
    {
        $response = Http::get("https://api.exchangerate.host/latest", [
            'base' => $from,
            'symbols' => $to,
            'access_key' => $this->apiKey,
        ]);

        return $response->json("rates.{$to}", 1.0);
    }
}
```

```php
<?php
// app/Providers/ExchangeServiceProvider.php

namespace App\Providers;

use App\Contracts\ExchangeRateService;
use App\Services\LiveExchangeRateService;
use Illuminate\Support\ServiceProvider;

class ExchangeServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(ExchangeRateService::class, function ($app) {
            return new LiveExchangeRateService(
                config('services.exchange.api_key')
            );
        });
    }
}
```

```php
<?php
// Usage in a controller

namespace App\Http\Controllers;

use App\Contracts\ExchangeRateService;

class CheckoutController extends Controller
{
    public function __construct(
        protected ExchangeRateService $exchange
    ) {}

    public function convert()
    {
        $rate = $this->exchange->rate('USD', 'EUR');
        return response()->json(['rate' => $rate]);
    }
}
```

**Expected Output:**

- The `LiveExchangeRateService` singleton is injected into the controller.
- The API key is read from configuration at construction time.
- Swapping to a `FakeExchangeRateService` in tests requires only changing the binding.

**Why This Code Produces That Result:**

- The interface is bound to a concrete implementation as a singleton.
- The concrete implementation receives its configuration via the closure.
- The controller depends on the interface, not the concrete class.

### Real-World Cases

**Case 1: API Client Wrapper**

A `StripeService` singleton wraps the Stripe SDK, sharing the HTTP client and API key across the application.

**Case 2: Feature Flag Manager**

A `FeatureFlagService` singleton reads feature flags from a database or config and caches them for the request.

**Case 3: Audit Logger**

An `AuditLogger` singleton collects audit events during the request and flushes them at the end.

### References

- Laravel Service Container: Binding Singletons - https://laravel.com/docs/12.x/container#binding-singletons
- Laravel Service Providers: The Register Method - https://laravel.com/docs/12.x/providers#the-register-method
- Stack Overflow: Laravel load and initialize custom helper class - https://stackoverflow.com/questions/46856505/laravel-load-and-initialize-custom-helper-class
- Laravel News: Advanced Application Architecture through Laravel's Service Container Management - https://laravel-news.com/service-container-management


## 2. Package Integration (Automating Service Setups via Package Auto-Discovery Pipelines Using `composer.json` Metadata Configurations)

### Definitions

**Core Definition:** Package auto-discovery is a Laravel feature that automatically registers a package's service providers and facades when the package is installed, eliminating the need for users to manually add them to `bootstrap/providers.php` or `config/app.php`.

**Technical Definition:** A Laravel package declares its service providers and aliases in the `extra.laravel` section of its `composer.json` file. During application bootstrap, Laravel's `PackageManifest` reads this metadata and registers the listed providers and facades automatically. The `extra.laravel.providers` array lists service provider class names, and `extra.laravel.aliases` maps facade aliases to class names. Consumers can opt out of auto-discovery by listing the package name in their application's `extra.laravel.dont-discover` array.

**Beginner-Friendly Explanation:** Imagine you buy a new appliance (a package) for your home (Laravel application). Instead of having to manually wire it into your electrical panel, the appliance comes with a note saying "I need these connections." Laravel reads that note and wires everything up automatically. You just install the package and it works—no manual configuration needed.

### Purposes

- To eliminate manual provider registration for package consumers.
- To provide a seamless installation experience for Laravel packages.
- To automatically register package facades and aliases.
- To allow consumers to opt out of auto-discovery when needed.
- To standardize package metadata across the Laravel ecosystem.
- To reduce installation friction and configuration errors.
- To enable zero-configuration package installation.

### Syntax Rules and Structure

#### Complete General Syntax

```json
{
    "name": "vendor/package-name",
    "description": "A Laravel package",
    "type": "library",
    "require": {
        "php": "^8.1",
        "illuminate/support": "^11.0"
    },
    "autoload": {
        "psr-4": {
            "Vendor\\PackageName\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "Vendor\\PackageName\\PackageServiceProvider"
            ],
            "aliases": {
                "PackageName": "Vendor\\PackageName\\Facades\\PackageName"
            }
        }
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `extra.laravel.providers` | Array of service provider class names |
| `extra.laravel.aliases` | Map of facade aliases to class names |
| `autoload.psr-4` | Autoloading namespace mapping |
| `dont-discover` (consumer) | Opts out of auto-discovery |

#### Syntax Rules

1. The `extra.laravel.providers` array **must** contain fully qualified class names.
2. The `extra.laravel.aliases` object **must** map alias strings to fully qualified class names.
3. The package **must** declare an autoload namespace for its classes.
4. Auto-discovery works automatically when the package is installed via Composer.
5. Consumers can disable auto-discovery per package via `extra.laravel.dont-discover` in their application's `composer.json`.
6. The `*` wildcard in `dont-discover` disables auto-discovery for all packages.
7. Auto-discovery requires `composer dump-autoload` to regenerate the manifest after changes.

#### Constraints and Limitations

- **Manifest Caching:** The package manifest is cached; changes to `extra.laravel` require `php artisan package:discover` or `composer dump-autoload`.
- **No Boot Method in Deferred:** Providers with a `boot()` method cannot be deferred; auto-discovery registers them normally.
- **Conflicting Providers:** Multiple packages registering the same facade alias can cause conflicts.
- **Debugging:** Auto-discovered providers are harder to trace; use `php artisan about` to list registered providers.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Package Auto-Discovery Setup

**Step-by-Step Setup Guide:**

1. Create a package directory with a `composer.json`.
2. Define the service provider in `extra.laravel.providers`.
3. Define facades in `extra.laravel.aliases`.
4. Install the package in a Laravel application.
5. Verify the provider is auto-registered.

**Complete Executable Code:**

```json
{
    "name": "acme/analytics",
    "description": "Analytics package for Laravel",
    "type": "library",
    "require": {
        "php": "^8.1",
        "illuminate/support": "^11.0"
    },
    "autoload": {
        "psr-4": {
            "Acme\\Analytics\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "Acme\\Analytics\\AnalyticsServiceProvider"
            ],
            "aliases": {
                "Analytics": "Acme\\Analytics\\Facades\\Analytics"
            }
        }
    }
}
```

```php
<?php
// src/AnalyticsServiceProvider.php

namespace Acme\Analytics;

use Illuminate\Support\ServiceProvider;

class AnalyticsServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('analytics', function ($app) {
            return new Analytics(config('analytics.api_key'));
        });
    }

    public function boot(): void
    {
        // Publish configuration
        $this->publishes([
            __DIR__.'/../config/analytics.php' => config_path('analytics.php'),
        ], 'analytics-config');

        // Merge configuration
        $this->mergeConfigFrom(
            __DIR__.'/../config/analytics.php', 'analytics'
        );
    }
}
```

```php
<?php
// src/Facades/Analytics.php

namespace Acme\Analytics\Facades;

use Illuminate\Support\Facades\Facade;

class Analytics extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'analytics';
    }
}
```

```bash
# Install the package in a Laravel application
composer require acme/analytics

# Verify auto-discovery
php artisan about | grep "Acme\\Analytics"
```

**Expected Output:**

- The `AnalyticsServiceProvider` is automatically registered.
- The `Analytics` facade is available without manual registration.
- The package's configuration can be published via `php artisan vendor:publish --tag=analytics-config`.

**Why This Code Produces That Result:**

- `extra.laravel.providers` tells Laravel to auto-register the provider.
- `extra.laravel.aliases` registers the facade alias.
- `PackageManifest` reads the metadata and registers everything automatically.

#### Example 2: Opting Out of Auto-Discovery

```json
{
    "name": "acme/my-app",
    "extra": {
        "laravel": {
            "dont-discover": [
                "acme/analytics"
            ]
        }
    }
}
```

**Expected Output:**

- The `acme/analytics` package's provider is **not** auto-registered.
- The consumer must manually register it in `bootstrap/providers.php`.

**Why This Code Produces That Result:**

- `dont-discover` tells Laravel's `PackageManifest` to skip the specified package.
- This is useful when the consumer wants to control registration order or configuration.

### Real-World Cases

**Case 1: Laravel Debugbar**

`barryvdh/laravel-debugbar` uses auto-discovery to register its `ServiceProvider` and `Debugbar` facade automatically.

**Case 2: Spatie Packages**

Spatie's packages (e.g., `spatie/laravel-permission`) use auto-discovery for seamless installation.

**Case 3: Custom Internal Packages**

A company's internal packages use auto-discovery to ensure consistent installation across all projects.

### References

- Laravel Package Development: Package Discovery - https://laravel.com/docs/12.x/packages#package-discovery
- Laravel Package Development: Opting Out of Package Discovery - https://laravel.com/docs/12.x/packages#opting-out-of-package-discovery
- Laravel API: PackageManifest - https://api.laravel.com/docs/11.x/Illuminate/Foundation/PackageManifest.html
- GitHub: Package Development Guide - https://github.com/sgflores/optioner/blob/main/PACKAGE-DEVELOPMENT-GUIDE.md


## 3. Custom Providers (Creating Distinct Domain Providers to Separate API Connections, Console Utilities, and Payment Processing Engines)

### Definitions

**Core Definition:** Custom providers are user-defined service provider classes organized by domain or functionality, each responsible for registering and bootstrapping a specific area of the application (e.g., API integrations, console commands, payment processing).

**Technical Definition:** A custom provider extends `Illuminate\Support\ServiceProvider` and is registered in `bootstrap/providers.php`. Unlike a monolithic `AppServiceProvider`, domain-specific providers separate concerns: a `PaymentServiceProvider` handles payment gateway bindings, a `ConsoleServiceProvider` registers Artisan commands, and an `ApiServiceProvider` configures external API clients. This follows the Single Responsibility Principle and makes the application easier to maintain, test, and extend. Each provider can have its own `register()` and `boot()` methods.

**Beginner-Friendly Explanation:** Imagine a large office building. Instead of having one person handle everything—mail, phones, IT, and cleaning—you have specialized departments: the mailroom handles mail, IT handles computers, and facilities handles maintenance. Custom providers are like those departments: each one is responsible for a specific area of the application, making the whole organization easier to manage.

### Purposes

- To separate application concerns into focused, maintainable providers.
- To make the codebase easier to navigate and reason about.
- To enable independent testing of each domain.
- To facilitate team collaboration by assigning providers to specific teams.
- To control the order of service registration and bootstrapping.
- To package domain-specific functionality for reuse.
- To reduce the size and complexity of `AppServiceProvider`.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// app/Providers/PaymentServiceProvider.php

namespace App\Providers;

use App\Contracts\PaymentGateway;
use App\Services\StripeGateway;
use App\Services\PayPalGateway;
use Illuminate\Support\ServiceProvider;

class PaymentServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(PaymentGateway::class, function ($app) {
            return match (config('services.payment.driver')) {
                'paypal' => new PayPalGateway(
                    config('services.paypal.client_id'),
                    config('services.paypal.secret')
                ),
                default => new StripeGateway(
                    config('services.stripe.key'),
                    config('services.stripe.secret')
                ),
            };
        });
    }

    public function boot(): void
    {
        // Register payment-related event listeners
        // Register payment-related view composers
    }
}
```

```php
<?php
// app/Providers/ConsoleServiceProvider.php

namespace App\Providers;

use App\Console\Commands\SyncOrders;
use App\Console\Commands\GenerateReports;
use Illuminate\Support\ServiceProvider;

class ConsoleServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->commands([
            SyncOrders::class,
            GenerateReports::class,
        ]);
    }
}
```

```php
<?php
// app/Providers/ApiServiceProvider.php

namespace App\Providers;

use App\Services\ApiClient;
use Illuminate\Support\ServiceProvider;

class ApiServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(ApiClient::class, function ($app) {
            return new ApiClient(
                config('services.api.base_url'),
                config('services.api.token')
            );
        });
    }
}
```

```php
// bootstrap/providers.php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ApiServiceProvider::class,
    App\Providers\ConsoleServiceProvider::class,
    App\Providers\PaymentServiceProvider::class,
];
```

**Component Breakdown:**

| Provider | Domain | Responsibility |
|----------|--------|----------------|
| `ApiServiceProvider` | External APIs | Binds API clients |
| `ConsoleServiceProvider` | CLI | Registers Artisan commands |
| `PaymentServiceProvider` | Payments | Binds payment gateways |
| `AppServiceProvider` | General | Miscellaneous bindings |

#### Syntax Rules

1. Each domain provider **must** extend `Illuminate\Support\ServiceProvider`.
2. Domain providers **must** be registered in `bootstrap/providers.php`.
3. The `register()` method **must** only bind services into the container.
4. The `boot()` method **may** register event listeners, routes, commands, and view composers.
5. Provider order in `bootstrap/providers.php` determines registration and boot order.
6. Providers that depend on other providers' bindings **must** be ordered after them.
7. Domain providers **should** be named descriptively (e.g., `PaymentServiceProvider`).

#### Constraints and Limitations

- **Order Dependency:** Providers boot in registration order; dependencies must be ordered correctly.
- **No Automatic Resolution:** Laravel does not automatically detect inter-provider dependencies.
- **Boot Performance:** Each provider adds boot overhead; defer where possible.
- **Over-Organization:** Too many providers can make the application harder to trace.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Payment Domain Provider

**Step-by-Step Setup Guide:**

1. Create a `PaymentServiceProvider` in `app/Providers`.
2. Register the payment gateway binding in `register()`.
3. Register the provider in `bootstrap/providers.php`.
4. Inject the payment gateway into a controller.

**Complete Executable Code:**

```php
<?php
// app/Contracts/PaymentGateway.php

namespace App\Contracts;

interface PaymentGateway
{
    public function charge(float $amount): array;
    public function refund(string $chargeId): array;
}
```

```php
<?php
// app/Services/StripeGateway.php

namespace App\Services;

use App\Contracts\PaymentGateway;

class StripeGateway implements PaymentGateway
{
    public function __construct(
        protected string $key,
        protected string $secret
    ) {}

    public function charge(float $amount): array
    {
        return ['gateway' => 'stripe', 'status' => 'succeeded', 'amount' => $amount];
    }

    public function refund(string $chargeId): array
    {
        return ['gateway' => 'stripe', 'status' => 'refunded'];
    }
}
```

```php
<?php
// app/Providers/PaymentServiceProvider.php

namespace App\Providers;

use App\Contracts\PaymentGateway;
use App\Services\StripeGateway;
use App\Services\PayPalGateway;
use Illuminate\Support\ServiceProvider;

class PaymentServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(PaymentGateway::class, function ($app) {
            return match (config('services.payment.driver', 'stripe')) {
                'paypal' => new PayPalGateway(
                    config('services.paypal.client_id'),
                    config('services.paypal.secret')
                ),
                default => new StripeGateway(
                    config('services.stripe.key'),
                    config('services.stripe.secret')
                ),
            };
        });
    }

    public function boot(): void
    {
        // Register payment webhook routes
        // Register payment event listeners
    }
}
```

```php
// bootstrap/providers.php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\PaymentServiceProvider::class,
];
```

```php
<?php
// app/Http/Controllers/CheckoutController.php

namespace App\Http\Controllers;

use App\Contracts\PaymentGateway;
use Illuminate\Http\Request;

class CheckoutController extends Controller
{
    public function __construct(
        protected PaymentGateway $gateway
    ) {}

    public function process(Request $request)
    {
        return response()->json(
            $this->gateway->charge($request->input('amount'))
        );
    }
}
```

**Expected Output:**

- The `PaymentServiceProvider` registers the `PaymentGateway` binding.
- The controller receives the configured gateway (Stripe or PayPal).
- Changing `services.payment.driver` changes the implementation without code changes.

**Why This Code Produces That Result:**

- The provider isolates payment-related bindings.
- The closure selects the implementation based on configuration.
- The controller depends on the interface, not the concrete class.

### Real-World Cases

**Case 1: Multi-Domain SaaS**

A SaaS platform has separate providers for Billing, Shipping, Inventory, and Reporting, each managed by a different team.

**Case 2: Modular Package Integration**

Each package installed in the application has its own provider, and the application's providers handle application-specific configuration.

**Case 3: Console Utilities**

A `ConsoleServiceProvider` registers all Artisan commands, keeping them separate from HTTP-related bindings.

### References

- Laravel Service Providers Documentation - https://laravel.com/docs/12.x/providers
- Laravel Service Providers: The Register Method - https://laravel.com/docs/12.x/providers#the-register-method
- Laravel Service Providers: The Boot Method - https://laravel.com/docs/12.x/providers#the-boot-method
- Laravel API: ServiceProvider - https://api.laravel.com/docs/11.x/Illuminate/Support/ServiceProvider.html


## 4. Framework Extension Points (Using Standard Managers to Introduce Unique Drivers into Built-in Systems)

### Definitions

**Core Definition:** Framework extension points are the standardized mechanisms Laravel provides for injecting custom drivers into its driver-based subsystems—Cache, Session, Authentication, Queue, and Broadcasting—through Manager classes that expose an `extend()` method.

**Technical Definition:** Laravel's Manager classes (`CacheManager`, `SessionManager`, `AuthManager`, `BroadcastManager`) extend `Illuminate\Support\Manager` and implement the Factory design pattern. Each manager exposes `extend(string $driver, Closure $callback)`, which registers a custom driver creator. The closure receives the application container and the driver's configuration array, and must return an instance implementing the subsystem's contract (e.g., `Illuminate\Contracts\Cache\Store` for cache drivers). The custom driver is then selectable via the subsystem's configuration file.

**Beginner-Friendly Explanation:** Laravel's driver-based systems are like universal power adapters. The `extend()` method is a blank slot where you can plug in your own adapter. For example, if you want to use a custom cache backend (like MongoDB), you plug in a MongoDB adapter using `Cache::extend('mongo', ...)`. Once plugged in, you can switch to it in your configuration just like the built-in drivers.

### Purposes

- To add support for a service not natively supported by Laravel.
- To swap the underlying implementation of a framework subsystem.
- To integrate with proprietary or internal infrastructure.
- To follow the Open/Closed Principle (extend without modifying framework code).
- To allow per-environment driver selection.
- To provide a consistent API across different driver implementations.
- To enable testing with custom test drivers.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// 1. Create a custom cache store implementing the contract

namespace App\Cache;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) { /* ... */ }
    public function put($key, $value, $seconds) { /* ... */ }
    public function increment($key, $value = 1) { /* ... */ }
    public function decrement($key, $value = 1) { /* ... */ }
    public function forever($key, $value) { /* ... */ }
    public function forget($key) { /* ... */ }
    public function flush() { /* ... */ }
    public function getPrefix() { /* ... */ }
}
```

```php
<?php
// 2. Register the custom driver in a service provider

namespace App\Providers;

use App\Cache\MongoStore;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Cache::extend('mongo', function ($app, $config) {
            return Cache::repository(new MongoStore($config));
        });
    }
}
```

```php
// 3. Configure the custom driver in config/cache.php

'stores' => [
    'mongo' => [
        'driver' => 'mongo',
        'connection' => 'mongodb://localhost:27017',
    ],
],
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `MongoStore` | Implements `Illuminate\Contracts\Cache\Store` |
| `Cache::extend('mongo', ...)` | Registers the custom driver |
| Closure `$app, $config` | Receives container and driver configuration |
| `Cache::repository()` | Wraps the store in a repository |
| `config/cache.php` | Selects the custom driver |

#### Syntax Rules

1. The custom driver **must** implement the subsystem's contract.
2. `extend()` **must** be called on the Manager (e.g., `Cache::extend()`).
3. The registration closure receives `$app` and `$config` (the driver configuration).
4. The closure **must** return an instance of the contract.
5. The driver **must** be configured in the subsystem's config file.
6. The active driver is selected via the subsystem's environment variable.
7. Registration **should** occur in a service provider's `boot()` method.

#### Constraints and Limitations

- **Contract Compliance:** The custom driver must fully implement the contract; missing methods cause runtime errors.
- **Configuration Caching:** After changing config, run `php artisan config:clear`.
- **Testing:** Custom drivers must be tested independently.
- **Version Compatibility:** Contract signatures may change between Laravel versions.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Custom Cache Driver (MongoDB)

```php
<?php
// app/Cache/MongoStore.php

namespace App\Cache;

use Illuminate\Contracts\Cache\Store;
use MongoDB\Client;

class MongoStore implements Store
{
    protected $collection;

    public function __construct(array $config)
    {
        $client = new Client($config['connection']);
        $this->collection = $client->selectDatabase('cache')->selectCollection('store');
    }

    public function get($key)
    {
        $doc = $this->collection->findOne(['key' => $key]);
        return $doc ? unserialize($doc['value']) : null;
    }

    public function put($key, $value, $seconds)
    {
        $this->collection->updateOne(
            ['key' => $key],
            ['$set' => [
                'value' => serialize($value),
                'expires_at' => now()->addSeconds($seconds),
            ]],
            ['upsert' => true]
        );
        return true;
    }

    // ... other Store methods
}
```

```php
<?php
// app/Providers/CacheServiceProvider.php

namespace App\Providers;

use App\Cache\MongoStore;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Cache::extend('mongo', function ($app, $config) {
            return Cache::repository(new MongoStore($config));
        });
    }
}
```

```php
// config/cache.php

'stores' => [
    'mongo' => [
        'driver' => 'mongo',
        'connection' => env('MONGODB_URI', 'mongodb://localhost:27017'),
    ],
],
```

**Expected Output:**

- The `mongo` cache driver is registered and available via `CACHE_STORE=mongo`.
- Cache operations use MongoDB instead of the default driver.

**Why This Code Produces That Result:**

- `Cache::extend()` registers the `mongo` driver.
- The closure returns a `Repository` wrapping the `MongoStore`.
- The manager selects the driver based on the config.

### Real-World Cases

**Case 1: Custom Session Driver**

A team using Redis Cluster registers a custom session driver that handles cluster-aware session storage.

**Case 2: Custom Broadcast Driver**

A company with an internal WebSocket server registers a custom broadcast driver to push events without using Pusher or Reverb.

**Case 3: Custom Queue Driver**

A team using a proprietary message queue registers a custom queue driver to integrate with their existing infrastructure.

### References

- Laravel Extending: Managers & Factories - https://laravel.com/docs/5.0/extending#managers-and-factories
- Laravel API: Manager - https://api.laravel.com/docs/11.x/Illuminate/Support/Manager.html
- Laravel API: CacheManager - https://api.laravel.com/docs/11.x/Illuminate/Cache/CacheManager.html
- Stack Overflow: How to register custom broadcaster - https://stackoverflow.com/questions/32887473/laravel-how-to-register-custom-broadcaster


## 5. Enhanced: Custom Driver Injection (`Auth::extend()`, `Cache::extend()`, `Storage::extend()`)

### Definitions

**Core Definition:** Custom driver injection is the practice of using a subsystem manager's `extend()` method—specifically `Auth::extend()`, `Cache::extend()`, and `Storage::extend()`—to register custom driver implementations that can be selected via configuration.

**Technical Definition:** `Auth::extend(string $driver, Closure $callback)` registers a custom authentication guard creator. The closure receives the application container, the guard name, and the guard configuration array, and must return an instance of `Illuminate\Contracts\Auth\Guard`. `Cache::extend()` registers a custom cache store creator returning an `Illuminate\Contracts\Cache\Store` (wrapped in a repository). `Storage::extend()` registers a custom filesystem driver creator returning an `Illuminate\Contracts\Filesystem\Filesystem` (or a Flysystem adapter). Each extension is registered in a service provider's `boot()` method.

**Beginner-Friendly Explanation:** Think of Laravel's subsystems as different types of vehicles—Auth is a car, Cache is a truck, Storage is a van. Each has standard engines (drivers). `Auth::extend()`, `Cache::extend()`, and `Storage::extend()` let you swap in a custom engine. You tell Laravel "when I select the 'jwt' engine for Auth, use my JWT guard." Once registered, you can select it in your configuration just like a built-in engine.

### Purposes

- To add custom authentication guards (e.g., JWT, API key).
- To add custom cache stores (e.g., MongoDB, DynamoDB).
- To add custom filesystem drivers (e.g., Dropbox, Minio).
- To integrate with proprietary or internal infrastructure.
- To enable per-environment driver selection.
- To provide a consistent API across different driver implementations.
- To test with custom test drivers.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Auth::extend() — custom authentication guard

namespace App\Providers;

use App\Auth\JwtGuard;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Auth::extend('jwt', function ($app, $name, array $config) {
            return new JwtGuard(
                Auth::createUserProvider($config['provider']),
                $app->make('request')
            );
        });
    }
}
```

```php
<?php
// Cache::extend() — custom cache store

namespace App\Providers;

use App\Cache\MongoStore;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Cache::extend('mongo', function ($app, $config) {
            return Cache::repository(new MongoStore($config));
        });
    }
}
```

```php
<?php
// Storage::extend() — custom filesystem driver

namespace App\Providers;

use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use League\Flysystem\Adapter\Local;

class StorageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Storage::extend('dropbox', function ($app, $config) {
            $client = new \Dropbox\Client($config['authorization_token']);
            $adapter = new \League\Flysystem\Dropbox\DropboxAdapter($client);
            return new Filesystem($adapter);
        });
    }
}
```

**Component Breakdown:**

| Method | Closure Signature | Returns | Config File |
|--------|------------------|---------|-------------|
| `Auth::extend()` | `($app, $name, $config)` | `Guard` | `config/auth.php` |
| `Cache::extend()` | `($app, $config)` | `Repository` | `config/cache.php` |
| `Storage::extend()` | `($app, $config)` | `Filesystem` | `config/filesystems.php` |

#### Syntax Rules

1. `extend()` **must** be called in a service provider's `boot()` method.
2. The closure **must** return an instance of the subsystem's contract.
3. The driver name **must** match the `driver` key in the config file.
4. For `Auth::extend()`, the closure receives the guard name and config.
5. For `Cache::extend()`, the closure receives the cache configuration.
6. For `Storage::extend()`, the closure receives the disk configuration.
7. The custom driver is selected via the subsystem's config file or environment variable.

#### Constraints and Limitations

- **Contract Compliance:** The custom driver must fully implement the contract; missing methods cause runtime errors.
- **Configuration Caching:** After changing config, run `php artisan config:clear`.
- **Testing:** Custom drivers must be tested independently.
- **Version Compatibility:** Contract signatures may change between Laravel versions.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Custom JWT Authentication Guard

```php
<?php
// app/Auth/JwtGuard.php

namespace App\Auth;

use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Auth\UserProvider;
use Illuminate\Http\Request;

class JwtGuard implements Guard
{
    protected ?\App\Models\User $user = null;

    public function __construct(
        protected UserProvider $provider,
        protected Request $request
    ) {}

    public function check(): bool
    {
        return $this->user() !== null;
    }

    public function user(): ?\App\Models\User
    {
        if ($this->user !== null) {
            return $this->user;
        }

        $token = $this->request->bearerToken();
        if (!$token) {
            return null;
        }

        $payload = \Firebase\JWT\JWT::decode($token, new \Firebase\JWT\Key(config('auth.jwt_secret'), 'HS256'));
        $this->user = $this->provider->retrieveById($payload->sub);

        return $this->user;
    }

    // ... other Guard methods
}
```

```php
<?php
// app/Providers/AuthServiceProvider.php

namespace App\Providers;

use App\Auth\JwtGuard;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Auth::extend('jwt', function ($app, $name, array $config) {
            return new JwtGuard(
                Auth::createUserProvider($config['provider']),
                $app->make('request')
            );
        });
    }
}
```

```php
// config/auth.php

'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

**Expected Output:**

- The `jwt` guard is registered and available via `auth('api')`.
- API requests with a valid JWT are authenticated.

**Why This Code Produces That Result:**

- `Auth::extend()` registers the `jwt` driver.
- The closure returns a `JwtGuard` instance.
- The guard is selected via `config/auth.php`.

#### Example 2: Custom Storage Driver (Dropbox)

```php
<?php
// app/Providers/StorageServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use League\Flysystem\Dropbox\DropboxAdapter;
use Dropbox\Client as DropboxClient;

class StorageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Storage::extend('dropbox', function ($app, $config) {
            $client = new DropboxClient(
                $config['authorization_token'],
                $config['app_secret']
            );

            $adapter = new DropboxAdapter($client);

            return new Filesystem($adapter);
        });
    }
}
```

```php
// config/filesystems.php

'disks' => [
    'dropbox' => [
        'driver' => 'dropbox',
        'authorization_token' => env('DROPBOX_TOKEN'),
        'app_secret' => env('DROPBOX_SECRET'),
    ],
],
```

**Expected Output:**

- The `dropbox` disk is registered and available via `Storage::disk('dropbox')`.
- Files can be stored and retrieved from Dropbox.

**Why This Code Produces That Result:**

- `Storage::extend()` registers the `dropbox` driver.
- The closure returns a Flysystem `Filesystem` with the Dropbox adapter.
- The disk is selected via `config/filesystems.php`.

### Real-World Cases

**Case 1: Custom API Key Authentication**

A B2B SaaS platform registers a custom `api_key` guard for machine-to-machine authentication.

**Case 2: Multi-Cloud Storage**

A multi-cloud application registers custom storage drivers for S3, GCS, and Azure, selecting per environment.

**Case 3: Redis Cluster Cache**

A high-traffic application registers a custom Redis Cluster cache driver.

### References

- Laravel Extending: Cache - https://laravel.com/docs/5.0/extending#cache
- Laravel Extending: Authentication - https://laravel.com/docs/5.0/extending#authentication
- Laravel Filesystem: Custom Filesystems - https://laravel.com/docs/12.x/filesystems#custom-filesystems
- Stack Overflow: Laravel Storage extend example - https://stackoverflow.com/questions/42558512/laravel-storage-extend-example
- GitHub: Adding Custom Guards - https://github.com/laravel/framework/blob/11.x/src/Illuminate/Auth/AuthManager.php


## 6. Enhanced: Publishing Assets (Configuring the `publishes()` Pipeline to Safely Export Customizable Views, Language Resources, and Migrations to User Application Roots)

### Definitions

**Core Definition:** Asset publishing is the mechanism by which a Laravel package exports its configuration files, views, translations, migrations, and other assets to the host application's directories, allowing users to customize them without modifying vendor files.

**Technical Definition:** The `publishes(array $paths, mixed $groups = null)` method on a Service Provider registers a mapping of package paths to application paths. When the user runs `php artisan vendor:publish`, Laravel copies the files from the package to the specified application locations. The method supports publish groups (tags), allowing users to publish specific asset categories via `--tag=group-name`. Assets can include config files, views, translations, migrations, and public assets. The `publishesMigrations()` method is a specialized version for migrations.

**Beginner-Friendly Explanation:** A package's assets are like the default settings on a new phone. `publishes()` lets you copy those defaults to your own phone so you can customize them. Instead of editing the manufacturer's settings (which get overwritten on update), you copy them to your own settings folder and edit them there. The `--tag` option lets you publish just the settings you want—like copying just the ringtones but not the wallpaper.

### Purposes

- To allow users to customize package configuration without editing vendor files.
- To override package views with application-specific templates.
- To translate package strings into the application's language.
- To customize package migrations before running them.
- To publish public assets (JS, CSS, images) to the application's public directory.
- To provide a clean, upgrade-safe customization workflow.
- To group publishable assets by category using tags.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// In a service provider's boot() method

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class PackageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Publish configuration
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php'),
        ], 'package-config');

        // Publish views
        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/package'),
        ], 'package-views');

        // Publish translations
        $this->publishes([
            __DIR__.'/../lang' => $this->app->langPath('vendor/package'),
        ], 'package-lang');

        // Publish migrations
        $this->publishesMigrations([
            __DIR__.'/../database/migrations' => database_path('migrations'),
        ], 'package-migrations');

        // Publish assets (JS/CSS/images)
        $this->publishes([
            __DIR__.'/../dist' => public_path('vendor/package'),
        ], 'package-assets');
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `publishes($paths, $group)` | Maps package paths to application paths |
| `$paths` | Array of `source => destination` mappings |
| `$group` | Optional tag name for selective publishing |
| `config_path()` | Resolves to `config/` |
| `resource_path()` | Resolves to `resources/` |
| `database_path()` | Resolves to `database/` |
| `public_path()` | Resolves to `public/` |
| `publishesMigrations()` | Specialized method for migrations |

#### Syntax Rules

1. `publishes()` **must** be called in a service provider's `boot()` method.
2. The `$paths` array **must** map source paths to destination paths.
3. The `$group` parameter is optional; when provided, it enables selective publishing.
4. `vendor:publish` copies files; `--force` overwrites existing files.
5. `publishesMigrations()` **should** be used for migrations instead of `publishes()`.
6. `loadViewsFrom()`, `loadTranslationsFrom()`, and `loadMigrationsFrom()` allow the package to use its own resources without publishing.
7. Published files **should not** be committed to version control in the package.

#### Constraints and Limitations

- **Overwrite Risk:** `vendor:publish` overwrites existing files; users should back up customizations.
- **Upgrade Conflicts:** Published files become stale after package updates; users must re-publish.
- **View Overriding:** Published views override package views; if the package updates, the published views may be outdated.
- **Migration Duplication:** If migrations are auto-loaded via `loadMigrationsFrom()`, publishing them can cause duplication.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Full Asset Publishing Pipeline

**Step-by-Step Setup Guide:**

1. Create a package service provider.
2. Register config, views, translations, and migrations for publishing.
3. Run `vendor:publish` with specific tags.
4. Verify the assets are published to the correct locations.

**Complete Executable Code:**

```php
<?php
// src/PackageServiceProvider.php

namespace Acme\Package;

use Illuminate\Support\ServiceProvider;

class PackageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Config
        $this->publishes([
            __DIR__.'/../config/acme.php' => config_path('acme.php'),
        ], 'acme-config');

        // Views
        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/acme'),
        ], 'acme-views');

        // Translations
        $this->publishes([
            __DIR__.'/../lang' => $this->app->langPath('vendor/acme'),
        ], 'acme-lang');

        // Migrations
        $this->publishesMigrations([
            __DIR__.'/../database/migrations' => database_path('migrations'),
        ], 'acme-migrations');

        // Public assets
        $this->publishes([
            __DIR__.'/../dist' => public_path('vendor/acme'),
        ], 'acme-assets');

        // Load views from package (fallback if not published)
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'acme');

        // Load translations
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'acme');

        // Load migrations (auto-run without publishing)
        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
    }
}
```

```bash
# Publish only the config
php artisan vendor:publish --tag=acme-config

# Publish only the views
php artisan vendor:publish --tag=acme-views

# Publish everything
php artisan vendor:publish --provider="Acme\Package\PackageServiceProvider"

# Force overwrite
php artisan vendor:publish --tag=acme-config --force
```

**Expected Output:**

- `acme-config` publishes `config/acme.php`.
- `acme-views` publishes views to `resources/views/vendor/acme/`.
- `acme-lang` publishes translations to `lang/vendor/acme/`.
- `acme-migrations` publishes migrations to `database/migrations/`.
- `acme-assets` publishes assets to `public/vendor/acme/`.

**Why This Code Produces That Result:**

- `publishes()` maps source paths to destination paths.
- The tag name allows selective publishing.
- `loadViewsFrom()`, `loadTranslationsFrom()`, and `loadMigrationsFrom()` provide fallback loading when not published.

#### Example 2: Publishing with Grouped Tags

```php
<?php
// src/PackageServiceProvider.php

namespace Acme\Package;

use Illuminate\Support\ServiceProvider;

class PackageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Group 1: Configuration
        $this->publishes([
            __DIR__.'/../config/acme.php' => config_path('acme.php'),
            __DIR__.'/../config/acme-advanced.php' => config_path('acme-advanced.php'),
        ], 'acme-config');

        // Group 2: Views and translations
        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/acme'),
            __DIR__.'/../lang' => $this->app->langPath('vendor/acme'),
        ], 'acme-ui');

        // Group 3: Migrations
        $this->publishesMigrations([
            __DIR__.'/../database/migrations' => database_path('migrations'),
        ], 'acme-migrations');
    }
}
```

```bash
# Publish only configuration
php artisan vendor:publish --tag=acme-config

# Publish UI assets (views + translations)
php artisan vendor:publish --tag=acme-ui
```

**Expected Output:**

- `acme-config` publishes both config files.
- `acme-ui` publishes views and translations together.
- `acme-migrations` publishes migrations.

**Why This Code Produces That Result:**

- Multiple paths can share the same group tag.
- Users can publish related assets together or individually.

### Real-World Cases

**Case 1: Laravel Horizon**

Horizon publishes its config via `horizon-config` and assets via `horizon-assets`, allowing users to customize the dashboard.

**Case 2: Laravel Telescope**

Telescope publishes its config and migrations, allowing users to customize the dashboard and database schema.

**Case 3: Spatie Packages**

Spatie packages publish config files, views, and migrations with clear tags like `permission-config` and `permission-migrations`.

### References

- Laravel Package Development: Resources - https://laravel.com/docs/12.x/packages#resources
- Laravel Package Development: Configuration - https://laravel.com/docs/12.x/packages#configuration
- Laravel Package Development: Views - https://laravel.com/docs/12.x/packages#views
- Laravel Package Development: Migrations - https://laravel.com/docs/12.x/packages#migrations
- Laravel API: ServiceProvider::publishes - https://api.laravel.com/docs/11.x/Illuminate/Support/ServiceProvider.html#method_publishes
- Laravel API: ServiceProvider::publishesMigrations - https://api.laravel.com/docs/11.x/Illuminate/Support/ServiceProvider.html#method_publishesMigrations


## Summary Table of Laravel Extending Concepts

| Concept | Mechanism | Key Method/Property | Use Case |
|---------|-----------|-------------------|----------|
| Custom Services | Container binding | `singleton()`, `bind()` | Helper singletons, business abstractions |
| Package Integration | Auto-discovery | `extra.laravel.providers` | Zero-config package installation |
| Custom Providers | Domain separation | `ServiceProvider` subclasses | API, console, payments |
| Framework Extension Points | Manager pattern | `Manager::extend()` | Custom cache, session, auth drivers |
| Custom Driver Injection | Subsystem managers | `Auth::extend()`, `Cache::extend()`, `Storage::extend()` | JWT guards, MongoDB cache, Dropbox storage |
| Publishing Assets | Asset pipeline | `publishes()`, `publishesMigrations()` | Config, views, translations, migrations |


## References

- Laravel Package Development Documentation (12.x) - https://laravel.com/docs/12.x/packages
- Laravel Service Providers Documentation (12.x) - https://laravel.com/docs/12.x/providers
- Laravel Service Container Documentation (12.x) - https://laravel.com/docs/12.x/container
- Laravel Filesystem: Custom Filesystems - https://laravel.com/docs/12.x/filesystems#custom-filesystems
- Laravel Extending: Managers & Factories - https://laravel.com/docs/5.0/extending#managers-and-factories
- Laravel Package Development: Package Discovery - https://laravel.com/docs/12.x/packages#package-discovery
- Laravel Package Development: Resources - https://laravel.com/docs/12.x/packages#resources
- Laravel API: ServiceProvider - https://api.laravel.com/docs/11.x/Illuminate/Support/ServiceProvider.html
- Laravel API: Manager - https://api.laravel.com/docs/11.x/Illuminate/Support/Manager.html
- Laravel API: CacheManager - https://api.laravel.com/docs/11.x/Illuminate/Cache/CacheManager.html
- Laravel API: BroadcastManager - https://api.laravel.com/docs/11.x/Illuminate/Broadcasting/BroadcastManager.html
- Laravel API: PackageManifest - https://api.laravel.com/docs/11.x/Illuminate/Foundation/PackageManifest.html
- GitHub: Laravel Framework - https://github.com/laravel/framework
- GitHub: Package Development Guide - https://github.com/sgflores/optioner/blob/main/PACKAGE-DEVELOPMENT-GUIDE.md
- Laravel News: Advanced Application Architecture through Laravel's Service Container Management - https://laravel-news.com/service-container-management
- Stack Overflow: Laravel Storage extend example - https://stackoverflow.com/questions/42558512/laravel-storage-extend-example
- Stack Overflow: How to register custom broadcaster - https://stackoverflow.com/questions/32887473/laravel-how-to-register-custom-broadcaster
- Stack Overflow: Laravel load and initialize custom helper class - https://stackoverflow.com/questions/46856505/laravel-load-and-initialize-custom-helper-class