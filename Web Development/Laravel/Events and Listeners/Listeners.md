# Laravel Listeners — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A listener is a PHP class or closure that handles an event by executing a specific piece of logic in response to that event being dispatched, enabling decoupled, reactive application behaviour.

**Technical Definition:** Listeners are resolved and executed by the `Illuminate\Events\Dispatcher` class. A class-based listener is a plain PHP class (no required base class) that contains a `handle` method or an `__invoke` method. The first parameter of the handler method is type-hinted with the event class it listens to, and Laravel's automatic event discovery uses PHP reflection to scan the `app/Listeners` directory and register these methods as listeners for their type-hinted events. Listeners may also be registered manually via the `Event` facade's `listen()` method. Listeners implementing the `Illuminate\Contracts\Queue\ShouldQueue` interface are dispatched as queued jobs (`CallQueuedListener`) rather than executed synchronously.

**Beginner-Friendly Explanation:** A listener is the "reaction" to an event. When your application fires an `OrderShipped` event, a listener is the code that actually sends the shipping notification email, updates the customer's loyalty points, or logs the shipment. You can have multiple listeners for a single event, and each one works independently. Listeners can run immediately (synchronous) or in the background (queued).

### Key Characteristics

- **Type-hinted event binding:** The first parameter's type-hint determines which event the listener responds to.
- **Automatic discovery:** Laravel scans `app/Listeners` and registers listeners without manual mapping.
- **Dependency injection:** Listeners are resolved via the service container; dependencies can be injected via the constructor.
- **Synchronous or queued execution:** Implement `ShouldQueue` to run the listener as a background job.
- **Union type support:** A single listener can handle multiple events using PHP union types.
- **Failure handling:** Queued listeners can define `failed()`, `$tries`, `$backoff`, and `retryUntil()` for resilience.
- **Queue customization:** `$connection`, `$queue`, `$delay`, `viaConnection()`, and `viaQueue()` control where and when the listener runs.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the `app/Events` and `app/Listeners` directories.
- For queued listeners: a configured queue connection.
- Basic understanding of Laravel's event system (see the Events cheat sheet).

### Related Programming Areas

- **Observer Pattern** — The design pattern underlying event-listener architecture.
- **Queue System** — Queued listeners are dispatched as jobs on the queue.
- **Service Container** — Listeners are resolved via the container, enabling dependency injection.
- **Event Discovery** — Automatic registration of listeners by scanning directories.
- **Scheduling** — Listeners can be scheduled indirectly via the events they respond to.
- **Testing** — `Event::fake()` and `Bus::fake()` facilitate testing listener behaviour.

### Core Concepts / Features

1. Listener Classes (Building dedicated handlers and utilizing type-hinted parameters to process event payloads)
2. Synchronous Listeners (Executing immediate, blocking tasks inline within the active request lifecycle)
3. Queued Listeners (Converting listeners into background jobs using the `ShouldQueue` interface)
4. Anonymous & Closure Listeners (Registering lightweight inline event observers directly inside service providers or routes)
5. Queue Customization (Configuring explicit queue connections, custom priority levels, and connection limits for heavy operations)

---

## 1. Listener Classes

### Definitions

**Core Definition:** A listener class is a dedicated PHP class that contains the logic to be executed in response to a specific event, with its handler method type-hinted to receive the event instance.

**Technical Definition:** Listener classes are stored in `app/Listeners` and generated via `php artisan make:listener`. They require no base class or interface, but must contain a method named `handle` or `__invoke`. The first parameter of this method must be type-hinted with the event class. Laravel's `DiscoverEvents` class uses PHP reflection to inspect the `handle` method's signature, extract the event type-hint, and register the listener with the `Dispatcher`. Listeners are resolved from the service container, allowing constructor injection of dependencies (services, repositories, etc.). The `handle` method can also type-hint additional dependencies after the event parameter, which are resolved via method injection.

**Beginner-Friendly Explanation:** A listener class is a dedicated PHP file where you write the code that runs when an event fires. You generate it with `php artisan make:listener`, then fill in the `handle()` method. The key detail is the type-hint on the first parameter — that's how Laravel knows which event this listener is for. You can also inject services into the listener's constructor or the `handle()` method.

### Purposes

- To encapsulate the logic that runs in response to an event in a dedicated, testable class.
- To enable dependency injection of services and repositories into listener logic.
- To allow the same event to have multiple independent listeners.
- To support queuing for background execution via `ShouldQueue`.
- To provide a structured, discoverable place for event reaction logic.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Listeners/SendShipmentNotification.php

namespace App\Listeners;

use App\Events\OrderShipped;
use App\Services\NotificationService;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     * Dependencies are injected via the constructor.
     */
    public function __construct(
        private NotificationService $notificationService,
    ) {}

    /**
     * Handle the event.
     * The first parameter must be type-hinted with the event class.
     * Additional dependencies can be injected after the event parameter.
     */
    public function handle(OrderShipped $event): void
    {
        $this->notificationService->sendShipmentAlert(
            $event->order,
            $event->trackingNumber
        );
    }
}
```

```bash
# Generate a listener for a specific event
php artisan make:listener SendShipmentNotification --event=OrderShipped
```

**Component Breakdown:**

- `__construct()` — Optional constructor for injecting dependencies. Resolved via the service container.
- `handle(EventClass $event)` — The handler method. The first parameter is type-hinted with the event class.
- `OrderShipped $event` — The event instance containing the payload data.
- Additional parameters after the event parameter are resolved via method injection.

**Syntax Rules:**

- The handler method must be named `handle` or `__invoke`.
- The first parameter must be type-hinted with the event class for auto-discovery to work.
- Listeners are resolved from the container; constructor dependencies are injected automatically.
- The `handle` method may type-hint additional dependencies after the event parameter.
- Union types are supported for handling multiple events: `handle(OrderShipped|OrderCancelled $event)`.

**Constraints and Limitations:**

- **Without a type-hint, the listener is not auto-discovered.** You must register it manually.
- **Listeners in non-standard directories are not discovered** unless configured via `withEvents()`.
- **The `handle` method should return `void` or `null`.** Returning `false` from a listener stops propagation to subsequent listeners.
- **Listener classes should not contain state** beyond injected dependencies; they are resolved fresh for each event.

### Annotated Code Examples

**Example 1: Generating and Implementing a Listener**

```bash
php artisan make:listener SendShipmentNotification --event=OrderShipped
```

**Expected Output:**

```
INFO  Listener [app/Listeners/SendShipmentNotification.php] created successfully.
```

**Generated File:**

```php
<?php
// File: app/Listeners/SendShipmentNotification.php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    public function __construct() {}

    public function handle(OrderShipped $event): void {}
}
```

**Implemented Listener:**

```php
<?php
// File: app/Listeners/SendShipmentNotification.php

namespace App\Listeners;

use App\Events\OrderShipped;
use App\Notifications\OrderShippedNotification;
use Illuminate\Support\Facades\Notification;

class SendShipmentNotification
{
    public function handle(OrderShipped $event): void
    {
        // Step 1: Access the event payload
        $order = $event->order;
        $trackingNumber = $event->trackingNumber;

        // Step 2: Send the notification
        Notification::send(
            $order->customer,
            new OrderShippedNotification($order, $trackingNumber)
        );

        // Step 3: Log the shipment
        \Log::info('Shipment notification sent', [
            'order_id'       => $order->id,
            'tracking_number' => $trackingNumber,
        ]);
    }
}
```

```php
// Dispatching the event
use App\Events\OrderShipped;

OrderShipped::dispatch($order, '1Z999AA10123456784');
```

**Expected Output:** The listener's `handle()` method is invoked with the `OrderShipped` event. The customer receives a shipping notification email, and the shipment is logged.

**Why This Output Occurs:** The `make:listener` command generates the class with the `--event` flag, which imports the event class and type-hints the `handle` method. Laravel's auto-discovery scans `app/Listeners`, finds the `handle` method, reads the type-hint (`OrderShipped`), and registers the listener for that event. When the event is dispatched, the `Dispatcher` resolves the listener from the container and calls `handle()` with the event instance.

---

**Example 2: Listener with Union Type for Multiple Events**

```php
<?php
// File: app/Listeners/NotifyCustomer.php

namespace App\Listeners;

use App\Events\OrderShipped;
use App\Events\OrderCancelled;
use App\Services\NotificationService;

class NotifyCustomer
{
    public function __construct(
        private NotificationService $notificationService,
    ) {}

    /**
     * Handle both OrderShipped and OrderCancelled events.
     * The union type allows a single listener to respond to multiple events.
     */
    public function handle(OrderShipped|OrderCancelled $event): void
    {
        if ($event instanceof OrderShipped) {
            $this->notificationService->sendShippedNotice($event->order);
        } elseif ($event instanceof OrderCancelled) {
            $this->notificationService->sendCancellationNotice(
                $event->order,
                $event->reason
            );
        }
    }
}
```

```php
// Dispatching either event triggers the listener
OrderShipped::dispatch($order, '1Z999AA10123456784');
OrderCancelled::dispatch($order, 'Customer requested cancellation');
```

**Expected Output:** The listener handles both event types, dispatching the appropriate notification based on the event instance.

**Why This Output Occurs:** The union type `OrderShipped|OrderCancelled` in the `handle` method's signature tells Laravel's discovery mechanism that this listener responds to both events. The `instanceof` checks inside the method determine which event was received and execute the corresponding logic.

### Real-World Cases

- **Order processing pipeline:** A `SendShipmentNotification` listener emails the customer when an order ships.
- **User registration:** A `SendWelcomeEmail` listener sends a welcome email when a user registers.
- **Inventory management:** An `UpdateStockLevel` listener decrements inventory when an order is placed.
- **Audit logging:** A `LogUserAction` listener records every user action for compliance.
- **Loyalty points:** An `AwardLoyaltyPoints` listener credits the customer's account when an order is delivered.

---

## 2. Synchronous Listeners

### Definitions

**Core Definition:** A synchronous listener is a listener that executes immediately and inline within the same process and request lifecycle as the event dispatch, blocking the triggering code until the listener completes.

**Technical Definition:** By default, all listeners are synchronous unless they implement the `ShouldQueue` interface. When an event is dispatched, the `Dispatcher` iterates over the registered listeners for that event and calls each listener's `handle` method sequentially in the same process. The dispatching code does not continue until all synchronous listeners have completed. If a listener throws an exception, it propagates back to the dispatching code (unless caught). Synchronous listeners are resolved fresh from the service container for each event dispatch.

**Beginner-Friendly Explanation:** A synchronous listener runs right away — the code that fired the event waits for the listener to finish before continuing. This is fine for fast operations like logging or updating a database record, but it can slow down your application if the listener does something slow like sending an email or calling an external API. For those cases, you should use a queued listener instead.

### Purposes

- To execute immediate, fast operations in direct response to an event.
- To ensure that critical logic runs before the response is returned to the user.
- To maintain transactional consistency when the listener must complete before the request ends.
- To provide a simple, no-infrastructure-needed way to react to events.
- To allow the dispatching code to handle exceptions thrown by listeners.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Synchronous listener (no ShouldQueue interface)
namespace App\Listeners;

use App\Events\OrderShipped;

class UpdateOrderStatus
{
    /**
     * Handle the event synchronously.
     */
    public function handle(OrderShipped $event): void
    {
        // This runs immediately when OrderShipped is dispatched
        $event->order->update(['status' => 'shipped']);
    }
}
```

**Component Breakdown:**

- No `ShouldQueue` interface — the listener runs synchronously by default.
- The `handle` method executes inline in the same process as the dispatch.
- Exceptions propagate to the dispatching code.

**Syntax Rules:**

- Synchronous listeners are the default; no interface or configuration is needed.
- The dispatching code blocks until all synchronous listeners complete.
- Exceptions thrown by synchronous listeners propagate to the dispatching code.
- Synchronous listeners are resolved fresh from the container for each dispatch.

**Constraints and Limitations:**

- **Slow listeners block the HTTP response.** Avoid time-consuming operations (email, SMS, external APIs) in synchronous listeners.
- **Exceptions propagate to the dispatching code.** This may be desirable or problematic depending on the context.
- **Synchronous listeners cannot be retried.** If they fail, the failure is immediate.
- **Ordering is not guaranteed across multiple listeners.** While Laravel executes them in registration order, relying on this is fragile.

### Annotated Code Examples

**Example 1: Synchronous Listener for Immediate Database Update**

```php
<?php
// File: app/Listeners/UpdateOrderStatus.php

namespace App\Listeners;

use App\Events\OrderShipped;

class UpdateOrderStatus
{
    /**
     * Handle the event synchronously.
     * This listener updates the order status immediately.
     */
    public function handle(OrderShipped $event): void
    {
        $event->order->update([
            'status'        => 'shipped',
            'shipped_at'    => now(),
            'tracking_number' => $event->trackingNumber,
        ]);
    }
}
```

```php
// Dispatching the event
use App\Events\OrderShipped;

$order = Order::find(123);
OrderShipped::dispatch($order, '1Z999AA10123456784');

// At this point, the order status is already 'shipped'
echo $order->fresh()->status; // 'shipped'
```

**Expected Output:** The order status is updated to 'shipped' before the dispatching code continues. The database is updated immediately.

**Why This Output Occurs:** The `UpdateOrderStatus` listener does not implement `ShouldQueue`, so it executes synchronously when `OrderShipped::dispatch()` is called. The `handle` method updates the order and returns. The dispatching code continues, and the order status is already updated.

---

**Example 2: Synchronous Listener with Exception Propagation**

```php
<?php
// File: app/Listeners/ValidateInventory.php

namespace App\Listeners;

use App\Events\OrderPlaced;
use App\Exceptions\InsufficientInventoryException;

class ValidateInventory
{
    public function handle(OrderPlaced $event): void
    {
        foreach ($event->order->items as $item) {
            $product = $item->product;

            if ($product->stock < $item->quantity) {
                throw new InsufficientInventoryException(
                    "Insufficient stock for {$product->name}. " .
                    "Requested: {$item->quantity}, Available: {$product->stock}"
                );
            }
        }
    }
}
```

```php
// Dispatching the event — the exception propagates to the controller
use App\Events\OrderPlaced;
use App\Exceptions\InsufficientInventoryException;

try {
    OrderPlaced::dispatch($order);
} catch (InsufficientInventoryException $e) {
    return back()->withError($e->getMessage());
}
```

**Expected Output:** If inventory is insufficient, the exception is thrown by the listener and caught by the dispatching code, allowing graceful error handling.

**Why This Output Occurs:** Synchronous listeners execute in the same process as the dispatch, so exceptions propagate naturally to the caller. This allows the controller to catch the exception and return a user-friendly error message. If the listener were queued, the exception would be handled by the queue worker instead.

### Real-World Cases

- **Order validation:** A synchronous `ValidateInventory` listener checks stock before the order is confirmed.
- **Audit logging:** A synchronous `LogAction` listener records the action in the database before the response is sent.
- **Cache invalidation:** A synchronous `ClearCache` listener invalidates relevant cache entries when data changes.
- **Data transformation:** A synchronous `NormalizeData` listener transforms event data before it is used by other listeners.
- **Security checks:** A synchronous `VerifyPermissions` listener checks authorisation before allowing an action to proceed.

---

## 3. Queued Listeners

### Definitions

**Core Definition:** A queued listener is a listener that implements the `ShouldQueue` interface, causing it to be dispatched to Laravel's queue system as a background job rather than executing inline with the event dispatch.

**Technical Definition:** When a listener implements `Illuminate\Contracts\Queue\ShouldQueue`, the `Dispatcher` does not call the listener's `handle` method directly. Instead, it serializes the listener and the event into a `CallQueuedListener` job and pushes it onto the configured queue. The queue worker later unserializes the job, resolves the listener from the container, and calls `handle()` with the event instance. The `SerializesModels` trait ensures that Eloquent models in the event payload are stored as identifiers and re-fetched from the database upon execution. Queued listeners support `$tries`, `$backoff`, `$maxExceptions`, `retryUntil()`, and a `failed()` method for error handling.

**Beginner-Friendly Explanation:** A queued listener runs in the background instead of immediately. When the event fires, Laravel packages the listener and event data into a job and adds it to the queue. A separate worker process picks up the job later and runs the listener. This means your users don't have to wait for slow operations like sending emails or calling external APIs — those happen behind the scenes.

### Purposes

- To prevent slow operations (email, SMS, API calls) from blocking HTTP responses.
- To enable retry logic for transient failures via the queue system.
- To decouple event dispatch from listener execution temporally.
- To allow listeners to be processed by dedicated queue workers on separate infrastructure.
- To support failure handling through the `failed()` method and retry configuration.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Listeners/SendShipmentNotification.php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * The number of times the queued listener may be attempted.
     */
    public $tries = 5;

    /**
     * The number of seconds to wait before retrying.
     */
    public $backoff = 3;

    /**
     * The maximum number of exceptions allowed before failing.
     */
    public $maxExceptions = 3;

    public function handle(OrderShipped $event): void
    {
        // This runs in the background on the queue
        \Notification::send(
            $event->order->customer,
            new \App\Notifications\OrderShippedNotification($event->order)
        );
    }

    /**
     * Handle a job failure.
     */
    public function failed(OrderShipped $event, Throwable $exception): void
    {
        \Log::error('Shipment notification failed', [
            'order_id' => $event->order->id,
            'error'    => $exception->getMessage(),
        ]);
    }
}
```

```bash
# Generate a queued listener
php artisan make:listener SendShipmentNotification --event=OrderShipped --queued
```

**Component Breakdown:**

- `implements ShouldQueue` — Marks the listener as queued.
- `InteractsWithQueue` — Provides access to `$this->release()`, `$this->delete()`, and other queue methods.
- `$tries` — Maximum number of attempts before failing.
- `$backoff` — Seconds to wait before retrying (or an array for progressive backoff).
- `$maxExceptions` — Maximum exceptions before failing early.
- `failed($event, $exception)` — Called when the listener exhausts all retries.
- `retryUntil()` — Alternative to `$tries`; returns a `DateTime` for time-based expiry.

**Syntax Rules:**

- The listener must implement `ShouldQueue` to be queued.
- The `InteractsWithQueue` trait provides access to queue job methods.
- `$tries` and `retryUntil()` are mutually exclusive; `retryUntil()` takes precedence.
- The `$backoff` property can be an integer or an array of integers.
- The `failed()` method receives the event instance and the exception.

**Constraints and Limitations:**

- **Queued listeners require a running queue worker.** Without one, jobs accumulate in the queue.
- **Serialized models may become stale.** The `SerializesModels` trait re-fetches models, but deleted models cause failures.
- **Queued listeners may run out of order.** Unlike synchronous listeners, ordering is not guaranteed.
- **The `$tries` count is per-listener, not per-event.** Multiple listeners for the same event each have their own retry count.
- **Queued listeners do not receive the original HTTP request context.** `request()` and `auth()` may not work as expected.

### Annotated Code Examples

**Example 1: Queued Listener with Retry Configuration**

```php
<?php
// File: app/Listeners/ProcessVideoUpload.php

namespace App\Listeners;

use App\Events\VideoUploaded;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class ProcessVideoUpload implements ShouldQueue
{
    use InteractsWithQueue;

    public $tries = 5;
    public $backoff = [10, 30, 60, 120, 300]; // Progressive backoff
    public $maxExceptions = 3;
    public $timeout = 300; // 5 minutes

    public function handle(VideoUploaded $event): void
    {
        // Step 1: Transcode the video
        $transcodedPath = $this->transcodeVideo($event->video);

        // Step 2: Generate thumbnails
        $this->generateThumbnails($event->video);

        // Step 3: Update the video record
        $event->video->update([
            'status'           => 'processed',
            'transcoded_path'  => $transcodedPath,
            'processed_at'     => now(),
        ]);
    }

    public function failed(VideoUploaded $event, Throwable $exception): void
    {
        $event->video->update(['status' => 'failed']);

        \Log::error('Video processing failed', [
            'video_id' => $event->video->id,
            'error'    => $exception->getMessage(),
        ]);
    }

    private function transcodeVideo($video): string
    {
        // Transcoder logic
        return '/processed/' . $video->id . '.mp4';
    }

    private function generateThumbnails($video): void
    {
        // Thumbnail generation logic
    }
}
```

```bash
# Run a queue worker for the listener
php artisan queue:work --tries=5 --backoff=10,30,60,120,300 --timeout=300
```

**Expected Output:**

- The video processing runs in the background.
- If it fails, it retries after 10, 30, 60, 120, and 300 seconds.
- After 5 attempts or 3 exceptions, the `failed()` method is called.
- The video status is set to 'failed', and the error is logged.

**Why This Output Occurs:** The `ShouldQueue` interface tells Laravel to dispatch the listener as a queued job. The `$tries`, `$backoff`, `$maxExceptions`, and `$timeout` properties configure the retry and timeout behaviour. The `failed()` method is invoked when the listener exhausts its retries, allowing cleanup and logging.

---

**Example 2: Conditionally Queued Listener**

```php
<?php
// File: app/Listeners/RewardGiftCard.php

namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class RewardGiftCard implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(OrderCreated $event): void
    {
        // Award a gift card to the customer
        $event->order->customer->giftCards()->create([
            'amount'   => $event->order->subtotal * 0.05,
            'order_id' => $event->order->id,
        ]);
    }

    /**
     * Determine whether the listener should be queued.
     * Only queue the listener for orders over $50.
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

**Expected Output:**

- For orders with a subtotal of $50 or more, the listener is queued and runs in the background.
- For orders below $50, the listener runs synchronously (or is not queued).

**Why This Output Occurs:** The `shouldQueue()` method is checked by the `Dispatcher` before dispatching the listener. If it returns `false`, the listener is executed synchronously. If it returns `true`, the listener is queued. This allows conditional queueing based on event data.

### Real-World Cases

- **Email notifications:** A `SendOrderConfirmation` listener is queued to avoid blocking the checkout response.
- **Video processing:** A `ProcessVideoUpload` listener is queued with a long timeout and retries for transcoding.
- **Third-party API calls:** A `SyncCustomerToCRM` listener is queued to avoid blocking the request when the CRM API is slow.
- **Report generation:** A `GenerateDailyReport` listener is queued with a high retry count to handle database timeouts.
- **Webhook delivery:** A `DeliverWebhook` listener is queued with exponential backoff to handle external service outages.

---

## 4. Anonymous & Closure Listeners

### Definitions

**Core Definition:** Anonymous and closure listeners are lightweight event handlers registered inline using closures, without creating a dedicated listener class, typically for simple, one-off logic that does not warrant its own class file.

**Technical Definition:** Closures are registered via the `Event` facade's `listen()` method, which accepts a closure as the second argument. The closure's first parameter may be type-hinted with the event class, allowing Laravel to pass the event instance. For queued closures, the `Illuminate\Events\queueable()` function wraps the closure in a `QueuedClosure` instance, which supports `onConnection()`, `onQueue()`, and `delay()` methods. Closures are registered in the `boot()` method of a service provider (typically `AppServiceProvider`). Closure-based listeners are not auto-discovered; they must be registered manually.

**Beginner-Friendly Explanation:** Sometimes you have a very simple reaction to an event — like logging a message or incrementing a counter — that doesn't justify creating a whole class file. For those cases, you can use a closure listener: just write the logic inline when you register the event. It's quick and keeps simple logic close to where it's registered.

### Purposes

- To handle simple event logic without creating a dedicated listener class.
- To register listeners conditionally or dynamically at runtime.
- To prototype event handling before committing to a full listener class.
- To keep trivial event reactions close to their registration point.
- To support queued closures for background execution of simple logic.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\Event;

// Basic closure listener
Event::listen(function (OrderShipped $event) {
    \Log::info('Order shipped', ['order_id' => $event->order->id]);
});

// Closure listener with explicit event class
Event::listen(OrderShipped::class, function (OrderShipped $event) {
    // Handle the event
});

// Queued closure listener
use function Illuminate\Events\queueable;

Event::listen(queueable(function (OrderShipped $event) {
    \Notification::send($event->order->customer, new OrderShippedNotification($event->order));
})->onConnection('redis')->onQueue('listeners')->delay(now()->addSeconds(30)));

// Multiple listeners for the same event
Event::listen(OrderShipped::class, function (OrderShipped $event) {
    // First listener
});

Event::listen(OrderShipped::class, function (OrderShipped $event) {
    // Second listener
});
```

**Component Breakdown:**

- `Event::listen(Closure)` — Registers a closure listener. The closure's type-hint determines the event.
- `Event::listen(EventClass::class, Closure)` — Explicitly associates the closure with an event class.
- `queueable(Closure)` — Wraps the closure in a `QueuedClosure` for queued execution.
- `->onConnection()` — Sets the queue connection for the queued closure.
- `->onQueue()` — Sets the queue name for the queued closure.
- `->delay()` — Sets a delay before the closure is executed.

**Syntax Rules:**

- Closure listeners must be registered in a service provider's `boot()` method.
- The closure's first parameter may be type-hinted with the event class for automatic event association.
- Queued closures require the `queueable()` function wrapper.
- `QueuedClosure` supports `onConnection()`, `onQueue()`, and `delay()` methods.
- Closure listeners are not auto-discovered; they must be registered explicitly.

**Constraints and Limitations:**

- **Closures cannot be cached or serialised.** They are not suitable for event discovery or event caching.
- **Closures are harder to test in isolation.** They are not resolved from the container and cannot be mocked.
- **Closure listeners do not support the `failed()` method.** Error handling must be done inside the closure.
- **Closures in service providers may not have access to request context** if registered during bootstrap.
- **Queued closures require the `queueable()` function and are serialised as `QueuedClosure` instances.**

### Annotated Code Examples

**Example 1: Basic Closure Listener for Logging**

```php
<?php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Events\OrderShipped;
use App\Events\UserRegistered;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Step 1: Register a closure listener for OrderShipped
        Event::listen(function (OrderShipped $event) {
            \Log::info('Order shipped', [
                'order_id'       => $event->order->id,
                'tracking_number' => $event->trackingNumber,
                'carrier'        => $event->carrier,
            ]);
        });

        // Step 2: Register a closure listener for UserRegistered
        Event::listen(function (UserRegistered $event) {
            \Log::info('User registered', [
                'user_id' => $event->user->id,
                'email'   => $event->user->email,
            ]);
        });
    }
}
```

```php
// Dispatching the event
use App\Events\OrderShipped;
use App\Events\UserRegistered;

OrderShipped::dispatch($order, '1Z999AA10123456784', 'UPS');
UserRegistered::dispatch($user);
```

**Expected Output:**

- The `OrderShipped` closure logs the order ID, tracking number, and carrier.
- The `UserRegistered` closure logs the user ID and email.

**Why This Output Occurs:** The closures are registered with the `Event` facade in the service provider's `boot()` method. The type-hints on the closure parameters (`OrderShipped`, `UserRegistered`) tell Laravel which events to associate with each closure. When the events are dispatched, the closures are invoked with the event instances.

---

**Example 2: Queued Closure Listener with Connection and Queue Customization**

```php
<?php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Events\VideoUploaded;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;
use function Illuminate\Events\queueable;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Register a queued closure listener for video processing
        Event::listen(
            queueable(function (VideoUploaded $event) {
                // Step 1: Transcode the video in the background
                $transcodedPath = app(\App\Services\VideoTranscoder::class)
                    ->transcode($event->video);

                // Step 2: Update the video record
                $event->video->update([
                    'status'          => 'processed',
                    'transcoded_path' => $transcodedPath,
                    'processed_at'    => now(),
                ]);

                // Step 3: Send a notification
                \Notification::send(
                    $event->video->user,
                    new \App\Notifications\VideoProcessedNotification($event->video)
                );
            })
            ->onConnection('redis-video')
            ->onQueue('video-processing')
            ->delay(now()->addSeconds(30))
        );
    }
}
```

```php
// Dispatching the event
use App\Events\VideoUploaded;

VideoUploaded::dispatch($video);
```

**Expected Output:**

- The video processing runs in the background on the `redis-video` connection and `video-processing` queue.
- The closure is executed 30 seconds after dispatch.
- The video is transcoded, updated, and the user is notified.

**Why This Output Occurs:** The `queueable()` function wraps the closure in a `QueuedClosure` instance, which supports the `onConnection()`, `onQueue()`, and `delay()` methods. The closure is serialised and pushed to the specified queue. A queue worker on the `redis-video` connection processes the job, unserialises the closure, and executes it.

### Real-World Cases

- **Simple logging:** A closure listener logs every user login for audit purposes without creating a dedicated class.
- **Cache invalidation:** A closure listener clears the relevant cache entries when a product is updated.
- **Counter increments:** A closure listener increments a view counter when an article is viewed.
- **Prototyping:** A closure listener is used during development to quickly test event handling before refactoring into a class.
- **Dynamic registrations:** A closure listener is registered conditionally based on the application environment (e.g., only in production).

---

## 5. Queue Customization

### Definitions

**Core Definition:** Queue customization is the practice of controlling how queued listeners are dispatched and processed by specifying the queue connection, queue name, delay, priority, and other queue-related properties on the listener class or at dispatch time.

**Technical Definition:** Queued listeners can customize their queue behaviour through class properties (`$connection`, `$queue`, `$delay`) or methods (`viaConnection()`, `viaQueue()`). The `$connection` property specifies the queue connection (e.g., `redis`, `sqs`, `database`); the `$queue` property specifies the queue name (e.g., `listeners`, `notifications`, `default`); and the `$delay` property specifies the number of seconds to wait before processing. For runtime decisions, the `viaConnection()` and `viaQueue()` methods receive the event instance and return the connection or queue name dynamically. Queue priority is managed by running workers with multiple queues in priority order (`--queue=high,default,low`). The `ThrottlesExceptions` middleware can be applied to listeners to throttle retry attempts. The `ShouldBeUnique` interface prevents duplicate listener execution.

**Beginner-Friendly Explanation:** Not all listeners are equally important. A password reset email is urgent, while a newsletter is not. Queue customization lets you control which queue a listener runs on (so you can have a dedicated "urgent" queue and a "slow" queue), how long to wait before processing it, and which connection to use. This ensures that critical listeners are processed first and heavy workloads don't block everything else.

### Purposes

- To isolate different types of listeners on dedicated queues and connections.
- To prioritise urgent listeners over routine ones using queue priority.
- To delay listener execution for a specified period.
- To distribute listener workloads across multiple queue connections for scalability.
- To throttle listener retries and prevent overwhelming external services.
- To prevent duplicate listener execution via uniqueness constraints.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Listeners/SendUrgentAlert.php

namespace App\Listeners;

use App\Events\CriticalAlert;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendUrgentAlert implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * The queue connection to use.
     */
    public $connection = 'redis-urgent';

    /**
     * The queue name.
     */
    public $queue = 'urgent';

    /**
     * The delay in seconds.
     */
    public $delay = 0;

    /**
     * Dynamic connection selection based on event data.
     */
    public function viaConnection(CriticalAlert $event): string
    {
        return $event->severity === 'critical' ? 'redis-urgent' : 'redis-default';
    }

    /**
     * Dynamic queue selection based on event data.
     */
    public function viaQueue(CriticalAlert $event): string
    {
        return $event->severity === 'critical' ? 'urgent' : 'default';
    }

    public function handle(CriticalAlert $event): void
    {
        // Send the alert
    }
}
```

```bash
# Run a worker with queue priority
php artisan queue:work redis --queue=urgent,high,default,low
```

**Component Breakdown:**

- `$connection` — The queue connection to use (e.g., `redis`, `sqs`, `database`).
- `$queue` — The queue name (e.g., `urgent`, `default`, `notifications`).
- `$delay` — Seconds to wait before processing.
- `viaConnection($event)` — Returns the connection name at runtime based on event data.
- `viaQueue($event)` — Returns the queue name at runtime based on event data.
- `--queue=urgent,high,default,low` — Worker command that processes queues in priority order.

**Syntax Rules:**

- The `$connection`, `$queue`, and `$delay` properties are checked by the `Dispatcher` when queuing the listener.
- The `viaConnection()` and `viaQueue()` methods take precedence over the class properties when defined.
- Queue priority is determined by the order of queues in the worker's `--queue` flag.
- The `ThrottlesExceptions` middleware can be applied via the `middleware()` method on the listener.
- The `ShouldBeUnique` interface prevents duplicate listener execution; the `uniqueId()` method returns the uniqueness key.

**Constraints and Limitations:**

- **Queue priority is worker-level, not job-level.** The worker processes queues in the order specified.
- **Different connections require separate workers.** A worker for `redis-urgent` cannot process jobs on `sqs`.
- **The `$delay` property uses seconds, not minutes.** Use `now()->addMinutes(5)` for longer delays.
- **Dynamic connection/queue selection adds overhead.** Use static properties when possible.
- **Throttling requires a cache driver that supports atomic locks.**

### Annotated Code Examples

**Example 1: Dedicated Queue Connection and Priority**

```php
<?php
// File: app/Listeners/SendCriticalAlert.php

namespace App\Listeners;

use App\Events\SystemAlert;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\Middleware\RateLimited;

class SendCriticalAlert implements ShouldQueue
{
    use InteractsWithQueue;

    // Dedicated connection and queue for critical alerts
    public $connection = 'redis-urgent';
    public $queue = 'urgent';
    public $delay = 0;

    public function handle(SystemAlert $event): void
    {
        // Send SMS and email to on-call engineers
        \Notification::send(
            $event->onCallEngineers,
            new \App\Notifications\CriticalAlertNotification($event->alert)
        );
    }

    // Rate limiting: max 10 alerts per minute per engineer
    public function middleware(): array
    {
        return [new RateLimited('critical-alerts')];
    }
}
```

```php
// File: app/Providers/AppServiceProvider.php

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('critical-alerts', function ($job) {
        return Limit::perMinute(10)->by($job->event->engineerId);
    });
}
```

```php
// File: app/Listeners/GenerateDailyReport.php

namespace App\Listeners;

use App\Events\DailyReportRequested;
use Illuminate\Contracts\Queue\ShouldQueue;

class GenerateDailyReport implements ShouldQueue
{
    // Low-priority connection and queue
    public $connection = 'redis-default';
    public $queue = 'low';
    public $delay = 60; // 1 minute delay

    public function handle(DailyReportRequested $event): void
    {
        // Generate the report (slow operation)
    }
}
```

```bash
# Run workers with priority
php artisan queue:work redis-urgent --queue=urgent
php artisan queue:work redis-default --queue=high,default,low
```

**Expected Output:**

- Critical alerts are processed immediately on the `redis-urgent` connection.
- Daily reports are delayed by 1 minute and processed on the `low` queue after higher-priority queues are drained.
- The rate limiter prevents more than 10 critical alerts per minute per engineer.

**Why This Output Occurs:** The `$connection` and `$queue` properties tell Laravel where to push the listener jobs. The `redis-urgent` worker processes only the `urgent` queue, ensuring critical alerts are not blocked by routine jobs. The `redis-default` worker processes `high`, `default`, and `low` queues in priority order. The `RateLimited` middleware enforces the rate limiter before executing the job.

---

**Example 2: Dynamic Connection and Queue Selection**

```php
<?php
// File: app/Listeners/SyncCustomerData.php

namespace App\Listeners;

use App\Events\CustomerUpdated;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SyncCustomerData implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Dynamically select the queue connection based on the customer's plan.
     */
    public function viaConnection(CustomerUpdated $event): string
    {
        // Enterprise customers get dedicated connections
        if ($event->customer->plan === 'enterprise') {
            return 'redis-enterprise';
        }

        return 'redis-default';
    }

    /**
     * Dynamically select the queue based on the customer's plan.
     */
    public function viaQueue(CustomerUpdated $event): string
    {
        return $event->customer->plan === 'enterprise' ? 'enterprise' : 'standard';
    }

    public function handle(CustomerUpdated $event): void
    {
        // Sync customer data to the CRM
        app(\App\Services\CrmSyncService::class)->sync($event->customer);
    }
}
```

```bash
# Workers for different plans
php artisan queue:work redis-enterprise --queue=enterprise
php artisan queue:work redis-default --queue=standard
```

**Expected Output:**

- Enterprise customers' data is synced on the `redis-enterprise` connection with high priority.
- Standard customers' data is synced on the `redis-default` connection.

**Why This Output Occurs:** The `viaConnection()` and `viaQueue()` methods receive the event instance and return the appropriate connection and queue names based on the customer's plan. This allows a single listener class to be processed with different priority levels depending on the event data.

### Real-World Cases

- **E-commerce order processing:** Order confirmation emails run on a `high` queue, while newsletter subscriptions run on a `low` queue.
- **SaaS multi-tenancy:** Enterprise customers' notifications run on dedicated connections, while free-tier customers share a default connection.
- **Healthcare systems:** Critical patient alerts run on an `urgent` queue with a dedicated worker, while administrative notifications run on a `default` queue.
- **Financial services:** Transaction notifications run on a `high` queue with rate limiting, while statement generation runs on a `low` queue with a delay.
- **Social media platforms:** Real-time engagement notifications (likes, comments) run on a `high` queue, while weekly digest emails run on a `low` queue.

---

## References

- Laravel Events Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/events
- Laravel Events Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/events
- Laravel Events Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/events
- Laravel Queues Documentation — https://laravel.com/docs/12.x/queues
- Laravel `Illuminate\Contracts\Queue\ShouldQueue` API — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Queue/ShouldQueue.html
- Laravel `Illuminate\Queue\InteractsWithQueue` API — https://api.laravel.com/docs/12.x/Illuminate/Queue/InteractsWithQueue.html
- Laravel `Illuminate\Queue\CallQueuedListener` API — https://api.laravel.com/docs/12.x/Illuminate/Queue/CallQueuedListener.html
- Laravel `Illuminate\Queue\Middleware\ThrottlesExceptions` API — https://api.laravel.com/docs/12.x/Illuminate/Queue/Middleware/ThrottlesExceptions.html
- Laravel `Illuminate\Events\Dispatcher` API — https://api.laravel.com/docs/12.x/Illuminate/Events/Dispatcher.html
- Laravel `Illuminate\Foundation\Events\DiscoverEvents` API — https://api.laravel.com/docs/12.x/Illuminate/Foundation/Events/DiscoverEvents.html
- Laravel Queued Event Listeners (GitHub) — https://github.com/Vectorial1024/docs/blob/12.x/events.md
- Laravel Queued Closure Listeners — https://github.com/DevStorm-Team/laravel-book/blob/master/laravel-docs-8.x.pdf
- Laravel Listener Execution Priority (Stack Overflow) — https://stackoverflow.com/questions/79009897/listener-execution-priority-in-laravel-11
- Laravel Closure Listeners (Laracasts) — https://laracasts.com/discuss/channels/general-discussion/how-to-have-multiple-handlers-for-events
- Laravel `QueuedClosure` API — https://api.laravel.com/docs/12.x/Illuminate/Events/QueuedClosure.html
- Laravel Manual Event Registration — https://github.com/laravel/docs/blob/12.x/events.md
- Laravel Queue Priorities (Laravel Docs) — https://laravel.com/docs/12.x/queues#queue-priorities
- Laravel `ShouldBeUnique` Interface — https://laravel.com/docs/12.x/queues#unique-jobs