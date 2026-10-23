# Laravel Migrations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Migrations are a version control system for your database schema, allowing your development team to define, share, and modify the application's database structure in a structured, repeatable, and reversible manner.

**Technical Definition**

Migrations are PHP classes stored in the `database/migrations` directory that use the `Illuminate\Database\Schema\Blueprint` class and the `Schema` facade to define database operations. Each migration file is timestamped for ordering and contains an `up()` method to apply changes and a `down()` method to reverse them. Laravel tracks executed migrations in a `migrations` table, enabling safe rollbacks and consistent schema state across all environments. Migrations are database-agnostic, supporting MySQL, PostgreSQL, SQLite, and SQL Server through a unified API.

**Beginner-Friendly Explanation**

Think of migrations as "version control for your database." Just like Git tracks changes to your code files, migrations track changes to your database tables. If you add a new column to a table, you write a migration file that describes that change. When a teammate pulls your code, they run one command and their database gets the same column. If something goes wrong, you can roll back to the previous state. No more manually telling teammates to "add this column to your local database" after pulling changes.

---

### Key Characteristics

- **Version Control for Schema**: Every database change is captured in a timestamped, versioned file that can be tracked in Git alongside your code.
- **Reversible Operations**: Each migration has an `up()` method for applying changes and a `down()` method for reversing them.
- **Database Agnostic**: The `Schema` facade provides a unified API that works across MySQL, PostgreSQL, SQLite, and SQL Server.
- **Automatic Ordering**: Timestamps in filenames ensure migrations run in the correct order.
- **Squashing**: Large migration collections can be collapsed into a single SQL schema dump for cleaner repositories and faster test setups.
- **Anonymous Classes**: Modern Laravel uses anonymous migration classes (`return new class extends Migration`) to prevent class name conflicts.
- **Atomic Locking**: Migrations can be run with `--isolated` to prevent conflicts across multiple servers or processes.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A configured database connection** in `config/database.php`.
- **Basic knowledge of SQL** (CREATE TABLE, ALTER TABLE, etc.) is helpful.
- **A text editor or IDE** with PHP support.

---

### Related Programming Areas

- **Eloquent Models**: Models rely on the database schema defined by migrations.
- **Schema Builder**: The `Blueprint` class provides the methods used inside migrations.
- **Database Seeding**: Seeders populate the tables created by migrations.
- **Query Builder**: Migrations execute DDL operations that the query builder normally handles at runtime.
- **CI/CD Pipelines**: Migrations are typically run automatically during deployment.

---

## Core Concepts

---

### 1. Migrations

#### Definitions

**Core Definition**

A migration is a PHP class that defines a single, atomic change to the database schema, such as creating a table, adding a column, or creating an index.

**Technical Definition**

Migrations are stored in `database/migrations` and follow a naming convention of `YYYY_MM_DD_HHMMSS_description.php`. Each migration extends `Illuminate\Database\Migrations\Migration` and uses the `Schema` facade to interact with the database. Laravel maintains a `migrations` table that records which migrations have been run, allowing the system to determine which migrations are pending. Migrations are the only supported way to modify the database schema in Laravel applications, ensuring consistency across development, staging, and production environments.

**Beginner-Friendly Explanation**

A migration is a file that tells Laravel how to change your database. Want to create a `users` table? Write a migration. Want to add a `phone` column to that table later? Write another migration. Laravel runs them in order and keeps track of what's been done.

#### Purposes

- To provide a version-controlled, shareable definition of the database schema.
- To eliminate manual database changes and the inconsistencies they cause across environments.
- To enable safe, reversible schema modifications through the `up()` and `down()` methods.
- To support database-agnostic schema management across multiple database systems.
- To automate database setup for new developers and CI/CD pipelines.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        // Schema changes to apply
    }

    public function down(): void
    {
        // Schema changes to reverse
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `return new class extends Migration` | Anonymous class syntax (Laravel 8+), prevents class name conflicts. |
| `up(): void` | Method that applies the migration's changes. |
| `down(): void` | Method that reverses the migration's changes. |
| `Schema::create()` | Creates a new table. |
| `Schema::table()` | Modifies an existing table. |
| `Blueprint` | Provides column and index definition methods. |

**Syntax Rules**

- Migration filenames must include a timestamp prefix for ordering.
- The `up()` method must be present; the `down()` method is optional but strongly recommended.
- The `down()` method should reverse the operations performed by `up()` exactly.
- Anonymous classes are the modern convention; named classes are legacy but still supported.

**Constraints and Limitations**

- Once a migration has been deployed to production, it should never be modified; create a new migration instead.
- Migrations that mix schema changes (DDL) and data manipulation (DML) can leave the database in an unrecoverable state if they fail partway through.
- Squashing is only available for MariaDB, MySQL, PostgreSQL, and SQLite.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Creating a Table**

**Code**

```php
<?php
// database/migrations/2026_10_23_000000_create_flights_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();                                // Auto-incrementing primary key
            $table->string('name');                      // VARCHAR column
            $table->string('airline');                   // VARCHAR column
            $table->timestamps();                        // created_at & updated_at
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('flights');                 // Reverse: drop the table
    }
};
```

**Command**

```bash
php artisan migrate
```

**Expected Output (Terminal)**

```
INFO  Running migrations.
2026_10_23_000000_create_flights_table ......................... 25ms DONE
```

**Why This Output Occurs**

The `up()` method calls `Schema::create()` to create the `flights` table with the specified columns. The `down()` method drops the table if it exists, allowing the migration to be rolled back safely.

---

**Example 2: Modifying an Existing Table**

**Code**

```php
<?php
// database/migrations/2026_10_23_010000_add_slug_to_posts_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->string('slug')->unique()->after('title'); // Add slug column
        });
    }

    public function down(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->dropColumn('slug');                       // Reverse: remove column
        });
    }
};
```

**Expected Output**

The `slug` column is added to the `posts` table after the `title` column, with a unique index.

**Why This Output Occurs**

`Schema::table()` modifies an existing table. The `after()` modifier positions the column. The `down()` method drops the column, restoring the original schema.

---

#### Real-World Cases with Explanation

**Case 1: Team Onboarding**

A new developer joins a project. Instead of receiving a list of manual SQL commands, they run `php artisan migrate` and their local database is automatically set up with the exact same schema as production.

**Explanation**: Migrations eliminate environment drift and onboarding friction.

---

### 2. Migration Creation (`artisan make:migration`)

#### Definitions

**Core Definition**

`artisan make:migration` is the Artisan command used to generate a new migration file with the correct timestamped filename and a pre-filled anonymous class template.

**Technical Definition**

The command is handled by `Illuminate\Database\Console\Migrations\MigrateMakeCommand`. It creates a PHP file in `database/migrations` with a timestamp prefix derived from the current date and time. The command attempts to infer the table name and whether the migration creates a new table from the migration name (e.g., `create_users_table` → table `users`, operation `create`). Options allow explicit specification of the table and operation.

**Beginner-Friendly Explanation**

Creating a migration is easy—just run a command and Laravel makes the file for you. You give it a descriptive name, and Laravel fills in as much as it can guess.

#### Purposes

- To automate the creation of migration files with correct naming and timestamps.
- To pre-fill the migration template with the correct table name and operation when possible.
- To support custom paths and stub customization.
- To reduce manual errors in migration file naming and structure.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:migration create_flights_table
php artisan make:migration add_slug_to_posts_table --table=posts
php artisan make:migration create_users_table --create=users
php artisan make:migration custom_migration --path=database/migrations/custom
```

**Component Breakdown**

| Argument/Option | Description |
|-----------------|-------------|
| `<name>` | Descriptive migration name (e.g., `create_flights_table`). |
| `--create=table` | Pre-fills the migration to create the specified table. |
| `--table=table` | Pre-fills the migration to modify the specified table. |
| `--path=path` | Custom output path relative to the application base path. |

**Syntax Rules**

- The migration name should describe the change (e.g., `create_posts_table`, `add_slug_to_posts`).
- Laravel guesses the table name and operation from the name (e.g., `create_` prefix → `--create`, `add_`/`to_` → `--table`).
- The `--path` option must be relative to the application base path.

**Constraints and Limitations**

- Laravel's guessing is heuristic and may not always be correct; verify the generated file.
- Custom stubs can be published with `stub:publish` for organization-specific templates.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Create Migration**

**Command**

```bash
php artisan make:migration create_users_table
```

**Generated File**

```php
<?php
// database/migrations/2026_10_23_000000_create_users_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

**Expected Output (Terminal)**

```
INFO  Created Migration: 2026_10_23_000000_create_users_table
```

**Why This Output Occurs**

Laravel recognizes the `create_` prefix and the `users` table name, and pre-fills the migration with `Schema::create('users', ...)`.

---

**Example 2: Generating a Modify Migration with `--table`**

**Command**

```bash
php artisan make:migration add_slug_to_posts_table --table=posts
```

**Generated File**

```php
<?php

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            // Add columns here
        });
    }

    public function down(): void
    {
        Schema::table('posts', function (Blueprint $table) {
            // Remove columns here
        });
    }
};
```

**Expected Output**

The migration is pre-filled with `Schema::table('posts', ...)` instead of `Schema::create()`.

**Why This Output Occurs**

The `--table` option explicitly tells Laravel the migration modifies an existing table.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Feature Development**

A developer adds a `slug` column to the `posts` table. They run `php artisan make:migration add_slug_to_posts_table --table=posts`, fill in the column definition, and run `php artisan migrate`.

**Explanation**: The command generates the correct file structure, reducing boilerplate.

---

### 3. Migration Structure

#### Definitions

**Core Definition**

Migration structure refers to the organization and components of a migration file, including the class declaration, the `up()` and `down()` methods, and the schema operations defined within them.

**Technical Definition**

A migration file consists of a PHP class (typically anonymous in modern Laravel) extending `Illuminate\Database\Migrations\Migration`. The class contains an `up()` method that defines the schema changes to apply and a `down()` method that reverses them. Inside these methods, the `Schema` facade and `Blueprint` class are used to define tables, columns, indexes, and foreign keys. The `$connection` property can specify a non-default database connection.

**Beginner-Friendly Explanation**

A migration file has two main parts: `up()` for applying changes and `down()` for undoing them. Inside these methods, you use `Schema::create()` or `Schema::table()` to describe what you want to do.

#### Purposes

- To provide a standardized, readable format for schema changes.
- To separate the forward migration (`up`) from the rollback (`down`) for clarity and safety.
- To enable the use of the `Blueprint` API for database-agnostic column and index definitions.
- To support multiple database connections within a single application.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    // Optional: specify a non-default connection
    // protected $connection = 'mysql_logs';

    public function up(): void
    {
        Schema::create('table_name', function (Blueprint $table) {
            $table->id();
            $table->string('column');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('table_name');
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `up(): void` | Applies the migration. |
| `down(): void` | Reverses the migration. |
| `Schema::create()` | Creates a new table. |
| `Schema::table()` | Modifies an existing table. |
| `$connection` | Optional property for non-default database connections. |
| `Blueprint` | Provides column type methods (`string`, `integer`, `text`, etc.). |

**Syntax Rules**

- The `up()` method must contain the schema changes.
- The `down()` method should exactly reverse the `up()` operations.
- Anonymous classes are recommended to avoid class name collisions.
- The `$connection` property can be set to use a different database connection for a specific migration.

**Constraints and Limitations**

- The `down()` method is optional but omitting it makes the migration irreversible.
- Migrations should not contain data manipulation (DML) alongside schema changes (DDL).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Complete Migration with Foreign Key**

**Code**

```php
<?php
// database/migrations/2026_10_23_000000_create_posts_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->string('title');
            $table->text('body');
            $table->string('slug')->unique()->after('title');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

**Expected Output**

The `posts` table is created with a foreign key to `users`, a unique `slug` column, and timestamps.

**Why This Output Occurs**

`foreignId('user_id')->constrained()` creates a foreign key referencing `users.id` with automatic naming and referential integrity. `cascadeOnDelete()` ensures posts are deleted when their author is deleted.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Database Application**

An application uses a primary database for transactional data and a separate logging database. A migration sets `protected $connection = 'mysql_logs';` to direct the migration to the logging database.

**Explanation**: The `$connection` property enables migrations to target specific databases.

---

### 4. Migration Methods: `up()` and `down()`

#### Definitions

**Core Definition**

The `up()` method defines the schema changes to apply when a migration is run, while the `down()` method defines how to reverse those changes when a migration is rolled back.

**Technical Definition**

`up()` is executed by the migrator when applying the migration. It typically calls `Schema::create()` or `Schema::table()` with a closure that receives a `Blueprint` instance. `down()` is executed during rollback and must undo everything `up()` did, using the inverse operations (e.g., `dropIfExists()` for `create()`, `dropColumn()` for `addColumn()`). If `down()` is omitted, the migration cannot be rolled back.

**Beginner-Friendly Explanation**

`up()` is "do this." `down()` is "undo this." They should always be mirror images of each other.

#### Purposes

- To apply schema changes safely and predictably.
- To enable rollback and recovery from failed deployments.
- To support CI/CD pipelines that run migrations and rollbacks automatically.
- To maintain database integrity across environments.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function up(): void
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('email')->unique();
        $table->timestamps();
    });
}

public function down(): void
{
    Schema::dropIfExists('users');
}
```

**Component Breakdown**

| Method | Operation | Inverse |
|--------|-----------|---------|
| `Schema::create()` | Create table | `Schema::dropIfExists()` |
| `$table->string()` | Add column | `$table->dropColumn()` |
| `$table->foreignId()` | Add foreign key | `$table->dropForeign()` |
| `$table->index()` | Add index | `$table->dropIndex()` |

**Syntax Rules**

- `down()` must reverse each operation in `up()`, ideally in reverse order.
- Use `dropIfExists()` instead of `drop()` for safety.
- For multi-step migrations, reverse the steps in reverse order.

**Constraints and Limitations**

- Some operations (e.g., dropping a column) may be irreversible without data loss; document this clearly.
- Modifying a column type may not have a clean inverse; use caution.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Reversible Column Addition**

**Code**

```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('phone')->nullable()->after('email');
    });
}

public function down(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('phone');
    });
}
```

**Expected Output**

The `phone` column is added in `up()` and removed in `down()`.

**Why This Output Occurs**

`dropColumn()` exactly reverses the `string()` column addition.

---

**Example 2: Irreversible Migration (Documented)**

**Code**

```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('legacy_field'); // Irreversible: data is lost
    });
}

public function down(): void
{
    // This migration is intentionally irreversible.
    // Data lost from dropping legacy_field cannot be restored.
    // Use a forward-fix migration to re-add the column if needed.
}
```

**Expected Output**

The column is dropped in `up()`, and `down()` intentionally does nothing.

**Why This Output Occurs**

Dropping a column destroys data; the `down()` method cannot recreate the lost data. A clear comment documents this intentional irreversibility.

---

#### Real-World Cases with Explanation

**Case 1: Safe Rollback During Deployment**

A migration fails during a production deployment. The deployment pipeline automatically runs `php artisan migrate:rollback --step=1`, and the `down()` method cleanly reverses the last migration, restoring the database to its previous state.

**Explanation**: Well-written `down()` methods enable safe, automated recovery from failed deployments.

---

### 5. Migration Squashing

#### Definitions

**Core Definition**

Migration squashing is the process of collapsing a large number of historical migration files into a single SQL schema dump, reducing repository clutter and accelerating database setup for new environments.

**Technical Definition**

The `php artisan schema:dump` command generates a SQL file in `database/schema` that represents the current database schema. When migrations are run on a fresh database, Laravel first executes the schema dump, then runs only the migrations that were created after the dump. The `--prune` option deletes the original migration files that were squashed. Squashing is supported for MariaDB, MySQL, PostgreSQL, and SQLite.

**Beginner-Friendly Explanation**

Over time, you accumulate hundreds of migration files. Squashing lets you say "take the current state of my database, save it as one SQL file, and delete all the old migration files that led to it." New developers then get the database set up in one step instead of running hundreds of tiny migrations.

#### Purposes

- To reduce the number of migration files in the repository.
- To accelerate database setup for new developers and CI/CD pipelines.
- To clean up the `database/migrations` directory.
- To preserve the exact schema state while discarding historical migration steps.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Dump the current schema (without pruning)
php artisan schema:dump

# Dump the current schema and prune all existing migrations
php artisan schema:dump --prune

# Dump for a specific database connection
php artisan schema:dump --database=testing --prune
```

**Component Breakdown**

| Option | Description |
|--------|-------------|
| `schema:dump` | Generates the SQL schema file. |
| `--prune` | Deletes the original migration files after dumping. |
| `--database` | Specifies the database connection to dump. |

**Syntax Rules**

- The schema file is written to `database/schema` and named after the database connection.
- The schema file should be committed to source control.
- After squashing, new migrations are still created and run normally.

**Constraints and Limitations**

- Only available for MariaDB, MySQL, PostgreSQL, and SQLite.
- If your tests use a different connection, dump a schema file for that connection as well.
- Squashing does not delete the `migrations` table; it remains for tracking post-squash migrations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Squashing Migrations**

**Command**

```bash
php artisan schema:dump --prune
```

**Expected Output (Terminal)**

```
INFO  Dumped the current schema.
INFO  Pruned 47 migrations.
```

**Files After Squashing**

```
database/
├── schema/
│   └── mysql-schema.sql          # The squashed schema dump
└── migrations/
    └── 2026_10_23_000000_create_new_table.php  # Only post-squash migrations
```

**Why This Output Occurs**

The command generates a SQL file representing the current schema and deletes the migration files that were squashed.

---

#### Real-World Cases with Explanation

**Case 1: Accelerating Test Suites**

A project has 300 migrations. Running them for every test suite takes 15 seconds. After `schema:dump`, tests run the schema file (under 1 second) and only the handful of new migrations, drastically speeding up the test suite.

**Explanation**: Squashing is particularly valuable for CI/CD and testing performance.

---

### 6. Migration Best Practices

#### Definitions

**Core Definition**

Migration best practices are the recommended guidelines for writing migrations that are focused, reversible, well-indexed, and safe for production deployment.

**Technical Definition**

Best practices encompass maintaining reversibility, avoiding modifications to deployed migrations, using `constrained()` for foreign keys, adding indexes in the migration rather than as an afterthought, mirroring column defaults in models, and keeping migrations focused on a single concern.

**Beginner-Friendly Explanation**

These are the rules experienced developers follow to avoid database disasters and keep migrations maintainable.

#### Key Best Practices

1. **Never Modify Deployed Migrations**: Once a migration has run in production, treat it as immutable. Create a new migration to make further changes.

2. **Always Write a `down()` Method**: Reversible migrations enable safe rollbacks during failed deployments and in CI/CD pipelines.

3. **Use `constrained()` for Foreign Keys**: This ensures automatic naming and referential integrity.

4. **Add Indexes in the Migration**: Don't treat indexes as an afterthought. Add them when you create the column or table.

5. **Keep Migrations Focused**: One concern per migration. Never mix DDL (schema changes) and DML (data manipulation) in a single migration.

6. **Mirror Column Defaults in Models**: When a column has a database default, mirror it in the model's `$attributes` so new instances have correct values before saving.

7. **Use `dropIfExists()` and `dropColumn()` Safely**: Prefer `dropIfExists()` over `drop()`, and be aware that dropping columns is irreversible.

8. **Test Migrations in Staging First**: Before running in production, test migrations in an environment that mirrors production.

9. **Plan for Downtime**: Run destructive migrations during off-peak hours and communicate with your team.

10. **Use `--isolated` for Multi-Server Deployments**: This prevents migration conflicts across multiple servers or processes.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Good vs. Bad Migration**

**Bad (Mixing DDL and DML, No Indexes, Irreversible)**

```php
public function up(): void
{
    Schema::create('orders', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained();
        $table->string('status');          // No index
        $table->timestamps();
    });

    // DML mixed with DDL — bad!
    DB::table('orders')->insert(['user_id' => 1, 'status' => 'pending']);
}
```

**Good (Focused, Indexed, Reversible)**

```php
public function up(): void
{
    Schema::create('orders', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->index();
        $table->string('status')->index();  // Index added from the start
        $table->timestamps();
    });
}

public function down(): void
{
    Schema::dropIfExists('orders');
}
```

**Expected Output**

The good migration is focused, indexed, reversible, and safe. The bad migration may leave the database in an inconsistent state if it fails partway through.

**Why This Output Occurs**

The good migration follows the principle of separating schema changes from data changes and includes performance-critical indexes from the start.

---

#### Real-World Cases with Explanation

**Case 1: Production Deployment with Zero Downtime**

A team follows best practices by writing a new migration to add a nullable column with an index, running it in staging first, and using `--isolated` in production. The deployment succeeds with no downtime or rollback needed.

**Explanation**: Best practices prevent common deployment pitfalls and ensure database integrity.

---

## References

- Laravel Database: Migrations (Laravel 12.x) — https://laravel.com/docs/12.x/migrations
- Laravel Database: Migrations (Laravel 13.x) — https://laravel.com/framework/docs/13.x/migrations
- Dutch Laravel Foundation: Write Effective Migrations — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/database-and-eloquent-orm/write-effective-migrations.md
- Laravel Artisan Cheatsheet: `make:migration` — https://github.com/Aminur670IT/Laravel9_Docker_PMA/blob/master/Laravel v9.x - Laravel Artisan Cheatsheet.pdf
- Laravel 12 Migration Updates: Squashing, Skipping, Isolating (LinkedIn) — https://www.linkedin.com/posts/im-rahulkumar-rajput_laravel-laravel12-php-activity-7373002451752431616-XmL_
- Laravel News: Conditional Migrations with `shouldRun()` — https://laravel-news.com/conditional-migrations
- Laracasts: Migrations in Production — https://laracasts.com/discuss/channels/laravel/migrations-in-production
- Laravel Schema Builder Documentation — https://laravel.com/docs/12.x/migrations#creating-tables

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Anonymous migration classes and `schema:dump` have been available since Laravel 8. The `--isolated` flag and `shouldRun()` method were introduced in Laravel 12. Migration squashing requires MariaDB, MySQL, PostgreSQL, or SQLite. Developers should always consult the documentation for their specific Laravel version.