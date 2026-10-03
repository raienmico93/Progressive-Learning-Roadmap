# Laravel Custom Artisan Commands — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Custom Artisan commands are user-defined classes that extend Laravel's `Illuminate\Console\Command` base class, enabling developers to encapsulate application-specific logic behind a terminal interface with arguments, options, interactive prompts, formatted output, and scheduling support.

**Technical Definition:** A custom Artisan command is a PHP class extending `Illuminate\Console\Command` (which itself extends `Symfony\Component\Console\Command\Command`). The command declares its interface through the `$signature` property — a declarative DSL parsed by `Illuminate\Console\Parser` into a Symfony `InputDefinition` containing `InputArgument` and `InputOption` instances. The command's `handle()` method is invoked by the `Illuminate\Console\Application` after the service container resolves the command and injects any dependencies declared in the method signature. Commands are auto-discovered from `app/Console/Commands` by the console kernel, or registered explicitly via the kernel's `$commands` array (Laravel 10-) or `withCommands()` (Laravel 11+).

**Beginner-Friendly Explanation:** Laravel ships with dozens of built-in commands (`migrate`, `make:model`, `queue:work`), but every application eventually needs its own. A custom Artisan command is a PHP class you write that can be run from the terminal with `php artisan your:command`. It can accept arguments, offer options, ask the user questions, display progress bars, and perform any task your application needs — from importing data to sending reports to cleaning up old records.

### Key Characteristics

- **Signature-driven interface:** A single `$signature` string defines the command name, arguments, and options declaratively.
- **Container-resolved dependencies:** The `handle()` method can type-hint any class resolvable by Laravel's service container, and it will be injected automatically.
- **Auto-discovery:** Commands placed in `app/Console/Commands` are automatically registered without manual configuration.
- **Rich I/O API:** Commands have access to formatted output helpers (`info()`, `error()`, `table()`, etc.) and interactive prompt methods.
- **Isolation support:** Commands can implement `Isolatable` to prevent concurrent execution.
- **Schedulable:** Any command can be registered in the scheduler for automatic recurring execution.
- **Testable:** Commands can be tested with `$this->artisan()` and assertions on output and exit codes.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the console kernel configured.
- Basic understanding of PHP classes, namespaces, and dependency injection.
- Familiarity with Artisan fundamentals (see the Artisan Fundamentals cheat sheet).

### Related Programming Areas

- **Symfony Console** — The underlying component providing command parsing, input handling, and output formatting.
- **Service Container** — Commands are resolved through and can inject dependencies from Laravel's DI container.
- **Scheduling** — Commands can be registered for recurring execution via the console kernel.
- **Testing** — Laravel's testing framework provides helpers for asserting command output and exit codes.
- **Laravel Prompts** — The modern prompt library used for interactive input.

### Core Concepts / Features

1. Command Classes (Creating custom commands using `make:command`)
2. The Signature Property (Defining names, arguments, and options concisely in one string)
3. Input & Output API (Using `info()`, `error()`, `line()`, and `table()` to format terminal outputs)
4. User Interaction (Implementing text fields, choices, secret inputs, and confirmation prompts)
5. Progress Indicators (Displaying visual loading feedback using progress bars and spinners)
6. Validation (Validating user-provided command-line arguments and input options)
7. Isolation & Locks (Preventing overlapping command executions via the `Isolatable` interface)

---

## 1. Command Classes

### Definitions

**Core Definition:** A command class is a PHP class extending `Illuminate\Console\Command` that encapsulates the logic, interface, and behaviour of a custom Artisan command.

**Technical Definition:** The `make:command` Artisan command generates a class extending `Illuminate\Console\Command` in the `app/Console/Commands` directory with the namespace `App\Console\Commands`. The generated class defines a `$signature` property (or `$name` and `$description` properties in legacy syntax) and a `handle()` method. The console kernel's `commands()` method calls `$this->load(__DIR__.'/Commands')`, which uses `Illuminate\Foundation\Console\Kernel::commands()` to scan the directory and register every class that extends `Command`. Commands are resolved from the container when executed, allowing constructor and method injection.

**Beginner-Friendly Explanation:** A command class is just a PHP file where you write the code that runs when someone types `php artisan your:command`. You generate it with `php artisan make:command YourCommandName`, then edit the generated file to define what the command is called, what arguments it takes, and what it does. Laravel automatically finds it in the `app/Console/Commands` folder — no registration required.

### Purposes

- To encapsulate application-specific CLI logic in a dedicated, testable class.
- To provide a structured interface for running maintenance, data processing, and automation tasks.
- To enable dependency injection of services and repositories into command logic.
- To support scheduling and programmatic invocation via `Artisan::call()`.
- To organise CLI functionality alongside the rest of the application codebase.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
php artisan make:command <Name> [--command=<name>] [--test] [--pest]
```

**Component Breakdown:**

- `<Name>` — The class name (e.g., `SendReminders`, `ImportUsers`). Placed in `app/Console/Commands`.
- `--command=<name>` — Specify the terminal command name (e.g., `reminders:send`). If omitted, Laravel guesses from the class name.
- `--test` — Generate an accompanying PHPUnit test.
- `--pest` — Generate an accompanying Pest test.

```php
<?php
// File: app/Console/Commands/SendReminders.php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class SendReminders extends Command
{
    protected $signature = 'reminders:send
                            {user : The ID of the user}
                            {--queue : Queue the reminder instead of sending immediately}';

    protected $description = 'Send a reminder to a user';

    public function handle(): int
    {
        $this->info('Reminder sent.');
        return Command::SUCCESS;
    }
}
```

**Syntax Rules:**

- The command class must extend `Illuminate\Console\Command`.
- The `$signature` property is preferred over the legacy `$name` and `$description` properties.
- The `handle()` method must return an integer exit code (`Command::SUCCESS` = 0, `Command::FAILURE` = 1).
- Commands are auto-discovered from `app/Console/Commands`; commands elsewhere must be registered explicitly.
- Constructor and method injection work as in any container-resolved class.

**Constraints and Limitations:**

- **Commands cannot be nested arbitrarily deep.** While subdirectories are supported, the namespace must match the directory structure.
- **Command names must be unique.** Registering two commands with the same name causes the later one to overwrite the earlier.
- **The `handle()` method should not call `exit()`.** Return an exit code instead to allow Laravel to perform cleanup.
- **Long-running commands should handle signals** (SIGTERM, SIGINT) for graceful shutdown.

### Annotated Code Examples

**Example 1: Generating and Implementing a Command**

```bash
# Step 1: Generate the command class
php artisan make:command SendWelcomeEmails --command=emails:welcome
```

**Expected Output:**

```
INFO  Command [app/Console/Commands/SendWelcomeEmails.php] created successfully.
```

**Generated File:**

```php
<?php
// File: app/Console/Commands/SendWelcomeEmails.php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class SendWelcomeEmails extends Command
{
    /**
     * The name and signature of the console command.
     */
    protected $signature = 'emails:welcome';

    /**
     * The console command description.
     */
    protected $description = 'Command description';

    /**
     * Execute the console command.
     */
    public function handle(): int
    {
        return Command::SUCCESS;
    }
}
```

**Step-by-Step Implementation:**

```php
<?php
// Step 2: Implement the command logic

namespace App\Console\Commands;

use App\Models\User;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmails extends Command
{
    protected $signature = 'emails:welcome
                            {--queue : Queue emails instead of sending immediately}
                            {--limit=50 : Maximum number of emails to send}';

    protected $description = 'Send welcome emails to newly registered users';

    public function handle(): int
    {
        $limit = (int) $this->option('limit');

        $users = User::whereNull('welcomed_at')
            ->limit($limit)
            ->get();

        if ($users->isEmpty()) {
            $this->info('No users to welcome.');
            return Command::SUCCESS;
        }

        $bar = $this->output->createProgressBar($users->count());
        $bar->start();

        foreach ($users as $user) {
            if ($this->option('queue')) {
                Mail::to($user)->queue(new \App\Mail\WelcomeEmail($user));
            } else {
                Mail::to($user)->send(new \App\Mail\WelcomeEmail($user));
            }

            $user->update(['welcomed_at' => now()]);
            $bar->advance();
        }

        $bar->finish();
        $this->newLine();
        $this->info("Sent welcome emails to {$users->count()} users.");

        return Command::SUCCESS;
    }
}
```

```bash
# Step 3: Run the command
php artisan emails:welcome --limit=10
```

**Expected Output:**

```
 10/10 [============================] 100%
Sent welcome emails to 10 users.
```

**Why This Output Occurs:** The command is auto-discovered from `app/Console/Commands` by the kernel's `load()` method. The `$signature` defines the command name (`emails:welcome`), a boolean option (`--queue`), and a value option (`--limit` with default `50`). The `handle()` method queries users, creates a progress bar, sends emails, updates each user, and returns `Command::SUCCESS`.

---

**Example 2: Dependency Injection in Commands**

```php
<?php
// File: app/Console/Commands/ProcessOrders.php

namespace App\Console\Commands;

use App\Services\OrderProcessor;
use Illuminate\Console\Command;

class ProcessOrders extends Command
{
    protected $signature = 'orders:process {--date=today : The date to process}';
    protected $description = 'Process pending orders';

    /**
     * Dependencies can be injected via the handle() method.
     * The container resolves them automatically.
     */
    public function handle(OrderProcessor $processor): int
    {
        $date = $this->option('date');
        $this->info("Processing orders for {$date}...");

        $count = $processor->processForDate($date);

        $this->info("Processed {$count} orders.");
        return Command::SUCCESS;
    }
}
```

**Expected Output:**

```
Processing orders for today...
Processed 42 orders.
```

**Why This Output Occurs:** The `handle()` method type-hints `OrderProcessor`. When the command is executed, the container resolves the `OrderProcessor` instance (including its own dependencies) and injects it. This makes commands easy to test — a mock `OrderProcessor` can be bound in the container during testing.

### Real-World Cases

- **Data import:** A `data:import` command reads a CSV, validates each row, and inserts records into the database.
- **Report generation:** A `reports:generate` command aggregates data and produces a PDF or CSV report.
- **Maintenance tasks:** A `cleanup:temp` command removes temporary files older than a specified age.
- **Notification dispatch:** A `notifications:send` command sends pending notifications to users.
- **System health checks:** A `system:health` command verifies database connectivity, cache availability, and queue status.

---

## 2. The Signature Property

### Definitions

**Core Definition:** The `$signature` property is a declarative string that defines a command's terminal name, its positional arguments, and its options in a single, concise expression.

**Technical Definition:** The `$signature` string is parsed by `Illuminate\Console\Parser::parse()` into a Symfony `InputDefinition`. The first token is the command name. Subsequent tokens enclosed in curly braces define arguments (`{name}`, `{name?}`, `{name=default}`, `{name*}`) and options (`{--flag}`, `{--option=}`, `{--option=default}`, `{--short|long}`, `{--array=*}`). Descriptions can be included after a colon (`{name : Description}`). The parser validates the syntax and constructs `InputArgument` and `InputOption` instances with the appropriate modes (`REQUIRED`, `OPTIONAL`, `IS_ARRAY`, `VALUE_NONE`, `VALUE_REQUIRED`, `VALUE_OPTIONAL`).

**Beginner-Friendly Explanation:** The `$signature` is a single line (or multiline string) where you describe everything about your command's interface: its name, what arguments it needs, and what options it accepts. Laravel reads this string and automatically sets up all the parsing and help documentation. It's like a blueprint for your command's terminal interface.

### Purposes

- To define the command's terminal name, arguments, and options in a single, readable expression.
- To enable automatic generation of help text and usage documentation.
- To provide type information (required, optional, array) for arguments and options.
- To attach descriptions to arguments and options for the help system.
- To eliminate the need for separate `configure()` method boilerplate.

### Syntax Rules and Structure

#### Complete General Syntax

```
protected $signature = 'namespace:command
    {requiredArgument : Description}
    {optionalArgument? : Description}
    {argumentWithDefault=value : Description}
    {arrayArgument* : Description}
    {--flag : Description}
    {--valueOption= : Description}
    {--optionWithDefault=default : Description}
    {--S|shortOption : Description}
    {--arrayOption=* : Description}';
```

**Component Breakdown:**

| Syntax | Type | Behaviour |
|--------|------|-----------|
| `{name}` | Required argument | Must be provided |
| `{name?}` | Optional argument | Returns `null` if omitted |
| `{name=default}` | Optional argument with default | Returns default if omitted |
| `{name*}` | Array argument | Collects multiple values |
| `{--flag}` | Boolean switch | `true` if present, `false` otherwise |
| `{--option=}` | Required-value option | Must be provided with a value |
| `{--option=default}` | Optional-value option | Returns default if omitted |
| `{--S|short}` | Option with short form | Both `-S` and `--short` work |
| `{--tag=*}` | Array option | Collects multiple values |
| `{name : Description}` | Description | Adds help text for the argument/option |

```php
protected $signature = 'orders:process
    {orderId : The ID of the order to process}
    {--queue : Queue the processing instead of running synchronously}
    {--priority=normal : The priority level (low, normal, high)}
    {--F|force : Skip confirmation prompts}
    {--tag=* : Tags to apply to the order}';
```

**Syntax Rules:**

- The command name is the first token; it should follow the `namespace:command` convention.
- Arguments are positional and must be defined before options.
- Array arguments (`{name*}`) must be the last argument.
- Descriptions are separated from the argument/option name by a colon and a space.
- Whitespace and newlines within the signature string are ignored by the parser, allowing multi-line formatting for readability.
- Short options are single letters and are defined with a pipe: `{--F|force}`.

**Constraints and Limitations:**

- **The signature must be a valid string literal.** It cannot be dynamically generated at runtime.
- **Argument names cannot contain spaces or special characters** other than underscores.
- **Option names are case-sensitive** and should be lowercase with hyphens for multi-word names.
- **Array arguments and array options cannot have defaults** — they default to empty arrays.
- **The parser will throw an exception if the signature is malformed.** Test the command after writing the signature.

### Annotated Code Examples

**Example 1: Comprehensive Signature Definition**

```php
<?php
// File: app/Console/Commands/ImportData.php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ImportData extends Command
{
    protected $signature = 'data:import
                            {file : The CSV file to import}
                            {--format=csv : The input format (csv, json, xml)}
                            {--batch-size=500 : Number of rows per batch}
                            {--D|dry-run : Simulate without writing to the database}
                            {--tag=* : Tags to apply to imported records}';

    protected $description = 'Import data from a file into the database';

    public function handle(): int
    {
        // Retrieve argument values
        $file = $this->argument('file');

        // Retrieve option values
        $format = $this->option('format');
        $batchSize = (int) $this->option('batch-size');
        $dryRun = $this->option('dry-run');
        $tags = $this->option('tag');

        $this->info("Importing {$file} as {$format}");
        $this->line("Batch size: {$batchSize}");
        $this->line("Dry run: " . ($dryRun ? 'yes' : 'no'));
        $this->line("Tags: " . (empty($tags) ? 'none' : implode(', ', $tags)));

        return Command::SUCCESS;
    }
}
```

```bash
# Step 1: Run with defaults
php artisan data:import users.csv
# Output:
# Importing users.csv as csv
# Batch size: 500
# Dry run: no
# Tags: none

# Step 2: Run with all options
php artisan data:import users.csv --format=json --batch-size=1000 --dry-run --tag=imported --tag=2025
# Output:
# Importing users.csv as json
# Batch size: 1000
# Dry run: yes
# Tags: imported, 2025

# Step 3: Using the short form
php artisan data:import users.csv -D
# Output:
# Importing users.csv as csv
# Batch size: 500
# Dry run: yes
# Tags: none
```

**Expected Output:** The command reads the required `file` argument, the `--format` option (default `csv`), the `--batch-size` option (default `500`), the `--dry-run` boolean switch (short form `-D`), and the `--tag` array option. All values are reflected in the output.

**Why This Output Occurs:** The parser converts the signature into an `InputDefinition`. The `argument('file')` method returns the required argument's value. The `option('format')` method returns the option's value or default. The `option('dry-run')` method returns `true` if the flag was present, `false` otherwise. The `option('tag')` method returns an array of all `--tag` values.

---

**Example 2: Verifying Signature with Help Output**

```bash
php artisan help data:import
```

**Expected Output:**

```
Description:
  Import data from a file into the database

Usage:
  data:import [options] [--] <file>

Arguments:
  file                     The CSV file to import

Options:
      --format[=FORMAT]    The input format (csv, json, xml) [default: "csv"]
      --batch-size[=BATCH-SIZE]
                           Number of rows per batch [default: "500"]
  -D, --dry-run            Simulate without writing to the database
      --tag[=TAG]          Tags to apply to imported records (multiple values allowed)
  -h, --help               Display help for the given command
  ...
```

**Why This Output Occurs:** The `help` command renders the `InputDefinition` derived from the signature. Required arguments appear without brackets (`<file>`), optional arguments with brackets (`[<file>]`), and options with their defaults shown. Array options note "(multiple values allowed)". The short form `-D` is displayed alongside the long form `--dry-run`.

### Real-World Cases

- **Data import commands:** `data:import {file} {--format=csv} {--batch-size=500}` defines a flexible import command.
- **Queue workers:** `queue:work {connection?} {--queue=default} {--tries=3} {--timeout=60}` defines a configurable queue worker.
- **Report generation:** `reports:generate {type} {--from=} {--to=} {--format=pdf}` defines a report generator with date range options.
- **User management:** `users:create {--name=} {--email=} {--role=viewer}` defines a user creation command with option-based input.
- **Scheduled tasks:** `cache:warm {--tags=*}` defines a cache warming command that accepts multiple tags.

---

## 3. Input & Output API

### Definitions

**Core Definition:** The Input & Output API is the set of methods provided by `Illuminate\Console\Command` for reading user input and writing formatted output to the terminal.

**Technical Definition:** The `Command` class provides output helpers (`info()`, `error()`, `warn()`, `line()`, `comment()`, `question()`, `table()`, `newLine()`) that delegate to the underlying `OutputInterface` (typically `ConsoleOutput`). These methods use Symfony's `OutputFormatter` to apply ANSI styling (colours, bold, background). The `$this->output` property exposes the `OutputStyle` instance, which provides additional methods like `createProgressBar()`, `ask()`, `confirm()`, and `choice()`. Output verbosity is controlled by the `-v`, `-vv`, `-vvv`, and `-q` flags.

**Beginner-Friendly Explanation:** When your command runs, it needs to tell the user what's happening. Laravel gives you simple methods for this: `info()` for general messages, `error()` for problems, `line()` for plain text, and `table()` for tabular data. You can also control how much output is shown using verbosity flags — from silent (`-q`) to very verbose (`-vvv`).

### Purposes

- To communicate command progress, results, and errors to the user in a formatted, readable way.
- To display structured data (tables, lists) in the terminal.
- To control output verbosity based on the user's preference.
- To provide visual distinction between different message types (info, error, warning, comment).
- To support scripting by producing machine-readable output when needed.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Basic output methods
$this->info(string $message);          // Green background, informational
$this->error(string $message);         // Red background, error
$this->warn(string $message);          // Yellow background, warning
$this->line(string $message);          // Plain text, no styling
$this->comment(string $message);       // Yellow text, commentary
$this->question(string $message);      // Cyan text, question
$this->newLine(int $count = 1);        // Blank line(s)
$this->table(array $headers, array $rows, string $style = 'default');

// Verbosity checks
if ($this->output->isVerbose()) { ... }       // -v
if ($this->output->isVeryVerbose()) { ... }   // -vv
if ($this->output->isDebug()) { ... }         // -vvv
if ($this->output->isQuiet()) { ... }         // -q
```

**Component Breakdown:**

- `info($message)` — Displays a message with a green background. Used for general status updates.
- `error($message)` — Displays a message with a red background. Used for failures.
- `warn($message)` — Displays a message with a yellow background. Used for warnings.
- `line($message)` — Displays plain text without background colouring.
- `comment($message)` — Displays text in yellow. Used for commentary.
- `table($headers, $rows, $style)` — Displays a formatted table. Styles include `default`, `box`, `compact`, `borderless`, `symfony-style-guide`.
- `newLine($count)` — Inserts blank lines.

```php
// Example usage
$this->info('Starting import...');
$this->warn('Skipping invalid rows.');
$this->error('Import failed: file not found.');
$this->line('Processed 1,234 rows.');
$this->newLine();

$this->table(
    ['ID', 'Name', 'Email'],
    [
        [1, 'Alice', 'alice@example.com'],
        [2, 'Bob', 'bob@example.com'],
    ]
);
```

**Syntax Rules:**

- The `info()`, `error()`, `warn()`, `line()`, and `comment()` methods accept a single string argument.
- The `table()` method accepts an array of headers and an array of rows, with optional style parameter.
- Verbosity checks return `true` or `false` based on the flags passed to the command.
- Output methods automatically respect the `-q` (quiet) flag — no output is produced when quiet mode is active.
- The `$this->output` property provides access to the underlying `OutputStyle` for advanced formatting.

**Constraints and Limitations:**

- **Output methods do not return values.** They are purely for display.
- **ANSI styling may not render correctly in all terminals.** Use `--no-ansi` to disable styling.
- **The `table()` method requires all rows to have the same number of columns** as the headers array.
- **Verbosity checks are not a substitute for proper error handling.** Use exceptions and exit codes for control flow.

### Annotated Code Examples

**Example 1: Formatted Output for a Data Processing Command**

```php
<?php
// File: app/Console/Commands/ProcessData.php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ProcessData extends Command
{
    protected $signature = 'data:process {--dry-run}';
    protected $description = 'Process data with formatted output';

    public function handle(): int
    {
        // Step 1: Display a header
        $this->info('=== Data Processing ===');
        $this->newLine();

        // Step 2: Display a warning if in dry-run mode
        if ($this->option('dry-run')) {
            $this->warn('DRY RUN MODE — no changes will be made.');
            $this->newLine();
        }

        // Step 3: Display a table of records to process
        $records = [
            ['id' => 1, 'name' => 'Record A', 'status' => 'pending'],
            ['id' => 2, 'name' => 'Record B', 'status' => 'pending'],
            ['id' => 3, 'name' => 'Record C', 'status' => 'processed'],
        ];

        $this->table(
            ['ID', 'Name', 'Status'],
            array_map(fn ($r) => [$r['id'], $r['name'], $r['status']], $records)
        );

        // Step 4: Process each record with verbose output
        foreach ($records as $record) {
            if ($record['status'] === 'processed') {
                $this->line("Skipping {$record['name']} (already processed).");
                continue;
            }

            $this->info("Processing {$record['name']}...");

            if ($this->output->isVerbose()) {
                $this->comment("  → Record ID: {$record['id']}");
                $this->comment("  → Previous status: {$record['status']}");
            }
        }

        // Step 5: Display a summary
        $this->newLine();
        $this->info('=== Processing Complete ===');
        $this->line('Processed: 2 records');
        $this->line('Skipped: 1 record');

        return Command::SUCCESS;
    }
}
```

```bash
# Step 1: Run normally
php artisan data:process

# Step 2: Run with verbose output
php artisan data:process -v

# Step 3: Run in dry-run mode
php artisan data:process --dry-run
```

**Expected Output (normal):**

```
=== Data Processing ===

+----+----------+-----------+
| ID | Name     | Status    |
+----+----------+-----------+
| 1  | Record A | pending   |
| 2  | Record B | pending   |
| 3  | Record C | processed |
+----+----------+-----------+

Skipping Record C (already processed).
Processing Record A...
Processing Record B...

=== Processing Complete ===
Processed: 2 records
Skipped: 1 record
```

**Expected Output (verbose, `-v`):**

```
...
Processing Record A...
  → Record ID: 1
  → Previous status: pending
Processing Record B...
  → Record ID: 2
  → Previous status: pending
...
```

**Why This Output Occurs:** The `table()` method renders a bordered table using Symfony's Table helper. The `info()`, `warn()`, `line()`, and `comment()` methods apply their respective formatting. The `isVerbose()` check enables additional output only when the `-v` flag is passed. The output respects the terminal's width and the command's verbosity settings.

---

**Example 2: Using the OutputStyle for Advanced Formatting**

```php
<?php
// File: app/Console/Commands/ReportCommand.php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ReportCommand extends Command
{
    protected $signature = 'report:summary';
    protected $description = 'Display a summary report with styled output';

    public function handle(): int
    {
        // Access the OutputStyle instance
        $output = $this->output;

        // Display a title with custom formatting
        $output->title('Monthly Report');
        $output->newLine();

        // Display sections
        $output->section('Sales');
        $output->listing([
            'Total revenue: $125,000',
            'Orders processed: 1,234',
            'Average order value: $101.30',
        ]);

        $output->newLine();
        $output->section('Customers');
        $output->listing([
            'New customers: 456',
            'Returning customers: 789',
            'Churn rate: 2.3%',
        ]);

        // Display a table
        $output->newLine();
        $output->table(
            ['Metric', 'Value', 'Change'],
            [
                ['Revenue', '$125,000', '+12%'],
                ['Orders', '1,234', '+8%'],
                ['Customers', '1,245', '+15%'],
            ]
        );

        return Command::SUCCESS;
    }
}
```

**Expected Output:**

```
Monthly Report
==============

Sales
-----
  * Total revenue: $125,000
  * Orders processed: 1,234
  * Average order value: $101.30

Customers
---------
  * New customers: 456
  * Returning customers: 789
  * Churn rate: 2.3%

+-----------+----------+--------+
| Metric    | Value    | Change |
+-----------+----------+--------+
| Revenue   | $125,000 | +12%   |
| Orders    | 1,234    | +8%    |
| Customers | 1,245    | +15%   |
+-----------+----------+--------+
```

**Why This Output Occurs:** The `OutputStyle` class provides convenience methods (`title()`, `section()`, `listing()`) that format output with appropriate styling. `title()` renders a centred, bold title with an underline. `section()` renders a section heading. `listing()` renders a bulleted list. `table()` renders a formatted table. These methods use Symfony's `OutputFormatter` to apply ANSI styling where supported.

### Real-World Cases

- **Data import commands:** Display progress tables, row counts, and error summaries.
- **Report generation:** Format financial or statistical data in tables and sections.
- **Deployment scripts:** Display step-by-step progress with `info()` and `comment()`.
- **Error reporting:** Use `error()` and `warn()` to highlight problems, and `-v` flags for detailed diagnostics.
- **CI/CD pipelines:** Use `line()` for machine-parseable output and `table()` for human-readable summaries.

---

## 4. User Interaction

### Definitions

**Core Definition:** User interaction is the set of methods and functions that allow a command to prompt the user for input — text, choices, secrets, and confirmations — during execution.

**Technical Definition:** Laravel provides two systems for user interaction: the legacy Symfony Console helpers (`$this->ask()`, `$this->secret()`, `$this->confirm()`, `$this->choice()`, `$this->anticipate()`) and the modern `Laravel\Prompts` package (introduced in Laravel 9/10) which provides functions like `text()`, `password()`, `confirm()`, `select()`, `multiselect()`, `search()`, and `suggest()`. The `Laravel\Prompts` package automatically detects terminal capabilities and falls back to simple input when advanced features are unavailable.

**Beginner-Friendly Explanation:** Sometimes a command needs more information than you provided on the command line. Instead of failing, it can ask you interactively: "What is your name?" or "Are you sure?" Laravel provides simple prompts for text input, yes/no confirmation, and selecting from a list. The modern `Laravel\Prompts` package adds even more options, like multi-select and autocomplete.

### Purposes

- To request missing required information without failing the command.
- To confirm potentially destructive actions before executing them.
- To allow users to select from a list of valid options.
- To collect passwords and secrets without echoing them to the terminal.
- To provide a guided, conversational experience for complex commands.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Legacy Symfony Console helpers
$this->ask(string $question, string $default = null);
$this->secret(string $question, bool $fallback = true);
$this->confirm(string $question, bool $default = false);
$this->choice(string $question, array $choices, string $default = null, $attempts = null, bool $multiple = false);
$this->anticipate(string $question, array $choices, string $default = null);

// Modern Laravel\Prompts functions
use function Laravel\Prompts\text;
use function Laravel\Prompts\password;
use function Laravel\Prompts\confirm;
use function Laravel\Prompts\select;
use function Laravel\Prompts\multiselect;
use function Laravel\Prompts\search;
use function Laravel\Prompts\suggest;

text(string $label, string $placeholder = '', string $default = '', bool $required = false, callable $validate = null);
password(string $label, string $placeholder = '', bool $required = false, callable $validate = null);
confirm(string $label, bool $default = false, bool $required = false);
select(string $label, array $options, string|int $default = null, int $scroll = 5);
multiselect(string $label, array $options, array $default = [], bool $required = false);
search(string $label, callable $options, string $placeholder = '', int $scroll = 5);
suggest(string $label, array $options, string $default = '', bool $required = false);
```

**Component Breakdown:**

- `ask($question, $default)` — Prompts for text input with an optional default.
- `secret($question, $fallback)` — Prompts for hidden input (passwords). Falls back to visible input if hidden input is unavailable.
- `confirm($question, $default)` — Prompts for yes/no confirmation.
- `choice($question, $choices, $default, $attempts, $multiple)` — Prompts for a single or multiple selection from a list.
- `anticipate($question, $choices, $default)` — Prompts for text input with autocomplete suggestions.
- `text($label, $placeholder, $default, $required, $validate)` — Modern text input with validation.
- `password($label, $placeholder, $required, $validate)` — Modern hidden input.
- `confirm($label, $default, $required)` — Modern confirmation prompt.
- `select($label, $options, $default, $scroll)` — Modern single-selection menu.
- `multiselect($label, $options, $default, $required)` — Modern multi-selection menu.
- `search($label, $options, $placeholder, $scroll)` — Modern searchable selection.
- `suggest($label, $options, $default, $required)` — Modern autocomplete input.

```php
// Legacy example
$name = $this->ask('What is your name?', 'Guest');
$password = $this->secret('What is the password?');
$confirmed = $this->confirm('Are you sure?', false);
$role = $this->choice('Which role?', ['admin', 'editor', 'viewer'], 'viewer');

// Modern example
use function Laravel\Prompts\text;
use function Laravel\Prompts\select;

$name = text(
    label: 'What is your name?',
    placeholder: 'e.g., Jane Doe',
    required: true,
    validate: fn (string $value) => strlen($value) < 2
        ? 'The name must be at least 2 characters.'
        : null
);

$role = select(
    label: 'Which role?',
    options: ['admin', 'editor', 'viewer'],
    default: 'viewer'
);
```

**Syntax Rules:**

- Prompts respect the `--no-interaction` (`-n`) flag. When set, prompts return their defaults or throw an exception.
- The `Laravel\Prompts` functions are imported individually with `use function`.
- Validation callbacks return `null` for valid input or a string error message for invalid input.
- The `select()` and `multiselect()` functions render arrow-key navigable menus in supported terminals.
- The `search()` function accepts a callable that returns the filtered options based on the user's query.

**Constraints and Limitations:**

- **Prompts block execution** until the user provides input. In non-interactive contexts (CI/CD, cron jobs), prompts may hang or fail.
- **`--no-interaction` disables prompts.** Commands must handle the non-interactive case, typically by using defaults.
- **`Laravel\Prompts` requires a compatible terminal.** Older terminals may not support advanced features like arrow-key selection.
- **Prompts are not testable in the same way as other command behaviour.** Laravel's testing helpers provide `expectsQuestion()` and `expectsConfirmation()` for simulating prompt responses.

### Annotated Code Examples

**Example 1: Interactive User Creation Command**

```php
<?php
// File: app/Console/Commands/CreateUser.php

namespace App\Console\Commands;

use App\Models\User;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Hash;
use function Laravel\Prompts\text;
use function Laravel\Prompts\password;
use function Laravel\Prompts\select;
use function Laravel\Prompts\confirm;
use function Laravel\Prompts\multiselect;

class CreateUser extends Command
{
    protected $signature = 'users:create
                            {--name= : The user\'s name}
                            {--email= : The user\'s email}
                            {--role= : The user\'s role}';

    protected $description = 'Create a new user interactively';

    public function handle(): int
    {
        // Step 1: Prompt for name if not provided via option
        $name = $this->option('name') ?: text(
            label: 'What is the user\'s name?',
            placeholder: 'e.g., Jane Doe',
            required: true,
            validate: fn (string $value) => strlen($value) < 2
                ? 'The name must be at least 2 characters.'
                : null
        );

        // Step 2: Prompt for email if not provided
        $email = $this->option('email') ?: text(
            label: 'What is the user\'s email?',
            required: true,
            validate: fn (string $value) => filter_var($value, FILTER_VALIDATE_EMAIL)
                ? null
                : 'Please enter a valid email address.'
        );

        // Step 3: Prompt for password (hidden)
        $password = password(
            label: 'What is the user\'s password?',
            required: true,
            validate: fn (string $value) => strlen($value) < 8
                ? 'The password must be at least 8 characters.'
                : null
        );

        // Step 4: Prompt for role with a selection menu
        $role = $this->option('role') ?: select(
            label: 'What role should the user have?',
            options: ['admin', 'editor', 'viewer'],
            default: 'viewer'
        );

        // Step 5: Prompt for permissions (multi-select)
        $permissions = multiselect(
            label: 'Which permissions should the user have?',
            options: [
                'create-posts',
                'edit-posts',
                'delete-posts',
                'manage-users',
                'view-analytics',
            ],
            default: ['create-posts', 'edit-posts']
        );

        // Step 6: Confirm before creating
        $confirmed = confirm(
            label: "Create user {$name} ({$email}) with role {$role}?",
            default: true
        );

        if (!$confirmed) {
            $this->warn('User creation cancelled.');
            return Command::FAILURE;
        }

        // Step 7: Create the user
        $user = User::create([
            'name'        => $name,
            'email'       => $email,
            'password'    => Hash::make($password),
            'role'        => $role,
            'permissions' => $permissions,
        ]);

        $this->info("User {$user->name} created successfully (ID: {$user->id}).");

        return Command::SUCCESS;
    }
}
```

```bash
# Step 1: Run interactively
php artisan users:create
```

**Expected Interactive Output:**

```
 What is the user's name? ────────────────────────────
 › Jane Doe

 What is the user's email? ───────────────────────────
 › jane@example.com

 What is the user's password? ────────────────────────
 › ••••••••••

 What role should the user have? ─────────────────────
 › viewer
    admin
    editor
  ▸ viewer

 Which permissions should the user have? ─────────────
 ─────────────────────────────────────────────────────
 ◉ create-posts
 ◉ edit-posts
 ◯ delete-posts
 ◯ manage-users
 ◯ view-analytics

 Create user Jane Doe (jane@example.com) with role viewer? (yes/no) [yes]:
 › yes

User Jane Doe created successfully (ID: 42).
```

**Why This Output Occurs:** The `Laravel\Prompts` functions render interactive UI elements in the terminal. `text()` displays a text input with validation. `password()` hides input. `select()` displays an arrow-key navigable menu. `multiselect()` allows toggling multiple options with the spacebar. `confirm()` displays a yes/no prompt. Validation callbacks provide immediate feedback.

---

**Example 2: Legacy Symfony Console Interaction**

```php
<?php
// File: app/Console/Commands/LegacyInteractive.php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class LegacyInteractive extends Command
{
    protected $signature = 'legacy:interactive';
    protected $description = 'Demonstrate legacy Symfony Console interaction';

    public function handle(): int
    {
        // Text input with default
        $name = $this->ask('What is your name?', 'Guest');

        // Secret input (hidden)
        $password = $this->secret('What is the password?');

        // Confirmation
        if (!$this->confirm('Do you want to continue?', true)) {
            $this->warn('Aborted.');
            return Command::FAILURE;
        }

        // Single choice
        $role = $this->choice(
            'Which role?',
            ['admin', 'editor', 'viewer'],
            'viewer'
        );

        // Multiple choice
        $permissions = $this->choice(
            'Which permissions?',
            ['read', 'write', 'delete'],
            null,
            null,
            true // multiple
        );

        // Anticipate (autocomplete)
        $model = $this->anticipate(
            'Which model?',
            ['User', 'Post', 'Order', 'Product']
        );

        $this->info("Name: {$name}");
        $this->info("Role: {$role}");
        $this->info("Permissions: " . implode(', ', (array) $permissions));
        $this->info("Model: {$model}");

        return Command::SUCCESS;
    }
}
```

```bash
php artisan legacy:interactive
```

**Expected Interactive Output:**

```
 What is your name? [Guest]:
 > Jane

 What is the password?:
 > 

 Do you want to continue? (yes/no) [yes]:
 > yes

 Which role? [viewer]:
  [0] admin
  [1] editor
  [2] viewer
 > 1

 Which permissions?:
  [0] read
  [1] write
  [2] delete
 > 0,1

 Which model?:
 > User

Name: Jane
Role: editor
Permissions: read, write
Model: User
```

**Why This Output Occurs:** The `ask()` method prompts for text input with a default. `secret()` hides the input. `confirm()` prompts for yes/no. `choice()` displays a numbered list and accepts the index (or a comma-separated list of indices for multiple choice). `anticipate()` provides autocomplete suggestions as the user types.

### Real-World Cases

- **User management commands:** Interactive prompts for name, email, password, and role when creating users.
- **Deployment commands:** Confirmation prompts before destructive operations like `migrate:fresh`.
- **Configuration wizards:** Step-by-step prompts for setting up API keys, database credentials, and other configuration.
- **Data import:** Prompts for file paths, date ranges, and import options.
- **Package installation:** Prompts for configuration values during package setup.

---

## 5. Progress Indicators

### Definitions

**Core Definition:** Progress indicators are visual elements — progress bars and spinners — that display the status of a long-running operation, providing feedback to the user about how much work has been completed.

**Technical Definition:** Laravel provides two progress indicator mechanisms: Symfony's `ProgressBar` helper (accessed via `$this->output->createProgressBar($max)`), which renders a horizontal bar that fills as iterations complete, and the `Laravel\Prompts\spin()` function, which displays an animated spinner while a callback executes. The `ProgressBar` supports multiple formats, custom messages, and advanced features like `setMessage()`, `setFormat()`, and `advance()`. The `spin()` function runs a closure and displays a spinner until it completes, returning the closure's result.

**Beginner-Friendly Explanation:** When a command processes hundreds or thousands of items, it's helpful to see how far along it is. A progress bar fills up as the work completes, showing a percentage and estimated time remaining. A spinner is a small rotating animation that indicates "something is happening" when the exact progress isn't known. Both are easy to add to your commands.

### Purposes

- To provide visual feedback during long-running operations.
- To display the percentage of work completed and estimated time remaining.
- To indicate ongoing activity when progress cannot be measured precisely.
- To improve the user experience of commands that process large datasets.
- To allow users to estimate how long a command will take.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Progress bar (Symfony)
$bar = $this->output->createProgressBar(int $max = 0);
$bar->start();
$bar->advance(int $step = 1);
$bar->setMessage(string $message);
$bar->finish();
$bar->clear();

// Spinner (Laravel\Prompts)
use function Laravel\Prompts\spin;

$result = spin(
    callback: fn () => $this->performLongOperation(),
    message: 'Processing...'
);
```

**Component Breakdown:**

- `createProgressBar($max)` — Creates a progress bar with the specified maximum value (number of steps).
- `start()` — Displays the initial progress bar.
- `advance($step)` — Advances the bar by the specified number of steps (default 1).
- `setMessage($message)` — Sets a custom message displayed alongside the bar.
- `finish()` — Completes the bar and moves to the next line.
- `clear()` — Removes the bar from the terminal.
- `spin($callback, $message)` — Displays a spinner while the callback executes and returns its result.

```php
// Example: Progress bar for processing records
$records = Record::all();
$bar = $this->output->createProgressBar($records->count());
$bar->start();

foreach ($records as $record) {
    $this->processRecord($record);
    $bar->advance();
}

$bar->finish();
$this->newLine();

// Example: Spinner for an indeterminate operation
$result = spin(
    callback: fn () => $this->fetchRemoteData(),
    message: 'Fetching remote data...'
);
```

**Syntax Rules:**

- The `createProgressBar()` method accepts an integer representing the total number of steps.
- The `advance()` method is called once per iteration (or with a step value for batch operations).
- The `finish()` method must be called to complete the bar and prevent overlapping output.
- The `spin()` function blocks until the callback completes and returns the callback's result.
- Progress bars automatically respect the `-q` (quiet) flag and produce no output in quiet mode.

**Constraints and Limitations:**

- **Progress bars require a known maximum.** For operations with unknown totals, use a spinner instead.
- **The `spin()` function cannot display progress percentage.** It only shows that work is ongoing.
- **Progress bar output may interfere with other output.** Call `finish()` before displaying other messages.
- **In non-interactive terminals, progress bars may render incorrectly.** Consider checking `$this->output->isDecorated()` before using them.

### Annotated Code Examples

**Example 1: Progress Bar for Batch Processing**

```php
<?php
// File: app/Console/Commands/ProcessOrders.php

namespace App\Console\Commands;

use App\Models\Order;
use Illuminate\Console\Command;

class ProcessOrders extends Command
{
    protected $signature = 'orders:process {--chunk=100}';
    protected $description = 'Process pending orders in batches';

    public function handle(): int
    {
        $total = Order::where('status', 'pending')->count();

        if ($total === 0) {
            $this->info('No pending orders to process.');
            return Command::SUCCESS;
        }

        $this->info("Processing {$total} pending orders...");

        // Step 1: Create a progress bar with the total count
        $bar = $this->output->createProgressBar($total);
        $bar->start();

        // Step 2: Process orders in chunks
        Order::where('status', 'pending')
            ->chunkById((int) $this->option('chunk'), function ($orders) use ($bar) {
                foreach ($orders as $order) {
                    // Simulate processing
                    $order->update(['status' => 'processed']);
                    $bar->advance();
                }
            });

        // Step 3: Complete the bar
        $bar->finish();
        $this->newLine();
        $this->info("Processed {$total} orders successfully.");

        return Command::SUCCESS;
    }
}
```

```bash
php artisan orders:process --chunk=100
```

**Expected Output:**

```
Processing 1,234 pending orders...
 1234/1234 [============================] 100%
Processed 1234 orders successfully.
```

**Why This Output Occurs:** The `createProgressBar($total)` method creates a bar with `$total` steps. Each call to `advance()` increments the bar by one step. The bar displays the current step, total steps, and a visual representation. When `finish()` is called, the bar is completed and a newline is inserted before the success message.

---

**Example 2: Spinner for Indeterminate Operations**

```php
<?php
// File: app/Console/Commands/FetchRemoteData.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;
use function Laravel\Prompts\spin;

class FetchRemoteData extends Command
{
    protected $signature = 'remote:fetch {url}';
    protected $description = 'Fetch data from a remote URL with a spinner';

    public function handle(): int
    {
        $url = $this->argument('url');

        // Step 1: Display a spinner while fetching
        $response = spin(
            callback: fn () => Http::timeout(30)->get($url),
            message: "Fetching {$url}..."
        );

        // Step 2: Check the response
        if (!$response->successful()) {
            $this->error("Failed to fetch: HTTP {$response->status()}");
            return Command::FAILURE;
        }

        // Step 3: Process the response
        $data = $response->json();
        $this->info('Data fetched successfully.');
        $this->line('Records: ' . count($data['records'] ?? []));

        return Command::SUCCESS;
    }
}
```

```bash
php artisan remote:fetch https://api.example.com/data
```

**Expected Output:**

```
 ⠿ Fetching https://api.example.com/data...
Data fetched successfully.
Records: 42
```

**Why This Output Occurs:** The `spin()` function displays an animated spinner (using Braille characters) while the callback executes. When the callback returns, the spinner is replaced with a success indicator (or the output continues). The spinner provides visual feedback that the operation is running, even though the exact progress is unknown.

### Real-World Cases

- **Data migration:** A progress bar shows how many records have been migrated out of the total.
- **File processing:** A progress bar displays the number of files processed during batch image resizing or compression.
- **API data fetching:** A spinner indicates that remote data is being fetched, especially when the response time is unpredictable.
- **Database seeding:** A progress bar shows the number of seeders that have run.
- **Report generation:** A spinner indicates that a complex report is being generated.

---

## 6. Validation

### Definitions

**Core Definition:** Command validation is the process of verifying that user-provided arguments and options meet the command's requirements before the command's logic executes.

**Technical Definition:** Laravel commands can validate input through three mechanisms: (1) the `$signature` definition, which enforces required arguments and option types at the parser level; (2) the `Laravel\Prompts` validation callbacks, which validate interactive input; and (3) manual validation using Laravel's `Validator` facade or the `$this->validate()` method (available on commands that use the `ValidatesInput` trait or call the validator directly). The `Validator` facade provides access to all of Laravel's validation rules, including `required`, `integer`, `email`, `in`, `exists`, and custom rules.

**Beginner-Friendly Explanation:** Before your command does its work, it should check that the input makes sense. Is the email address valid? Does the file exist? Is the number within the allowed range? Laravel's validation system can check all of this, and it can also provide helpful error messages if something is wrong.

### Purposes

- To ensure that required arguments are provided and optional arguments have valid values.
- To verify that option values match expected formats (integers, emails, existing records).
- To provide clear, actionable error messages when validation fails.
- To prevent commands from executing with invalid input and causing errors downstream.
- To validate interactive prompt input before accepting it.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Using the Validator facade
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, $rules);

if ($validator->fails()) {
    foreach ($validator->errors()->all() as $error) {
        $this->error($error);
    }
    return Command::FAILURE;
}

// Using Laravel\Prompts validation callbacks
$email = text(
    label: 'What is the email?',
    validate: fn (string $value) => filter_var($value, FILTER_VALIDATE_EMAIL)
        ? null
        : 'Please enter a valid email address.'
);

// Manual validation
$id = $this->argument('id');
if (!is_numeric($id) || (int) $id < 1) {
    $this->error('The ID must be a positive integer.');
    return Command::FAILURE;
}
```

**Component Breakdown:**

- `Validator::make($data, $rules)` — Creates a validator instance with the given data and rules.
- `$validator->fails()` — Returns `true` if validation failed.
- `$validator->errors()->all()` — Returns an array of all error messages.
- `validate: fn ($value) => ...` — A callback that returns `null` for valid input or an error string for invalid input.
- Manual validation — Direct checks on argument/option values.

```php
// Example: Validating command input
$data = [
    'email' => $this->argument('email'),
    'role'  => $this->option('role'),
    'age'   => $this->option('age'),
];

$validator = Validator::make($data, [
    'email' => 'required|email|unique:users,email',
    'role'  => 'required|in:admin,editor,viewer',
    'age'   => 'nullable|integer|min:18|max:120',
]);

if ($validator->fails()) {
    foreach ($validator->errors()->all() as $error) {
        $this->error($error);
    }
    return Command::FAILURE;
}
```

**Syntax Rules:**

- The `Validator` facade provides access to all Laravel validation rules.
- Validation errors should be displayed using `$this->error()` and the command should return `Command::FAILURE`.
- The `Laravel\Prompts` validation callbacks return `null` for valid input or a string error message.
- Manual validation is appropriate for simple checks that don't warrant the full validator.
- Validation should occur at the beginning of the `handle()` method, before any processing.

**Constraints and Limitations:**

- **The `$signature` parser validates argument/option presence and type, not values.** A required argument being present doesn't mean its value is valid.
- **Interactive validation callbacks only run in interactive mode.** In non-interactive mode (`--no-interaction`), prompts return defaults without validation.
- **The `Validator` facade requires the database for `unique` and `exists` rules.** Ensure the database connection is available.
- **Validation errors should not be thrown as exceptions** unless the command is being called programmatically. Display them and return `Command::FAILURE`.

### Annotated Code Examples

**Example 1: Validating Command Arguments and Options**

```php
<?php
// File: app/Console/Commands/ImportUsers.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Validator;

class ImportUsers extends Command
{
    protected $signature = 'users:import
                            {file : The CSV file to import}
                            {--delimiter=, : The CSV delimiter}
                            {--role=viewer : The default role for imported users}';

    protected $description = 'Import users from a CSV file';

    public function handle(): int
    {
        // Step 1: Gather input data
        $data = [
            'file'      => $this->argument('file'),
            'delimiter' => $this->option('delimiter'),
            'role'      => $this->option('role'),
        ];

        // Step 2: Define validation rules
        $rules = [
            'file'      => 'required|string|file|mimes:csv,txt',
            'delimiter' => 'required|string|size:1',
            'role'      => 'required|in:admin,editor,viewer',
        ];

        // Step 3: Validate
        $validator = Validator::make($data, $rules);

        if ($validator->fails()) {
            $this->error('Validation failed:');
            foreach ($validator->errors()->all() as $error) {
                $this->line("  • {$error}");
            }
            return Command::FAILURE;
        }

        // Step 4: Additional manual validation
        $filePath = $data['file'];
        if (!file_exists($filePath)) {
            $this->error("File not found: {$filePath}");
            return Command::FAILURE;
        }

        // Step 5: Proceed with import
        $this->info("Importing {$filePath} with role {$data['role']}...");

        // Import logic here...

        return Command::SUCCESS;
    }
}
```

```bash
# Step 1: Run with valid input
php artisan users:import users.csv --role=editor
# Output:
# Importing users.csv with role editor...

# Step 2: Run with invalid role
php artisan users:import users.csv --role=superadmin
# Output:
# Validation failed:
#   • The selected role is invalid.

# Step 3: Run with non-existent file
php artisan users:import missing.csv
# Output:
# Validation failed:
#   • The file must be a file of type: csv, txt.
```

**Expected Output:** The command validates the file, delimiter, and role. If validation fails, it displays all errors and returns `Command::FAILURE`. If validation passes, it proceeds with the import.

**Why This Output Occurs:** The `Validator::make()` method applies the rules to the input data. The `file` rule checks that the file exists and is uploaded (or a valid path). The `mimes:csv,txt` rule checks the file extension. The `in:admin,editor,viewer` rule checks that the role is one of the allowed values. Errors are collected and displayed.

---

**Example 2: Interactive Validation with Laravel\Prompts**

```php
<?php
// File: app/Console/Commands/ConfigureApi.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use function Laravel\Prompts\text;
use function Laravel\Prompts\password;
use function Laravel\Prompts\confirm;

class ConfigureApi extends Command
{
    protected $signature = 'api:configure';
    protected $description = 'Configure API credentials interactively';

    public function handle(): int
    {
        // Step 1: Prompt for API endpoint with URL validation
        $endpoint = text(
            label: 'What is the API endpoint?',
            placeholder: 'https://api.example.com',
            required: true,
            validate: fn (string $value) => filter_var($value, FILTER_VALIDATE_URL)
                ? null
                : 'Please enter a valid URL.'
        );

        // Step 2: Prompt for API key with length validation
        $apiKey = text(
            label: 'What is the API key?',
            required: true,
            validate: fn (string $value) => strlen($value) < 32
                ? 'The API key must be at least 32 characters.'
                : null
        );

        // Step 3: Prompt for API secret (hidden)
        $apiSecret = password(
            label: 'What is the API secret?',
            required: true,
            validate: fn (string $value) => strlen($value) < 16
                ? 'The API secret must be at least 16 characters.'
                : null
        );

        // Step 4: Confirm the configuration
        $confirmed = confirm(
            label: "Save configuration for {$endpoint}?",
            default: true
        );

        if (!$confirmed) {
            $this->warn('Configuration cancelled.');
            return Command::FAILURE;
        }

        // Step 5: Save the configuration
        $this->info('API configuration saved.');
        $this->line("Endpoint: {$endpoint}");
        $this->line("API Key: " . substr($apiKey, 0, 8) . '...');
        $this->line("API Secret: " . str_repeat('•', 16));

        return Command::SUCCESS;
    }
}
```

```bash
php artisan api:configure
```

**Expected Interactive Output:**

```
 What is the API endpoint? ────────────────────────────
 › https://api.example.com

 What is the API key? ─────────────────────────────────
 › abcdefghijklmnopqrstuvwxyz123456

 What is the API secret? ──────────────────────────────
 › ••••••••••••••••

 Save configuration for https://api.example.com? (yes/no) [yes]:
 › yes

API configuration saved.
Endpoint: https://api.example.com
API Key: abcdefgh...
API Secret: ••••••••••••••••
```

**Why This Output Occurs:** The `text()` and `password()` functions accept a `validate` callback that runs when the user submits input. If the callback returns a string (error message), the prompt redisplays with the error. If it returns `null`, the input is accepted. The `confirm()` function provides a final confirmation before saving.

### Real-World Cases

- **Data import commands:** Validate that the file exists, is readable, and has the expected format before importing.
- **User creation commands:** Validate email format, password strength, and role selection.
- **Deployment commands:** Validate that the environment is correct and required configuration is present before deploying.
- **API configuration commands:** Validate URLs, API keys, and secrets before saving them.
- **Database commands:** Validate that the connection is available and the target table exists before running migrations.

---

## 7. Isolation & Locks

### Definitions

**Core Definition:** Isolation and locks are mechanisms that prevent a command from running concurrently with itself, ensuring that overlapping executions do not cause data corruption or duplicate work.

**Technical Definition:** Laravel provides the `Illuminate\Contracts\Console\Isolatable` interface, which commands can implement to indicate that only one instance should run at a time. When a command implements `Isolatable`, Laravel wraps its execution in an atomic lock using the application's cache driver (`Cache::lock()`). The `--isolated` option can be passed to any command to enforce isolation even if the command does not implement the interface. The `WithoutOverlapping` middleware provides finer control when scheduling commands, allowing a specified number of minutes before the lock expires.

**Beginner-Friendly Explanation:** If a command takes a long time to run, you might accidentally start a second copy before the first one finishes. This can cause duplicate processing, data corruption, or resource exhaustion. Isolation prevents this by using a "lock" — the first instance acquires the lock, and any subsequent instances wait or fail immediately. This is especially important for scheduled commands that run frequently.

### Purposes

- To prevent concurrent execution of commands that modify shared state.
- To avoid duplicate processing of the same data by overlapping command instances.
- To ensure that scheduled commands do not stack up when the previous execution is still running.
- To provide a safe mechanism for long-running commands that should not be interrupted.
- To control the lock timeout for commands that may run longer than expected.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Implementing the Isolatable interface
use Illuminate\Contracts\Console\Isolatable;

class ProcessOrders extends Command implements Isolatable
{
    // ...
}

// Using the --isolated option on any command
php artisan process:orders --isolated

// Scheduling with WithoutOverlapping middleware
$schedule->command('process:orders')
         ->everyMinute()
         ->withoutOverlapping(10); // Lock expires after 10 minutes
```

**Component Breakdown:**

- `Isolatable` — An interface that marks a command as requiring isolation.
- `--isolated` — A CLI option that enforces isolation for a single invocation.
- `withoutOverlapping($expiresAt)` — A scheduling method that prevents overlapping scheduled executions, with an optional lock expiration in minutes.
- `onOneServer()` — A scheduling method that ensures the command runs on only one server in a multi-server deployment.

```php
// Example: Isolatable command
use Illuminate\Contracts\Console\Isolatable;

class ImportData extends Command implements Isolatable
{
    protected $signature = 'data:import {file}';
    protected $description = 'Import data from a file';

    public function handle(): int
    {
        // This command will not run if another instance is already running.
        $this->info('Importing data...');
        // ...
        return Command::SUCCESS;
    }
}
```

```bash
# Run the command with isolation enforced
php artisan data:import users.csv

# If another instance is running, the command fails immediately
php artisan data:import users.csv
# Output: The command is already running.
```

**Syntax Rules:**

- Implementing `Isolatable` automatically applies isolation to all invocations of the command.
- The `--isolated` option can be used on any command, even if it does not implement `Isolatable`.
- Isolation uses the application's default cache driver to store the lock.
- The lock is released when the command completes, whether successfully or with an error.
- Scheduled commands should use `withoutOverlapping()` to prevent stacking.

**Constraints and Limitations:**

- **Isolation requires a cache driver that supports atomic locks.** The `file` and `database` drivers support locks; the `array` driver does not (it's per-process only).
- **The lock is not released if the process is killed with SIGKILL.** The lock will remain until the cache entry expires (typically 24 hours by default). Configure a shorter expiry if needed.
- **Isolation does not queue commands.** If a second instance is attempted, it fails immediately rather than waiting.
- **`withoutOverlapping()` requires the scheduler to be running.** It does not affect manual invocations of the command.

### Annotated Code Examples

**Example 1: Isolatable Command for Data Import**

```php
<?php
// File: app/Console/Commands/ImportLargeDataset.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;

class ImportLargeDataset extends Command implements Isolatable
{
    protected $signature = 'data:import-large {file : The file to import}';
    protected $description = 'Import a large dataset (prevents concurrent execution)';

    public function handle(): int
    {
        $file = $this->argument('file');

        $this->info("Starting import of {$file}...");
        $this->comment('This command is isolated — no other instance can run concurrently.');

        // Simulate a long-running import
        $total = 1000;
        $bar = $this->output->createProgressBar($total);
        $bar->start();

        for ($i = 0; $i < $total; $i++) {
            usleep(1000); // Simulate work
            $bar->advance();
        }

        $bar->finish();
        $this->newLine();
        $this->info("Import of {$file} completed.");

        return Command::SUCCESS;
    }
}
```

```bash
# Terminal 1: Start the import
php artisan data:import-large dataset.csv

# Terminal 2: Attempt to start another import while the first is running
php artisan data:import-large dataset2.csv
# Output:
# The command is already running.
```

**Expected Output (Terminal 1):**

```
Starting import of dataset.csv...
This command is isolated — no other instance can run concurrently.
 1000/1000 [============================] 100%
Import of dataset.csv completed.
```

**Expected Output (Terminal 2):**

```
The command is already running.
```

**Why This Output Occurs:** The `Isolatable` interface tells Laravel to acquire a cache lock before executing the command. The first instance acquires the lock and runs. The second instance attempts to acquire the same lock, fails, and exits with a message. When the first instance completes, it releases the lock, allowing subsequent instances to run.

---

**Example 2: Scheduled Command with WithoutOverlapping**

```php
<?php
// File: app/Console/Kernel.php (Laravel 10 and below)

namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    protected function schedule(Schedule $schedule): void
    {
        // Run every minute, but skip if the previous run is still going
        $schedule->command('process:orders')
                 ->everyMinute()
                 ->withoutOverlapping(10)  // Lock expires after 10 minutes
                 ->onOneServer();           // Only run on one server
    }
}
```

```php
// File: routes/console.php (Laravel 11+)
use Illuminate\Support\Facades\Schedule;

Schedule::command('process:orders')
    ->everyMinute()
    ->withoutOverlapping(10)
    ->onOneServer();
```

**Expected Output (scheduler log):**

```
Running [process:orders] ....................... 0.5s DONE
Running [process:orders] ....................... 0.3s DONE
Skipping [process:orders] ...................... (locked)
Running [process:orders] ....................... 0.4s DONE
```

**Why This Output Occurs:** The `withoutOverlapping(10)` middleware creates a cache lock before executing the command and releases it when the command completes. If the previous run is still holding the lock (because it hasn't finished), the next scheduled execution is skipped. The `10` argument specifies that the lock should expire after 10 minutes, preventing deadlocks if the command crashes without releasing the lock. The `onOneServer()` middleware ensures that in a multi-server deployment, only one server runs the command.

### Real-World Cases

- **Scheduled data processing:** A `process:orders` command runs every minute; `withoutOverlapping()` prevents stacking if the previous run is still processing.
- **Data import commands:** An `data:import-large` command is marked `Isolatable` to prevent two imports from corrupting the same database.
- **Report generation:** A `reports:generate` command is isolated to prevent duplicate reports from being generated and emailed.
- **Cache warming:** A `cache:warm` command uses `withoutOverlapping()` to prevent multiple workers from warming the same cache simultaneously.
- **Multi-server deployments:** Commands scheduled with `onOneServer()` ensure that only one server in a load-balanced cluster runs the task.

---

## References

- Laravel Artisan Console Documentation (Master) — https://laravel.com/docs/master/artisan 
- Laravel Artisan Console Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/artisan 
- Laravel Prompts Documentation — https://laravel.com/docs/11.x/prompts 
- Laravel Scheduling Documentation — https://laravel.com/docs/master/scheduling 
- Laravel `Illuminate\Console\Command` API — https://api.laravel.com/docs/11.x/Illuminate/Console/Command.html 
- Laravel `Illuminate\Console\Parser` API — https://api.laravel.com/docs/11.x/Illuminate/Console/Parser.html 
- Laravel `Illuminate\Contracts\Console\Isolatable` API — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Console/Isolatable.html 
- Laravel `Illuminate\Console\Scheduling\WithoutOverlapping` API — https://api.laravel.com/docs/12.x/Illuminate/Console/Scheduling/WithoutOverlapping.html 
- Symfony Console Documentation — https://symfony.com/doc/current/console.html 
- Symfony Console Progress Bar — https://symfony.com/doc/current/console/helpers/progressbar.html 
- Laravel Prompts Package (GitHub) — https://github.com/laravel/prompts 
- Laravel `make:command` Documentation — https://laravel.com/docs/master/artisan#generating-commands 
- Laravel Command Isolation (Laravel News) — https://laravel-news.com/laravel-command-isolation 
- Laravel `--isolated` Option (Laravel Blog) — https://laravel.com/blog/laravel-isolated-commands 
- Laravel `withoutOverlapping` Middleware (Laravel Daily) — https://laraveldaily.com/post/laravel-scheduler-withoutoverlapping 
- Laravel Validation Documentation — https://laravel.com/docs/master/validation 
- Laravel `Laravel\Prompts\spin` (Laravel News) — https://laravel-news.com/laravel-prompts-spin