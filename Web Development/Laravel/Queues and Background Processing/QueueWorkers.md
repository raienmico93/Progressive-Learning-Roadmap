# Laravel Queue Workers & Runtime — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Queue Workers & Runtime is the subsystem responsible for executing queued jobs in background processes, encompassing worker orchestration commands (`queue:work`, `queue:listen`), queue priority routing, resource and timeout guardrails, and graceful lifecycle management during deployments.

**Technical Definition:** Queue workers are long-lived PHP processes that poll a queue backend, retrieve serialized job payloads, resolve the job classes from the service container, and invoke their `handle()` methods. The `Illuminate\Queue\Worker` class manages the worker lifecycle, using `WorkerOptions` to configure memory limits, timeouts, sleep intervals, and job retry limits. The `queue:work` command (implemented by `Illuminate\Queue\Console\WorkCommand`) starts a daemon worker that processes jobs continuously, while `queue:listen` (implemented by `Illuminate\Queue\Console\ListenCommand`) spawns a new `queue:work --once` subprocess for each job, reloading the framework between jobs. Queue priorities are managed by passing a comma-separated list of queue names to `--queue`, with workers polling queues in priority order. Graceful maintenance is handled by `queue:restart`, which signals workers to exit after completing their current job, and process managers like Supervisor or Horizon which automatically restart them.

**Beginner-Friendly Explanation:** Queue workers are the "background employees" of your application. While your web server handles user requests, queue workers process slow tasks like sending emails, processing videos, or syncing data. You start them with `php artisan queue:work`, and they run continuously, picking up jobs as they arrive. During deployments, you need to restart them so they pick up your new code — that's where `queue:restart` and process managers come in. This cheat sheet covers how to run, prioritize, monitor, and gracefully maintain these workers in production.

### Key Characteristics

- **Long-lived daemon processes:** `queue:work` boots the framework once and keeps it in memory, making it highly efficient but requiring restarts after code changes.
- **Zero-overhead reload alternative:** `queue:listen` spawns a fresh subprocess for each job, reloading the framework every time — slower but always picks up code changes.
- **Priority-based polling:** Workers poll multiple queues in the order specified, draining higher-priority queues first on each poll cycle.
- **Resource guardrails:** Memory limits, timeouts, and process recycling parameters prevent workers from degrading over time.
- **Graceful shutdown:** `queue:restart` signals workers to finish their current job and exit, enabling zero-downtime deployments.
- **Process manager integration:** Supervisor and Horizon ensure workers are automatically restarted if they crash or exit.
- **Horizon orchestration:** For Redis-backed queues, Horizon provides code-driven worker configuration, auto-balancing, and real-time monitoring.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with `config/queue.php` configured.
- A queue backend (Redis, database, SQS, etc.) with a running worker or process manager.
- For production: a process manager such as Supervisor (or Laravel Horizon for Redis queues).
- The `pcntl` PHP extension (required for timeout support in workers).
- A persistent cache driver (Redis, Memcached, database) for `queue:restart` to work correctly.

### Related Programming Areas

- **Artisan Console** — Queue workers are started and managed via Artisan commands.
- **Queue System** — The foundation for job dispatching and processing.
- **Process Management** — Supervisor and systemd keep workers running and restart them when they exit.
- **Laravel Horizon** — Code-driven worker configuration, auto-balancing, and monitoring for Redis queues.
- **Deployment** — `queue:restart` is a standard step in zero-downtime deployment scripts.
- **Caching** — The restart signal is stored in the cache, requiring a persistent cache driver.

### Core Concepts / Features

1. Worker Orchestration (Managing differences between standard daemon processes `queue:work` and zero-overhead environments `queue:listen`)
2. Queue Priorities (Directing worker focus across multi-tiered queues like `high`, `default`, `low`)
3. Resource & Timeout Guardrails (Monitoring worker lifecycles, memory limits, and process recycling parameters)
4. Graceful Maintenance (Pausing, draining, and restarting worker pools seamlessly during zero-downtime application deployments)

---

## 1. Worker Orchestration

### Definitions

**Core Definition:** Worker orchestration is the practice of starting, managing, and configuring queue worker processes, including choosing between the high-performance daemon mode (`queue:work`) and the code-reload-friendly listener mode (`queue:listen`).

**Technical Definition:** The `queue:work` command starts a daemon worker that boots the Laravel application once and processes jobs continuously in a loop. It is highly efficient but does not pick up code changes until restarted. The `queue:listen` command is a wrapper that spawns a new `queue:work --once` subprocess for each job, reloading the framework every time. Internally, `queue:listen` creates a child process using Symfony's `Process` class and invokes `queue:work --once` in a `while` loop. The `Worker` class (`Illuminate\Queue\Worker`) manages the worker lifecycle, using `WorkerOptions` to configure memory, timeout, sleep, and retry parameters.

**Beginner-Friendly Explanation:** `queue:work` is the production worker — it starts once, boots your application into memory, and processes jobs as fast as possible. The catch is that if you change your code, the worker doesn't see the changes until you restart it. `queue:listen` is the development worker — it starts a fresh process for every job, so it always uses the latest code, but it's much slower because it has to boot the entire framework each time.

### Purposes

- To provide a high-performance daemon worker for production environments (`queue:work`).
- To provide a code-reload-friendly worker for development environments (`queue:listen`).
- To start workers with specific connections, queues, and options.
- To control worker behaviour through options like `--once`, `--stop-when-empty`, and `--max-jobs`.
- To integrate with process managers for automatic restart and monitoring.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Start a daemon worker (production)
php artisan queue:work [connection] [options]

# Start a listener worker (development)
php artisan queue:listen [connection] [options]
```

**Common Options:**

| Option | Description |
|--------|-------------|
| `--queue=high,default,low` | Comma-separated list of queues to process in priority order |
| `--once` | Process a single job and exit |
| `--stop-when-empty` | Process all jobs and exit when the queue is empty |
| `--max-jobs=1000` | Process N jobs and exit |
| `--max-time=3600` | Process jobs for N seconds and exit |
| `--tries=3` | Maximum number of attempts per job |
| `--timeout=60` | Maximum seconds a job can run |
| `--sleep=3` | Seconds to sleep when the queue is empty |
| `--memory=128` | Maximum RAM the worker may consume (MB) |
| `--backoff=0` | Seconds to wait before retrying a failed job |
| `--force` | Run the worker even in maintenance mode |

```bash
# Production: daemon worker with priority queues
php artisan queue:work redis --queue=high,default,low --tries=3 --timeout=90 --sleep=3

# Development: listener worker (auto-reloads code)
php artisan queue:listen redis --queue=default

# Process a single job and exit
php artisan queue:work --once

# Process all jobs and exit (useful for Docker)
php artisan queue:work --stop-when-empty
```

**Component Breakdown:**

- `queue:work` — Starts a daemon worker. Boots the framework once and keeps it in memory.
- `queue:listen` — Starts a listener that spawns a new `queue:work --once` subprocess for each job.
- `--once` — Processes one job and exits. Used internally by `queue:listen`.
- `--stop-when-empty` — Processes all available jobs and exits gracefully.
- `--max-jobs` / `--max-time` — Process recycling parameters (see Section 3).
- `--memory` — Memory limit in megabytes (default: 128).
- `--timeout` — Job timeout in seconds (default: 60).

**Syntax Rules:**

- `queue:work` runs indefinitely until stopped manually or by a process manager.
- `queue:listen` is significantly less efficient than `queue:work` because it boots the framework for each job.
- The `--once` option is used by `queue:listen` internally and can also be used standalone for testing.
- The `--stop-when-empty` option is useful for Docker containers and batch processing.
- The `pcntl` extension is required for the `--timeout` option to work correctly.

**Constraints and Limitations:**

- **Daemon workers do not pick up code changes.** You must restart them after deployment via `queue:restart`.
- **`queue:listen` is much slower than `queue:work`.** It is suitable only for local development or debugging.
- **Daemon workers do not "reboot" the framework between jobs.** Heavy resources (database connections, file handles) must be released manually.
- **The `--timeout` option requires the `pcntl` extension.** Without it, timeouts are not enforced.
- **Workers are long-lived processes.** Memory leaks in job code accumulate over time; use process recycling to mitigate.

### Annotated Code Examples

**Example 1: Starting a Production Daemon Worker**

```bash
# Step 1: Start a daemon worker with priority queues
php artisan queue:work redis --queue=high,default,low --tries=3 --timeout=90 --sleep=3 --memory=256
```

**Expected Output:**

```
Processing jobs from the [high,default,low] queue.

  2025-06-15 10:00:00 App\Jobs\SendEmail ....................... RUNNING
  2025-06-15 10:00:01 App\Jobs\SendEmail ....................... DONE
  2025-06-15 10:00:05 App\Jobs\ProcessOrder .................... RUNNING
  2025-06-15 10:00:08 App\Jobs\ProcessOrder .................... DONE
```

**Why This Output Occurs:** The worker connects to the `redis` connection, boots the framework once, and begins polling the `high` queue first. When `high` is empty, it moves to `default`, then `low`. The `--tries=3` option limits each job to 3 attempts, `--timeout=90` kills jobs running longer than 90 seconds, and `--memory=256` restarts the worker if it exceeds 256 MB.

---

**Example 2: Using queue:listen for Development**

```bash
# Start a listener worker that reloads code on every job
php artisan queue:listen redis --queue=default --tries=3
```

**Expected Output:**

```
Listening to the [default] queue on the [redis] connection.

  2025-06-15 10:00:00 App\Jobs\SendEmail ....................... RUNNING
  2025-06-15 10:00:01 App\Jobs\SendEmail ....................... DONE
```

**Why This Output Occurs:** The `queue:listen` command spawns a new `queue:work --once` subprocess for each job. Each subprocess boots the Laravel framework from scratch, picks up any code changes, processes one job, and exits. The parent listener then spawns a new subprocess for the next job.

### Real-World Cases

- **Production daemon workers:** Run `queue:work` under Supervisor with `--max-time=3600` to recycle workers hourly and release accumulated memory.
- **Local development:** Use `queue:listen` to avoid restarting the worker after every code change.
- **Docker containers:** Use `queue:work --stop-when-empty` to process all jobs and exit, allowing the container to shut down cleanly.
- **Batch processing:** Use `queue:work --once` in a loop or `--max-jobs=100` to process a limited number of jobs and exit.
- **Multi-connection setups:** Run separate workers for different connections (e.g., `queue:work redis` and `queue:work sqs`).

---

## 2. Queue Priorities

### Definitions

**Core Definition:** Queue priorities are the mechanism by which workers process jobs from multiple queues in a defined order, ensuring that critical jobs are handled before less important ones.

**Technical Definition:** The `--queue` option accepts a comma-separated list of queue names. The worker polls the queues in the order specified: it checks the first queue for available jobs, then the second, and so on. If a job is found in a higher-priority queue, it is processed before any jobs in lower-priority queues. This polling happens on each cycle — the worker does not drain the first queue entirely before moving to the second. Jobs are routed to specific queues using the `onQueue()` method on the job instance or the `$queue` property on the job class.

**Beginner-Friendly Explanation:** Queue priorities let you say "process password reset emails before newsletter emails." You assign each job to a queue — `high`, `default`, or `low` — and then start a worker with `--queue=high,default,low`. The worker always checks `high` first, then `default`, then `low`. If a `high` job arrives while the worker is processing a `low` job, the `high` job waits until the current job finishes, but it will be picked up before the next `low` job.

### Purposes

- To ensure that critical jobs (password resets, payment processing, alerts) are processed before routine jobs.
- To prevent low-priority jobs (newsletters, report generation) from blocking high-priority work.
- To segment workloads across multiple queues for independent scaling and monitoring.
- To allow different queues to have different retry, timeout, and memory configurations.
- To enable dedicated workers for specific queues (e.g., a worker that only processes `urgent` jobs).

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Start a worker with priority queues
php artisan queue:work --queue=high,default,low
```

```php
// Assign a job to a specific queue
dispatch((new ProcessOrder($order))->onQueue('high'));

// Or set the queue on the job class
class ProcessOrder implements ShouldQueue
{
    public $queue = 'high';
}
```

```php
// Configuring default queue in config/queue.php
'redis' => [
    'driver'  => 'redis',
    'queue'   => env('REDIS_QUEUE', 'default'),
    // ...
],
```

**Component Breakdown:**

- `--queue=high,default,low` — Comma-separated list of queues in priority order.
- `->onQueue('high')` — Assigns the job to the `high` queue at dispatch time.
- `public $queue = 'high'` — Sets the default queue for the job class.
- `'queue' => 'default'` — The default queue name in the connection configuration.

**Syntax Rules:**

- The first queue in the `--queue` list has the highest priority.
- The worker polls queues in order on each cycle; it does not drain one queue before moving to the next.
- Jobs are routed to queues using `onQueue()` or the `$queue` property.
- If no queue is specified, the job goes to the connection's default queue.
- Multiple workers can be run with different `--queue` configurations for independent scaling.

**Constraints and Limitations:**

- **Queue priority is worker-level, not job-level.** You cannot assign a priority number to individual jobs; you assign them to named queues.
- **A job in a lower-priority queue may still run before a job in a higher-priority queue** if the worker is already processing it when the higher-priority job arrives.
- **Draining a queue does not mean it is empty.** The worker polls in order but may process jobs from lower-priority queues if the higher-priority queue is momentarily empty.
- **Horizon's auto-balancing may override manual priority.** When using Horizon's `auto` strategy, workers are allocated dynamically based on workload.

### Annotated Code Examples

**Example 1: Configuring Priority Queues**

```php
// Step 1: Assign jobs to priority queues
use App\Jobs\SendPasswordReset;
use App\Jobs\ProcessOrder;
use App\Jobs\SendNewsletter;

// High priority: password resets
SendPasswordReset::dispatch($user)->onQueue('high');

// Default priority: order processing
ProcessOrder::dispatch($order)->onQueue('default');

// Low priority: newsletters
SendNewsletter::dispatch($subscribers)->onQueue('low');
```

```bash
# Step 2: Start a worker with priority queues
php artisan queue:work redis --queue=high,default,low
```

**Expected Output:** The worker processes `high` queue jobs first, then `default`, then `low`. If a `high` job arrives while the worker is processing a `low` job, it waits until the current job finishes but is picked up before the next `low` job.

**Why This Output Occurs:** The `--queue=high,default,low` option tells the worker to poll queues in that order. On each cycle, the worker checks `high` first, then `default`, then `low`. This ensures that high-priority jobs are processed as soon as the worker is free, even if lower-priority jobs are waiting.

---

**Example 2: Running Dedicated Workers for Different Queues**

```bash
# Terminal 1: High-priority worker (2 processes)
php artisan queue:work redis --queue=high

# Terminal 2: Default-priority worker (4 processes)
php artisan queue:work redis --queue=default

# Terminal 3: Low-priority worker (1 process)
php artisan queue:work redis --queue=low
```

**Expected Output:** Each worker processes only its assigned queue. High-priority jobs are processed immediately by dedicated workers, while low-priority jobs wait for available capacity.

**Why This Output Occurs:** Running dedicated workers for each queue allows independent scaling. High-priority queues get more worker processes (or more powerful servers), while low-priority queues use fewer resources. This is more flexible than a single worker with `--queue=high,default,low` because each queue can have its own retry, timeout, and memory configuration.

### Real-World Cases

- **E-commerce order pipeline:** Order confirmations and payment processing run on `high`; inventory updates and shipping labels run on `default`; marketing emails and reports run on `low`.
- **SaaS incident management:** Critical alerts run on `urgent` with dedicated workers; user notifications run on `default`; analytics syncing runs on `low`.
- **Healthcare systems:** Appointment reminders and prescription notifications run on `high`; billing and claims processing run on `default`; report generation runs on `low`.
- **Financial services:** Transaction alerts run on `high`; statement generation runs on `default`; portfolio analysis runs on `low`.
- **Social media platforms:** Real-time engagement notifications (likes, comments) run on `high`; content moderation runs on `default`; weekly digest emails run on `low`.

---

## 3. Resource & Timeout Guardrails

### Definitions

**Core Definition:** Resource and timeout guardrails are the configuration parameters that prevent queue workers from consuming excessive memory, running indefinitely, or accumulating state over time, ensuring stable long-term operation.

**Technical Definition:** The `WorkerOptions` class (`Illuminate\Queue\WorkerOptions`) defines the configurable parameters for queue workers: `$memory` (maximum RAM in MB, default 128), `$timeout` (maximum seconds a child worker may run, default 60), `$sleep` (seconds to wait between polling, default 3), `$maxTries` (maximum attempts per job, default 1), `$maxJobs` (maximum jobs before exiting, default 0 = unlimited), `$maxTime` (maximum seconds a worker may live, default 0 = unlimited), and `$rest` (seconds to rest between jobs). When a worker exceeds its memory limit, it exits gracefully, and a process manager (Supervisor, Horizon) restarts it. When a job exceeds its timeout, the child worker process is killed, and the job is released back to the queue.

**Beginner-Friendly Explanation:** Queue workers are long-lived processes, which means they can accumulate memory leaks over time. If a worker runs out of memory, it crashes and stops processing jobs. Resource guardrails prevent this by setting a memory limit — when the worker reaches it, it exits gracefully and gets restarted by Supervisor. Similarly, the timeout prevents a single frozen job from blocking the worker forever. Process recycling (`--max-jobs`, `--max-time`) restarts workers periodically to release accumulated memory.

### Purposes

- To prevent workers from crashing due to memory exhaustion.
- To ensure that frozen or hung jobs do not block the queue indefinitely.
- To periodically recycle workers to release accumulated memory and database connections.
- To control the trade-off between throughput (more jobs per worker) and stability (more frequent restarts).
- To provide visibility into worker lifecycle events (memory exceeded, timeout, restart) for monitoring and alerting.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Worker with resource guardrails
php artisan queue:work [connection] --memory=256 --timeout=90 --sleep=3 --max-jobs=1000 --max-time=3600
```

**Component Breakdown:**

- `--memory=256` — Maximum RAM the worker may consume in megabytes. When exceeded, the worker exits gracefully.
- `--timeout=90` — Maximum seconds a child worker may run before being killed.
- `--sleep=3` — Seconds to sleep when no jobs are available.
- `--max-jobs=1000` — Process 1000 jobs and exit (process recycling).
- `--max-time=3600` — Process jobs for 1 hour and exit (process recycling).
- `--tries=3` — Maximum attempts per job.
- `--backoff=10` — Seconds to wait before retrying a failed job.

```bash
# Production worker with recycling
php artisan queue:work redis --queue=high,default --memory=256 --timeout=90 --max-time=3600 --tries=3

# Supervisor configuration (recommended for production)
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/app/artisan queue:work redis --queue=high,default --sleep=3 --tries=3 --max-time=3600 --memory=256
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
numprocs=4
redirect_stderr=true
stdout_logfile=/var/www/app/storage/logs/worker.log
stopwaitsecs=3600
```

**Syntax Rules:**

- The `--memory` option causes the worker to exit gracefully when exceeded, allowing the process manager to restart it.
- The `--timeout` option requires the `pcntl` extension to be installed.
- The `--max-jobs` and `--max-time` options are mutually exclusive alternatives for process recycling.
- The `stopwaitsecs` in Supervisor must be greater than the longest-running job's expected duration.
- Worker lifecycle events (memory exceeded, timeout, restart) are now logged by Laravel for easier diagnosis.

**Constraints and Limitations:**

- **Memory limits are per-worker, not per-job.** A single job can exceed the limit if its `handle()` method allocates too much memory.
- **The `--timeout` option kills the entire worker process, not just the job.** The job is released back to the queue for retry.
- **Process recycling adds overhead.** Each restart re-boots the framework, taking a few seconds.
- **The `--max-jobs` counter resets on worker restart.** If the worker crashes, the count is lost.
- **Horizon has its own `maxJobs` and `maxTime` configuration** that overrides the command-line options for Redis queues.

### Annotated Code Examples

**Example 1: Configuring Worker Guardrails**

```bash
# Production worker with comprehensive guardrails
php artisan queue:work redis \
    --queue=high,default,low \
    --tries=3 \
    --timeout=90 \
    --sleep=3 \
    --memory=256 \
    --max-jobs=1000 \
    --max-time=3600
```

**Expected Output:**

```
Processing jobs from the [high,default,low] queue.
Worker STARTED
  [2025-06-15 10:00:00] Job processed: App\Jobs\SendEmail
  [2025-06-15 10:00:01] Job processed: App\Jobs\ProcessOrder
  ...
Worker STOPPED Max time exceeded
```

**Why This Output Occurs:** The worker processes jobs until it reaches 1000 jobs or 3600 seconds (1 hour), whichever comes first. It also exits if memory usage exceeds 256 MB. When the worker exits, Supervisor automatically restarts it. The log message `Worker STOPPED Max time exceeded` indicates the reason for the restart.

---

**Example 2: Supervisor Configuration for Worker Recycling**

```ini
; /etc/supervisor/conf.d/laravel-worker.conf

[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/app/artisan queue:work redis --queue=high,default --sleep=3 --tries=3 --max-time=3600 --memory=256
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
numprocs=4
redirect_stderr=true
stdout_logfile=/var/www/app/storage/logs/worker.log
stopwaitsecs=3600
```

```bash
# Step 1: Reload Supervisor configuration
sudo supervisorctl reread
sudo supervisorctl update

# Step 2: Start the workers
sudo supervisorctl start laravel-worker:*
```

**Expected Output:** Supervisor starts 4 worker processes. Each worker processes jobs for 1 hour, then exits and is automatically restarted by Supervisor. If a worker exceeds 256 MB of memory, it exits and is restarted.

**Why This Output Occurs:** Supervisor monitors the worker processes and restarts them when they exit (whether from `--max-time`, `--memory`, or a crash). The `stopwaitsecs` setting tells Supervisor to wait up to 3600 seconds for a worker to finish its current job before forcefully killing it, ensuring graceful shutdown.

### Real-World Cases

- **High-volume e-commerce:** Workers are configured with `--max-time=1800` (30 minutes) and `--memory=512` to process thousands of orders per day without memory degradation.
- **Video processing platform:** Workers use `--timeout=600` (10 minutes) to accommodate long-running transcoding jobs, with `--max-jobs=100` to recycle frequently.
- **SaaS with Redis queues:** Horizon's `maxJobs` and `maxTime` configuration provides per-supervisor recycling, automatically balancing workers based on queue load.
- **Financial batch processing:** Workers run with `--memory=1024` (1 GB) for memory-intensive report generation, with `--max-time=7200` (2 hours) for long batch runs.
- **Multi-tenant applications:** Workers use `--max-jobs=500` to ensure fair resource allocation across tenants, preventing one tenant's jobs from monopolizing a worker.

---

## 4. Graceful Maintenance

### Definitions

**Core Definition:** Graceful maintenance is the practice of pausing, draining, and restarting queue workers during deployments without losing jobs or causing service interruptions, enabling zero-downtime application updates.

**Technical Definition:** The `queue:restart` command writes a restart signal to the application's cache. Workers check for this signal before processing the next job; if the signal is present, they exit gracefully after completing their current job, ensuring no jobs are lost. Process managers (Supervisor, Horizon) automatically restart the workers, which then pick up the new code. The `queue:restart` command requires a persistent cache driver (Redis, Memcached, database); the `array` driver will fail silently because it stores data in memory that is lost between requests. For Horizon, the equivalent command is `horizon:terminate`, which gracefully restarts the Horizon master process.

**Beginner-Friendly Explanation:** When you deploy new code, your queue workers are still running the old code. You can't just kill them because they might be in the middle of processing a job. The solution is `queue:restart` — it tells all workers to finish their current job and then exit. Supervisor immediately starts new workers with the new code. This is how you update your application without interrupting background processing.

### Purposes

- To ensure queue workers pick up new code after deployment.
- To prevent job loss during worker restarts.
- To enable zero-downtime deployments for applications with queue workers.
- To coordinate worker restarts across multiple servers and processes.
- To provide a clean, controlled shutdown of worker pools.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Gracefully restart all queue workers
php artisan queue:restart

# Gracefully restart Horizon (Redis queues)
php artisan horizon:terminate

# Forge zero-downtime deployment macro
$RESTART_QUEUES()
```

```bash
# Standard deployment script
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan optimize:clear
php artisan optimize
php artisan queue:restart  # Signal workers to restart
```

**Component Breakdown:**

- `queue:restart` — Writes a restart signal to the cache. Workers exit after their current job.
- `horizon:terminate` — Gracefully restarts the Horizon master process (for Redis queues).
- `$RESTART_QUEUES()` — Laravel Forge macro that handles queue restarts during zero-downtime deployments.
- The cache driver must persist data between requests (Redis, Memcached, database).

**Syntax Rules:**

- `queue:restart` requires a persistent cache driver. The `array` driver will fail silently.
- Workers check for the restart signal before processing each job.
- The signal is not instantaneous — workers may take a few seconds to notice it.
- Process managers must be configured to automatically restart workers when they exit.
- For Horizon, use `horizon:terminate` instead of `queue:restart`.

**Constraints and Limitations:**

- **`queue:restart` does not restart workers immediately.** It signals them to exit after their current job. Workers with long-running jobs may take minutes to restart.
- **The `array` cache driver silently fails with `queue:restart`.** Always use a persistent cache driver in production.
- **Workers on different servers may take different amounts of time to notice the signal.** The cache must be shared across all servers.
- **`queue:restart` does not drain the queue.** Jobs that are already in the queue remain there and are processed by the new workers.
- **Horizon's `horizon:terminate` is different from `queue:restart`.** It restarts the Horizon master process, which manages all supervisors and workers.

### Annotated Code Examples

**Example 1: Standard Deployment with queue:restart**

```bash
# Step 1: Put the application into maintenance mode (optional)
php artisan down --secret="deploy-secret"

# Step 2: Pull the latest code
git pull origin main

# Step 3: Install dependencies
composer install --no-dev --optimize-autoloader

# Step 4: Run migrations
php artisan migrate --force

# Step 5: Clear and rebuild caches
php artisan optimize:clear
php artisan optimize

# Step 6: Gracefully restart queue workers
php artisan queue:restart

# Step 7: Bring the application back online
php artisan up
```

**Expected Output:**

```
Application is now in maintenance mode.
...
Application is now live.
Broadcasting queue restart signal.
```

**Why This Output Occurs:** The `queue:restart` command writes a restart signal to the cache. All running workers check for this signal before processing their next job. When they detect it, they finish their current job and exit gracefully. Supervisor (or Horizon) automatically restarts them with the new code.

---

**Example 2: Horizon Deployment with horizon:terminate**

```bash
# Step 1: Pull the latest code
git pull origin main

# Step 2: Install dependencies
composer install --no-dev --optimize-autoloader

# Step 3: Run migrations
php artisan migrate --force

# Step 4: Clear and rebuild caches
php artisan optimize:clear
php artisan optimize

# Step 5: Gracefully terminate Horizon
php artisan horizon:terminate
```

**Expected Output:**

```
Horizon terminated successfully.
```

**Why This Output Occurs:** The `horizon:terminate` command signals the Horizon master process to gracefully terminate. Horizon waits for all in-progress jobs to complete, then exits. The process manager (Supervisor or Forge's daemon) automatically restarts Horizon with the new code. This is the recommended deployment method for Horizon-managed queues.

### Real-World Cases

- **Zero-downtime deployments:** Deploy new code while workers continue processing jobs; `queue:restart` ensures they pick up the new code without downtime.
- **Multi-server deployments:** In a load-balanced environment, `queue:restart` writes the signal to a shared cache (Redis), and all workers across all servers pick it up.
- **Horizon-managed queues:** Use `horizon:terminate` in deployment scripts for Redis-backed queues with Horizon.
- **Rolling deployments:** Restart workers one server at a time, ensuring that capacity is maintained throughout the deployment.
- **Scheduled maintenance windows:** Use `queue:restart` during planned maintenance to ensure workers are running the latest code after the window.

---

## References

- Laravel Queues Documentation (Master) — https://laravel.com/docs/master/queues
- Laravel Queues Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/queues
- Laravel Queues Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/queues
- Laravel Queues Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/queues
- Laravel Horizon Documentation — https://laravel.com/docs/12.x/horizon
- Laravel `WorkerOptions` API — https://api.laravel.com/docs/8.x/Illuminate/Queue/WorkerOptions.html
- Laravel `Worker` API — https://api.laravel.com/docs/12.x/Illuminate/Queue/Worker.html
- Laravel `queue:restart` Command Documentation — https://laravel.com/docs/master/queues#queue-workers-and-deployment
- Laravel `queue:listen` vs `queue:work` (Stack Overflow) — https://stackoverflow.com/questions/78516911/what-is-the-difference-queuework-and-queuelisten
- Laravel Worker Lifecycle and Memory Limits (Laravel News) — https://laravel-news.com/laravel-queue-work-now-prints-why-the-worker-stopped
- Laravel Forge Queues Documentation — https://laravel.com/forge/docs/sites/queues
- Laravel Queue Priorities (Laravel Docs) — https://laravel.com/docs/12.x/queues#queue-priorities
- Laravel Queue Worker Timeouts (Laravel Docs) — https://laravel.com/docs/12.x/queues#worker-timeouts
- Laravel Queue Worker Memory Limits (Laravel Docs) — https://laravel.com/docs/12.x/queues#worker-memory-limit
- Laravel Horizon Auto-Balancing — https://laravel.com/docs/12.x/horizon#balance-options
- Laravel Queue Deployment with Supervisor — https://laravel.com/docs/12.x/queues#supervisor-configuration
- Laravel `queue:restart` Cache Requirement (Laravel Forge) — https://laravel.com/forge/docs/sites/queues#restarting-queue-workers-after-deployment
- Laravel Zero-Downtime Deployments (Laravel Forge) — https://forge.laravel.com/docs/sites/deployments#zero-downtime-deployments