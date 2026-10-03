# Laravel Observability, Metrics & Resiliency — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Observability, Metrics & Resiliency in Laravel's Task Scheduling context is the discipline of instrumenting scheduled tasks with output capture, lifecycle hooks, heartbeat monitoring, deadman switches, and high-frequency tick scheduling, enabling operators to detect silent failures, measure performance, and guarantee that critical automation never goes dark.

**Technical Definition:** Observability in Laravel's scheduler is implemented through the `Illuminate\Console\Scheduling\Event` and `CallbackEvent` classes, which expose methods for output handling (`sendOutputTo()`, `appendOutputTo()`, `emailOutputTo()`, `emailOutputOnFailure()`), lifecycle callbacks (`before()`, `after()`, `onSuccess()`, `onFailure()`, `then()`, `finally()`), and ping-based heartbeat integration (`pingBefore()`, `thenPing()`, `pingOnSuccess()`, `pingOnFailure()`). Execution durations are captured in the `schedule:list` output and can be logged via callbacks. Deadman switches are implemented by combining heartbeat pings (using `pingBefore()` and `thenPing()`) with external monitoring platforms like Oh Dear, Flare, or Sentry. Sub-second execution is achieved by running `schedule:work` in a daemon loop that evaluates tasks every second, using `->everySecond()`, `->everyFiveSeconds()`, and similar frequency methods.

**Beginner-Friendly Explanation:** A scheduler that runs silently is a scheduler that fails silently. Observability means capturing output, logging successes and failures, and setting up alerts so you know immediately when something goes wrong. Deadman switches are the ultimate safety net: if your scheduled task doesn't "check in" on time, an external service alerts you. Sub-second scheduling lets you run tasks more frequently than once per minute — down to every second — for high-frequency workloads like metrics collection or queue polling.

### Key Characteristics

- **Output capture:** Task output can be appended to log files or overwritten for the latest run.
- **Email notifications:** Output can be emailed on every run or only on failure.
- **Lifecycle hooks:** `before()`, `after()`, `onSuccess()`, `onFailure()`, `then()`, and `finally()` provide precise points for logging, alerting, and cleanup.
- **Heartbeat pings:** `pingBefore()`, `thenPing()`, `pingOnSuccess()`, and `pingOnFailure()` notify external monitoring services before or after task execution.
- **Deadman switches:** External platforms (Oh Dear, Flare, Sentry) alert you when a task fails to check in within the expected window.
- **Sub-second execution:** `schedule:work` evaluates tasks every second, enabling `->everySecond()` and similar frequencies.
- **Execution duration tracking:** Task runtime is displayed by `schedule:list` and can be captured in callbacks for metrics.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the scheduler configured.
- For `schedule:work`: the `pcntl` extension (recommended) for signal handling.
- For heartbeat pings: an external monitoring service (Oh Dear, Healthchecks.io, etc.) with a ping URL.
- For error tracking: Flare, Sentry, or a similar platform with the appropriate SDK installed.
- For sub-second scheduling: a long-running `schedule:work` process managed by Supervisor.

### Related Programming Areas

- **Task Scheduling** — The foundation for all observability and resilience features.
- **Logging** — Output capture integrates with Laravel's logging system.
- **Mail** — Email notifications use the configured mail transport.
- **Error Tracking** — Flare, Sentry, and similar services capture exceptions.
- **Monitoring** — Oh Dear, Healthchecks.io, and UptimeRobot provide heartbeat monitoring.
- **Process Management** — Supervisor keeps `schedule:work` running in production.

### Core Concepts / Features

1. Output Capturing (Appending or overwriting terminal output records to specific disk files or emailing logs on system changes)
2. Lifecycle Hooks (Leveraging automation callbacks `before()`, `after()`, `onSuccess()`, `onFailure()` to wire up external alert channels)
3. Deadman Switches & Monitoring (Monitoring schedule heartbeats, tracking execution durations, and integrating third-party uptime platforms)
4. Sub-Second Execution (Tick scheduling — high-frequency automation strategies for micro-interval processes within modern background loops)

---

## 1. Output Capturing

### Definitions

**Core Definition:** Output capturing is the practice of recording the terminal output produced by a scheduled task — either by appending it to a log file, overwriting the previous run's output, or emailing it to administrators — enabling post-hoc inspection of task results and failures.

**Technical Definition:** The `Illuminate\Console\Scheduling\Event` class provides methods for output capture: `sendOutputTo($path)` writes the task's output to a file, overwriting any existing content; `appendOutputTo($path)` appends the output to a file; `emailOutputTo($addresses, $onlyIfOutputExists)` emails the output to the specified addresses; `emailOutputOnFailure($addresses)` emails the output only if the task fails; and `emailWrittenOutputTo($addresses)` emails the output only if it was written. The `runInBackground()` method changes output handling because the task runs in a subprocess; output redirection must be handled by the process itself. The `$output` property of the event stores the captured output, which is passed to lifecycle callbacks.

**Beginner-Friendly Explanation:** When a scheduled task runs, it produces output — messages like "Backup completed" or "Error: connection failed." By default, that output is discarded. Output capturing lets you save it to a file so you can inspect it later, or email it to yourself so you know what happened. You can choose to always capture output, or only capture it when something goes wrong.

### Purposes

- To preserve task output for post-hoc inspection and debugging.
- To maintain a historical record of task executions.
- To receive email notifications with the task's output on success or failure.
- To detect silent failures by examining the absence of expected output.
- To integrate with log aggregation systems by writing to files that are monitored.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Overwrite output to a file
Schedule::command('reports:generate')
    ->daily()
    ->sendOutputTo(storage_path('logs/reports.log'));

// Append output to a file
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->appendOutputTo(storage_path('logs/backup.log'));

// Email output to administrators
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->emailOutputTo('admin@example.com');

// Email output only on failure
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->emailOutputOnFailure('admin@example.com');

// Combine output capture with email on failure
Schedule::command('reports:generate')
    ->daily()
    ->appendOutputTo(storage_path('logs/reports.log'))
    ->emailOutputOnFailure('ops@example.com');

// Output capture with background execution
Schedule::command('import:large')
    ->dailyAt('01:00')
    ->runInBackground()
    ->appendOutputTo(storage_path('logs/import.log'));
```

**Component Breakdown:**

- `sendOutputTo($path)` — Writes the task's output to the specified file, overwriting existing content.
- `appendOutputTo($path)` — Appends the task's output to the specified file.
- `emailOutputTo($addresses, $onlyIfOutputExists = false)` — Emails the task's output to the specified addresses.
- `emailOutputOnFailure($addresses)` — Emails the output only if the task fails (non-zero exit code).
- `emailWrittenOutputTo($addresses)` — Emails the output only if it was actually written.
- `sendOutputTo($path, $append = true)` — Laravel 11+ supports appending via the second parameter.

**Syntax Rules:**

- The `$path` argument should be an absolute path (e.g., `storage_path('logs/task.log')`).
- The `$addresses` argument can be a string or an array of strings.
- The `emailOutputTo()` method requires a configured mail transport.
- Output capture with `runInBackground()` requires the task to redirect its own output or use `appendOutputTo()`.
- The `emailOutputOnFailure()` method is the recommended approach for production alerting.

**Constraints and Limitations:**

- **Output capture does not include stderr by default.** Use `2>&1` in shell commands to capture stderr.
- **Background tasks may not capture output correctly.** Use `appendOutputTo()` and ensure the task writes to stdout.
- **The `emailOutputTo()` method sends an email on every run**, which can be noisy. Use `emailOutputOnFailure()` for production.
- **Output files can grow indefinitely.** Use `sendOutputTo()` (overwrite) for high-frequency tasks, or implement log rotation.
- **The `runInBackground()` method may not capture output from subprocesses.** Test thoroughly.

### Annotated Code Examples

**Example 1: Comprehensive Output Capture Configuration**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Step 1: Overwrite output for a frequently running task
Schedule::command('metrics:collect')
    ->everyFiveMinutes()
    ->sendOutputTo(storage_path('logs/metrics.log'))
    ->description('Collect application metrics');

// Step 2: Append output for a daily backup with email on failure
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->appendOutputTo(storage_path('logs/backup.log'))
    ->emailOutputOnFailure('admin@example.com')
    ->description('Daily database backup');

// Step 3: Email output on every run for a critical task
Schedule::command('billing:process')
    ->dailyAt('03:00')
    ->emailOutputTo(['billing@example.com', 'ops@example.com'])
    ->description('Process billing');

// Step 4: Background task with output capture
Schedule::command('import:large-dataset')
    ->dailyAt('01:00')
    ->runInBackground()
    ->appendOutputTo(storage_path('logs/import.log'))
    ->emailOutputOnFailure('data-team@example.com')
    ->description('Import large dataset');
```

**Expected Output (storage/logs/backup.log):**

```
Starting backup...
Backing up database...
Backing up files...
Backup completed successfully in 45.2 seconds.
```

**Expected Output (email on failure):**

```
Subject: Scheduled Task Failed: backup:run

Command: php artisan backup:run
Exit Code: 1
Output:
  Starting backup...
  Error: Could not connect to database.
  Backup failed.
```

**Why This Output Occurs:** The `appendOutputTo()` method redirects the task's stdout to the specified file, appending each run's output. The `emailOutputOnFailure()` method configures the scheduler to email the output only if the task returns a non-zero exit code or throws an exception. The `sendOutputTo()` method overwrites the file on each run, which is appropriate for high-frequency tasks where only the latest output matters.

---

**Example 2: Shell Command Output Capture with stderr**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Capture both stdout and stderr from a shell command
Schedule::exec('pg_dump mydb > /backups/db.sql 2>> /backups/db-errors.log')
    ->dailyAt('02:00')
    ->appendOutputTo(storage_path('logs/pg_dump.log'))
    ->emailOutputOnFailure('dba@example.com')
    ->description('Database dump with error capture');
```

**Expected Output (storage/logs/pg_dump.log):**

```
pg_dump: dumping contents of table "users"
pg_dump: dumping contents of table "orders"
pg_dump: dumping contents of table "products"
```

**Expected Output (backups/db-errors.log on failure):**

```
pg_dump: error: connection to server at "localhost" failed: Connection refused
```

**Why This Output Occurs:** The shell command redirects stdout to the SQL dump file and stderr to a separate error log. The `appendOutputTo()` method captures the command's remaining output (none in this case). The `emailOutputOnFailure()` method sends an email if `pg_dump` returns a non-zero exit code. This pattern provides comprehensive visibility into shell command execution.

### Real-World Cases

- **Database backups:** Append output to a log file and email on failure, ensuring the DBA team is alerted if a backup fails.
- **Report generation:** Overwrite output on each run, providing the latest report generation status without accumulating logs.
- **Billing processing:** Email output on every run to the finance team, providing a complete audit trail.
- **Data imports:** Capture output from background imports, with email on failure to the data engineering team.
- **Health checks:** Append output to a log file for trend analysis, with email on failure to the on-call engineer.

---

## 2. Lifecycle Hooks

### Definitions

**Core Definition:** Lifecycle hooks are callback methods that execute at specific points in a scheduled task's lifecycle — before execution, after execution, on success, on failure, or regardless of outcome — enabling precise instrumentation, alerting, and cleanup.

**Technical Definition:** The `Illuminate\Console\Scheduling\Event` and `CallbackEvent` classes provide the following lifecycle methods: `before(Closure $callback)` — runs before the task executes; `after(Closure $callback)` — runs after the task completes (deprecated in favour of `then()` and `finally()`); `onSuccess(Closure $callback)` — runs if the task exits with a zero exit code; `onFailure(Closure $callback)` — runs if the task exits with a non-zero exit code; `then(Closure $callback)` — runs after successful completion; `finally(Closure $callback)` — runs regardless of outcome; and `thenWithOutput(Closure $callback)` — runs after completion with the task's output passed to the callback. These closures are serialized and executed by the `ScheduleRunCommand`.

**Beginner-Friendly Explanation:** Lifecycle hooks let you hook into a scheduled task's execution at specific moments. For example, you can log "starting backup" before the task runs, log "backup succeeded" after it completes, and send an alert if it fails. This gives you fine-grained visibility into what's happening and lets you wire up external alerting systems.

### Purposes

- To log task execution start, completion, success, and failure events.
- To send alerts to external systems (Slack, PagerDuty, Opsgenie) when tasks fail.
- To perform cleanup or state reset before or after task execution.
- To record execution durations for metrics and performance monitoring.
- To wire up custom logic that depends on the task's outcome.

### Syntax Rules and Structure

#### Complete General Syntax

```php
Schedule::command('backup:run')
    ->dailyAt('02:00')
    // Runs before the task executes
    ->before(function () {
        \Log::info('Backup starting...');
    })
    // Runs after successful completion
    ->onSuccess(function () {
        \Log::info('Backup succeeded.');
        \Http::post('https://hooks.slack.com/...', [
            'text' => '✅ Backup succeeded.',
        ]);
    })
    // Runs after failure
    ->onFailure(function () {
        \Log::error('Backup failed.');
        \Http::post('https://hooks.slack.com/...', [
            'text' => '🚨 Backup failed!',
        ]);
    })
    // Runs regardless of outcome
    ->finally(function () {
        Cache::forget('backup:running');
    })
    // Runs after success (alternative to onSuccess)
    ->then(function () {
        \Log::info('Backup then() callback.');
    })
    // Runs after completion with the task's output
    ->thenWithOutput(function (string $output) {
        \Log::info('Backup output: ' . $output);
    });
```

**Component Breakdown:**

- `before(Closure)` — Runs before the task executes.
- `after(Closure)` — Runs after the task completes (deprecated; use `then()` or `finally()`).
- `onSuccess(Closure)` — Runs if the task exits with a zero exit code.
- `onFailure(Closure)` — Runs if the task exits with a non-zero exit code.
- `then(Closure)` — Runs after successful completion.
- `finally(Closure)` — Runs regardless of outcome.
- `thenWithOutput(Closure)` — Runs after completion with the task's output.

**Syntax Rules:**

- The closures are serialized and executed by the `ScheduleRunCommand`.
- The closures do not receive arguments (except `thenWithOutput()`, which receives the output string).
- Multiple hooks of the same type can be chained; they execute in registration order.
- The `onFailure()` callback is called when the task returns a non-zero exit code or throws an exception.
- The `finally()` callback is always called, even if the task fails.

**Constraints and Limitations:**

- **The closures are serialized and executed later.** Do not use `$this` within them.
- **The `after()` method is deprecated.** Use `then()` or `finally()` instead.
- **The `onFailure()` callback is not called for manual job releases.** It is called only for actual failures.
- **Exceptions thrown in callbacks may be silently ignored.** Log them explicitly.
- **The callbacks are executed in the scheduler process, not the task's process.** They cannot access the task's internal state.

### Annotated Code Examples

**Example 1: Comprehensive Lifecycle Hook Configuration**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Schedule;

Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->before(function () {
        // Step 1: Mark the backup as running
        Cache::put('backup:running', true, 3600);
        Log::info('Backup starting...');
    })
    ->then(function () {
        // Step 2: Log success and notify Slack
        Log::info('Backup completed successfully.');
        Http::post(config('services.slack.webhook'), [
            'text' => '✅ Daily backup completed successfully.',
        ]);
    })
    ->onFailure(function () {
        // Step 3: Log failure and alert the on-call engineer
        Log::error('Backup failed.');
        Http::post(config('services.slack.webhook'), [
            'text' => '🚨 Daily backup FAILED. Check logs immediately.',
        ]);
        // Optionally, page the on-call engineer
        // PagerDuty::trigger('backup-failure', 'Daily backup failed');
    })
    ->finally(function () {
        // Step 4: Clean up the running flag
        Cache::forget('backup:running');
    })
    ->description('Daily database backup');
```

**Expected Output (on success):**

```
[2025-06-15 02:00:00] local.INFO: Backup starting...
[2025-06-15 02:00:45] local.INFO: Backup completed successfully.
```

**Expected Output (on failure):**

```
[2025-06-15 02:00:00] local.INFO: Backup starting...
[2025-06-15 02:00:30] local.ERROR: Backup failed.
```

**Why This Output Occurs:** The `before()` callback runs before the task, setting a cache flag and logging the start. The `then()` callback runs after successful completion, logging the success and sending a Slack notification. The `onFailure()` callback runs after a failure, logging the error and sending a Slack alert. The `finally()` callback always runs, cleaning up the cache flag.

---

**Example 2: Tracking Execution Duration with Lifecycle Hooks**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Schedule;

Schedule::command('reports:generate')
    ->dailyAt('06:00')
    ->before(function () {
        Cache::put('reports:start_time', microtime(true), 3600);
    })
    ->thenWithOutput(function (string $output) {
        $startTime = Cache::pull('reports:start_time');
        $duration = microtime(true) - $startTime;
        Log::info('Report generation completed', [
            'duration_seconds' => round($duration, 2),
            'output_length'    => strlen($output),
        ]);
    })
    ->onFailure(function () {
        $startTime = Cache::pull('reports:start_time');
        $duration = microtime(true) - $startTime;
        Log::error('Report generation failed', [
            'duration_seconds' => round($duration, 2),
        ]);
    })
    ->description('Generate daily report with duration tracking');
```

**Expected Output (storage/logs/laravel.log):**

```
[2025-06-15 06:00:00] local.INFO: Report generation completed {"duration_seconds":45.23,"output_length":1024}
```

**Why This Output Occurs:** The `before()` callback stores the start time in the cache. The `thenWithOutput()` callback retrieves the start time, calculates the duration, and logs it along with the output length. The `onFailure()` callback does the same for failed runs. This provides valuable metrics for performance monitoring.

### Real-World Cases

- **Backup monitoring:** Send Slack alerts on backup success and failure, with duration tracking.
- **Report generation:** Log execution duration for performance trending.
- **Billing processing:** Alert the finance team on failure and log success for audit.
- **Data synchronization:** Notify the data team on failure and track sync duration.
- **Health checks:** Alert the on-call engineer on failure and log recovery on success.

---

## 3. Deadman Switches & Monitoring

### Definitions

**Core Definition:** A deadman switch is a monitoring mechanism that alerts operators when a scheduled task fails to check in within an expected window. In Laravel, deadman switches are implemented by pinging an external monitoring service before or after task execution, combined with the service's alerting rules.

**Technical Definition:** The `Illuminate\Console\Scheduling\Event` class provides ping methods: `pingBefore($url)` — pings the URL before the task executes; `pingBeforeIf($condition, $url)` — pings conditionally; `thenPing($url)` — pings after the task completes; `thenPingIf($condition, $url)` — pings conditionally; `pingOnSuccess($url)` — pings only on success; `pingOnFailure($url)` — pings only on failure. These methods use `Illuminate\Support\Facades\Http` to send GET requests to the specified URLs. External monitoring platforms (Oh Dear, Healthchecks.io, UptimeRobot, Cronitor) provide a unique ping URL per task and alert if the ping is not received within the expected interval.

**Beginner-Friendly Explanation:** A deadman switch is like a "heartbeat" for your scheduled tasks. Every time the task runs, it sends a signal to an external monitoring service. If the service doesn't receive the signal within the expected time, it alerts you. This catches silent failures — cases where the task didn't run at all (because the cron stopped, the server crashed, or the scheduler was disabled). It's the ultimate safety net for critical automation.

### Purposes

- To detect when scheduled tasks fail to run (not just when they fail during execution).
- To alert operators when the scheduler itself stops working.
- To provide external, independent verification that tasks are executing on schedule.
- To track task execution frequency and detect anomalies.
- To integrate with established monitoring platforms for unified alerting.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Ping before task execution
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->pingBefore('https://hc-ping.com/your-uuid-here');

// Ping after successful completion
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->thenPing('https://hc-ping.com/your-uuid-here');

// Ping on success only
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->pingOnSuccess('https://hc-ping.com/your-uuid-here/success');

// Ping on failure only
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->pingOnFailure('https://hc-ping.com/your-uuid-here/fail');

// Conditional pings
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->pingBeforeIf($condition, 'https://hc-ping.com/your-uuid-here');

// Combined heartbeat pattern (ping before and after)
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->pingBefore('https://hc-ping.com/your-uuid-here/start')
    ->thenPing('https://hc-ping.com/your-uuid-here')
    ->pingOnFailure('https://hc-ping.com/your-uuid-here/fail');
```

**Component Breakdown:**

- `pingBefore($url)` — Sends a GET request to the URL before the task executes.
- `pingBeforeIf($condition, $url)` — Pings before execution only if the condition is true.
- `thenPing($url)` — Sends a GET request after the task completes.
- `thenPingIf($condition, $url)` — Pings after completion only if the condition is true.
- `pingOnSuccess($url)` — Pings only if the task succeeds.
- `pingOnFailure($url)` — Pings only if the task fails.

**Syntax Rules:**

- The ping methods use Laravel's HTTP client to send GET requests.
- The URL should be the unique ping URL provided by the monitoring service.
- Multiple ping methods can be chained on the same task.
- The ping methods are executed by the `ScheduleRunCommand` as part of the task lifecycle.
- The monitoring service must be configured with the expected interval and alerting rules.

**Constraints and Limitations:**

- **Pings require outbound HTTP access.** Ensure the server can reach the monitoring service.
- **Pings are fire-and-forget.** Failures to ping are silently ignored unless logging is configured.
- **The monitoring service must be configured correctly.** The expected interval and grace period must match the task's schedule.
- **Ping URLs are secrets.** Store them in `.env` and never commit them to version control.
- **Deadman switches detect absence, not correctness.** A task that runs but produces wrong results will still ping successfully.

### Annotated Code Examples

**Example 1: Healthchecks.io Integration**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Step 1: Daily backup with heartbeat pings
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->pingBefore(env('HEALTHCHECKS_BACKUP_START'))
    ->thenPing(env('HEALTHCHECKS_BACKUP_SUCCESS'))
    ->pingOnFailure(env('HEALTHCHECKS_BACKUP_FAIL'))
    ->description('Daily backup with heartbeat monitoring');

// Step 2: Hourly report with ping after completion
Schedule::command('reports:hourly')
    ->hourly()
    ->thenPing(env('HEALTHCHECKS_REPORTS'))
    ->description('Hourly report with heartbeat');

// Step 3: Every-minute health check with conditional ping
Schedule::command('health:check')
    ->everyMinute()
    ->thenPingIf(
        fn () => app()->isProduction(),
        env('HEALTHCHECKS_HEALTH')
    )
    ->description('Health check with conditional heartbeat');
```

```bash
# .env file
HEALTHCHECKS_BACKUP_START=https://hc-ping.com/uuid-1/start
HEALTHCHECKS_BACKUP_SUCCESS=https://hc-ping.com/uuid-1
HEALTHCHECKS_BACKUP_FAIL=https://hc-ping.com/uuid-1/fail
HEALTHCHECKS_REPORTS=https://hc-ping.com/uuid-2
HEALTHCHECKS_HEALTH=https://hc-ping.com/uuid-3
```

**Expected Behaviour:**

- Healthchecks.io receives a ping when the backup starts, completes successfully, or fails.
- If the backup does not ping within the expected interval, Healthchecks.io sends an alert.
- The hourly report pings after each successful run.
- The health check pings every minute in production.

**Why This Output Occurs:** The `pingBefore()`, `thenPing()`, and `pingOnFailure()` methods send GET requests to the Healthchecks.io URLs at the appropriate lifecycle points. Healthchecks.io tracks the expected interval for each check and alerts if a ping is not received within the grace period. This provides independent verification that the scheduler is running and tasks are executing.

---

**Example 2: Oh Dear Integration with Flare Error Tracking**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Oh Dear heartbeat monitoring
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->thenPing('https://ohdear.app/api/heartbeat/your-heartbeat-id')
    ->description('Daily backup with Oh Dear heartbeat');

// Flare is automatically integrated via its Laravel package.
// Any exception thrown by a scheduled task is reported to Flare.
// No additional configuration is needed in the schedule.
```

```bash
# .env file
FLARE_KEY=your-flare-key
OH_DEAR_HEARTBEAT_URL=https://ohdear.app/api/heartbeat/your-heartbeat-id
```

**Expected Behaviour:**

- Oh Dear receives a heartbeat ping after each successful backup.
- Flare captures any exceptions thrown by scheduled tasks.
- Both platforms send alerts if something goes wrong.

**Why This Output Occurs:** The `thenPing()` method sends a heartbeat to Oh Dear after the backup completes. Flare's Laravel package automatically hooks into the framework's exception handler, capturing any exception thrown by scheduled tasks and reporting it to the Flare dashboard. This combination provides both heartbeat monitoring (did the task run?) and error tracking (what went wrong?).

### Real-World Cases

- **Critical backups:** Use Healthchecks.io to alert if the nightly backup doesn't run.
- **Billing processing:** Use Oh Dear to monitor the billing task's heartbeat and Flare to capture any exceptions.
- **Data synchronization:** Use Cronitor to monitor sync frequency and alert on anomalies.
- **Health checks:** Use UptimeRobot to monitor the health check endpoint and alert on downtime.
- **Multi-region deployments:** Use a heartbeat monitor per region to ensure all schedulers are running.

---

## 4. Sub-Second Execution (Tick Scheduling)

### Definitions

**Core Definition:** Sub-second execution, or tick scheduling, is the practice of running scheduled tasks at intervals shorter than one minute — down to every second — using Laravel's `schedule:work` daemon, which evaluates tasks every second.

**Technical Definition:** Laravel's `schedule:work` command (implemented by `Illuminate\Console\Scheduling\ScheduleWorkCommand`) runs a continuous loop that evaluates the schedule every second. Unlike `schedule:run`, which is invoked once per minute by cron, `schedule:work` sleeps for one second between iterations, enabling sub-minute frequencies. The `ManagesFrequencies` trait provides `everySecond()`, `everyTwoSeconds()`, `everyFiveSeconds()`, `everyTenSeconds()`, `everyFifteenSeconds()`, `everyTwentySeconds()`, and `everyThirtySeconds()` methods. These methods generate cron expressions with a `*` in the minute field and a step value in the second field (using the six-field cron format supported by `dragonmantank/cron-expression`).

**Beginner-Friendly Explanation:** Standard cron can only run tasks once per minute. But some tasks need to run more frequently — like collecting metrics every 5 seconds or polling a queue every second. Laravel's `schedule:work` command runs in a continuous loop, checking the schedule every second. This allows tasks to run at sub-minute intervals. The trade-off is that `schedule:work` must run as a long-lived daemon process, managed by Supervisor.

### Purposes

- To run tasks at intervals shorter than one minute (every second, every 5 seconds, etc.).
- To support high-frequency workloads like metrics collection, queue polling, and real-time monitoring.
- To eliminate the one-minute granularity limitation of standard cron.
- To provide a daemon-based alternative to cron for high-frequency scheduling.
- To enable micro-interval automation for real-time applications.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Sub-second frequencies
Schedule::command('metrics:collect')->everySecond();
Schedule::command('metrics:collect')->everyTwoSeconds();
Schedule::command('metrics:collect')->everyFiveSeconds();
Schedule::command('metrics:collect')->everyTenSeconds();
Schedule::command('metrics:collect')->everyFifteenSeconds();
Schedule::command('metrics:collect')->everyTwentySeconds();
Schedule::command('metrics:collect')->everyThirtySeconds();

// Combine with other controls
Schedule::command('metrics:collect')
    ->everyFiveSeconds()
    ->withoutOverlapping(1)
    ->runInBackground()
    ->description('Collect metrics every 5 seconds');
```

```bash
# Run the scheduler as a daemon (required for sub-second execution)
php artisan schedule:work
```

**Component Breakdown:**

- `everySecond()` — Runs the task every second.
- `everyFiveSeconds()` — Runs the task every 5 seconds.
- `everyThirtySeconds()` — Runs the task every 30 seconds.
- `schedule:work` — The daemon command that evaluates the schedule every second.

**Syntax Rules:**

- Sub-second frequencies require `schedule:work` to be running. Standard cron (once per minute) cannot execute them.
- The `schedule:work` command must be managed by a process manager (Supervisor) in production.
- The `withoutOverlapping()` method should be used with sub-second frequencies to prevent task pile-up.
- The `runInBackground()` method can be used to prevent long-running sub-second tasks from blocking the scheduler.

**Constraints and Limitations:**

- **Sub-second frequencies require `schedule:work`.** They do not work with standard cron.
- **`schedule:work` must run as a daemon.** It is not invoked by cron; it runs continuously.
- **Sub-second tasks can consume significant resources.** Use them only when necessary.
- **The `withoutOverlapping()` method is essential for sub-second tasks.** Without it, tasks can pile up.
- **`schedule:work` sleeps for one second between iterations.** Tasks scheduled at 500ms intervals are not supported.

### Annotated Code Examples

**Example 1: High-Frequency Metrics Collection**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Step 1: Collect metrics every 5 seconds
Schedule::command('metrics:collect')
    ->everyFiveSeconds()
    ->withoutOverlapping(1)
    ->runInBackground()
    ->description('Collect metrics every 5 seconds');

// Step 2: Poll the queue every second
Schedule::command('queue:work --stop-when-empty --max-time=1')
    ->everySecond()
    ->withoutOverlapping(1)
    ->description('Poll queue every second');

// Step 3: Check system health every 10 seconds
Schedule::command('health:check')
    ->everyTenSeconds()
    ->withoutOverlapping(1)
    ->description('Health check every 10 seconds');
```

```bash
# Run the scheduler as a daemon
php artisan schedule:work
```

**Expected Output (schedule:work):**

```
  2025-06-15 10:00:00 Running [metrics:collect] ......... 0.5s DONE
  2025-06-15 10:00:05 Running [metrics:collect] ......... 0.4s DONE
  2025-06-15 10:00:01 Running [queue:work --stop-when-empty --max-time=1]  DONE
  2025-06-15 10:00:10 Running [health:check] ............ 0.1s DONE
```

**Why This Output Occurs:** The `schedule:work` command runs a continuous loop, evaluating the schedule every second. When a task's sub-second cron expression is due, the task is executed. The `withoutOverlapping()` method prevents tasks from piling up if they run longer than their interval. The `runInBackground()` method allows the metrics collection to run in parallel without blocking the scheduler.

---

**Example 2: Supervisor Configuration for schedule:work**

```ini
; /etc/supervisor/conf.d/laravel-scheduler.conf

[program:laravel-scheduler]
process_name=%(program_name)s
command=php /var/www/app/artisan schedule:work
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/app/storage/logs/scheduler.log
stopwaitsecs=3600
```

```bash
# Step 1: Reload Supervisor configuration
sudo supervisorctl reread
sudo supervisorctl update

# Step 2: Start the scheduler
sudo supervisorctl start laravel-scheduler
```

**Expected Output:** Supervisor starts the `schedule:work` process and automatically restarts it if it crashes. The scheduler runs continuously, evaluating the schedule every second.

**Why This Output Occurs:** The `schedule:work` command runs as a long-lived daemon process. Supervisor monitors the process and restarts it if it exits unexpectedly. The `stopwaitsecs` setting tells Supervisor to wait up to 3600 seconds for the scheduler to finish its current task before forcefully killing it, ensuring graceful shutdown.

### Real-World Cases

- **Real-time metrics collection:** Collect application metrics every 5 seconds for real-time dashboards.
- **Queue polling:** Poll the queue every second to reduce latency for time-sensitive jobs.
- **Health checks:** Check system health every 10 seconds for rapid detection of issues.
- **Real-time data synchronization:** Sync data from external sources every 5 seconds for real-time applications.
- **IoT data ingestion:** Process IoT device data every second for real-time monitoring.

---

## References

- Laravel Task Scheduling Documentation (Master) — https://laravel.com/framework/docs/master/scheduling
- Laravel Task Scheduling Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/scheduling
- Laravel Task Scheduling Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/scheduling
- Laravel `Illuminate\Console\Scheduling\Event` API — https://api.laravel.com/docs/master/Illuminate/Console/Scheduling/Event.html
- Laravel `Illuminate\Console\Scheduling\ScheduleWorkCommand` API — https://api.laravel.com/docs/master/Illuminate/Console/Scheduling/ScheduleWorkCommand.html
- Laravel `Illuminate\Console\Scheduling\ManagesFrequencies` API — https://api.laravel.com/docs/master/Illuminate/Console/Scheduling/ManagesFrequencies.html
- Laravel Task Scheduling: Task Output — https://laravel.com/docs/master/scheduling#task-output
- Laravel Task Scheduling: Task Hooks — https://laravel.com/docs/master/scheduling#task-hooks
- Laravel Task Scheduling: Ping URLs — https://laravel.com/docs/master/scheduling#ping-urls
- Laravel Task Scheduling: Sub-Minute Scheduled Tasks — https://laravel.com/docs/master/scheduling#sub-minute-scheduled-tasks
- Laravel Task Scheduling: Running the Scheduler Locally — https://laravel.com/docs/master/scheduling#running-the-scheduler-locally
- Laravel News: Streamlining Application Automation with Laravel's Task Scheduler — https://laravel-news.com/task-scheduler
- Laravel `thenWithOutput` Method (Laravel News) — https://laravel-news.com/laravel-then-with-output
- Laravel Scheduler Output Handling (Laravel Daily) — https://laraveldaily.com/post/laravel-scheduler-output
- Laravel Scheduler Lifecycle Hooks (Laravel Daily) — https://laraveldaily.com/post/laravel-scheduler-hooks
- Healthchecks.io Documentation — https://healthchecks.io/docs/
- Oh Dear Documentation — https://ohdear.app/docs
- Flare Documentation — https://flareapp.io/docs
- Sentry Laravel Documentation — https://docs.sentry.io/platforms/php/guides/laravel/
- Cronitor Documentation — https://cronitor.io/docs
- Laravel Scheduler Heartbeat Monitoring (Laravel News) — https://laravel-news.com/laravel-scheduler-heartbeat
- Laravel `schedule:work` Command (Laravel News) — https://laravel-news.com/laravel-schedule-work
- Laravel Sub-Minute Scheduling (Laravel News) — https://laravel-news.com/laravel-sub-minute-scheduling
- Laravel Scheduler Deadman Switch (Stack Overflow) — https://stackoverflow.com/questions/46604614
- Laravel Supervisor Configuration for Scheduler — https://laravel.com/docs/master/scheduling#running-the-scheduler-locally