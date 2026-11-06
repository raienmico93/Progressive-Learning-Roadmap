# Laravel Queue Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Queues provide a unified API for deferring time-consuming tasks—such as sending emails, processing uploads, or generating reports—to background workers, allowing web requests to respond quickly while heavy work is processed asynchronously.

**Technical Definition**

Laravel's queue system is composed of **connections** (backend services like Redis, Amazon SQS, or a database) and **queues** (named stacks within a connection). Jobs are dispatched onto a queue and processed by **queue workers**—long-running processes that poll the connection for pending jobs and execute them. The system supports configurable **retries**, **timeouts**, **backoff strategies**, and **failure handling**, with failed jobs stored for inspection and retry. Configuration lives in `config/queue.php`, and drivers include `sync`, `database`, `redis`, `sqs`, `beanstalkd`, and `null`.

**Beginner-Friendly Explanation**

Imagine a restaurant. When you order, the waiter does not cook your meal—they write the order down and pass it to the kitchen. You get to sit and chat while the kitchen works. Laravel Queues work the same way: your web request drops a “job” into a queue (the order ticket), and a background worker (the kitchen) picks it up and does the slow work later. This keeps your website fast and responsive.

---

### Key Characteristics

- **Unified API Across Backends**: The same job classes and dispatch syntax work with Redis, SQS, database, Beanstalkd, or even synchronous execution.
- **Connections vs. Queues**: A connection is the backend service; a queue is a named stack within that connection. A single connection can host many queues for prioritization or segmentation.
- **Worker-Based Processing**: Workers are long-running CLI processes (`queue:work`) that poll for and execute jobs, managed by process supervisors like Supervisor or Horizon.
- **Configurable Resilience**: Jobs support `$tries`, `$maxExceptions`, `$timeout`, `$backoff`, and `retryUntil()` for fine-grained retry and failure behaviour.
- **Failure Isolation**: Failed jobs are recorded in a `failed_jobs` table (after running `queue:failed-table`), where they can be inspected and retried.
- **Unique and Overlapping Job Control**: Middleware such as `WithoutOverlapping` and contracts like `ShouldBeUnique` prevent duplicate or concurrent execution of the same job.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with `config/queue.php` published.
- **A queue backend**: Redis, Amazon SQS, Beanstalkd, or a database table (the `jobs` table migration is included by default).
- **Driver-specific dependencies**: `predis/predis` for Redis, `aws/aws-sdk-php` for SQS, `pda/pheanstalk` for Beanstalkd.
- **A process supervisor** (Supervisor, systemd, or Laravel Horizon) for production workers.
- **`ext-pcntl`** for full worker features (signals, timeouts).

---

### Related Programming Areas

- **Queued Jobs**: The unit of work dispatched onto a queue.
- **Events and Listeners**: Listeners can implement `ShouldQueue` to run asynchronously.
- **Notifications and Mail**: Both can be queued to avoid blocking requests.
- **Laravel Horizon**: A Redis-specific dashboard and worker manager.
- **Task Scheduling**: Scheduled commands can dispatch jobs for heavy work.
- **Testing**: `Queue::fake()` allows assertions on dispatched jobs without executing them.

---

## Core Concepts

---

### 1. Queues

#### Definitions

**Core Definition**

A queue is a named stack (or “pile”) of pending jobs within a queue connection, from which workers pull jobs for processing.

**Technical Definition**

In `config/queue.php`, each connection configuration contains a `queue` attribute specifying the default queue for that connection. Jobs dispatched without an explicit queue name go to this default; `->onQueue('emails')` targets a specific queue. Multiple queues enable prioritization—a worker can be told to process `high` before `default` (`--queue=high,default`).

**Beginner-Friendly Explanation**

If a connection is the kitchen, a queue is a specific station: the grill station handles steaks, the salad station handles salads. You can tell a worker to check the grill station first, then the salad station.

#### Purposes

- To prioritize critical jobs (e.g., password resets) over lower-priority work (e.g., report generation).
- To segment jobs by type, team, or resource requirements.
- To scale workers independently per queue (e.g., more workers for the `emails` queue).
- To isolate failures or slowdowns to a specific workload.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Default queue on the default connection
ProcessPodcast::dispatch();

// Specific queue on the default connection
ProcessPodcast::dispatch()->onQueue('emails');

// Specific connection and queue
ProcessPodcast::dispatch()->onConnection('redis')->onQueue('high');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `dispatch()` | Pushes the job onto the queue. |
| `onQueue('name')` | Targets a named queue. |
| `onConnection('name')` | Targets a named connection. |

**Syntax Rules**

- The default queue is defined per connection in `config/queue.php`.
- Queue names are arbitrary strings; choose descriptive names (`high`, `default`, `low`, `emails`).
- A worker can process multiple queues in priority order.

**Constraints and Limitations**

- A worker processes one job at a time per process; scaling requires multiple workers.
- Queue names are case-sensitive.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Dispatching to a Specific Queue**

```php
ProcessPodcast::dispatch($podcast)->onQueue('podcasts');
```

**Expected Output**

The job is placed on the `podcasts` queue. A worker running `php artisan queue:work --queue=podcasts` will process it.

**Why This Output Occurs**

`onQueue('podcasts')` overrides the connection’s default queue attribute, directing the job to the named stack.

---

**Example 2: Worker Priority Across Queues**

```bash
php artisan queue:work --queue=high,default,low
```

**Expected Output**

The worker processes all jobs on `high` before moving to `default`, and all `default` jobs before `low`.

**Why This Output Occurs**

The `--queue` option accepts a comma-separated priority list; the worker polls queues in order.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Pipeline**

An e-commerce application uses `high` for payment confirmations, `default` for order emails, and `low` for analytics exports. Separate workers handle each queue, ensuring payment jobs are never delayed by analytics exports.

**Explanation**: Queue segmentation prevents low-priority work from starving critical jobs.

---

### 2. Queue Drivers

#### Definitions

**Core Definition**

Queue drivers are the backend implementations that store and retrieve queued jobs, selected via the `default` key in `config/queue.php`.

**Technical Definition**

Laravel supports `sync` (immediate execution), `database` (relational table), `redis` (in-memory store), `sqs` (Amazon Simple Queue Service), `beanstalkd` (dedicated queue server), and `null` (discards jobs). Each driver has distinct performance, persistence, and operational characteristics.

**Beginner-Friendly Explanation**

Different drivers are like different types of mailboxes. A `sync` mailbox delivers instantly (no queue at all). A `database` mailbox is a filing cabinet. A `redis` mailbox is a super-fast pneumatic tube. An `sqs` mailbox is a cloud-based postal service. Choose the mailbox that fits your needs.

#### Purposes

- To match queue infrastructure to application scale, budget, and operational constraints.
- To separate development (`sync`) from production (Redis/SQS) behaviour.
- To leverage managed services (SQS) or self-hosted solutions (Redis, Beanstalkd).

#### Syntax Rules and Structure

**Configuration (`config/queue.php`)**

```php
'default' => env('QUEUE_CONNECTION', 'database'),

'connections' => [
    'sync' => ['driver' => 'sync'],
    'database' => [
        'driver' => 'database',
        'table' => 'jobs',
        'queue' => 'default',
        'retry_after' => 90,
    ],
    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => 90,
        'block_for' => null,
    ],
    'sqs' => [
        'driver' => 'sqs',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'prefix' => env('SQS_PREFIX'),
        'queue' => env('SQS_QUEUE', 'default'),
    ],
],
```

**Component Breakdown**

| Driver | Storage | Performance | Use Case |
|--------|---------|-------------|----------|
| `sync` | In-memory | Immediate | Development/testing |
| `database` | Relational table | Moderate | Simple setups, no extra infra |
| `redis` | In-memory store | High | Production, high throughput |
| `sqs` | AWS managed | High | AWS-native, serverless |
| `beanstalkd` | Dedicated server | High | Specialized queue infrastructure |

**Syntax Rules**

- The `default` key selects the active connection.
- Each connection must specify a `driver`.
- Driver-specific options (`table`, `connection`, `key`, etc.) are required.

**Constraints and Limitations**

- `sync` ignores `ShouldQueue` and executes jobs immediately.
- `database` requires the `jobs` table migration.
- `redis` requires the `predis/predis` package or the `phpredis` extension.
- `sqs` requires `aws/aws-sdk-php`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Switching to Redis**

**`.env`**

```
QUEUE_CONNECTION=redis
REDIS_QUEUE=default
REDIS_QUEUE_RETRY_AFTER=90
```

**Expected Output**

Jobs are stored in Redis and processed by workers using the Redis driver.

**Why This Output Occurs**

The `default` connection is set to `redis`, and the Redis connection configuration is used.

---

#### Real-World Cases with Explanation

**Case 1: Laravel Vapor (Serverless)**

A serverless application on Laravel Vapor uses the `sqs` driver, leveraging AWS’s managed queue service to avoid maintaining Redis or Beanstalkd infrastructure.

**Explanation**: SQS is the natural choice for AWS-native serverless deployments.

---

### 3. Queue Configuration

#### Definitions

**Core Definition**

Queue configuration is the set of settings in `config/queue.php` and `.env` that control the active connection, driver options, retry behaviour, and worker defaults.

**Technical Definition**

`config/queue.php` defines the `default` connection and a `connections` array. Each connection includes driver-specific options: `queue` (default queue name), `retry_after` (seconds before a reserved job is retried), `after_commit` (dispatch after database transaction commit), and for Redis, `block_for` (blocking pop timeout). Environment variables allow per-environment overrides.

**Beginner-Friendly Explanation**

Configuration is the control panel. It tells Laravel which queue service to use, how long to wait before retrying, and which queue is the default.

#### Purposes

- To select the active queue backend per environment.
- To tune retry timing and worker behaviour.
- To configure driver credentials and endpoints.
- To control transaction-aware dispatching.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/queue.php
'default' => env('QUEUE_CONNECTION', 'database'),

'connections' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
        'block_for' => null,
        'after_commit' => false,
    ],
],
```

**Component Breakdown**

| Key | Description |
|-----|-------------|
| `default` | Active connection name. |
| `driver` | Backend type (`redis`, `database`, `sqs`, etc.). |
| `queue` | Default queue name for the connection. |
| `retry_after` | Seconds before a reserved job is retried. |
| `block_for` | Redis blocking pop timeout (0 = indefinite). |
| `after_commit` | Dispatch jobs after the enclosing transaction commits. |

**Syntax Rules**

- `retry_after` must be longer than the longest job’s execution time.
- `block_for` applies only to Redis.
- `after_commit` prevents jobs from being dispatched before a transaction commits.

**Constraints and Limitations**

- Setting `retry_after` too low can cause duplicate job execution.
- `after_commit` requires a database transaction context.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Tuning Redis `retry_after`**

```php
'redis' => [
    'driver' => 'redis',
    'queue' => 'default',
    'retry_after' => 120, // jobs have up to 2 minutes before retry
],
```

**Expected Output**

A job reserved by a worker but not completed within 120 seconds is released back to the queue for another attempt.

**Why This Output Occurs**

`retry_after` tells the queue how long to wait before considering a reserved job lost and making it available again.

---

#### Real-World Cases with Explanation

**Case 1: Transaction-Aware Dispatching**

An order service dispatches an `OrderPlaced` job inside a database transaction. With `after_commit => true`, the job is only dispatched after the transaction commits, preventing a worker from processing an order that was rolled back.

**Explanation**: `after_commit` prevents race conditions between transactions and queue workers.

---

### 4. Queue Connections

#### Definitions

**Core Definition**

A queue connection is a named configuration that points to a backend queue service (Redis, SQS, database, etc.) and defines its driver and options.

**Technical Definition**

Connections are defined in the `connections` array of `config/queue.php`. A single application can define multiple connections—for example, a `redis` connection for high-throughput jobs and an `sqs` connection for AWS-integrated jobs. Jobs can be dispatched to a specific connection with `onConnection('name')`. Each connection has its own default queue and driver-specific settings.

**Beginner-Friendly Explanation**

A connection is a specific queue service account. You might have one connection to your Redis server and another to AWS SQS. Each connection can have multiple queues.

#### Purposes

- To use multiple queue backends within one application.
- To isolate workloads across different infrastructure (e.g., Redis for speed, SQS for durability).
- To support migration from one backend to another without rewriting job code.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/queue.php
'connections' => [
    'redis' => [ /* ... */ ],
    'sqs' => [ /* ... */ ],
    'database' => [ /* ... */ ],
],
```

**Dispatching to a Specific Connection**

```php
ProcessPodcast::dispatch()->onConnection('sqs');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Connection name | Key in the `connections` array. |
| `onConnection('name')` | Directs the job to a specific connection. |

**Syntax Rules**

- The default connection is used unless `onConnection()` is called.
- Each connection must have a unique name.
- Connections can share a driver but differ in configuration (e.g., two Redis connections to different servers).

**Constraints and Limitations**

- A worker processes one connection at a time (specified via `queue:work {connection}`).
- Cross-connection transactions are not supported.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Multiple Connections**

```php
// Dispatch to Redis
ProcessPodcast::dispatch()->onConnection('redis');

// Dispatch to SQS
SendInvoice::dispatch()->onConnection('sqs');
```

**Expected Output**

Each job is routed to its designated backend service.

**Why This Output Occurs**

`onConnection()` overrides the default connection for the dispatched job.

---

#### Real-World Cases with Explanation

**Case 1: Hybrid Infrastructure**

A company uses Redis for internal, high-speed jobs and SQS for jobs that must survive infrastructure outages or integrate with AWS Lambda. Two connections coexist, and each job is dispatched to the appropriate backend.

**Explanation**: Multiple connections allow architectural flexibility without code duplication.

---

### 5. Queue Workers

#### Definitions

**Core Definition**

A queue worker is a long-running CLI process (`php artisan queue:work`) that polls a connection for pending jobs and executes them.

**Technical Definition**

Workers are started with `queue:work` (persistent) or `queue:listen` (less efficient, reloads code on each job). They accept options such as `--queue` (priority list), `--tries` (max attempts), `--timeout` (max seconds per job), `--sleep` (poll interval), `--backoff` (retry delay), and `--max-jobs`/`--max-time` (graceful shutdown). In production, workers are supervised by Supervisor, systemd, or Laravel Horizon to restart on failure and scale horizontally.

**Beginner-Friendly Explanation**

A worker is the kitchen staff. It constantly checks the order tickets (queue) and cooks each dish (job). You run one or more workers; more workers mean more dishes can be cooked at once.

#### Purposes

- To process queued jobs asynchronously from web requests.
- To scale processing horizontally by running multiple workers.
- To control job execution limits (tries, timeout, memory).
- To enable graceful restarts and zero-downtime deployments.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Basic worker
php artisan queue:work

# Specific connection and queue priority
php artisan queue:work redis --queue=high,default

# With retry and timeout options
php artisan queue:work --tries=3 --timeout=60 --backoff=10

# Graceful shutdown after 100 jobs or 1 hour
php artisan queue:work --max-jobs=100 --max-time=3600
```

**Component Breakdown**

| Option | Description |
|--------|-------------|
| `--queue` | Comma-separated priority list. |
| `--tries` | Maximum attempts per job. |
| `--timeout` | Seconds before a job is killed. |
| `--backoff` | Seconds to wait before retrying. |
| `--max-jobs` | Stop after N jobs. |
| `--max-time` | Stop after N seconds. |
| `--sleep` | Seconds to sleep when no job is available. |

**Syntax Rules**

- `queue:work` is the production worker; `queue:listen` is for development.
- `--timeout` must be shorter than `retry_after`.
- `--tries` can be overridden per job via `$tries`.
- Workers should be restarted after code changes (`queue:restart`).

**Constraints and Limitations**

- Workers are long-lived; code changes require a restart.
- Memory leaks in jobs can exhaust worker memory over time; use `--max-jobs` for periodic restarts.
- `queue:listen` is significantly slower and not recommended for production.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Supervisor Configuration**

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/app/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
numprocs=4
```

**Expected Output**

Four worker processes run continuously, automatically restarting if they crash, and restarting every hour to avoid memory leaks.

**Why This Output Occurs**

Supervisor manages the workers, and the options ensure resilience and periodic refresh.

---

#### Real-World Cases with Explanation

**Case 1: Scaling Email Processing**

An application sends thousands of transactional emails per hour. Running eight workers on the `emails` queue reduces processing time from hours to minutes.

**Explanation**: Horizontal worker scaling is the primary mechanism for increasing queue throughput.

---

### 6. Queue Processing

#### Definitions

**Core Definition**

Queue processing is the end-to-end lifecycle of a job: dispatch → enqueue → reservation by a worker → execution → success or failure → cleanup or retry.

**Technical Definition**

When a job is dispatched, the queue driver serializes the job class and its data onto the queue. A worker reserves the job (making it invisible to other workers), executes its `handle()` method, and then deletes it from the queue on success. If the job throws an exception, it is released back to the queue for retry (until `$tries` is exhausted) or marked as failed. The `retry_after` setting determines how long a reserved job can run before being released for another attempt.

**Beginner-Friendly Explanation**

Processing is the full journey: your code drops the job off, a worker picks it up, does the work, and either finishes it or puts it back for another try.

#### Purposes

- To ensure jobs are executed exactly as intended, with configurable retry and failure behaviour.
- To prevent jobs from being lost or double-processed.
- To provide visibility into job lifecycle and outcomes.

#### Syntax Rules and Structure

**Complete General Syntax (Job Class)**

```php
<?php
namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    public int $tries = 3;
    public int $timeout = 120;

    public function __construct(public Podcast $podcast) {}

    public function handle(): void
    {
        // Process the podcast
    }
}
```

**Component Breakdown**

| Property | Description |
|----------|-------------|
| `$tries` | Maximum attempts before failure. |
| `$timeout` | Seconds before the job is killed. |
| `$backoff` | Seconds to wait before retry. |
| `$maxExceptions` | Max exceptions before failure. |
| `handle()` | The job’s work. |

**Syntax Rules**

- Jobs must implement `ShouldQueue` to be queued.
- `$tries`, `$timeout`, and `$backoff` can be set as properties or via worker options.
- `handle()` is resolved by the container, allowing dependency injection.

**Constraints and Limitations**

- Job payloads must be serializable; closures and non-serializable objects cannot be queued.
- `$timeout` must be less than `retry_after`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Job with Retry and Timeout**

```php
class SendWelcomeEmail implements ShouldQueue
{
    use Queueable;

    public int $tries = 5;
    public int $timeout = 30;
    public int $backoff = 10;

    public function handle(Mailer $mailer): void
    {
        $mailer->send(new WelcomeEmail($this->user));
    }
}
```

**Expected Output**

The job is attempted up to 5 times, each attempt limited to 30 seconds, with a 10-second wait between attempts.

**Why This Output Occurs**

The `$tries`, `$timeout`, and `$backoff` properties configure the worker’s handling of this specific job.

---

#### Real-World Cases with Explanation

**Case 1: Third-Party API Integration**

A job that calls a flaky third-party API is configured with `$tries = 5` and `$backoff = 30`. Transient failures are retried automatically, while permanent failures eventually exhaust the retries and are marked as failed.

**Explanation**: Retry and backoff configuration turns fragile integrations into resilient ones.

---

### 7. Queue Retries

#### Definitions

**Core Definition**

Queue retries are the automatic re-attempts of a failed job, controlled by `$tries`, `--tries`, `$backoff`, and `retryUntil()`.

**Technical Definition**

A job can specify `$tries` (maximum attempts) as a class property or receive it via `--tries` on the worker command. `$backoff` (seconds) or an array of backoff intervals controls the delay between attempts. `retryUntil()` returns a timestamp after which no further attempts are made. `$maxExceptions` limits how many exceptions can occur before failure, independent of attempts.

**Beginner-Friendly Explanation**

Retries are “try again” rules. If a job fails, Laravel can wait a bit and try again—up to a limit you set.

#### Purposes

- To recover from transient failures (network blips, temporary outages).
- To avoid permanent failure from one-off errors.
- To control the cost and latency of retries with backoff strategies.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public int $tries = 3;
public int $backoff = 10;
public array $backoff = [10, 30, 60]; // escalating delays

public function retryUntil(): DateTime
{
    return now()->addMinutes(30);
}
```

**Component Breakdown**

| Mechanism | Description |
|-----------|-------------|
| `$tries` | Max attempts. |
| `--tries=N` | Worker-level max attempts. |
| `$backoff` | Seconds (or array) between retries. |
| `retryUntil()` | Absolute deadline for retries. |
| `$maxExceptions` | Max exceptions before failure. |

**Syntax Rules**

- If `--tries` is set, it overrides the job’s `$tries` for that worker.
- `retryUntil()` takes precedence over `$tries` when both are present.
- Backoff applies between attempts, not after the final failure.

**Constraints and Limitations**

- Infinite retries are possible with `$tries = 0` or a far-future `retryUntil()`, but can cause runaway processing.
- `retryUntil()` and `$tries` should not both be used unless intentional.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Escalating Backoff**

```php
public int $tries = 3;
public array $backoff = [10, 60, 300]; // 10s, 1m, 5m
```

**Expected Output**

After the first failure, the job retries after 10 seconds; after the second failure, after 60 seconds; after the third, after 300 seconds.

**Why This Output Occurs**

The `$backoff` array provides per-attempt delays, allowing more time for transient issues to resolve.

---

#### Real-World Cases with Explanation

**Case 1: Payment Processing**

A payment job retries three times with escalating backoff (5s, 30s, 2m). This handles temporary gateway outages without retrying a permanently declined card indefinitely.

**Explanation**: Escalating backoff balances responsiveness with resilience.

---

### 8. Queue Failures

#### Definitions

**Core Definition**

Queue failures occur when a job exhausts its retries or throws an exception that prevents successful completion; failed jobs are recorded for inspection and manual retry.

**Technical Definition**

When a job fails permanently, Laravel writes a record to the `failed_jobs` table (created via `php artisan queue:failed-table` and `php artisan migrate`). The `failed_jobs` table stores the connection, queue, payload, exception, and failure timestamp. Failed jobs can be retried with `queue:retry`, deleted with `queue:forget`, or flushed with `queue:flush`. The `failed()` method on a job is called when the job fails permanently.

**Beginner-Friendly Explanation**

Failures are the “I give up” state. After all retries are exhausted, the job is set aside in a special list so you can inspect it later and decide whether to try again.

#### Purposes

- To capture and preserve failed jobs for debugging.
- To allow manual retry after fixing the underlying issue.
- To prevent silent data loss when jobs cannot complete.
- To provide an audit trail of failures.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Create the failed_jobs table
php artisan queue:failed-table
php artisan migrate

# Inspect failed jobs
php artisan queue:failed

# Retry a specific failed job
php artisan queue:retry {id}

# Retry all failed jobs
php artisan queue:retry all

# Delete a failed job
php artisan queue:forget {id}

# Flush all failed jobs
php artisan queue:flush
```

**Component Breakdown**

| Command | Description |
|---------|-------------|
| `queue:failed-table` | Generates the migration for `failed_jobs`. |
| `queue:failed` | Lists failed jobs. |
| `queue:retry` | Retries a failed job. |
| `queue:forget` | Deletes a failed job. |
| `queue:flush` | Deletes all failed jobs. |

**Syntax Rules**

- The `failed_jobs` table must exist for failures to be recorded.
- `queue:retry` re-dispatches the job with a fresh attempt count.
- A job’s `failed()` method is invoked on permanent failure.

**Constraints and Limitations**

- Failed jobs are not automatically retried; manual intervention is required.
- The `failed_jobs` table can grow large; prune old records periodically.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Retrying a Failed Job**

```bash
php artisan queue:failed
# Output: ID 1, connection: redis, queue: default, class: App\Jobs\SendEmail

php artisan queue:retry 1
```

**Expected Output**

The failed job with ID 1 is re-dispatched onto its original queue and processed again.

**Why This Output Occurs**

`queue:retry` retrieves the payload from `failed_jobs` and pushes it back onto the queue.

---

#### Real-World Cases with Explanation

**Case 1: Post-Deployment Failure Recovery**

A bug causes all invoice jobs to fail. The team fixes the bug, deploys, and runs `php artisan queue:retry all` to reprocess the failed invoices without manual data entry.

**Explanation**: Failed job storage enables recovery from systemic failures after a fix is deployed.

---

### 9. Queue Best Practices

#### Definitions

**Core Definition**

Queue best practices are the guidelines that keep queue-based systems reliable, observable, efficient, and maintainable at scale.

**Technical Definition**

Best practices encompass choosing the right driver for the workload, tuning `retry_after` relative to job timeouts, using `after_commit` for transactional dispatch, leveraging queue prioritization, supervising workers for resilience, implementing idempotent jobs, monitoring failed jobs, and using Horizon (for Redis) or equivalent observability tooling.

#### Key Best Practices

1. **Choose the Right Driver**: Use `redis` for high-throughput production workloads, `database` for simple setups without extra infrastructure, and `sqs` for AWS-native or serverless deployments.

2. **Set `retry_after` Longer Than Job Timeout**: If `retry_after` is shorter than a job’s execution time, the job may be retried while still running, causing duplicate execution.

3. **Use `after_commit` for Transactional Dispatch**: When dispatching jobs inside database transactions, set `after_commit => true` to prevent workers from processing jobs for rolled-back transactions.

4. **Prioritize with Multiple Queues**: Use `--queue=high,default,low` to ensure critical jobs are processed first.

5. **Supervise Workers in Production**: Never run `queue:work` manually in production; use Supervisor, systemd, or Horizon to restart on failure and scale horizontally.

6. **Implement Idempotent Jobs**: Jobs may be retried after partial success; design them to be safe to run more than once (e.g., use `updateOrCreate`).

7. **Use `WithoutOverlapping` and `ShouldBeUnique`**: Prevent concurrent execution of the same logical job when duplicates would cause harm.

8. **Monitor Failed Jobs**: Regularly inspect `queue:failed` and set up alerts for spikes in failures.

9. **Set Memory and Time Limits**: Use `--max-jobs` and `--max-time` to periodically restart workers and avoid memory leaks.

10. **Use Horizon for Redis**: Horizon provides a dashboard, metrics, and advanced worker configuration for Redis-backed queues.

11. **Test with `Queue::fake()`**: In tests, fake the queue and assert on dispatched jobs rather than executing them, keeping tests fast and deterministic.

12. **Restart Workers on Deploy**: Run `php artisan queue:restart` after deploying code so workers pick up the new code.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Transaction-Safe Dispatch**

```php
// config/queue.php
'redis' => [
    'driver' => 'redis',
    'retry_after' => 90,
    'after_commit' => true,
],
```

**Expected Output**

Jobs dispatched inside a database transaction are only pushed to the queue after the transaction commits.

**Why This Output Occurs**

`after_commit` defers the dispatch until the transaction is successfully committed, preventing race conditions.

---

**Example 2: Unique Job**

```php
class GenerateReport implements ShouldQueue, ShouldBeUnique
{
    public int $uniqueFor = 3600; // uniqueness window in seconds

    public function uniqueId(): string
    {
        return (string) $this->userId;
    }
}
```

**Expected Output**

Only one `GenerateReport` job per `userId` exists in the queue at any time within the 1-hour window.

**Why This Output Occurs**

`ShouldBeUnique` and `uniqueId()` enforce uniqueness, preventing duplicate report generation.

---

#### Real-World Cases with Explanation

**Case 1: Production Queue Architecture**

A SaaS application uses Redis with four supervised workers per queue, `retry_after` set to 120 seconds with job timeouts of 60 seconds, `after_commit` enabled for order jobs, and Horizon for monitoring. Failed jobs are reviewed daily, and `queue:restart` runs on every deployment. The queue system has maintained 99.9% job success for over a year.

**Explanation**: Combining driver choice, worker supervision, transactional safety, and observability yields a reliable production queue system.

---

## References

- Queues — Laravel 12.x Documentation — https://laravel.com/docs/12.x/queues
- Queues — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/queues
- Queue Driver Choice — database vs redis vs sqs — https://raw.githubusercontent.com/…/queue-driver-choice.md
- Jobs and Queue System — DeepWiki — https://deepwiki.com/unicodeveloper/laravel-exam
- Laravel Queues: Mastering Asynchronous Job Processing — https://vecardigitalprogramming.com
- Laravel Queues & Jobs — Complete Reference — https://raw.githubusercontent.com/…/laravel-queues-jobs-reference.md
- Laravel Queues: Handling Errors Gracefully — PHP Architect — https://www.phparch.com
- Queues — Laravel 10.x Documentation — https://laravel.com/docs/10.x/queues
- Queue Driver Prerequisites — Packagist — https://packagist.org/packages/illuminate/queue

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. The `sync`, `database`, `redis`, `sqs`, `beanstalkd`, and `null` drivers are available across these versions. `after_commit`, `ShouldBeUnique`, and `WithoutOverlapping` are available in recent Laravel versions. `retry_after` and `block_for` semantics are Redis-specific. Driver dependencies (e.g., `predis/predis`, `aws/aws-sdk-php`, `pda/pheanstalk`) must be installed separately. Always consult the documentation for your specific Laravel version.