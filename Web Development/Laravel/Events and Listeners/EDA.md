# Laravel Event-Driven Architecture & Scale — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Event-Driven Architecture in Laravel is a design paradigm where application components communicate through the production, detection, and consumption of events — immutable records of something that has happened — enabling loose coupling, asynchronous processing, and scalable, reactive system design.

**Technical Definition:** Event-Driven Architecture is implemented through the `Illuminate\Events\Dispatcher`, which maintains a registry of event-listener bindings, and the `Illuminate\Contracts\Events\Dispatcher` contract. Domain events are plain PHP objects representing business facts; application events represent technical or infrastructural occurrences. The `ShouldDispatchAfterCommit` interface and the `after_commit` queue configuration option ensure transactional consistency by deferring event dispatch until database transactions commit. The `ThrottlesExceptions` middleware and `RateLimited` middleware provide failure resiliency and rate limiting. Event Subscribers (`Illuminate\Events\Dispatcher::subscribe()`) group related event handlers into a single class. For distributed systems, the outbox/inbox pattern and idempotent handlers ensure eventual consistency across aggregate boundaries.

**Beginner-Friendly Explanation:** Event-driven architecture is a way of building your application where different parts communicate by "announcing" what happened rather than calling each other directly. When an order is placed, the order module announces "OrderPlaced" — it doesn't know or care who listens. The email module, the inventory module, and the analytics module each listen independently and react in their own way. This makes your application easier to extend and scale because adding a new reaction doesn't require changing the original code.

### Key Characteristics

- **Loose coupling:** Event producers and consumers have no direct dependency on each other.
- **Temporal decoupling:** Queued listeners execute asynchronously, allowing the producer to continue without waiting.
- **Transactional consistency:** `ShouldDispatchAfterCommit` and `after_commit` ensure events fire only after the database commits.
- **Failure isolation:** A failing listener does not roll back the primary transaction if it is queued or isolated.
- **Rate limiting and throttling:** `RateLimited` and `ThrottlesExceptions` middleware prevent overwhelming external services.
- **Event orchestration:** Event Subscribers group related handlers into unified classes.
- **Scalability:** Events can be processed by dedicated queue workers on separate infrastructure.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the event system configured (enabled by default).
- A configured queue connection for asynchronous listeners (Redis recommended for production).
- For event caching: the `event:cache` Artisan command (optional, for production performance).

### Related Programming Areas

- **Observer Pattern** — The design pattern underlying event-driven architecture.
- **Queue System** — Queued listeners are dispatched as background jobs.
- **Database Transactions** — `ShouldDispatchAfterCommit` ensures consistency with database state.
- **Rate Limiting** — `RateLimited` and `ThrottlesExceptions` middleware control throughput.
- **Event Subscribers** — Group related event handlers into unified classes.
- **Testing** — `Event::fake()` and assertion helpers facilitate event-driven testing.

### Core Concepts / Features

1. Domain & Application Events (Designing decoupled domain boundaries and distinct event naming spaces)
2. Side-Effect Isolation (Shielding primary application actions from secondary business logic failures)
3. Database Transaction Awareness (Ensuring queued listeners only fire after parent database changes are fully committed)
4. Resiliency & Throttling (Implementing custom retry parameters, exponential backoffs, and rate-limiting rules directly inside the listener)
5. Subscribers & Orchestration (Managing multiple related event handlers cleanly inside unified Event Subscriber classes)

---

## 1. Domain & Application Events

### Definitions

**Core Definition:** Domain events represent significant business facts that have occurred within a bounded context, named in the past tense and expressed in the ubiquitous language of the business; application events represent technical or infrastructural occurrences that are relevant to the application's operation but not to the business domain itself.

**Technical Definition:** Domain events are immutable PHP objects typically extending no base class, using the `Dispatchable` and `SerializesModels` traits. They are named as verbs in the past tense (`OrderPlaced`, `PaymentReceived`, `UserRegistered`) and live within their bounded context's namespace (e.g., `App\Domain\Orders\Events\OrderPlaced`). Application events (`Illuminate\Auth\Events\Login`, `Illuminate\Queue\Events\JobProcessed`) are framework-level occurrences that may trigger listeners but are not part of the domain's ubiquitous language. The distinction ensures that domain boundaries remain clean: the domain does not depend on application or infrastructure concerns, while application events may be used across boundaries.

**Beginner-Friendly Explanation:** Domain events are business announcements — "an order was placed," "a payment was received," "a user registered." They're named in the past tense because they represent things that have already happened. Application events are more technical — "a user logged in," "a job was processed." Keeping them separate ensures your business logic doesn't get tangled up with infrastructure details.

### Purposes

- To represent business facts as immutable, past-tense domain events within bounded contexts.
- To maintain clean boundaries between domain, application, and infrastructure layers.
- To enable eventual consistency across aggregate boundaries through domain event propagation.
- To distinguish between business-critical events and technical/infrastructural events.
- To provide a shared vocabulary (ubiquitous language) for domain events that both developers and domain experts can understand.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Domain event (past tense, within bounded context)
namespace App\Domain\Orders\Events;

use App\Domain\Orders\Models\Order;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderPlaced
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
        public string $customerId,
        public float $totalAmount,
    ) {}
}
```

```php
// Application event (technical occurrence)
namespace App\Events;

use Illuminate\Foundation\Events\Dispatchable;

class CacheWarmed
{
    use Dispatchable;

    public function __construct(
        public string $cacheKey,
        public int $ttl,
    ) {}
}
```

**Component Breakdown:**

- `OrderPlaced` — Past-tense domain event name, representing a business fact.
- `App\Domain\Orders\Events` — Namespace reflecting the bounded context.
- `CacheWarmed` — Application event name, representing a technical occurrence.
- `Dispatchable` — Provides static dispatch methods.
- `SerializesModels` — Safely serializes Eloquent models for queued listeners.

**Syntax Rules:**

- Domain events are named in the past tense (`OrderPlaced`, not `PlaceOrder`).
- Domain events live within their bounded context's namespace.
- Application events live in `App\Events` and are typically framework-level or infrastructural.
- Both domain and application events use the `Dispatchable` trait for static dispatch.
- Domain events use `SerializesModels` when they contain Eloquent models and will be queued.

**Constraints and Limitations:**

- **Domain events should not depend on application or infrastructure concerns.** They are pure data objects.
- **Event naming must be consistent across the codebase.** Pick past tense and stick with it.
- **Cross-module communication should use domain events, not direct model access.** This maintains boundary integrity.
- **Application events may be used across boundaries but should not leak domain knowledge.** Keep them technical.

### Annotated Code Examples

**Example 1: Domain Event with Bounded Context Namespace**

```php
<?php
// File: app/Domain/Orders/Events/OrderPlaced.php

namespace App\Domain\Orders\Events;

use App\Domain\Orders\Models\Order;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderPlaced
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
        public string $customerId,
        public float $totalAmount,
        public array $items,
    ) {}
}
```

```php
// File: app/Domain/Orders/Services/OrderService.php

namespace App\Domain\Orders\Services;

use App\Domain\Orders\Events\OrderPlaced;
use App\Domain\Orders\Models\Order;

class OrderService
{
    public function place(array $data): Order
    {
        // Step 1: Create the order within the domain
        $order = Order::create($data);

        // Step 2: Dispatch the domain event
        OrderPlaced::dispatch(
            $order,
            $data['customer_id'],
            $data['total'],
            $data['items']
        );

        return $order;
    }
}
```

```php
// File: app/Domain/Notifications/Listeners/SendOrderConfirmation.php

namespace App\Domain\Notifications\Listeners;

use App\Domain\Orders\Events\OrderPlaced;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendOrderConfirmation implements ShouldQueue
{
    public function handle(OrderPlaced $event): void
    {
        // React to the domain event from a different bounded context
        \Notification::send(
            $event->order->customer,
            new \App\Notifications\OrderConfirmation($event->order)
        );
    }
}
```

**Expected Output:** The `OrderPlaced` domain event is dispatched within the Orders bounded context. The Notifications bounded context's listener reacts independently, sending an order confirmation without the Orders module knowing about notifications.

**Why This Output Occurs:** The domain event is defined within the Orders bounded context and named in the past tense (`OrderPlaced`). The listener lives in a different bounded context (Notifications) and type-hints the domain event, enabling auto-discovery. The domain event carries only the data needed for cross-boundary communication, maintaining clean boundaries.

---

**Example 2: Application Event vs. Domain Event**

```php
// Application event: technical occurrence
namespace App\Events;

use Illuminate\Foundation\Events\Dispatchable;

class CacheMissed
{
    use Dispatchable;

    public function __construct(
        public string $key,
        public string $store,
    ) {}
}
```

```php
// Domain event: business fact
namespace App\Domain\Inventory\Events;

use App\Domain\Inventory\Models\Product;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class StockLevelDropped
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Product $product,
        public int $previousLevel,
        public int $currentLevel,
        public int $threshold,
    ) {}
}
```

**Expected Output:** The application event `CacheMissed` is dispatched by infrastructure code, while the domain event `StockLevelDropped` is dispatched by domain logic. Listeners can react to either type independently.

**Why This Output Occurs:** Application events are technical and live in `App\Events`; domain events are business-focused and live within bounded contexts. The naming conventions (`CacheMissed` vs. `StockLevelDropped`) reflect the distinction. Keeping them separate ensures that domain logic remains free of infrastructure concerns.

### Real-World Cases

- **E-commerce order pipeline:** `OrderPlaced`, `OrderPaid`, `OrderShipped`, `OrderDelivered` domain events represent the order lifecycle, with listeners in notifications, inventory, and analytics bounded contexts.
- **User onboarding:** `UserRegistered`, `EmailVerified`, `ProfileCompleted` domain events track the onboarding progression across bounded contexts.
- **Inventory management:** `StockLevelDropped`, `StockReplenished`, `ProductDiscontinued` domain events represent inventory changes, with listeners in purchasing, alerting, and reporting.
- **Financial systems:** `PaymentInitiated`, `PaymentSucceeded`, `PaymentFailed` domain events represent payment lifecycle transitions, with listeners in accounting, notifications, and fraud detection.
- **Content management:** `ArticlePublished`, `ArticleUpdated`, `ArticleArchived` domain events represent content lifecycle changes, with listeners in search indexing, caching, and notifications.

---

## 2. Side-Effect Isolation

### Definitions

**Core Definition:** Side-effect isolation is the architectural practice of ensuring that secondary business logic (side effects) — such as sending emails, updating external systems, or generating reports — cannot cause the failure of primary application actions (such as placing an order or registering a user).

**Technical Definition:** Side-effect isolation in Laravel is achieved through three primary mechanisms: (1) queued listeners implementing `ShouldQueue`, which execute asynchronously in a separate process, so their failures do not affect the primary transaction; (2) the `NotificationSending` event listener returning `false` to cancel secondary dispatch, preventing failed side effects from propagating; and (3) transaction-aware event dispatch via `ShouldDispatchAfterCommit`, which ensures that events (and their side effects) are not dispatched at all if the primary transaction fails. When a synchronous listener throws an exception within a transaction, the entire transaction rolls back — hence the recommendation to queue side-effect listeners.

**Beginner-Friendly Explanation:** Side-effect isolation means that if sending a confirmation email fails, it doesn't prevent the order from being placed. The primary action (placing the order) is shielded from failures in secondary actions (sending the email, updating the CRM, generating a report). The key technique is to queue side-effect listeners so they run in the background, separately from the primary transaction.

### Purposes

- To prevent failures in secondary business logic from rolling back primary application actions.
- To ensure that the core business transaction completes successfully regardless of external system failures.
- To provide resilience against third-party API outages (email, SMS, CRM, analytics).
- To maintain data consistency by ensuring side effects only occur after the primary transaction commits.
- To enable independent retry and failure handling for side effects without affecting the primary action.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// Side-effect listener implementing ShouldQueue (isolated from primary transaction)
namespace App\Listeners;

use App\Events\OrderPlaced;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendOrderConfirmation implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(OrderPlaced $event): void
    {
        // This runs in the background, isolated from the primary transaction
        \Notification::send($event->order->customer, new OrderConfirmation($event->order));
    }

    public function failed(OrderPlaced $event, \Throwable $exception): void
    {
        \Log::error('Order confirmation failed', [
            'order_id' => $event->order->id,
            'error'    => $exception->getMessage(),
        ]);
    }
}
```

```php
// Transaction-aware dispatch via ShouldDispatchAfterCommit
namespace App\Events;

use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderPlaced implements ShouldDispatchAfterCommit
{
    use Dispatchable, SerializesModels;

    public function __construct(public \App\Domain\Orders\Models\Order $order) {}
}
```

**Component Breakdown:**

- `ShouldQueue` — Moves the listener to the queue, isolating it from the primary process.
- `InteractsWithQueue` — Provides queue job methods (`release`, `delete`, `fail`).
- `failed($event, $exception)` — Called when the queued listener exhausts retries.
- `ShouldDispatchAfterCommit` — Defers event dispatch until the database transaction commits.

**Syntax Rules:**

- Side-effect listeners should implement `ShouldQueue` to run asynchronously.
- The `failed()` method receives the event instance and the exception for logging and cleanup.
- `ShouldDispatchAfterCommit` ensures that side effects only occur after the primary transaction commits.
- Synchronous listeners within a transaction will roll back the transaction if they throw an exception.

**Constraints and Limitations:**

- **Synchronous side-effect listeners within a transaction can roll back the primary action.** Always queue side-effect listeners.
- **Queued listeners may process before the transaction commits** unless `after_commit` is enabled or `ShouldDispatchAfterCommit` is implemented.
- **The `failed()` method is only called for queued listeners, not synchronous ones.**
- **Side-effect isolation does not guarantee exactly-once delivery.** Idempotency must be implemented in the listener.

### Annotated Code Examples

**Example 1: Isolating Side Effects with Queued Listeners**

```php
<?php
// File: app/Domain/Orders/Listeners/SendOrderConfirmation.php

namespace App\Domain\Orders\Listeners;

use App\Domain\Orders\Events\OrderPlaced;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendOrderConfirmation implements ShouldQueue
{
    use InteractsWithQueue;

    public $tries = 3;
    public $backoff = [10, 30, 60];

    public function handle(OrderPlaced $event): void
    {
        // Step 1: Send the confirmation email
        \Notification::send(
            $event->order->customer,
            new \App\Notifications\OrderConfirmation($event->order)
        );

        // Step 2: Notify the warehouse (external API)
        \Http::timeout(30)->post(config('services.warehouse.url') . '/orders', [
            'order_id' => $event->order->id,
            'items'    => $event->order->items->toArray(),
        ]);
    }

    public function failed(OrderPlaced $event, Throwable $exception): void
    {
        // Log the failure for manual intervention
        \Log::error('Order side effects failed', [
            'order_id' => $event->order->id,
            'error'    => $exception->getMessage(),
        ]);

        // Optionally, mark the order for manual review
        $event->order->update(['status' => 'pending_review']);
    }
}
```

```php
// File: app/Domain/Orders/Services/OrderService.php

namespace App\Domain\Orders\Services;

use App\Domain\Orders\Events\OrderPlaced;
use App\Domain\Orders\Models\Order;
use Illuminate\Support\Facades\DB;

class OrderService
{
    public function place(array $data): Order
    {
        return DB::transaction(function () use ($data) {
            // Step 1: Create the order (primary action)
            $order = Order::create($data);

            // Step 2: Dispatch the event (side effects run in the background)
            OrderPlaced::dispatch($order);

            return $order;
        });
    }
}
```

**Expected Output:** The order is created within the database transaction. The `OrderPlaced` event is dispatched, and the `SendOrderConfirmation` listener is queued. The transaction commits immediately without waiting for the email or warehouse notification. If the email or warehouse API fails, the order is still created successfully.

**Why This Output Occurs:** The `SendOrderConfirmation` listener implements `ShouldQueue`, so it runs asynchronously on the queue. The primary transaction (order creation) completes independently. The listener's `failed()` method handles the failure by logging the error and marking the order for manual review, without rolling back the primary action.

---

**Example 2: Transaction-Aware Dispatch with ShouldDispatchAfterCommit**

```php
<?php
// File: app/Domain/Orders/Events/OrderPlaced.php

namespace App\Domain\Orders\Events;

use App\Domain\Orders\Models\Order;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderPlaced implements ShouldDispatchAfterCommit
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}
}
```

```php
// Dispatching inside a transaction
use App\Domain\Orders\Events\OrderPlaced;
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    $order = Order::create([
        'customer_id' => 123,
        'total'       => 99.99,
    ]);

    // This event is NOT dispatched until the transaction commits
    OrderPlaced::dispatch($order);

    // If an exception occurs here, the transaction rolls back
    // and the event is never dispatched
});
```

**Expected Output:** The `OrderPlaced` event is dispatched only after the database transaction commits successfully. If the transaction rolls back, the event is discarded and no listeners are executed.

**Why This Output Occurs:** The `ShouldDispatchAfterCommit` interface is detected by the `Dispatcher`, which registers the event to be dispatched after the current transaction commits. The `Illuminate\Database\Concerns\ManagesTransactions` class manages the deferred dispatch queue. If the transaction commits, all deferred events are dispatched. If it rolls back, the deferred events are discarded.

### Real-World Cases

- **E-commerce order processing:** Order creation is the primary action; email confirmation, warehouse notification, and loyalty point crediting are side effects that run in the background.
- **User registration:** User creation is the primary action; welcome email, CRM sync, and analytics tracking are side effects.
- **Payment processing:** Payment recording is the primary action; receipt email, accounting sync, and fraud check are side effects.
- **Content publishing:** Article publication is the primary action; search indexing, cache warming, and social media posting are side effects.
- **Healthcare appointments:** Appointment booking is the primary action; SMS reminder scheduling, calendar sync, and insurance verification are side effects.

---

## 3. Database Transaction Awareness

### Definitions

**Core Definition:** Database transaction awareness is the architectural principle that queued listeners must not execute until the database transaction that triggered them has committed, preventing race conditions where a listener reads stale or non-existent data.

**Technical Definition:** Laravel provides three mechanisms for transaction-aware dispatch: (1) the `after_commit` configuration option on queue connections, which defers all queued event listeners, mailables, notifications, and broadcast events until open transactions commit; (2) the `ShouldDispatchAfterCommit` interface on event classes, which defers that specific event; and (3) the `ShouldQueueAfterCommit` interface on listener classes, which defers that specific listener. The `CallQueuedListener` class has an `$afterCommit` property that can be set to `true`. When a transaction is rolled back, deferred events and jobs are discarded. The `laravel-transaction-guard` package provides static analysis to detect unsafe jobs without a proven after-commit strategy.

**Beginner-Friendly Explanation:** When you dispatch an event inside a database transaction, the queued listener might run before the transaction commits. This means the listener could try to read a record that doesn't exist yet — because the transaction hasn't been saved. Transaction awareness fixes this by telling Laravel: "Don't run this listener until the database transaction is safely committed."

### Purposes

- To prevent queued listeners from executing before the database transaction commits.
- To avoid race conditions where listeners read stale or non-existent data.
- To ensure that events are discarded when the parent transaction rolls back.
- To provide a consistent view of the database for listeners.
- To maintain data integrity across the event-driven pipeline.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Option 1: Global queue connection configuration
// config/queue.php
'redis' => [
    'driver'       => 'redis',
    'connection'   => 'default',
    'queue'        => 'default',
    'retry_after'  => 90,
    'block_for'    => null,
    'after_commit' => true, // Defer all queued listeners/notifications
],

// Option 2: Event-level interface
namespace App\Events;

use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderPlaced implements ShouldDispatchAfterCommit
{
    use Dispatchable, SerializesModels;

    public function __construct(public Order $order) {}
}

// Option 3: Listener-level interface
namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\ShouldQueueAfterCommit;

class SendOrderConfirmation implements ShouldQueue, ShouldQueueAfterCommit
{
    use InteractsWithQueue;

    public function handle(OrderPlaced $event): void
    {
        // This listener will only run after the transaction commits
    }
}
```

**Component Breakdown:**

- `'after_commit' => true` — Global queue connection option that defers all queued events.
- `ShouldDispatchAfterCommit` — Event-level interface that defers that specific event.
- `ShouldQueueAfterCommit` — Listener-level interface that defers that specific listener.
- `$afterCommit` property — Can be set to `true` on jobs and listeners for inline control.

**Syntax Rules:**

- The `after_commit` configuration option applies to all queued events, mailables, notifications, and broadcasts on that connection.
- The `ShouldDispatchAfterCommit` interface is implemented on the event class.
- The `ShouldQueueAfterCommit` interface is implemented on the listener class.
- If no transaction is open, the event or job is dispatched immediately.
- If the transaction rolls back, deferred events and jobs are discarded.

**Constraints and Limitations:**

- **The `after_commit` option applies to all queued events on the connection.** It cannot be selectively disabled for specific events.
- **`ShouldDispatchAfterCommit` only works when a transaction is active.** If no transaction is in progress, the event dispatches immediately.
- **The `ShouldQueueAfterCommit` interface is only relevant for queued listeners.** Synchronous listeners execute inline.
- **Deferred events are discarded on transaction rollback.** Ensure that any state changes are also rolled back.

### Annotated Code Examples

**Example 1: Global after_commit Configuration**

```php
// File: config/queue.php

'connections' => [
    'redis' => [
        'driver'       => 'redis',
        'connection'   => 'default',
        'queue'        => 'default',
        'retry_after'  => 90,
        'block_for'    => null,
        'after_commit' => true, // Defer all queued events until transaction commits
    ],
],
```

```php
// Dispatching an event inside a transaction
use App\Events\OrderPlaced;
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    $order = Order::create([
        'customer_id' => 123,
        'total'       => 99.99,
    ]);

    // With after_commit => true, this event is not dispatched
    // until the transaction commits successfully
    OrderPlaced::dispatch($order);
});
```

**Expected Output:** The `OrderPlaced` event is dispatched only after the transaction commits. If the transaction rolls back, the event is discarded.

**Why This Output Occurs:** The `after_commit` configuration option tells Laravel to defer all queued events, mailables, notifications, and broadcasts until all open database transactions have been committed. The `Dispatcher` checks the queue connection's `after_commit` setting and registers the event for deferred dispatch. If the transaction commits, the deferred events are dispatched; if it rolls back, they are discarded.

---

**Example 2: Listener-Level After-Commit with ShouldQueueAfterCommit**

```php
<?php
// File: app/Listeners/UpdateInventory.php

namespace App\Listeners;

use App\Events\OrderPlaced;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\ShouldQueueAfterCommit;

class UpdateInventory implements ShouldQueue, ShouldQueueAfterCommit
{
    use InteractsWithQueue;

    public function handle(OrderPlaced $event): void
    {
        // This listener will only run after the transaction commits
        foreach ($event->order->items as $item) {
            $item->product->decrement('stock', $item->quantity);
        }
    }
}
```

```php
// Dispatching the event
use App\Events\OrderPlaced;
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    $order = Order::create([...]);

    // The event is dispatched immediately, but the UpdateInventory
    // listener will not run until the transaction commits
    OrderPlaced::dispatch($order);
});
```

**Expected Output:** The `OrderPlaced` event is dispatched immediately, but the `UpdateInventory` listener waits until the transaction commits before executing. Other listeners without `ShouldQueueAfterCommit` may run immediately.

**Why This Output Occurs:** The `ShouldQueueAfterCommit` interface on the listener tells the `CallQueuedListener` to set its `$afterCommit` property to `true`. When the listener is queued, it checks whether a transaction is open; if so, it defers execution until the transaction commits. This allows fine-grained control over which listeners are transaction-aware.

### Real-World Cases

- **E-commerce order placement:** The `UpdateInventory` listener must wait for the order transaction to commit before decrementing stock.
- **Payment processing:** The `SendReceipt` listener must wait for the payment transaction to commit before emailing the receipt.
- **User registration:** The `SendWelcomeEmail` listener must wait for the user creation transaction to commit before sending the email.
- **Content publishing:** The `IndexArticle` listener must wait for the article publishing transaction to commit before indexing in search.
- **Financial transactions:** The `UpdateAccountBalance` listener must wait for the transaction record to commit before updating the account balance.

---

## 4. Resiliency & Throttling

### Definitions

**Core Definition:** Resiliency and throttling are the mechanisms that ensure queued listeners can recover from transient failures (network issues, API rate limits, temporary outages) through retry logic with exponential backoff, while preventing them from overwhelming external services through rate limiting and exception throttling.

**Technical Definition:** Laravel provides several tools for listener resiliency: the `$tries` property sets the maximum number of attempts; `$backoff` (integer or array) sets the delay(s) between retries; `$maxExceptions` sets the maximum exceptions before failing; `retryUntil()` sets a time-based retry limit; the `ThrottlesExceptions` middleware delays retries after exceptions; and the `RateLimited` middleware enforces a named rate limiter. The `ThrottlesExceptionsWithRedis` middleware provides Redis-backed exception throttling. The `failed()` method is called when the listener exhausts all retries. The `ShouldBeUnique` interface prevents duplicate listener execution, and FIFO queues with `onGroup()` and `withDeduplicator()` ensure ordered, deduplicated processing.

**Beginner-Friendly Explanation:** Resiliency means your listeners try again when things go wrong — but they don't try forever, and they wait longer between each attempt. Throttling means your listeners don't overwhelm external services (like Twilio or Slack) by sending too many requests too quickly. Together, these mechanisms make your event-driven system robust and respectful of external APIs.

### Purposes

- To ensure transient failures are retried automatically without manual intervention.
- To prevent overwhelming external services with retry storms through exponential backoff.
- To enforce rate limits on API calls to avoid throttling and bans.
- To provide a maximum number of attempts and exceptions before failing permanently.
- To enable time-based retry limits for time-sensitive notifications.
- To prevent duplicate listener execution via uniqueness constraints.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Listeners/SendShipmentNotification.php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\Middleware\RateLimited;
use Illuminate\Queue\Middleware\ThrottlesExceptions;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    // Retry configuration
    public $tries = 5;
    public $backoff = [10, 30, 60, 120, 300]; // Progressive backoff
    public $maxExceptions = 3;
    public $timeout = 30;
    public $failOnTimeout = true;

    // Rate limiting middleware
    public function middleware(): array
    {
        return [
            new RateLimited('shipment-notifications'),
            (new ThrottlesExceptions(5, 5))->backoff(5),
        ];
    }

    public function handle(OrderShipped $event): void
    {
        // Send the shipment notification
    }

    public function failed(OrderShipped $event, Throwable $exception): void
    {
        // Handle permanent failure
    }
}
```

```php
// Rate limiter definition in AppServiceProvider
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('shipment-notifications', function ($job) {
        return Limit::perMinute(60)->by($job->event->order->customer_id);
    });
}
```

**Component Breakdown:**

- `$tries` — Maximum number of attempts before failing.
- `$backoff` — Integer (fixed) or array (progressive) delays in seconds.
- `$maxExceptions` — Maximum exceptions before failing early.
- `$timeout` — Maximum seconds the listener can run.
- `$failOnTimeout` — Whether to mark the job as failed on timeout.
- `RateLimited` — Middleware that enforces a named rate limiter.
- `ThrottlesExceptions` — Middleware that delays retries after exceptions.
- `failed($event, $exception)` — Called when the listener exhausts retries.

**Syntax Rules:**

- The `RateLimited` middleware requires a rate limiter with the same name defined in a service provider.
- The `ThrottlesExceptions` middleware accepts `$maxAttempts` and `$decayMinutes` as constructor arguments.
- The `backoff()` method on `ThrottlesExceptions` sets the delay (in minutes) before retrying.
- The `$backoff` array must have at least as many elements as `$tries` minus one.
- The `failed()` method receives the event instance and the exception.

**Constraints and Limitations:**

- **Rate limiting requires a cache driver that supports atomic locks** (Redis, Memcached, database).
- **`ThrottlesExceptions` releases the job back to the queue** rather than marking it as failed immediately.
- **The `$backoff` array is consumed in order.** If the array has fewer elements than attempts, the last value is reused.
- **`retryUntil()` and `$tries` are mutually exclusive.** `retryUntil()` takes precedence.
- **Rate limiting and throttling add overhead.** Use them only where necessary.

### Annotated Code Examples

**Example 1: Comprehensive Retry and Throttling Configuration**

```php
<?php
// File: app/Listeners/SendCriticalAlert.php

namespace App\Listeners;

use App\Events\SystemAlert;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\Middleware\RateLimited;
use Illuminate\Queue\Middleware\ThrottlesExceptions;
use Throwable;

class SendCriticalAlert implements ShouldQueue
{
    use InteractsWithQueue;

    public $connection = 'redis-urgent';
    public $queue = 'urgent';

    // Retry configuration
    public $tries = 5;
    public $backoff = [5, 15, 60, 300, 900];
    public $maxExceptions = 3;
    public $timeout = 30;
    public $failOnTimeout = true;

    public function middleware(): array
    {
        return [
            // Rate limit: max 10 alerts per minute per engineer
            new RateLimited('critical-alerts'),

            // Throttle exceptions: after 5 exceptions in 5 minutes,
            // wait 5 minutes before retrying
            (new ThrottlesExceptions(5, 5))->backoff(5),
        ];
    }

    public function handle(SystemAlert $event): void
    {
        // Send SMS and email to on-call engineers
        foreach ($event->onCallEngineers as $engineer) {
            \Notification::send($engineer, new \App\Notifications\CriticalAlertNotification($event->alert));
        }
    }

    public function failed(SystemAlert $event, Throwable $exception): void
    {
        \Log::critical('Critical alert delivery failed', [
            'alert_id' => $event->alert->id,
            'error'    => $exception->getMessage(),
        ]);
    }
}
```

```php
// Rate limiter definition in AppServiceProvider
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('critical-alerts', function ($job) {
        return Limit::perMinute(10)->by($job->event->onCallEngineers->first()->id ?? 'global');
    });
}
```

```bash
# Run a dedicated worker for critical alerts
php artisan queue:work redis-urgent --queue=urgent --tries=5 --backoff=5,15,60,300,900 --timeout=30
```

**Expected Output:**

- Critical alerts are attempted up to 5 times with progressive backoff (5, 15, 60, 300, 900 seconds).
- If 3 exceptions occur before 5 attempts, the listener fails early.
- The rate limiter prevents more than 10 alerts per minute per engineer.
- The `ThrottlesExceptions` middleware delays retries after 5 exceptions in 5 minutes.
- After all retries are exhausted, the `failed()` method logs a critical error.

**Why This Output Occurs:** The `$tries`, `$backoff`, `$maxExceptions`, `$timeout`, and `$failOnTimeout` properties configure the retry behaviour. The `RateLimited` middleware enforces the named rate limiter. The `ThrottlesExceptions` middleware releases the job back to the queue with a delay after exceptions. The `failed()` method handles permanent failure.

---

**Example 2: Exponential Backoff with Time-Based Retry Limit**

```php
<?php
// File: app/Listeners/SyncCustomerToCRM.php

namespace App\Listeners;

use App\Events\CustomerUpdated;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use DateTime;

class SyncCustomerToCRM implements ShouldQueue
{
    use InteractsWithQueue;

    public $tries = 10;
    public $backoff = [10, 30, 60, 120, 300, 600, 900, 1800, 3600];

    /**
     * Determine the time at which the listener should timeout.
     * Retry until 24 hours have passed.
     */
    public function retryUntil(): DateTime
    {
        return now()->addDay();
    }

    public function handle(CustomerUpdated $event): void
    {
        // Sync customer data to the CRM
        \Http::timeout(30)->put(
            config('services.crm.url') . '/customers/' . $event->customer->id,
            $event->customer->toArray()
        );
    }
}
```

**Expected Output:** The listener retries up to 10 times with exponential backoff over a 24-hour period. After 24 hours, no further attempts are made, even if fewer than 10 attempts have occurred.

**Why This Output Occurs:** The `retryUntil()` method returns a `DateTime` instance representing the deadline for retries. The queue worker checks this deadline before each attempt; if the deadline has passed, the listener is marked as failed. This allows time-based retry limits that are independent of the attempt count, which is useful for notifications or syncs that become irrelevant after a certain period.

### Real-World Cases

- **SMS notifications via Twilio:** Rate-limited to 60 per minute per customer, with 5 retry attempts and progressive backoff.
- **Slack alerts for system incidents:** Retried up to 10 times with exponential backoff over 24 hours, with a deadline for time-sensitive alerts.
- **Email delivery via SES:** Retried 3 times with a 30-second delay between attempts, with `ThrottlesExceptions` to handle SES throttling.
- **CRM synchronization:** Retried with exponential backoff over 24 hours, respecting the CRM's API rate limits.
- **Webhook delivery:** Retried 5 times with progressive backoff, with a 5-minute delay after exceptions to avoid overwhelming the endpoint.

---

## 5. Subscribers & Orchestration

### Definitions

**Core Definition:** Event Subscribers are classes that group multiple related event handlers into a single unit, providing a centralised place to define and register listeners for multiple events, enabling cleaner orchestration of related event-driven logic.

**Technical Definition:** An Event Subscriber is a class that defines a `subscribe()` method receiving an `Illuminate\Events\Dispatcher` instance. The `subscribe()` method registers listeners by calling `$events->listen(EventClass::class, [SubscriberClass::class, 'methodName'])` or by returning an array mapping event classes to method names. Subscribers are registered in the `EventServiceProvider`'s `$subscribe` property (Laravel 10-) or auto-discovered in Laravel 11+. The subscriber pattern is useful when multiple events share related logic or when a single class should handle a cohesive set of event reactions.

**Beginner-Friendly Explanation:** An Event Subscriber is a class that groups together several related event handlers. Instead of having separate listener classes for `UserLogin`, `UserLogout`, and `UserRegistered`, you can have one `UserEventSubscriber` class with methods for each. This keeps related logic together and reduces the number of files in your `app/Listeners` directory.

### Purposes

- To group multiple related event handlers into a single cohesive class.
- To reduce the number of listener class files in the application.
- To provide a centralised place for registering listeners for a related set of events.
- To enable shared dependencies and state across multiple event handlers.
- To simplify the orchestration of complex event-driven workflows.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Listeners/UserEventSubscriber.php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Auth\Events\Registered;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void
    {
        // Log the login, update last_login_at, etc.
    }

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void
    {
        // Log the logout, clear session data, etc.
    }

    /**
     * Handle user registration events.
     */
    public function handleUserRegistered(Registered $event): void
    {
        // Send welcome email, create profile, etc.
    }

    /**
     * Register the listeners for the subscriber.
     * Returns an array of event => method mappings.
     */
    public function subscribe(Dispatcher $events): array
    {
        return [
            Login::class      => 'handleUserLogin',
            Logout::class     => 'handleUserLogout',
            Registered::class => 'handleUserRegistered',
        ];
    }
}
```

```php
// Registering the subscriber in EventServiceProvider (Laravel 10 and below)
namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    protected $subscribe = [
        UserEventSubscriber::class,
    ];
}
```

```php
// In Laravel 11+, subscribers are auto-discovered
// from the app/Listeners directory
```

**Component Breakdown:**

- `subscribe(Dispatcher $events)` — The method that registers listeners. Returns an array of event-to-method mappings.
- `handleUserLogin(Login $event)` — Handler method for the `Login` event.
- `handleUserLogout(Logout $event)` — Handler method for the `Logout` event.
- `handleUserRegistered(Registered $event)` — Handler method for the `Registered` event.
- `$subscribe` — Property on `EventServiceProvider` listing subscriber classes (Laravel 10-).

**Syntax Rules:**

- The subscriber must define a `subscribe()` method.
- The `subscribe()` method may return an array mapping event classes to method names, or call `$events->listen()` directly.
- Subscriber methods are regular methods that receive the event instance as their first parameter.
- Subscribers are registered in `EventServiceProvider::$subscribe` (Laravel 10-) or auto-discovered (Laravel 11+).
- Subscriber methods are not type-hinted for auto-discovery; they are registered explicitly via the `subscribe()` method.

**Constraints and Limitations:**

- **Subscribers do not support automatic type-hinted discovery.** They must be registered explicitly.
- **Subscriber methods must be defined on the subscriber class.** They cannot be closures or external callables.
- **Subscribers are resolved from the container.** Constructor injection works as expected.
- **Event caching includes subscribers.** Run `event:cache` after modifying subscribers.

### Annotated Code Examples

**Example 1: UserEventSubscriber with Multiple Handlers**

```php
<?php
// File: app/Listeners/UserEventSubscriber.php

namespace App\Listeners;

use App\Services\AuditLogger;
use App\Services\NotificationService;
use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Auth\Events\Registered;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    public function __construct(
        private AuditLogger $auditLogger,
        private NotificationService $notificationService,
    ) {}

    public function handleUserLogin(Login $event): void
    {
        $this->auditLogger->log('user.login', $event->user->id);
        $event->user->update(['last_login_at' => now()]);
    }

    public function handleUserLogout(Logout $event): void
    {
        $this->auditLogger->log('user.logout', $event->user->id);
    }

    public function handleUserRegistered(Registered $event): void
    {
        $this->auditLogger->log('user.registered', $event->user->id);
        $this->notificationService->sendWelcomeEmail($event->user);
    }

    /**
     * Register the listeners for the subscriber.
     *
     * @return array<string, string>
     */
    public function subscribe(Dispatcher $events): array
    {
        return [
            Login::class      => 'handleUserLogin',
            Logout::class     => 'handleUserLogout',
            Registered::class => 'handleUserRegistered',
        ];
    }
}
```

**Expected Output:** When a user logs in, logs out, or registers, the corresponding method on the subscriber is invoked. The audit logger and notification service are injected via the constructor and shared across all handler methods.

**Why This Output Occurs:** The `subscribe()` method returns an array mapping event classes to method names. Laravel registers these mappings with the `Dispatcher`. When an event is dispatched, the corresponding subscriber method is invoked. The subscriber is resolved from the container, so constructor dependencies are injected automatically.

---

**Example 2: Orchestrating a Complex Workflow with a Subscriber**

```php
<?php
// File: app/Listeners/OrderWorkflowSubscriber.php

namespace App\Listeners;

use App\Domain\Orders\Events\OrderPlaced;
use App\Domain\Orders\Events\OrderPaid;
use App\Domain\Orders\Events\OrderShipped;
use App\Domain\Orders\Events\OrderDelivered;
use App\Services\InventoryService;
use App\Services\NotificationService;
use App\Services\ShippingService;
use Illuminate\Events\Dispatcher;

class OrderWorkflowSubscriber
{
    public function __construct(
        private InventoryService $inventory,
        private NotificationService $notifications,
        private ShippingService $shipping,
    ) {}

    public function handleOrderPlaced(OrderPlaced $event): void
    {
        $this->inventory->reserve($event->order);
        $this->notifications->sendOrderConfirmation($event->order);
    }

    public function handleOrderPaid(OrderPaid $event): void
    {
        $this->notifications->sendPaymentReceipt($event->order);
        $this->shipping->schedulePickup($event->order);
    }

    public function handleOrderShipped(OrderShipped $event): void
    {
        $this->notifications->sendShippingNotification($event->order);
    }

    public function handleOrderDelivered(OrderDelivered $event): void
    {
        $this->inventory->commit($event->order);
        $this->notifications->sendDeliveryConfirmation($event->order);
    }

    public function subscribe(Dispatcher $events): array
    {
        return [
            OrderPlaced::class    => 'handleOrderPlaced',
            OrderPaid::class      => 'handleOrderPaid',
            OrderShipped::class   => 'handleOrderShipped',
            OrderDelivered::class => 'handleOrderDelivered',
        ];
    }
}
```

**Expected Output:** The subscriber orchestrates the entire order workflow, reacting to each stage of the order lifecycle with the appropriate inventory, notification, and shipping actions.

**Why This Output Occurs:** The subscriber groups all order-related event handlers into a single class, sharing dependencies (InventoryService, NotificationService, ShippingService) across handlers. The `subscribe()` method registers all four event-handler mappings. When any of the four events is dispatched, the corresponding handler is invoked, keeping the order workflow logic cohesive and centralised.

### Real-World Cases

- **User lifecycle management:** A `UserEventSubscriber` handles login, logout, registration, email verification, and password reset events.
- **Order workflow orchestration:** An `OrderWorkflowSubscriber` handles the entire order lifecycle from placement to delivery.
- **Payment processing pipeline:** A `PaymentEventSubscriber` handles payment initiation, success, failure, refund, and dispute events.
- **Content moderation workflow:** A `ContentModerationSubscriber` handles submission, flagging, approval, rejection, and appeal events.
- **System monitoring orchestration:** A `SystemEventSubscriber` handles alert creation, acknowledgment, escalation, and resolution events.

---

## References

- Laravel Events Documentation (Master) — https://laravel.com/framework/docs/master/events
- Laravel Events Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/events
- Laravel Events Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/events
- Laravel Events Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/events
- Laravel Queues Documentation (Master) — https://laravel.com/framework/docs/master/queues
- Laravel `Illuminate\Contracts\Events\ShouldDispatchAfterCommit` API — https://api.laravel.com/docs/master/Illuminate/Contracts/Events/ShouldDispatchAfterCommit.html
- Laravel `Illuminate\Queue\Middleware\ThrottlesExceptions` API — https://api.laravel.com/docs/10.x/Illuminate/Queue/Middleware/ThrottlesExceptions.html
- Laravel `Illuminate\Queue\Middleware\RateLimited` API — https://api.laravel.com/docs/master/Illuminate/Queue/Middleware/RateLimited.html
- Laravel `Illuminate\Events\Dispatcher` API — https://api.laravel.com/docs/master/Illuminate/Events/Dispatcher.html
- Laravel `Illuminate\Foundation\Events\DiscoverEvents` API — https://api.laravel.com/docs/master/Illuminate/Foundation/Events/DiscoverEvents.html
- Laravel `ShouldQueueAfterCommit` Interface — https://www.bookstack.cn/read/laravel-11.x-en/events.md
- Laravel `after_commit` Queue Configuration — https://laravel.com/framework/docs/master/queues#jobs-and-database-transactions
- Dispatch Events after a DB Transaction in Laravel 10.30 (Laravel News) — https://laravel-news.com/dispatch-events-after-db-transaction
- Laravel Transaction Guard (Packagist) — https://packagist.org/packages/codegenie-be/laravel-transaction-guard
- Laravel Transactional Events (Packagist) — https://packagist.org/packages/fntneves/laravel-transactional-events
- DDD Domain Events Reference (GitHub) — https://github.com/wondelai/skills/blob/main/domain-driven-design/references/domain-events.md
- Domain-Driven Design in PHP (O'Reilly) — https://www.oreilly.com/library/view/domain-driven-design-in/9781787284944/
- Laravel Event Subscribers (GitHub) — https://github.com/laravel/docs/blob/12.x/events.md
- Laravel `EventServiceProvider` API — https://api.laravel.com/docs/10.x/Illuminate/Foundation/Support/Providers/EventServiceProvider.html
- Laravel Clean Architecture with Domain Events (Packagist) — https://packagist.org/packages/elber/laravel-clean-architecture
- Laravel Boost DDD (GitHub) — https://github.com/maiobarbero/laravel-boost-ddd