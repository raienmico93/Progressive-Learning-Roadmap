# Laravel Scheduler Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Scheduler is a fluent, code-based task scheduling system that allows developers to define recurring tasks — Artisan commands, queued jobs, closures, and shell commands — within the Laravel application itself, replacing the need for multiple system-level cron entries with a single cron directive.

**Technical Definition:** Laravel's scheduler is implemented by the `Illuminate\Console\Scheduling\Schedule` class, which is resolved from the service container and populated during the console kernel's bootstrap. Each scheduled task is represented by an `Illuminate\Console\Scheduling\Event` (for commands and shell commands) or `CallbackEvent` (for closures) instance, configured with a cron expression via the `Illuminate\Console\Scheduling\ManagesFrequencies` trait. In Laravel 11+, the console kernel (`app/Console/Kernel.php`) was removed; scheduled tasks are defined in `routes/console.php` using the `Schedule` facade, or in `bootstrap/app.php` using the `withSchedule()` method. A single system cron entry invoking `php artisan schedule:run` every minute drives task evaluation and execution.

**Beginner-Friendly Explanation:** Instead of adding dozens of separate cron jobs to your server, Laravel lets you define all your scheduled tasks in one place using readable PHP code. You write something like `Schedule::command('emails:send')->daily()` and Laravel handles the rest. A single cron entry runs Laravel's scheduler every minute, which checks which tasks are due and runs them. This makes scheduling portable, testable, and version-controlled.

### Key Characteristics

- **Single cron entry:** Only one system cron entry is needed regardless of how many tasks are scheduled.
- **Fluent, expressive API:** Frequency options read like English (`->daily()`, `->weeklyOn(1, '8:00')`, `->everyFiveMinutes()`).
- **Cron expression support:** Custom cron expressions are supported via `->cron('* * * * *')`.
- **Multiple target types:** Tasks can be Artisan commands, queued jobs, closures, invokable objects, or shell commands.
- **Timezone support:** Tasks can be scheduled in a specific timezone via `->timezone('America/New_York')`.
- **Constraint system:** Tasks can be constrained by environment, time, day, and custom truth tests.
- **Overlap prevention:** `withoutOverlapping()` and `onOneServer()` prevent concurrent execution.
- **Testable:** `php artisan schedule:list` and `php artisan schedule:test` allow inspection and verification.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the console kernel configured.
- A system cron entry (or the `schedule:work` daemon) to invoke `schedule:run` every minute.
- For `onOneServer()`: a shared cache driver (Redis, Memcached, or database) accessible by all servers.
- For `runInBackground()`: the `exec()` function must be available in PHP.

### Related Programming Areas

- **Artisan Console** — Commands are the most common type of scheduled task.
- **Queue System** — `Schedule::job()` dispatches jobs to the queue.
- **Cache System** — Locks and mutexes are used for overlap prevention.
- **Timezone Handling** — Carbon and PHP's DateTimeZone manage localized scheduling.
- **Maintenance Mode** — Tasks can be configured to run during maintenance.

### Core Concepts / Features

1. The Modern Architecture (Defining tasks within `routes/console.php` in Laravel 11+ vs. the legacy `Console/Kernel.php` engine)
2. Frequency Expressions (Implementing human-readable intervals and mapping native custom Cron expressions)
3. Target Orchestration (Scheduling native Artisan commands, background Job classes, raw Closures, or shell script executables)
4. Timezone Synchronization (Managing localized cron execution targets and safeguarding against daylight savings time shifts)

---

## 1. The Modern Architecture

### Definitions

**Core Definition:** The modern scheduler architecture in Laravel 11+ replaces the legacy `app/Console/Kernel.php` file with task definitions in `routes/console.php` (using the `Schedule` facade) or `bootstrap/app.php` (using the `withSchedule()` method).

**Technical Definition:** In Laravel 10 and below, the `App\Console\Kernel` class contained a `schedule(Schedule $schedule)` method where tasks were registered. In Laravel 11+, this file was removed; the console kernel is now configured through `bootstrap/app.php` using the `Application::configure()` fluent builder, and scheduled tasks are defined in `routes/console.php` via the `Schedule` facade (e.g., `Schedule::command('emails:send')->daily()`). Alternatively, tasks can be defined in `bootstrap/app.php` using `->withSchedule(function (Schedule $schedule) { ... })`.

**Beginner-Friendly Explanation:** Before Laravel 11, you defined scheduled tasks in a file called `Kernel.php` inside the `app/Console` folder. Starting with Laravel 11, that file is gone. Instead, you write your scheduled tasks in `routes/console.php` — a file that's already set up for you. If you prefer, you can also put them in `bootstrap/app.php`. The syntax is the same; only the location changed.

### Purposes

- To provide a central, version-controlled location for all scheduled task definitions.
- To simplify the Laravel application structure by removing the console kernel boilerplate.
- To unify console routing and scheduling in a single file (`routes/console.php`).
- To enable flexible task definition via either the `Schedule` facade or the `withSchedule()` method.
- To maintain backward compatibility with the legacy kernel approach for existing applications.

### Syntax Rules and Structure

#### Complete General Syntax (Laravel 11+)

```php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->daily();
Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
Schedule::job(new ProcessDailyMetrics)->dailyAt('01:00');
Schedule::exec('node /home/forge/script.js')->daily();
```

```php
// File: bootstrap/app.php (alternative location)

use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->command('emails:send')->daily();
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

**Legacy (Laravel 10 and below):**

```php
// File: app/Console/Kernel.php

namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    protected function schedule(Schedule $schedule): void
    {
        $schedule->command('emails:send')->daily();
    }
}
```

**Component Breakdown:**

- `Schedule::command($command)` — Schedules an Artisan command by name or class.
- `Schedule::call($callback)` — Schedules a closure or invokable object.
- `Schedule::job($job)` — Schedules a queued job.
- `Schedule::exec($command)` — Schedules a shell command.
- `->withSchedule(closure)` — Alternative registration in `bootstrap/app.php`.

**Key Frequency Options Reference**
Laravel offers highly readable methods to control exactly when your tasks run. Here is how they translate to standard cron expressions:

| Frequency Method | Description | Equivalent Cron |
|---|---|---|
| ->hourly() | Runs at the top of every hour | 0 * * * * |
| ->daily() | Runs every day at midnight | 0 0 * * * |
| ->dailyAt('13:00') | Runs every day at 1:00 PM | 0 13 * * * |
| ->twiceDaily(1, 13) | Runs twice a day (e.g., 1 AM and 1 PM) | 0 1,13 * * * |
| ->weekly() | Runs every Sunday at midnight | 0 0 * * 0 |
| ->monthly() | Runs on the 1st of every month | 0 0 1 * * |


**Syntax Rules:**

- In Laravel 11+, scheduled tasks are defined in `routes/console.php` using the `Schedule` facade.
- In Laravel 10 and below, tasks are defined in `app/Console/Kernel.php`'s `schedule()` method.
- The `Schedule` facade provides static methods (`command`, `call`, `job`, `exec`) that return `Event` instances for chaining.
- The `schedule:run` command must be invoked every minute by the system cron (or `schedule:work` must be running).

**Constraints and Limitations:**

- **The scheduler requires the system cron to be running.** Without the cron entry (or `schedule:work`), no tasks will execute.
- **Closure-based tasks cannot be serialized.** They only work with the cron-based scheduler.
- **The legacy kernel approach is still supported in Laravel 11+** but is not the default.

### Annotated Code Examples

**Example 1: Modern Schedule Definition (Laravel 11+)**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;
use Illuminate\Support\Facades\DB;

// =========================================================================
// Step 1: Schedule an Artisan Command
// Best for: Pre-written application logic wrapped in a reusable command.
// =========================================================================
Schedule::command('emails:send')
    ->daily()
    ->at('08:00');

// =========================================================================
// Step 2: Schedule a Closure (Anonymous Function)
// Best for: Quick, one-off database operations or simple maintenance tasks.
// =========================================================================
Schedule::call(function () {
    DB::table('sessions')
        ->where('last_activity', '<', now()->subHours(24))
        ->delete();
})->hourly()->description('Clean expired sessions');

// =========================================================================
// Step 3: Schedule a Queued Job
// Best for: Heavy processing tasks that should run in the background.
// =========================================================================
Schedule::job(new \App\Jobs\ProcessDailyMetrics)
    ->dailyAt('01:00');

// =========================================================================
// Step 4: Schedule a Shell Command
// Best for: Interacting with the operating system or running server tools.
// =========================================================================
Schedule::exec('pg_dump mydb > /backups/db.sql')
    ->dailyAt('02:00');
```

**Expected Output (schedule:list):**

```
  0 8 * * *    php artisan emails:send .................... Next Due: 8 hours from now
  0 * * * *    php artisan closure ........................ Next Due: 42 minutes from now
  0 1 * * *    php artisan queue:job ProcessDailyMetrics .. Next Due: 1 hour from now
  0 2 * * *    exec pg_dump mydb > /backups/db.sql ........ Next Due: 2 hours from now
```

**Why This Output Occurs:** The `schedule:list` command reads the `Schedule` instance populated in `routes/console.php` and renders each task with its cron expression, command, and next due time. The next due time is calculated by evaluating the cron expression against the current time.

---

**Example 2: Using withSchedule in bootstrap/app.php**

```php
<?php
<?php
// File: bootstrap/app.php

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php', // Registers your console route file
    )
    ->withSchedule(function (Schedule \$schedule) {
        
        // =========================================================================
        // Task 1: Generate Daily Reports
        // Triggers your custom Artisan report generator command every morning.
        // =========================================================================
        \$schedule->command('reports:generate')->dailyAt('06:00');

        // =========================================================================
        // Task 2: Prune Laravel Telescope Data
        // Maintenance command that keeps your database light by removing records older than 48 hours.
        // =========================================================================
        \$schedule->command('telescope:prune --hours=48')->daily();

        // =========================================================================
        // Task 3: Invoke an Invocable Class
        // Executes a dedicated class containing an `__invoke()` method. Clean separation of concerns!
        // =========================================================================
        \$schedule->call(new \App\Console\DeleteRecentUsers)->daily();
            
    })
    ->create();

```

**Expected Output:** The scheduled tasks are registered when the application boots, without defining them in `routes/console.php`.

**Why This Output Occurs:** The `withSchedule()` method accepts a closure that receives the `Schedule` instance, allowing tasks to be registered during the application's bootstrap phase. This is useful for applications that prefer to keep scheduling logic in `bootstrap/app.php` rather than `routes/console.php`.

### Real-World Cases

- **New Laravel 11+ projects:** All scheduled tasks are defined in `routes/console.php`, often alongside closure-based console commands.
- **Existing applications upgrading to Laravel 11:** The `Kernel.php` file can be retained or removed; if retained, it continues to work, but new tasks should be added to `routes/console.php`.
- **Package development:** Packages can register scheduled tasks via their service providers using the `Schedule` facade.
- **Multi-environment deployments:** The same schedule definitions work across all environments; environment-specific tasks are controlled via `->environments()`.

---

## 2. Frequency Expressions

### Definitions

**Core Definition:** Frequency expressions are the human-readable methods and custom cron expressions that determine when a scheduled task runs.

**Technical Definition:** The `Illuminate\Console\Scheduling\ManagesFrequencies` trait, used by the `Event` class, provides methods such as `everyMinute()`, `everyFiveMinutes()`, `hourly()`, `hourlyAt()`, `daily()`, `dailyAt()`, `twiceDaily()`, `weekly()`, `weeklyOn()`, `monthly()`, `monthlyOn()`, `quarterly()`, `yearly()`, `yearlyOn()`, and `cron()`. Each method sets the event's cron expression, which is later evaluated by `CronExpression::isDue()` from the `dragonmantank/cron-expression` package. The `->at()` method can be used after a frequency method to specify a time (e.g., `->daily()->at('13:00')`).

**Beginner-Friendly Explanation:** Laravel gives you dozens of readable methods for setting when a task runs. Instead of writing `0 2 * * *` for "every day at 2 AM," you write `->dailyAt('02:00')`. If none of the built-in methods fit, you can write the raw cron expression with `->cron('* * * * *')`.

### Purposes

- To provide a readable, English-like API for defining task frequencies.
- To eliminate the need for memorising cron syntax for common schedules.
- To support custom cron expressions for unusual schedules.
- To allow per-task time configuration via `->at()` and `->dailyAt()`.
- To enable complex schedules like "twice daily" and "last day of month."

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
->everyOddHour()
->everyTwoHours()
->everyThreeHours()
->everyFourHours()
->everySixHours()

// Day frequencies
->daily()
->dailyAt('13:00')
->twiceDaily(1, 13)          // At 1 AM and 1 PM
->twiceDailyAt(1, 13, 15)    // At 1:15 AM and 1:15 PM
->days([1, 3, 5])            // On specific days of the week

// Week frequencies
->weekly()
->weeklyOn(1, '8:00')        // Monday at 8 AM
->weekdays()                 // Monday through Friday

// Month frequencies
->monthly()
->monthlyOn(4, '15:00')      // 4th of the month at 3 PM
->twiceMonthly(1, 16, '13:00')
->lastDayOfMonth('15:00')

// Quarter frequencies
->quarterly()
->quarterlyOn(4, '14:00')    // 4th day of quarter at 2 PM

// Year frequencies
->yearly()
->yearlyOn(6, 1, '17:00')    // June 1st at 5 PM

// Custom cron expression
->cron('* * * * *')

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
- `->weeklyOn($dayOfWeek, $time)` — Runs weekly on the specified day (0 = Sunday, 6 = Saturday).
- `->monthlyOn($dayOfMonth, $time)` — Runs monthly on the specified day.
- `->yearlyOn($month, $day, $time)` — Runs yearly on the specified date.
- `->cron($expression)` — Sets a custom cron expression.
- `->days($days)` — Constrains to specific days of the week.

**Syntax Rules:**

- Frequency methods can be chained with day constraints (e.g., `->weekly()->mondays()`).
- The `->at()` method can be used after frequency methods to specify a time.
- Custom cron expressions use the standard five-field format: `minute hour day-of-month month day-of-week`.
- Sub-minute frequencies (everySecond, etc.) require `schedule:work` or a cron entry that runs more frequently than once per minute.

**Constraints and Limitations:**

- **Sub-minute frequencies require `schedule:work` or a high-frequency cron entry.** Standard cron runs at most once per minute.
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

**Example 2: Custom Cron Expression with Multiple Constraints**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// Run every 15 minutes during business hours (9 AM - 5 PM) on weekdays
Schedule::command('support:check')
    ->cron('*/15 9-17 * * 1-5')
    ->timezone('America/New_York')
    ->withoutOverlapping();

// Run at 6 AM on the first Monday of every month
Schedule::command('reports:monthly')
    ->cron('0 6 1-7 * 1')  // First Monday at 6 AM
    ->timezone('Europe/London');

// Run every 6 hours
Schedule::command('data:sync')
    ->cron('0 */6 * * *');
```

**Expected Output (schedule:list):**

```
  */15 9-17 * * 1-5  php artisan support:check ............ Next Due: 12 minutes from now
  0 6 1-7 * 1  php artisan reports:monthly ................ Next Due: (Europe/London) 5 days from now
  0 */6 * * *  php artisan data:sync ...................... Next Due: 4 hours from now
```

**Why This Output Occurs:** The `->cron()` method accepts a standard five-field cron expression. The `schedule:list` command parses the expression and calculates the next due time. The `->timezone()` method ensures the task runs at the correct local time regardless of the server's timezone. The `->withoutOverlapping()` method prevents concurrent executions.

### Real-World Cases

- **Daily backups:** `->dailyAt('02:00')` runs a backup every night at 2 AM.
- **Hourly metrics collection:** `->hourly()` collects application metrics every hour.
- **Weekly reports:** `->weeklyOn(1, '08:00')` generates a report every Monday at 8 AM.
- **Monthly billing:** `->monthlyOn(1, '00:00')` runs billing on the first of each month.
- **Business-hours monitoring:** `->cron('*/15 9-17 * * 1-5')` checks system health every 15 minutes during business hours.

---

## 3. Target Orchestration

### Definitions

**Core Definition:** Target orchestration is the practice of scheduling different types of executable targets — Artisan commands, queued jobs, closures, invokable objects, and shell commands — using the appropriate `Schedule` method for each target type.

**Technical Definition:** The `Schedule` facade provides four primary methods for registering tasks: `command()` (for Artisan commands, accepting either a command name or class name with optional arguments), `call()` (for closures and invokable objects), `job()` (for queued jobs, accepting an instance and optional queue/connection), and `exec()` (for shell commands). Each method returns an `Event` or `CallbackEvent` instance that can be further configured with frequency, constraints, timezone, output handling, and overlap prevention methods. The `Schedule::job()` method dispatches the job to the queue rather than executing it inline, providing the benefits of queue-based processing (retries, timeouts, etc.).

**Beginner-Friendly Explanation:** Laravel lets you schedule more than just Artisan commands. You can schedule a queued job (which runs in the background), a closure (inline PHP code), an invokable class, or even a raw shell command. Each type has its own method: `Schedule::command()` for Artisan, `Schedule::job()` for queued jobs, `Schedule::call()` for closures, and `Schedule::exec()` for shell commands.

### Purposes

- To schedule Artisan commands for recurring maintenance tasks.
- To schedule queued jobs for background processing with retry and timeout support.
- To schedule closures for lightweight, one-off tasks that don't warrant a dedicated command or job.
- To schedule shell commands for system-level operations (backups, file cleanup).
- To provide a consistent API across all target types, with shared frequency and constraint methods.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Schedule an Artisan command (by name or class)
Schedule::command('emails:send Taylor --force')->daily();
Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();

// Schedule a queued job
Schedule::job(new ProcessDailyMetrics)->dailyAt('01:00');
Schedule::job(new ProcessAnalytics, 'analytics', 'redis')->hourly();

// Schedule a closure
Schedule::call(function () {
    DB::table('sessions')->where('last_activity', '<', now()->subHours(24))->delete();
})->hourly()->description('Clean expired sessions');

// Schedule an invokable object
Schedule::call(new DeleteRecentUsers)->daily();

// Schedule a shell command
Schedule::exec('pg_dump mydb > /backups/db.sql')->daily();
Schedule::exec('node /home/forge/script.js')->daily();
```

**Component Breakdown:**

- `Schedule::command($command, $args)` — Schedules an Artisan command. `$command` can be a command name string or a class name; `$args` is an optional array of arguments.
- `Schedule::job($job, $queue, $connection)` — Schedules a queued job. `$job` is a job instance; `$queue` and `$connection` are optional.
- `Schedule::call($callback)` — Schedules a closure or invokable object.
- `Schedule::exec($command)` — Schedules a shell command.
- `->description($text)` — Adds a human-readable description for `schedule:list`.
- `->runInBackground()` — Runs the task in the background (only for `command` and `exec`).

**Syntax Rules:**

- The `Schedule::command()` method accepts either a command name (with arguments as a string) or a class name (with arguments as an array).
- The `Schedule::job()` method dispatches the job to the queue rather than executing it inline.
- The `Schedule::call()` method accepts a closure or an object with an `__invoke` method.
- The `Schedule::exec()` method runs the command in the operating system's shell.
- The `->runInBackground()` method may only be used with `command()` and `exec()`.

**Constraints and Limitations:**

- **Scheduled closures cannot be serialized.** They cannot be used with `schedule:work` in some configurations.
- **Scheduled jobs run on the queue, not inline.** They are subject to queue retry and timeout configuration.
- **Shell commands run with the permissions of the web server user.** Ensure the user has the necessary permissions.
- **The `->runInBackground()` method is not available for `job()` or `call()`.**
- **Artisan commands scheduled by class name must be registered.** Use the command's name if unsure.

### Annotated Code Examples

**Example 1: Scheduling All Target Types**

```php
<?php
// File: routes/console.php

use App\Console\Commands\SendEmailsCommand;
use App\Jobs\ProcessDailyMetrics;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

// Step 1: Schedule an Artisan command by name
Schedule::command('emails:send Taylor --force')
    ->dailyAt('08:00')
    ->description('Send daily emails to Taylor');

// Step 2: Schedule an Artisan command by class
Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])
    ->dailyAt('09:00');

// Step 3: Schedule a queued job
Schedule::job(new ProcessDailyMetrics, 'metrics', 'redis')
    ->dailyAt('01:00')
    ->withoutOverlapping();

// Step 4: Schedule a closure
Schedule::call(function () {
    DB::table('sessions')
        ->where('last_activity', '<', now()->subHours(24))
        ->delete();
})->hourly()
  ->description('Clean expired sessions')
  ->withoutOverlapping();

// Step 5: Schedule an invokable object
Schedule::call(new \App\Console\DeleteRecentUsers)->daily();

// Step 6: Schedule a shell command
Schedule::exec('pg_dump mydb > /backups/db.sql')
    ->dailyAt('02:00')
    ->runInBackground();
```

**Expected Output (schedule:list):**

```
  0 8 * * *    php artisan emails:send Taylor --force ..... Send daily emails to Taylor
  0 9 * * *    php artisan emails:send Taylor --force .....
  0 1 * * *    php artisan queue:job ProcessDailyMetrics ..
  0 * * * *    php artisan closure ........................ Clean expired sessions
  0 0 * * *    php artisan closure ........................
  0 2 * * *    exec pg_dump mydb > /backups/db.sql ........
```

**Why This Output Occurs:** Each scheduling method registers a different type of task. The `command()` method schedules an Artisan command; `job()` schedules a queued job; `call()` schedules a closure or invokable object; `exec()` schedules a shell command. The `schedule:list` command renders each task with its type, frequency, and description.

---

**Example 2: Scheduling a Job with Queue and Connection**

```php
<?php
// File: routes/console.php

use App\Jobs\ProcessAnalytics;
use App\Jobs\SyncExternalData;
use Illuminate\Support\Facades\Schedule;

// Schedule a job to a specific queue and connection
Schedule::job(new ProcessAnalytics, 'analytics', 'redis')
    ->hourly()
    ->withoutOverlapping(30);

// Schedule a job with a delay
Schedule::job(new SyncExternalData)
    ->everyFiveMinutes()
    ->runInBackground();
```

**Expected Output:** The `ProcessAnalytics` job is dispatched to the `analytics` queue on the `redis` connection every hour. The `SyncExternalData` job is dispatched every five minutes.

**Why This Output Occurs:** The `Schedule::job()` method accepts the job instance, an optional queue name, and an optional connection name. When the scheduler runs, it dispatches the job to the specified queue and connection. The job then runs on a queue worker, benefiting from retry, timeout, and overlap prevention.

### Real-World Cases

- **Artisan commands:** Scheduling `backup:run`, `queue:prune-failed`, `telescope:prune`, and other maintenance commands.
- **Queued jobs:** Scheduling heavy processing jobs (video transcoding, report generation) that benefit from queue retries and timeouts.
- **Closures:** Scheduling lightweight cleanup tasks (session cleanup, cache invalidation) without creating a dedicated command.
- **Shell commands:** Scheduling database backups (`pg_dump`), log rotation (`logrotate`), and file cleanup (`find ... -delete`).
- **Invokable objects:** Scheduling reusable task classes that implement `__invoke`.

---

## 4. Timezone Synchronization

### Definitions

**Core Definition:** Timezone synchronization is the practice of ensuring that scheduled tasks run at the correct local time regardless of the server's timezone, while safeguarding against daylight saving time (DST) shifts that can cause tasks to run twice or not at all.

**Technical Definition:** Laravel's scheduler supports per-task timezone configuration via the `->timezone($tz)` method, which accepts a PHP timezone identifier (e.g., `'America/New_York'`, `'Europe/London'`). When a timezone is set, the `CronExpression::isDue()` method converts the current server time to the task's timezone before evaluating the cron expression. Laravel internally creates a new `CronExpression` instance with the task's timezone and sets the current time via `Carbon::now($timezone)`. However, DST transitions can cause tasks scheduled between 2:00 AM and 3:00 AM to be skipped or duplicated because the clock moves forward or backward. The recommended mitigation is to schedule tasks outside the DST transition window (e.g., at 1:59 AM or 3:01 AM) or use UTC for all scheduling.

**Beginner-Friendly Explanation:** If your server is in UTC but you want a task to run at 7 AM London time, you can use `->timezone('Europe/London')`. Laravel will calculate when that is in UTC. But daylight saving time can cause problems: if you schedule a task for 2:30 AM on the night the clocks change, it might run twice or not at all. The safest approach is to either use UTC for scheduling (and convert inside the task) or schedule tasks outside the DST transition window.

### Purposes

- To run scheduled tasks at a specific local time regardless of the server's timezone.
- To support applications serving users in multiple timezones with localized task execution.
- To avoid the complexity of manually converting times to UTC.
- To safeguard against DST transitions that can cause tasks to be skipped or duplicated.
- To provide a consistent scheduling experience across servers in different regions.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Set a task's timezone
Schedule::command('reports:generate')
    ->dailyAt('08:00')
    ->timezone('America/New_York');

// Set a task's timezone with a custom cron expression
Schedule::command('support:check')
    ->cron('*/15 9-17 * * 1-5')
    ->timezone('Europe/London');

// Use the application's default timezone (from config/app.php)
Schedule::command('emails:send')
    ->dailyAt('08:00');
// Uses config('app.timezone') — defaults to UTC
```

**Component Breakdown:**

- `->timezone($tz)` — Sets the timezone for the task. Accepts any PHP timezone identifier.
- The application's default timezone is configured in `config/app.php` under the `'timezone'` key (default: `'UTC'`).
- The `->timezone()` method can be chained with any frequency method.

**Syntax Rules:**

- The timezone must be a valid PHP timezone identifier (e.g., `'America/New_York'`, `'Europe/London'`, `'Asia/Tokyo'`).
- The timezone affects only the scheduled time evaluation; internal date operations in the task use the application's default timezone unless changed explicitly.
- The `->timezone()` method can be combined with `->between()`, `->unlessBetween()`, and other time-based constraints.

**Constraints and Limitations:**

- **DST transitions can cause tasks to be skipped or duplicated.** Tasks scheduled between 2:00 AM and 3:00 AM on DST transition days are particularly affected.
- **Laravel does not automatically adjust for DST.** The scheduler evaluates the cron expression against the current time in the task's timezone, but DST transitions can cause anomalies.
- **The recommended mitigation is to schedule outside the DST window** (e.g., at 1:59 AM or 3:01 AM) or use UTC for all scheduling.
- **Changing the application's default timezone affects all tasks without an explicit `->timezone()` call.**
- **Some timezones have non-hour offsets** (e.g., `'Asia/Kolkata'` is UTC+5:30). Laravel handles these correctly via Carbon.

### Annotated Code Examples

**Example 1: Scheduling in a Specific Timezone**

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

---

**Example 2: Safeguarding Against DST Shifts**

```php
<?php
// File: routes/console.php

use Illuminate\Support\Facades\Schedule;

// UNSAFE: Scheduled at 2:30 AM, which falls in the DST transition window
Schedule::command('reports:generate')
    ->dailyAt('02:30')
    ->timezone('America/New_York');
    // On DST transition nights, this may run twice or not at all.

// SAFE: Scheduled at 1:59 AM, outside the DST transition window
Schedule::command('reports:generate')
    ->dailyAt('01:59')
    ->timezone('America/New_York');
    // Runs consistently every night.

// SAFE: Scheduled at 3:01 AM, outside the DST transition window
Schedule::command('reports:generate')
    ->dailyAt('03:01')
    ->timezone('America/New_York');
    // Runs consistently every night.

// SAFEST: Use UTC for scheduling and convert inside the task
Schedule::command('reports:generate')
    ->dailyAt('07:00'); // 7 AM UTC = 2 AM EST / 3 AM EDT
    // The task itself can convert to local time for display.
```

**Expected Output:** Tasks scheduled at 1:59 AM or 3:01 AM run consistently on DST transition days. Tasks scheduled at 2:30 AM may be skipped or duplicated.

**Why This Output Occurs:** DST transitions occur at 2:00 AM in most timezones. When the clock springs forward (e.g., 2:00 AM becomes 3:00 AM), tasks scheduled between 2:00 AM and 2:59 AM are skipped. When the clock falls back (e.g., 3:00 AM becomes 2:00 AM), tasks scheduled between 2:00 AM and 2:59 AM run twice. Scheduling outside this window avoids both anomalies.

### Real-World Cases

- **Multi-region SaaS:** Tasks are scheduled in each region's local timezone (e.g., `'America/New_York'` for US users, `'Europe/London'` for UK users).
- **Daily reports:** A report generation task runs at 8 AM in the company's headquarters timezone.
- **Timezone-sensitive notifications:** Reminder emails are sent at 9 AM in each user's local timezone.
- **DST-safe scheduling:** Financial batch processing avoids the DST transition window by scheduling at 1:59 AM or 3:01 AM.
- **UTC-first approach:** All scheduling is done in UTC, and the tasks themselves convert to local time for user-facing operations.

---

## References

- Laravel Task Scheduling Documentation (Master) — https://laravel.com/framework/docs/master/scheduling
- Laravel Task Scheduling Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/scheduling
- Laravel Task Scheduling Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/scheduling
- Laravel 11 Release Notes: Scheduling — https://laravel.com/docs/11.x/releases#scheduling
- Laravel `Illuminate\Console\Scheduling\Schedule` API — https://api.laravel.com/docs/master/Illuminate/Console/Scheduling/Schedule.html
- Laravel `Illuminate\Console\Scheduling\Event` API — https://api.laravel.com/docs/master/Illuminate/Console/Scheduling/Event.html
- Laravel `Illuminate\Console\Scheduling\ManagesFrequencies` API — https://api.laravel.com/docs/master/Illuminate/Console/Scheduling/ManagesFrequencies.html
- Laravel `Illuminate\Console\Scheduling\CallbackEvent` API — https://api.laravel.com/docs/master/Illuminate/Console/Scheduling/CallbackEvent.html
- Laravel News: Streamlining Application Automation with Laravel's Task Scheduler — https://laravel-news.com/task-scheduler
- Laravel Kernel.php file alternative (Stack Overflow) — https://stackoverflow.com/questions/78532447/laravel-kernel-php-file-alternative
- Laravel 12: Run scheduled command at 07:00 UK time while server is on UTC (Stack Overflow) — https://stackoverflow.com/questions/79755514
- How does Laravel scheduler handle daylight saving time? (Stack Overflow) — https://stackoverflow.com/questions/46604614
- Laravel Task Scheduling Best Practices (GitHub) — https://github.com/NeverSight/skills_feed/blob/main/data/skills-md/iserter/laravel-claude-agents/laravel-task-scheduling/SKILL.md
- Laravel Scheduling Guidelines (GitHub) — https://github.com/event4u-app/agent-config/blob/53e066135e7ae8a993e9f17c541acc6603f60608/dist/agent-src/skills/laravel-scheduling/SKILL.md
- Dragonmantank Cron Expression (GitHub) — https://github.com/dragonmantank/cron-expression
- Laravel `schedule:list` Command — https://laravel.com/docs/master/scheduling#viewing-the-schedule
- Laravel `schedule:test` Command — https://laravel.com/docs/master/scheduling#testing-the-schedule
- Laravel `schedule:work` Command — https://laravel.com/docs/master/scheduling#running-the-scheduler-locally
- Laravel Maintenance Mode Documentation — https://laravel.com/docs/master/configuration#maintenance-mode
- Laravel `withoutOverlapping` Documentation — https://laravel.com/docs/master/scheduling#preventing-task-overlaps
- Laravel `onOneServer` Documentation — https://laravel.com/docs/master/scheduling#running-tasks-on-one-server
- Laravel `runInBackground` Documentation — https://laravel.com/docs/master/scheduling#running-tasks-in-the-background
- Laravel `environments` Documentation — https://laravel.com/docs/master/scheduling#scheduling-tasks-conditionally
- Laravel `timezone` Documentation — https://laravel.com/docs/master/scheduling#timezones
- Laravel `between` and `unlessBetween` Documentation — https://laravel.com/docs/master/scheduling#time-based-constraints