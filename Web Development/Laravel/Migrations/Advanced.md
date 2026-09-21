# Laravel Advanced Migration & Production Management (New): A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Advanced Migration & Production Management encompasses the tools, commands, and strategic patterns required to manage database schema evolution at scale—including consolidating historical migration files, auditing database structures from the CLI, and safely executing schema changes in zero-downtime production environments.

**Technical Definition:** Advanced migration management leverages Laravel's `schema:dump` command (which uses the database's native command-line client to generate SQL schema files), the `db:show` and `db:table` Artisan commands (introduced in Laravel 9.24) for database introspection, and a combination of safety mechanisms including the `--force` flag, the Expand-Contract pattern, and external online DDL tools (gh-ost, pt-online-schema-change) to perform schema alterations on high-traffic tables without locking or downtime.

**Beginner-Friendly Explanation:** As your application grows, you accumulate hundreds of migration files. Squashing lets you consolidate them into a single SQL file so new developers don't have to run them one by one. Database inspection commands let you peek at your database structure directly from the terminal. And production safety protocols ensure that when you change your database schema on a live site with thousands of users, you don't break anything or cause downtime.

### Key Characteristics

- **Schema Consolidation:** `schema:dump` compresses hundreds of migration files into a single SQL file, dramatically speeding up initial database setup.
- **CLI-First Inspection:** `db:show` and `db:table` provide comprehensive database and table insights without requiring a GUI client.
- **Production Guardrails:** The `--force` flag prevents accidental destructive operations; packages like `laravel-migration-guard` catch dangerous migrations before they run.
- **Zero-Downtime Pattern:** The Expand-Contract pattern enables schema changes that are safe for multi-server rolling deployments.
- **Online DDL Integration:** Packages like `daursu/laravel-zero-downtime-migration` integrate gh-ost and pt-online-schema-change into Laravel's migration workflow.
- **Database-Specific Support:** Migration squashing is available for MariaDB, MySQL, PostgreSQL, and SQLite.

### Prerequisites

- **Laravel 9.24+** for `db:show` and `db:table`; **Laravel 8.x+** for `schema:dump`.
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A configured database connection in `config/database.php` and `.env`.
- For online DDL: MySQL 5.6+ (5.7.20+ recommended for enhanced Online DDL), gh-ost or Percona Toolkit installed.
- Basic understanding of migrations, database schema, and deployment workflows.

### Related Programming Areas

- **Database Migrations:** Squashing and production management are advanced extensions of the migration system.
- **Deployment & DevOps:** Zero-downtime migration strategies are critical components of deployment pipelines.
- **Database Performance:** Online DDL and schema optimization directly impact application performance.
- **Security & Safety:** `--force` protection and migration guard packages prevent production incidents.
- **Database Administration:** `db:show` and `db:table` provide DBA-level insights from the application CLI.

### Core Concepts / Features

1. Migration Squashing (`schema:dump`)
2. Database Inspection (`db:show`, `db:table`)
3. Production Safety Protocols (`--force`, Zero-Downtime, Online DDL)

---

## 1. Migration Squashing

### Definitions

**Core Definition:** Migration squashing is the process of consolidating hundreds of individual migration files into a single SQL schema file that Laravel executes first, followed by any remaining migrations not included in the dump.

**Technical Definition:** The `php artisan schema:dump` command uses the database's native command-line client (`mysqldump`, `pg_dump`, `sqlite3 .dump`) to export the current database schema into a SQL file stored in `database/schema/`. The file's name corresponds to the database connection. When `migrate` is executed and no migrations have been run, Laravel first executes the SQL statements in the schema file, then runs any remaining migrations not included in the dump. The `--prune` flag removes all existing migration files after a successful dump, and the `--database` flag allows dumping a specific connection's schema.

**Beginner-Friendly Explanation:** Imagine you've been working on a project for two years and have 300 migration files. Every time a new developer joins the team, they have to run all 300 migrations one by one to set up their local database. That's slow and error-prone. Squashing lets you say "take the current state of my database, save it as a single SQL file, and delete all the old migration files." Now new developers just run one command, and they have the exact same database structure in seconds.

### Purposes

- To clean up bloated `database/migrations` directories by consolidating historical migrations.
- To dramatically speed up initial database setup for new developers.
- To reduce the risk of migration errors by eliminating hundreds of historical migrations.
- To provide a single, authoritative schema file that can be committed to version control.
- To enable faster CI/CD pipelines by reducing migration execution time.

### Syntax Rules and Structure

**Complete General Syntax — Basic Schema Dump:**

```bash
php artisan schema:dump
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `schema:dump` | Generates a SQL schema file from the current database state. |
| Output location | `database/schema/{connection}.sql` |

**Complete General Syntax — Dump and Prune:**

```bash
php artisan schema:dump --prune
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--prune` | Deletes all existing migration files after the dump. |

**Complete General Syntax — Dump Specific Connection:**

```bash
php artisan schema:dump --database=testing --prune
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `--database=testing` | Dumps the schema for the specified connection. |

**Syntax Rules:**

- `schema:dump` requires the database's command-line client to be installed and in the system PATH.
- Migration squashing is available for MariaDB, MySQL, PostgreSQL, and SQLite.
- Schema dumps may not be restored to in-memory SQLite databases.
- The `--prune` flag removes migration files after a successful dump—use with caution.
- The schema file should be committed to source control so new developers can quickly set up their databases.
- When `migrate` runs and no migrations have been executed, Laravel executes the schema file first, then any remaining migrations.

**Constraints and Limitations:**

- **CLI client dependency:** The database's command-line client must be installed and accessible. If `mysqldump` is not in the PATH, the command fails.
- **In-memory SQLite:** Schema dumps cannot be restored to in-memory SQLite databases—this affects testing setups.
- **Data loss on prune:** The `--prune` flag permanently deletes migration files. Ensure the schema dump is correct before pruning.
- **No data export:** Migration squashing only dumps the schema, not the data. Use a separate package like `dragon-code/laravel-data-dumper` if you need to include data from specific tables.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Schema Dump Workflow**

```bash
# Step 1: Ensure all migrations have been run
php artisan migrate

# Step 2: Dump the current schema
php artisan schema:dump

# Output:
# Dumped the current schema to database/schema/mysql-schema.sql

# Step 3: Verify the schema file
ls database/schema/
# mysql-schema.sql

# Step 4: Commit the schema file to version control
git add database/schema/mysql-schema.sql
git commit -m "Add database schema dump"
```

**Expected Output:** A `database/schema/mysql-schema.sql` file is created containing the full database schema.

**Why:** The schema dump captures the current state of the database. When a new developer clones the repository and runs `php artisan migrate`, Laravel executes the schema file first, creating the entire database structure in one step, then runs any remaining migrations.

---

**Example 2: Dump and Prune All Migrations**

```bash
# Dump the schema and remove all migration files
php artisan schema:dump --prune

# Output:
# Dumped the current schema to database/schema/mysql-schema.sql
# Pruned 247 migration files

# The database/migrations directory is now empty (or contains only new migrations)
ls database/migrations/
# (empty)

# New developers now run:
php artisan migrate
# Laravel executes the schema file first, then any new migrations
```

**Expected Output:** The `database/migrations` directory is cleared of all pruned migrations. The schema file contains the complete database structure.

**Why:** The `--prune` flag removes historical migration files that are now represented in the schema dump, keeping the migrations directory clean and the setup process fast.

---

**Example 3: Dumping Multiple Connections**

```bash
# Dump the default connection schema
php artisan schema:dump

# Dump the testing connection schema (e.g., SQLite in-memory)
php artisan schema:dump --database=testing --prune

# Both schema files are now in database/schema/
ls database/schema/
# mysql-schema.sql
# sqlite-schema.sql
```

**Expected Output:** Two schema files are created—one for the default MySQL connection and one for the testing SQLite connection.

**Why:** If your tests use a different database connection than local development, you need a schema dump for each connection so tests can build the database quickly.

### Real-World Cases

- **Team Onboarding:** New developers run `php artisan migrate` and get a complete database in seconds instead of running 300 migrations.
- **CI/CD Pipelines:** Tests run faster because the database is built from a single schema file instead of hundreds of migrations.
- **Production Deployment:** New servers can be provisioned with the schema file, reducing deployment time.
- **Legacy Cleanup:** After years of development, the migrations directory is cleaned up and consolidated.

### References

- Laravel Database: Migrations — Squashing Migrations — https://laravel.com/docs/12.x/migrations#squashing-migrations
- Laravel Data Dumper Package (Dragon Code) — https://github.com/TheDragonCode/laravel-data-dumper
- Stack Overflow: How schema:dump works — https://stackoverflow.com/questions/75000000

---

## 2. Database Inspection

### Definitions

**Core Definition:** Database inspection in Laravel refers to the `db:show` and `db:table` Artisan commands introduced in Laravel 9.24, which provide comprehensive database and table insights directly from the terminal without requiring a GUI database client.

**Technical Definition:** The `db:show` command provides an overview of the database as a whole, including its type, connection details, number of open connections, and a summary of its tables. The `db:table` command provides a detailed view of an individual table, including its size, number of rows, column definitions with attributes and data types, indexes, and foreign keys. Both commands use Laravel's database connection layer to introspect the schema and are database-agnostic.

**Beginner-Friendly Explanation:** Instead of opening phpMyAdmin, TablePlus, or another database GUI, you can just type a command in your terminal and see everything you need to know about your database. `db:show` gives you the big picture—what database you're connected to, how many connections are open, and what tables exist. `db:table` zooms in on one table and tells you everything about its structure—columns, indexes, foreign keys, and more.

### Purposes

- To audit database structure directly from the CLI using `db:show`.
- To inspect individual table details (columns, indexes, foreign keys) using `db:table`.
- To monitor database connection counts with `db:monitor`.
- To quickly verify schema changes after running migrations.
- To troubleshoot database issues without needing a GUI client.
- To generate documentation from the database structure.

### Syntax Rules and Structure

**Complete General Syntax — `db:show`:**

```bash
php artisan db:show
php artisan db:show --counts
php artisan db:show --database=pgsql
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `--counts` | Includes row counts for each table. |
| `--database` | Shows details for a specific connection. |

**Complete General Syntax — `db:table`:**

```bash
php artisan db:table users
php artisan db:table users --database=pgsql
php artisan db:table users --json
```

**Component Breakdown:**

| Argument/Option | Description |
|-----------------|-------------|
| `users` | The table to inspect. |
| `--database` | Inspects a table on a specific connection. |
| `--json` | Outputs the table details as JSON. |

**Complete General Syntax — `db:monitor`:**

```bash
php artisan db:monitor --databases=mysql,pgsql --max=100
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `--databases` | Comma-separated list of connections to monitor. |
| `--max` | Maximum open connections before dispatching a `DatabaseBusy` event. |

**Syntax Rules:**

- `db:show` and `db:table` are available from Laravel 9.24+.
- `db:table` accepts a table name as its primary argument.
- The `--json` flag produces machine-readable output for scripting.
- `db:monitor` should be scheduled to run periodically for continuous monitoring.
- The `DatabaseBusy` event must be listened for in a service provider to trigger notifications.

**Constraints and Limitations:**

- **Multiple schemas:** `db:show` and `db:table` may fail when the database has tables in multiple schemas (e.g., PostgreSQL with multiple schemas).
- **MongoDB support:** Since `laravel-mongodb` 4.7, `db:show` and `db:table` work with MongoDB collections.
- **CLI only:** These commands are only available via the Artisan CLI, not programmatically (without additional effort).

### Multiple Annotated Complete Code Examples

**Example 1: Basic Database Overview**

```bash
php artisan db:show

# Output:
# +---------------------+------------------+
# | Database            | mysql            |
# | Host                | 127.0.0.1        |
# | Port                | 3306             |
# | Database            | laravel          |
# | Open Connections    | 5                |
# | Tables              | 12               |
# | Views               | 2                |
# +---------------------+------------------+
```

**Expected Output:** A summary of the database connection and table statistics.

**Why:** `db:show` provides a high-level overview of the database, including connection details and the number of open connections. This is useful for quickly verifying which database you're connected to and how many tables it contains.

---

**Example 2: Detailed Table Inspection**

```bash
php artisan db:table users

# Output:
# +----------------+---------------------+------+-----+---------+----------------+
# | Column         | Type                | Null | Key | Default | Extra          |
# +----------------+---------------------+------+-----+---------+----------------+
# | id             | bigint unsigned     | NO   | PRI | NULL    | auto_increment |
# | name           | varchar(255)        | NO   |     | NULL    |                |
# | email          | varchar(255)        | NO   | UNI | NULL    |                |
# | password       | varchar(255)        | NO   |     | NULL    |                |
# | remember_token | varchar(100)        | YES  |     | NULL    |                |
# | created_at     | timestamp           | YES  |     | NULL    |                |
# | updated_at     | timestamp           | YES  |     | NULL    |                |
# +----------------+---------------------+------+-----+---------+----------------+
#
# Indexes:
# +-------+----------------+----------+
# | Name  | Columns        | Type     |
# +-------+----------------+----------+
# | PRIMARY | id           | BTREE    |
# | users_email_unique | email | BTREE |
# +-------+----------------+----------+
```

**Expected Output:** A detailed breakdown of the `users` table, including all columns, their types, nullability, keys, defaults, and indexes.

**Why:** `db:table` provides a complete view of a table's structure, including column definitions, indexes, and foreign keys. This is useful for auditing schema changes and understanding table relationships.

---

**Example 3: Monitoring Database Connections**

```bash
php artisan db:monitor --databases=mysql --max=100

# If connections exceed 100:
# DatabaseBusy event dispatched for connection [mysql] with 101 connections.
```

**Expected Output:** The command checks the number of open connections. If the count exceeds the `--max` value, a `DatabaseBusy` event is dispatched.

**Why:** `db:monitor` is useful for detecting connection pool exhaustion in production. By scheduling this command to run every minute, you can proactively monitor database health.

### Real-World Cases

- **Post-Migration Verification:** Run `db:table users` after a migration to verify the schema was applied correctly.
- **Troubleshooting:** Use `db:show` to verify connection details when debugging database connectivity issues.
- **Production Monitoring:** Schedule `db:monitor` to alert when connection counts exceed thresholds.
- **Documentation:** Use `db:table --json` to generate schema documentation automatically.
- **Code Reviews:** Reviewers run `db:table` to understand schema changes without needing a local database.

### References

- Laravel: New DB Commands (Laravel Blog) — https://laravel.com/blog/laravel-new-db-commands-and-more
- Laravel 9.24 Released (Laravel News) — https://laravel-news.com/laravel-9-24-0
- Laravel Artisan Cheatsheet — https://github.com/laravel/artisan-cheatsheet

---

## 3. Production Safety Protocols

### Definitions

**Core Definition:** Production safety protocols are the practices, flags, and patterns used to safely execute database migrations in live production environments without causing downtime, data loss, or service interruptions.

**Technical Definition:** Production safety protocols include: (1) the `--force` flag, which bypasses Laravel's confirmation prompt for destructive commands in production; (2) the Expand-Contract pattern, a multi-deploy strategy for renaming or dropping columns without breaking running code; (3) online DDL tools (gh-ost, pt-online-schema-change) that perform schema alterations on a copy of the table while the original remains fully available; and (4) migration guard packages that statically analyse migrations for dangerous operations before they run.

**Beginner-Friendly Explanation:** Changing your database schema on a live website with thousands of users is risky. If you rename a column, the old code (still running on some servers) will break because it's looking for the old column name. If you add a column to a huge table, the database might lock the table for minutes, causing your site to go down. Production safety protocols solve these problems: the `--force` flag prevents accidental destructive commands, the Expand-Contract pattern lets you change columns in stages, and online DDL tools let you alter tables without locking them.

### Purposes

- To protect production databases from accidental destructive commands via the `--force` flag.
- To execute zero-downtime schema migrations using the Expand-Contract pattern.
- To manage online DDL constraints with tools like gh-ost and pt-online-schema-change.
- To catch dangerous migrations before they run using static analysis packages.
- To ensure safe multi-server rolling deployments where old and new code coexist.
- To minimize locking and replication lag during schema alterations.

### Syntax Rules and Structure

**Complete General Syntax — `--force` Flag:**

```bash
php artisan migrate --force
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `--force` | Bypasses the production confirmation prompt. |

**Complete General Syntax — Expand-Contract Pattern:**

```php
// Deploy #1: EXPAND — Add new column alongside old one
Schema::table('users', function (Blueprint $table) {
    $table->string('full_name')->nullable()->after('name');
});

// Deploy #2: MIGRATE DATA — Backfill and switch code to new column
User::whereNull('full_name')->chunkById(1000, function ($users) {
    foreach ($users as $user) {
        $user->update(['full_name' => $user->name]);
    }
});

// Deploy #3: CONTRACT — Drop old column after all code uses new one
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('name');
});
```

**Complete General Syntax — Online DDL with gh-ost:**

```bash
# Install the zero-downtime migration package
composer require daursu/laravel-zero-downtime-migration

# Configure the connection in config/zero-downtime-migration.php
# 'connections' => [
#     'zero-downtime' => [
#         'driver' => 'mysql',
#         'host' => env('DB_HOST'),
#         ...
#     ],
# ]

# Use the ZeroDowntimeSchema facade in migrations
use Daursu\ZeroDowntimeMigration\ZeroDowntimeSchema;

ZeroDowntimeSchema::table('users', function (Blueprint $table) {
    $table->string('full_name')->nullable()->after('name');
});
```

**Complete General Syntax — Migration Guard:**

```bash
# Install the migration guard package
composer require --dev malikad778/laravel-migration-guard

# Run the guard check before migrating
php artisan migrate:check

# Output:
# ⚠️  DANGEROUS: Dropping column 'name' from 'users' may break running code.
#    Consider using the Expand-Contract pattern.
```

**Syntax Rules:**

- The `--force` flag is required for production deployments; without it, Laravel prompts for confirmation.
- The Expand-Contract pattern requires three separate deployments: expand, migrate data, contract.
- Online DDL tools (gh-ost, pt-online-schema-change) require MySQL 5.6+ with binlog enabled (ROW format).
- Migration guard packages hook into `artisan migrate` and warn before dangerous operations run.
- The `daursu/laravel-zero-downtime-migration` package uses the `ZeroDowntimeSchema` facade instead of the standard `Schema` facade.

**Constraints and Limitations:**

- **`--force` is not a safety feature:** The `--force` flag bypasses the confirmation prompt—it does not protect against destructive commands. Use migration guard packages for actual protection.
- **Expand-Contract complexity:** Requires careful planning and multiple deployments. Not suitable for simple, non-breaking changes.
- **Online DDL tool dependency:** gh-ost and pt-online-schema-change require external tools to be installed and configured. They also need binlog access.
- **Multi-server deployments:** The Expand-Contract pattern is essential for multi-server deployments where old and new code coexist during rolling deployments.
- **Replication lag:** Online DDL tools can cause replication lag if not configured properly.

### Multiple Annotated Complete Code Examples

**Example 1: Expand-Contract Pattern for Column Rename**

```php
<?php
// Deploy #1: EXPAND — Add new column alongside old one
// database/migrations/2025_01_01_000000_add_full_name_to_users_table.php

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('full_name')->nullable()->after('name');
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('full_name');
        });
    }
};
```

```php
<?php
// Deploy #2: MIGRATE DATA — Backfill the new column
// database/migrations/2025_01_15_000000_backfill_full_name_on_users_table.php

return new class extends Migration
{
    public function up(): void
    {
        User::whereNull('full_name')->chunkById(1000, function ($users) {
            foreach ($users as $user) {
                $user->update(['full_name' => $user->name]);
            }
        });
    }
};
```

```php
<?php
// Deploy #3: CONTRACT — Drop the old column
// database/migrations/2025_02_01_000000_drop_name_from_users_table.php

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('name');
        });
    }
};
```

**Expected Output:** The `users` table gains a `full_name` column, data is backfilled from `name`, and finally the `name` column is dropped—all without breaking running code.

**Why:** The Expand-Contract pattern ensures that old and new code both work at every step of the migration. During Deploy #1, old code writes to `name`; new code writes to `full_name`. During Deploy #2, data is backfilled. During Deploy #3, old code is completely removed and the old column is dropped.

---

**Example 2: Online DDL with gh-ost**

```bash
# Install the zero-downtime migration package
composer require daursu/laravel-zero-downtime-migration

# Publish the configuration
php artisan vendor:publish --provider="Daursu\ZeroDowntimeMigration\ZeroDowntimeMigrationServiceProvider"
```

```php
<?php
// config/zero-downtime-migration.php

return [
    'connections' => [
        'zero-downtime' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE'),
            'username' => env('DB_USERNAME'),
            'password' => env('DB_PASSWORD'),
            'tool' => 'gh-ost', // or 'pt-online-schema-change'
            'gh-ost' => [
                'max-load' => 'Threads_running=50',
                'critical-load' => 'Threads_running=100',
                'chunk-size' => 1000,
            ],
        ],
    ],
];
```

```php
<?php
// Migration using online DDL

use Daursu\ZeroDowntimeMigration\ZeroDowntimeSchema;
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;

return new class extends Migration
{
    public function up(): void
    {
        ZeroDowntimeSchema::table('user_events', function (Blueprint $table) {
            $table->json('metadata')->nullable()->after('event_type');
        });
    }
};
```

**Expected Output:** The `metadata` column is added to the `user_events` table using gh-ost, which creates a shadow copy of the table, applies the schema change, and swaps the tables—all without locking the original table.

**Why:** gh-ost and pt-online-schema-change perform schema alterations on a copy of the table while the original remains fully available for reads and writes. This eliminates table locks and minimizes downtime for high-traffic tables.

---

**Example 3: Migration Guard for Pre-Deployment Safety**

```bash
# Install the migration guard package
composer require --dev malikad778/laravel-migration-guard

# Run the guard check before migrating
php artisan migrate:check

# Output:
# Analyzing pending migrations...
#
# ⚠️  DANGEROUS OPERATIONS DETECTED:
#
# 1. Migration: 2025_02_01_000000_drop_name_from_users_table
#    Operation: Schema::table('users', ...)->dropColumn('name')
#    Risk: Dropping a column may break running code during deployment.
#    Suggestion: Use the Expand-Contract pattern.
#
# 2. Migration: 2025_02_15_000000_add_index_to_orders_table
#    Operation: Schema::table('orders', ...)->index('status')
#    Risk: Adding an index to a large table may cause a table lock.
#    Suggestion: Use online DDL (gh-ost or pt-online-schema-change).
#
# Review the above risks before proceeding.
```

**Expected Output:** The migration guard analyses pending migrations and warns about dangerous operations before they run.

**Why:** Migration guard packages perform static analysis on migration files to detect operations that could cause downtime, data loss, or locking. This provides an additional safety layer beyond the `--force` flag.

### Real-World Cases

- **High-Traffic E-commerce:** Online DDL tools add columns to the `orders` table (millions of rows) without locking it.
- **Multi-Server SaaS:** The Expand-Contract pattern renames columns safely across rolling deployments.
- **Database Migrations at Scale:** Migration guard packages prevent dangerous migrations from reaching production.
- **Legacy System Modernisation:** Online DDL tools alter legacy tables without disrupting existing applications.
- **CI/CD Pipelines:** Migration guard checks are integrated into CI pipelines to catch dangerous operations before deployment.

### References

- Laravel Database: Migrations — Forcing Migrations in Production — https://laravel.com/docs/12.x/migrations#forcing-migrations-in-production
- Expand-Contract Pattern for Zero-Downtime Migrations (voku/agent-skills) — https://github.com/voku/agent-skills/blob/main/skills/laravel-database-optimization/rules/migrate-zero-downtime.md
- daursu/laravel-zero-downtime-migration (LaraPlugins) — https://laraplugins.io/daursu/laravel-zero-downtime-migration
- malikad778/laravel-migration-guard (Packagist) — https://packagist.org/packages/malikad778/laravel-migration-guard
- How to design a zero-downtime schema migration strategy (Stack Overflow) — https://stackoverflow.com/questions/79980863

---

## References

- Laravel Database: Migrations — Squashing Migrations — https://laravel.com/docs/12.x/migrations#squashing-migrations
- Laravel: New DB Commands (Laravel Blog) — https://laravel.com/blog/laravel-new-db-commands-and-more
- Laravel 9.24 Released (Laravel News) — https://laravel-news.com/laravel-9-24-0
- Expand-Contract Pattern for Zero-Downtime Migrations — https://github.com/voku/agent-skills/blob/main/skills/laravel-database-optimization/rules/migrate-zero-downtime.md
- daursu/laravel-zero-downtime-migration (LaraPlugins) — https://laraplugins.io/daursu/laravel-zero-downtime-migration
- malikad778/laravel-migration-guard (Packagist) — https://packagist.org/packages/malikad778/laravel-migration-guard
- How to design a zero-downtime schema migration strategy (Stack Overflow) — https://stackoverflow.com/questions/79980863
- Laravel Data Dumper Package (Dragon Code) — https://github.com/TheDragonCode/laravel-data-dumper
- Laravel Strong Migrations Package (grazulex) — https://packagist.org/packages/grazulex/laravel-strongmigrations
- Prevent Destructive Commands Package — https://laraplugins.io/mattyeend/prevent-destructive-commands