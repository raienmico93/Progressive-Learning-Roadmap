# Laravel Queue Workers & Horizon: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel queue workers are long-running background processes that continuously poll a queue connection for pending jobs and execute them, decoupling slow or resource-intensive work from the HTTP request lifecycle. **Laravel Horizon** is a first-party dashboard and code-driven process manager that supervises Redis-backed queue workers, providing autoscaling, real-time monitoring, and metrics.

**Technical Definition**

A queue worker is a CLI process started with `php artisan queue:work` that uses the queue driver’s API to reserve, execute, and acknowledge or release jobs. Workers are configured via CLI options (`--queue`, `--tries`, `--timeout`, `--sleep`, `--max-time`, `--max-jobs`, `--memory`) and require a process monitor in production to restart them on failure. **Horizon** replaces hand-written Supervisor configurations with a single `config/horizon.php` file, introduces supervisor groups with `minProcesses` / `maxProcesses` bounds and balancing strategies (`simple`, `auto`, `false`), and exposes a web dashboard at `/horizon` with throughput, runtime, wait time, and failure metrics.

**Beginner-Friendly Explanation**

Imagine a restaurant kitchen. The **queue worker** is the cook who keeps checking the order tickets and preparing each dish. A **process manager** (Supervisor) is the kitchen manager who makes sure the cook never leaves their station—if they faint, the manager revives them. **Horizon** is a smart kitchen manager for restaurants that use a specific type of order system (Redis): it watches how many orders are waiting, hires extra cooks automatically when it gets busy, and gives you a live dashboard showing every order’s progress.

---

### Key Characteristics

- **Workers Are Long-Lived, Stateful Processes**: Unlike PHP-FPM requests, workers boot the framework once and reuse it across many jobs. Code changes require a restart (`queue:restart` or `horizon:terminate`).
- **Process Monitoring Is Mandatory in Production**: Workers can crash due to memory limits, exceptions, or server restarts. Supervisor or Horizon must restart them automatically.
- **Scaling Is Horizontal and Queue-Aware**: More workers (more processes) increase throughput. Multiple queues with priority (`--queue=high,default,low`) ensure critical jobs are not starved.
- **Horizon Requires Redis**: Horizon supervises Redis queue connections only; it does not support the database, SQS, or Beanstalkd drivers.
- **Timeout Chain Ordering Is Critical**: The job `timeout` must be less than the supervisor `timeout`, which must be less than `retry_after`. Wrong ordering causes duplicate job execution.
- **Horizon Metrics Require Scheduling**: The metrics dashboard stays blank until `horizon:snapshot` is scheduled every five minutes.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with `config/queue.php` and a configured queue connection.
- **A queue backend**: Redis (required for Horizon), database, SQS, or Beanstalkd.
- **`laravel/horizon`** installed via Composer for Horizon users.
- **Supervisor** installed on the server (`apt-get install supervisor` on Ubuntu) for process monitoring.
- **`predis/predis`** or the `phpredis` extension for Redis connections.
- **The `pcntl` PHP extension** for signal-based worker termination and timeouts.

---

### Related Programming Areas

- **Queues**: Workers consume jobs from queues; queue configuration determines worker behaviour.
- **Jobs**: The unit of work executed by workers.
- **Laravel Horizon**: A Redis-specific worker manager and monitoring dashboard.
- **Process Supervision**: Supervisor, systemd, or container orchestrators keep workers alive.
- **Task Scheduling**: `horizon:snapshot` must be scheduled; scheduled commands can dispatch jobs.
- **Deployment**: Workers must be restarted (`queue:restart` / `horizon:terminate`) on every deploy.

---

## Core Concepts

---

### 1. Queue Workers

#### Definitions

**Core Definition**

A queue worker is a long-running CLI process that polls a queue connection for pending jobs and executes them, using the framework’s queue driver to reserve, run, and acknowledge or release each job.

**Technical Definition**

`php artisan queue:work {connection} --queue={queues} --tries={n} --timeout={s} --sleep={s} --max-time={s} --max-jobs={n} --memory={mb}` starts a daemon worker. The worker boots the application once and enters a loop: reserve a job, fire `JobProcessing`, execute `handle()`, fire `JobProcessed` on success or release/retry on failure, then sleep if no job is available. `queue:work` is the production worker; `queue:listen` restarts the framework after each job and is intended for development.

**Beginner-Friendly Explanation**

A worker is the “cook” that keeps checking the order queue. It runs continuously, picks up jobs one by one, and does the work. You run one or more workers depending on how much work you have.

#### Purposes

- To process queued jobs asynchronously, keeping web requests fast.
- To execute background work reliably in a long-lived process.
- To scale throughput horizontally by running multiple workers.
- To control job execution with timeouts, retries, and memory limits.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Basic worker
php artisan queue:work

# Specific connection and queue priority
php artisan queue:work redis --queue=high,default,low

# Retry, timeout, and sleep configuration
php artisan queue:work --tries=3 --timeout=60 --sleep=3 --backoff=30

# Graceful shutdown after limits
php artisan queue:work --max-jobs=1000 --max-time=3600 --memory=512
```

**Component Breakdown**

| Option | Description |
|--------|-------------|
| `{connection}` | Queue connection (e.g., `redis`, `database`). |
| `--queue` | Comma-separated priority list. |
| `--tries` | Max attempts per job. |
| `--timeout` | Seconds before the job is killed. |
| `--sleep` | Seconds to sleep when no job is available. |
| `--max-time` | Stop after N seconds. |
| `--max-jobs` | Stop after N jobs. |
| `--memory` | Memory limit in MB. |

**Syntax Rules**

- `queue:work` is the production worker; `queue:listen` is for development.
- `--timeout` must be shorter than the connection’s `retry_after`.
- Workers cache code at boot; run `php artisan queue:restart` after deploying changes.

**Constraints and Limitations**

- Workers are stateful; memory leaks in jobs accumulate over time. Use `--max-jobs` or `--max-time` for periodic restarts.
- `queue:listen` is significantly slower and not recommended for production.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Starting a Worker with Priority Queues**

```bash
php artisan queue:work redis --queue=high,default,low --tries=3 --timeout=60
```

**Expected Output**

The worker processes all `high` jobs before moving to `default`, and all `default` before `low`, on every poll cycle. Each job is attempted up to 3 times with a 60-second timeout.

**Why This Output Occurs**

The `--queue` option provides a priority list; the worker polls queues in order on each cycle. `--tries` and `--timeout` configure retry and execution limits.

---

**Example 2: Graceful Worker with Limits**

```bash
php artisan queue:work --max-jobs=1000 --max-time=3600
```

**Expected Output**

The worker exits gracefully after processing 1,000 jobs or after 3,600 seconds, whichever comes first. A process monitor (Supervisor or Horizon) then restarts it with a fresh process.

**Why This Output Occurs**

`--max-jobs` and `--max-time` instruct the worker to stop after the specified limit, releasing accumulated memory.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Email Processing**

An e-commerce application sends order confirmations, shipping updates, and promotional emails. Separate workers process the `emails` queue with `--tries=5` and `--timeout=30`, ensuring transient SMTP failures are retried without blocking other work.

**Explanation**: Dedicated workers per workload isolate failure domains and allow independent scaling.

---

### 2. Worker Configuration

#### Definitions

**Core Definition**

Worker configuration is the set of CLI options, connection settings, and job-level properties that control how a worker reserves, executes, retries, and times out jobs.

**Technical Definition**

Worker configuration spans three layers: **CLI options** (`--queue`, `--tries`, `--timeout`, `--sleep`, `--backoff`, `--max-jobs`, `--max-time`, `--memory`), **connection configuration** (`retry_after`, `block_for`, `queue`), and **job-level properties** (`$tries`, `$timeout`, `$backoff`, `$maxExceptions`, `$failOnTimeout`). The worker’s `--timeout` must be several seconds shorter than the connection’s `retry_after` to prevent duplicate execution. The timeout chain is: **job `timeout` < supervisor `timeout` < `retry_after`**.

**Beginner-Friendly Explanation**

Configuration is the control panel: how long a job can run, how many times to retry, how long to wait between retries, and when to restart the worker to avoid memory leaks.

#### Purposes

- To tune job execution to the workload’s reliability and latency requirements.
- To prevent duplicate job execution caused by timeout and retry misalignment.
- To control worker memory usage with periodic restarts.
- To prioritize critical queues over lower-priority work.

#### Syntax Rules and Structure

**Complete General Syntax**

**CLI configuration**

```bash
php artisan queue:work redis --queue=high,default --tries=3 --timeout=60 --sleep=3 --backoff=30 --max-time=3600 --memory=512
```

**Connection configuration (`config/queue.php`)**

```php
'redis' => [
    'driver' => 'redis',
    'queue' => 'default',
    'retry_after' => 90,
    'block_for' => null,
],
```

**Job-level configuration**

```php
public int $tries = 3;
public int $timeout = 60;
public int $backoff = 30;
public bool $failOnTimeout = true;
```

**Component Breakdown**

| Layer | Key Settings |
|-------|--------------|
| CLI | `--queue`, `--tries`, `--timeout`, `--sleep`, `--backoff`, `--max-jobs`, `--max-time`, `--memory` |
| Connection | `retry_after`, `block_for`, `queue` |
| Job | `$tries`, `$timeout`, `$backoff`, `$maxExceptions`, `$failOnTimeout` |

**Syntax Rules**

- `--timeout` must be less than `retry_after` with a safety margin of at least a few seconds.
- `--tries` on the worker overrides the job’s `$tries` for that worker.
- `--max-time` and `--max-jobs` trigger graceful worker exits for periodic restarts.

**Constraints and Limitations**

- If `retry_after` is shorter than the worker `--timeout`, a slow job is released and can run twice simultaneously.
- Worker-level `--tries` does not override a job’s `retryUntil()` deadline.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Aligned Timeout and Retry**

```php
// config/queue.php
'redis' => [
    'retry_after' => 90,
],
```

```bash
php artisan queue:work redis --timeout=60 --tries=3
```

**Expected Output**

A job that exceeds 60 seconds is killed by the worker before the 90-second `retry_after` window expires, preventing a second worker from picking it up while it is still running.

**Why This Output Occurs**

The timeout (60s) is shorter than `retry_after` (90s), ensuring the worker terminates the job before the queue considers it lost and releases it again.

---

**Example 2: Job-Level Overrides**

```php
class SendReport implements ShouldQueue
{
    public int $tries = 5;
    public int $timeout = 120;
    public array $backoff = [10, 30, 60];
}
```

**Expected Output**

This specific job is attempted up to 5 times, each attempt limited to 120 seconds, with escalating backoff delays of 10, 30, and 60 seconds.

**Why This Output Occurs**

Job-level properties override worker-level defaults for this job, allowing per-job tuning.

---

#### Real-World Cases with Explanation

**Case 1: Preventing Duplicate Invoice Generation**

An invoice job takes 45 seconds on average. The connection’s `retry_after` is set to 90 seconds, and the worker `--timeout` is set to 75 seconds. This prevents the job from being retried while still running, avoiding duplicate invoices.

**Explanation**: Timeout and retry alignment is a critical correctness safeguard, not just a performance tuning parameter.

---

### 3. Worker Scaling

#### Definitions

**Core Definition**

Worker scaling is the process of increasing or decreasing the number of worker processes to match queue workload, either manually (more processes) or automatically (Horizon’s autoscaling).

**Technical Definition**

Scaling can be **vertical** (more workers on the same server) or **horizontal** (more servers/containers). With plain workers, scaling means increasing `numprocs` in Supervisor or starting more `queue:work` processes. With Horizon, scaling is controlled per supervisor via `minProcesses` and `maxProcesses`, and the `balance` strategy (`auto`, `simple`, `false`) determines how processes are distributed across queues while the application is running.

**Beginner-Friendly Explanation**

Scaling means hiring more cooks when the queue gets long, and letting some go home when it is quiet. Horizon can do this automatically; with Supervisor, you edit the number of processes and reload.

#### Purposes

- To increase job throughput when queues back up.
- To reduce idle workers during quiet periods (with autoscaling).
- To ensure critical queues get more processing capacity.
- To handle traffic spikes without manual intervention.

#### Syntax Rules and Structure

**Complete General Syntax**

**Manual scaling (Supervisor)**

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/app/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
numprocs=8
```

**Horizon autoscaling (`config/horizon.php`)**

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default', 'notifications'],
            'balance' => 'auto',
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

**Component Breakdown**

| Strategy | Behaviour |
|----------|-----------|
| `simple` | Splits processes evenly across queues (default). |
| `auto` | Scales each queue between `minProcesses` and `maxProcesses` based on workload. |
| `false` | Fixed process count; processes queues strictly in priority order. |

**Syntax Rules**

- `minProcesses` must be ≥ 1; `maxProcesses` is the ceiling for the entire supervisor.
- `balance` `auto` requires both `minProcesses` and `maxProcesses`.
- With `balance` `false`, Horizon processes queues in strict priority order (exhausts `default` before `notifications`).

**Constraints and Limitations**

- Horizon autoscaling applies only to Redis connections.
- Over-provisioning `maxProcesses` can exhaust server memory and CPU.
- `balance` `false` with `maxProcesses` can cause all processes to be allocated to the first queue, starving later queues.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Horizon Auto-Balancing**

```php
'supervisor-1' => [
    'connection' => 'redis',
    'queue' => ['default', 'notifications'],
    'balance' => 'auto',
    'minProcesses' => 1,
    'maxProcesses' => 10,
],
```

**Expected Output**

When both queues are busy, Horizon scales up to 10 total processes, allocating them proportionally to the queues’ workloads. Each queue always retains at least 1 process.

**Why This Output Occurs**

`balance` `auto` monitors queue sizes and adjusts process allocation between `minProcesses` and `maxProcesses` dynamically.

---

**Example 2: Priority-Only Processing**

```php
'supervisor-1' => [
    'queue' => ['default', 'notifications'],
    'balance' => false,
],
```

**Expected Output**

Horizon fully processes all `default` jobs before starting any `notifications` jobs.

**Why This Output Occurs**

`balance` `false` uses a fixed pool and strict priority ordering; it does not distribute processes across queues.

---

#### Real-World Cases with Explanation

**Case 1: Black Friday Traffic Spike**

An e-commerce platform runs Horizon with `balance => 'auto'`, `minProcesses => 2`, `maxProcesses => 20` on its `orders` queue. During Black Friday, queue depth spikes and Horizon scales to 20 processes automatically; after the spike, it scales back to 2.

**Explanation**: Autoscaling matches capacity to demand without manual intervention.

---

### 4. Worker Supervision

#### Definitions

**Core Definition**

Worker supervision is the practice of running queue workers under a process monitor that automatically restarts them if they crash, exit, or are killed, ensuring continuous job processing.

**Technical Definition**

Process monitors for Laravel workers include **Supervisor** (Linux process control system), **systemd** (service manager), **Laravel Horizon** (Redis-specific), and container orchestrators (Kubernetes, ECS). In production, a worker must never be run manually; it must be supervised so that crashes, memory exhaustion, and deployments do not silently halt processing. After deploying code, `php artisan queue:restart` signals workers to exit gracefully so the monitor restarts them with the new code.

**Beginner-Friendly Explanation**

Supervision is the manager that makes sure the cook never leaves the kitchen. If the cook faints (the worker crashes), the manager revives them (restarts the process) immediately.

#### Purposes

- To guarantee that queue processing continues after crashes or restarts.
- To automatically restart workers after deployments.
- To manage multiple worker processes (scaling).
- To capture worker output and errors in logs.

#### Syntax Rules and Structure

**Complete General Syntax**

**Supervisor configuration**

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/app/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=8
redirect_stderr=true
stdout_logfile=/var/www/app/storage/logs/worker.log
stopwaitsecs=3600
```

**Supervisor commands**

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

**Component Breakdown**

| Directive | Description |
|-----------|-------------|
| `autostart` | Start the program when Supervisor starts. |
| `autorestart` | Restart the program if it exits. |
| `numprocs` | Number of worker processes. |
| `stopasgroup` / `killasgroup` | Ensure child processes are stopped/killed with the parent. |
| `stopwaitsecs` | Seconds to wait for graceful shutdown. |

**Syntax Rules**

- `autorestart=true` is mandatory for production reliability.
- `stopasgroup=true` and `killasgroup=true` prevent orphaned worker processes.
- `stopwaitsecs` should exceed the longest job timeout to allow graceful completion.

**Constraints and Limitations**

- Supervisor does not provide queue metrics or a dashboard; it only keeps processes alive.
- Supervisor configuration is static; scaling requires editing `numprocs` and reloading.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Supervisor Configuration**

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/html/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=8
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/worker.log
stopwaitsecs=3600
```

**Expected Output**

Eight worker processes run continuously, restart automatically on failure, and restart hourly (`--max-time=3600`) to release accumulated memory.

**Why This Output Occurs**

Supervisor manages each process per the directives; `numprocs=8` starts eight parallel workers, and `autorestart=true` keeps them alive.

---

**Example 2: Reloading Supervisor After Configuration Change**

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

**Expected Output**

Supervisor reads the new configuration, applies it, and starts the worker processes.

**Why This Output Occurs**

`reread` parses the configuration; `update` applies changes; `start` launches the programs.

---

#### Real-World Cases with Explanation

**Case 1: Single VPS Deployment**

A small SaaS application runs on a single VPS with Supervisor managing eight workers on the `emails` queue. When a worker crashes due to a memory leak, Supervisor restarts it within seconds, and the `--max-time=3600` option ensures fresh processes every hour.

**Explanation**: Supervisor is the pragmatic choice for single-server deployments where Horizon’s Redis-only constraint or additional complexity is unnecessary.

---

### 5. Supervisor

#### Definitions

**Core Definition**

Supervisor is a Linux process control system that monitors and controls configured processes, automatically restarting them when they exit unexpectedly.

**Technical Definition**

Supervisor reads INI-style configuration files from `/etc/supervisor/conf.d/`, starts the configured programs, and restarts them according to the `autostart` and `autorestart` directives. It provides `supervisorctl` for runtime management (`status`, `start`, `stop`, `restart`, `reread`, `update`). Laravel’s documentation recommends Supervisor for running `queue:work` in production.

**Beginner-Friendly Explanation**

Supervisor is the “manager” that watches your workers. If a worker stops, Supervisor starts it again. You configure it once, and it handles the rest.

#### Purposes

- To keep queue workers running continuously on Linux servers.
- To restart workers after crashes or memory exhaustion.
- To manage multiple worker processes from a single configuration.
- To provide a simple, reliable supervision solution without additional dependencies.

#### Syntax Rules and Structure

**Complete General Syntax**

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /path/to/artisan queue:work {connection} --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=8
redirect_stderr=true
stdout_logfile=/path/to/storage/logs/worker.log
stopwaitsecs=3600
```

**Component Breakdown**

| Section | Purpose |
|---------|---------|
| `[program:name]` | Defines a program. |
| `command` | The Artisan command to run. |
| `numprocs` | Number of worker processes. |
| `autostart` / `autorestart` | Start on boot / restart on exit. |
| `stopasgroup` / `killasgroup` | Kill child processes with the parent. |
| `user` | The user to run the process as. |

**Syntax Rules**

- Configuration files belong in `/etc/supervisor/conf.d/`.
- `autorestart=true` is required for reliability.
- `stopasgroup=true` and `killasgroup=true` prevent orphaned processes.

**Constraints and Limitations**

- Supervisor is Linux-only.
- It does not provide queue metrics, dashboards, or autoscaling.
- Scaling requires editing `numprocs` and reloading.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Multiple Queues Under Supervisor**

```ini
[program:laravel-worker-default]
command=php /var/www/app/artisan queue:work redis --queue=default --sleep=3 --tries=3 --max-time=3600
numprocs=4
autostart=true
autorestart=true

[program:laravel-worker-emails]
command=php /var/www/app/artisan queue:work redis --queue=emails --sleep=3 --tries=5 --max-time=3600
numprocs=2
autostart=true
autorestart=true
```

**Expected Output**

Four workers process the `default` queue, and two workers process the `emails` queue independently, each with its own retry and timeout configuration.

**Why This Output Occurs**

Separate `[program:...]` blocks allow different worker configurations per queue.

---

#### Real-World Cases with Explanation

**Case 1: Hybrid Queue Strategy**

A team uses Supervisor for database and SQS queues (which Horizon does not support) and Horizon for Redis queues. Supervisor keeps the non-Redis workers alive, while Horizon handles Redis with autoscaling and metrics.

**Explanation**: Supervisor and Horizon are complementary; Supervisor can manage any queue driver, while Horizon is Redis-only.

---

### 6. Horizon

#### Definitions

**Core Definition**

Laravel Horizon is a first-party package that provides a code-driven configuration and dashboard for supervising Redis-backed queue workers, with autoscaling, real-time monitoring, and metrics.

**Technical Definition**

Horizon (`laravel/horizon`) replaces manual Supervisor configurations with `config/horizon.php`, which defines supervisors (groups of workers), queues, balancing strategies, and process bounds. It runs as a single `php artisan horizon` master process that spawns and manages worker processes. The `/horizon` dashboard displays throughput, runtime, wait time, failed jobs, and job tags. Horizon stores metrics in Redis and requires the `horizon:snapshot` command to be scheduled every five minutes to populate the metrics graphs.

**Beginner-Friendly Explanation**

Horizon is a smart kitchen manager for Redis queues. It watches the queue, hires and fires cooks automatically, and gives you a live dashboard showing every order’s status. You configure it in one PHP file instead of many Supervisor files.

#### Purposes

- To replace hand-written Supervisor configurations with a single, version-controlled PHP file.
- To autoscale worker processes based on queue workload.
- To provide real-time visibility into queue throughput, wait times, and failures.
- To offer job tagging, filtering, and retry from the browser.

#### Syntax Rules and Structure

**Installation**

```bash
composer require laravel/horizon
php artisan horizon:install
php artisan migrate
```

**Configuration (`config/horizon.php`)**

```php
'defaults' => [
    'supervisor-1' => [
        'connection' => 'redis',
        'queue' => ['default'],
        'balance' => 'auto',
        'minProcesses' => 1,
        'maxProcesses' => 10,
        'tries' => 3,
    ],
],

'environments' => [
    'production' => [
        'supervisor-1' => [
            'maxProcesses' => 20,
        ],
    ],
    'local' => [
        'supervisor-1' => [
            'maxProcesses' => 2,
        ],
    ],
],
```

**Running Horizon**

```bash
php artisan horizon
php artisan horizon:status
php artisan horizon:terminate
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `defaults` | Base supervisor configuration. |
| `environments` | Per-environment overrides merged into defaults. |
| `balance` | `simple`, `auto`, or `false`. |
| `minProcesses` / `maxProcesses` | Autoscaling bounds. |
| `horizon:snapshot` | Scheduled command for metrics. |

**Syntax Rules**

- Horizon requires the `redis` queue connection and does not support database or SQS drivers.
- `environments` keys merge into `defaults`; they do not replace the entire supervisor block.
- The `horizon:snapshot` command must be scheduled every five minutes for metrics to appear.

**Constraints and Limitations**

- Horizon is Redis-only; it does not supervise database, SQS, or Beanstalkd queues.
- Redis Cluster is not supported; Horizon requires a standalone Redis connection.
- The metrics dashboard remains blank until `horizon:snapshot` is scheduled.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Installing and Starting Horizon**

```bash
composer require laravel/horizon
php artisan horizon:install
php artisan migrate
php artisan horizon
```

**Expected Output**

Horizon starts and supervises the configured workers. Visiting `/horizon` shows the dashboard (once authorization is configured).

**Why This Output Occurs**

`horizon:install` publishes the configuration and service provider; `php artisan horizon` starts the master process that spawns workers.

---

**Example 2: Environment-Specific Scaling**

```php
'environments' => [
    'production' => [
        'supervisor-1' => ['maxProcesses' => 20],
    ],
    'local' => [
        'supervisor-1' => ['maxProcesses' => 2],
    ],
],
```

**Expected Output**

In production, the supervisor scales up to 20 processes; locally, it scales up to 2.

**Why This Output Occurs**

The `environments` array overrides specific keys of the `defaults` supervisor block per environment.

---

#### Real-World Cases with Explanation

**Case 1: Migrating from Supervisor to Horizon**

A team running plain `queue:work` under Supervisor migrates to Horizon on a Redis connection. They gain autoscaling, a dashboard, and job tagging, and remove their hand-written Supervisor INI files. They keep Supervisor only to monitor the `php artisan horizon` master process itself.

**Explanation**: Horizon replaces Supervisor for worker management on Redis, but Supervisor is still used to keep the Horizon master process alive.

---

### 7. Horizon Installation

#### Definitions

**Core Definition**

Horizon installation is the process of adding the `laravel/horizon` package, running `horizon:install`, migrating the database, and configuring dashboard authorization.

**Technical Definition**

Installation steps: (1) `composer require laravel/horizon`; (2) `php artisan horizon:install`, which publishes `config/horizon.php` and `app/Providers/HorizonServiceProvider.php`; (3) `php artisan migrate` (Horizon uses the existing Redis connection but may need migrations for certain features); (4) configure the `viewHorizon` gate in `HorizonServiceProvider` to restrict dashboard access; (5) start Horizon with `php artisan horizon`.

**Beginner-Friendly Explanation**

Installing Horizon is four commands: install the package, run the install command, migrate, and set up who can see the dashboard.

#### Purposes

- To publish Horizon’s configuration and authorization scaffolding.
- To restrict dashboard access to authorized users.
- To prepare the application for Horizon’s monitoring and autoscaling.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
composer require laravel/horizon
php artisan horizon:install
php artisan migrate
```

**Dashboard authorization (`app/Providers/HorizonServiceProvider.php`)**

```php
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user) {
        return $user->is_admin;
    });
}
```

**Component Breakdown**

| Step | Command / File | Purpose |
|------|----------------|---------|
| 1 | `composer require laravel/horizon` | Adds the package. |
| 2 | `horizon:install` | Publishes config and provider. |
| 3 | `migrate` | Runs any required migrations. |
| 4 | `HorizonServiceProvider::gate()` | Restricts dashboard access. |

**Syntax Rules**

- The `viewHorizon` gate must be defined to secure the dashboard.
- `horizon:install` publishes `config/horizon.php` and `HorizonServiceProvider`.
- Horizon should be installed as a production dependency (not `--dev`).

**Constraints and Limitations**

- Without the gate, the dashboard may be publicly accessible (depending on environment).
- Horizon requires a working Redis connection; installation does not validate Redis connectivity.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Securing the Horizon Dashboard**

```php
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user) {
        return $user->is_admin;
    });
}
```

**Expected Output**

Only users with `is_admin = true` can access `/horizon`; others receive a 403.

**Why This Output Occurs**

The `viewHorizon` gate is checked by Horizon’s middleware before rendering the dashboard.

---

#### Real-World Cases with Explanation

**Case 1: Restricting Access in Production**

A team deploys Horizon and configures the `viewHorizon` gate to allow only users with the `admin` role. They verify that a non-admin user receives a 403 when visiting `/horizon`, preventing sensitive queue data from being exposed.

**Explanation**: Dashboard authorization is a mandatory security step, not an optional convenience.

---

### 8. Horizon Configuration

#### Definitions

**Core Definition**

Horizon configuration is the definition of supervisors, queues, balancing strategies, and environment-specific overrides in `config/horizon.php`.

**Technical Definition**

`config/horizon.php` contains `defaults` (base supervisor definitions) and `environments` (per-environment overrides). Each supervisor block specifies `connection`, `queue`, `balance`, `minProcesses`, `maxProcesses`, `tries`, `timeout`, `nice`, and `balanceCooldown`. The `environments` array merges into `defaults`; it does not replace the entire supervisor block. The `waits` configuration defines the `LongWaitDetected` threshold per queue.

**Beginner-Friendly Explanation**

Configuration is the “recipe” for Horizon: which queues to process, how many workers to run, how aggressively to scale, and when to alert you about long wait times.

#### Purposes

- To define worker groups (supervisors) per environment.
- To control autoscaling bounds and balancing strategy.
- To set retry and timeout defaults for all jobs on a supervisor.
- To configure wait-time alerts and metrics retention.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'defaults' => [
    'supervisor-1' => [
        'connection' => 'redis',
        'queue' => ['default'],
        'balance' => 'auto',
        'minProcesses' => 1,
        'maxProcesses' => 10,
        'tries' => 3,
        'timeout' => 60,
    ],
],

'environments' => [
    'production' => [
        'supervisor-1' => ['maxProcesses' => 20],
    ],
    'local' => [
        'supervisor-1' => ['maxProcesses' => 2],
    ],
],

'waits' => [
    'redis:default' => 60,
],
```

**Component Breakdown**

| Key | Description |
|-----|-------------|
| `connection` | Queue connection (must be `redis`). |
| `queue` | Array of queue names (priority order). |
| `balance` | `simple`, `auto`, or `false`. |
| `minProcesses` / `maxProcesses` | Autoscaling bounds. |
| `tries` / `timeout` | Job retry and timeout defaults. |
| `waits` | `LongWaitDetected` thresholds. |

**Syntax Rules**

- `connection` must be a Redis connection.
- `environments` merges into `defaults`; specify only the keys you want to override.
- `waits` keys use `{connection}:{queue}` format, with values in seconds.

**Constraints and Limitations**

- Multiple supervisors can be defined to isolate workloads (e.g., one for `default`, one for `emails`).
- `balance` `auto` requires both `minProcesses` and `maxProcesses`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Multi-Supervisor Configuration**

```php
'defaults' => [
    'supervisor-default' => [
        'connection' => 'redis',
        'queue' => ['default'],
        'balance' => 'auto',
        'minProcesses' => 1,
        'maxProcesses' => 10,
    ],
    'supervisor-emails' => [
        'connection' => 'redis',
        'queue' => ['emails'],
        'balance' => 'simple',
        'maxProcesses' => 4,
    ],
],
```

**Expected Output**

Horizon runs two independent supervisors: one autoscaling the `default` queue between 1 and 10 processes, and one running a fixed 4 processes for `emails`.

**Why This Output Occurs**

Separate supervisor blocks allow independent configuration and scaling per workload.

---

#### Real-World Cases with Explanation

**Case 1: Isolating Critical Workloads**

A team defines two supervisors: one for `payments` with `minProcesses=2` and `maxProcesses=4`, and one for `analytics` with `minProcesses=1` and `maxProcesses=20`. Payment processing is never starved by analytics load.

**Explanation**: Multiple supervisors isolate capacity between critical and non-critical workloads.

---

### 9. Horizon Monitoring

#### Definitions

**Core Definition**

Horizon monitoring is the use of the `/horizon` dashboard and CLI commands to observe queue throughput, job runtime, wait times, failures, and worker status in real time.

**Technical Definition**

The Horizon dashboard displays **throughput** (jobs per minute), **runtime** (execution time), **wait time** (time in queue before processing), **failed jobs** (with stack traces and payloads), and **job tags** (for filtering). CLI commands include `horizon:status`, `horizon:supervisor-status`, `horizon:pause`, `horizon:continue`, and `horizon:terminate`. The `LongWaitDetected` event fires when a queue’s wait time exceeds the `waits` threshold.

**Beginner-Friendly Explanation**

Monitoring is the dashboard that shows you what your workers are doing: how many jobs are waiting, how long they are waiting, which ones failed, and whether your autoscaling is keeping up.

#### Purposes

- To detect queue backups and bottlenecks before they affect users.
- To diagnose failed jobs with full payloads and stack traces.
- To verify that autoscaling is responding to load.
- To pause or terminate workers during deployments or incidents.

#### Syntax Rules and Structure

**Dashboard**

```
https://your-app.com/horizon
```

**CLI monitoring**

```bash
php artisan horizon:status
php artisan horizon:supervisor-status supervisor-1
php artisan horizon:pause
php artisan horizon:continue
php artisan horizon:terminate
```

**LongWaitDetected configuration (`config/horizon.php`)**

```php
'waits' => [
    'redis:default' => 60,
    'redis:emails' => 120,
],
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Throughput | Jobs processed per minute. |
| Runtime | Average job execution time. |
| Wait time | Time jobs spend in the queue. |
| Failed jobs | Stack traces, payloads, retry buttons. |
| Tags | Filter jobs by model or custom tag. |

**Syntax Rules**

- The dashboard is served at `/horizon` (configurable in `config/horizon.php`).
- `waits` keys must match the connection and queue names.
- `horizon:terminate` gracefully stops all workers after current jobs finish.

**Constraints and Limitations**

- The dashboard does not populate metrics until `horizon:snapshot` is scheduled.
- Tagging requires jobs to implement `tags()` or use automatic model tagging.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Detecting Long Wait Times**

```php
'waits' => [
    'redis:default' => 60,
],
```

**Expected Output**

If a job waits more than 60 seconds in the `default` queue, Horizon fires a `LongWaitDetected` event, which can be routed to Slack or email.

**Why This Output Occurs**

Horizon monitors wait time against the configured threshold and dispatches the event when exceeded.

---

**Example 2: Pausing Workers During Deployment**

```bash
php artisan horizon:pause
# deploy code
php artisan horizon:continue
```

**Expected Output**

Horizon stops picking up new jobs while paused, allowing in-flight jobs to finish; after `continue`, it resumes processing.

**Why This Output Occurs**

`horizon:pause` instructs the master process to stop dispatching new jobs to workers until resumed.

---

#### Real-World Cases with Explanation

**Case 1: Incident Response**

During a production incident, the on-call engineer opens `/horizon`, sees that the `emails` queue has a 40-minute wait time, and uses the dashboard to identify a stuck job. They retry the failed job and pause the queue while investigating, preventing further backlog.

**Explanation**: Horizon’s monitoring turns opaque queue behaviour into actionable operational intelligence.

---

### 10. Horizon Metrics

#### Definitions

**Core Definition**

Horizon metrics are the recorded snapshots of queue throughput, wait time, and job counts that populate the dashboard’s graphs, generated by the scheduled `horizon:snapshot` command.

**Technical Definition**

The `horizon:snapshot` Artisan command captures the current queue metrics and stores them in Redis. The metrics graphs are built from these snapshots, which must be scheduled to run every five minutes via Laravel’s scheduler. Without scheduling, the dashboard’s metrics section remains blank even if Horizon is running. Metrics include jobs per minute, wait time per queue, and failed job counts.

**Beginner-Friendly Explanation**

Metrics are the “history graphs” on the Horizon dashboard. They show how your queues performed over time. But the graphs stay empty until you schedule the snapshot command to record data every five minutes.

#### Purposes

- To visualise queue performance trends over time.
- To identify recurring bottlenecks and capacity issues.
- To correlate deployments or traffic spikes with queue behaviour.
- To provide data for capacity planning and autoscaling tuning.

#### Syntax Rules and Structure

**Scheduling the snapshot (`routes/console.php` or `app/Console/Kernel.php`)**

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

**Or via Artisan**

```bash
php artisan horizon:snapshot
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `horizon:snapshot` | Captures current metrics. |
| `everyFiveMinutes()` | Recommended schedule interval. |
| Dashboard graphs | Built from snapshots. |

**Syntax Rules**

- The snapshot command must be scheduled; running Horizon alone does not generate metrics.
- The recommended interval is every five minutes.
- Metrics are stored in Redis and retained according to Horizon’s configuration.

**Constraints and Limitations**

- Metrics are not real-time; they are snapshots at five-minute intervals.
- If the scheduler is not running, no metrics are recorded.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Scheduling the Snapshot**

```php
// routes/console.php (Laravel 11+)
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

**Expected Output**

Every five minutes, Laravel’s scheduler runs `horizon:snapshot`, capturing the current queue metrics and populating the dashboard graphs.

**Why This Output Occurs**

The snapshot command records the metrics that the dashboard reads; without scheduling, the graphs remain blank.

---

#### Real-World Cases with Explanation

**Case 1: Capacity Planning**

A team schedules `horizon:snapshot` and reviews the metrics graphs weekly. They notice that the `orders` queue consistently spikes every Monday at 9 AM, and they pre-emptively increase `maxProcesses` for that supervisor.

**Explanation**: Historical metrics enable proactive capacity planning rather than reactive firefighting.

---

### 11. Horizon Best Practices

#### Definitions

**Core Definition**

Horizon best practices are the guidelines for configuring, deploying, and monitoring Horizon to achieve reliable, scalable, and observable queue processing on Redis.

#### Key Best Practices

1. **Use Horizon Only with Redis**: Horizon does not support database, SQS, or Beanstalkd drivers. If you use those drivers, use Supervisor instead.

2. **Enforce the Timeout Chain**: Job `timeout` < supervisor `timeout` < `retry_after`. Wrong ordering causes duplicate job execution.

3. **Schedule `horizon:snapshot`**: The metrics dashboard stays blank until this command runs every five minutes.

4. **Secure the Dashboard**: Define the `viewHorizon` gate to restrict access to authorized users only.

5. **Use `balance => 'auto'` for Variable Loads**: Autoscaling between `minProcesses` and `maxProcesses` matches capacity to demand.

6. **Isolate Workloads with Multiple Supervisors**: Run separate supervisors for critical queues (e.g., payments) and background queues (e.g., analytics) to prevent starvation.

7. **Run Horizon Under Supervisor**: Use Supervisor to monitor the `php artisan horizon` master process itself, ensuring it restarts if it crashes.

8. **Terminate Horizon on Deploy**: Run `php artisan horizon:terminate` during deployment so Supervisor restarts it with the new code.

9. **Configure `LongWaitDetected` Alerts**: Set `waits` thresholds and route the event to Slack or email for early warning of backups.

10. **Tag Jobs for Filtering**: Implement `tags()` on jobs or rely on automatic model tagging to make the dashboard searchable.

11. **Monitor Memory and Scale Conservatively**: Over-provisioning `maxProcesses` can exhaust server memory; tune based on available RAM and CPU.

12. **Keep Horizon and Workers on Separate Clusters**: For large applications, run Horizon workers on dedicated infrastructure separate from web servers.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Production-Ready Horizon Configuration**

```php
'defaults' => [
    'supervisor-default' => [
        'connection' => 'redis',
        'queue' => ['default', 'notifications'],
        'balance' => 'auto',
        'minProcesses' => 1,
        'maxProcesses' => 10,
        'tries' => 3,
        'timeout' => 60,
    ],
    'supervisor-payments' => [
        'connection' => 'redis',
        'queue' => ['payments'],
        'balance' => 'simple',
        'maxProcesses' => 4,
        'tries' => 5,
        'timeout' => 90,
    ],
],

'environments' => [
    'production' => [
        'supervisor-default' => ['maxProcesses' => 20],
        'supervisor-payments' => ['maxProcesses' => 8],
    ],
],

'waits' => [
    'redis:default' => 60,
    'redis:payments' => 30,
],
```

```php
// routes/console.php
Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

**Expected Output**

In production, Horizon autoscales the `default` queue up to 20 processes and runs 8 fixed processes for `payments`. Payments jobs alert after 30 seconds of waiting; default jobs alert after 60 seconds. Metrics graphs populate every five minutes.

**Why This Output Occurs**

The configuration combines multiple supervisors, environment overrides, autoscaling, wait-time alerts, and scheduled metrics—each aligned with the best practices above.

---

#### Real-World Cases with Explanation

**Case 1: Mature Production Setup**

A high-traffic SaaS application runs Horizon under Supervisor on a dedicated worker cluster. They schedule `horizon:snapshot` every five minutes, alert on `LongWaitDetected` at 60 seconds, use `balance => 'auto'` with `maxProcesses` tuned to their server’s memory, and terminate Horizon on every deploy. The queue system handles millions of jobs per day with sub-minute wait times.

**Explanation**: Combining Horizon’s autoscaling and monitoring with Supervisor’s process-level resilience yields a production-grade queue infrastructure.

---

## References

- Laravel Horizon — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/horizon
- Laravel Horizon — Laravel 10.x Documentation — https://laravel.com/docs/10.x/horizon
- Queues — Laravel 11.x Documentation (Supervisor, Horizon) — https://laravel.com/docs/11.x/queues
- Laravel Queue Workers (OneUptime Blog) — https://github.com/oneuptime/blog/tree/master/posts/2025-07-02-laravel-queue-workers
- Laravel Horizon: Monitor and Scale Redis Queues in 2026 — https://laracopilot.com/blog/laravel-horizon-redis-queues/
- Running Laravel Queue Workers in Production — Supervisor vs Horizon — https://medium.com/@chinthakahasan/running-laravel-queue-workers-in-production-supervisor-vs-horizon-229e04e7dd3f
- Configuring Horizon (Coolify Skill) — https://github.com/coollabsio/coolify/blob/abc701ac6503afc602661fcfc52e0c455f289e73/.cursor/skills/configuring-horizon/SKILL.md
- Horizon Balancing Strategies — https://laravel.com/docs/6.x/horizon
- Metrics & Snapshots (Horizon) — https://github.com/laravel/horizon/blob/1.x/docs/metrics.md
- Timeout vs. `retry_after` Best Practices — https://dev.to/…/laravel-job-timeout-vs-retry_after-the-ordering-rule-nothing-enforces
- Supervisor Configuration for Laravel Workers — https://laravel.com/docs/10.x/queues#supervisor-configuration

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Horizon requires Redis and does not support database, SQS, or Beanstalkd drivers. The `balance` strategies (`simple`, `auto`, `false`) and `minProcesses` / `maxProcesses` are available across these versions. `horizon:snapshot` scheduling is mandatory for metrics. The timeout chain (job `timeout` < supervisor `timeout` < `retry_after`) must be respected to prevent duplicate execution. Always consult the documentation for your specific Laravel version.