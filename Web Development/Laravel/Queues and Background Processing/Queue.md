# Laravel Queue Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Queues provide a unified API for deferring time-consuming tasks — such as sending emails, processing uploads, or calling external APIs — to background workers, allowing web requests to respond quickly while heavy work is processed asynchronously.

**Technical Definition:** Laravel's queue system is built around the `Illuminate\Queue\QueueManager` class, which resolves queue connections from `config/queue.php` into driver-specific `Queue` implementations (`SyncQueue`, `DatabaseQueue`, `RedisQueue`, `SqsQueue`, `BeanstalkdQueue`). Jobs are PHP classes implementing `ShouldQueue` that are serialized and pushed onto a queue backend via the `Illuminate\Bus\Dispatcher`. A queue worker (`queue:work` Artisan command) polls the backend, deserializes the job payload, resolves the job class from the service container, and invokes its `handle()` method. The `after_commit` configuration option and the `afterCommit()` / `beforeCommit()` dispatch methods ensure jobs are not processed before parent database transactions commit.

**Beginner-Friendly Explanation:** When your application needs to do something slow — like sending a welcome email or processing a video — you don't want to make the user wait. Laravel Queues let you say "do this later" by pushing a job onto a queue. A background worker picks up the job and processes it while your application continues serving requests. It's like having a to-do list for your server: you write down what needs to happen, and a worker checks off the items one by one.

### Key Characteristics

- **Unified API:** The same `dispatch()` method works across all queue drivers (sync, database, Redis, SQS, Beanstalkd).
- **Driver-agnostic:** Switching queue backends requires only configuration changes, not code changes.
- **Serialization bridge:** Jobs are serialized into a payload that can be stored in any backend and deserialized by any worker.
- **Transaction awareness:** Jobs can be deferred until database transactions commit, preventing race conditions.
- **Retry and failure handling:** Jobs support `$tries`, `$backoff`, `$timeout`, and a `failed()` method.
- **Conditional dispatching:** `dispatchIf()` and `dispatchUnless()` enable conditional job dispatch.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with `config/queue.php` present.
- For Redis: the `predis/predis` package or the `phpredis` extension.
- For SQS: the `aws/aws-sdk-php` package and AWS credentials.
- For Beanstalkd: a running Beanstalkd server and the `pda/pheanstalk` package.
- For database: the `jobs` table migration (`php artisan queue:table && php artisan migrate`).

### Related Programming Areas

- **Artisan Console** — Queue workers are started and managed via Artisan commands.
- **Service Container** — Jobs are resolved from the container, enabling dependency injection.
- **Database Transactions** — `after_commit` ensures jobs are dispatched only after transactions commit.
- **Events & Listeners** — Queued listeners are dispatched as jobs.
- **Notifications & Mail** — Queued mailables and notifications use the queue system.
- **Horizon** — Provides a dashboard and auto-scaling for Redis queues.

### Core Concepts / Features

1. Queue Drivers & Connections (Configuring environments for sync, database, Redis, SQS, and Beanstalkd)
2. The Background Lifecycle (Understanding how serialization bridges HTTP request states over into terminal background runners)
3. Dispatching Variations (Utilizing `dispatch()`, `dispatchSync()`, closures, and conditional dispatching via `dispatchIf()` or `dispatchUnless()`)
4. Database Transaction Coupling (Preventing premature worker reads by delaying job dispatch until open database transactions are fully committed)

---

## 1. Queue Drivers & Connections

### Definitions

**Core Definition:** Queue drivers are the backend implementations that store and retrieve queued jobs, while connections are named configurations in `config/queue.php` that bind a driver to its credentials and options.

**Technical Definition:** The `config/queue.php` file defines a `default` connection key (controlled by the `QUEUE_CONNECTION` environment variable) and a `connections` array where each key is a connection name and each value is a configuration array with a `driver` key and driver-specific options. Laravel supports the following built-in drivers: `sync` (immediate execution), `database` (relational database table), `redis` (Redis lists), `sqs` (Amazon Simple Queue Service), `beanstalkd` (Beanstalkd work queue), `null` (discards jobs), and `deferred` / `background` (Laravel 12+). Each driver has distinct characteristics suited to different scenarios.

**Beginner-Friendly Explanation:** Think of a queue connection as a "storage profile" for your jobs. You might use the `sync` driver during development (jobs run immediately), the `database` driver for simple production setups (jobs are stored in a database table), and `redis` or `sqs` for high-volume production workloads. Laravel lets you define multiple connections and switch between them by changing one environment variable.

### Purposes

- To provide a unified API across multiple queue backends, eliminating code changes when switching drivers.
- To configure different queue backends for different environments (sync in development, Redis in production).
- To support high-volume production workloads with scalable backends like Redis and SQS.
- To enable simple setups with the database driver for applications that already have a relational database.
- To provide a null driver for testing scenarios where jobs should be discarded.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// config/queue.php

return [
    'default' => env('QUEUE_CONNECTION', 'database'),

    'connections' => [
        'sync' => [
            'driver' => 'sync',
        ],

        'database' => [
            'driver'      => 'database',
            'connection'  => env('DB_QUEUE_CONNECTION'),
            'table'       => env('DB_QUEUE_TABLE', 'jobs'),
            'queue'       => env('DB_QUEUE', 'default'),
            'retry_after' => (int) env('DB_QUEUE_RETRY_AFTER', 90),
            'after_commit' => false,
        ],

        'redis' => [
            'driver'      => 'redis',
            'connection'  => env('REDIS_QUEUE_CONNECTION', 'default'),
            'queue'       => env('REDIS_QUEUE', 'default'),
            'retry_after' => (int) env('REDIS_QUEUE_RETRY_AFTER', 90),
            'block_for'   => null,
            'after_commit' => false,
        ],

        'sqs' => [
            'driver'  => 'sqs',
            'key'     => env('AWS_ACCESS_KEY_ID'),
            'secret'  => env('AWS_SECRET_ACCESS_KEY'),
            'prefix'  => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
            'queue'   => env('SQS_QUEUE', 'default'),
            'suffix'  => env('SQS_SUFFIX'),
            'region'  => env('AWS_DEFAULT_REGION', 'us-east-1'),
            'after_commit' => false,
        ],

        'beanstalkd' => [
            'driver'      => 'beanstalkd',
            'host'        => env('BEANSTALKD_QUEUE_HOST', 'localhost'),
            'queue'       => env('BEANSTALKD_QUEUE', 'default'),
            'retry_after' => (int) env('BEANSTALKD_QUEUE_RETRY_AFTER', 90),
            'block_for'   => 0,
            'after_commit' => false,
        ],

        'null' => [
            'driver' => 'null',
        ],
    ],
];
```

**Component Breakdown:**

- `'default'` — The connection used when no explicit connection is specified. Controlled by `QUEUE_CONNECTION`.
- `'driver' => 'sync'` — Jobs execute immediately in the same process (development/testing).
- `'driver' => 'database'` — Jobs are stored in a database table (simple production setups).
- `'driver' => 'redis'` — Jobs are stored in Redis lists (high-performance production).
- `'driver' => 'sqs'` — Jobs are stored in Amazon SQS (AWS-native, fully managed).
- `'driver' => 'beanstalkd'` — Jobs are stored in a Beanstalkd server (specialised queuing).
- `'driver' => 'null'` — Jobs are discarded (testing).
- `'after_commit'` — When `true`, jobs are not dispatched until open database transactions commit.
- `'retry_after'` — Seconds after which a job is considered failed if not completed.
- `'block_for'` — Seconds to block waiting for a job (Redis/Beanstalkd).

**Syntax Rules:**

- The `default` key must match one of the keys in the `connections` array.
- The `after_commit` option is available on all queued connections.
- The `retry_after` option should be longer than the longest job's execution time.
- Redis requires either the `phpredis` extension or the `predis/predis` package.
- SQS requires the `aws/aws-sdk-php` package.
- Beanstalkd requires the `pda/pheanstalk` package.
- The `database` driver requires a `jobs` table.

**Constraints and Limitations:**

- **The `sync` driver blocks the request.** It is not suitable for production.
- **The `database` driver is not suitable for high-volume workloads.** It uses the same database as your application, which can cause contention.
- **Redis and SQS have message size limits.** SQS limits messages to 256 KB; Redis has a 512 MB limit but practical limits are much lower.
- **The `null` driver discards jobs silently.** Use it only for testing.
- **`retry_after` must be longer than the job's `timeout`.** If a job times out but `retry_after` has already passed, the job may be retried while still running.

### Annotated Code Examples

**Example 1: Configuring Multiple Queue Connections**

```php
// .env file
QUEUE_CONNECTION=redis

REDIS_QUEUE_CONNECTION=default
REDIS_QUEUE=default
REDIS_QUEUE_RETRY_AFTER=90

DB_QUEUE_CONNECTION=mysql
DB_QUEUE_TABLE=jobs
DB_QUEUE=default
DB_QUEUE_RETRY_AFTER=90

AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_DEFAULT_REGION=us-east-1
SQS_PREFIX=https://sqs.us-east-1.amazonaws.com/123456789012
SQS_QUEUE=default
```

```php
// config/queue.php
'default' => env('QUEUE_CONNECTION', 'redis'),

'connections' => [
    'sync' => [
        'driver' => 'sync',
    ],

    'database' => [
        'driver'      => 'database',
        'connection'  => env('DB_QUEUE_CONNECTION'),
        'table'       => env('DB_QUEUE_TABLE', 'jobs'),
        'queue'       => env('DB_QUEUE', 'default'),
        'retry_after' => (int) env('DB_QUEUE_RETRY_AFTER', 90),
        'after_commit' => false,
    ],

    'redis' => [
        'driver'      => 'redis',
        'connection'  => env('REDIS_QUEUE_CONNECTION', 'default'),
        'queue'       => env('REDIS_QUEUE', 'default'),
        'retry_after' => (int) env('REDIS_QUEUE_RETRY_AFTER', 90),
        'block_for'   => null,
        'after_commit' => false,
    ],

    'sqs' => [
        'driver'  => 'sqs',
        'key'     => env('AWS_ACCESS_KEY_ID'),
        'secret'  => env('AWS_SECRET_ACCESS_KEY'),
        'prefix'  => env('SQS_PREFIX'),
        'queue'   => env('SQS_QUEUE', 'default'),
        'region'  => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'after_commit' => false,
    ],
],
```

**Expected Output:** The application uses Redis as the default queue connection. Jobs dispatched without an explicit connection go to Redis. Jobs dispatched with `onConnection('sqs')` go to SQS.

**Why This Output Occurs:** The `default` key reads `QUEUE_CONNECTION` from the environment, which is set to `redis`. The `connections` array defines three usable backends. Each connection's `driver` key tells the `QueueManager` which driver class to instantiate. The credentials and options are passed to the driver's constructor.

---

**Example 2: Switching to the Sync Driver for Local Development**

```php
// .env (local development)
QUEUE_CONNECTION=sync
```

```php
// config/queue.php
'default' => env('QUEUE_CONNECTION', 'sync'),
```

```php
// Dispatching a job in development
dispatch(new ProcessPodcast($podcast));
// The job runs immediately in the same process.
```

**Expected Output:** Jobs execute immediately in the same process as the dispatch. No queue worker is required.

**Why This Output Occurs:** The `sync` driver executes jobs inline when dispatched. This is ideal for local development because it allows you to debug job logic without running a queue worker. In production, the `QUEUE_CONNECTION` is set to `redis` or `sqs`, and the same dispatch code pushes jobs to the queue for background processing.

### Real-World Cases

- **Small applications:** Use the `database` driver to avoid additional infrastructure. Jobs are stored in the `jobs` table and processed by a worker.
- **High-volume applications:** Use the `redis` driver for low-latency, high-throughput job processing, often paired with Laravel Horizon for monitoring and auto-scaling.
- **AWS-native applications:** Use the `sqs` driver to leverage Amazon's fully managed queue service, with automatic scaling and built-in dead-letter queue support.
- **Development and testing:** Use the `sync` driver to execute jobs immediately, simplifying debugging and eliminating the need for a queue worker.
- **Specialised workloads:** Use `beanstalkd` for applications that need a dedicated, high-performance queue server with tube-based priority routing.

---

## 2. The Background Lifecycle

### Definitions

**Core Definition:** The background lifecycle is the complete journey of a queued job — from dispatch in the HTTP request process, through serialization and storage in the queue backend, to deserialization and execution in a separate worker process.

**Technical Definition:** When a job is dispatched, the `Illuminate\Bus\Dispatcher` serializes the job class and its constructor arguments into a JSON payload using Laravel's `Serializer`. The `SerializesModels` trait replaces Eloquent model instances with `ModelIdentifier` objects (class name + primary key) to avoid serializing entire models. The serialized payload is pushed to the queue backend (Redis list, database table, SQS queue, etc.). A queue worker (`php artisan queue:work`) runs in a separate process, polls the backend using `BRPOP` (Redis) or equivalent, deserializes the payload, resolves the job class from the service container, and invokes the `handle()` method. The `SerializesModels` trait's `__wakeup()` method re-fetches models from the database, ensuring the worker has fresh data.

**Beginner-Friendly Explanation:** When you dispatch a job, Laravel takes a snapshot of the job and its data, converts it into a format that can be stored (like JSON), and saves it in the queue backend. Later, a separate worker process — which runs independently of your web server — picks up the job, converts it back into a PHP object, and runs its `handle()` method. The `SerializesModels` trait is a clever trick: instead of storing the entire model in the queue (which could become stale), it stores just the model's ID and re-fetches the fresh model when the job runs.

### Purposes

- To decouple job dispatch (in the HTTP request) from job execution (in a background worker).
- To bridge the gap between request-scoped state and long-running background processes through serialization.
- To ensure that jobs can be stored in any queue backend, regardless of the backend's data format.
- To provide fresh model data to workers through `SerializesModels`.
- To enable horizontal scaling by allowing multiple workers to process jobs from the same queue.

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
        // This runs in the worker process
        // $this->podcast is re-fetched from the database
    }
}
```

**Component Breakdown:**

- `Dispatchable` — Provides static `dispatch()`, `dispatchIf()`, `dispatchUnless()`, `dispatchSync()`, `dispatchAfterResponse()`, and `dispatchNow()` (deprecated) methods.
- `InteractsWithQueue` — Provides access to the queue job instance (`$this->job`), enabling `release()`, `delete()`, `fail()`, and other queue methods.
- `Queueable` — Provides `onConnection()`, `onQueue()`, `delay()`, `afterCommit()`, `beforeCommit()`, and `chain()` methods.
- `SerializesModels` — Replaces Eloquent models with identifiers during serialization and re-fetches them during unserialization.

**Syntax Rules:**

- The `SerializesModels` trait must be used if the job contains Eloquent models.
- The `handle()` method is invoked by the worker after deserialization.
- Constructor arguments are serialized; public properties are available in `handle()`.
- Binary data should be base64-encoded before being passed to a queued job.
- The `$podcast` property is re-fetched from the database by `SerializesModels`.

**Constraints and Limitations:**

- **Serialized models may be stale or deleted.** If a model is deleted before the worker runs, the job will fail or receive `null`.
- **Serialization adds overhead.** Large jobs with many properties or large data payloads take longer to serialize and deserialize.
- **Queue message size limits apply.** SQS limits messages to 256 KB; Redis has practical limits based on memory.
- **The worker process does not share state with the HTTP request.** Request-scoped data (session, auth) is not available unless explicitly passed.
- **Binary data must be base64-encoded.** Otherwise, the job may not serialize correctly to JSON.

### Annotated Code Examples

**Example 1: Complete Job Lifecycle**

```php
<?php
// Step 1: Define the job
// File: app/Jobs/SendWelcomeEmail.php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public User $user,
    ) {}

    public function handle(): void
    {
        // Step 4: The worker runs this method
        // $this->user is re-fetched from the database by SerializesModels
        \Notification::send($this->user, new \App\Notifications\WelcomeNotification());
    }
}
```

```php
// Step 2: Dispatch the job (in the HTTP request)
use App\Jobs\SendWelcomeEmail;
use App\Models\User;

$user = User::find(123);
SendWelcomeEmail::dispatch($user);
```

```bash
# Step 3: Run the worker (in a separate terminal or process)
php artisan queue:work
```

**Expected Output:**

- The job is serialized and pushed onto the queue.
- The worker deserializes the job, re-fetches the `User` model from the database, and executes `handle()`.
- The welcome notification is sent to the user.

**Why This Output Occurs:** The `Dispatchable` trait's `dispatch()` method serializes the job using Laravel's `Serializer`. The `SerializesModels` trait's `__sleep()` method replaces the `User` model with a `ModelIdentifier` containing only the ID and class name. The serialized payload is pushed to the queue backend. The worker polls the backend, retrieves the payload, deserializes it (re-fetching the `User` model via `__wakeup()`), resolves the job class from the container, and invokes `handle()`.

---

**Example 2: Inspecting the Serialized Payload**

```php
// Dispatching a job and inspecting the payload
use App\Jobs\SendWelcomeEmail;
use App\Models\User;

$user = User::find(123);
$job = new SendWelcomeEmail($user);

// Serialize the job (this is what happens during dispatch)
$serialized = serialize($job);

// The serialized payload contains the ModelIdentifier, not the full model
// O:30:"App\Jobs\SendWelcomeEmail":1:{s:4:"user";O:38:"Illuminate\Queue\SerializesModels\..."
```

**Expected Output:** The serialized payload contains a `ModelIdentifier` object instead of the full `User` model. This keeps the payload small and ensures the worker gets a fresh model.

**Why This Output Occurs:** The `SerializesModels` trait's `__sleep()` method inspects the job's properties, identifies Eloquent models, and replaces them with `ModelIdentifier` instances containing only the model's class name and primary key. When the worker unserializes the payload, `__wakeup()` uses the `ModelIdentifier` to re-fetch the model from the database.

### Real-World Cases

- **Email sending:** Jobs are serialized with the user ID and re-fetched in the worker, ensuring the email is sent to the current user record.
- **Video processing:** A job is dispatched with the video ID and re-fetched in the worker, allowing the worker to access the video file from storage.
- **Report generation:** A job is dispatched with the report parameters (date range, filters) and generates the report in the worker without blocking the request.
- **Third-party API sync:** A job is dispatched with the customer ID and syncs the customer's data to the CRM in the worker.
- **Image resizing:** A job is dispatched with the image ID and generates multiple sizes in the worker.

---

## 3. Dispatching Variations

### Definitions

**Core Definition:** Dispatching variations are the different methods available for pushing jobs onto the queue, including standard asynchronous dispatch, synchronous dispatch, conditional dispatch, and dispatch-after-response.

**Technical Definition:** The `Dispatchable` trait provides the following dispatch methods: `dispatch(...$arguments)` — asynchronous dispatch to the queue; `dispatchIf(bool $boolean, ...$arguments)` — conditional asynchronous dispatch; `dispatchUnless(bool $boolean, ...$arguments)` — inverse conditional dispatch; `dispatchSync(...$arguments)` — synchronous dispatch in the current process (bypasses the queue); `dispatchAfterResponse(...$arguments)` — dispatch after the HTTP response is sent; `dispatchNow(...$arguments)` — deprecated alias for `dispatchSync()`. Additionally, the global `dispatch()` helper function and the `Bus` facade provide alternative dispatch mechanisms. Jobs can also be dispatched conditionally using `dispatchIf()` and `dispatchUnless()` with closures or boolean values.

**Beginner-Friendly Explanation:** Laravel gives you several ways to dispatch a job. The normal `dispatch()` puts it on the queue for background processing. `dispatchSync()` runs it immediately, which is useful when you need the result before continuing. `dispatchIf()` and `dispatchUnless()` only dispatch if a condition is true or false. `dispatchAfterResponse()` sends the response to the user first, then runs the job.

### Purposes

- To provide a consistent API for dispatching jobs across all queue drivers.
- To enable conditional dispatching based on application state.
- To support synchronous execution for testing or when the result is needed immediately.
- To improve perceived performance by dispatching jobs after the HTTP response is sent.
- To allow jobs to be dispatched from anywhere in the application (controllers, services, commands).

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Standard asynchronous dispatch
ProcessPodcast::dispatch($podcast);

// Conditional dispatch
ProcessPodcast::dispatchIf($condition, $podcast);
ProcessPodcast::dispatchUnless($condition, $podcast);

// Synchronous dispatch (runs immediately)
ProcessPodcast::dispatchSync($podcast);

// Dispatch after HTTP response is sent
ProcessPodcast::dispatchAfterResponse($podcast);

// Deprecated: use dispatchSync() instead
// ProcessPodcast::dispatchNow($podcast);

// Using the global dispatch() helper
dispatch(new ProcessPodcast($podcast));

// Using the Bus facade
use Illuminate\Support\Facades\Bus;
Bus::dispatch(new ProcessPodcast($podcast));

// Conditional dispatch with a closure
ProcessPodcast::dispatchIf(fn () => $podcast->isReady(), $podcast);
```

**Component Breakdown:**

- `dispatch(...$arguments)` — Asynchronous dispatch to the configured queue connection.
- `dispatchIf(bool $boolean, ...$arguments)` — Dispatches only if the boolean is truthy.
- `dispatchUnless(bool $boolean, ...$arguments)` — Dispatches only if the boolean is falsy.
- `dispatchSync(...$arguments)` — Dispatches to the `sync` queue, executing in the current process.
- `dispatchAfterResponse(...$arguments)` — Dispatches after the HTTP response is sent to the browser.
- `dispatchNow(...$arguments)` — **Deprecated.** Use `dispatchSync()` instead.

**Syntax Rules:**

- The `Dispatchable` trait must be used on the job class for static dispatch methods.
- All arguments after the boolean condition in `dispatchIf()` / `dispatchUnless()` are passed to the job constructor.
- `dispatchSync()` uses the `sync` queue connection regardless of the default connection.
- `dispatchAfterResponse()` requires a FastCGI or similar SAPI to work correctly.
- The global `dispatch()` helper accepts a job instance and dispatches it.

**Constraints and Limitations:**

- **`dispatchSync()` bypasses the queue entirely.** It runs in the current process and blocks until completion.
- **`dispatchAfterResponse()` may not work in all server environments.** It relies on the FastCGI `fastcgi_finish_request()` function.
- **`dispatchNow()` is deprecated.** Use `dispatchSync()` instead.
- **Conditional dispatch does not queue the job for later.** If the condition is false, the job is simply not dispatched.
- **`dispatchIf()` and `dispatchUnless()` accept closures** but the closure is evaluated immediately, not in the worker.

### Annotated Code Examples

**Example 1: Dispatching Variations in Practice**

```php
<?php
// File: app/Http/Controllers/OrderController.php

namespace App\Http\Controllers;

use App\Jobs\ProcessOrder;
use App\Jobs\SendOrderConfirmation;
use App\Models\Order;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        $order = Order::create($request->validated());

        // Standard asynchronous dispatch
        ProcessOrder::dispatch($order);

        // Conditional dispatch: only send confirmation if the customer opted in
        SendOrderConfirmation::dispatchIf(
            $order->customer->wants_confirmation,
            $order
        );

        // Dispatch after response: log the order without delaying the response
        \App\Jobs\LogOrderActivity::dispatchAfterResponse($order);

        return response()->json(['order_id' => $order->id], 201);
    }

    public function syncProcess(Order $order)
    {
        // Synchronous dispatch: process immediately and use the result
        $result = ProcessOrder::dispatchSync($order);

        return response()->json(['result' => $result]);
    }
}
```

**Expected Output:**

- The `ProcessOrder` job is queued for background processing.
- The `SendOrderConfirmation` job is queued only if the customer wants confirmation.
- The `LogOrderActivity` job is dispatched after the HTTP response is sent.
- The `syncProcess` method runs `ProcessOrder` immediately and returns the result.

**Why This Output Occurs:** Each dispatch method has different semantics. `dispatch()` pushes the job to the queue. `dispatchIf()` checks the condition before dispatching. `dispatchAfterResponse()` defers dispatch until after the response is sent. `dispatchSync()` executes the job in the current process and returns its result.

---

**Example 2: Conditional Dispatch with Closures**

```php
use App\Jobs\SendInvoice;
use App\Models\Invoice;

$invoice = Invoice::find(123);

// Dispatch if the invoice is unpaid and past due
SendInvoice::dispatchIf(
    fn () => $invoice->isUnpaid() && $invoice->isPastDue(),
    $invoice
);

// Dispatch unless the invoice is already paid
SendInvoice::dispatchUnless(
    fn () => $invoice->isPaid(),
    $invoice
);
```

**Expected Output:** The job is dispatched only if the closure returns `true` (for `dispatchIf`) or `false` (for `dispatchUnless`). The closure is evaluated at dispatch time.

**Why This Output Occurs:** The `dispatchIf()` and `dispatchUnless()` methods accept a boolean or a closure. If a closure is provided, it is invoked immediately to determine whether to dispatch. This allows complex conditions to be evaluated inline without verbose `if` statements.

### Real-World Cases

- **E-commerce checkout:** Dispatch the order processing job asynchronously, send the confirmation email after the response, and conditionally dispatch a fraud check if the order total exceeds a threshold.
- **User registration:** Dispatch the welcome email asynchronously, log the registration after the response, and conditionally dispatch a CRM sync if the user opted in.
- **Video upload:** Dispatch the transcoding job asynchronously and conditionally dispatch a thumbnail generation job if the video is a certain format.
- **Payment processing:** Dispatch the receipt email after the response and conditionally dispatch a fraud alert if the payment amount is unusual.
- **Report generation:** Dispatch the report generation job synchronously when the user requests an immediate download, or asynchronously when the report is scheduled.

---

## 4. Database Transaction Coupling

### Definitions

**Core Definition:** Database transaction coupling is the practice of ensuring that queued jobs are not dispatched to the queue until the database transaction that triggered them has been fully committed, preventing workers from reading stale or non-existent data.

**Technical Definition:** When a job is dispatched inside a database transaction, the queue worker may pick it up and execute it before the transaction commits. This means any models created or updated within the transaction may not yet exist or may have stale values when the worker reads them. Laravel provides two mechanisms to address this: (1) the `after_commit` configuration option on the queue connection, which defers all jobs (and queued event listeners, mailables, notifications, and broadcasts) until open transactions commit; and (2) the `afterCommit()` and `beforeCommit()` methods on the `PendingDispatch` instance, which allow per-job control. The `ShouldQueueAfterCommit` interface and the `$afterCommit` property on listeners provide additional granularity.

**Beginner-Friendly Explanation:** Imagine you're creating a user record in a database transaction and dispatching a welcome email job inside that same transaction. If the worker picks up the job before the transaction commits, the user record doesn't exist yet — the email job fails. Transaction coupling fixes this by telling Laravel: "Don't dispatch this job until the database transaction is safely committed."

### Purposes

- To prevent queued workers from reading stale or non-existent data.
- To ensure that jobs are not dispatched if the parent transaction rolls back.
- To provide a consistent view of the database for workers.
- To eliminate race conditions between transaction commits and job execution.
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
    'after_commit' => true,
],

// Option 2: Per-job afterCommit()
ProcessPodcast::dispatch($podcast)->afterCommit();

// Option 3: Per-job beforeCommit() (when after_commit is globally true)
ProcessPodcast::dispatch($podcast)->beforeCommit();
```

```php
// Dispatching inside a transaction
use App\Jobs\SendWelcomeEmail;
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    $user = User::create([
        'name'  => 'Jane Doe',
        'email' => 'jane@example.com',
    ]);

    // With after_commit => true, this job is not dispatched
    // until the transaction commits successfully
    SendWelcomeEmail::dispatch($user);
});
```

**Component Breakdown:**

- `'after_commit' => true` — Global connection option that defers all jobs until open transactions commit.
- `->afterCommit()` — Per-job method that defers that specific job until open transactions commit.
- `->beforeCommit()` — Per-job method that dispatches the job immediately, even if `after_commit` is globally `true`.
- The deferred jobs are discarded if the transaction rolls back.

**Syntax Rules:**

- The `after_commit` configuration option applies to all queued jobs, event listeners, mailables, notifications, and broadcasts on that connection.
- The `afterCommit()` method can be chained onto the `dispatch()` call.
- The `beforeCommit()` method can be used to override the global `after_commit` setting for a specific job.
- The `ShouldQueueAfterCommit` interface can be implemented on a listener class for fine-grained control.
- If no transaction is open, the job is dispatched immediately.

**Constraints and Limitations:**

- **The `after_commit` option is per-connection, not per-job.** It affects all jobs on that connection.
- **Deferred jobs are discarded on transaction rollback.** Ensure that any state changes are also rolled back.
- **`afterCommit()` has no effect if no transaction is open.** The job dispatches immediately.
- **The `after_commit` option adds a small overhead** because Laravel must track open transactions.
- **Nested transactions are handled correctly** — jobs are dispatched after the outermost transaction commits.

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
        'after_commit' => true,
    ],
],
```

```php
// Dispatching a job inside a transaction
use App\Jobs\SendWelcomeEmail;
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    $user = User::create([
        'name'  => 'Jane Doe',
        'email' => 'jane@example.com',
    ]);

    // With after_commit => true, this job is not dispatched
    // until the transaction commits successfully
    SendWelcomeEmail::dispatch($user);

    // If an exception occurs here, the transaction rolls back
    // and the job is never dispatched
});
```

**Expected Output:** The `SendWelcomeEmail` job is dispatched only after the transaction commits. If the transaction rolls back, the job is discarded.

**Why This Output Occurs:** The `after_commit` configuration option tells Laravel to defer all queued jobs until all open database transactions have been committed. The `QueueManager` checks the connection's `after_commit` setting and registers the job for deferred dispatch. If the transaction commits, the deferred jobs are dispatched; if it rolls back, they are discarded.

---

**Example 2: Per-Job afterCommit and beforeCommit**

```php
use App\Jobs\ProcessPodcast;
use App\Jobs\LogPodcastActivity;
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    $podcast = Podcast::create([...]);

    // This job is deferred until the transaction commits
    ProcessPodcast::dispatch($podcast)->afterCommit();

    // This job is dispatched immediately, even if after_commit is globally true
    LogPodcastActivity::dispatch($podcast)->beforeCommit();
});
```

**Expected Output:**

- `ProcessPodcast` is dispatched only after the transaction commits.
- `LogPodcastActivity` is dispatched immediately, regardless of the transaction state.

**Why This Output Occurs:** The `afterCommit()` method sets a per-job flag that instructs the `Dispatcher` to defer dispatch until the transaction commits. The `beforeCommit()` method overrides the global `after_commit` setting for that specific job, dispatching it immediately. This allows fine-grained control over which jobs are transaction-aware.

### Real-World Cases

- **User registration:** The user creation transaction commits before the welcome email job is dispatched, ensuring the user record exists when the worker runs.
- **Order placement:** The order creation transaction commits before the inventory update job is dispatched, preventing the worker from trying to update stock for a non-existent order.
- **Payment processing:** The payment recording transaction commits before the receipt email job is dispatched, ensuring the payment record exists when the email is generated.
- **Content publishing:** The article publishing transaction commits before the search indexing job is dispatched, ensuring the article is indexed with its final published state.
- **Inventory management:** The stock adjustment transaction commits before the low-stock alert job is dispatched, ensuring the alert reflects the committed stock level.

---

## References

- Laravel Queues Documentation (Master) — https://laravel.com/framework/docs/master/queues
- Laravel Queues Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/queues
- Laravel Queues Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/queues
- Laravel Queues Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/queues
- Laravel `Dispatchable` Trait API — https://api.laravel.com/docs/8.x/Illuminate/Foundation/Bus/Dispatchable.html
- Laravel `Queueable` Trait API — https://api.laravel.com/docs/master/Illuminate/Bus/Queueable.html
- Laravel `SerializesModels` Trait API — https://api.laravel.com/docs/master/Illuminate/Queue/SerializesModels.html
- Laravel `InteractsWithQueue` Trait API — https://api.laravel.com/docs/master/Illuminate/Queue/InteractsWithQueue.html
- Laravel `QueueManager` API — https://api.laravel.com/docs/master/Illuminate/Queue/QueueManager.html
- Laravel Horizon Documentation — https://laravel.com/docs/master/horizon
- Laravel Queue Drivers Comparison (DeepWiki) — https://deepwiki.com/unicodeveloper/laravel-exam/5.1-jobs-and-queue-system
- Laravel Queue Config Driver Choice (GitHub) — https://github.com/voku/agent-skills
- Laravel Serialization of Models in Jobs (LinkedIn) — https://www.linkedin.com/posts/atiqur-rahman
- Laravel Queue Dispatch Patterns (GitHub) — https://github.com/fusengine/agents
- Laravel Queue Transactions (Laracasts) — https://laracasts.com/discuss/channels/laravel/queue-and-transactions
- Laravel Transactional Jobs Package — https://github.com/therezor/laravel-transactional-jobs
- Laravel Transaction Commit Queue Package — https://packagist.org/packages/jlorente/laravel-transaction-commit-queue
- Laravel `after_commit` Queue Configuration — https://laravel.com/docs/master/queues#jobs-and-database-transactions
- Laravel `ShouldQueueAfterCommit` Interface — https://laravel.com/docs/master/queues#jobs-and-database-transactions
- Laravel Queue Configuration File (GitLab) — https://gitlab.com/config/queue.php