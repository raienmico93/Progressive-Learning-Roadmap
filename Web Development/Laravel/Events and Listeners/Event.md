# Laravel Events: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Events provide a simple observer pattern implementation that allows you to subscribe to and listen for various things that happen within your application, decoupling the code that *triggers* an action from the code that *reacts* to it.

**Technical Definition**

An event is a PHP class that acts as a data container, holding information related to something that happened (e.g., an order was shipped). A listener is a class that contains the logic to be executed when a specific event is dispatched. The framework's event dispatcher maps events to their listeners, invoking each listener's `handle` method when the event is fired. Events can be dispatched synchronously or queued for asynchronous processing, and listeners can be registered manually, via discovery, or through subscribers.

**Beginner-Friendly Explanation**

Think of events as announcements. When something important happens in your app—like a user registering or an order being placed—you “announce” it (dispatch an event). Other parts of your app can “listen” for that announcement and spring into action. For example, when an `OrderShipped` event is announced, one listener might send a shipping confirmation email, while another updates the user's loyalty points. The code that ships the order doesn't need to know about emails or loyalty points; it just makes the announcement, and the listeners handle the rest.

---

### Key Characteristics

- **Observer Pattern**: Implements a classic publish-subscribe mechanism where the subject (event) is decoupled from its observers (listeners).
- **Single Event, Multiple Listeners**: One event can trigger many independent listeners, each handling a separate concern (e.g., sending emails, logging, updating caches).
- **Automatic Discovery**: By default, Laravel automatically scans the `app/Listeners` directory and registers listeners based on the type-hinted event in their `handle` or `__invoke` method.
- **Queueable Listeners**: Listeners can implement `ShouldQueue` to be processed asynchronously by queue workers, keeping the request cycle fast.
- **Transactional Safety**: Events can implement `ShouldDispatchAfterCommit` to ensure they are only fired after a database transaction has been successfully committed.
- **Testing-Friendly**: `Event::fake()` allows you to prevent listeners from executing during tests and assert that specific events were dispatched.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with the `app/Events` and `app/Listeners` directories (created automatically by Artisan).
- **Basic knowledge of PHP classes and dependency injection**.
- **A configured queue driver** (for queued listeners; optional but recommended for production).

---

### Related Programming Areas

- **Eloquent Model Events**: Model lifecycle hooks (`created`, `updated`, `deleted`) are a specialized form of events handled by observers.
- **Queues**: Queued listeners are processed by the same queue system as jobs.
- **Broadcasting**: Events implementing `ShouldBroadcast` can be broadcast to a frontend via WebSockets.
- **Testing**: Event fakes are part of Laravel's testing helpers for isolating side effects.
- **Event Subscribers**: Classes that subscribe to multiple events, useful for grouping related listeners.

---

## Core Concepts

---

### 1. Events

#### Definitions

**Core Definition**

An event is a simple PHP class that represents something that has happened in the application, acting as a data container that holds information relevant to that occurrence.

**Technical Definition**

Event classes are typically stored in `app/Events` and usually contain public properties that hold the data associated with the event (e.g., an `Order` model instance). They are plain PHP objects (POPOs) and do not extend any base class, though they often use the `Dispatchable`, `InteractsWithSockets`, and `SerializesModels` traits for convenience. Events can be dispatched using the `dispatch()` helper or the static `dispatch()` method provided by the `Dispatchable` trait.

**Beginner-Friendly Explanation**

An event is an announcement that “something happened.” It holds the details—like which order was shipped—so that listeners have the information they need to react.

#### Purposes

- To encapsulate the details of a domain occurrence in a dedicated, reusable class.
- To decouple the code that triggers an action from the code that reacts to it.
- To provide a structured payload for listeners to consume.
- To enable broadcasting the occurrence to a frontend or external system.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Events/OrderShipped.php

namespace App\Events;

use App\Models\Order;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Dispatchable` trait | Provides the static `dispatch()` method. |
| `SerializesModels` trait | Ensures Eloquent models are serialized correctly for queued listeners. |
| Public properties | Hold the event data (e.g., the `Order` instance). |

**Syntax Rules**

- Event classes do not extend a base class.
- Public properties are the primary mechanism for carrying data.
- `Dispatchable` and `SerializesModels` are optional but commonly used.

**Constraints and Limitations**

- Events should be immutable data containers; business logic belongs in listeners.
- Large or complex objects in the payload can cause serialization issues for queued listeners.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Defining an Event**

```php
<?php
// app/Events/OrderShipped.php

namespace App\Events;

use App\Models\Order;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, SerializesModels;

    public function __construct(public Order $order) {}
}
```

**Expected Output**

The event class is created and ready to be dispatched.

**Why This Output Occurs**

Artisan’s `make:event` command generates this structure, providing a clean data container with the necessary traits.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Pipeline**

An `OrderShipped` event is dispatched when an order is marked as shipped. It carries the `Order` model, which listeners use to send a confirmation email, update the order status log, and notify the warehouse.

**Explanation**: The event acts as a single source of truth for the shipping occurrence, and multiple listeners can react without coupling.

---

### 2. Event Creation (`artisan make:event`)

#### Definitions

**Core Definition**

`artisan make:event` is the Artisan command that scaffolds a new event class in the `app/Events` directory.

**Technical Definition**

The command generates a PHP class with the `Dispatchable` and `SerializesModels` traits already imported (though not necessarily used, depending on the stub). It optionally prompts for the event name if not provided.

**Beginner-Friendly Explanation**

Creating an event is as easy as running `php artisan make:event OrderShipped`. Laravel creates the file with the right structure.

#### Purposes

- To scaffold a correctly structured event class without manual boilerplate.
- To enforce Laravel’s directory and naming conventions.
- To reduce the friction of creating new events.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:event OrderShipped
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:event {name}` | Creates `app/Events/{name}.php`. |

**Syntax Rules**

- The event name should be PascalCase (e.g., `OrderShipped`).
- The generated file includes the `Dispatchable` and `SerializesModels` traits by default.

**Constraints and Limitations**

- The generated stub does not include a constructor or properties; you must add them manually.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating an Event**

```bash
php artisan make:event PodcastProcessed
```

**Expected Output**

```
INFO  Event [app/Events/PodcastProcessed.php] created successfully.
```

**Why This Output Occurs**

The command creates the file in the correct directory with the standard namespace and traits.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Event-Driven Development**

A developer needs a `UserRegistered` event and runs `php artisan make:event UserRegistered`, then immediately adds the `$user` property in the constructor.

**Explanation**: The scaffold removes setup friction, allowing focus on the event’s data.

---

### 3. Event Classes

#### Definitions

**Core Definition**

Event classes are the PHP classes that represent a specific occurrence, carrying the data relevant to that occurrence.

**Technical Definition**

An event class is a plain PHP object (POPO) that typically resides in `app/Events`. It may use the `Dispatchable`, `InteractsWithSockets`, and `SerializesModels` traits. The `SerializesModels` trait ensures that Eloquent models are correctly serialized when the event is passed to a queued listener. Event classes can also implement interfaces like `ShouldBroadcast` or `ShouldDispatchAfterCommit` to modify their behaviour.

**Beginner-Friendly Explanation**

An event class is the “announcement” itself. It holds the information about what happened, like the order that was shipped.

#### Purposes

- To serve as a typed, structured payload for listeners.
- To provide a clear, discoverable name for a domain occurrence.
- To enable optional behaviours like broadcasting or transactional safety via interfaces.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Events/OrderShipped.php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(public Order $order) {}
}
```

**Component Breakdown**

| Interface/Trait | Purpose |
|-----------------|---------|
| `ShouldDispatchAfterCommit` | Delays dispatch until the DB transaction commits. |
| `ShouldBroadcast` | Enables WebSocket broadcasting. |
| `Dispatchable` | Provides the static `dispatch()` method. |
| `SerializesModels` | Ensures models are serialized for queued listeners. |

**Syntax Rules**

- Event classes should be immutable (readonly properties or no setters).
- Use public properties for data access by listeners.
- Implement interfaces only when their behaviour is needed.

**Constraints and Limitations**

- `ShouldDispatchAfterCommit` requires an active database transaction to have an effect.
- Broadcasting requires a configured broadcast driver.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Event with After-Commit Safety**

```php
<?php
// app/Events/OrderShipped.php

namespace App\Events;

use App\Models\Order;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, SerializesModels;

    public function __construct(public Order $order) {}
}
```

**Expected Output**

The event is dispatched only after the current database transaction commits; if the transaction rolls back, the event is discarded.

**Why This Output Occurs**

The `ShouldDispatchAfterCommit` interface instructs Laravel to defer dispatch until after the transaction completes successfully.

---

#### Real-World Cases with Explanation

**Case 1: Transactional Order Events**

An order service dispatches an `OrderPlaced` event inside a database transaction. By implementing `ShouldDispatchAfterCommit`, the event is only fired after the order is successfully persisted, preventing listeners from acting on a rolled-back order.

**Explanation**: After-commit safety prevents race conditions between database state and event-driven side effects.

---

### 4. Event Dispatching

#### Definitions

**Core Definition**

Event dispatching is the act of firing an event, which triggers the event dispatcher to invoke all registered listeners for that event.

**Technical Definition**

Dispatching can be done via the global `dispatch()` helper or the static `dispatch()` method on the event class (provided by the `Dispatchable` trait). The dispatcher iterates over the registered listeners and calls their `handle` method, passing the event instance. Conditional dispatching is supported via `dispatchIf()` and `dispatchUnless()`. For queued listeners, the listener is pushed onto the queue instead of being executed immediately.

**Beginner-Friendly Explanation**

Dispatching means “announce this event.” When you dispatch, Laravel finds all the listeners that care about it and runs their code.

#### Purposes

- To trigger the observer pattern, notifying all interested listeners.
- To conditionally dispatch events based on runtime conditions.
- To enable both synchronous and asynchronous listener execution.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use App\Events\OrderShipped;

// Standard dispatch
OrderShipped::dispatch($order);

// Helper dispatch
event(new OrderShipped($order));

// Conditional dispatch
OrderShipped::dispatchIf($condition, $order);
OrderShipped::dispatchUnless($condition, $order);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `dispatch()` | Dispatches the event unconditionally. |
| `dispatchIf($condition, $payload)` | Dispatches only if the condition is true. |
| `dispatchUnless($condition, $payload)` | Dispatches only if the condition is false. |

**Syntax Rules**

- The event instance is passed as the sole argument to `dispatch()`.
- `dispatchIf` and `dispatchUnless` accept the condition as the first argument.
- Dispatching returns the array of listener responses.

**Constraints and Limitations**

- Synchronous listeners block the request until they complete.
- Queued listeners are dispatched asynchronously but may not preserve ordering relative to synchronous listeners.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Dispatching an Event from a Controller**

```php
<?php
// app/Http/Controllers/OrderShipmentController.php

namespace App\Http\Controllers;

use App\Events\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class OrderShipmentController extends Controller
{
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);
        $order->ship();

        OrderShipped::dispatch($order);

        return redirect('/orders');
    }
}
```

**Expected Output**

When the controller method runs, the `OrderShipped` event is dispatched, and all registered listeners execute.

**Why This Output Occurs**

`OrderShipped::dispatch($order)` invokes the event dispatcher, which resolves and calls each listener for that event.

---

#### Real-World Cases with Explanation

**Case 1: Conditional Dispatch**

A `UserLoggedIn` event is dispatched only if the user opted into login notifications, using `UserLoggedIn::dispatchIf($user->wants_login_notifications, $user)`.

**Explanation**: Conditional dispatch avoids unnecessary listener execution when the triggering condition is not met.

---

### 5. Event Payloads

#### Definitions

**Core Definition**

Event payloads are the data carried by an event instance, typically stored in public properties, that provide listeners with the information they need to react.

**Technical Definition**

The payload is defined by the event class’s constructor parameters and public properties. For queued listeners, the payload must be serializable; the `SerializesModels` trait handles Eloquent models by serializing only their identifiers and re-retrieving them from the database when the listener runs. Non-serializable data (e.g., closures) cannot be used in queued event payloads.

**Beginner-Friendly Explanation**

The payload is the “details” inside the announcement. It tells the listener which order was shipped, which user registered, etc.

#### Purposes

- To provide listeners with the context they need to perform their actions.
- To decouple the event’s data structure from the listener’s implementation.
- To ensure safe serialization for queued listeners.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
class OrderShipped
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
        public string $trackingNumber,
    ) {}
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Public properties | Hold the payload data. |
| `SerializesModels` | Ensures Eloquent models are serialized safely. |

**Syntax Rules**

- Payload properties should be `public` for easy access by listeners.
- Use `SerializesModels` when including Eloquent models.
- Avoid closures or resources in the payload.

**Constraints and Limitations**

- Eloquent models in queued listeners are re-retrieved from the database, which can fail if the record was deleted.
- Large payloads increase queue storage and processing time.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Multi-Property Payload**

```php
class OrderShipped
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
        public string $trackingNumber,
        public \DateTimeInterface $shippedAt,
    ) {}
}
```

**Expected Output**

Listeners can access `$event->order`, `$event->trackingNumber`, and `$event->shippedAt`.

**Why This Output Occurs**

The public properties expose the payload to any listener that receives the event.

---

#### Real-World Cases with Explanation

**Case 1: Tracking Number for Shipping Emails**

An `OrderShipped` event includes both the `Order` model and a `trackingNumber` string, allowing the email listener to include the tracking link without querying the order again.

**Explanation**: A well-designed payload minimises extra database queries in listeners.

---

### 6. Event Listeners

#### Definitions

**Core Definition**

An event listener is a class that contains the logic to be executed when a specific event is dispatched, typically implemented as a `handle` method that type-hints the event.

**Technical Definition**

Listeners are stored in `app/Listeners` and are automatically discovered by Laravel if their `handle` or `__invoke` method type-hints an event class. They can be synchronous (executed immediately) or queued (implementing `ShouldQueue`). Listeners can also define a `failed` method for handling failures when queued.

**Beginner-Friendly Explanation**

A listener is the code that “reacts” to an event. When the `OrderShipped` event is announced, the `SendShipmentNotification` listener sends the email.

#### Purposes

- To encapsulate the reaction to an event in a dedicated, testable class.
- To allow multiple independent reactions to the same event.
- To keep the event-dispatching code free of side-effect logic.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Listeners/SendShipmentNotification.php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(OrderShipped $event): void
    {
        // Access $event->order, $event->trackingNumber
    }

    public function failed(OrderShipped $event, \Throwable $exception): void
    {
        // Handle failure
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `handle()` | The listener logic; receives the event instance. |
| `ShouldQueue` | Makes the listener asynchronous. |
| `failed()` | Handles queued listener failures. |

**Syntax Rules**

- The `handle` method must type-hint the event class.
- Implement `ShouldQueue` for asynchronous processing.
- Use `InteractsWithQueue` for queue-specific methods (e.g., `release`, `delete`).

**Constraints and Limitations**

- Queued listeners share the queue system’s retry and timeout semantics.
- Synchronous listeners block the dispatching request.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Queued Listener**

```php
<?php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    public function handle(OrderShipped $event): void
    {
        Mail::to($event->order->user)->send(
            new ShipmentConfirmation($event->order, $event->trackingNumber)
        );
    }
}
```

**Expected Output**

The listener is pushed onto the queue and processed asynchronously, sending the email without blocking the request.

**Why This Output Occurs**

`ShouldQueue` tells the dispatcher to push the listener onto the queue instead of executing it immediately.

---

#### Real-World Cases with Explanation

**Case 1: Multiple Listeners for One Event**

An `OrderShipped` event has three listeners: one sends an email, one updates the order log, and one notifies the warehouse API. Each listener is independent and can be queued or synchronous as appropriate.

**Explanation**: Listeners allow separation of concerns, with each handling a single responsibility.

---

### 7. Event Subscribers

#### Definitions

**Core Definition**

Event subscribers are classes that subscribe to multiple events from within the subscriber class itself, allowing you to define several event handlers within a single class.

**Technical Definition**

Subscribers define a `subscribe` method that receives an event dispatcher instance. They can call `$events->listen()` to register listeners, or return an array of events and method names. Subscribers are registered via the `$subscribe` property on the `EventServiceProvider` or manually.

**Beginner-Friendly Explanation**

A subscriber is a single class that listens to multiple events. Instead of creating separate listener classes, you group related event handling in one place.

#### Purposes

- To group related event handling logic into a single class.
- To reduce the number of listener classes for closely related events.
- To provide a centralized place for subscribing to multiple events.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Listeners/UserEventSubscriber.php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    public function handleUserLogin(Login $event): void
    {
        // ...
    }

    public function handleUserLogout(Logout $event): void
    {
        // ...
    }

    public function subscribe(Dispatcher $events): void
    {
        $events->listen(Login::class, [self::class, 'handleUserLogin']);
        $events->listen(Logout::class, [self::class, 'handleUserLogout']);
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `subscribe()` | Registers the subscriber’s listeners. |
| `$events->listen()` | Associates an event with a handler method. |

**Syntax Rules**

- The `subscribe` method receives a `Dispatcher` instance.
- Handlers can be defined as methods on the subscriber class.
- Subscribers are registered in `EventServiceProvider::$subscribe`.

**Constraints and Limitations**

- Subscribers do not support automatic discovery in the same way as individual listeners.
- The `subscribe` method must be manually defined.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Subscriber Registration**

```php
// app/Providers/EventServiceProvider.php
protected $subscribe = [
    \App\Listeners\UserEventSubscriber::class,
];
```

**Expected Output**

The subscriber’s `subscribe` method is called, registering its event handlers.

**Why This Output Occurs**

The `$subscribe` property tells Laravel to instantiate the subscriber and call its `subscribe` method.

---

#### Real-World Cases with Explanation

**Case 1: Authentication Event Grouping**

A `UserEventSubscriber` handles `Login`, `Logout`, `Registered`, and `PasswordReset` events, grouping all authentication-related side effects into one cohesive class.

**Explanation**: Subscribers are ideal when several events share a common domain or concern.

---

### 8. Event Discovery

#### Definitions

**Core Definition**

Event discovery is Laravel’s automatic registration of event listeners by scanning the `app/Listeners` directory for classes with `handle` or `__invoke` methods that type-hint an event.

**Technical Definition**

By default, Laravel automatically finds and registers listeners by scanning the `Listeners` directory. The discovery process looks for any method starting with `handle` or `__invoke` and maps it to the type-hinted event. Custom directories can be added via `withEvents(discover: [...])` in `bootstrap/app.php`. In production, `event:cache` should be run to cache the listener manifest for performance.

**Beginner-Friendly Explanation**

Event discovery means Laravel automatically finds your listeners and connects them to events—you don’t have to manually register them.

#### Purposes

- To eliminate manual listener registration in most cases.
- To reduce boilerplate and configuration.
- To keep listener registration automatic and consistent.

#### Syntax Rules and Structure

**Complete General Syntax**

**Default discovery**

Laravel automatically scans `app/Listeners`.

**Custom directories (`bootstrap/app.php`)**

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
    __DIR__.'/../app/Domain/*/Listeners',
])
```

**Caching for production**

```bash
php artisan event:cache
php artisan event:clear
```

**Component Breakdown**

| Command/Config | Purpose |
|----------------|---------|
| `withEvents()` | Adds custom discovery paths. |
| `event:cache` | Caches the listener manifest. |
| `event:clear` | Clears the cache. |

**Syntax Rules**

- Discovery works by type-hint matching.
- Listener methods must start with `handle` or `__invoke`.
- Production deployments should run `event:cache`.

**Constraints and Limitations**

- Discovery adds a small overhead in development; caching mitigates this in production.
- Complex listener registration (e.g., closures) still requires manual registration.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Discovery Path**

```php
// bootstrap/app.php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```

**Expected Output**

Laravel scans the custom directory and registers any valid listeners it finds.

**Why This Output Occurs**

`withEvents()` extends the default discovery paths to include domain-specific directories.

---

#### Real-World Cases with Explanation

**Case 1: Domain-Driven Directory Structure**

A project organises listeners by domain (`app/Domain/Orders/Listeners`, `app/Domain/Users/Listeners`). The `withEvents()` method adds these paths, keeping discovery automatic while respecting the domain structure.

**Explanation**: Discovery can be customised to match any architectural style.

---

### 9. Event Testing

#### Definitions

**Core Definition**

Event testing is the practice of verifying that specific events are dispatched with the correct payload, using Laravel’s `Event::fake()` helper to prevent listeners from executing.

**Technical Definition**

`Event::fake()` swaps the event dispatcher for a fake that records all dispatched events without invoking listeners. Assertions include `assertDispatched()`, `assertDispatchedTimes()`, `assertNotDispatched()`, `assertNothingDispatched()`, and `assertDispatchedOnce()`. A subset of events can be faked by passing class names to `Event::fake([...])`.

**Beginner-Friendly Explanation**

Event testing checks that your code announces the right events without running the listeners.

#### Purposes

- To verify that events are dispatched under the correct conditions.
- To assert on the event payload without triggering side effects.
- To isolate the code under test from listener side effects.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Event;
use App\Events\OrderShipped;

Event::fake([OrderShipped::class]);

// ... code that dispatches ...

Event::assertDispatched(OrderShipped::class, function ($event) use ($order) {
    return $event->order->id === $order->id;
});

Event::assertDispatchedTimes(OrderShipped::class, 1);
Event::assertNotDispatched(OrderCancelled::class);
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertDispatched()` | Event was dispatched (with optional closure). |
| `assertDispatchedTimes()` | Event was dispatched a specific number of times. |
| `assertNotDispatched()` | Event was not dispatched. |
| `assertNothingDispatched()` | No events were dispatched. |

**Syntax Rules**

- `Event::fake()` must be called before the code under test.
- Pass an array of event classes to fake a subset; other events dispatch normally.
- Assertions are made after the code executes.

**Constraints and Limitations**

- `Event::fake()` prevents all listeners from running, including important side effects.
- Model events are not faked by `Event::fake()` by default; use `Event::fake([ModelCreated::class])` for model events.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Asserting an Event Was Dispatched**

```php
test('shipping an order dispatches OrderShipped', function () {
    Event::fake([OrderShipped::class]);

    $order = Order::factory()->create();
    $order->ship();

    Event::assertDispatched(OrderShipped::class, function ($event) use ($order) {
        return $event->order->is($order);
    });
});
```

**Expected Output**

```
PASS  Tests\Feature\OrderTest
✓ shipping an order dispatches OrderShipped
```

**Why This Output Occurs**

`Event::fake()` records the dispatch, and the assertion confirms the correct event and payload.

---

#### Real-World Cases with Explanation

**Case 1: Controller Event Verification**

A controller dispatches a `UserRegistered` event after registration. A feature test fakes the event, posts to the registration endpoint, and asserts that the event was dispatched with the correct user.

**Explanation**: Event testing verifies the contract between the triggering code and the event system without exercising listeners.

---

### 10. Event Best Practices

#### Definitions

**Core Definition**

Event best practices are the guidelines for designing, dispatching, and handling events to ensure maintainability, performance, and reliability.

#### Key Best Practices

1. **Do Not Rely on Events for Core Logic**: Events are best for side effects (notifications, logging, third-party calls), not for essential application flow. Keep core business logic explicit and independent of event listeners.

2. **Use Events for Multiple Actions**: When one occurrence should trigger multiple independent actions, events are the right tool. For a single action, a direct method call or job is simpler.

3. **Keep Event Classes Immutable**: Use readonly properties or avoid setters to ensure the payload cannot be mutated after dispatch.

4. **Use `SerializesModels` for Queued Listeners**: Always include the `SerializesModels` trait when the event carries Eloquent models, ensuring correct serialization.

5. **Implement `ShouldDispatchAfterCommit` for Transactional Events**: When dispatching inside a database transaction, use `ShouldDispatchAfterCommit` to prevent listeners from acting on rolled-back data.

6. **Queue Side-Effect Listeners**: Listeners that perform slow or unreliable work (emails, API calls) should implement `ShouldQueue` to keep the request fast.

7. **Cache Listener Manifests in Production**: Run `php artisan event:cache` during deployment to eliminate discovery overhead.

8. **Test Event Dispatch**: Use `Event::fake()` to assert that events are dispatched with the correct payload without executing listeners.

9. **Use Subscribers for Related Events**: Group closely related event handlers (e.g., authentication events) into a single subscriber class for cohesion.

10. **Monitor Queued Listener Failures**: Queued listeners can fail; ensure `failed()` methods are defined and that failed jobs are monitored and retried.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Best-Practice Event and Listener**

```php
// Event: immutable, after-commit safe, serializes models
class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, SerializesModels;

    public function __construct(public readonly Order $order) {}
}

// Listener: queued, handles its own failure
class SendShipmentNotification implements ShouldQueue
{
    public function handle(OrderShipped $event): void
    {
        // Send email
    }

    public function failed(OrderShipped $event, \Throwable $e): void
    {
        Log::error('Shipment notification failed', ['order' => $event->order->id, 'error' => $e->getMessage()]);
    }
}
```

**Expected Output**

The event is immutable, dispatched only after the transaction commits, and the queued listener sends the email asynchronously with failure logging.

**Why This Output Occurs**

Each best practice is applied: immutability, after-commit safety, serialization, queuing, and failure handling.

---

#### Real-World Cases with Explanation

**Case 1: Production Event Architecture**

An e-commerce application dispatches an `OrderPlaced` event inside a transaction. The event implements `ShouldDispatchAfterCommit`. Three listeners handle it: one synchronous listener updates a local cache, and two queued listeners send emails and notify the warehouse API. Listener manifests are cached on deploy, and all queued listeners log failures to a monitoring service.

**Explanation**: Combining transactional safety, queuing, and caching yields a robust, production-grade event system.

---

## References

- Events — Laravel 12.x Documentation — https://laravel.com/docs/12.x/events
- Events — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/events
- Events — Laravel 10.x Documentation — https://laravel.com/docs/10.x/events
- Automatic Event / Listener Discovery — Laravel Blog — https://laravel.com/blog/automatic-event-listener-discovery
- EventFake API — https://api.laravel.com/docs/12.x/Illuminate/Support/Testing/Fakes/EventFake.html
- Event Subscribers — Laravel 10.x Documentation — https://laravel.com/docs/10.x/events#event-subscribers
- Recommendation: where to start with events, observers, listeners — Laracasts — https://laracasts.com/discuss/channels/general-discussion/recommendation-where-to-start-with-events-observers-listeners
- Event-Driven Architecture Best Practices — https://github.com/turkeryildirim/ai-skills/blob/main/laravel/rules/arch-event-driven.md
- ShouldDispatchAfterCommit — https://laravel.com/docs/10.x/events#dispatching-events-after-database-transactions

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Event discovery, dispatching, listeners, subscribers, and testing have remained stable across these versions. The `ShouldDispatchAfterCommit` interface is available in Laravel 8+. Event discovery is enabled by default in modern Laravel applications; older versions required opt-in via `shouldDiscoverEvents()`. Always consult the documentation for your specific Laravel version.