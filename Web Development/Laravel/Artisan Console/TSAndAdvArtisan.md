# Laravel Task Scheduling & Advanced Artisan — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Task Scheduling is a fluent, code-based scheduler that allows developers to define recurring tasks — Artisan commands, queued jobs, shell commands, and closures — in a single file (`routes/console.php` in Laravel 11+ or `app/Console/Kernel.php` in Laravel 10 and below), which are then executed by a single system cron entry.

**Technical Definition:** Laravel's scheduler is implemented by `Illuminate\Console\Scheduling\Schedule`, which is resolved from the container and populated during the console kernel's bootstrap. Each scheduled task is an instance of `Illuminate\Console\Scheduling\Event` (for commands and closures) or `Illuminate\Console\Scheduling\CallbackEvent` (for closures), configured with a cron expression, timezone, constraints, and middleware. A single system cron entry (`* * * * * cd /path-to-project && php artisan schedule:run >> /dev/null 2>&1`) invokes `schedule:run` every minute, which evaluates each task's cron expression and executes those that are due. Advanced features include `withoutOverlapping()`, `onOneServer()`, `runInBackground()`, `evenInMaintenanceMode()`, and the `schedule:work` command (a daemon alternative to cron).

**Beginner-Friendly Explanation:** Instead of adding multiple cron entries to your server for each recurring task, Laravel lets you define all scheduled tasks in one place using readable PHP code. You write something like `Schedule::command('emails:send')->daily();` and Laravel handles the rest. A single cron entry runs Laravel's scheduler every minute, which checks which tasks are due and runs them. This makes scheduling tasks portable, testable, and version-controlled.

### Key Characteristics

- **Single cron entry:** Only one system cron entry is needed regardless of how many tasks are scheduled.
- **Fluent, expressive API:** Frequency options read like English (`->daily()`, `->weeklyOn(1, '8:00')`, `->everyFiveMinutes()`).
- **Cron expression support:** Custom cron expressions are supported via `->cron('* * * * *')`.
- **Constraint system:** Tasks can be constrained by environment, timezone, day of week, and custom truth tests.
- **Overlap prevention:** `withoutOverlapping()` and `onOneServer()` prevent concurrent execution.
- **Background execution:** `runInBackground()` allows tasks to run in parallel without blocking the scheduler.
- **Output handling:** Task output can be logged, emailed, or appended to files.
- **Failure handling:** Tasks can be configured to email on failure, retry, or trigger callbacks.
- **Maintenance mode integration:** Tasks can be configured to run (or not run) when the application is in maintenance mode.
- **Daemon alternative:** `schedule:work` runs the scheduler as a long-lived process, eliminating the need for cron.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the scheduler configured.
- A system cron entry (or the `schedule:work` daemon) to invoke `schedule:run` every minute.
- For `onOneServer()`: a shared cache driver (Redis, Memcached, or database) accessible by all servers.
- For `runInBackground()`: the `exec()` function must be available in PHP.

### Related Programming Areas

- **Artisan Console** — Commands are the most common type of scheduled task.
- **Queue System** — `Schedule::job()` dispatches jobs to the queue.
- **Cache System** — Locks and mutexes are used for overlap prevention.
- **Logging** — Task output can be directed to log files.
- **Mail** — Task output and failure notifications can be emailed.
- **Maintenance Mode** — Tasks can be configured to run during maintenance.

### Core Concepts / Features

1. The Scheduler (Defining background jobs and commands inside `routes/console.php`)
2. Frequency Options (Scheduling tasks hourly, daily, weekly, or using custom Cron expressions)
3. Task Outputs (Logging outputs to files, emailing results, and handling failures)
4. Maintenance & Control (Putting the app in maintenance mode (`down`) and bringing it back (`up`))

---

## 1. The Scheduler

### Definitions

**Core Definition:** The Scheduler is Laravel's code-based task scheduling system that replaces traditional server cron configuration with a fluent PHP API defined in `routes/console.php` (Laravel 11+) or `app/Console/Kernel.php` (Laravel 10 and below).

**Technical Definition:** The `Illuminate\Console\Scheduling\Schedule` class is a singleton resolved from the container and populated during the console kernel's bootstrap. Each scheduled task is represented by an `Event` or `CallbackEvent` instance, which holds the task's command, cron expression, timezone, constraints, and middleware. The `schedule:run` command (implemented by `Illuminate\Console\Scheduling\ScheduleRunCommand`) is invoked every minute by the system cron; it iterates over all registered events, checks whether each is due (by evaluating its cron expression against the current time and applying constraints), and executes the due events. The `schedule:work` command (implemented by `Illuminate\Console\Scheduling\ScheduleWorkCommand`) runs the scheduler in a continuous loop without requiring cron.

**Beginner-Friendly Explanation:** The Scheduler is where you tell Laravel "run this task every day" or "run that task every hour." You write these instructions in a single file using readable PHP code. On the server, you add one cron entry that runs Laravel's scheduler every minute. Laravel then figures out which of your tasks are due and runs them. This means you don't have to touch the server's cron configuration when you add or change tasks — you just edit your code.

### Purposes

- To define all recurring tasks in a single, version-controlled file.
- To replace multiple system cron entries with one.
- To provide a readable, fluent API for defining task frequencies.
- To enable testing of scheduled tasks without waiting for the actual schedule.
- To centralise task configuration (constraints, output, failure handling) in code.
- To support both cron-based and daemon-based scheduling.

### Syntax Rules and Structure

#### Complete General Syntax

**Laravel 11+ (`routes/console.php`):**

```php
<?php

use Illuminate\Support\Facades\Schedule;

// Schedule an Artisan command
Schedule::command('emails:send')
    ->daily();

// Schedule a closure
Schedule::call(function () {
    // Task logic
})->hourly();

// Schedule a queued job
Schedule::job(new ProcessPodcast)
    ->everyFiveMinutes();

// Schedule a shell command
Schedule::exec('node /home/forge/script.js')
    ->daily();
```

**Laravel 10 and below (`app/Console/Kernel.php`):**

```php
<?php

namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    protected function schedule(Schedule $schedule): void
    {
        $schedule->command('emails:send')->daily();
        $schedule->call(function () {
            // Task logic
        })->hourly();
        $schedule->job(new ProcessPodcast)->everyFiveMinutes();
        $schedule->exec('node /home/forge/script.js')->daily();
    }
}
```

**System Cron Entry (required once):**

```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

**Component Breakdown:**

- `Schedule::command($command)` — Schedules an Artisan command by name.
- `Schedule::call($callback)` — Schedules a closure.
- `Schedule::job($job)` — Schedules a queued job.
- `Schedule::exec($command)` — Schedules a shell command.
- `->daily()` — Sets the frequency (see Section 2 for all options).
- `schedule:run` — The command invoked by cron every minute to evaluate and run due tasks.
- `schedule:work` — A daemon alternative that runs the scheduler continuously.

```php
// Multiple tasks in routes/console.php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->daily();
Schedule::command('queue:prune-failed')->weekly();
Schedule::command('backup:run')->dailyAt('02:00');
Schedule::call(fn () => Cache::flush())->weekly();
Schedule::job(new CleanupTempFiles)->hourly();
```

**Syntax Rules:**

- In Laravel 11+, scheduled tasks are defined in `routes/console.php` using the `Schedule` facade.
- In Laravel 10 and below, tasks are defined in the `schedule()` method of `app/Console/Kernel.php`.
- Each task must have a frequency method (`->daily()`, `->hourly()`, etc.) or a custom cron expression (`->cron('* * * * *')`).
- The `Schedule` facade provides static methods (`command`, `call`, `job`, `exec`) that return `Event` instances for chaining.
- The `schedule:run` command must be invoked every minute by the system cron (or `schedule:work` must be running).

**Constraints and Limitations:**

- **The scheduler requires the system cron to be running.** Without the cron entry (or `schedule:work`), no tasks will execute.
- **The `schedule:run` command evaluates tasks based on the server's current time.** Ensure the server's timezone is correctly configured, or set the timezone explicitly per task.
- **The scheduler does not queue tasks by default.** Long-running tasks block the scheduler until they complete. Use `runInBackground()` or schedule jobs to the queue for non-blocking execution.
- **`schedule:run` runs for a maximum of 60 seconds.** If a task takes longer, it may overlap with the next minute's run. Use `withoutOverlapping()` to prevent this.
- **Closure-based tasks cannot be serialized.** They only work with the cron-based scheduler, not with `schedule:work` in some configurations.

### Annotated Code Examples

**Example 1: Defining a Comprehensive Schedule (Laravel 11+)**

```php
<?php
// File: routes/console.php

use App\Jobs\CleanupTempFiles;
use App\Jobs\ProcessPodcast;
use Illuminate\Support\Facades\Schedule;

// Step 1: Send welcome emails every day at midnight
Schedule::command('emails:welcome')
    ->daily()
    ->at('00:00');

// Step 2: Prune failed queue jobs weekly
Schedule::command('queue:prune-failed')
    ->weekly()
    ->sundays()
    ->at('03:00');

// Step 3: Run database backups daily at 2 AM
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->onOneServer()
    ->withoutOverlapping(60);

// Step 4: Flush application cache every six hours
Schedule::call(function () {
    Cache::flush();
})->everySixHours();

// Step 5: Dispatch a job every five minutes
Schedule::job(new ProcessPodcast)
    ->everyFiveMinutes()
    ->withoutOverlapping();

// Step 6: Run a shell command every hour
Schedule::exec('node /home/forge/script.js')
    ->hourly()
    ->runInBackground();
```

**Step-by-Step Setup:**

1. Add the cron entry to the server: `* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1`.
2. Ensure the `Schedule` facade is imported in `routes/console.php`.
3. Verify the schedule with `php artisan schedule:list`.
4. Run `php artisan schedule:run` manually to test.

**Expected Output (schedule:list):**

```
  0 0 * * *  php artisan emails:welcome ...................... Next Due: 1 day from now
  0 3 * * 0  php artisan queue:prune-failed .................. Next Due: 6 days from now
  0 2 * * *  php artisan backup:run ........................... Next Due: 14 hours from now
  0 */6 * * *  php artisan closure ............................ Next Due: 2 hours from now
  */5 * * * *  php artisan queue:job ProcessPodcast ........... Next Due: 3 minutes from now
  0 * * * *  exec node /home/forge/script.js ................. Next Due: 42 minutes from now
```

**Why This Output Occurs:** The `schedule:list` command reads the `Schedule` instance populated in `routes/console.php` and renders each task with its cron expression, command, and next due time. The next due time is calculated by evaluating the cron expression against the current time.

---

**Example 2: Defining a Schedule in Laravel 10 (Kernel-Based)**

```php
<?php
// File: app/Console/Kernel.php

namespace App\Console;

use App\Jobs\ProcessPodcast;
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    protected function schedule(Schedule $schedule): void
    {
        // Daily backup at 2 AM, on one server only
        $schedule->command('backup:run')
                 ->dailyAt('02:00')
                 ->onOneServer();

        // Weekly cleanup on Sundays at 3 AM
        $schedule->command('cleanup:temp')
                 ->weeklyOn(0, '03:00');

        // Hourly cache flush
        $schedule->call(function () {
            Cache::flush();
        })->hourly();

        // Every-five-minute job dispatch
        $schedule->job(new ProcessPodcast)
                 ->everyFiveMinutes();
    }

    protected function commands(): void
    {
        $this->load(__DIR__.'/Commands');
        require base_path('routes/console.php');
    }
}
```

**Expected Output:** Identical to Laravel 11+ but defined in the kernel's `schedule()` method.

**Why This Output Occurs:** In Laravel 10 and below, the `schedule()` method receives the `Schedule` instance and defines tasks on it. The kernel's `commands()` method loads the commands and the `routes/console.php` file (which can also define scheduled tasks).

### Real-World Cases

- **Daily backups:** `Schedule::command('backup:run')->dailyAt('02:00')->onOneServer()` ensures backups run once per day on one server.
- **Queue maintenance:** `Schedule::command('queue:prune-failed')->weekly()` cleans up failed jobs weekly.
- **Cache warming:** `Schedule::call(fn () => Cache::flush())->everySixHours()` clears the cache every six hours.
- **Report generation:** `Schedule::command('reports:daily')->dailyAt('06:00')->emailOutputTo('admin@example.com')` generates and emails a daily report.
- **Data synchronisation:** `Schedule::job(new SyncExternalData)->everyFifteenMinutes()` syncs data from an external API every 15 minutes.

---

## 2. Frequency Options

### Definitions

**Core Definition:** Frequency options are the fluent methods and custom cron expressions that determine when a scheduled task runs.

**Technical Definition:** The `Illuminate\Console\Scheduling\ManagesFrequencies` trait, used by the `Event` class, provides methods such as `everyMinute()`, `everyFiveMinutes()`, `hourly()`, `daily()`, `weekly()`, `monthly()`, `quarterly()`, `yearly()`, and their variants (`dailyAt()`, `weeklyOn()`, `monthlyOn()`). Each method sets the event's cron expression, which is later evaluated by `CronExpression::isDue()` from the `dragonmantank/cron-expression` package. The `cron()` method allows setting a custom cron expression directly. Timezone can be set with `timezone()`.

**Beginner-Friendly Explanation:** Laravel provides a huge list of readable methods for setting when a task runs. Instead of writing `0 2 * * *` for "every day at 2 AM," you write `->dailyAt('02:00')`. If none of the built-in methods fit, you can write the raw cron expression with `->cron('...')`.

### Purposes

- To provide a readable, English-like API for defining task frequencies.
- To eliminate the need for memorising cron syntax for common schedules.
- To support custom cron expressions for unusual schedules.
- To allow per-task timezone configuration.
- To enable scheduling on specific days, dates, and times.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Sub-minute frequencies
->everySecond()
->everyTwoSeconds()
->everyFiveSeconds()
->everyTenSeconds()
->everyFifteenSeconds()
->everyTwentySeconds()
->everyThirtySeconds()

// Minute frequencies
->everyMinute()
->everyTwoMinutes()
->everyThreeMinutes()
->everyFourMinutes()
->everyFiveMinutes()
->everyTenMinutes()
->everyFifteenMinutes()
->everyThirtyMinutes()

// Hour frequencies
->hourly()
->hourlyAt(17)  // At minute 17 of every hour
->everyTwoHours()
->everyThreeHours()
->everyFourHours()
->everySixHours()

// Day frequencies
->daily()
->dailyAt('13:00')
->twiceDaily(1, 13)
->twiceDailyAt(1, 13, 15)  // At minute 15
->days([1, 3, 5])           // On specific days of the week

// Week frequencies
->weekly()
->weeklyOn(1, '8:00')  // Monday at 8 AM
->weekdays()           // Monday through Friday

// Month frequencies
->monthly()
->monthlyOn(4, '15:00')  // 4th of the month at 3 PM
->twiceMonthly(1, 16, '13:00')
->lastDayOfMonth('15:00')

// Quarter frequencies
->quarterly()
->quarterlyOn(4, '14:00')

// Year frequencies
->yearly()
->yearlyOn(6, 1, '17:00')  // June 1st at 5 PM

// Custom cron expression
->cron('* * * * *')

// Timezone
->timezone('America/New_York')

// Day constraints
->sundays()
->mondays()
->tuesdays()
->wednesdays()
->thursdays()
->fridays()
->saturdays()
->days([1, 3, 5])
```

**Component Breakdown:**

- `->everyMinute()` etc. — Set the task to run at the specified interval.
- `->hourlyAt($minute)` — Runs at a specific minute of every hour.
- `->dailyAt($time)` — Runs once per day at the specified time.
- `->twiceDaily($first, $second)` — Runs twice per day at the specified hours.
- `->weeklyOn($dayOfWeek, $time)` — Runs weekly on the specified day of the week (0 = Sunday, 6 = Saturday).
- `->monthlyOn($dayOfMonth, $time)` — Runs monthly on the specified day of the month.
- `->yearlyOn($month, $day, $time)` — Runs yearly on the specified month, day, and time.
- `->cron($expression)` — Sets a custom cron expression.
- `->timezone($tz)` — Sets the timezone for the task.
- `->days($days)` — Constrains the task to specific days of the week.

**Syntax Rules:**

- Frequency methods can be chained with day constraints (e.g., `->weekly()->mondays()`).
- The `->at()` method can be used after frequency methods to specify a time (e.g., `->daily()->at('13:00')`).
- Custom cron expressions use the standard five-field format: `minute hour day-of-month month day-of-week`.
- Timezone defaults to the application's timezone (`config/app.php` → `timezone`), which defaults to UTC.
- The `->days()` method accepts an array of day numbers (0 = Sunday through 6 = Saturday).

**Constraints and Limitations:**

- **Sub-minute frequencies (everySecond, etc.) require `schedule:work` or a cron entry that runs more frequently than once per minute.** Standard cron runs at most once per minute.
- **Some frequency methods are aliases.** `->daily()` is equivalent to `->cron('0 0 * * *')`.
- **The `->at()` method must be called after a frequency method.** `->at('13:00')` alone does not set a frequency.
- **Timezone changes affect only the scheduled time evaluation.** The task itself runs in the server's timezone for any internal date operations.

### Annotated Code Examples

**Example 1: Common Frequency Patterns**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Every minute
Schedule::command('metrics:collect')->everyMinute();

// Every five minutes, only on weekdays
Schedule::command('reports:generate')
    ->everyFiveMinutes()
    ->weekdays();

// Every hour at minute 17
Schedule::command('sync:data')->hourlyAt(17);

// Every day at 2:30 AM
Schedule::command('backup:run')->dailyAt('02:30');

// Twice daily at 1 AM and 1 PM
Schedule::command('emails:digest')->twiceDaily(1, 13);

// Weekly on Monday at 8 AM
Schedule::command('reports:weekly')->weeklyOn(1, '08:00');

// Monthly on the 1st at midnight
Schedule::command('billing:invoice')->monthlyOn(1, '00:00');

// Quarterly on January 1, April 1, July 1, October 1 at 3 AM
Schedule::command('reports:quarterly')->quarterlyOn(1, '03:00');

// Yearly on June 1 at 5 PM
Schedule::command('reports:annual')->yearlyOn(6, 1, '17:00');

// Custom cron expression: every 15 minutes on weekdays between 9 AM and 5 PM
Schedule::command('support:check')->cron('*/15 9-17 * * 1-5');
```

**Expected Output (schedule:list):**

```
  * * * * *    php artisan metrics:collect .................... Next Due: 1 minute from now
  */5 * * * 1-5  php artisan reports:generate ................. Next Due: 3 minutes from now
  17 * * * *   php artisan sync:data .......................... Next Due: 42 minutes from now
  30 2 * * *   php artisan backup:run ......................... Next Due: 14 hours from now
  0 1,13 * * *  php artisan emails:digest ..................... Next Due: 8 hours from now
  0 8 * * 1    php artisan reports:weekly ..................... Next Due: 5 days from now
  0 0 1 * *    php artisan billing:invoice .................... Next Due: 20 days from now
  0 3 1 1,4,7,10 *  php artisan reports:quarterly ............ Next Due: 45 days from now
  0 17 1 6 *   php artisan reports:annual ..................... Next Due: 320 days from now
  */15 9-17 * * 1-5  php artisan support:check ............... Next Due: 12 minutes from now
```

**Why This Output Occurs:** Each frequency method sets a specific cron expression on the `Event` instance. The `schedule:list` command renders the cron expression and calculates the next due time by evaluating the expression against the current time. The `->weekdays()` constraint adds `1-5` to the day-of-week field.

---

**Example 2: Timezone-Specific Scheduling**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Run daily at 8 AM New York time, regardless of server timezone
Schedule::command('reports:morning')
    ->dailyAt('08:00')
    ->timezone('America/New_York');

// Run weekly on Monday at 9 AM London time
Schedule::command('reports:weekly')
    ->weeklyOn(1, '09:00')
    ->timezone('Europe/London');

// Run hourly in the application's default timezone (UTC)
Schedule::command('sync:data')
    ->hourly();
```

**Expected Output (schedule:list):**

```
  0 8 * * *  php artisan reports:morning .................... Next Due: (America/New_York) 6 hours from now
  0 9 * * 1  php artisan reports:weekly ..................... Next Due: (Europe/London) 5 days from now
  0 * * * *  php artisan sync:data .......................... Next Due: 42 minutes from now
```

**Why This Output Occurs:** The `->timezone()` method sets the timezone on the event. When the scheduler evaluates whether the task is due, it converts the current time to the specified timezone before comparing against the cron expression. This ensures that tasks run at the correct local time regardless of the server's timezone.

### Real-World Cases

- **Daily backups:** `->dailyAt('02:00')->timezone('America/New_York')` ensures backups run at 2 AM Eastern.
- **Weekly reports:** `->weeklyOn(1, '09:00')` runs a report every Monday at 9 AM.
- **Monthly billing:** `->monthlyOn(1, '00:00')` runs billing on the first of each month.
- **Sub-minute monitoring:** `->everyFiveSeconds()` (with `schedule:work`) collects metrics every five seconds.
- **Business-hours-only tasks:** `->cron('*/15 9-17 * * 1-5')` runs every 15 minutes during business hours on weekdays.

---

## 3. Task Outputs

### Definitions

**Core Definition:** Task outputs are the mechanisms by which scheduled tasks report their results — including logging output to files, emailing output to recipients, and handling task failures.

**Technical Definition:** The `Illuminate\Console\Scheduling\Event` class provides methods for directing task output: `sendOutputTo($file)`, `appendOutputTo($file)`, `emailOutputTo($address)`, `emailOutputOnFailure($address)`, `emailWrittenOutputTo($address)`, and `onFailure($callback)`. These methods configure the event's output handling, which is executed by the `ScheduleRunCommand` after the task completes. Output can also be directed to the application log via `emailOutputTo` with a log channel, or handled via `onSuccess`/`onFailure` callbacks. The `->runInBackground()` method changes output handling because background tasks write directly to their configured output destination.

**Beginner-Friendly Explanation:** When your scheduled task runs, it produces output (like "Backup completed" or an error message). By default, this output is discarded. But you can tell Laravel to save it to a file, email it to you, or both. You can also configure what happens if the task fails — for example, emailing an error notification to the administrator.

### Purposes

- To capture task output for auditing and debugging.
- To notify administrators of task results via email.
- To alert on task failures immediately.
- To append task output to log files for historical analysis.
- To execute custom callbacks on success or failure.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Output handling methods
->sendOutputTo(string $path, bool $append = false)
->appendOutputTo(string $path)
->emailOutputTo(string|array $addresses, bool $onlyIfOutputExists = false)
->emailOutputOnFailure(string|array $addresses)
->emailWrittenOutputTo(string|array $addresses)
->onSuccess(callable $callback)
->onFailure(callable $callback)
->then(callable $callback)         // After success
->finally(callable $callback)      // Always, regardless of outcome
->before(callable $callback)       // Before execution
->after(callable $callback)        // After execution (deprecated; use then/finally)
```

**Component Breakdown:**

- `sendOutputTo($path)` — Writes the task's output to the specified file (overwriting).
- `appendOutputTo($path)` — Appends the task's output to the specified file.
- `emailOutputTo($addresses)` — Emails the task's output to the specified addresses.
- `emailOutputOnFailure($addresses)` — Emails the output only if the task fails.
- `onSuccess($callback)` — Runs the callback if the task succeeds.
- `onFailure($callback)` — Runs the callback if the task fails.
- `then($callback)` — Runs the callback after the task completes successfully.
- `finally($callback)` — Runs the callback regardless of the task's outcome.

```php
// Example: Send output to a file and email on failure
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->sendOutputTo(storage_path('logs/backup.log'))
    ->emailOutputOnFailure('admin@example.com');

// Example: Append output and run callbacks
Schedule::command('reports:generate')
    ->daily()
    ->appendOutputTo(storage_path('logs/reports.log'))
    ->onSuccess(fn () => Log::info('Report generated successfully.'))
    ->onFailure(fn () => Log::error('Report generation failed.'));
```

**Syntax Rules:**

- Output methods can be chained with frequency and constraint methods.
- `sendOutputTo()` overwrites the file; `appendOutputTo()` preserves existing content.
- `emailOutputTo()` requires a configured mail system and a valid recipient address.
- Callbacks receive no arguments; use closures with `use` to capture context if needed.
- `onFailure()` receives the exception instance as an argument when the task fails via an exception.

**Constraints and Limitations:**

- **`emailOutputTo()` and `emailOutputOnFailure()` require the mail system to be configured.** If mail is not configured, the email will fail silently or throw an error.
- **Output from `runInBackground()` tasks may not be captured** because the task runs in a separate process.
- **Callbacks run synchronously** and can block the scheduler. Keep them short.
- **`then()` and `finally()` require Laravel 8+.** In older versions, use `after()`.

### Annotated Code Examples

**Example 1: Comprehensive Output Handling**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Schedule;

// Step 1: Daily backup with output to file and email on failure
Schedule::command('backup:run')
    ->dailyAt('02:00')
    ->sendOutputTo(storage_path('logs/backup.log'))
    ->emailOutputOnFailure('admin@example.com')
    ->onSuccess(function () {
        Log::info('Backup completed successfully.');
    })
    ->onFailure(function () {
        Log::error('Backup failed.');
    });

// Step 2: Hourly report with appended output
Schedule::command('reports:hourly')
    ->hourly()
    ->appendOutputTo(storage_path('logs/hourly-reports.log'));

// Step 3: Weekly cleanup with email notification
Schedule::command('cleanup:temp')
    ->weekly()
    ->sundays()
    ->at('03:00')
    ->emailOutputTo('ops@example.com');
```

**Expected Output (backup.log):**

```
Starting backup...
Backing up database...
Backing up files...
Backup completed in 45.2 seconds.
```

**Expected Output (email on failure):**

```
Subject: Scheduled Task Failed: backup:run

Command: php artisan backup:run
Exit Code: 1
Output:
  Starting backup...
  Error: Could not connect to database.
```

**Why This Output Occurs:** The `sendOutputTo()` method directs the task's stdout to the specified file. The `emailOutputOnFailure()` method configures the scheduler to email the output only if the task returns a non-zero exit code or throws an exception. The `onSuccess()` and `onFailure()` callbacks run after the task completes, logging the outcome.

---

**Example 2: Using Callbacks for Custom Handling**

```php
<?php
// File: routes/console.php

use App\Models\ScheduledTaskLog;
use Illuminate\Support\Facades\Schedule;

Schedule::command('sync:external-data')
    ->everyFifteenMinutes()
    ->before(function () {
        // Runs before the task starts
        logger('Starting external data sync...');
    })
    ->then(function () {
        // Runs after successful completion
        ScheduledTaskLog::create([
            'task'       => 'sync:external-data',
            'status'     => 'success',
            'completed_at' => now(),
        ]);
    })
    ->finally(function () {
        // Runs regardless of outcome
        cache()->forget('sync:in-progress');
    })
    ->onFailure(function () {
        // Runs if the task fails
        ScheduledTaskLog::create([
            'task'       => 'sync:external-data',
            'status'     => 'failed',
            'completed_at' => now(),
        ]);
    });
```

**Expected Output:** The callbacks execute in order, creating log entries and clearing the cache flag. The `finally` callback runs whether the task succeeds or fails.

**Why This Output Occurs:** The `before()`, `then()`, `finally()`, and `onFailure()` callbacks are registered as event listeners on the scheduled task. The `ScheduleRunCommand` invokes them in the appropriate order after the task completes. This pattern is useful for recording task history in a database or clearing temporary state.

### Real-World Cases

- **Backup monitoring:** `->emailOutputOnFailure('admin@example.com')` alerts administrators when a backup fails.
- **Audit logging:** `->appendOutputTo(storage_path('logs/tasks.log'))` maintains a historical record of all task executions.
- **Task history:** `->then()` and `->onFailure()` callbacks record task outcomes in a database table.
- **Cleanup:** `->finally()` callbacks clear temporary locks or cache entries regardless of the task's outcome.
- **Success notifications:** `->emailOutputTo('reports@example.com')` emails the output of a report generation task to stakeholders.

---

## 4. Maintenance & Control

### Definitions

**Core Definition:** Maintenance and control commands are Artisan commands that put the application into maintenance mode (displaying a maintenance page to all visitors) and bring it back online, with support for bypassing maintenance mode for specific IPs, secrets, and scheduled tasks.

**Technical Definition:** The `down` command (implemented by `Illuminate\Foundation\Console\DownCommand`) creates a `storage/framework/down` file and optionally writes a pre-rendered view to `storage/framework/maintenance.php`. The `up` command (implemented by `Illuminate\Foundation\Console\UpCommand`) removes these files. Middleware (`Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance`) checks for the `down` file and returns a 503 response with the pre-rendered view. The `--secret` option allows access via a query string or cookie, `--redirect` redirects all requests to a specific URL, and `--render` pre-renders the maintenance view for performance.

**Beginner-Friendly Explanation:** When you need to deploy new code, run a database migration, or perform server maintenance, you can put your application into "maintenance mode." Visitors see a friendly "We'll be right back" page instead of errors. When you're done, you bring the application back online with the `up` command. You can also allow specific people (like your team) to access the application during maintenance using a secret URL.

### Purposes

- To display a maintenance page to visitors during deployments or maintenance.
- To prevent errors caused by incomplete deployments or migrations.
- To allow administrators to bypass maintenance mode for testing.
- To redirect visitors to an alternative URL during maintenance.
- To pre-render the maintenance page for performance.
- To bring the application back online after maintenance is complete.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Put the application into maintenance mode
php artisan down [--redirect=url] [--retry=seconds] [--refresh=seconds] [--secret=token] [--render=view] [--status=code]

# Bring the application back online
php artisan up

# Check if the application is in maintenance mode
php artisan down --status=503
```

**Component Breakdown:**

- `down` — Creates the maintenance mode files and enables the maintenance middleware.
- `--redirect=<url>` — Redirects all requests to the specified URL during maintenance.
- `--retry=<seconds>` — Sets the `Retry-After` HTTP header value.
- `--refresh=<seconds>` — Sets the `Refresh` HTTP header value.
- `--secret=<token>` — Allows access to the application via `?secret=<token>` or a cookie.
- `--render=<view>` — Pre-renders the specified view for the maintenance page.
- `--status=<code>` — Sets the HTTP status code (default 503).
- `up` — Removes the maintenance mode files and disables the maintenance middleware.

```bash
# Basic maintenance mode
php artisan down

# Maintenance mode with a secret bypass
php artisan down --secret="my-secret-token"

# Maintenance mode with a redirect
php artisan down --redirect=https://status.example.com

# Maintenance mode with a custom retry header
php artisan down --retry=60

# Pre-render the maintenance page for performance
php artisan down --render="errors::503"

# Bring the application back online
php artisan up
```

**Syntax Rules:**

- The `down` command creates `storage/framework/down` and optionally `storage/framework/maintenance.php`.
- The `--secret` option allows access via `https://example.com/my-secret-token`, which sets a cookie granting continued access.
- The `--render` option pre-renders the view and stores it as a static HTML file for fast serving.
- The `up` command removes the maintenance files and restores normal operation.
- Scheduled tasks are **not** affected by maintenance mode by default, but they can be configured to run with `->evenInMaintenanceMode()`.

**Constraints and Limitations:**

- **Maintenance mode affects all requests** unless a secret is configured or the request comes from a whitelisted IP.
- **The `down` command does not stop queue workers.** Workers continue processing jobs unless explicitly stopped.
- **The `--secret` option requires a session cookie.** Users who access the secret URL get a cookie that allows continued access.
- **Pre-rendered maintenance pages are static.** They do not reflect dynamic data or per-user customisation.
- **Maintenance mode does not prevent database migrations from running.** Commands like `migrate` still work in maintenance mode.

### Annotated Code Examples

**Example 1: Standard Deployment with Maintenance Mode**

```bash
# Step 1: Put the application into maintenance mode with a secret bypass
php artisan down --secret="deploy-secret-2025" --retry=60

# Step 2: Pull the latest code
git pull origin main

# Step 3: Install dependencies
composer install --no-dev --optimize-autoloader

# Step 4: Run database migrations
php artisan migrate --force

# Step 5: Clear and rebuild caches
php artisan optimize:clear
php artisan optimize

# Step 6: Bring the application back online
php artisan up
```

**Expected Output:**

```
Application is now in maintenance mode.
...
Application is now live.
```

**Why This Output Occurs:** The `down` command creates the `storage/framework/down` file, which the `PreventRequestsDuringMaintenance` middleware checks on every request. With `--secret`, accessing `https://example.com/deploy-secret-2025` sets a cookie that bypasses maintenance mode for that browser. The `up` command removes the file, restoring normal operation.

---

**Example 2: Custom Maintenance Page with Redirect**

```bash
# Step 1: Create a custom maintenance view
# File: resources/views/errors/503.blade.php
```

```blade
{{-- File: resources/views/errors/503.blade.php --}}
<!DOCTYPE html>
<html>
<head>
    <title>Under Maintenance</title>
</head>
<body>
    <h1>We'll Be Right Back</h1>
    <p>Our application is currently undergoing scheduled maintenance.</p>
    <p>Please check back soon. Expected completion: {{ now()->addHour()->format('g:i A') }}</p>
</body>
</html>
```

```bash
# Step 2: Put the application into maintenance mode with the custom view
php artisan down --render="errors::503" --retry=3600

# Step 3: Verify the maintenance page
curl -I https://example.com
# HTTP/1.1 503 Service Unavailable
# Retry-After: 3600

# Step 4: Bring the application back online
php artisan up
```

**Expected Output:**

```
Application is now in maintenance mode.
Application is now live.
```

**Why This Output Occurs:** The `--render="errors::503"` option pre-renders the `errors/503` Blade view and stores it as a static file. The maintenance middleware serves this pre-rendered HTML directly, bypassing the full framework bootstrap for faster response times. The `Retry-After: 3600` header tells clients to retry after one hour.

### Real-World Cases

- **Deployment windows:** Put the application into maintenance mode during deployments to prevent users from seeing errors or incomplete states.
- **Database migrations:** Enable maintenance mode before running destructive migrations, then bring the application back online.
- **Emergency maintenance:** Use `php artisan down --redirect=https://status.example.com` to redirect users to a status page during unexpected outages.
- **Team access during maintenance:** Use `--secret` to allow developers and stakeholders to test the application while it's in maintenance mode.
- **Scheduled maintenance windows:** Schedule `php artisan down` and `php artisan up` commands for planned maintenance windows.

---

## References

- Laravel Task Scheduling Documentation (Master) — https://laravel.com/docs/master/scheduling 
- Laravel Task Scheduling Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/scheduling 
- Laravel Artisan Console Documentation — https://laravel.com/docs/master/artisan 
- Laravel Maintenance Mode Documentation — https://laravel.com/docs/master/configuration#maintenance-mode 
- Laravel `Illuminate\Console\Scheduling\Schedule` API — https://api.laravel.com/docs/11.x/Illuminate/Console/Scheduling/Schedule.html 
- Laravel `Illuminate\Console\Scheduling\Event` API — https://api.laravel.com/docs/11.x/Illuminate/Console/Scheduling/Event.html 
- Laravel `Illuminate\Console\Scheduling\ManagesFrequencies` API — https://api.laravel.com/docs/11.x/Illuminate/Console/Scheduling/ManagesFrequencies.html 
- Laravel `schedule:work` Command (Laravel News) — https://laravel-news.com/laravel-schedule-work 
- Laravel `Schedule::call` and `Schedule::job` (Laravel Daily) — https://laraveldaily.com/post/laravel-scheduler-call-job-exec 
- Laravel Maintenance Mode (Laravel News) — https://laravel-news.com/laravel-maintenance-mode 
- Laravel `php artisan down` with Secret (Laravel Daily) — https://laraveldaily.com/post/laravel-maintenance-mode-secret 
- Laravel Scheduler Output Handling (Laravel News) — https://laravel-news.com/laravel-scheduler-output 
- Laravel `onSuccess` and `onFailure` Callbacks (Laravel News) — https://laravel-news.com/laravel-8-47-0 
- Laravel `then` and `finally` Methods (Laravel News) — https://laravel-news.com/laravel-8-38-0 
- Cron Expression Syntax (Cronitor) — https://crontab.guru/ 
- Dragonmantank Cron Expression (GitHub) — https://github.com/dragonmantank/cron-expression 
- Laravel Scheduler Timezones (Stack Overflow) — https://stackoverflow.com/questions/41766580/laravel-scheduler-timezone