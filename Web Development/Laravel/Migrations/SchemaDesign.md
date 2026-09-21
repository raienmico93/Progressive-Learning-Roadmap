# Laravel Schema Design & Fluent Column Blueprinting: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Schema Design is the database-agnostic, fluent PHP DSL for defining, modifying, and dropping database tables, columns, indexes, and foreign key constraints through the `Schema` facade and the `Blueprint` class.

**Technical Definition:** The `Illuminate\Database\Schema\Builder` class (accessed via the `Schema` facade) provides high-level methods for table operations (`create`, `table`, `drop`, `rename`). The `Illuminate\Database\Schema\Blueprint` class provides the fluent DSL for defining columns, indexes, and constraints. Blueprint definitions are compiled into database-specific SQL DDL by driver-specific `Grammar` classes (MySqlGrammar, PostgresGrammar, SQLiteGrammar, SqlServerGrammar). Column type methods return `ColumnDefinition` instances that accept modifier chains (`nullable()`, `default()`, etc.). All schema definitions are typically expressed inside migration files, executed by `php artisan migrate`.

**Beginner-Friendly Explanation:** Laravel's Schema Builder lets you design your database tables using PHP code instead of writing SQL by hand. You use `Schema::create()` to make a table, then define its columns with methods like `$table->string('name')` and `$table->integer('votes')`. It works with MySQL, PostgreSQL, SQLite, and SQL Server — the same code produces the correct SQL for each database. Because it's PHP, you can version-control your database structure and share it with your team.

### Key Characteristics

- **Database Agnostic:** The same schema definition works across MySQL, PostgreSQL, SQLite, SQL Server, and MariaDB.
- **Fluent DSL:** Column definitions and modifiers are chainable: `$table->string('email')->unique()->nullable()`.
- **ColumnDefinition Return:** Every column type method returns a `ColumnDefinition` instance, enabling modifier chaining.
- **Grammar Compilation:** Blueprint definitions are compiled into database-specific SQL by Grammar classes.
- **Migration Integration:** Schema definitions are typically written inside migrations, enabling version-controlled schema changes.
- **Index & Constraint Support:** Primary keys, unique constraints, standard indexes, full-text indexes, foreign keys, and raw expression indexes are all supported.
- **Modern Column Types:** UUID, ULID, JSON, geometry, and spatial types are supported alongside traditional types.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (the Schema Builder API is stable across versions; ULID support since Laravel 9.x; raw indexes since Laravel 10.x).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A configured database connection in `config/database.php` and `.env`.
- Basic understanding of SQL DDL and database design concepts.

### Related Programming Areas

- **Migrations:** Schema definitions live inside migration files, executed in version-controlled order.
- **Eloquent ORM:** Eloquent models depend on the database schema defined by the Schema Builder.
- **Database Seeding:** Seeders populate tables after schema creation.
- **Query Builder:** The Query Builder operates on the schema defined by the Schema Builder.
- **Performance Tuning:** Indexes and foreign keys defined via the Schema Builder directly impact query performance.

### Core Concepts / Features

1. Table Management
2. Column Types
3. Column Modifiers
4. Keys & Relational Constraints
5. Performance Modifiers

---

## 1. Table Management

### Definitions

**Core Definition:** Table management in the Schema Builder encompasses the methods for creating new tables, modifying existing tables, renaming tables, and dropping tables from the database schema.

**Technical Definition:** The `Schema::create($table, Closure $callback)` method creates a new table, passing a `Blueprint` instance to the callback for column and index definition. The `Schema::table($table, Closure $callback)` method modifies an existing table. The `Schema::rename($from, $to)` method renames a table. The `Schema::drop($table)` and `Schema::dropIfExists($table)` methods remove tables. These operations are compiled into database-specific DDL statements by the active Grammar class.

**Beginner-Friendly Explanation:** Creating a table is like drawing a blueprint for a new spreadsheet. `Schema::create()` builds the table, `Schema::table()` modifies it (adds or removes columns), `Schema::rename()` changes its name, and `Schema::drop()` deletes it. Always use `dropIfExists()` to avoid errors if the table doesn't exist.

### Purposes

- To create new database tables with `Schema::create()`.
- To modify existing tables (add columns, indexes, constraints) with `Schema::table()`.
- To rename tables without data loss with `Schema::rename()`.
- To drop tables with `Schema::drop()` or `Schema::dropIfExists()`.
- To check table existence with `Schema::hasTable()`.
- To check column existence with `Schema::hasColumn()`.

### Syntax Rules and Structure

**Complete General Syntax — Creating a Table:**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamps();
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Schema::create('users', ...)` | Creates a new table named `users`. |
| `function (Blueprint $table)` | Callback receiving the Blueprint instance. |
| `$table->id()` | Auto-incrementing primary key. |
| `$table->string('name')` | VARCHAR column. |
| `$table->timestamps()` | `created_at` and `updated_at` columns. |

**Complete General Syntax — Modifying a Table:**

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->nullable()->after('email');
    $table->dropColumn('legacy_field');
});
```

**Complete General Syntax — Renaming a Table:**

```php
Schema::rename('users', 'members');
```

**Complete General Syntax — Dropping a Table:**

```php
Schema::drop('users');
Schema::dropIfExists('users');
```

**Syntax Rules:**

- `Schema::create()` requires the table name and a Closure receiving a `Blueprint`.
- `Schema::table()` modifies an existing table; use `$table->dropColumn()` to remove columns.
- `Schema::rename()` preserves table data; use with caution in production.
- `Schema::dropIfExists()` is safer than `Schema::drop()` because it doesn't error if the table is missing.
- Table names should be plural and snake_case by Laravel convention.
- The `Schema::hasTable()` and `Schema::hasColumn()` methods check existence without modifying the schema.

**Constraints and Limitations:**

- **Renaming tables with foreign keys:** Before renaming a table that contains foreign keys, ensure the foreign key constraints have explicit names; otherwise, constraint names will still reference the old table name.
- **Dropping tables with foreign keys:** Drop dependent foreign keys before dropping a table, or use `Schema::disableForeignKeyConstraints()`.
- **`Schema::table()` performance:** On large tables, altering columns can lock the table for extended periods. Use online DDL tools for production migrations.

### Multiple Annotated Complete Code Examples

**Example 1: Creating a Complete Table**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();                              // BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
            $table->string('name');                    // VARCHAR(255)
            $table->string('airline');                 // VARCHAR(255)
            $table->string('airport');                 // VARCHAR(255)
            $table->timestamp('departed_at')->nullable();
            $table->timestamps();                      // created_at, updated_at
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('flights');
    }
};
```

**Expected Output:** Running `php artisan migrate` creates the `flights` table with the specified columns.

**Why:** The `Schema::create()` method defines the table structure using the Blueprint DSL. The `down()` method uses `dropIfExists()` for safe rollback.

---

**Example 2: Modifying an Existing Table**

```php
Schema::table('users', function (Blueprint $table) {
    // Add a new column after the email column
    $table->string('phone', 20)->nullable()->after('email');

    // Add an index
    $table->index('phone');

    // Drop an unused column
    $table->dropColumn('legacy_token');
});
```

**Expected Output:** The `users` table gains a `phone` column and loses the `legacy_token` column.

**Why:** The `Schema::table()` method modifies the existing table. The `after()` modifier positions the new column; `dropColumn()` removes the old column.

---

**Example 3: Renaming a Table**

```php
Schema::rename('users', 'members');

// Verify the table was renamed
if (Schema::hasTable('members')) {
    echo 'Table renamed successfully.';
}
```

**Expected Output:** The `users` table is renamed to `members`; `Schema::hasTable('members')` returns `true`.

**Why:** `Schema::rename()` performs a database-level table rename, preserving all data and structure.

### Real-World Cases

- **User Management:** `Schema::create('users', ...)` for the core authentication table.
- **E-commerce:** `Schema::create('orders', ...)` with foreign keys to users and products.
- **Content Management:** `Schema::create('posts', ...)` with soft-delete support.
- **Multi-Tenancy:** `Schema::create('tenants', ...)` for tenant management.
- **Audit Logging:** `Schema::create('activity_log', ...)` with polymorphic relationships.

### References

- Laravel Database: Migrations — Creating Tables — https://laravel.com/docs/12.x/migrations#creating-tables
- Laravel Database: Migrations — Updating Tables — https://laravel.com/docs/12.x/migrations#updating-tables
- Laravel Database: Migrations — Renaming / Dropping Tables — https://laravel.com/docs/12.x/migrations#renaming-and-dropping-tables
- DeepWiki: Schema Builder and Migrations — https://deepwiki.com/laravel/framework/2.3.4-schema-builder-and-migrations

---

## 2. Column Types

### Definitions

**Core Definition:** Column types in the Schema Builder are the Blueprint methods that define the data type of each column, including strings, text, integers, decimals, booleans, dates, JSON, UUIDs, ULIDs, and spatial types.

**Technical Definition:** The `Blueprint` class provides one method per supported column type. Each method returns a `ColumnDefinition` instance that accepts modifier chains. The methods are categorised as: **integers** (`tinyInteger`, `smallInteger`, `integer`, `bigInteger`), **strings** (`char`, `string`, `text`, `mediumText`, `longText`), **decimals** (`float`, `double`, `decimal`), **dates** (`date`, `dateTime`, `time`, `timestamp`), and **special** (`boolean`, `json`, `enum`, `uuid`, `ulid`, `vector`).

**Beginner-Friendly Explanation:** Column types tell the database what kind of data each column holds. A `string` column holds short text like names. A `text` column holds long content like blog posts. An `integer` holds whole numbers. A `boolean` holds true/false. A `json` column holds structured data. A `uuid` holds a universally unique identifier. Choosing the right type ensures data integrity and optimal performance.

### Purposes

- To define the data type of each column in a table.
- To enforce data integrity at the database level through appropriate types.
- To optimise storage and query performance with correctly sized types.
- To support modern identifiers (UUID, ULID) as alternatives to auto-incrementing integers.
- To store structured data (JSON) alongside traditional relational data.
- To support spatial data for geographic applications.

### Syntax Rules and Structure

**Complete General Syntax — Common Column Types:**

```php
$table->id();                          // BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
$table->string('name', 100);          // VARCHAR(100)
$table->text('description');           // TEXT
$table->integer('votes');              // INTEGER
$table->decimal('amount', 10, 2);     // DECIMAL(10,2)
$table->boolean('confirmed');          // BOOLEAN
$table->json('options');               // JSON
$table->uuid('id');                    // UUID (CHAR(36))
$table->ulid('id');                    // ULID (CHAR(26))
$table->timestamp('created_at');       // TIMESTAMP
```

**Component Breakdown:**

| Method | SQL Equivalent | Description |
|--------|----------------|-------------|
| `$table->id()` | `BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Primary key shortcut. |
| `$table->string('name', 100)` | `VARCHAR(100)` | Variable-length string. |
| `$table->text('description')` | `TEXT` | Long text. |
| `$table->integer('votes')` | `INTEGER` | 32-bit integer. |
| `$table->decimal('amount', 10, 2)` | `DECIMAL(10,2)` | Fixed-point number. |
| `$table->boolean('confirmed')` | `BOOLEAN` | True/false. |
| `$table->json('options')` | `JSON` | JSON document. |
| `$table->uuid('id')` | `CHAR(36)` | UUID identifier. |
| `$table->ulid('id')` | `CHAR(26)` | ULID identifier. |

**Complete General Syntax — Available Column Types (Extended):**

```php
// Integers
$table->bigIncrements('id');
$table->bigInteger('votes');
$table->tinyInteger('votes');
$table->smallInteger('votes');
$table->mediumInteger('votes');

// Strings
$table->char('name', 4);
$table->mediumText('description');
$table->longText('description');

// Decimals
$table->float('amount', 8, 2);
$table->double('column', 15, 8);

// Dates
$table->date('created_at');
$table->dateTime('created_at');
$table->dateTimeTz('created_at');
$table->time('sunrise');
$table->timeTz('sunrise');

// Special
$table->enum('level', ['easy', 'hard']);
$table->ipAddress('visitor');
$table->macAddress('device');
$table->geometry('positions');
$table->geometryCollection('positions');
$table->jsonb('options');
$table->lineString('positions');
$table->multiLineString('positions');
$table->multiPoint('positions');
$table->multiPolygon('positions');
$table->point('position');
$table->polygon('positions');
$table->year('birth_year');
```

**Syntax Rules:**

- Every column type method returns a `ColumnDefinition` instance for modifier chaining.
- String length defaults to 255 if not specified.
- The `id()` method is a shortcut for `bigIncrements('id')`.
- UUID and ULID columns are 36 and 26 characters respectively.
- The `json()` type is supported by MySQL 5.7+, PostgreSQL, SQLite 3.38+, and SQL Server.
- Geometry types are supported by MySQL and PostgreSQL; SQLite does not support spatial types.

**Constraints and Limitations:**

- **String length limits:** MySQL's default index length limit may require `Schema::defaultStringLength(191)` for older MySQL versions with `utf8mb4`.
- **JSON column indexing:** JSON columns cannot be directly indexed; use raw indexes on extracted paths.
- **ULID naming bug:** In some versions, `$table->ulid()` without arguments registers a column named `uuid` instead of `ulid`.
- **Enum limitations:** PostgreSQL does not support native ENUM types in the same way as MySQL; Laravel uses a VARCHAR with a check constraint.

### Multiple Annotated Complete Code Examples

**Example 1: Standard Column Types**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();                              // Primary key
    $table->string('name', 200);              // VARCHAR(200)
    $table->string('sku')->unique();           // Unique VARCHAR
    $table->text('description');               // TEXT
    $table->decimal('price', 10, 2);           // DECIMAL(10,2)
    $table->integer('stock');                  // INTEGER
    $table->boolean('is_active')->default(true);
    $table->json('metadata')->nullable();      // JSON
    $table->timestamps();
});
```

**Expected Output:** The `products` table is created with all specified column types.

**Why:** Each column type method maps to the appropriate SQL data type for the active database driver.

---

**Example 2: UUID and ULID Primary Keys**

```php
// UUID primary key
Schema::create('users', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('name');
    $table->timestamps();
});

// ULID primary key
Schema::create('posts', function (Blueprint $table) {
    $table->ulid('id')->primary();
    $table->string('title');
    $table->timestamps();
});
```

**Expected Output:** The `users` table uses a UUID primary key; the `posts` table uses a ULID primary key.

**Why:** UUIDs and ULIDs are alternatives to auto-incrementing integers, useful for distributed systems where global uniqueness is required.

---

**Example 3: Enum and Special Types**

```php
Schema::create('subscriptions', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained();
    $table->enum('status', ['active', 'cancelled', 'expired'])->default('active');
    $table->ipAddress('last_login_ip')->nullable();
    $table->macAddress('device_mac')->nullable();
    $table->year('start_year');
    $table->timestamps();
});
```

**Expected Output:** The `subscriptions` table is created with an enum status column, IP address, MAC address, and year columns.

**Why:** Enum columns constrain values to a predefined set. IP and MAC address columns are optimised for network data.

### Real-World Cases

- **User Tables:** `$table->string('email')->unique()` for authentication.
- **Financial Data:** `$table->decimal('amount', 10, 2)` for monetary values.
- **Configuration:** `$table->json('settings')` for flexible configuration storage.
- **Distributed Systems:** `$table->uuid('id')->primary()` for globally unique identifiers.
- **Geographic Applications:** `$table->point('location')` for spatial data.

### References

- Laravel Database: Migrations — Available Column Types — https://laravel.com/docs/12.x/migrations#available-column-types
- Laravel API: Blueprint — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/Blueprint.html
- Laravel Daily: UUID vs ULID — https://laraveldaily.com/post/uuid-or-ulid
- DeepWiki: Schema Builder Column Types — https://deepwiki.com/laravel/framework/2.3.4-schema-builder-and-migrations

---

## 3. Column Modifiers

### Definitions

**Core Definition:** Column modifiers are methods chained onto column type methods to further define column behaviour, including nullability, default values, character sets, auto-increment settings, and positioning.

**Technical Definition:** The `ColumnDefinition` class (returned by every column type method) provides modifier methods that set attributes on the column definition. These modifiers are compiled by the Grammar into the appropriate SQL clauses. Available modifiers include `nullable()`, `default()`, `charset()`, `collation()`, `autoIncrement()`, `unsigned()`, `comment()`, `after()`, `first()`, `useCurrent()`, `storedAs()`, `virtualAs()`, and `invisible()`.

**Beginner-Friendly Explanation:** Column modifiers are like extra settings on a column. `nullable()` means the column can be empty. `default()` sets a starting value. `charset()` specifies the character encoding. `after()` positions the column after another one. You chain them onto the column type: `$table->string('email')->nullable()->after('name')`.

### Purposes

- To allow NULL values in a column using `nullable()`.
- To set a default value for a column using `default()`.
- To specify a character set for a column using `charset()`.
- To specify a collation for sorting and comparison using `collation()`.
- To set integer columns as unsigned or auto-incrementing using `unsigned()` and `autoIncrement()`.
- To position columns in the table using `after()` and `first()`.
- To add comments to columns using `comment()`.

### Syntax Rules and Structure

**Complete General Syntax — Common Modifiers:**

```php
$table->string('email')->nullable()->unique();
$table->string('status')->default('active');
$table->string('name')->charset('utf8mb4')->collation('utf8mb4_unicode_ci');
$table->integer('votes')->unsigned();
$table->string('address')->after('city');
$table->string('code')->first();
$table->string('notes')->comment('Internal notes');
```

**Component Breakdown:**

| Modifier | Description |
|----------|-------------|
| `->nullable($value = true)` | Allows NULL values (default). |
| `->default($value)` | Sets a default value. |
| `->charset('utf8mb4')` | Sets the character set (MySQL). |
| `->collation('utf8mb4_unicode_ci')` | Sets the collation (MySQL/PostgreSQL/SQL Server). |
| `->unsigned()` | Sets INTEGER columns as UNSIGNED (MySQL). |
| `->autoIncrement()` | Sets INTEGER columns as auto-incrementing (primary key). |
| `->after('column')` | Places the column after another column (MySQL). |
| `->first()` | Places the column first in the table (MySQL). |
| `->comment('...')` | Adds a comment to the column (MySQL/PostgreSQL). |
| `->useCurrent()` | Sets TIMESTAMP columns to use CURRENT_TIMESTAMP as default. |
| `->invisible()` | Makes the column invisible to SELECT * (MySQL). |

**Complete General Syntax — Default Expressions:**

```php
use Illuminate\Database\Query\Expression;

$table->json('movies')->default(new Expression('(JSON_ARRAY())'));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `new Expression('(JSON_ARRAY())')` | Raw SQL expression as default value. |
| Use case | Database-specific functions that cannot be expressed as literal values. |

**Syntax Rules:**

- All modifiers must be chained onto a column type method.
- `nullable()` accepts an optional boolean; `nullable(false)` makes the column NOT NULL.
- `default()` accepts a value or an `Expression` instance.
- `after()` and `first()` are MySQL-specific positioning modifiers.
- Modifiers are order-independent for most operations, but `constrained()` must come after other modifiers on foreign key columns.

**Constraints and Limitations:**

- **Default expressions:** Support for default expressions depends on the database driver, version, and field type.
- **Column positioning:** `after()` and `first()` are MySQL-only. Other databases do not support positional column modification.
- **`change()` requires all modifiers:** When modifying a column with `->change()`, you must explicitly include all modifiers you want to keep.
- **Charset/collation:** Primarily MySQL-specific; PostgreSQL and SQL Server have different mechanisms.

### Multiple Annotated Complete Code Examples

**Example 1: Nullable and Default Values**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->string('phone')->nullable();
    $table->string('role')->default('user');
    $table->boolean('is_active')->default(true);
    $table->timestamp('last_login_at')->nullable();
    $table->timestamps();
});
```

**Expected Output:** The `users` table is created with `phone` nullable, `role` defaulting to `user`, and `is_active` defaulting to `true`.

**Why:** `nullable()` allows NULL values; `default()` sets the initial value when no value is provided.

---

**Example 2: Charset and Collation**

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title')->charset('utf8mb4')->collation('utf8mb4_unicode_ci');
    $table->text('body')->charset('utf8mb4');
    $table->timestamps();
});
```

**Expected Output:** The `articles` table uses `utf8mb4` charset and `utf8mb4_unicode_ci` collation for the title column.

**Why:** `charset()` and `collation()` control character encoding and sorting behaviour, important for internationalisation.

---

**Example 3: Unsigned and Auto-Increment**

```php
Schema::create('counters', function (Blueprint $table) {
    $table->id();
    $table->unsignedInteger('count')->default(0);
    $table->unsignedBigInteger('total_views')->default(0);
    $table->tinyInteger('rating')->unsigned();
    $table->timestamps();
});
```

**Expected Output:** The `counters` table is created with unsigned integer columns that cannot hold negative values.

**Why:** Unsigned integers double the positive range of the column and are appropriate for values that should never be negative (counts, IDs, ratings).

### Real-World Cases

- **User Profiles:** `nullable()` for optional fields like phone or bio.
- **Status Tracking:** `default('pending')` for order or task status columns.
- **Internationalisation:** `charset('utf8mb4')` for columns storing emoji or multilingual text.
- **Counters:** `unsignedInteger('views')->default(0)` for view counters.
- **Audit Fields:** `useCurrent()` for timestamp columns that default to the current time.

### References

- Laravel Database: Migrations — Column Modifiers — https://laravel.com/docs/12.x/migrations#column-modifiers
- Laravel API: ColumnDefinition — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/ColumnDefinition.html
- Laravel Daily: Column Modifiers — https://laraveldaily.com/post/column-modifiers

---

## 4. Keys & Relational Constraints

### Definitions

**Core Definition:** Keys and relational constraints in the Schema Builder define primary keys, unique constraints, foreign key relationships, and composite keys that enforce data integrity and define relationships between tables.

**Technical Definition:** The `primary()` method defines a primary key, accepting a single column or an array for composite keys. The `unique()` method creates a unique constraint. The `foreignId()` method creates an unsigned BIGINT column; when chained with `constrained()`, it creates a foreign key constraint using convention to determine the referenced table and column. The `onDelete()` and `onUpdate()` methods specify referential actions (`cascade`, `set null`, `restrict`, `no action`). The `foreign()` method provides explicit foreign key definition when convention does not apply.

**Beginner-Friendly Explanation:** Keys and constraints are rules that keep your data consistent. A primary key is the unique identifier for each row. A unique constraint means no two rows can have the same value in that column. A foreign key is a link between two tables — it says "this column's value must match an ID in another table." For example, a post's `user_id` must match an existing user's `id`. If you delete the user, the database can automatically delete their posts (cascade) or set the post's user_id to NULL.

### Purposes

- To define primary keys that uniquely identify rows using `primary()`.
- To enforce unique values in columns using `unique()`.
- To create foreign key relationships between tables using `foreignId()->constrained()`.
- To define composite primary keys for pivot tables using `primary([...])`.
- To specify referential actions (cascade, set null, restrict) with `onDelete()` and `onUpdate()`.
- To drop foreign keys and constraints with `dropForeign()`, `dropUnique()`, and `dropPrimary()`.

### Syntax Rules and Structure

**Complete General Syntax — Primary Key:**

```php
$table->primary('id');
$table->primary(['user_id', 'role_id']);  // Composite primary key
```

**Complete General Syntax — Unique Constraint:**

```php
$table->unique('email');
$table->unique(['first_name', 'last_name']);
$table->unique('email', 'unique_email_index');
```

**Complete General Syntax — Foreign Key with Convention:**

```php
$table->foreignId('user_id')->constrained();
$table->foreignId('user_id')->constrained('users');
$table->foreignId('user_id')->constrained()->onDelete('cascade');
```

**Complete General Syntax — Foreign Key with Explicit Definition:**

```php
$table->unsignedBigInteger('user_id');
$table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
```

**Complete General Syntax — Referential Actions:**

```php
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
$table->foreignId('user_id')->constrained()->nullOnDelete();
$table->foreignId('user_id')->constrained()->restrictOnDelete();
$table->foreignId('user_id')->constrained()->cascadeOnUpdate();
$table->foreignId('user_id')->constrained()->onDelete('set null');
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `foreignId('user_id')` | Creates an unsigned BIGINT column. |
| `constrained()` | Creates a foreign key to the `users` table (by convention). |
| `constrained('users')` | Specifies the referenced table explicitly. |
| `cascadeOnDelete()` | Deletes child rows when the parent is deleted. |
| `nullOnDelete()` | Sets child foreign key to NULL when the parent is deleted. |
| `restrictOnDelete()` | Prevents deletion of the parent if children exist. |
| `onDelete('set null')` | Same as `nullOnDelete()`, using explicit syntax. |

**Syntax Rules:**

- `foreignId()` is an alias for `unsignedBigInteger()`.
- `constrained()` uses convention: `user_id` references `id` on the `users` table.
- Any column modifiers (`nullable()`, `default()`) must be called **before** `constrained()`.
- Foreign key constraint names follow the convention `{table}_{column}_foreign`.
- Composite primary keys are supported for pivot tables but not by Eloquent models.
- SQLite requires foreign key constraints to be defined at table creation time (not via ALTER TABLE).

**Constraints and Limitations:**

- **Composite primary keys and Eloquent:** Eloquent does not support composite primary keys natively. Use a unique constraint instead, or a package like `dyrynda/eloquent-composite-primary-keys`.
- **SQLite foreign keys:** SQLite disables foreign key constraints by default. Enable them in `config/database.php` before creating them.
- **Modifier order:** `nullable()` and other modifiers must precede `constrained()`; otherwise, the foreign key constraint may fail.
- **Dropping foreign keys:** Drop the foreign key before dropping the column or table.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Foreign Key Relationship**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->string('title');
    $table->text('body');
    $table->timestamps();
});
```

**Expected Output:** The `posts` table is created with a `user_id` foreign key referencing the `users` table. Deleting a user automatically deletes their posts.

**Why:** `foreignId('user_id')->constrained()` creates the foreign key using convention. `cascadeOnDelete()` enforces referential integrity by deleting child records.

---

**Example 2: Composite Primary Key for Pivot Table**

```php
Schema::create('role_user', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
    $table->foreignId('role_id')->constrained();
    $table->primary(['user_id', 'role_id']);  // Composite primary key
    $table->timestamps();
});
```

**Expected Output:** The `role_user` pivot table has a composite primary key on `user_id` and `role_id`, ensuring each user-role combination is unique.

**Why:** Composite primary keys are standard for pivot tables. Eloquent does not support them directly, but the database enforces uniqueness.

---

**Example 3: Foreign Key with Explicit Table and Referential Actions**

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->foreignId('customer_id')
          ->nullable()
          ->constrained('customers')
          ->nullOnDelete()
          ->cascadeOnUpdate();
    $table->decimal('total', 10, 2);
    $table->timestamps();
});
```

**Expected Output:** The `orders` table has a nullable foreign key to `customers`. Deleting a customer sets the order's `customer_id` to NULL; updating the customer's ID cascades the change.

**Why:** `nullable()` allows orders without a customer. `constrained('customers')` specifies the referenced table explicitly. `nullOnDelete()` and `cascadeOnUpdate()` define referential actions.

### Real-World Cases

- **User Posts:** `$table->foreignId('user_id')->constrained()->cascadeOnDelete()` for user-authored posts.
- **Order Items:** `$table->foreignId('order_id')->constrained()->cascadeOnDelete()` for line items.
- **Pivot Tables:** `$table->primary(['user_id', 'role_id'])` for many-to-many relationships.
- **Soft-Deletable Parents:** `$table->foreignId('category_id')->nullable()->constrained()->nullOnDelete()` for optional relationships.
- **Audit Trails:** `$table->foreignId('created_by')->constrained('users')->restrictOnDelete()` to preserve audit records.

### References

- Laravel Database: Migrations — Foreign Key Constraints — https://laravel.com/docs/12.x/migrations#foreign-key-constraints
- Laravel Database: Migrations — Indexes — https://laravel.com/docs/12.x/migrations#indexes
- Laravel Daily: Foreign Key Constraints — https://laraveldaily.com/post/foreign-key-constraints
- Laravel News: Composite Primary Keys Package — https://laravel-news.com/composite-primary-keys

---

## 5. Performance Modifiers

### Definitions

**Core Definition:** Performance modifiers in the Schema Builder are the index types — standard indexes, unique constraints, full-text indexes, spatial indexes, and raw expression indexes — that optimise query performance and enforce data integrity.

**Technical Definition:** The `index()` method creates a standard B-tree index. The `unique()` method creates a unique constraint (which also creates an index). The `fulltext()` method creates a full-text index (MySQL/PostgreSQL). The `spatialIndex()` method creates a spatial index (MySQL/PostgreSQL, not SQLite). The `rawIndex()` method creates an index using a raw SQL expression (available in Laravel 10.x+). All index methods accept an optional second argument for the index name; if omitted, Laravel generates a name based on the table, columns, and index type.

**Beginner-Friendly Explanation:** Indexes are like the index at the back of a book — they help the database find rows quickly without scanning the entire table. A standard index speeds up `WHERE` and `ORDER BY` queries. A unique index enforces that no two rows have the same value. A full-text index is specialised for searching text content. A raw index lets you index a calculated expression (like `LOWER(email)`).

### Purposes

- To speed up `WHERE`, `ORDER BY`, and `JOIN` queries with standard indexes via `index()`.
- To enforce uniqueness and speed up lookups with `unique()`.
- To enable full-text search with `fulltext()` on text columns.
- To optimise spatial queries with `spatialIndex()`.
- To create functional indexes on calculated expressions with `rawIndex()`.
- To drop indexes when they are no longer needed with `dropIndex()`, `dropUnique()`, etc.

### Syntax Rules and Structure

**Complete General Syntax — Standard Index:**

```php
$table->index('state');
$table->index(['account_id', 'created_at']);  // Composite index
$table->index('state', 'my_state_index');     // Named index
```

**Complete General Syntax — Unique Constraint:**

```php
$table->unique('email');
$table->unique(['first_name', 'last_name']);
$table->unique('email', 'unique_email');
```

**Complete General Syntax — Full-Text Index:**

```php
$table->fulltext('body');
$table->fulltext('body')->language('english');
$table->fulltext(['title', 'body']);
```

**Complete General Syntax — Spatial Index:**

```php
$table->spatialIndex('location');
```

**Complete General Syntax — Raw Expression Index:**

```php
$table->rawIndex("(data->'items')", 'items_index');
$table->rawIndex('(LOWER(email))', 'email_lower_index');
$table->rawIndex("DATE_TRUNC('year'::text, created_at)", 'radar_records_created_at_trunc_year_idx');
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `index('column')` | Creates a standard B-tree index. |
| `unique('column')` | Creates a unique constraint and index. |
| `fulltext('column')` | Creates a full-text index (MySQL/PostgreSQL). |
| `spatialIndex('column')` | Creates a spatial index (not SQLite). |
| `rawIndex('expression', 'name')` | Creates an index on a raw SQL expression. |

**Complete General Syntax — Dropping Indexes:**

```php
$table->dropPrimary('users_id_primary');
$table->dropUnique('users_email_unique');
$table->dropIndex('geo_state_index');
$table->dropSpatialIndex('geo_location_spatialindex');
$table->dropFullText('posts_body_fulltext');
```

**Syntax Rules:**

- Index names are auto-generated if not provided, based on table, column(s), and index type.
- Composite indexes are created by passing an array of column names.
- Full-text indexes are supported by MySQL and PostgreSQL only.
- Spatial indexes are supported by MySQL and PostgreSQL; SQLite does not support them.
- Raw indexes are supported in Laravel 10.x and later.
- Index column order matters for composite indexes: the leftmost column is used for lookups.

**Constraints and Limitations:**

- **MySQL index length:** MySQL's default index length (767 bytes for InnoDB) may be exceeded with `utf8mb4` strings longer than 191 characters. Use `Schema::defaultStringLength(191)` or enable `innodb_large_prefix`.
- **Full-text index limitations:** Full-text indexes are not supported by SQLite and have limited support in SQL Server.
- **Raw index portability:** Raw index expressions are database-specific and may not work across different database systems.
- **Index overhead:** Every index adds write overhead (INSERT/UPDATE/DELETE must update the index). Avoid over-indexing.

### Multiple Annotated Complete Code Examples

**Example 1: Standard and Unique Indexes**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('email')->unique();           // Unique index
    $table->string('username')->unique();
    $table->string('status');
    $table->index('status');                      // Standard index
    $table->index(['last_name', 'first_name']);  // Composite index
    $table->timestamps();
});
```

**Expected Output:** The `users` table is created with unique indexes on `email` and `username`, and a composite index on `last_name` and `first_name`.

**Why:** Unique indexes enforce uniqueness; standard indexes speed up lookups; composite indexes optimise queries that filter or sort by multiple columns.

---

**Example 2: Full-Text Index for Search**

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('body');
    $table->timestamps();
});

// Add full-text index in a separate migration
Schema::table('articles', function (Blueprint $table) {
    $table->fulltext(['title', 'body']);
});
```

```php
// Query using full-text search
$articles = DB::table('articles')
    ->whereFullText(['title', 'body'], 'laravel')
    ->get();
```

**Expected Output:** The `articles` table has a full-text index on `title` and `body`. `whereFullText()` performs a fast full-text search.

**Why:** Full-text indexes are optimised for natural language search, much faster than `LIKE '%term%'` on large text columns.

---

**Example 3: Raw Index on JSON Column**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->json('attributes');
    $table->timestamps();
});

// Create an index on a JSON path
Schema::table('products', function (Blueprint $table) {
    $table->rawIndex("(attributes->>'$.color')", 'products_color_idx');
});
```

**Expected Output:** The `products` table has a raw index on the `color` key within the JSON `attributes` column.

**Why:** Raw indexes allow indexing of JSON paths or calculated expressions, enabling fast lookups on nested data.

### Real-World Cases

- **Authentication:** `$table->unique('email')` for login lookups.
- **Search:** `$table->fulltext(['title', 'body'])` for blog or product search.
- **Geographic Apps:** `$table->spatialIndex('location')` for distance queries.
- **Composite Filtering:** `$table->index(['status', 'created_at'])` for admin dashboards.
- **JSON Data:** `$table->rawIndex("(attributes->>'$.sku')", 'sku_idx')` for indexing JSON fields.

### References

- Laravel Database: Migrations — Indexes — https://laravel.com/docs/12.x/migrations#indexes
- Laravel Database: Migrations — Full-Text Indexes — https://laravel.com/docs/12.x/migrations#full-text-indexes
- Laravel Database: Migrations — Raw Indexes — https://laravel.com/docs/12.x/migrations#raw-indexes
- Laravel Daily: Laravel Indexes — https://laraveldaily.com/post/laravel-indexes
- Laravel News: Full-Text Search in Laravel — https://laravel-news.com/full-text-search

---

## References

- Laravel Database: Migrations (12.x) — https://laravel.com/docs/12.x/migrations
- Laravel Database: Migrations — Creating Tables — https://laravel.com/docs/12.x/migrations#creating-tables
- Laravel Database: Migrations — Updating Tables — https://laravel.com/docs/12.x/migrations#updating-tables
- Laravel Database: Migrations — Available Column Types — https://laravel.com/docs/12.x/migrations#available-column-types
- Laravel Database: Migrations — Column Modifiers — https://laravel.com/docs/12.x/migrations#column-modifiers
- Laravel Database: Migrations — Indexes — https://laravel.com/docs/12.x/migrations#indexes
- Laravel Database: Migrations — Foreign Key Constraints — https://laravel.com/docs/12.x/migrations#foreign-key-constraints
- Laravel API: Blueprint — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/Blueprint.html
- Laravel API: ColumnDefinition — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/ColumnDefinition.html
- Laravel API: Builder — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/Builder.html
- DeepWiki: Schema Builder and Migrations — https://deepwiki.com/laravel/framework/2.3.4-schema-builder-and-migrations
- Laravel Daily: Column Modifiers — https://laraveldaily.com/post/column-modifiers
- Laravel Daily: UUID or ULID — https://laraveldaily.com/post/uuid-or-ulid
- Laravel News: Composite Primary Keys Package — https://laravel-news.com/composite-primary-keys
- Laravel News: Full-Text Search in Laravel — https://laravel-news.com/full-text-search
- Packagist: dyrynda/eloquent-composite-primary-keys — https://packagist.org/packages/dyrynda/eloquent-composite-primary-keys
- Stack Overflow: Composite Primary Key in Laravel — https://stackoverflow.com/questions/28394251
- GitHub: Laravel Framework Issue #53936 — Raw Index Handling — https://github.com/laravel/framework/issues/53936
- GitHub: Laravel Framework Pull Request #56748 — Configurable Default ID Type — https://github.com/laravel/framework/pull/56748