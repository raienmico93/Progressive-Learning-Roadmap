# Laravel Migration Fundamentals: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel migrations are a version-control system for your database schema, allowing development teams to define, share, and modify the application's database structure through PHP files that are executed in a deterministic order.

**Technical Definition:** Migrations are PHP classes stored in the `database/migrations` directory, each containing `up()` and `down()` methods that use the `Illuminate\Database\Schema\Builder` (accessed via the `Schema` facade) to create, modify, or drop database tables, columns, and indexes. Laravel tracks which migrations have been executed in a special `migrations` table, recording the migration filename and a batch number that determines rollback order. The migration system uses timestamped filenames (e.g., `2025_01_15_000000_create_users_table.php`) to ensure deterministic execution ordering across all environments.

**Beginner-Friendly Explanation:** Imagine you're working on a team project where everyone has their own copy of the database. When one developer adds a new column, everyone else needs to add it too—otherwise their applications break. Laravel migrations solve this by letting you write the database change once in a PHP file. When your teammates pull your code and run a single command, their databases are updated automatically. It's like Git for your database structure.

### Key Characteristics

- **Version-Controlled Schema:** Migration files are committed to version control alongside application code, ensuring every environment has a consistent database structure.
- **Deterministic Ordering:** Timestamped filenames guarantee migrations execute in the same order across all environments.
- **Reversible Operations:** The `down()` method allows migrations to be rolled back, enabling safe schema experimentation and deployment rollbacks.
- **Anonymous Classes:** Modern Laravel migrations use anonymous classes (`return new class extends Migration`) to prevent class name collisions.
- **Batch Tracking:** The `migrations` table records which migrations have run and in which batch, enabling precise rollback control.
- **Database Agnostic:** The Schema Builder generates database-specific SQL for MySQL, PostgreSQL, SQLite, SQL Server, and MariaDB.
- **Isolation Support:** The `--isolated` flag prevents concurrent migration execution in multi-server deployments.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (the migration API is stable across versions; anonymous migrations since Laravel 8.37; `--isolated` since Laravel 9.38).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12) with the appropriate PDO extension.
- A configured database connection in `config/database.php` and `.env`.
- Basic understanding of SQL DDL (Data Definition Language) and database schema concepts.

### Related Programming Areas

- **Schema Builder:** The `Schema` facade provides the fluent API for defining tables, columns, and indexes within migrations.
- **Eloquent ORM:** Eloquent models depend on the database schema defined by migrations.
- **Database Seeding:** Seeders populate the database with test or reference data after migrations run.
- **Deployment & DevOps:** Migrations are a standard step in production deployment pipelines.
- **Testing:** Test databases are built from migrations to ensure schema consistency.

### Core Concepts / Features

1. Core Concepts (Version Control for Databases, Migrations Table)
2. Structure (up(), down(), Anonymous Migrations)
3. Execution Workflow (migrate, migrate:status, --isolated)

---

## 1. Core Concepts

### Definitions

**Core Definition:** The core concepts of Laravel migrations are the principles of database version control and the metadata tracking mechanism that records which migrations have been executed.

**Technical Definition:** The migration system operates on two fundamental pillars: (1) **Version control for databases** — migration files are stored in `database/migrations` with timestamped filenames that define execution order, and each file contains the schema changes for a single logical unit of work. (2) **The migrations table** — a database table named `migrations` (created automatically on the first `migrate` run) containing columns `id`, `migration` (the filename without `.php`), and `batch` (an integer grouping migrations executed in the same run). The batch number is critical for rollback: `migrate:rollback` reverses migrations from the most recent batch first .

**Beginner-Friendly Explanation:** Think of migrations as chapters in a book about your database. Each chapter describes one change—like "add a users table" or "add an email column." The `migrations` table is the table of contents that tells Laravel which chapters have already been read. When you run `migrate`, Laravel reads all chapters that haven't been read yet. When you run `migrate:rollback`, Laravel un-reads the most recent batch of chapters.

### Purposes

- To provide version control for database schema, allowing teams to define and share schema changes through code.
- To ensure every development, staging, and production environment has an identical database structure.
- To enable deterministic, repeatable database setup from scratch.
- To track which migrations have been executed via the `migrations` table.
- To enable safe rollback of schema changes through batch-based tracking.
- To eliminate manual, error-prone database modifications.

### Syntax Rules and Structure

**Complete General Syntax — Migrations Table Structure:**

```sql
-- The migrations table is created automatically by Laravel
CREATE TABLE migrations (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    migration VARCHAR(255) NOT NULL,
    batch INT NOT NULL
);
```

**Component Breakdown:**

| Column | Description |
|--------|-------------|
| `id` | Auto-incrementing primary key. |
| `migration` | The migration filename without the `.php` extension. |
| `batch` | The batch number, incremented each time `migrate` is run. |

**Complete General Syntax — Migration Filename Convention:**

```
database/migrations/
├── 0001_01_01_000000_create_users_table.php
├── 0001_01_01_000001_create_cache_table.php
├── 2025_01_15_000000_create_posts_table.php
└── 2025_02_20_000000_add_slug_to_posts_table.php
```

**Syntax Rules:**

- The `migrations` table is created automatically when `php artisan migrate` is run for the first time.
- The `migration` column stores the filename without the `.php` extension.
- The `batch` column is incremented for each `migrate` run; migrations executed in the same run share the same batch number .
- Migration filenames use the format `YYYY_MM_DD_HHMMSS_description.php`.
- Laravel uses the timestamp prefix to determine execution order.
- Migration files are stored in `database/migrations` by default.

**Constraints and Limitations:**

- **Batch-based rollback:** `migrate:rollback` rolls back the entire last batch, not individual migrations. Use `migrate:rollback --step=1` to roll back a specific number of migrations.
- **Migration table not version-controlled:** The `migrations` table exists only in the database, not in version control. It is automatically maintained by Laravel.
- **Timestamp collisions:** If two migrations share the same timestamp prefix, Laravel resolves the order alphabetically by filename.

### Multiple Annotated Complete Code Examples

**Example 1: Generating a Migration**

```bash
php artisan make:migration create_flights_table
```

**Expected Output:** A new file is created at `database/migrations/2025_01_15_143000_create_flights_table.php`.

**Why:** The `make:migration` command generates a migration file with the current timestamp as a prefix. Laravel guesses the table name from the migration name (if it follows the `create_*_table` or `add_*_to_*_table` conventions) and pre-fills the generated file with the appropriate table name .

---

**Example 2: The Migrations Table After Running Migrations**

```bash
php artisan migrate
```

**Expected Output:** The `migrations` table contains:

| id | migration | batch |
|----|-----------|-------|
| 1 | 0001_01_01_000000_create_users_table | 1 |
| 2 | 0001_01_01_000001_create_cache_table | 1 |
| 3 | 2025_01_15_143000_create_flights_table | 1 |

**Why:** All migrations executed in the first `migrate` run share batch number 1. When new migrations are added and `migrate` is run again, they receive batch number 2 .

---

**Example 3: Batch-Based Rollback**

```bash
# Run a second batch of migrations
php artisan migrate

# The migrations table now contains:
# | 4 | 2025_02_20_100000_add_slug_to_posts_table | 2 |

# Roll back the last batch (batch 2 only)
php artisan migrate:rollback

# Only the migration with batch 2 is rolled back.
# The migrations from batch 1 remain intact.
```

**Expected Output:** Only migrations in the most recent batch are reversed. The `add_slug_to_posts_table` migration is rolled back, but the `create_users_table` and `create_flights_table` migrations remain .

**Why:** The `migrate:rollback` command uses the batch number to determine which migrations to reverse. It rolls back all migrations in the highest batch number .

### Real-World Cases

- **Team Collaboration:** Every developer runs `php artisan migrate` after pulling changes to sync their local database with the team's schema.
- **Deployment Pipelines:** `php artisan migrate --force` is run during deployment to apply pending schema changes to production.
- **Database Reset:** `php artisan migrate:fresh` drops all tables and re-runs all migrations, useful for local development and testing.
- **Migration Squashing:** `php artisan schema:dump --prune` compresses hundreds of migrations into a single SQL file for faster initial setup .

### References

- Laravel Database: Migrations — Introduction — https://laravel.com/docs/12.x/migrations#introduction
- Laravel Database: Migrations — Generating Migrations — https://laravel.com/docs/12.x/migrations#generating-migrations
- Laravel Database: Migrations — Squashing Migrations — https://laravel.com/docs/12.x/migrations#squashing-migrations
- Stack Overflow: Why does migrate create a migrations table? — https://stackoverflow.com/questions/27943744

---

## 2. Structure: up(), down(), and Anonymous Migrations

### Definitions

**Core Definition:** Every Laravel migration class contains two methods—`up()` for applying schema changes and `down()` for reversing them—and modern Laravel uses anonymous classes to prevent class name collisions.

**Technical Definition:** The `up()` method is executed when the migration is applied and contains the schema changes (creating tables, adding columns, creating indexes). The `down()` method is executed when the migration is rolled back and should reverse the operations performed by `up()`. Since Laravel 8.37, migrations use anonymous classes (`return new class extends Migration`) instead of named classes, eliminating class name collisions when multiple migrations share the same descriptive name . The anonymous class is assigned to no variable and is returned directly from the file, allowing Laravel to instantiate it without requiring a unique class name .

**Beginner-Friendly Explanation:** The `up()` method is what happens when you run the migration—it builds your table or adds your column. The `down()` method is the undo button—it removes the table or column if you need to roll back. Anonymous migrations are a modern convention where the migration class doesn't have a name, so two developers can both create a "create_users_table" migration without PHP complaining about duplicate class names.

### Purposes

- To define the schema changes to apply when running the migration via `up()`.
- To define the reversal operations when rolling back the migration via `down()`.
- To prevent class name collisions across migrations using anonymous classes.
- To follow Laravel 12+ conventions and modern PHP practices.
- To enable safe, reversible schema changes that can be rolled back in any environment.
- To support database-agnostic schema definitions via the `Schema` facade and `Blueprint` class.

### Syntax Rules and Structure

**Complete General Syntax — Anonymous Migration:**

```php
<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('flights');
    }
};
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `return new class extends Migration` | Anonymous class syntax; no class name required. |
| `public function up(): void` | Defines the schema changes to apply. |
| `Schema::create(...)` | Creates a new table. |
| `public function down(): void` | Defines the reversal operations. |
| `Schema::dropIfExists(...)` | Drops the table if it exists. |

**Complete General Syntax — Named Migration (Legacy):**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateFlightsTable extends Migration
{
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('flights');
    }
}
```

**Syntax Rules:**

- The `up()` method is executed by `php artisan migrate`.
- The `down()` method is executed by `php artisan migrate:rollback`.
- Anonymous migrations end with a semicolon after the closing brace (`};`).
- The `declare(strict_types=1);` declaration is recommended for modern PHP code .
- Return types should be declared as `: void` on both methods .
- The `Schema` facade provides database-agnostic table creation and modification.
- The `Blueprint` class provides the fluent column and index definition API.

**Constraints and Limitations:**

- **`down()` must be implemented:** If `down()` is empty or missing, the migration cannot be rolled back. Laravel will throw an error if you attempt to roll back a migration without a `down()` implementation.
- **Irreversible migrations:** Some schema changes (e.g., dropping a column) cannot be reversed without data loss. The `down()` method should restore the column but cannot restore the data.
- **Anonymous class limitation:** Anonymous migrations cannot be referenced by class name in code (e.g., for testing or manual instantiation).
- **Strict types:** Adding `declare(strict_types=1);` enforces stricter type checking—ensure all method signatures use correct types.

### Multiple Annotated Complete Code Examples

**Example 1: Creating a Table with up() and down()**

```php
<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations — create the flights table.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();                          // Auto-incrementing primary key
            $table->string('name');                // VARCHAR column
            $table->string('airline');             // VARCHAR column
            $table->string('airport');             // VARCHAR column
            $table->timestamps();                  // created_at and updated_at
        });
    }

    /**
     * Reverse the migrations — drop the flights table.
     */
    public function down(): void
    {
        Schema::dropIfExists('flights');
    }
};
```

**Expected Output:** Running `php artisan migrate` creates the `flights` table with `id`, `name`, `airline`, `airport`, `created_at`, and `updated_at` columns. Running `php artisan migrate:rollback` drops the table.

**Why:** The `up()` method defines the table structure using `Schema::create()`. The `down()` method uses `Schema::dropIfExists()` to safely remove the table if the migration is rolled back .

---

**Example 2: Adding a Column with up() and down()**

```php
<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Add a slug column to the posts table.
     */
    public function up(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->string('slug')->unique()->after('title');
        });
    }

    /**
     * Remove the slug column from the posts table.
     */
    public function down(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->dropColumn('slug');
        });
    }
};
```

**Expected Output:** Running `php artisan migrate` adds the `slug` column after the `title` column in the `posts` table. Rolling back removes the column.

**Why:** The `up()` method uses `Schema::table()` to modify an existing table. The `down()` method reverses the operation by dropping the column. Note that the data in the `slug` column is lost when rolling back .

---

**Example 3: Creating a Table with Foreign Key Constraints**

```php
<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->foreignId('post_id')->constrained()->cascadeOnDelete();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->text('body');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('comments');
    }
};
```

**Expected Output:** The `comments` table is created with foreign key constraints referencing the `posts` and `users` tables. Deleting a post or user cascades to delete their comments.

**Why:** The `foreignId()->constrained()` method creates a foreign key column with an index and constraint. The `cascadeOnDelete()` method ensures referential integrity when parent records are deleted.

### Real-World Cases

- **User Management:** `create_users_table` with columns for name, email, password, and timestamps.
- **E-commerce:** `create_orders_table` with foreign keys to users and products.
- **Content Management:** `add_slug_to_posts_table` to add SEO-friendly URL slugs.
- **Multi-Tenancy:** `create_tenants_table` with subdomain and database configuration columns.
- **Audit Logging:** `create_activity_log_table` with polymorphic subject relationships.

### References

- Laravel Database: Migrations — Migration Structure — https://laravel.com/docs/12.x/migrations#migration-structure
- Laravel Database: Migrations — Creating Tables — https://laravel.com/docs/12.x/migrations#creating-tables
- Laravel Database: Migrations — Updating Tables — https://laravel.com/docs/12.x/migrations#updating-tables
- Laravel News: Anonymous Migrations in Laravel — https://laravel-news.com/anonymous-migrations-in-laravel
- Laravel Daily: Anonymous Migrations — https://laraveldaily.com/post/anonymous-migrations

---

## 3. Execution Workflow

### Definitions

**Core Definition:** The migration execution workflow encompasses the Artisan commands used to run migrations, check their status, and ensure safe execution across multi-server deployments through isolation.

**Technical Definition:** The `php artisan migrate` command executes all pending migrations in timestamp order, recording each in the `migrations` table with the current batch number. The `php artisan migrate:status` command displays a table showing which migrations have been run (`Ran?` column) and their batch numbers. The `--isolated` flag, introduced in Laravel 9.38, acquires an atomic lock via the application's cache driver before running migrations, preventing concurrent execution across multiple servers . Additional commands include `migrate:rollback` (reverses the last batch), `migrate:reset` (reverses all migrations), `migrate:refresh` (rolls back and re-runs all migrations), and `migrate:fresh` (drops all tables and re-runs all migrations).

**Beginner-Friendly Explanation:** Running migrations is a simple command: `php artisan migrate`. But there are several variations for different situations. `migrate:status` shows you what's already been done and what's still pending. `migrate:rollback` undoes the last batch of changes. `migrate:fresh` wipes everything and starts over—useful when you're experimenting in development. And `--isolated` is a safety flag that prevents two servers from running migrations at the same time during deployment.

### Purposes

- To run all pending migrations using `php artisan migrate`.
- To check which migrations have been executed and which are pending using `php artisan migrate:status`.
- To prevent concurrent migration execution in multi-server deployments using `--isolated`.
- To reverse the last batch of migrations using `php artisan migrate:rollback`.
- To reverse all migrations using `php artisan migrate:reset`.
- To rebuild the entire database from scratch using `php artisan migrate:fresh`.
- To safely apply migrations in production using `--force`.

### Syntax Rules and Structure

**Complete General Syntax — Running Migrations:**

```bash
php artisan migrate
php artisan migrate --force
php artisan migrate --isolated
php artisan migrate --path=/database/migrations/custom
```

**Component Breakdown:**

| Command / Option | Description |
|------------------|-------------|
| `php artisan migrate` | Runs all pending migrations. |
| `--force` | Runs migrations in production without confirmation. |
| `--isolated` | Acquires an atomic lock before running migrations. |
| `--path` | Runs migrations from a specific directory. |

**Complete General Syntax — Checking Status:**

```bash
php artisan migrate:status
php artisan migrate:status --pending
php artisan migrate:status --database=mysql
```

**Component Breakdown:**

| Command / Option | Description |
|------------------|-------------|
| `php artisan migrate:status` | Shows the status of all migrations. |
| `--pending` | Shows only pending migrations (Laravel 10+). |
| `--database` | Shows status for a specific connection. |

**Complete General Syntax — Rollback and Reset:**

```bash
php artisan migrate:rollback
php artisan migrate:rollback --step=1
php artisan migrate:reset
php artisan migrate:refresh
php artisan migrate:fresh
```

**Component Breakdown:**

| Command | Description |
|---------|-------------|
| `migrate:rollback` | Reverses the last batch of migrations. |
| `migrate:rollback --step=1` | Reverses a specific number of migrations. |
| `migrate:reset` | Reverses all migrations. |
| `migrate:refresh` | Rolls back all migrations and re-runs them. |
| `migrate:fresh` | Drops all tables and re-runs all migrations. |

**Syntax Rules:**

- `php artisan migrate` runs all migrations that have not yet been recorded in the `migrations` table.
- The `--force` flag is required in production environments to bypass the confirmation prompt.
- The `--isolated` flag acquires an atomic lock using the application's cache driver; if the lock is held, the command exits successfully without running migrations .
- `migrate:status` displays a table with columns `Ran?`, `Migration`, and `Batch`.
- `migrate:fresh` drops all tables regardless of the `migrations` table—use with caution.

**Constraints and Limitations:**

- **`--isolated` requires cache:** The `--isolated` flag depends on a working cache driver (Redis, Memcached, or database). If the cache is unavailable, the lock cannot be acquired.
- **`--isolated` is application-level:** The lock is an application-level lock, not a database-level lock. It prevents two Laravel processes from running migrations simultaneously but does not prevent external database changes .
- **`migrate:fresh` data loss:** `migrate:fresh` drops all tables, including tables not managed by migrations. Use only in development or with explicit intent.
- **Production safety:** In production, always use `--force` and consider running migrations during a maintenance window.

### Multiple Annotated Complete Code Examples

**Example 1: Running Migrations and Checking Status**

```bash
# Run all pending migrations
php artisan migrate

# Output:
# Migration table created successfully.
# Migrating: 0001_01_01_000000_create_users_table
# Migrated:  0001_01_01_000000_create_users_table (12.34ms)
# Migrating: 2025_01_15_143000_create_flights_table
# Migrated:  2025_01_15_143000_create_flights_table (8.21ms)

# Check the status of all migrations
php artisan migrate:status

# Output:
# +------+-------------------------------------------------------+-------+
# | Ran? | Migration                                             | Batch |
# +------+-------------------------------------------------------+-------+
# | Yes  | 0001_01_01_000000_create_users_table                  | 1     |
# | Yes  | 2025_01_15_143000_create_flights_table               | 1     |
# | No   | 2025_02_20_100000_add_slug_to_posts_table            |       |
# +------+-------------------------------------------------------+-------+
```

**Expected Output:** Migrations are executed and recorded. The status command shows which migrations have run (`Yes`), which are pending (`No`), and their batch numbers .

**Why:** `php artisan migrate` runs all pending migrations. `migrate:status` provides a clear overview of the migration state .

---

**Example 2: Isolated Migration Execution**

```bash
# Deploy to multiple servers — only one should run migrations
php artisan migrate --isolated

# On the first server:
# Migration table created successfully.
# Migrating: 2025_02_20_100000_add_slug_to_posts_table
# Migrated:  2025_02_20_100000_add_slug_to_posts_table (5.43ms)

# On the second server (concurrent):
# (No output — lock is held, command exits successfully)
```

**Expected Output:** Only the first server to acquire the lock runs the migrations. The second server exits with a successful status code but does not execute any migrations .

**Why:** The `--isolated` flag acquires an atomic lock via the cache driver before running migrations. This prevents two servers from attempting to modify the database schema simultaneously during deployment .

---

**Example 3: Rollback and Refresh**

```bash
# Roll back the last batch of migrations
php artisan migrate:rollback

# Output:
# Rolling back: 2025_02_20_100000_add_slug_to_posts_table
# Rolled back:  2025_02_20_100000_add_slug_to_posts_table (3.21ms)

# Rebuild the database from scratch (development only)
php artisan migrate:fresh --seed

# Output:
# Dropped all tables successfully.
# Migration table created successfully.
# Migrating: 0001_01_01_000000_create_users_table
# Migrated:  0001_01_01_000000_create_users_table (10.12ms)
# Seeding: Database\Seeders\DatabaseSeeder
# Seeded:  Database\Seeders\DatabaseSeeder (45.67ms)
```

**Expected Output:** `migrate:rollback` reverses the last batch. `migrate:fresh --seed` drops all tables, re-runs all migrations, and seeds the database .

**Why:** `migrate:rollback` uses the batch number to reverse the most recent migrations. `migrate:fresh` is a development convenience that rebuilds the database from scratch and optionally seeds it.

### Real-World Cases

- **Deployment Pipelines:** `php artisan migrate --force --isolated` is run during deployment to safely apply schema changes across multiple servers.
- **Local Development:** `php artisan migrate:fresh --seed` resets the local database to a known state with test data.
- **Troubleshooting:** `php artisan migrate:status` identifies which migrations are pending after a failed deployment.
- **Rollback After Failed Deployment:** `php artisan migrate:rollback --step=1` reverses a problematic migration without affecting earlier batches.
- **CI/CD Testing:** `php artisan migrate:fresh` is run in CI pipelines to ensure migrations work from a clean slate.

### References

- Laravel Database: Migrations — Running Migrations — https://laravel.com/docs/12.x/migrations#running-migrations
- Laravel Database: Migrations — Rolling Back Migrations — https://laravel.com/docs/12.x/migrations#rolling-back-migrations
- Laravel Database: Migrations — Isolating Migration Execution — https://laravel.com/docs/12.x/migrations#isolating-migration-execution
- Laravel 9.38: Isolated Migrations (GitHub PR #44743) — https://github.com/laravel/framework/pull/44743
- Stack Overflow: Do Laravel migrations lock tables? — https://stackoverflow.com/questions/46354270

---

## References

- Laravel Database: Migrations (12.x) — https://laravel.com/docs/12.x/migrations
- Laravel Database: Migrations — Introduction — https://laravel.com/docs/12.x/migrations#introduction
- Laravel Database: Migrations — Generating Migrations — https://laravel.com/docs/12.x/migrations#generating-migrations
- Laravel Database: Migrations — Squashing Migrations — https://laravel.com/docs/12.x/migrations#squashing-migrations
- Laravel Database: Migrations — Migration Structure — https://laravel.com/docs/12.x/migrations#migration-structure
- Laravel Database: Migrations — Running Migrations — https://laravel.com/docs/12.x/migrations#running-migrations
- Laravel Database: Migrations — Rolling Back Migrations — https://laravel.com/docs/12.x/migrations#rolling-back-migrations
- Laravel Database: Migrations — Isolating Migration Execution — https://laravel.com/docs/12.x/migrations#isolating-migration-execution
- Laravel News: Anonymous Migrations in Laravel — https://laravel-news.com/anonymous-migrations-in-laravel
- Laravel Daily: Anonymous Migrations — https://laraveldaily.com/post/anonymous-migrations
- Stack Overflow: Why does migrate create a migrations table? — https://stackoverflow.com/questions/27943744
- Stack Overflow: Do Laravel migrations lock tables? — https://stackoverflow.com/questions/46354270
- Stack Overflow: Laravel migrations batch number — https://stackoverflow.com/questions/54414147
- GitHub: Laravel Framework PR #44743 — Make migrate command isolated — https://github.com/laravel/framework/pull/44743
- GitHub: Laravel Framework PR #36906 — Anonymous migrations — https://github.com/laravel/framework/pull/36906