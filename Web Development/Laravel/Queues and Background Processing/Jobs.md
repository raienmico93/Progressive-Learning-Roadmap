# Laravel Jobs: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A job in Laravel is a self-contained PHP class that encapsulates a unit of work to be executed asynchronously or synchronously, typically representing a background task such as sending an email, processing an upload, or generating a report.

**Technical Definition**

A Laravel job is a class stored in `app/Jobs` that implements the `Illuminate\Contracts\Queue\ShouldQueue` interface, indicating that it should be pushed onto a queue rather than executed immediately. Jobs contain a `handle()` method that is invoked when the queue worker processes the job, and they can optionally define a `failed()` method for failure handling. Jobs support serialization, dependency injection, middleware, retries, timeouts, chaining, batching, and unique constraints. The `make:job` Artisan command scaffolds the class with the `ShouldQueue` interface and the `Queueable` trait.

**Beginner-Friendly Explanation**

A job is a task you want Laravel to do later, in the background. Instead of making your user wait while a slow task runs (like sending a welcome email or processing a large file), you create a job, hand it to Laravel’s queue system, and Laravel does the work in the background. Your app stays fast, and the job gets done reliably. Think of a job as a “to-do note” you leave for a background worker.

---

### Key Characteristics

- **Asynchronous by Default**: Jobs implementing `ShouldQueue` are pushed onto a queue and processed by background workers, freeing the web request to respond immediately.
- **Serialization**: Job instances (and their constructor properties) are serialized onto the queue; Eloquent models are automatically serialized and re-retrieved.
- **Dependency Injection**: The `handle()` method is resolved by Laravel’s service container, allowing type-hinted dependencies to be injected automatically.
- **Lifecycle Hooks**: Jobs can define `handle()`, `failed()`, `middleware()`, `retryUntil()`, and properties such as `$tries`, `$timeout`, `$backoff`, and `$maxExceptions`.
- **Composable Workflows**: Jobs can be chained (`Bus::chain()`), batched (`Bus::batch()`), and combined with middleware for rate limiting, overlap prevention, and exception throttling.
- **Testing-Friendly**: `Queue::fake()` and `Bus::fake()` allow tests to assert on dispatched jobs without executing them.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with a configured queue connection in `config/queue.php`.
- **A queue driver**: `database`, `redis`, `sqs`, `beanstalkd`, or `sync` for local development.
- **A queue worker** (Supervisor, Horizon, or `php artisan queue:work`) for production processing.
- **Composer** for installing driver dependencies (e.g., `predis/predis` for Redis, `aws/aws-sdk-php` for SQS).

---

### Related Programming Areas

- **Queues**: Jobs are the unit of work placed onto queues.
- **Events and Listeners**: Queued listeners are jobs under the hood and use the same infrastructure.
- **Mail and Notifications**: Both can be queued via `ShouldQueue` and are tested with fakes.
- **Task Scheduling**: Scheduled commands dispatch jobs for heavy work.
- **Laravel Horizon**: A Redis-specific dashboard for monitoring queue jobs and workers.

---

## Core Concepts

---

### 1. Jobs

#### Definitions

**Core Definition**

A job is a PHP class that represents a single unit of work to be executed by the queue system, encapsulating both the data needed for the work and the logic to perform it.

**Technical Definition**

A Laravel job is a class typically stored in `app/Jobs` that implements `ShouldQueue` and uses the `Queueable` trait. It contains a `handle()` method for the work and may define properties such as `$tries`, `$timeout`, `$backoff`, `$maxExceptions`, and `$failOnTimeout`, as well as a `failed()` method for failure handling and a `middleware()` method for job middleware. Jobs are dispatched via the `dispatch()` helper or the static `dispatch()` method provided by the `Dispatchable` trait.

**Beginner-Friendly Explanation**

A job is a class that describes a task. It holds the data the task needs (like a user ID or a file path) and the code that does the work (the `handle()` method). You create a job class, fill in the `handle()` method, and dispatch it.

#### Purposes

- To encapsulate background work in a reusable, testable, serializable class.
- To decouple slow or unreliable work from the web request lifecycle.
- To provide a structured unit for retry, timeout, and failure handling.
- To enable dependency injection of services into background work.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Throwable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    public int $tries = 3;
    public int $timeout = 120;
    public int $backoff = 10;

    public function __construct(
        public Podcast $podcast,
    ) {}

    public function handle(AudioProcessor $processor): void
    {
        $processor->process($this->podcast);
    }

    public function failed(?Throwable $exception): void
    {
        // Handle permanent failure
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `implements ShouldQueue` | Marks the class as queueable. |
| `use Queueable` | Provides dispatch helpers and queue configuration methods. |
| `__construct()` | Receives the job’s data; supports dependency injection. |
| `handle()` | The job’s work; resolved by the container. |
| `failed()` | Called when the job fails permanently. |
| `$tries` / `$timeout` / `$backoff` | Retry and timeout configuration. |

**Syntax Rules**

- The class must implement `ShouldQueue` to be queued; otherwise, it runs synchronously.
- The `handle()` method is required and is resolved via the service container.
- Public properties are serialized; closures and non-serializable objects cannot be queued.
- Use `base64_encode()` for binary data before passing it to a job.

**Constraints and Limitations**

- Job payloads must be serializable to JSON; large payloads (e.g., full Eloquent relationships) can cause performance issues.
- The `failed()` method receives a fresh instance of the job, so state changes from `handle()` are lost.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Job**

```php
<?php
// app/Jobs/SendWelcomeEmail.php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class SendWelcomeEmail implements ShouldQueue
{
    use Queueable;

    public function __construct(public User $user) {}

    public function handle(): void
    {
        // Send the welcome email
        Mail::to($this->user->email)->send(new WelcomeEmail($this->user));
    }
}
```

**Dispatching**

```php
SendWelcomeEmail::dispatch($user);
```

**Expected Output**

The job is serialized onto the queue and processed by a worker, which sends the welcome email.

**Why This Output Occurs**

`dispatch()` pushes the job onto the default queue. The worker deserializes it, resolves `handle()` via the container, and executes the email-sending logic.

---

#### Real-World Cases with Explanation

**Case 1: User Registration**

When a user registers, the application dispatches a `SendWelcomeEmail` job instead of sending the email during the request. The user gets an immediate response, and the email is sent moments later by a worker.

**Explanation**: Jobs decouple slow side effects from the user-facing request.

---

### 2. Job Creation (`artisan make:job`)

#### Definitions

**Core Definition**

`artisan make:job` is the Artisan command that generates a new job class in `app/Jobs` with the `ShouldQueue` interface and `Queueable` trait pre-configured.

**Technical Definition**

The command is handled by Laravel’s `JobMakeCommand` and creates a PHP file in `app/Jobs` (creating the directory if necessary). The generated class implements `Illuminate\Contracts\Queue\ShouldQueue` and uses the `Illuminate\Foundation\Queue\Queueable` trait. The `--batched` flag (Laravel 12+) generates a batch-ready job with the `Batchable` trait.

**Beginner-Friendly Explanation**

Creating a job is as simple as running `php artisan make:job ProcessPodcast`. Laravel creates the file with the right structure, and you fill in the `handle()` method.

#### Purposes

- To scaffold a correctly structured job class without manual boilerplate.
- To ensure the job implements the necessary interfaces and traits.
- To support batch-ready and queue-ready job generation.
- To enforce Laravel’s directory and naming conventions.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:job ProcessPodcast
php artisan make:job ProcessDocument --batched
```

**Component Breakdown**

| Command / Flag | Description |
|----------------|-------------|
| `make:job {name}` | Generates `app/Jobs/{name}.php`. |
| `--batched` | Adds `Batchable` trait for batch processing. |

**Syntax Rules**

- The job name should be PascalCase (e.g., `ProcessPodcast`).
- The `--batched` flag is available in Laravel 12+.
- The generated file implements `ShouldQueue` and uses `Queueable`.

**Constraints and Limitations**

- Generated jobs do not include a `failed()` method by default; add it manually if needed.
- The `middleware()` method is not scaffolded; add it manually.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Job**

```bash
php artisan make:job ProcessPodcast
```

**Generated File**

```php
<?php
// app/Jobs/ProcessPodcast.php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    public function __construct() {}

    public function handle(): void
    {
        //
    }
}
```

**Expected Output**

```
INFO  Job [app/Jobs/ProcessPodcast.php] created successfully.
```

**Why This Output Occurs**

The command scaffolds the class with the correct namespace, interface, and trait.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Job Development**

A developer runs `php artisan make:job GenerateReport` and immediately fills in the `handle()` method with the report-generation logic. The scaffold removes setup friction and enforces consistency.

**Explanation**: Artisan job generation accelerates development while ensuring correct structure.

---

### 3. Job Dispatching

#### Definitions

**Core Definition**

Job dispatching is the act of pushing a job instance onto a queue so that it will be processed by a worker.

**Technical Definition**

Dispatching can be done via the global `dispatch()` helper, the static `dispatch()` method provided by the `Dispatchable` trait, or the `Bus` facade. Jobs are dispatched to the default connection and queue unless `onConnection()` or `onQueue()` is used. The `dispatchAfterResponse()` method delays dispatching until after the HTTP response is sent, and `afterCommit` configuration defers dispatch until a database transaction commits.

**Beginner-Friendly Explanation**

Dispatching means “put this job on the queue.” You call `SendWelcomeEmail::dispatch($user)` and Laravel hands the job to the queue system.

#### Purposes

- To hand a job to the queue system for asynchronous processing.
- To control which connection and queue the job is placed on.
- To defer dispatch until after a response or transaction commit.
- To support conditional and delayed dispatching.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Helper
dispatch(new SendWelcomeEmail($user));

// Static method
SendWelcomeEmail::dispatch($user);

// With queue and connection
SendWelcomeEmail::dispatch($user)
    ->onQueue('emails')
    ->onConnection('redis');

// Delayed dispatch
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(10));

// After response
SendWelcomeEmail::dispatchAfterResponse($user);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `dispatch()` | Pushes the job onto the queue. |
| `onQueue('name')` | Targets a specific queue. |
| `onConnection('name')` | Targets a specific connection. |
| `delay()` | Delays processing until a given time. |
| `dispatchAfterResponse()` | Dispatches after the HTTP response. |

**Syntax Rules**

- `dispatch()` respects the connection’s default queue unless `onQueue()` is called.
- `delay()` accepts a `DateTimeInterface`, `DateInterval`, or integer seconds.
- `dispatchAfterResponse()` runs the job synchronously after the response is sent.

**Constraints and Limitations**

- Jobs dispatched inside a transaction may run before the transaction commits unless `afterCommit` is configured.
- `dispatchAfterResponse()` does not survive process termination.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Dispatching to a Specific Queue**

```php
SendWelcomeEmail::dispatch($user)->onQueue('emails');
```

**Expected Output**

The job is placed on the `emails` queue and processed by a worker listening to that queue.

**Why This Output Occurs**

`onQueue('emails')` overrides the connection’s default queue for this dispatch.

---

**Example 2: Delayed Dispatch**

```php
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(5));
```

**Expected Output**

The job becomes available for processing 5 minutes after dispatch.

**Why This Output Occurs**

The `delay()` method sets an `available_at` timestamp on the queued job.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Confirmation**

An order confirmation email is dispatched to the `emails` queue with a 2-minute delay, giving the payment webhook time to settle before the email is sent.

**Explanation**: Delayed dispatch handles timing-sensitive side effects.

---

### 4. Job Chaining

#### Definitions

**Core Definition**

Job chaining is a pattern where a sequence of jobs is executed in order, with each job running only after the previous one succeeds.

**Technical Definition**

Chaining is implemented via `Bus::chain([...])->dispatch()`. Jobs in the chain are executed sequentially on the same connection and queue by default. If any job in the chain fails, the remaining jobs are not executed, and the chain’s `catch()` callback (if provided) is invoked. Chains can also include batches within them and can be nested within batches.

**Beginner-Friendly Explanation**

Chaining means “do this, then that, then that.” If one step fails, the rest of the chain stops.

#### Purposes

- To model multi-step workflows where order matters.
- To ensure a job only runs if its prerequisites succeeded.
- To handle failures at the workflow level with a single `catch()` callback.
- To compose complex processes from smaller, reusable jobs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

**With `catch()`**

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
])->catch(function (Throwable $e) {
    // Handle chain failure
})->dispatch();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Bus::chain([...])` | Defines the sequence of jobs. |
| `->dispatch()` | Dispatches the first job and queues the rest. |
| `->catch()` | Handles failure of any job in the chain. |

**Syntax Rules**

- Jobs in the chain run in the order provided.
- If a job fails, the chain stops.
- The `catch()` callback receives the exception.

**Constraints and Limitations**

- Chain jobs must be serializable.
- Chained jobs share the same queue unless overridden.
- Closures can be chained in Laravel 11+.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Chain**

```php
Bus::chain([
    new ProcessPodcast($podcast),
    new OptimizePodcast($podcast),
    new ReleasePodcast($podcast),
])->dispatch();
```

**Expected Output**

`ProcessPodcast` runs first; on success, `OptimizePodcast` runs; on success, `ReleasePodcast` runs.

**Why This Output Occurs**

`Bus::chain()` queues each job with a dependency on the previous job’s completion.

---

#### Real-World Cases with Explanation

**Case 1: Podcast Publishing Pipeline**

A podcast is uploaded, processed, optimized, and released in a three-step chain. If processing fails, optimization and release are skipped, and the team is notified via the chain’s `catch()` callback.

**Explanation**: Chaining models sequential workflows with automatic failure propagation.

---

### 5. Job Batching

#### Definitions

**Core Definition**

Job batching is a pattern where a group of jobs is dispatched together as a batch, with callbacks for completion, failure, and progress tracking.

**Technical Definition**

Batching is implemented via `Bus::batch([...])->dispatch()`. Jobs in a batch must use the `Illuminate\Bus\Batchable` trait, which provides access to the batch instance (`$this->batch()`). Batches support `then()`, `catch()`, and `finally()` callbacks, progress tracking (`progress()`, `processedJobs()`, `failedJobs()`), cancellation (`cancel()`), and adding jobs from within a batched job (`$this->batch()->add([...])`). Batches require a `job_batches` table, created via `php artisan make:queue-batches-table`.

**Beginner-Friendly Explanation**

Batching means “run all these jobs together and tell me when they’re all done.” It is useful when you have many independent jobs that should be tracked as a group.

#### Purposes

- To group related jobs and track their collective progress.
- To execute callbacks when all jobs succeed, any job fails, or the batch finishes.
- To cancel remaining jobs if needed.
- To process large numbers of independent tasks efficiently.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([
    new ProcessImage('img1.jpg'),
    new ProcessImage('img2.jpg'),
    new ProcessImage('img3.jpg'),
])->then(function (Batch $batch) {
    // All jobs completed successfully
})->catch(function (Batch $batch, Throwable $e) {
    // A job failed
})->finally(function (Batch $batch) {
    // Batch finished (success or failure)
})->name('Image Processing')->dispatch();
```

**Batchable Job**

```php
use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessImage implements ShouldQueue
{
    use Batchable, Queueable;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Bus::batch([...])` | Defines the batch. |
| `->then()` | Runs when all jobs succeed. |
| `->catch()` | Runs when any job fails. |
| `->finally()` | Runs when the batch finishes. |
| `Batchable` trait | Provides `$this->batch()` access. |

**Syntax Rules**

- Jobs in a batch must use the `Batchable` trait.
- The `job_batches` table must exist.
- `$this->batch()->add()` can add jobs from within a batched job.
- `$this->batch()->cancelled()` checks for cancellation.

**Constraints and Limitations**

- Batch jobs must be serializable.
- The `finally()` callback runs even if the batch is cancelled.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Batch with Callbacks**

```php
$batch = Bus::batch([
    new ProcessImage('a.jpg'),
    new ProcessImage('b.jpg'),
    new ProcessImage('c.jpg'),
])->then(function (Batch $batch) {
    Log::info("Batch {$batch->id} completed");
})->catch(function (Batch $batch, Throwable $e) {
    Log::error("Batch {$batch->id} failed: {$e->getMessage()}");
})->dispatch();
```

**Expected Output**

When all three jobs succeed, the `then()` callback logs success. If one fails, the `catch()` callback logs the failure.

**Why This Output Occurs**

The batch tracks each job’s outcome and invokes the appropriate callback.

---

**Example 2: Progress Tracking**

```php
$batch = Bus::findBatch($batchId);
$progress = $batch->progress(); // 0-100
$pending = $batch->pendingJobs;
$failed = $batch->failedJobs;
```

**Expected Output**

The batch’s progress percentage, pending jobs, and failed jobs can be inspected at any time.

**Why This Output Occurs**

The `Batch` instance exposes methods and properties for real-time inspection.

---

#### Real-World Cases with Explanation

**Case 1: Image Processing Pipeline**

A user uploads 100 images. The application dispatches a batch of 100 `ProcessImage` jobs, tracks progress on a dashboard, and sends a notification when all images are processed.

**Explanation**: Batching is ideal for independent, parallelizable work that needs collective tracking.

---

### 6. Job Middleware

#### Definitions

**Core Definition**

Job middleware are classes that wrap the execution of a job, allowing you to intercept, modify, delay, or conditionally skip the job’s execution.

**Technical Definition**

Job middleware implement a `handle($job, $next)` method, where `$job` is the job instance and `$next` is the callback that continues processing. Middleware are returned from the job’s `middleware()` method. Laravel includes built-in middleware: `RateLimited`, `RateLimitedWithRedis`, `WithoutOverlapping`, `ThrottlesExceptions`, and `Skip`. Custom middleware can be generated with `php artisan make:job-middleware`.

**Beginner-Friendly Explanation**

Job middleware are like security checkpoints for a job. They can delay the job, prevent it from running if another instance is already running, or throttle it if it keeps failing.

#### Purposes

- To enforce rate limits on jobs (e.g., API calls).
- To prevent overlapping execution of the same job.
- To throttle exceptions from unstable third-party services.
- To conditionally skip jobs based on runtime conditions.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Queue\Middleware\RateLimited;
use Illuminate\Queue\Middleware\WithoutOverlapping;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

public function middleware(): array
{
    return [
        new RateLimited('api-calls'),
        new WithoutOverlapping($this->order->id),
        new ThrottlesExceptions(10, 300),
    ];
}
```

**Component Breakdown**

| Middleware | Purpose |
|------------|---------|
| `RateLimited` | Limits job execution rate. |
| `RateLimitedWithRedis` | Redis-optimized rate limiting. |
| `WithoutOverlapping` | Prevents concurrent execution by key. |
| `ThrottlesExceptions` | Delays retries after repeated exceptions. |
| `Skip` | Conditionally skips the job. |

**Syntax Rules**

- Middleware are returned from `middleware()`.
- `WithoutOverlapping` requires a lock-supporting cache driver.
- `RateLimited` requires a named limiter defined via `RateLimiter::for()`.

**Constraints and Limitations**

- Middleware that releases the job back to the queue still increments the attempt count.
- `WithoutOverlapping` locks can expire; use `expireAfter()` to set a lock TTL.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Preventing Overlap**

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

**Expected Output**

Only one `UpdateCreditScore` job for the same user can run at a time; overlapping jobs are released back to the queue.

**Why This Output Occurs**

`WithoutOverlapping` acquires an atomic lock keyed by the user ID; if the lock is held, the job is released.

---

**Example 2: Throttling Exceptions**

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

public function middleware(): array
{
    return [new ThrottlesExceptions(10, 5 * 60)];
}

public function retryUntil(): DateTime
{
    return now()->addMinutes(30);
}
```

**Expected Output**

After 10 exceptions, the job is delayed for 5 minutes before the next attempt; attempts stop after 30 minutes.

**Why This Output Occurs**

`ThrottlesExceptions` counts exceptions and delays retries until the interval elapses.

---

#### Real-World Cases with Explanation

**Case 1: Third-Party API Integration**

A job that calls a flaky third-party API uses `RateLimited` to stay within the API’s rate limits and `ThrottlesExceptions` to back off after repeated failures.

**Explanation**: Middleware makes fragile integrations resilient without cluttering the job’s `handle()` method.

---

### 7. Job Rate Limiting

#### Definitions

**Core Definition**

Job rate limiting constrains how frequently a job (or group of jobs) can be executed, typically to stay within API quotas or to avoid overwhelming a resource.

**Technical Definition**

Rate limiting is implemented via the `RateLimited` middleware, which uses a named limiter defined with `RateLimiter::for()`. The limiter returns a `Limit` object specifying the maximum attempts per time window and an optional key (e.g., user ID). When the limit is exceeded, the middleware releases the job back to the queue with an appropriate delay. `RateLimitedWithRedis` provides a more efficient Redis-specific implementation.

**Beginner-Friendly Explanation**

Rate limiting means “don’t run this job too often.” If you are calling an API that allows 60 requests per minute, you use rate limiting to ensure your jobs stay within that limit.

#### Purposes

- To respect third-party API rate limits.
- To prevent overwhelming internal or external resources.
- To ensure fair usage across tenants or users.
- To avoid service bans or throttling from providers.

#### Syntax Rules and Structure

**Complete General Syntax**

**Define the limiter**

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('backups', function (object $job) {
        return $job->user->isVip()
            ? Limit::none()
            : Limit::perHour(1)->by($job->user->id);
    });
}
```

**Attach the middleware**

```php
use Illuminate\Queue\Middleware\RateLimited;

public function middleware(): array
{
    return [new RateLimited('backups')];
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `RateLimiter::for()` | Defines a named limiter. |
| `Limit::perHour()` / `perMinute()` | Sets the time window and max attempts. |
| `->by()` | Segments the limit by a key. |
| `RateLimited('name')` | Applies the limiter to the job. |

**Syntax Rules**

- The limiter name must match the middleware argument.
- `releaseAfter()` sets the delay before retry.
- `dontRelease()` prevents retries when rate limited.

**Constraints and Limitations**

- Releasing a rate-limited job increments its attempt count.
- `RateLimitedWithRedis` requires Redis.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Hourly Backup Limit**

```php
RateLimiter::for('backups', function ($job) {
    return Limit::perHour(1)->by($job->user->id);
});
```

```php
public function middleware(): array
{
    return [new RateLimited('backups')];
}
```

**Expected Output**

Each user can run one backup job per hour; additional dispatches are released back to the queue.

**Why This Output Occurs**

The limiter keys by user ID, and the middleware releases jobs that exceed the limit.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant API Integration**

A multi-tenant application rate-limits outbound API calls per tenant using `Limit::perMinute(50)->by($tenant->id)`, ensuring no single tenant exhausts the shared API quota.

**Explanation**: Rate limiting prevents noisy-neighbour problems in shared infrastructure.

---

### 8. Job Retries

#### Definitions

**Core Definition**

Job retries are the automatic re-attempts of a failed job, controlled by `$tries`, `--tries`, `$backoff`, `$maxExceptions`, and `retryUntil()`.

**Technical Definition**

A job’s maximum attempts can be defined via the `$tries` property (or the `--tries` worker option). The delay between attempts is controlled by `$backoff` (seconds or an array of escalating delays). `retryUntil()` specifies an absolute deadline after which no further attempts are made. `$maxExceptions` limits the number of exceptions that can occur regardless of attempts. Jobs that fail due to exceeding attempts throw `MaxAttemptsExceededException`; timeouts throw `TimeoutExceededException`.

**Beginner-Friendly Explanation**

Retries mean “try again if it fails.” You can set how many times to retry, how long to wait between tries, and when to give up.

#### Purposes

- To recover from transient failures (network blips, temporary outages).
- To avoid permanent failure from one-off errors.
- To control the cost and latency of retries with backoff strategies.
- To set absolute deadlines for time-sensitive jobs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public int $tries = 3;
public int $backoff = 10;
public array $backoff = [10, 30, 60]; // escalating delays
public int $maxExceptions = 2;

public function retryUntil(): DateTime
{
    return now()->addMinutes(30);
}
```

**Component Breakdown**

| Mechanism | Description |
|-----------|-------------|
| `$tries` | Maximum attempts. |
| `--tries=N` | Worker-level override. |
| `$backoff` | Seconds (or array) between retries. |
| `retryUntil()` | Absolute deadline. |
| `$maxExceptions` | Max exceptions before failure. |

**Syntax Rules**

- `--tries` overrides `$tries` for that worker.
- `retryUntil()` takes precedence over `$tries`.
- `$backoff` can be a single integer or an array of delays.

**Constraints and Limitations**

- Infinite retries are possible with `$tries = 0` or a far-future `retryUntil()`.
- `$maxExceptions` counts only exceptions, not timeouts or worker crashes (unless configured).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Escalating Backoff**

```php
public int $tries = 3;
public array $backoff = [10, 60, 300];
```

**Expected Output**

After the first failure, retry after 10 seconds; after the second, 60 seconds; after the third, 300 seconds.

**Why This Output Occurs**

The `$backoff` array provides per-attempt delays, allowing more time for transient issues to resolve.

---

**Example 2: Time-Based Attempts**

```php
public function retryUntil(): DateTime
{
    return now()->addMinutes(30);
}
```

**Expected Output**

The job is retried until 30 minutes have passed since it was first dispatched, regardless of attempt count.

**Why This Output Occurs**

`retryUntil()` sets an absolute deadline; the worker checks this before each attempt.

---

#### Real-World Cases with Explanation

**Case 1: Payment Gateway Integration**

A payment job retries three times with escalating backoff (5s, 30s, 2m). This handles temporary gateway outages without retrying a permanently declined card indefinitely.

**Explanation**: Escalating backoff balances responsiveness with resilience.

---

### 9. Job Timeouts

#### Definitions

**Core Definition**

A job timeout is the maximum number of seconds a job is allowed to run before being killed by the worker.

**Technical Definition**

Timeouts are configured via the `$timeout` property on the job or the `--timeout` worker option. When a job exceeds its timeout, the worker kills the process and the job is marked as failed with a `TimeoutExceededException`. The `$failOnTimeout = true` property forces a timeout to be treated as a failure rather than a retryable event. Timeouts must be shorter than the queue connection’s `retry_after` setting to avoid duplicate execution.

**Beginner-Friendly Explanation**

A timeout is a “you have this many seconds to finish” rule. If the job takes longer, it is stopped and marked as failed.

#### Purposes

- To prevent a single job from blocking a worker indefinitely.
- To avoid runaway jobs consuming resources.
- To enforce service-level agreements for job completion.
- To fail fast when a job is stuck.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public int $timeout = 120;
public bool $failOnTimeout = true;
```

**Component Breakdown**

| Property | Description |
|----------|-------------|
| `$timeout` | Maximum seconds the job can run. |
| `$failOnTimeout` | Treat timeout as failure (no retry). |

**Syntax Rules**

- `$timeout` must be less than `retry_after`.
- `--timeout` on the worker overrides the job’s timeout.
- `$failOnTimeout = true` prevents retries on timeout.

**Constraints and Limitations**

- Timeouts require the `pcntl` extension for signal-based termination.
- A timed-out job may leave partial side effects.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Timeout with Fail on Timeout**

```php
public int $timeout = 60;
public bool $failOnTimeout = true;
```

**Expected Output**

The job is killed after 60 seconds and marked as failed without retrying.

**Why This Output Occurs**

`$failOnTimeout = true` instructs the worker to treat the timeout as a permanent failure.

---

#### Real-World Cases with Explanation

**Case 1: Report Generation**

A report-generation job is capped at 120 seconds. If it exceeds that limit, it fails and is logged, preventing a runaway report from blocking other jobs.

**Explanation**: Timeouts protect worker capacity from unresponsive jobs.

---

### 10. Job Failures

#### Definitions

**Core Definition**

A job failure occurs when a job exhausts all allowed attempts, throws an unhandled exception, or times out (if `failOnTimeout` is set); failed jobs are recorded in the `failed_jobs` table for inspection and retry.

**Technical Definition**

When a job fails permanently, Laravel invokes its `failed()` method (if defined) and records the job in the `failed_jobs` table. The `failed()` method receives the exception (or `MaxAttemptsExceededException`/`TimeoutExceededException`). Failed jobs can be listed (`queue:failed`), retried (`queue:retry`), forgotten (`queue:forget`), or flushed (`queue:flush`). Failed jobs can be pruned with `queue:prune-failed`.

**Beginner-Friendly Explanation**

A failure is the “gave up” state. After all retries are exhausted, the job is set aside in a special list so you can inspect it later and decide whether to try again.

#### Purposes

- To capture and preserve failed jobs for debugging.
- To allow manual retry after fixing the underlying issue.
- To prevent silent data loss when jobs cannot complete.
- To provide an audit trail of failures.

#### Syntax Rules and Structure

**Complete General Syntax**

**Job failed method**

```php
public function failed(?Throwable $exception): void
{
    // Notify the user, log the failure, etc.
}
```

**Failed job commands**

```bash
php artisan queue:failed
php artisan queue:retry {id}
php artisan queue:retry all
php artisan queue:forget {id}
php artisan queue:flush
php artisan queue:prune-failed --hours=48
```

**Component Breakdown**

| Command | Description |
|---------|-------------|
| `queue:failed` | Lists failed jobs. |
| `queue:retry` | Retries a failed job. |
| `queue:forget` | Deletes a failed job. |
| `queue:flush` | Deletes all failed jobs. |
| `queue:prune-failed` | Prunes old failed jobs. |

**Syntax Rules**

- The `failed_jobs` table must exist (`queue:failed-table` + `migrate`).
- `failed()` receives the exception that caused the failure.
- `queue:retry` re-dispatches the job with a fresh attempt count.

**Constraints and Limitations**

- Failed jobs are not automatically retried; manual intervention is required.
- The `failed()` method receives a fresh instance; state from `handle()` is lost.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Handling Failure**

```php
public function failed(?Throwable $exception): void
{
    Log::error("Job failed: {$exception->getMessage()}");
    $this->user->notify(new JobFailedNotification($exception));
}
```

**Expected Output**

When the job fails permanently, the exception is logged and the user is notified.

**Why This Output Occurs**

The `failed()` method is invoked by the worker when the job exhausts its attempts or throws an unhandled exception.

---

**Example 2: Retrying Failed Jobs**

```bash
php artisan queue:failed
# ID: 1, class: App\Jobs\SendEmail

php artisan queue:retry 1
```

**Expected Output**

The failed job is re-dispatched onto its original queue.

**Why This Output Occurs**

`queue:retry` retrieves the payload from `failed_jobs` and pushes it back onto the queue.

---

#### Real-World Cases with Explanation

**Case 1: Post-Deployment Recovery**

A bug causes all invoice jobs to fail. The team fixes the bug, deploys, and runs `php artisan queue:retry all` to reprocess the failed invoices without manual data entry.

**Explanation**: Failed job storage enables recovery from systemic failures after a fix is deployed.

---

### 11. Job Events

#### Definitions

**Core Definition**

Job events are lifecycle events dispatched by the queue system at key points in a job’s processing: queued, processing, processed, failed, timed out, and others.

**Technical Definition**

Laravel dispatches the following queue events: `JobQueued` (job added to queue), `JobProcessing` (before execution), `JobProcessed` (successful completion), `JobFailed` (permanent failure), `JobTimedOut` (timeout), `JobExceptionOccurred` (any exception), `JobRetryRequested`, `Looping`, `QueueBusy`, and `WorkerStopping`. These events can be listened to in a service provider for monitoring, logging, or custom side effects.

**Beginner-Friendly Explanation**

Job events are notifications that fire at each stage of a job’s life. You can listen for them to log activity, send alerts, or trigger other actions.

#### Purposes

- To monitor job lifecycle and performance.
- To log job outcomes for auditing or debugging.
- To trigger alerts on failures or timeouts.
- To integrate with external monitoring systems.

#### Syntax Rules and Structure

**Complete General Syntax**

**Listening in a service provider**

```php
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobFailed;
use Illuminate\Support\Facades\Event;

public function boot(): void
{
    Event::listen(JobProcessed::class, function (JobProcessed $event) {
        Log::info("Job processed: {$event->job->resolveName()}");
    });

    Event::listen(JobFailed::class, function (JobFailed $event) {
        Log::error("Job failed: {$event->job->resolveName()}", [
            'exception' => $event->exception,
        ]);
    });
}
```

**Component Breakdown**

| Event | When Fired |
|-------|------------|
| `JobQueued` | Job added to the queue. |
| `JobProcessing` | Before job execution. |
| `JobProcessed` | After successful execution. |
| `JobFailed` | After permanent failure. |
| `JobTimedOut` | When a job times out. |
| `JobExceptionOccurred` | When an exception occurs. |

**Syntax Rules**

- Events are dispatched by the queue worker.
- Listeners receive the event with the job instance and related data.
- Events can be listened to globally or per-queue.

**Constraints and Limitations**

- Event listeners run synchronously unless queued.
- Some events (e.g., `JobProcessing`) fire for every attempt, not just the final one.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Logging Failed Jobs**

```php
Event::listen(JobFailed::class, function (JobFailed $event) {
    Log::error('Queue job failed', [
        'job' => $event->job->resolveName(),
        'connection' => $event->connectionName,
        'exception' => $event->exception->getMessage(),
    ]);
});
```

**Expected Output**

When a job fails permanently, an error log entry is created with the job name and exception message.

**Why This Output Occurs**

The `JobFailed` event is dispatched by the worker when the job is marked as failed.

---

#### Real-World Cases with Explanation

**Case 1: Queue Monitoring Dashboard**

An application listens for `JobProcessed`, `JobFailed`, and `JobTimedOut` events to feed a real-time queue monitoring dashboard, displaying throughput and failure rates.

**Explanation**: Job events provide the raw data for observability tooling.

---

### 12. Job Testing

#### Definitions

**Core Definition**

Job testing is the practice of verifying that jobs are dispatched correctly, contain the right data, and behave as expected when executed, using Laravel’s queue fakes and job-specific assertions.

**Technical Definition**

Jobs are tested with `Queue::fake()` (for dispatch assertions) or `Bus::fake()` (for chains and batches). Assertions include `Queue::assertPushed()`, `Queue::assertPushedWithChain()`, `Queue::assertPushedWithoutChain()`, `Bus::assertBatched()`, `Bus::assertChained()`, and `Bus::assertDispatched()`. Jobs can be executed directly (`$job->handle()`) for integration-style testing. Laravel 13 introduced `assertFailedWith()` for precise failure testing.

**Beginner-Friendly Explanation**

Testing a job means checking two things: first, that your code dispatches the job when it should; second, that the job itself does the right work when it runs.

#### Purposes

- To verify that jobs are dispatched with the correct payload.
- To test chains and batches without executing them.
- To execute jobs directly and assert on their side effects.
- To test failure conditions precisely with `assertFailedWith()`.

#### Syntax Rules and Structure

**Complete General Syntax**

**Dispatch testing**

```php
use Illuminate\Support\Facades\Queue;

Queue::fake();

// ... code that dispatches ...

Queue::assertPushed(SendWelcomeEmail::class, function ($job) use ($user) {
    return $job->user->is($user);
});

Queue::assertNotPushed(SendWelcomeEmail::class);
```

**Chain testing**

```php
use Illuminate\Support\Facades\Bus;

Bus::fake();

Bus::chain([new ProcessPodcast, new OptimizePodcast])->dispatch();

Bus::assertChained([
    ProcessPodcast::class,
    OptimizePodcast::class,
]);
```

**Batch testing**

```php
Bus::fake();

Bus::batch([new ProcessImage('a.jpg'), new ProcessImage('b.jpg')])->dispatch();

Bus::assertBatched(function ($batch) {
    return $batch->jobs->count() === 2;
});
```

**Failure testing**

```php
$job = (new ProcessDocument)->withFakeQueueInteractions();
$job->handle();
$job->assertFailedWith(ProcessingException::class);
```

**Component Breakdown**

| Method | Purpose |
|--------|---------|
| `Queue::fake()` | Fakes job dispatching. |
| `Bus::fake()` | Fakes chains and batches. |
| `Queue::assertPushed()` | Asserts a job was dispatched. |
| `Bus::assertChained()` | Asserts a job chain. |
| `Bus::assertBatched()` | Asserts a batch was dispatched. |
| `assertFailedWith()` | Asserts a specific failure. |

**Syntax Rules**

- Call `Queue::fake()` or `Bus::fake()` before the code under test.
- Use `Queue::fake()` for simple dispatches; `Bus::fake()` for chains and batches.
- `assertPushed()` accepts a class name and an optional closure.
- `withFakeQueueInteractions()` enables failure assertions on a job instance.

**Constraints and Limitations**

- Fakes prevent real execution; side effects must be tested separately.
- `Queue::fake()` does not intercept `Bus::chain()` or `Bus::batch()`; use `Bus::fake()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Asserting Dispatch**

```php
Queue::fake();

$user = User::factory()->create();
$service = new RegistrationService();
$service->register($user);

Queue::assertPushed(SendWelcomeEmail::class, function ($job) use ($user) {
    return $job->user->id === $user->id;
});
```

**Expected Output**

The test passes if `RegistrationService` dispatched `SendWelcomeEmail` for the correct user.

**Why This Output Occurs**

`Queue::fake()` records dispatched jobs without executing them, and `assertPushed()` verifies the class and payload.

---

**Example 2: Testing a Job’s Logic**

```php
$user = User::factory()->create();
$job = new SendWelcomeEmail($user);
$job->handle();

Mail::assertSent(WelcomeEmail::class, function ($mail) use ($user) {
    return $mail->hasTo($user->email);
});
```

**Expected Output**

The test passes if the job’s `handle()` method sends the welcome email to the correct user.

**Why This Output Occurs**

The job is instantiated and executed directly, and `Mail::fake()` captures the email for assertion.

---

**Example 3: Testing Failure**

```php
Http::fake(['*' => Http::response([], 429)]);

$job = (new SyncExternalDataJob($endpoint))->withFakeQueueInteractions();
$job->handle();

$job->assertFailedWith(RateLimitException::class);
```

**Expected Output**

The test passes if the job fails with a `RateLimitException` when the API returns 429.

**Why This Output Occurs**

`assertFailedWith()` verifies the exact exception class and message.

---

#### Real-World Cases with Explanation

**Case 1: Registration Flow**

A feature test uses `Queue::fake()` to assert that the registration controller dispatches a `SendWelcomeEmail` job with the correct user. A separate unit-style test executes the job directly and asserts that the email is sent. Together, they verify both the dispatch and the job’s behaviour.

**Explanation**: Separating dispatch testing from execution testing keeps tests focused and fast.

---

### 13. Job Best Practices

#### Definitions

**Core Definition**

Job best practices are the guidelines that keep queued jobs reliable, idempotent, observable, and maintainable in production.

#### Key Best Practices

1. **Keep Jobs Small and Focused**: Each job should do one thing. Extract orchestration into chains or batches.

2. **Make Jobs Idempotent**: Jobs may be retried; use `updateOrCreate`, `firstOrCreate`, or explicit idempotency keys to avoid duplicate side effects.

3. **Use `WithoutRelations()` or `#[WithoutRelations]`**: Prevent large relationship payloads from being serialized onto the queue.

4. **Set `$timeout` Less Than `retry_after`**: Prevents duplicate execution when a job runs longer than the queue’s reservation window.

5. **Use `$failOnTimeout = true` for Time-Sensitive Jobs**: Avoid retrying jobs that are fundamentally stuck.

6. **Use `afterCommit` for Transactional Dispatch**: When dispatching inside a database transaction, set `after_commit => true` on the connection or use `->afterCommit()` to prevent workers from processing rolled-back data.

7. **Prefer `Bus::chain()` for Sequential Workflows**: Chains stop on failure and support `catch()` callbacks.

8. **Use `Bus::batch()` for Independent Parallel Work**: Batches provide progress tracking, cancellation, and completion callbacks.

9. **Use `WithoutOverlapping` and `ShouldBeUnique`**: Prevent concurrent or duplicate execution when it would cause harm.

10. **Use `RateLimited` for External API Calls**: Respect third-party quotas and avoid service bans.

11. **Test Dispatch and Execution Separately**: Use `Queue::fake()` / `Bus::fake()` for dispatch tests; execute `handle()` directly for logic tests.

12. **Monitor Failed Jobs and Events**: Listen for `JobFailed` and regularly inspect `queue:failed`; prune old records with `queue:prune-failed`.

13. **Restart Workers on Deploy**: Run `php artisan queue:restart` so workers pick up new code.

14. **Use Horizon (Redis) for Observability**: Horizon provides a dashboard, metrics, and advanced worker configuration.

---

## References

- Queues — Laravel 13.x Documentation — https://laravel.com/docs/13.x/queues
- Queues — Laravel 12.x Documentation — https://laravel.com/docs/12.x/queues
- Queues — Laravel 11.x Documentation — https://laravel.com/docs/11.x/queues
- Jobs + Queue (Laravel/CraftCMS docs) — https://github.com/craftcms/docs/blob/7a3a5fb25791f28716e6949f0235915f1a17f056/docs/6.x/extend/queue.md
- Job Chaining and Batching — https://laravel.com/framework/docs/11.x/queues#4
- Job Middleware and Rate Limiting — https://laravel.com/framework/docs/12.x/queues#2
- Retrying Failed Jobs — https://laravel.com/framework/docs/queues#7
- Test Job Failures Precisely with `assertFailedWith` — https://laravel-news.com/index.php/assertfailedwith
- Monitoring Laravel Queued Jobs with Lifecycle Events — https://betalectic.com
- Queue::fake / Bus::fake — Testing Dispatch Behaviour — https://github.com/voku/agent-skills/blob/main/skills/laravel-queues/rules/ops-queue-fake.md
- Simplified Batch Job Creation with Laravel’s Enhanced Artisan Command — https://laravel-news.com
- Laravel 13.34: Count Worker Crashes as Job Exceptions — https://laravel-news.com
- Job Middleware (WithoutOverlapping, ThrottlesExceptions) — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-queues/references/middleware.md

---

**Version Note**: This cheat sheet is based on Laravel 11.x through 13.x documentation. Job chaining, batching, middleware, rate limiting, retries, timeouts, failures, events, and testing are stable across these versions. `assertFailedWith()` was introduced in Laravel 13. The `--batched` flag for `make:job` is available in Laravel 12+. PHP attributes (`#[WithoutRelations]`, `#[FailOnTimeout]`) are available in Laravel 12+. Always consult the documentation for your specific Laravel version.