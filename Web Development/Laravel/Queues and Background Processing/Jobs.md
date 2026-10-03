# Laravel Advanced Job Construction — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced Job Construction is the set of structural patterns, interfaces, and dispatch mechanisms in Laravel's queue system that enable developers to build robust, deduplicated, time-shifted, sequentially orchestrated, and massively parallel background workloads.

**Technical Definition:** Advanced job construction is built on Laravel's `Illuminate\Contracts\Queue\ShouldQueue` interface and the `Illuminate\Bus\Dispatcher`. Jobs are PHP classes implementing `ShouldQueue` that are serialized and pushed onto a queue backend via the `Illuminate\Bus\Queueable` trait. Advanced features include: the `ShouldBeUnique` interface (implemented via the `UniqueLock` class using atomic cache locks), the `delay()` method on the `PendingDispatch` instance, the `Bus::chain()` method (which serializes a sequence of jobs as a chain using the `ChainedBatch` class), and the `Bus::batch()` method (which creates a `PendingBatch` that tracks job completion via the `job_batches` table and the `Batchable` trait). The `SerializesModels` trait handles Eloquent model auto-serialization using `ModelIdentifier` objects.

**Beginner-Friendly Explanation:** Basic queueing just pushes a job to the background. Advanced job construction gives you more control: you can make sure the same job never runs twice at the same time (unique jobs), delay a job until a specific time (delayed jobs), run jobs in a specific order where one failure stops the rest (chaining), or run hundreds of jobs in parallel and get notified when they all finish (batching). These tools let you build complex, production-grade background processing pipelines.

### Key Characteristics

- **Deduplication:** `ShouldBeUnique` prevents duplicate job processing using atomic cache locks.
- **Time-shifting:** `delay()` defers job execution to a specific time or offset.
- **Sequential orchestration:** `Bus::chain()` runs jobs in order, halting on failure.
- **Mass parallel processing:** `Bus::batch()` runs many jobs in parallel with unified lifecycle callbacks.
- **Model auto-serialization:** `SerializesModels` stores model identifiers, not full models.
- **Failure isolation:** Chains and batches provide dedicated failure handling via `catch()` callbacks.
- **Transaction awareness:** Jobs can be deferred until database transactions commit.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A configured queue connection (Redis recommended for production).
- For unique jobs: a cache driver that supports atomic locks (Redis, Memcached, database).
- For batching: the `job_batches` table migration (`php artisan make:queue-batches-table && php artisan migrate`).
- Basic understanding of Laravel queues (see the Queue Fundamentals cheat sheet).

### Related Programming Areas

- **Queue System** — The foundation for all advanced job construction.
- **Cache System** — Used for atomic locks in unique jobs.
- **Database** — The `job_batches` table stores batch metadata.
- **Events & Listeners** — Batch lifecycle callbacks are similar to event listeners.
- **Service Container** — Jobs are resolved from the container.
- **Horizon** — Provides monitoring and auto-scaling for advanced job patterns.

### Core Concepts / Features

1. Job Classes & Payloads (Building structural jobs using the `ShouldQueue` interface and handling Eloquent model auto-serialization)
2. Unique Jobs (Preventing duplicate processing bottlenecks using `ShouldBeUnique` and custom unique locks/keys)
3. Delayed & Scheduled Jobs (Offloading tasks with exact time-delay offsets via `delay()`)
4. Job Chaining (Orchestrating clean sequential execution lines where step failures halt the remaining chain)
5. Job Batching (Bundling mass parallel jobs together with unified `then()`, `catch()`, and `finally()` lifecycle hooks)

---

## 1. Job Classes & Payloads

### Definitions

**Core Definition:** A job class is a PHP class implementing `ShouldQueue` that encapsulates a unit of background work, with its constructor arguments forming the payload that is serialized and stored in the queue.

**Technical Definition:** Job classes are generated via `php artisan make:job` and extend no base class but implement `Illuminate\Contracts\Queue\ShouldQueue`. They use the `Dispatchable`, `InteractsWithQueue`, `Queueable`, and `SerializesModels` traits. The `SerializesModels` trait implements `__sleep()` and `__wakeup()` magic methods that replace Eloquent model instances with `ModelIdentifier` objects (containing only the model's class name and primary key) during serialization, and re-fetch the models from the database during unserialization. The `withoutRelations()` method on Eloquent models strips loaded relationships before serialization, reducing payload size. The `DeleteWhenMissingModels` attribute automatically discards jobs whose models have been deleted.

**Beginner-Friendly Explanation:** A job class is where you put the code that runs in the background. The constructor receives the data the job needs — like an Eloquent model — and `SerializesModels` handles the tricky part of storing that model in the queue. Instead of storing the entire model (which could become stale), it stores just the model's ID and re-fetches the fresh model when the worker runs the job.

### Purposes

- To encapsulate a unit of background work in a dedicated, testable class.
- To enable Eloquent model auto-serialization via `SerializesModels`, avoiding stale data.
- To reduce payload size by storing only model identifiers instead of full models.
- To automatically discard jobs when their models have been deleted.
- To provide dependency injection of services and repositories into the `handle()` method.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Jobs/ProcessPodcast.php

namespace App\Jobs;

use App\Models\Podcast;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\Attributes\DeleteWhenMissingModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

#[DeleteWhenMissingModels]
class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Podcast $podcast,
    ) {
        // Strip loaded relationships to reduce payload size
        $this->podcast = $podcast->withoutRelations();
    }

    public function handle(PodcastProcessor $processor): void
    {
        // $this->podcast is re-fetched from the database
        $processor->process($this->podcast);
    }
}
```

**Component Breakdown:**

- `ShouldQueue` — Marks the class as a queueable job.
- `Dispatchable` — Provides static `dispatch()` methods.
- `InteractsWithQueue` — Provides queue job methods (`release`, `delete`, `fail`).
- `Queueable` — Provides `onConnection()`, `onQueue()`, `delay()`, `afterCommit()`, `chain()`.
- `SerializesModels` — Replaces Eloquent models with identifiers during serialization.
- `#[DeleteWhenMissingModels]` — Automatically discards the job if the model is deleted.
- `withoutRelations()` — Strips loaded relationships before serialization.

**Syntax Rules:**

- The `SerializesModels` trait must be used if the job contains Eloquent models.
- The `withoutRelations()` method should be called in the constructor to strip relationships.
- The `#[DeleteWhenMissingModels]` attribute is optional but recommended for jobs with model dependencies.
- The `handle()` method can type-hint additional dependencies for method injection.
- Public properties are automatically available in the `handle()` method.

**Constraints and Limitations:**

- **`SerializesModels` re-fetches models from the database.** If the model is deleted, the job fails unless `DeleteWhenMissingModels` is used.
- **Unsaved model changes are lost.** The model state at dispatch time is discarded; only the ID is stored.
- **Large payloads can exceed queue message size limits.** Use `withoutRelations()` to reduce payload size.
- **Binary data must be base64-encoded** before being passed to a queued job.

### Annotated Code Examples

**Example 1: Job with SerializesModels and withoutRelations**

```php
<?php
// File: app/Jobs/ProcessPodcast.php

namespace App\Jobs;

use App\Models\Podcast;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Podcast $podcast,
    ) {
        // Strip loaded relationships to keep the payload small
        $this->podcast = $podcast->withoutRelations();
    }

    public function handle(): void
    {
        // $this->podcast is re-fetched from the database
        // Any unsaved changes made before dispatch are discarded
        $this->podcast->update(['status' => 'processing']);

        // Process the podcast...
        $this->podcast->update(['status' => 'processed']);
    }
}
```

```php
// Dispatching the job
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;

$podcast = Podcast::with('episodes')->find(123);
ProcessPodcast::dispatch($podcast);
// The serialized payload contains only the podcast ID,
// not the full model or its loaded episodes.
```

**Expected Output:** The job is serialized with only the podcast ID. When the worker runs the job, it re-fetches the podcast from the database (without the `episodes` relationship). Any changes made to the podcast between dispatch and execution are reflected.

**Why This Output Occurs:** The `withoutRelations()` method strips the loaded `episodes` relationship from the in-memory model. The `SerializesModels` trait's `__sleep()` method replaces the `Podcast` model with a `ModelIdentifier` containing only the ID and class name. The `__wakeup()` method re-fetches the podcast from the database.

---

**Example 2: Job with DeleteWhenMissingModels**

```php
<?php
// File: app/Jobs/GenerateInvoice.php

namespace App\Jobs;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\Attributes\DeleteWhenMissingModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

#[DeleteWhenMissingModels]
class GenerateInvoice implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function handle(): void
    {
        // Generate the invoice PDF
        $pdf = \PDF::loadView('invoices.order', ['order' => $this->order]);
        $pdf->save(storage_path("invoices/{$this->order->id}.pdf"));
    }
}
```

**Expected Output:** If the order is deleted before the job runs, the job is silently discarded instead of throwing a `ModelNotFoundException`.

**Why This Output Occurs:** The `DeleteWhenMissingModels` attribute tells the `CallQueuedHandler` to catch `ModelNotFoundException` and delete the job without raising an exception. This prevents failed job accumulation for expected deletions.

### Real-World Cases

- **Video processing:** A job processes uploaded videos, re-fetching the video model from the database to access the file path and metadata.
- **Invoice generation:** A job generates PDF invoices, using `DeleteWhenMissingModels` to handle cancelled orders gracefully.
- **Email sending:** A job sends transactional emails, re-fetching the user model to ensure the email is sent to the current address.
- **Data synchronization:** A job syncs customer data to a CRM, using `withoutRelations()` to avoid serializing large relationship graphs.
- **Report generation:** A job generates reports, using `DeleteWhenMissingModels` to handle deleted report requests.

---

## 2. Unique Jobs

### Definitions

**Core Definition:** A unique job is a job that implements the `ShouldBeUnique` interface, ensuring that only one instance of the job with the same unique key exists on the queue at any given time, preventing duplicate processing.

**Technical Definition:** The `ShouldBeUnique` interface is implemented by job classes to signal that duplicate dispatches should be discarded. When a unique job is dispatched, the `Illuminate\Bus\UniqueLock` class attempts to acquire an atomic lock (via the configured cache driver) using a key derived from the job class and either the default unique ID or a custom `uniqueId()` method. If the lock cannot be acquired, the job is not dispatched. The lock is released when the job completes processing or fails all retry attempts. The `uniqueFor` property specifies the lock's TTL in seconds. The `uniqueVia()` method allows specifying a non-default cache driver for the lock. The `ShouldBeUniqueUntilProcessing` interface releases the lock before processing begins, rather than after completion.

**Beginner-Friendly Explanation:** A unique job prevents the same job from being queued more than once at the same time. For example, if you have a job that recalculates a product's search index, you don't want to queue ten of them when the product is updated ten times in a row — you just need one. The `ShouldBeUnique` interface handles this automatically. You can also define a custom "unique key" so that different products get their own unique jobs.

### Purposes

- To prevent duplicate job processing when the same job is dispatched multiple times.
- To reduce unnecessary work by collapsing multiple dispatches into a single execution.
- To prevent race conditions and data corruption caused by concurrent identical jobs.
- To provide fine-grained control over uniqueness scope via `uniqueId()` and `uniqueFor`.
- To allow uniqueness to be released before processing begins via `ShouldBeUniqueUntilProcessing`.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Jobs/UpdateSearchIndex.php

namespace App\Jobs;

use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\SerializesModels;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    use Dispatchable, SerializesModels;

    public Product $product;

    /**
     * The number of seconds after which the unique lock is released.
     */
    public int $uniqueFor = 3600;

    public function __construct(Product $product)
    {
        $this->product = $product->withoutRelations();
    }

    /**
     * The unique ID of the job.
     */
    public function uniqueId(): string
    {
        return (string) $this->product->id;
    }

    public function handle(): void
    {
        // Update the search index for the product...
    }
}
```

**Component Breakdown:**

- `ShouldBeUnique` — Marks the job as unique.
- `$uniqueFor` — Seconds after which the unique lock expires (default: 0, meaning permanent until released).
- `uniqueId()` — Returns the uniqueness key (default: empty string, meaning all instances are unique by class).
- `ShouldBeUniqueUntilProcessing` — Alternative interface that releases the lock before processing.
- `uniqueVia()` — Returns the cache driver for the lock (default: default cache driver).

**Syntax Rules:**

- The `ShouldBeUnique` interface requires a cache driver that supports atomic locks (Redis, Memcached, database, file, array).
- The `uniqueId()` method must return a string.
- The `$uniqueFor` property must be an integer (seconds).
- Unique job constraints do not apply to jobs within batches.
- The lock is released when the job completes or fails all retry attempts.

**Constraints and Limitations:**

- **Unique jobs require a cache driver that supports atomic locks.** The `sync` and `null` queue drivers do not support locks.
- **Unique jobs are not supported within batches.** Batch jobs cannot be unique.
- **If `$uniqueFor` is 0 and the worker crashes, the lock remains forever.** Set a reasonable `$uniqueFor` value.
- **Unique jobs are released after processing by default.** Use `ShouldBeUniqueUntilProcessing` to release before processing.
- **The `uniqueId()` method must return a string.** Integers are cast to strings automatically.

### Annotated Code Examples

**Example 1: Unique Job with Custom uniqueId**

```php
<?php
// File: app/Jobs/RecalculateProductStats.php

namespace App\Jobs;

use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\SerializesModels;

class RecalculateProductStats implements ShouldQueue, ShouldBeUnique
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Product $product,
    ) {
        $this->product = $product->withoutRelations();
    }

    public int $uniqueFor = 300; // 5 minutes

    public function uniqueId(): string
    {
        return 'product-stats-' . $this->product->id;
    }

    public function handle(): void
    {
        // Recalculate sales, views, and ratings for the product...
    }
}
```

```php
// Dispatching multiple times in quick succession
use App\Jobs\RecalculateProductStats;

$product = Product::find(123);

// First dispatch: job is queued
RecalculateProductStats::dispatch($product);

// Second dispatch: ignored because the unique lock is held
RecalculateProductStats::dispatch($product);

// After 5 minutes, the lock expires and a new job can be dispatched
```

**Expected Output:** Only the first dispatch is queued. Subsequent dispatches are silently ignored until the first job completes or the `uniqueFor` TTL expires.

**Why This Output Occurs:** When `RecalculateProductStats::dispatch($product)` is called, the `UniqueLock` class attempts to acquire an atomic lock with the key `laravel_unique_job:App\Jobs\RecalculateProductStats:product-stats-123`. If the lock is already held, the dispatch is discarded. The lock is released when the job completes or after 300 seconds.

---

**Example 2: ShouldBeUniqueUntilProcessing**

```php
<?php
// File: app/Jobs/SendInvoiceEmail.php

namespace App\Jobs;

use App\Models\Invoice;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\SerializesModels;

class SendInvoiceEmail implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Invoice $invoice,
    ) {}

    public function uniqueId(): string
    {
        return (string) $this->invoice->id;
    }

    public function handle(): void
    {
        // Send the invoice email...
    }
}
```

**Expected Output:** The unique lock is released before the job begins processing, allowing another instance to be queued while the first is still running.

**Why This Output Occurs:** The `ShouldBeUniqueUntilProcessing` interface tells the queue system to release the lock immediately before the `handle()` method is invoked. This is useful when the uniqueness constraint is only needed to prevent duplicate queueing, not duplicate execution.

### Real-World Cases

- **Search index updates:** A unique job per product prevents multiple index updates when a product is edited rapidly.
- **Stats recalculation:** A unique job per user prevents duplicate stats calculations when multiple events trigger the same recalculation.
- **Email sending:** A unique job per invoice prevents duplicate emails when the send action is triggered multiple times.
- **Cache warming:** A unique job per cache key prevents redundant cache warming operations.
- **Data export:** A unique job per user prevents duplicate export jobs when the user clicks the export button multiple times.

---

## 3. Delayed & Scheduled Jobs

### Definitions

**Core Definition:** A delayed job is a queued job that is not available for processing until a specified time or offset has passed, allowing tasks to be scheduled for future execution without blocking the current request.

**Technical Definition:** The `delay()` method on the `Illuminate\Bus\Queueable` trait (and the `PendingDispatch` instance) sets the job's `$delay` property, which is serialized into the job payload. The queue backend stores the job with an "available at" timestamp. Workers skip delayed jobs until the timestamp has passed. The `delay()` method accepts an `int` (seconds), a `DateTimeInterface` instance (e.g., a Carbon object), or a `DateInterval`. The `withoutDelay()` method resets the delay to zero. The `delay()` method is available on jobs dispatched via `ProcessPodcast::dispatch($podcast)->delay(now()->addMinutes(10))`.

**Beginner-Friendly Explanation:** A delayed job says "do this, but not right now — wait 10 minutes" or "run this at 3 PM tomorrow." It's useful for sending reminder emails, scheduling follow-ups, or deferring work until a specific time. The job is queued immediately but only becomes available for processing after the delay.

### Purposes

- To schedule tasks for future execution without using a separate cron job.
- To defer work until a specific time or offset has passed.
- To implement retry delays for transient failures.
- To stagger workload spikes by spreading job execution over time.
- To support time-sensitive workflows (reminders, follow-ups, scheduled reports).

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Delay by seconds
ProcessPodcast::dispatch($podcast)->delay(600); // 10 minutes

// Delay by Carbon instance
ProcessPodcast::dispatch($podcast)->delay(now()->addMinutes(10));

// Delay by DateInterval
ProcessPodcast::dispatch($podcast)->delay(now()->diffAsCarbonInterval(now()->addHour()));

// Delay inside a job class (property)
class ProcessPodcast implements ShouldQueue
{
    public $delay = 300; // 5 minutes
}

// Delay with method
ProcessPodcast::dispatch($podcast)->delay(now()->addDays(1));

// Remove delay
ProcessPodcast::dispatch($podcast)->withoutDelay();
```

**Component Breakdown:**

- `delay(int|DateTimeInterface|DateInterval $delay)` — Sets the delay for the job.
- `$delay` property — Sets the delay directly on the job class.
- `withoutDelay()` — Resets the delay to zero.
- `now()->addMinutes(10)` — Carbon method for creating a future `DateTime` instance.

**Syntax Rules:**

- The `delay()` method accepts an integer (seconds), a `DateTimeInterface` (Carbon, DateTime), or a `DateInterval`.
- The `$delay` property on the job class is used when no explicit delay is set at dispatch time.
- The `delay()` method takes precedence over the `$delay` property.
- Delayed jobs are stored in the queue with an "available at" timestamp.
- The `withoutDelay()` method overrides any delay set on the class.

**Constraints and Limitations:**

- **Not all queue drivers support delayed jobs natively.** Redis, database, and SQS support delays; Beanstalkd has limited support.
- **The maximum delay varies by driver.** Redis has no hard limit; SQS supports up to 15 minutes; database supports any duration.
- **Delayed jobs are still "queued" immediately.** They occupy space in the queue backend even though they are not yet available.
- **Delay resolution is limited by the queue driver's precision.** Redis and database are second-precision; SQS is minute-precision.

### Annotated Code Examples

**Example 1: Delayed Job with Carbon Instance**

```php
<?php
// File: app/Http/Controllers/ReminderController.php

namespace App\Http\Controllers;

use App\Jobs\SendAppointmentReminder;
use App\Models\Appointment;
use Illuminate\Http\Request;

class ReminderController extends Controller
{
    public function schedule(Request $request, Appointment $appointment)
    {
        // Send the reminder 24 hours before the appointment
        $delay = $appointment->scheduled_at->subHours(24);

        // If the appointment is less than 24 hours away, send immediately
        if ($delay->isPast()) {
            SendAppointmentReminder::dispatch($appointment);
        } else {
            SendAppointmentReminder::dispatch($appointment)->delay($delay);
        }

        return response()->json(['message' => 'Reminder scheduled.']);
    }
}
```

**Expected Output:** The reminder job is queued and will become available for processing 24 hours before the appointment. If the appointment is less than 24 hours away, the job is dispatched immediately.

**Why This Output Occurs:** The `delay()` method accepts a `DateTimeInterface` instance. The queue backend stores the job with the computed "available at" timestamp. Workers skip the job until the timestamp has passed, at which point it is processed normally.

---

**Example 2: Delay with Conditional Release**

```php
<?php
// File: app/Jobs/ProcessPayment.php

namespace App\Jobs;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPayment implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function handle(): void
    {
        // Attempt to process the payment
        $success = $this->attemptPayment();

        if (!$success) {
            // Release the job back to the queue with a delay
            $this->release(300); // Retry after 5 minutes
        }
    }

    private function attemptPayment(): bool
    {
        // Payment gateway logic...
        return false;
    }
}
```

**Expected Output:** If the payment fails, the job is released back to the queue and retried after 300 seconds (5 minutes).

**Why This Output Occurs:** The `release()` method (available via the `InteractsWithQueue` trait) puts the job back on the queue with a delay. This is useful for retrying transient failures without consuming a retry attempt. The job will be picked up again after the delay.

### Real-World Cases

- **Appointment reminders:** Delaying reminder jobs until 24 hours before the appointment.
- **Abandoned cart emails:** Delaying a follow-up email for 1 hour after cart abandonment.
- **Payment retries:** Releasing a failed payment job with a delay for retry.
- **Scheduled reports:** Delaying a report generation job until the end of the day.
- **Trial expiration notices:** Delaying a notice job until the trial period is about to expire.

---

## 4. Job Chaining

### Definitions

**Core Definition:** Job chaining is the orchestration of a sequence of jobs where each job runs only if the previous job in the chain succeeded, providing a clean, sequential execution line for multi-step workflows.

**Technical Definition:** The `Bus::chain()` method accepts an array of job instances and dispatches them as a chain using the `Illuminate\Bus\ChainedBatch` class. Each job in the chain is dispatched only after the previous job completes successfully. If any job in the chain fails, the remaining jobs are not dispatched, and the chain's `catch()` callback is invoked with the exception. The `Bus::chain()` method returns a `PendingChain` instance that supports `onConnection()`, `onQueue()`, `delay()`, `catch()`, and `dispatch()`. The `prependToChain()` and `appendToChain()` methods allow modifying the chain from within a job.

**Beginner-Friendly Explanation:** A job chain is like a relay race: job A runs, then job B runs, then job C runs. If job A fails, job B and C never run. This is perfect for workflows where each step depends on the previous one — like validating an order, then charging the payment, then sending a confirmation, then updating inventory.

### Purposes

- To orchestrate multi-step workflows where each step depends on the previous one.
- To ensure that a failure at any step halts the remaining steps.
- To provide a single `catch()` callback for handling failures at any point in the chain.
- To allow chain-level configuration (connection, queue, delay).
- To support modifying the chain from within a job via `prependToChain()` and `appendToChain()`.

### Syntax Rules and Structure

#### Complete General Syntax

```php
use App\Jobs\ChargePayment;
use App\Jobs\SendConfirmation;
use App\Jobs\UpdateInventory;
use App\Jobs\ValidateOrder;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ValidateOrder($order),
    new ChargePayment($order),
    new SendConfirmation($order),
    new UpdateInventory($order),
])
    ->onConnection('redis')
    ->onQueue('orders')
    ->catch(function (\Throwable $e) use ($order) {
        // Handle chain failure
        $order->update(['status' => 'failed']);
        \Log::error('Order chain failed', ['order_id' => $order->id]);
    })
    ->dispatch();
```

**Component Breakdown:**

- `Bus::chain([...])` — Creates a chain of jobs to run sequentially.
- `->onConnection()` — Sets the queue connection for all jobs in the chain.
- `->onQueue()` — Sets the queue name for all jobs in the chain.
- `->catch()` — Sets the callback to invoke if any job in the chain fails.
- `->dispatch()` — Dispatches the chain.
- `prependToChain()` / `appendToChain()` — Add jobs to the chain from within a job.

**Syntax Rules:**

- All jobs in a chain must use the `Queueable` trait.
- The `catch()` callback receives the `Throwable` instance that caused the failure.
- If a job in the chain fails, the remaining jobs are not dispatched.
- The `catch()` callback is serialized and executed later; do not use `$this` within it.
- Chained jobs can be delayed as a group using `->delay()` on the `PendingChain`.

**Constraints and Limitations:**

- **If a job in the chain fails, the entire remaining chain is abandoned.** There is no automatic resume.
- **The `catch()` callback is not invoked for manual job releases.** Only actual failures (exceptions) trigger it.
- **Chain jobs cannot be unique.** `ShouldBeUnique` is not supported within chains.
- **The first job in a chain is dispatched immediately.** Subsequent jobs are dispatched as each previous job completes.
- **Chain jobs are serialized as a single payload.** Very long chains can exceed queue message size limits.

### Annotated Code Examples

**Example 1: Complete Order Processing Chain**

```php
<?php
// File: app/Http/Controllers/OrderController.php

namespace App\Http\Controllers;

use App\Jobs\ChargePayment;
use App\Jobs\SendConfirmation;
use App\Jobs\UpdateInventory;
use App\Jobs\ValidateOrder;
use App\Models\Order;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Bus;
use Throwable;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        $order = Order::create($request->validated());

        // Step 1: Build the chain
        $chain = [
            new ValidateOrder($order),
            new ChargePayment($order),
            new SendConfirmation($order),
            new UpdateInventory($order),
        ];

        // Step 2: Dispatch the chain with failure handling
        Bus::chain($chain)
            ->onQueue('orders')
            ->catch(function (Throwable $e) use ($order) {
                $order->update(['status' => 'failed']);
                \Log::error('Order processing chain failed', [
                    'order_id' => $order->id,
                    'error'    => $e->getMessage(),
                ]);
            })
            ->dispatch();

        return response()->json(['order_id' => $order->id], 201);
    }
}
```

**Expected Output:**

- If all jobs succeed: the order is validated, payment is charged, confirmation is sent, and inventory is updated.
- If `ChargePayment` fails: `SendConfirmation` and `UpdateInventory` are not dispatched. The `catch()` callback updates the order status to 'failed' and logs the error.

**Why This Output Occurs:** The `Bus::chain()` method serializes the array of jobs and dispatches the first job. When the first job completes successfully, the next job is dispatched, and so on. If any job throws an exception, the chain's `catch()` callback is invoked, and the remaining jobs are discarded.

---

**Example 2: Modifying a Chain from Within a Job**

```php
<?php
// File: app/Jobs/ValidateOrder.php

namespace App\Jobs;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ValidateOrder implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function handle(): void
    {
        // Validate the order
        if (!$this->order->isValid()) {
            throw new \RuntimeException('Order validation failed.');
        }

        // If the order is a gift, add a gift-wrapping job to the chain
        if ($this->order->is_gift) {
            $this->prependToChain(new \App\Jobs\AddGiftWrapping($this->order));
        }
    }
}
```

**Expected Output:** If the order is valid, the chain continues. If the order is a gift, an `AddGiftWrapping` job is prepended to the chain and runs before the next job.

**Why This Output Occurs:** The `prependToChain()` method (available via the `InteractsWithQueue` trait when the job is part of a chain) adds a job to the beginning of the remaining chain. The chain dispatcher picks up the modified chain and continues execution.

### Real-World Cases

- **E-commerce order processing:** Validate → charge payment → send confirmation → update inventory.
- **User onboarding:** Create account → send welcome email → set up profile → assign trial.
- **Content publishing:** Validate content → index in search → notify subscribers → publish.
- **Payment refunds:** Validate refund → process refund → update ledger → notify customer.
- **Data pipeline:** Extract data → transform → load → generate report.

---

## 5. Job Batching

### Definitions

**Core Definition:** Job batching is the bundling of multiple jobs into a single batch that runs in parallel, with unified `then()`, `catch()`, and `finally()` lifecycle callbacks that execute when the entire batch completes, when any job fails, or when the batch finishes regardless of outcome.

**Technical Definition:** The `Bus::batch()` method accepts an array of jobs and returns a `PendingBatch` instance. The batch is stored in the `job_batches` table (created via `php artisan make:queue-batches-table`), and each job in the batch is dispatched to the queue with a reference to the batch ID. Jobs in a batch must use the `Illuminate\Bus\Batchable` trait, which provides access to the current batch via `$this->batch()`. The batch tracks completion via the `job_batches` table and invokes the `then()`, `catch()`, `progress()`, `before()`, and `finally()` callbacks as the batch progresses. The `allowFailures()` method allows the batch to continue even if some jobs fail. Batched jobs are wrapped in database transactions.

**Beginner-Friendly Explanation:** A batch is like a project with many tasks: you assign all the tasks at once, they run in parallel, and you get notified when the whole project is done (or when something goes wrong). You can also define what happens when everything succeeds, when something fails, and when the batch finishes no matter what.

### Purposes

- To run many jobs in parallel for efficient mass processing.
- To track batch progress and completion via the `job_batches` table.
- To provide unified lifecycle callbacks for batch success, failure, and completion.
- To allow batches to continue even if some jobs fail via `allowFailures()`.
- To enable cancellation of an entire batch via `$batch->cancel()`.

### Syntax Rules and Structure

#### Complete General Syntax

```php
use App\Jobs\ImportCsv;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch([
    new ImportCsv(1, 1000),
    new ImportCsv(1001, 2000),
    new ImportCsv(2001, 3000),
    new ImportCsv(3001, 4000),
])
    ->before(function (Batch $batch) {
        // The batch has been created but no jobs have been added
    })
    ->progress(function (Batch $batch) {
        // A single job has completed successfully
    })
    ->then(function (Batch $batch) {
        // All jobs completed successfully
    })
    ->catch(function (Batch $batch, Throwable $e) {
        // The first batch job failure detected
    })
    ->finally(function (Batch $batch) {
        // The batch has finished executing
    })
    ->name('Import CSV')
    ->onConnection('redis')
    ->onQueue('imports')
    ->allowFailures()
    ->dispatch();

return $batch->id;
```

**Component Breakdown:**

- `Bus::batch([...])` — Creates a batch of jobs to run in parallel.
- `->before()` — Runs when the batch is created but before any jobs are added.
- `->progress()` — Runs each time a job in the batch completes successfully.
- `->then()` — Runs when all jobs in the batch complete successfully.
- `->catch()` — Runs when the first job in the batch fails.
- `->finally()` — Runs when the batch finishes (success or failure).
- `->name()` — Assigns a human-readable name to the batch.
- `->onConnection()` / `->onQueue()` — Sets the connection and queue for all jobs.
- `->allowFailures()` — Allows the batch to continue if some jobs fail.

**Syntax Rules:**

- Jobs in a batch must use the `Illuminate\Bus\Batchable` trait.
- The `job_batches` table migration must be run before using batches.
- Batch callbacks are serialized and executed later; do not use `$this` within them.
- Batched jobs are wrapped in database transactions; avoid statements that trigger implicit commits.
- The batch ID (`$batch->id`) can be used to query batch status via the `Bus::findBatch()` method.

**Constraints and Limitations:**

- **Batch callbacks are serialized and executed later.** They cannot access `$this`.
- **Batched jobs are wrapped in database transactions.** Avoid DDL statements (CREATE, ALTER, DROP) within batched jobs.
- **Unique jobs are not supported within batches.** `ShouldBeUnique` is ignored for batched jobs.
- **The `finally()` callback is not guaranteed to run** if the worker crashes or the batch is cancelled abruptly.
- **Batch jobs must use the `Batchable` trait.** Without it, `$this->batch()` will return `null`.

### Annotated Code Examples

**Example 1: Complete Batch with Lifecycle Callbacks**

```php
<?php
// File: app/Jobs/ImportCsvChunk.php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ImportCsvChunk implements ShouldQueue
{
    use Batchable, Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public int $startRow,
        public int $endRow,
    ) {}

    public function handle(): void
    {
        // Check if the batch has been cancelled
        if ($this->batch()->cancelled()) {
            return;
        }

        // Import a chunk of the CSV file...
    }
}
```

```php
// Dispatching the batch
use App\Jobs\ImportCsvChunk;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch([
    new ImportCsvChunk(1, 1000),
    new ImportCsvChunk(1001, 2000),
    new ImportCsvChunk(2001, 3000),
    new ImportCsvChunk(3001, 4000),
])
    ->before(function (Batch $batch) {
        \Log::info('Batch created', ['batch_id' => $batch->id]);
    })
    ->progress(function (Batch $batch) {
        \Log::info('Job completed', [
            'batch_id' => $batch->id,
            'progress' => $batch->progress() . '%',
        ]);
    })
    ->then(function (Batch $batch) {
        \Log::info('All jobs completed successfully');
        \Mail::to('admin@example.com')->send(new ImportComplete($batch));
    })
    ->catch(function (Batch $batch, Throwable $e) {
        \Log::error('Batch job failure detected', [
            'batch_id' => $batch->id,
            'error'    => $e->getMessage(),
        ]);
    })
    ->finally(function (Batch $batch) {
        \Log::info('Batch finished executing', [
            'batch_id'    => $batch->id,
            'total_jobs'  => $batch->totalJobs,
            'failed_jobs' => $batch->failedJobs,
        ]);
    })
    ->name('CSV Import')
    ->allowFailures()
    ->dispatch();

return $batch->id;
```

**Expected Output:**

- The batch is created with four jobs.
- As each job completes, the `progress()` callback logs the progress percentage.
- If any job fails, the `catch()` callback logs the error, but the remaining jobs continue (due to `allowFailures()`).
- When all jobs finish, the `finally()` callback logs the batch summary.
- If all jobs succeed, the `then()` callback sends an email.

**Why This Output Occurs:** The `Bus::batch()` method creates a batch record in the `job_batches` table and dispatches each job with a reference to the batch ID. As jobs complete, the batch's progress is updated, and the appropriate callbacks are invoked. The `allowFailures()` method tells the batch to continue even if some jobs fail, and the `finally()` callback runs regardless of outcome.

---

**Example 2: Inspecting and Cancelling a Batch**

```php
use Illuminate\Support\Facades\Bus;

// Find a batch by ID
$batch = Bus::findBatch($batchId);

if ($batch) {
    echo "Name: {$batch->name}\n";
    echo "Total jobs: {$batch->totalJobs}\n";
    echo "Processed: {$batch->processedJobs()}\n";
    echo "Failed: {$batch->failedJobs}\n";
    echo "Progress: {$batch->progress()}%\n";
    echo "Finished: " . ($batch->finished() ? 'Yes' : 'No') . "\n";
    echo "Cancelled: " . ($batch->cancelled() ? 'Yes' : 'No') . "\n";

    // Cancel the batch
    $batch->cancel();
}

// Clean up old batch records
Bus::pruneBatches();
```

**Expected Output:** The batch details are displayed. Calling `cancel()` marks the batch as cancelled, and any remaining jobs check `$this->batch()->cancelled()` and return early.

**Why This Output Occurs:** The `Batch` instance provides methods for inspecting the batch's state: `totalJobs`, `processedJobs()`, `failedJobs`, `progress()`, `finished()`, and `cancelled()`. The `cancel()` method sets the cancelled flag, which batched jobs check via `$this->batch()->cancelled()`.

### Real-World Cases

- **CSV imports:** Splitting a large CSV file into chunks and processing them in parallel.
- **Mass email sending:** Sending newsletters to thousands of recipients in parallel batches.
- **Image processing:** Resizing hundreds of product images in parallel.
- **Data synchronization:** Syncing records from an external API in parallel batches.
- **Report generation:** Generating reports for multiple departments in parallel.

---

## References

- Laravel Queues Documentation (Master) — https://laravel.com/framework/docs/master/queues
- Laravel Queues Documentation (Laravel 13.x) — https://laravel.com/framework/docs/queues
- Laravel Queues Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/queues
- Laravel `Illuminate\Contracts\Queue\ShouldBeUnique` API — https://api.laravel.com/docs/master/Illuminate/Contracts/Queue/ShouldBeUnique.html
- Laravel `Illuminate\Bus\UniqueLock` API — https://api.laravel.com/docs/master/Illuminate/Bus/UniqueLock.html
- Laravel `Illuminate\Bus\Batchable` API — https://api.laravel.com/docs/master/Illuminate/Bus/Batchable.html
- Laravel `Illuminate\Bus\Batch` API — https://api.laravel.com/docs/master/Illuminate/Bus/Batch.html
- Laravel `Illuminate\Bus\PendingBatch` API — https://api.laravel.com/docs/master/Illuminate/Bus/PendingBatch.html
- Laravel `Illuminate\Bus\ChainedBatch` API — https://api.laravel.com/docs/master/Illuminate/Bus/ChainedBatch.html
- Laravel `Illuminate\Queue\SerializesModels` API — https://api.laravel.com/docs/master/Illuminate/Queue/SerializesModels.html
- Laravel Queue Patterns (GitHub) — https://github.com/NeverSight/skills_feed/blob/main/data/skills-md/iserter/laravel-claude-agents/laravel-queue-patterns/SKILL.md
- Laravel `ShouldBeUniqueUntilProcessing` Interface — https://laravel.com/docs/master/queues#keeping-jobs-unique-until-processing-begins
- Laravel `DeleteWhenMissingModels` Attribute — https://laravel.com/docs/master/queues#ignoring-missing-models
- Laravel Batch Callbacks Documentation — https://laravel.com/docs/master/queues#batch-callbacks
- Laravel Chained Jobs Documentation — https://laravel.com/docs/master/queues#job-chaining
- Laravel Delayed Dispatching Documentation — https://laravel.com/docs/master/queues#delayed-dispatching
- BoringO11y Unique Job Middleware — https://packagist.org/packages/boring-o11y/unique-job-middleware
- Laravel Queue Tips for Reliable Background Jobs — https://laravelmagazine.com/laravel-queue-tips-for-reliable-background-jobs