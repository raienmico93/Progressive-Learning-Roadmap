# Laravel Job Batching: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Job Batching is a feature that allows you to dispatch a group of jobs as a single, trackable unit, execute them in parallel, monitor their collective progress, and run callbacks when the entire batch completes, fails, or finishes.

**Technical Definition**

Job batching in Laravel is implemented by the `Illuminate\Bus\Batch` class and the `Bus::batch()` method. A batch is a persisted record (in the `job_batches` table) that tracks the total jobs, pending jobs, failed jobs, and completion status. Jobs in a batch must use the `Illuminate\Bus\Batchable` trait, which provides access to the batch instance via `$this->batch()`. Batches support lifecycle callbacks (`then`, `catch`, `finally`, `progress`), cancellation, dynamic job addition, and failure tolerance via `allowFailures()`.

**Beginner-Friendly Explanation**

Imagine you need to process 1,000 images. Instead of running them one by one, you group them into a batch. Laravel runs them in parallel (using multiple workers), keeps track of how many are done, and notifies you when the whole group finishes. If one fails, you decide whether to stop the whole batch or let the rest continue. A batch is like a “project” that contains many individual tasks.

---

### Key Characteristics

- **Parallel Execution**: Jobs in a batch are processed concurrently by multiple queue workers, so completion order is not guaranteed to match dispatch order.
- **Progress Tracking**: The `Batch` instance exposes `progress()` (0–100), `totalJobs`, `pendingJobs`, `failedJobs`, and `processedJobs()`.
- **Lifecycle Callbacks**: `then()` fires on success, `catch()` fires on failure, `finally()` fires regardless of outcome, and `progress()` fires after each successful job.
- **Failure Policy**: By default, a single job failure cancels the entire batch. `allowFailures()` changes this to let remaining jobs continue.
- **Cancellation**: A batch can be cancelled programmatically (`$batch->cancel()`) or from within a job (`$this->batch()->cancelled()`).
- **Dynamic Expansion**: Jobs can be added to a batch from within a job that belongs to the same batch (`$this->batch()->add([...])`), enabling “loader” patterns for very large workloads.
- **Persistence**: Batch metadata is stored in the `job_batches` table (or DynamoDB), and records should be pruned regularly.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with a configured queue connection and at least one queue worker.
- **The `job_batches` table** created via `php artisan make:queue-batches-table` and `php artisan migrate`.
- **Jobs implementing `ShouldQueue`** and using the `Batchable` trait.
- **Familiarity with queue dispatching** and basic callback patterns.

---

### Related Programming Areas

- **Queues**: Batches are dispatched to queues and processed by workers.
- **Jobs**: The individual units of work inside a batch.
- **Job Chaining**: Chains can be nested inside batches and vice versa.
- **Failure Handling**: Failed jobs are recorded in `failed_jobs`, and batch failures can trigger callbacks.
- **Monitoring**: Laravel Horizon displays batch progress and failures in its dashboard.
- **Pruning**: The `queue:prune-batches` command cleans up completed, cancelled, and unfinished batch records.

---

## Core Concepts

---

### 1. Job Batching

#### Definitions

**Core Definition**

Job batching is the practice of grouping multiple queue jobs into a single batch that is dispatched, processed, and tracked as a unit.

**Technical Definition**

A batch is defined by an array of job instances passed to `Bus::batch([...])`. Each job must use the `Batchable` trait, which provides `$this->batch()` returning the current `Batch` instance. The batch is persisted in the `job_batches` table with a UUID, total job count, pending count, and metadata. Workers process jobs independently; the batch tracks completion across all jobs.

**Beginner-Friendly Explanation**

A batch is a container for many jobs. Instead of dispatching 1,000 jobs individually and losing track of them, you put them in a batch, and Laravel tracks the whole group.

#### Purposes

- To group independent jobs that should be tracked together.
- To process large workloads in parallel with a single dispatch call.
- To attach completion, failure, and progress callbacks to a group of jobs.
- To enable cancellation and dynamic expansion of a running workload.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([
    new ProcessImage('img1.jpg'),
    new ProcessImage('img2.jpg'),
    new ProcessImage('img3.jpg'),
])->dispatch();
```

**Batchable Job**

```php
use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessImage implements ShouldQueue
{
    use Batchable, Queueable;

    public function __construct(public string $path) {}

    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            return;
        }
        // Process the image
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Bus::batch([...])` | Defines the batch with an array of jobs. |
| `Batchable` trait | Provides `$this->batch()` inside the job. |
| `$this->batch()->cancelled()` | Checks if the batch was cancelled. |
| `dispatch()` | Dispatches the batch to the queue. |

**Syntax Rules**

- Every job in the batch must use the `Batchable` trait.
- The `job_batches` table must exist.
- Batch jobs can be dispatched to a specific connection or queue via `->onConnection()` / `->onQueue()` on the `PendingBatch`.
- Jobs in a batch must be serializable.

**Constraints and Limitations**

- Batch jobs are processed in parallel, so completion order is not deterministic.
- The `job_batches` table grows with every batch and must be pruned.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Batch**

```php
$batch = Bus::batch([
    new ProcessImage('a.jpg'),
    new ProcessImage('b.jpg'),
    new ProcessImage('c.jpg'),
])->dispatch();
```

**Expected Output**

The three jobs are dispatched to the queue and processed in parallel by available workers. The batch tracks their completion.

**Why This Output Occurs**

`Bus::batch()` creates a `PendingBatch`, persists it, and dispatches each job with a reference to the batch UUID. Workers process the jobs independently.

---

#### Real-World Cases with Explanation

**Case 1: Image Processing Pipeline**

A user uploads 100 images. The application dispatches a batch of 100 `ProcessImage` jobs, and each job generates a thumbnail, applies a filter, and stores the result. The batch tracks progress and sends a notification when all images are processed.

**Explanation**: Batching parallelises independent work and provides a single point for progress and completion handling.

---

### 2. Batch Dispatching

#### Definitions

**Core Definition**

Batch dispatching is the act of creating and pushing a batch of jobs onto the queue, optionally configuring its name, queue, connection, and failure policy.

**Technical Definition**

Dispatching is done via `Bus::batch([...])->dispatch()`. The `PendingBatch` object returned before `dispatch()` supports fluent configuration: `->name('...')`, `->onQueue('...')`, `->onConnection('...')`, `->allowFailures()`, and `->before(callback)`. The `dispatch()` method persists the batch and dispatches all jobs. The resulting `Batch` instance is returned and can be stored for later inspection.

**Beginner-Friendly Explanation**

Dispatching a batch means “start this group of jobs.” You can give it a name, choose which queue it runs on, and decide whether one failure should stop the whole batch.

#### Purposes

- To configure batch behaviour before dispatch.
- To assign a human-readable name for identification.
- To control the queue and connection used by batch jobs.
- To enable failure tolerance with `allowFailures()`.
- To run a setup callback before jobs are dispatched.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([
    new ProcessImage('a.jpg'),
    new ProcessImage('b.jpg'),
])
    ->name('Image Processing')
    ->onQueue('images')
    ->onConnection('redis')
    ->allowFailures()
    ->before(function (Batch $batch) {
        // Runs before jobs are dispatched
    })
    ->dispatch();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `->name('...')` | Assigns a name to the batch. |
| `->onQueue('...')` | Sets the queue for all jobs. |
| `->onConnection('...')` | Sets the connection for all jobs. |
| `->allowFailures()` | Continues the batch on job failures. |
| `->before(callback)` | Runs before jobs are dispatched. |
| `->dispatch()` | Dispatches the batch. |

**Syntax Rules**

- `before()` callbacks receive the `Batch` instance.
- `allowFailures()` disables the default failure-cancels-batch behaviour.
- Queue and connection settings apply to all jobs unless overridden per job.

**Constraints and Limitations**

- `before()` runs synchronously during dispatch.
- `allowFailures()` changes callback semantics (see Batch Failures).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Named Batch on a Specific Queue**

```php
$batch = Bus::batch([
    new ImportCsv(1, 1000),
    new ImportCsv(1001, 2000),
])
    ->name('CSV Import')
    ->onQueue('imports')
    ->dispatch();
```

**Expected Output**

The batch is named “CSV Import” and all jobs are pushed onto the `imports` queue.

**Why This Output Occurs**

`->name()` sets the batch name for identification in the dashboard and database; `->onQueue()` overrides the default queue for the dispatched jobs.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant Imports**

A multi-tenant application dispatches CSV imports for multiple tenants. Each batch is named after the tenant (`->name("Import: {$tenant->name}")`) and dispatched to a tenant-specific queue, allowing prioritisation.

**Explanation**: Naming and queue targeting make batches identifiable and schedulable in production.

---

### 3. Batch Callbacks

#### Definitions

**Core Definition**

Batch callbacks are methods that attach behaviour to specific points in a batch’s lifecycle: success (`then`), failure (`catch`), completion regardless of outcome (`finally`), and per-job progress (`progress`).

**Technical Definition**

`PendingBatch` supports `then(callable)`, `catch(callable)`, `finally(callable)`, and `progress(callable)`. Each callback receives the `Batch` instance (and the exception for `catch`). `then` fires when all jobs succeed; `catch` fires when a job fails (or when a failure is not allowed); `finally` fires after all jobs have been processed, regardless of success or failure; `progress` fires after each successful job.

**Beginner-Friendly Explanation**

Callbacks are notifications: “tell me when everything succeeds” (`then`), “tell me if something fails” (`catch`), and “tell me when it’s all over either way” (`finally`).

#### Purposes

- To run logic when the entire batch succeeds (`then`).
- To handle failures, log errors, or alert teams (`catch`).
- To clean up resources or send a final status regardless of outcome (`finally`).
- To update a progress indicator after each job (`progress`).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Bus\Batch;
use Throwable;

$batch = Bus::batch([...])
    ->then(function (Batch $batch) {
        // All jobs succeeded
    })
    ->catch(function (Batch $batch, Throwable $e) {
        // A job failed
    })
    ->finally(function (Batch $batch) {
        // Batch finished (success or failure)
    })
    ->progress(function (Batch $batch) {
        // A job completed successfully
    })
    ->dispatch();
```

**Component Breakdown**

| Callback | When Fired | Arguments |
|----------|------------|-----------|
| `then` | All jobs succeeded. | `Batch $batch` |
| `catch` | A job failed (and failures not allowed). | `Batch $batch, Throwable $e` |
| `finally` | Batch finished (success or failure). | `Batch $batch` |
| `progress` | After each successful job. | `Batch $batch` |

**Syntax Rules**

- `then` fires only when every job succeeds.
- `catch` fires on the first failure (unless `allowFailures()` is used).
- `finally` always fires once the batch is no longer processing.
- `progress` fires after each successful job, not after failures.

**Constraints and Limitations**

- With `allowFailures()`, `catch` does not fire; `then` fires when all jobs have either succeeded or failed.
- Callbacks are not queued by default; they run in the worker process that completes the batch.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Success and Failure Callbacks**

```php
$batch = Bus::batch([
    new ProcessImage('a.jpg'),
    new ProcessImage('b.jpg'),
])
    ->then(function (Batch $batch) {
        Log::info("Batch {$batch->id} succeeded");
    })
    ->catch(function (Batch $batch, Throwable $e) {
        Log::error("Batch {$batch->id} failed: {$e->getMessage()}");
    })
    ->finally(function (Batch $batch) {
        Cache::forget("batch_progress:{$batch->id}");
    })
    ->dispatch();
```

**Expected Output**

If both jobs succeed, `then` logs success and `finally` clears the cache. If one fails, `catch` logs the error and `finally` still clears the cache.

**Why This Output Occurs**

`then` runs only on full success; `catch` runs on failure; `finally` runs in both cases.

---

#### Real-World Cases with Explanation

**Case 1: Report Generation**

A nightly report batch sends a Slack notification on success (`then`), alerts the on-call engineer on failure (`catch`), and deletes temporary files in `finally`.

**Explanation**: Lifecycle callbacks centralise post-batch behaviour and ensure cleanup happens regardless of outcome.

---

### 4. Batch Progress

#### Definitions

**Core Definition**

Batch progress is the real-time measurement of how many jobs in a batch have been processed, expressed as counts, percentages, and pending/failed totals.

**Technical Definition**

The `Batch` instance provides `progress()` (0–100), `totalJobs`, `pendingJobs`, `failedJobs`, `processedJobs()`, and `finished()`. Batches can be retrieved by ID with `Bus::findBatch($batchId)`, which returns a `Batch` instance that can be returned directly from a route as JSON for frontend polling. The `progress()` callback on `PendingBatch` fires after each successful job, enabling incremental updates.

**Beginner-Friendly Explanation**

Progress tells you “how much is done.” You can poll the batch from your UI to show a progress bar, or use the `progress` callback to update a counter after each job.

#### Purposes

- To display a progress bar or percentage in a UI.
- To poll batch status from a frontend via a JSON endpoint.
- To trigger incremental side effects after each successful job.
- To determine when a batch is finished or still running.

#### Syntax Rules and Structure

**Complete General Syntax**

**Retrieving a batch**

```php
use Illuminate\Support\Facades\Bus;
use Illuminate\Support\Facades\Route;

Route::get('/batch/{batchId}', function (string $batchId) {
    return Bus::findBatch($batchId);
});
```

**Inspecting a batch**

```php
$batch = Bus::findBatch($batchId);

$batch->id;           // UUID
$batch->name;         // Batch name
$batch->totalJobs;    // Total jobs in the batch
$batch->pendingJobs;  // Jobs not yet processed
$batch->failedJobs;   // Jobs that failed
$batch->processedJobs(); // Jobs processed so far
$batch->progress();   // 0–100
$batch->finished();   // All jobs done?
$batch->cancelled();  // Was the batch cancelled?
```

**Progress callback**

```php
Bus::batch([...])
    ->progress(function (Batch $batch) {
        // Runs after each successful job
    })
    ->dispatch();
```

**Component Breakdown**

| Property / Method | Description |
|-------------------|-------------|
| `progress()` | Percentage complete (0–100). |
| `totalJobs` | Total jobs in the batch. |
| `pendingJobs` | Jobs not yet processed. |
| `failedJobs` | Jobs that failed. |
| `processedJobs()` | Jobs processed so far. |
| `finished()` | Whether all jobs have been processed. |
| `cancelled()` | Whether the batch was cancelled. |

**Syntax Rules**

- `Bus::findBatch()` returns `null` if the batch does not exist.
- The `Batch` instance is `JsonSerializable`, so returning it from a route produces a JSON payload.
- `progress()` fires after each successful job, not after failures.

**Constraints and Limitations**

- Polling requires a route or controller endpoint that returns the batch.
- Progress is not pushed automatically; the frontend must poll or use the `progress` callback.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Progress Endpoint**

```php
Route::get('/batch/{batchId}', function (string $batchId) {
    $batch = Bus::findBatch($batchId);
    abort_if($batch === null, 404);
    return $batch;
});
```

**Expected Output (JSON)**

```json
{
  "id": "uuid-here",
  "name": "Image Processing",
  "totalJobs": 100,
  "pendingJobs": 40,
  "failedJobs": 2,
  "processedJobs": 58,
  "progress": 58,
  "finished": false,
  "cancelled": false
}
```

**Why This Output Occurs**

The `Batch` instance serializes its public properties and computed values when returned from a route.

---

#### Real-World Cases with Explanation

**Case 1: Upload Progress UI**

A frontend uploads a ZIP of images and receives a batch ID. It polls `/batch/{id}` every two seconds to update a progress bar from 0% to 100%.

**Explanation**: Progress polling turns batch processing into a measurable, user-visible operation.

---

### 5. Batch Failures

#### Definitions

**Core Definition**

Batch failures occur when one or more jobs in a batch fail; Laravel’s default policy cancels the entire batch on the first failure, but this can be changed with `allowFailures()`.

**Technical Definition**

By default, when a job in a batch fails, Laravel marks the batch as cancelled and stops processing remaining jobs. The failed job is recorded in `failed_jobs` as usual. `allowFailures()` disables automatic cancellation, allowing remaining jobs to continue. With `allowFailures()`, the `then` callback fires when all jobs have either succeeded or failed; the `catch` callback does **not** fire. Failed jobs can be retried with `queue:retry`, and the batch record can be pruned with `queue:prune-batches`.

**Beginner-Friendly Explanation**

By default, one failure stops the whole batch. If you want the batch to keep going despite failures, call `allowFailures()`. Either way, failed jobs are recorded so you can retry them.

#### Purposes

- To control whether a single failure aborts the entire batch.
- To allow partially successful batches (e.g., importing 95 of 100 rows).
- To record failed jobs for manual retry.
- To trigger failure-specific callbacks when failures are not tolerated.

#### Syntax Rules and Structure

**Complete General Syntax**

**Default failure policy (cancel on first failure)**

```php
Bus::batch([...])
    ->catch(function (Batch $batch, Throwable $e) {
        // Runs on first failure; batch is cancelled
    })
    ->dispatch();
```

**Allow failures**

```php
Bus::batch([...])
    ->allowFailures()
    ->then(function (Batch $batch) {
        // Runs when all jobs have succeeded OR failed
    })
    ->dispatch();
```

**Component Breakdown**

| Policy | Behaviour |
|--------|-----------|
| Default | First failure cancels the batch; `catch` fires. |
| `allowFailures()` | Remaining jobs continue; `catch` does not fire; `then` fires when all jobs are done. |

**Syntax Rules**

- `allowFailures()` must be called on the `PendingBatch` before `dispatch()`.
- Failed jobs are always recorded in `failed_jobs`.
- With `allowFailures()`, use `$batch->failedJobs` to inspect failures.

**Constraints and Limitations**

- `catch` and `allowFailures()` are mutually exclusive in behaviour; `catch` will not fire when failures are allowed.
- Cancelled batches must be pruned explicitly.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Allow Failures**

```php
$batch = Bus::batch([
    new ImportRow(1),
    new ImportRow(2),
    new ImportRow(3),
])
    ->allowFailures()
    ->then(function (Batch $batch) {
        Log::info("Import finished with {$batch->failedJobs} failures");
    })
    ->dispatch();
```

**Expected Output**

If row 2 fails, rows 1 and 3 still process. The `then` callback fires after all rows are processed, and `failedJobs` reports `1`.

**Why This Output Occurs**

`allowFailures()` prevents the failure from cancelling the batch; `then` fires once all jobs have been attempted (success or failure).

---

**Example 2: Default Failure Policy**

```php
$batch = Bus::batch([
    new ImportRow(1),
    new ImportRow(2),
    new ImportRow(3),
])
    ->catch(function (Batch $batch, Throwable $e) {
        Log::error("Import aborted: {$e->getMessage()}");
    })
    ->dispatch();
```

**Expected Output**

If row 2 fails, row 3 is not processed, and the `catch` callback fires.

**Why This Output Occurs**

The default policy cancels the batch on the first failure and invokes `catch`.

---

#### Real-World Cases with Explanation

**Case 1: CSV Import with Partial Success**

A CSV import batch uses `allowFailures()` so that invalid rows do not prevent valid rows from being imported. After the batch finishes, the application reports the number of failed rows and offers a retry for those rows.

**Explanation**: `allowFailures()` is essential for workloads where partial success is acceptable and desirable.

---

### 6. Batch Best Practices

#### Definitions

**Core Definition**

Batch best practices are the guidelines for designing, dispatching, monitoring, and cleaning up batches to ensure reliability, observability, and maintainability.

#### Key Best Practices

1. **Use `Batchable` on Every Job**: Every job in a batch must use the `Illuminate\Bus\Batchable` trait; otherwise, `$this->batch()` will fail.

2. **Check `cancelled()` in Long-Running Jobs**: Jobs that run for a long time should check `$this->batch()->cancelled()` and return early if the batch was cancelled.

3. **Name Your Batches**: Use `->name('...')` to make batches identifiable in the database and in monitoring dashboards.

4. **Choose the Failure Policy Deliberately**: Use the default (cancel on failure) when all jobs must succeed; use `allowFailures()` when partial success is acceptable.

5. **Use `then` for Success, `finally` for Cleanup**: Do not rely on `then` for cleanup; use `finally` so cleanup runs regardless of outcome.

6. **Add Jobs Dynamically for Large Workloads**: For thousands of jobs that would be slow to dispatch in a web request, dispatch a small batch of “loader” jobs that call `$this->batch()->add([...])` to hydrate the batch.

7. **Prune Batches Regularly**: Schedule `queue:prune-batches` daily. Use `--unfinished` and `--cancelled` to prune stale records. By default, finished batches older than 24 hours are pruned.

8. **Poll Progress from a Route**: Expose `Bus::findBatch($id)` from a route or controller to let the frontend poll progress.

9. **Avoid Huge Batches**: Very large batches can overwhelm the queue and the `job_batches` table. Use the loader pattern to add jobs incrementally.

10. **Use `progress` Callback for Incremental Updates**: Attach a `progress` callback to update counters or caches after each successful job.

11. **Test Batches with `Bus::fake()`**: Use `Bus::assertBatched()` to assert that a batch was dispatched, and `Bus::assertDispatched()` for individual jobs.

12. **Monitor Failed Jobs**: Failed batch jobs appear in `failed_jobs`; use `queue:retry` to retry them after fixing the underlying issue.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Best-Practice Batch**

```php
$batch = Bus::batch([
    new ImportCsv(1, 1000),
    new ImportCsv(1001, 2000),
])
    ->name('CSV Import')
    ->allowFailures()
    ->onQueue('imports')
    ->then(function (Batch $batch) {
        Log::info("Import {$batch->id} finished with {$batch->failedJobs} failures");
    })
    ->finally(function (Batch $batch) {
        Cache::forget("import_progress:{$batch->id}");
    })
    ->progress(function (Batch $batch) {
        Cache::put("import_progress:{$batch->id}", $batch->progress());
    })
    ->dispatch();
```

**Expected Output**

The batch is named, allows failures, runs on the `imports` queue, logs success with failure count, clears cache in `finally`, and updates progress in cache after each job.

**Why This Output Occurs**

Each best practice is applied: naming, failure policy, queue targeting, lifecycle callbacks, and progress reporting.

---

**Example 2: Pruning Batches**

```php
// routes/console.php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches')->daily();
```

**Expected Output**

Completed batches older than 24 hours are pruned daily, keeping the `job_batches` table manageable.

**Why This Output Occurs**

`queue:prune-batches` removes finished batch records according to the configured retention.

---

#### Real-World Cases with Explanation

**Case 1: Large-Scale Data Import**

A data platform imports millions of rows nightly. Instead of dispatching millions of jobs at once, it dispatches a small batch of 10 “loader” jobs. Each loader adds 100,000 import jobs to the same batch. The batch uses `allowFailures()`, reports progress via polling, and is pruned daily.

**Explanation**: Combining the loader pattern, `allowFailures()`, progress polling, and pruning makes even very large batch workloads manageable.

---

## References

- Queues — Laravel 13.x Documentation (Job Batching) — https://laravel.com/framework/docs/queues#5
- Queues — Laravel 12.x Documentation (Batch Failures, Pruning) — https://laravel.com/docs/12.x/queues
- Queues — Laravel 11.x Documentation (Chained Jobs in Batches, Adding Jobs, Inspecting) — https://laravel.com/framework/docs/11.x/queues#4
- Queues — Laravel 10.x Documentation (Inspecting Batches, findBatch) — https://laravel.com/docs/10.x/queues
- Queues — Laravel 9.x Documentation (Cancelling, Failures, Pruning) — https://laravel.com/docs/9.x/queues
- Batching Reference — fusengine/agents — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-queues/references/batching.md
- Batch Failure Handling — voku/agent-skills — https://github.com/voku/agent-skills/blob/main/skills/laravel-queues/rules/bus-batch-failure-handling.md
- Simplified Batch Job Creation with `--batched` — Laravel News — https://laravel-news.com/…
- `Bus::findBatch()` API — https://api.laravel.com/docs/12.x/Illuminate/Support/Facades/Bus.html
- Pruning Batches (Laravel 12.x) — https://laravel.com/docs/12.x/queues#pruning-batches

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Job batching has been available since Laravel 8. The `Batchable` trait, `Bus::batch()`, lifecycle callbacks (`then`, `catch`, `finally`, `progress`), `allowFailures()`, dynamic job addition, and `Bus::findBatch()` are stable across these versions. The `queue:prune-batches` command supports `--unfinished` and `--cancelled` options for more aggressive pruning. The `--batched` flag for `make:job` is available in Laravel 12+. Always consult the documentation for your specific Laravel version.