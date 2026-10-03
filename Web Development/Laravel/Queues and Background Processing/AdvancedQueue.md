# Laravel Advanced Queue Engineering & Resiliency — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced Queue Engineering & Resiliency is the discipline of designing, hardening, and operating Laravel's queue system to guarantee reliable job execution under failure conditions, concurrent workloads, and production-scale traffic, through middleware, rate limiting, fault-tolerant retry strategies, dead-letter queue management, idempotent job design, and real-time observability.

**Technical Definition:** Advanced queue engineering is built on Laravel's `Illuminate\Queue\` namespace, encompassing the `Worker` class (job execution), the `QueueManager` (connection resolution), and the `CallQueuedHandler` (job deserialization and dispatch). Resiliency mechanisms include: job middleware (`Illuminate\Queue\Middleware\`) for cross-cutting concerns; `RateLimited`, `RateLimitedWithRedis`, `WithoutOverlapping`, and `ThrottlesExceptions` middleware for throughput control; `$tries`, `$backoff`, `$timeout`, and `retryUntil()` for fault tolerance; the `failed_jobs` table and `queue:retry` / `queue:forget` / `queue:flush` commands for dead-letter queue handling; idempotency keys and atomic locks for state safety; and Laravel Horizon for Redis-backed queue monitoring, metrics collection, and auto-scaling.

**Beginner-Friendly Explanation:** Basic queues push jobs to the background. Advanced queue engineering makes sure those jobs run reliably — even when external APIs are down, when the same job is accidentally dispatched twice, or when a worker crashes mid-job. It's the difference between "I hope this email gets sent" and "this email will be sent exactly once, retried intelligently if it fails, and I'll be alerted if it never succeeds."

### Key Characteristics

- **Middleware-driven cross-cutting concerns:** Rate limiting, concurrency control, and exception throttling are applied as reusable job middleware, not inline in `handle()` methods.
- **Time-bucket rate limiting:** `RateLimited` and `RateLimitedWithRedis` enforce request-per-time-window limits on external API calls.
- **Concurrency limiting:** `WithoutOverlapping` uses atomic locks to prevent simultaneous execution of the same job, with configurable release and expiration behaviour.
- **Intelligent retry curves:** `$backoff` arrays define progressive delays (e.g., `[1, 5, 30, 120, 600]`) that let external services "breathe" between retries.
- **Dead-letter queue management:** Failed jobs are persisted in the `failed_jobs` table with full context (connection, queue, payload, exception) for inspection and targeted retry.
- **Idempotency guarantees:** Jobs are designed to run safely multiple times without corrupting state, using idempotency keys, status checks, and atomic transactions.
- **Real-time observability:** Laravel Horizon provides a dashboard with job throughput, runtime, failure metrics, and per-queue wait times for Redis queues.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A configured queue connection (Redis strongly recommended for advanced rate limiting and Horizon).
- A cache driver that supports atomic locks (Redis, Memcached, DynamoDB, database, file, array).
- The `failed_jobs` table migration (`php artisan queue:failed-table && php artisan migrate`).
- For Horizon: the `laravel/horizon` package installed and configured (`composer require laravel/horizon && php artisan horizon:install`).
- For Redis-based rate limiting: the `predis/predis` package or the `phpredis` extension.

### Related Programming Areas

- **Queue System** — The foundation for dispatching and processing jobs.
- **Cache System** — Atomic locks used by `WithoutOverlapping` and `ShouldBeUnique`.
- **Database** — The `failed_jobs` and `job_batches` tables persist job state.
- **Horizon** — Provides monitoring, metrics, and auto-scaling for Redis queues.
- **Events & Listeners** — Job middleware can also be applied to queued event listeners, mailables, and notifications.
- **Deployment** — `queue:restart` and `horizon:terminate` ensure workers pick up new code.

### Core Concepts / Features

1. Job Middleware (Extracting repetitive logic such as rate limits or locking rules into reusable job-level filters)
2. Rate Limiting & Concurrency Control (Restricting execution rates using time-bucket limits or limiting maximum simultaneous jobs via Redis)
3. Fault Tolerance & Backoffs (Hardening jobs using custom try limits, explicit timeouts, and intelligent exponential backoff curves)
4. Dead-Letter Queue Handling (Intercepting hard exceptions, inspecting failed records, and executing targeted retries via `queue:retry`)
5. Idempotency & State Safety (Structuring application logic to ensure repeated background runs never corrupt database records)
6. Enterprise Observability (Monitoring performance, throughput, and error states in real-time using Laravel Horizon)

---

## 1. Job Middleware

### Definitions

**Core Definition:** Job middleware is a mechanism for wrapping custom logic around the execution of queued jobs, allowing cross-cutting concerns — such as rate limiting, locking, and exception throttling — to be extracted into reusable, composable filters that run before (and sometimes after) the job's `handle()` method.

**Technical Definition:** Job middleware are classes that implement a `handle($job, $next)` method, where `$job` is the job instance and `$next` is a closure that continues processing. They are attached to a job by returning them from the job's `middleware()` method, which returns an array of middleware instances. Laravel does not have a default location for job middleware; developers typically place them in `app/Jobs/Middleware`. Job middleware are resolved from the service container, allowing dependency injection. They can also be assigned to queueable event listeners, mailables, and notifications. Laravel includes built-in middleware: `RateLimited`, `RateLimitedWithRedis`, `WithoutOverlapping`, `ThrottlesExceptions`, `ThrottlesExceptionsWithRedis`, and `Skip`.

**Beginner-Friendly Explanation:** Job middleware is like HTTP middleware but for queued jobs. Instead of cluttering your `handle()` method with rate-limiting logic, locking checks, and exception handling, you extract that logic into separate middleware classes that wrap around the job. You attach them by returning them from a `middleware()` method on the job. The middleware can decide whether to continue processing the job, release it back to the queue, or skip it entirely.

### Purposes

- To extract repetitive cross-cutting logic (rate limiting, locking, throttling) out of job `handle()` methods.
- To apply consistent behaviour across multiple job classes without code duplication.
- To enable conditional job execution (skipping jobs based on runtime conditions).
- To provide a composable, testable mechanism for job-level concerns.
- To wrap job execution with before/after hooks for logging, monitoring, or state management.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Jobs/ProcessOrder.php

namespace App\Jobs;

use App\Jobs\Middleware\RateLimited;
use App\Jobs\Middleware\WithoutOverlapping;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessOrder implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    /**
     * Get the middleware the job should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [
            new RateLimited('orders'),
            new WithoutOverlapping($this->order->id),
        ];
    }

    public function handle(): void
    {
        // Job logic...
    }
}
```

```php
<?php
// File: app/Jobs/Middleware/CustomMiddleware.php

namespace App\Jobs\Middleware;

use Closure;

class CustomMiddleware
{
    /**
     * Process the queued job.
     *
     * @param  mixed  $job
     * @param  Closure  $next
     * @return mixed
     */
    public function handle(object $job, Closure $next): mixed
    {
        // Before job logic...
        $result = $next($job);
        // After job logic...
        return $result;
    }
}
```

**Component Breakdown:**

- `middleware()` — Method on the job class that returns an array of middleware instances.
- `handle($job, $next)` — Method on the middleware class. `$job` is the job instance, `$next` is the closure to continue processing.
- `$next($job)` — Invokes the next middleware in the chain (or the job's `handle()` method).
- Middleware are resolved from the container; constructor dependencies are injected.

**Syntax Rules:**

- The `middleware()` method must return an array of middleware instances.
- Each middleware must have a `handle` method accepting `$job` and `$next`.
- The `$next` closure must be invoked to continue processing; failing to do so stops the chain.
- Middleware can release the job (`$job->release()`) or delete it (`$job->delete()`) instead of continuing.
- Built-in middleware (`RateLimited`, `WithoutOverlapping`, `ThrottlesExceptions`, `Skip`) are provided by Laravel.

**Constraints and Limitations:**

- **Middleware are resolved from the container per job execution.** They cannot hold state between jobs.
- **The `$next` closure must be called for the job to execute.** If the middleware returns without calling `$next`, the job is silently skipped.
- **Middleware order matters.** Rate limiting should typically come before locking; exception throttling should come last.
- **`WithoutOverlapping` requires a cache driver that supports atomic locks.** The `array` driver works in-process but not across processes.

### Annotated Code Examples

**Example 1: Job with RateLimited and WithoutOverlapping Middleware**

```php
<?php
// File: app/Jobs/SyncCustomerToCrm.php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\Middleware\RateLimited;
use Illuminate\Queue\Middleware\WithoutOverlapping;
use Illuminate\Queue\SerializesModels;

class SyncCustomerToCrm implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public int $customerId,
    ) {}

    public function middleware(): array
    {
        return [
            // Rate limit: max 30 CRM syncs per minute
            new RateLimited('crm-sync'),

            // Prevent overlapping syncs for the same customer
            (new WithoutOverlapping("crm-sync-{$this->customerId}"))
                ->releaseAfter(60)   // Release if lock is held
                ->expireAfter(180),  // Lock expires after 3 minutes
        ];
    }

    public function handle(): void
    {
        // Sync customer data to CRM...
    }
}
```

```php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        RateLimiter::for('crm-sync', function ($job) {
            return Limit::perMinute(30)->by($job->customerId);
        });
    }
}
```

**Expected Output:** The job is rate-limited to 30 executions per minute per customer. If a sync is already running for the same customer, the new job is released back to the queue for 60 seconds. If the lock is held longer than 180 seconds (e.g., due to a worker crash), it expires automatically.

**Why This Output Occurs:** The `RateLimited` middleware checks the named rate limiter before executing the job. If the limit is exceeded, the job is released back to the queue. The `WithoutOverlapping` middleware attempts to acquire an atomic lock using the job's unique key; if the lock is already held, the job is released after `releaseAfter` seconds. The `expireAfter` option ensures the lock is not held indefinitely if the job crashes.

---

**Example 2: Conditional Job Skipping with Skip Middleware**

```php
<?php
// File: app/Jobs/SendPromotionalEmail.php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\Middleware\Skip;
use Illuminate\Queue\SerializesModels;

class SendPromotionalEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public User $user,
        public string $campaign,
    ) {}

    public function middleware(): array
    {
        return [
            // Skip if the user has unsubscribed
            Skip::when(fn () => $this->user->unsubscribed_at !== null),

            // Skip unless the user has opted in to marketing
            Skip::unless(fn () => $this->user->marketing_opt_in),
        ];
    }

    public function handle(): void
    {
        // Send promotional email...
    }
}
```

**Expected Output:** The job is silently skipped (not executed, not failed) if the user has unsubscribed or has not opted in to marketing emails.

**Why This Output Occurs:** The `Skip::when()` middleware skips the job if the closure returns `true`. The `Skip::unless()` middleware skips the job if the closure returns `false`. Skipped jobs are deleted from the queue without executing their `handle()` method or recording a failure.

### Real-World Cases

- **CRM synchronization:** Rate-limited to 30 syncs per minute per customer, with overlapping prevention to avoid duplicate syncs.
- **Payment processing:** `WithoutOverlapping` ensures only one payment job runs per order at a time.
- **Email sending:** `RateLimitedWithRedis` enforces provider-specific rate limits (e.g., 10 emails per second for SES).
- **Third-party API calls:** `ThrottlesExceptions` backs off after repeated failures, preventing retry storms.
- **Marketing campaigns:** `Skip` middleware prevents sending promotional emails to unsubscribed users.

---

## 2. Rate Limiting & Concurrency Control

### Definitions

**Core Definition:** Rate limiting restricts the number of jobs that can be executed within a given time window (time-bucket limiting), while concurrency control restricts the number of jobs that can be executed simultaneously (in-flight limiting). Together, they protect external APIs and shared resources from being overwhelmed by queued work.

**Technical Definition:** Time-bucket rate limiting is implemented via the `RateLimited` and `RateLimitedWithRedis` middleware, which use Laravel's `RateLimiter` facade (backed by atomic cache operations) to enforce a maximum number of job executions per time window. Concurrency limiting is implemented via the `WithoutOverlapping` middleware, which uses atomic locks (via the cache driver) to ensure that only one instance of a job (or a job with a shared key) runs at a time. The `Redis::funnel()` method provides a Redis-native concurrency limiter. The distinction is critical: `RateLimited` bounds how many jobs start in a time window; `WithoutOverlapping` bounds how many jobs are in flight simultaneously.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer at a club who lets in 10 people per minute — it controls the flow. Concurrency control is like a bouncer who lets in only 1 person at a time — it controls how many are inside. Rate limiting protects against exceeding API quotas; concurrency control protects against overwhelming a resource with simultaneous requests.

### Purposes

- To enforce external API rate limits (e.g., 10 requests per second for AWS SES) without exceeding quotas.
- To prevent overwhelming third-party services with concurrent requests.
- To protect shared resources (databases, file systems) from contention.
- To ensure fair usage across tenants in multi-tenant applications.
- To provide a throttling mechanism that releases excess jobs back to the queue rather than failing them.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Rate limiting: time-bucket limit
use Illuminate\Queue\Middleware\RateLimited;
use Illuminate\Queue\Middleware\RateLimitedWithRedis;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

// Define the rate limiter
RateLimiter::for('external-api', function ($job) {
    return Limit::perSecond(10)->by($job->tenantId);
});

// Apply to the job
public function middleware(): array
{
    return [new RateLimited('external-api')];
    // Or for Redis-optimised:
    // return [new RateLimitedWithRedis('external-api')];
}
```

```php
// Concurrency control: without overlapping
use Illuminate\Queue\Middleware\WithoutOverlapping;

public function middleware(): array
{
    return [
        (new WithoutOverlapping("process-order-{$this->order->id}"))
            ->releaseAfter(30)    // Release if locked
            ->expireAfter(120)    // Lock expires after 2 minutes
            ->dontRelease(),      // Or: delete instead of release
    ];
}
```

```php
// Redis-native concurrency limiter
use Illuminate\Support\Facades\Redis;

public function handle(): void
{
    Redis::funnel('process-order-' . $this->order->id)
        ->limit(1)
        ->then(function () {
            // Job logic...
        }, function () {
            return $this->release(10);
        });
}
```

**Component Breakdown:**

- `RateLimited($limiterName)` — Middleware that enforces the named rate limiter.
- `RateLimitedWithRedis($limiterName)` — Redis-optimised rate limiting middleware.
- `RateLimiter::for($name, $callback)` — Defines a rate limiter with a name and a callback returning a `Limit`.
- `Limit::perMinute($max)->by($key)` — Creates a limit of `$max` per minute, keyed by `$key`.
- `WithoutOverlapping($key)` — Middleware that prevents overlapping jobs with the same key.
- `->releaseAfter($seconds)` — Seconds to release the job if the lock is held.
- `->expireAfter($seconds)` — Seconds after which the lock expires automatically.
- `->dontRelease()` — Delete the job instead of releasing it.
- `->shared()` — Share the lock key across job classes.
- `Redis::funnel($key)->limit($max)` — Redis-native concurrency limiter.

**Syntax Rules:**

- The rate limiter must be defined in a service provider's `boot()` method.
- The limiter name passed to `RateLimited` must match the name defined in `RateLimiter::for()`.
- `WithoutOverlapping` requires a cache driver that supports atomic locks.
- By default, `WithoutOverlapping` only prevents overlapping jobs of the same class; use `->shared()` to share the lock across classes.
- The `RateLimitedWithRedis` middleware requires Redis and is more efficient than the basic rate limiter.

**Constraints and Limitations:**

- **`RateLimited` releases jobs back to the queue**, which can cause them to be retried indefinitely if the limit is never available. Use `ThrottlesExceptions` or a maximum retry limit.
- **`WithoutOverlapping` locks are not released if the worker crashes.** The `expireAfter` option prevents permanent locks.
- **Concurrency limiting with `WithoutOverlapping` is per-key, not global.** Use a shared key for global concurrency limits.
- **Redis funnels require Redis.** They are not available for other cache drivers.
- **Rate limiting adds overhead** (cache operations per job). Use Redis-optimised middleware for high-volume queues.

### Annotated Code Examples

**Example 1: Time-Bucket Rate Limiting with Redis**

```php
<?php
// File: app/Jobs/SendEmailViaSes.php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\Middleware\RateLimitedWithRedis;
use Illuminate\Queue\SerializesModels;

class SendEmailViaSes implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public string $to,
        public string $subject,
        public string $body,
    ) {}

    public function middleware(): array
    {
        // SES allows 14 emails per second
        return [new RateLimitedWithRedis('ses-email')];
    }

    public function handle(): void
    {
        // Send email via SES...
    }
}
```

```php
// File: app/Providers/AppServiceProvider.php

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('ses-email', function ($job) {
        return Limit::perSecond(14);
    });
}
```

**Expected Output:** Jobs are executed at a maximum rate of 14 per second. If the rate is exceeded, excess jobs are released back to the queue and retried when capacity becomes available.

**Why This Output Occurs:** The `RateLimitedWithRedis` middleware uses Redis atomic operations to increment a counter keyed by the limiter name and the current second. If the counter exceeds 14, the job is released back to the queue with a delay. The `RateLimitedWithRedis` middleware is optimised for Redis, using a single Lua script to check and increment the counter atomically.

---

**Example 2: Concurrency Control with WithoutOverlapping and Shared Keys**

```php
<?php
// File: app/Jobs/DeployApplication.php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\Middleware\WithoutOverlapping;
use Illuminate\Queue\SerializesModels;

class DeployApplication implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public string $environment,
        public string $commit,
    ) {}

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("deployment-{$this->environment}"))
                ->shared()           // Share lock across all deployment jobs
                ->releaseAfter(30)   // Retry after 30 seconds if locked
                ->expireAfter(600),  // Lock expires after 10 minutes
        ];
    }

    public function handle(): void
    {
        // Deploy the application...
    }
}
```

**Expected Output:** Only one deployment job per environment runs at a time. If a second deployment is dispatched while the first is running, it is released back to the queue and retried after 30 seconds.

**Why This Output Occurs:** The `WithoutOverlapping` middleware with `->shared()` uses a shared lock key (`deployment-production`) across all job classes that use the same key. This ensures that even if different job classes target the same environment, they won't overlap. The `releaseAfter` option releases the job if the lock is held, and `expireAfter` prevents permanent locks if the worker crashes.

### Real-World Cases

- **AWS SES email sending:** Rate-limited to 14 emails per second (SES default) with `RateLimitedWithRedis`.
- **Stripe payment processing:** Concurrency-limited to 1 payment per order with `WithoutOverlapping`, preventing double charges.
- **Database migrations:** Concurrency-limited with a shared lock to prevent simultaneous migrations on multiple servers.
- **Multi-tenant API calls:** Rate-limited per tenant ID to ensure fair usage across customers.
- **Video transcoding:** Concurrency-limited to a fixed number of simultaneous transcoding jobs to avoid CPU exhaustion.

---

## 3. Fault Tolerance & Backoffs

### Definitions

**Core Definition:** Fault tolerance is the ability of a queued job to recover from transient failures through configurable retry limits, explicit timeouts, and intelligent exponential backoff curves that space out retry attempts to avoid overwhelming external services.

**Technical Definition:** Fault tolerance is configured through job class properties: `$tries` (maximum attempts), `$backoff` (delay between retries, as an integer or array), `$timeout` (maximum seconds the job may run), `$failOnTimeout` (whether to fail permanently on timeout), `$maxExceptions` (maximum exceptions before failing early), and the `retryUntil()` method (time-based retry deadline). The `backoff()` method can also be defined to return a dynamic array of delays. The `Illuminate\Queue\Middleware\ThrottlesExceptions` middleware provides additional protection by throttling retries after a configurable number of exceptions. The `failed()` method is invoked when the job exhausts all retries, allowing cleanup and logging.

**Beginner-Friendly Explanation:** Fault tolerance means your job doesn't give up after one failure. It tries again — but not immediately, because the external service might be temporarily overloaded. Instead, it waits a bit longer each time: 1 second, then 5 seconds, then 30 seconds, then 2 minutes. This "exponential backoff" gives the external service time to recover. If the job still fails after all attempts, it goes to the failed jobs table for manual inspection.

### Purposes

- To ensure transient failures (network blips, temporary outages, rate limits) are retried automatically.
- To prevent retry storms by spacing out attempts with exponential backoff.
- To cap the total number of attempts and total time a job can consume.
- To handle permanent failures gracefully via the `failed()` method.
- To provide visibility into failure patterns through logging and monitoring.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Jobs/ChargeCustomer.php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\Attributes\Backoff;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Throwable;

class ChargeCustomer implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Maximum number of attempts.
     */
    public int $tries = 5;

    /**
     * Progressive backoff delays (seconds).
     */
    public array $backoff = [1, 5, 30, 120, 600];

    /**
     * Maximum exceptions before failing early.
     */
    public int $maxExceptions = 3;

    /**
     * Maximum seconds the job may run.
     */
    public int $timeout = 30;

    /**
     * Fail permanently on timeout (do not retry).
     */
    public bool $failOnTimeout = true;

    /**
     * Retry until this deadline (alternative to $tries).
     */
    // public function retryUntil(): DateTime
    // {
    //     return now()->addDay();
    // }

    public function handle(): void
    {
        // Charge the customer...
    }

    public function failed(Throwable $exception): void
    {
        // Handle permanent failure...
        \Log::error('Payment failed permanently', [
            'customer_id' => $this->customerId,
            'error'       => $exception->getMessage(),
        ]);
    }
}
```

```php
// Using the Backoff attribute (Laravel 10+)
use Illuminate\Queue\Attributes\Backoff;

#[Backoff([1, 5, 30, 120, 600])]
class ChargeCustomer implements ShouldQueue
{
    // ...
}
```

**Component Breakdown:**

- `$tries` — Maximum number of attempts (default: 1).
- `$backoff` — Integer (fixed delay) or array (progressive delays) in seconds.
- `$maxExceptions` — Maximum exceptions before failing early (even if `$tries` hasn't been reached).
- `$timeout` — Maximum seconds the job may run before being killed.
- `$failOnTimeout` — When `true`, the job fails permanently on timeout instead of being retried.
- `retryUntil()` — Returns a `DateTime`; the job is retried until this time, regardless of attempts.
- `failed(Throwable $exception)` — Called when the job exhausts all retries.

**Syntax Rules:**

- The `$backoff` array must have at least as many elements as `$tries` minus one.
- `retryUntil()` and `$tries` are mutually exclusive; `retryUntil()` takes precedence.
- `$failOnTimeout` requires the `pcntl` extension for timeout enforcement.
- The `failed()` method receives the exception that caused the failure.
- The `Backoff` attribute (PHP 8) can be used instead of the `$backoff` property.

**Constraints and Limitations:**

- **The `$timeout` option requires the `pcntl` extension.** Without it, timeouts are not enforced.
- **The `$backoff` array is consumed in order.** If the array has fewer elements than attempts, the last value is reused.
- **`retryUntil()` does not limit the number of attempts.** A job can be retried many times within the deadline.
- **The `failed()` method is only called for queued jobs.** Synchronous job failures propagate as exceptions.
- **The `config/queue.php` `retry_after` value must be longer than the job's `$timeout`.** Otherwise, the job may be retried while still running.

### Annotated Code Examples

**Example 1: Exponential Backoff with Progressive Delays**

```php
<?php
// File: app/Jobs/SyncWithStripe.php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Throwable;

class SyncWithStripe implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 5;
    public array $backoff = [1, 5, 30, 120, 600];

    public function __construct(
        public int $customerId,
    ) {}

    public function handle(): void
    {
        // Sync with Stripe API...
    }

    public function failed(Throwable $exception): void
    {
        \Log::error('Stripe sync failed permanently', [
            'customer_id' => $this->customerId,
            'error'       => $exception->getMessage(),
        ]);
    }
}
```

**Expected Output:** The job is attempted up to 5 times with delays of 1, 5, 30, 120, and 600 seconds between attempts. After the fifth failure, the `failed()` method logs the permanent failure.

**Why This Output Occurs:** The `$tries` property limits the job to 5 attempts. The `$backoff` array specifies the delay for each retry attempt. After each failure, the queue worker releases the job back to the queue with the next delay value. After the final attempt fails, the `failed()` method is invoked.

---

**Example 2: Time-Based Retry with retryUntil()**

```php
<?php
// File: app/Jobs/SendTimeSensitiveAlert.php

namespace App\Jobs;

use DateTime;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendTimeSensitiveAlert implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function handle(): void
    {
        // Send the alert...
    }

    /**
     * Retry the job until 2 hours from now.
     */
    public function retryUntil(): DateTime
    {
        return now()->addHours(2);
    }
}
```

**Expected Output:** The job is retried until 2 hours have passed, regardless of the number of attempts. After 2 hours, the job is marked as failed.

**Why This Output Occurs:** The `retryUntil()` method returns a `DateTime` instance representing the retry deadline. The queue worker checks this deadline before each attempt; if the deadline has passed, the job is marked as failed. This is useful for time-sensitive alerts that become irrelevant after a certain period.

### Real-World Cases

- **Payment processing:** Retried 5 times with exponential backoff `[1, 5, 30, 120, 600]` to handle transient Stripe API failures.
- **Email sending:** Retried 3 times with a 30-second backoff to handle temporary SMTP outages.
- **Webhook delivery:** Retried 10 times with exponential backoff over 24 hours to handle external service downtime.
- **Time-sensitive alerts:** Retried until a 2-hour deadline, after which the alert is discarded.
- **Database deadlocks:** Retried with a short backoff (1 second) to resolve transient lock contention.

---

## 4. Dead-Letter Queue Handling

### Definitions

**Core Definition:** A dead-letter queue (DLQ) is a storage mechanism for jobs that have permanently failed after exhausting all retry attempts. In Laravel, the `failed_jobs` database table serves as the dead-letter queue, and the `queue:failed`, `queue:retry`, `queue:forget`, and `queue:flush` Artisan commands provide inspection and management capabilities.

**Technical Definition:** When a job exhausts all retries (`$tries`) or exceeds its retry deadline (`retryUntil()`), the `Illuminate\Queue\Worker` invokes the `failJob()` method, which logs the job to the `failed_jobs` table (or DynamoDB if configured). The `failed_jobs` table stores the job's UUID, connection, queue, payload, exception, and failure timestamp. The `queue:failed` command lists failed jobs; `queue:retry` retries jobs by ID, queue, or all; `queue:forget` deletes a single failed job; and `queue:flush` deletes all failed jobs. The `queue:prune-failed` command removes old failed jobs based on age. The `DeleteWhenMissingModels` attribute automatically discards jobs whose models have been deleted, preventing expected `ModelNotFoundException` failures from cluttering the DLQ.

**Beginner-Friendly Explanation:** When a job fails permanently — after all retries are exhausted — it doesn't just disappear. It goes to a "dead-letter queue" (the `failed_jobs` table) where you can inspect what went wrong, see the full exception, and decide whether to retry it or delete it. This prevents permanent failures from being silently lost and gives you a way to recover from bugs or outages that have been fixed.

### Purposes

- To persist permanently failed jobs for inspection and debugging.
- To enable targeted retries of failed jobs after fixing the underlying issue.
- To provide visibility into failure patterns and recurring exceptions.
- To allow cleanup of old failed jobs through pruning commands.
- To handle expected failures (e.g., deleted models) gracefully without cluttering the DLQ.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# List all failed jobs
php artisan queue:failed

# Retry a single failed job by ID
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece

# Retry multiple failed jobs
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece 91401d2c-0784-4f43-824c-34f94a33c24d

# Retry all failed jobs for a specific queue
php artisan queue:retry --queue=name

# Retry all failed jobs
php artisan queue:retry all

# Delete a single failed job
php artisan queue:forget ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece

# Delete all failed jobs
php artisan queue:flush

# Prune failed jobs older than 48 hours
php artisan queue:prune-failed --hours=48
```

```php
// Auto-discard jobs with missing models
use Illuminate\Queue\Attributes\DeleteWhenMissingModels;

#[DeleteWhenMissingModels]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

**Component Breakdown:**

- `queue:failed` — Lists failed jobs with ID, connection, queue, and failure time.
- `queue:retry {id}` — Retries a specific failed job by UUID.
- `queue:retry --queue={name}` — Retries all failed jobs on a specific queue.
- `queue:retry all` — Retries all failed jobs.
- `queue:forget {id}` — Deletes a specific failed job.
- `queue:flush` — Deletes all failed job records.
- `queue:prune-failed --hours={n}` — Prunes failed jobs older than `n` hours (default: 24).
- `DeleteWhenMissingModels` — Attribute that discards jobs whose models have been deleted.

**Syntax Rules:**

- The `failed_jobs` table migration must be run before failed jobs can be stored.
- The `queue:retry` command accepts UUIDs, queue names, or `all`.
- The `queue:flush` command deletes all failed jobs regardless of age.
- The `queue:prune-failed` command uses the `--hours` option to filter by age.
- The `DeleteWhenMissingModels` attribute silently discards the job without logging a failure.

**Constraints and Limitations:**

- **Failed jobs are stored in the database, not the queue backend.** They must be manually retried or flushed.
- **Retrying a failed job re-dispatches it to the queue.** If the underlying issue is not fixed, it will fail again.
- **The `failed_jobs` table can grow large.** Use `queue:prune-failed` to manage its size.
- **The `DeleteWhenMissingModels` attribute only handles `ModelNotFoundException`.** Other exceptions are still logged as failures.
- **DynamoDB storage requires manual table creation.** The table must have a string primary partition key named `application` and a string sort key named `uuid`.

### Annotated Code Examples

**Example 1: Inspecting and Retrying Failed Jobs**

```bash
# Step 1: List all failed jobs
php artisan queue:failed
```

**Expected Output:**

```
+--------------------------------------+------------+---------+---------------------+
| ID                                   | Connection | Queue   | Failed At           |
+--------------------------------------+------------+---------+---------------------+
| ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece | redis      | default | 2025-06-15 10:00:00 |
| 91401d2c-0784-4f43-824c-34f94a33c24d | redis      | high    | 2025-06-15 10:05:00 |
+--------------------------------------+------------+---------+---------------------+
```

```bash
# Step 2: Retry a specific failed job
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece
```

**Expected Output:**

```
The failed job [ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece] has been pushed back onto the queue!
```

**Why This Output Occurs:** The `queue:failed` command queries the `failed_jobs` table and displays the job UUID, connection, queue, and failure timestamp. The `queue:retry` command retrieves the job payload from the table, re-pushes it to the queue, and removes it from the failed jobs table.

---

**Example 2: Using DeleteWhenMissingModels to Avoid DLQ Clutter**

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
    ) {}

    public function handle(): void
    {
        // Process the podcast...
    }
}
```

**Expected Output:** If the podcast is deleted before the job runs, the job is silently discarded (not failed, not logged in `failed_jobs`).

**Why This Output Occurs:** The `DeleteWhenMissingModels` attribute tells the `CallQueuedHandler` to catch `ModelNotFoundException` and delete the job without raising an exception. This prevents expected failures (deleted models) from cluttering the dead-letter queue.

### Real-World Cases

- **Payment processing:** Failed payment jobs are inspected in the DLQ; after fixing a Stripe integration bug, `queue:retry all` re-processes them.
- **Email delivery:** Failed email jobs are retried after an SMTP outage; `queue:prune-failed --hours=48` cleans up old failures.
- **Video processing:** Jobs whose video models have been deleted are automatically discarded via `DeleteWhenMissingModels`.
- **Data synchronization:** Failed CRM sync jobs are manually inspected and retried after API credentials are updated.
- **Report generation:** Failed report jobs are retried with `queue:retry --queue=reports` after a database performance issue is resolved.

---

## 5. Idempotency & State Safety

### Definitions

**Core Definition:** Idempotency is the property of a job that ensures repeated executions produce the same result without causing duplicate side effects or corrupting database state. State safety is the broader practice of designing jobs so that retries, duplicate dispatches, and partial failures never leave the system in an inconsistent state.

**Technical Definition:** Idempotency in Laravel is achieved through several mechanisms: (1) checking the current state before performing an action (e.g., "is this payment already processed?"); (2) using idempotency keys stored in a database table or cache with a unique constraint; (3) using `ShouldBeUnique` to prevent duplicate dispatches; (4) wrapping job logic in database transactions so that partial failures are rolled back; (5) using the `afterCommit()` method to defer job dispatch until the parent transaction commits; and (6) using the `SaveQuietly` or `withoutEvents` methods to prevent recursive event dispatch. The `Illuminate\Bus\UniqueLock` class handles uniqueness locks for `ShouldBeUnique` jobs. Packages like `squipix/laravel-idempotency` provide middleware that caches idempotency keys and prevents duplicate job execution.

**Beginner-Friendly Explanation:** Idempotency means "it's safe to run this twice." Imagine a job that charges a customer $50. If the job runs twice due to a retry, the customer gets charged $100. An idempotent version of the job first checks "has this order already been charged?" — if yes, it skips the charge. This ensures that even if the job runs multiple times, the customer is charged exactly once.

### Purposes

- To ensure that retried jobs do not cause duplicate side effects (double charges, duplicate emails).
- To prevent concurrent duplicate dispatches from corrupting data.
- To maintain database consistency even when jobs fail partway through.
- To enable safe retry strategies without manual intervention.
- To provide exactly-once semantics for critical operations (payments, inventory adjustments).

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Approach 1: State check before action
public function handle(): void
{
    if ($this->order->status === 'charged') {
        return; // Already processed
    }

    // Charge the order...
    $this->order->update(['status' => 'charged']);
}
```

```php
// Approach 2: Idempotency key with unique constraint
public function handle(): void
{
    $key = "charge-order-{$this->orderId}";

    // Try to create the idempotency record; fails if already exists
    $record = IdempotencyRecord::firstOrCreate(
        ['key' => $key],
        ['processed_at' => now()]
    );

    if (!$record->wasRecentlyCreated) {
        return; // Already processed
    }

    // Charge the order...
}
```

```php
// Approach 3: ShouldBeUnique interface
use Illuminate\Contracts\Queue\ShouldBeUnique;

class ChargeOrder implements ShouldQueue, ShouldBeUnique
{
    public function uniqueId(): string
    {
        return 'charge-order-' . $this->orderId;
    }

    public function uniqueFor(): int
    {
        return 3600; // Lock expires after 1 hour
    }
}
```

```php
// Approach 4: Database transaction with rollback
public function handle(): void
{
    DB::transaction(function () {
        $this->order->update(['status' => 'charged']);
        $this->payment->create([...]);
    });
}
```

```php
// Approach 5: afterCommit dispatch
ProcessOrder::dispatch($order)->afterCommit();
```

**Component Breakdown:**

- State check — Query the current state before performing an action.
- Idempotency key — A unique string stored in the database or cache to track processed operations.
- `ShouldBeUnique` — Interface that prevents duplicate concurrent dispatches.
- `uniqueId()` — Returns the uniqueness key for the job.
- `uniqueFor()` — Seconds after which the uniqueness lock expires.
- `DB::transaction()` — Wraps job logic in a database transaction for atomicity.
- `afterCommit()` — Defers job dispatch until the parent transaction commits.

**Syntax Rules:**

- The `ShouldBeUnique` interface requires a cache driver that supports atomic locks.
- The `uniqueId()` method must return a string.
- The `uniqueFor()` method returns the lock TTL in seconds (default: 0, meaning permanent until released).
- Database transactions automatically roll back on exceptions, reverting all uncommitted changes.
- The `afterCommit()` method prevents workers from reading data before the transaction commits.

**Constraints and Limitations:**

- **State checks are not atomic by default.** Two concurrent workers could both pass the check before either updates the state. Use database locks or atomic operations for critical sections.
- **Idempotency keys require a unique constraint** to prevent race conditions.
- **`ShouldBeUnique` prevents duplicate dispatches, not duplicate executions.** A job that has already been queued can still be executed once.
- **Database transactions do not span multiple databases.** Cross-database consistency requires additional coordination.
- **The `afterCommit()` method only works when a transaction is active.** If no transaction is open, the job dispatches immediately.

### Annotated Code Examples

**Example 1: Idempotent Payment Job with State Check and Transaction**

```php
<?php
// File: app/Jobs/ChargeOrder.php

namespace App\Jobs;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\DB;

class ChargeOrder implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function handle(): void
    {
        // Step 1: Check if the order has already been charged
        if ($this->order->status === 'charged') {
            return; // Idempotent: already processed
        }

        // Step 2: Wrap the charge and status update in a transaction
        DB::transaction(function () {
            // Re-check inside the transaction with a lock
            $order = Order::where('id', $this->order->id)
                ->lockForUpdate()
                ->first();

            if ($order->status === 'charged') {
                return; // Another worker charged it first
            }

            // Charge the order...
            $this->chargePayment($order);

            $order->update(['status' => 'charged']);
        });
    }

    private function chargePayment(Order $order): void
    {
        // Payment gateway logic...
    }
}
```

**Expected Output:** The job is idempotent — if it runs twice, the second run skips the charge because the order status is already `charged`. The `lockForUpdate()` ensures that concurrent workers cannot both pass the status check.

**Why This Output Occurs:** The initial state check provides a fast path for already-processed orders. The database transaction with `lockForUpdate()` ensures that only one worker can proceed to charge the order. If two workers run simultaneously, the first acquires the lock, charges the order, and commits. The second waits for the lock, re-checks the status (now `charged`), and skips the charge.

---

**Example 2: Idempotency Key with Unique Constraint**

```php
<?php
// File: app/Jobs/SendInvoiceEmail.php

namespace App\Jobs;

use App\Models\IdempotencyRecord;
use App\Models\Invoice;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendInvoiceEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Invoice $invoice,
    ) {}

    public function handle(): void
    {
        $key = "send-invoice-email-{$this->invoice->id}";

        // Attempt to create the idempotency record
        $record = IdempotencyRecord::firstOrCreate(
            ['key' => $key],
            ['processed_at' => now()]
        );

        if (!$record->wasRecentlyCreated) {
            // Already processed — skip
            return;
        }

        // Send the email...
        \Mail::to($this->invoice->customer->email)
            ->send(new \App\Mail\InvoicePaid($this->invoice));
    }
}
```

```php
// IdempotencyRecord model with unique constraint
Schema::create('idempotency_records', function (Blueprint $table) {
    $table->id();
    $table->string('key')->unique();
    $table->timestamp('processed_at');
    $table->timestamps();
});
```

**Expected Output:** The email is sent only once per invoice, even if the job is dispatched multiple times.

**Why This Output Occurs:** The `firstOrCreate()` method attempts to create a record with the unique key. If the record already exists (from a previous execution), it returns the existing record and `wasRecentlyCreated` is `false`. The job skips the email send. The unique constraint on the `key` column prevents race conditions — even if two workers run simultaneously, only one can create the record.

### Real-World Cases

- **Payment processing:** Idempotent charge jobs with state checks and `lockForUpdate()` prevent double charges.
- **Email sending:** Idempotency keys prevent duplicate emails when the same job is dispatched multiple times.
- **Inventory adjustments:** State checks and transactions ensure inventory is decremented exactly once per order.
- **Subscription renewals:** Idempotency keys prevent duplicate renewals when the renewal job is retried.
- **Notification dispatch:** `ShouldBeUnique` prevents duplicate notifications for the same event.

---

## 6. Enterprise Observability

### Definitions

**Core Definition:** Enterprise observability is the practice of monitoring queue performance, throughput, error states, and worker health in real time using Laravel Horizon, providing the visibility needed to diagnose issues, plan capacity, and ensure service-level objectives.

**Technical Definition:** Laravel Horizon is a first-party package that provides a dashboard and code-driven configuration for Redis-backed queues. It collects metrics on job throughput, runtime, and failures; displays queue wait times and worker status; supports job tagging for filtering; and provides notifications for long wait times. Horizon's `config/horizon.php` file defines supervisors and workers, with auto-balancing strategies (`simple`, `auto`, `false`) that dynamically allocate worker processes based on queue load. Metrics are stored in Redis and can be exported to Prometheus for integration with Grafana dashboards. The `horizon:terminate` command gracefully restarts Horizon during deployments.

**Beginner-Friendly Explanation:** Horizon is like a mission control dashboard for your queues. It shows you how many jobs are waiting, how fast they're being processed, which jobs are failing, and how long jobs are waiting before being picked up. If a queue is backing up, you can see it in real time. You can also configure Horizon to automatically balance workers across queues based on load, so you don't have to manually manage process counts.

### Purposes

- To provide real-time visibility into job throughput, runtime, and failure rates.
- To monitor queue wait times and detect backlogs before they become incidents.
- To enable code-driven worker configuration with auto-balancing for optimal resource allocation.
- To support job tagging and filtering for debugging specific job types.
- To integrate with external monitoring systems (Prometheus, Grafana) for enterprise-wide observability.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Install Horizon
composer require laravel/horizon
php artisan horizon:install
php artisan migrate

# Start Horizon
php artisan horizon

# Gracefully terminate Horizon (during deployment)
php artisan horizon:terminate

# List Horizon supervisors
php artisan horizon:list

# Pause processing
php artisan horizon:pause

# Resume processing
php artisan horizon:continue

# Clear metrics
php artisan horizon:clear
```

```php
// File: config/horizon.php

return [
    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection'    => 'redis',
                'queue'         => ['high', 'default', 'low'],
                'balance'       => 'auto',
                'autoScalingStrategy' => 'time',
                'maxProcesses'  => 10,
                'maxTime'       => 0,
                'maxJobs'       => 0,
                'memory'        => 128,
                'tries'         => 3,
                'timeout'       => 60,
                'nice'          => 0,
            ],
        ],

        'local' => [
            'supervisor-1' => [
                'connection'    => 'redis',
                'queue'         => ['default'],
                'balance'       => 'simple',
                'maxProcesses'  => 3,
                'maxTime'       => 0,
                'maxJobs'       => 0,
                'memory'        => 128,
                'tries'         => 3,
                'timeout'       => 60,
            ],
        ],
    ],
];
```

**Component Breakdown:**

- `horizon:install` — Publishes the Horizon configuration file and dashboard assets.
- `horizon` — Starts the Horizon master process, which manages supervisors and workers.
- `horizon:terminate` — Gracefully terminates Horizon (waits for in-progress jobs to complete).
- `horizon:pause` / `horizon:continue` — Pauses and resumes job processing.
- `horizon:clear` — Clears Horizon metrics from Redis.
- `balance` — Auto-balancing strategy (`simple`, `auto`, `false`).
- `autoScalingStrategy` — Strategy for auto-scaling (`time` or `size`).
- `maxProcesses` — Maximum worker processes per supervisor.
- `maxTime` / `maxJobs` — Process recycling parameters.
- `memory` — Maximum RAM per worker (MB).
- `tries` / `timeout` — Retry and timeout configuration.

**Syntax Rules:**

- Horizon requires Redis as the queue backend.
- The `balance` option must be `simple`, `auto`, or `false`.
- The `autoScalingStrategy` can be `time` (scale based on queue wait time) or `size` (scale based on queue size).
- The dashboard is available at `/horizon` by default; access must be restricted in production.
- Horizon's metrics are stored in Redis; the `horizon:clear` command removes them.

**Constraints and Limitations:**

- **Horizon only works with Redis queues.** It does not support database, SQS, or Beanstalkd.
- **The Horizon dashboard must be secured.** Exposing it publicly reveals sensitive queue information.
- **Horizon's auto-balancing may override manual queue priorities.** When `balance` is `auto`, Horizon allocates workers based on load, not priority.
- **Metrics are stored in Redis and can consume memory.** Use `horizon:clear` or configure metrics retention.
- **Horizon does not support SQS FIFO queues.** FIFO ordering guarantees are not available with Horizon.

### Annotated Code Examples

**Example 1: Configuring Horizon with Auto-Balancing**

```php
// File: config/horizon.php

return [
    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection'    => 'redis',
                'queue'         => ['high', 'default', 'low'],
                'balance'       => 'auto',
                'autoScalingStrategy' => 'time',
                'maxProcesses'  => 10,
                'minProcesses'  => 1,
                'maxTime'       => 3600,
                'maxJobs'       => 1000,
                'memory'        => 256,
                'tries'         => 3,
                'timeout'       => 90,
                'nice'          => 0,
            ],
        ],
    ],
];
```

```bash
# Start Horizon
php artisan horizon
```

**Expected Output:** Horizon starts a supervisor that dynamically allocates worker processes across the `high`, `default`, and `low` queues based on wait time. When the `high` queue is backed up, more workers are allocated to it. When it is empty, workers are reallocated to `default` and `low`.

**Why This Output Occurs:** The `balance` option is set to `auto`, which enables Horizon's auto-balancing strategy. The `autoScalingStrategy` is `time`, meaning Horizon scales based on the average wait time of jobs in each queue. The `maxProcesses` and `minProcesses` options bound the number of worker processes. The `maxTime` and `maxJobs` options recycle workers after 1 hour or 1000 jobs, preventing memory accumulation.

---

**Example 2: Job Tagging for Filtering in Horizon**

```php
<?php
// File: app/Jobs/ProcessPodcast.php

namespace App\Jobs;

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
    ) {}

    public function handle(): void
    {
        // Process the podcast...
    }

    /**
     * Get the tags that should be assigned to the job.
     *
     * @return array<int, string>
     */
    public function tags(): array
    {
        return [
            'podcast',
            'podcast:' . $this->podcast->id,
        ];
    }
}
```

**Expected Output:** The job appears in the Horizon dashboard with the tags `podcast` and `podcast:123`. You can filter the dashboard by these tags to see all jobs related to a specific podcast.

**Why This Output Occurs:** The `tags()` method returns an array of strings that Horizon assigns to the job when it is dispatched. The Horizon dashboard provides a tag filter that allows you to search for jobs by tag, making it easy to debug issues with specific job types or entities.

### Real-World Cases

- **E-commerce order processing:** Horizon monitors the `orders` queue, with auto-balancing allocating more workers during peak hours.
- **SaaS multi-tenant queues:** Job tags (`tenant:123`, `tenant:456`) allow per-tenant filtering in the Horizon dashboard.
- **Financial batch processing:** Horizon's metrics show throughput and runtime for nightly batch jobs, with alerts on long wait times.
- **Video processing platform:** Horizon's auto-balancing scales workers based on the `transcoding` queue's wait time.
- **Enterprise observability:** Horizon metrics are exported to Prometheus and visualised in Grafana alongside application metrics.

---

## References

- Laravel Queues Documentation (Master) — https://laravel.com/framework/docs/master/queues
- Laravel Queues Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/queues
- Laravel Queues Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/queues
- Laravel Horizon Documentation — https://laravel.com/docs/12.x/horizon
- Laravel `RateLimited` Middleware API — https://api.laravel.com/docs/master/Illuminate/Queue/Middleware/RateLimited.html
- Laravel `WithoutOverlapping` Middleware API — https://api.laravel.com/docs/master/Illuminate/Queue/Middleware/WithoutOverlapping.html
- Laravel `ThrottlesExceptions` Middleware API — https://api.laravel.com/docs/master/Illuminate/Queue/Middleware/ThrottlesExceptions.html
- Laravel `RateLimitedWithRedis` Middleware — https://laravel.com/docs/12.x/queues#rate-limiting-with-redis
- Laravel `ThrottlesExceptionsWithRedis` Middleware — https://laravel.com/docs/12.x/queues#throttling-exceptions
- Laravel Failed Jobs Documentation — https://laravel.com/docs/12.x/queues#dealing-with-failed-jobs
- Laravel `DeleteWhenMissingModels` Attribute — https://laravel.com/docs/12.x/queues#ignoring-missing-models
- Laravel Horizon Prometheus Exporter — https://packagist.org/packages/boring-o11y/horizon-prometheus-exporter
- Laravel Horizon Telemetry (OpenTelemetry) — https://packagist.org/packages/worksome/horizon-telemetry
- Managing API Rate Limits in Laravel Through Job Throttling (Laravel News) — https://laravel-news.com/managing-api-rate-limits-in-laravel-through-job-throttling
- Laravel Queues: Handling Errors Gracefully (PHP Architect) — https://www.phparch.com/2025/08/laravel-queues-handling-errors-gracefully/
- Enhanced Queue Job Control with `ThrottlesExceptions` `failWhen()` (Laravel News) — https://laravel-news.com/throttlesexceptions-failwhen
- `squipix/laravel-idempotency` Package — https://packagist.org/packages/squipix/laravel-idempotency
- `adilazhari/laravel-idempotency` Package — https://packagist.org/packages/adilazhari/laravel-idempotency
- Laravel Queue Configuration Best Practices (Dutch Laravel Foundation) — https://github.com/Dutch-Laravel-Foundation/best-practices
- Laravel Queue Exponential Backoff Rules — https://github.com/voku/agent-skills
- Laravel `ShouldBeUnique` Interface — https://laravel.com/docs/12.x/queues#unique-jobs
- Laravel Transactional Events Package — https://packagist.org/packages/fntneves/laravel-transactional-events