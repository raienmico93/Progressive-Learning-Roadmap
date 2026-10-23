# Laravel Indexes & Foreign Keys: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Indexes and foreign keys are database schema constructs defined within Laravel migrations that enforce data integrity, accelerate query performance, and establish relationships between tables. Indexes are data structures that speed up data retrieval, while foreign keys are constraints that enforce referential integrity between related tables.

**Technical Definition**

In Laravel's Schema Builder, indexes are created via methods on the `Blueprint` class (`primary()`, `unique()`, `index()`, `fullText()`, `spatialIndex()`), which compile to the corresponding DDL statements for the target database driver. Foreign keys are created via `foreign()` (or the fluent `foreignId()->constrained()`) and compile to `ALTER TABLE ... ADD CONSTRAINT ... FOREIGN KEY` statements with `ON DELETE` and `ON UPDATE` referential actions. Laravel automatically generates index and constraint names based on the table name, column(s), and type, but explicit names can be provided. Indexes and foreign keys can be dropped using dedicated `drop*` methods.

**Beginner-Friendly Explanation**

Indexes are like the index at the back of a book—they help the database find information quickly without scanning every row. Foreign keys are like rules that say "this record must belong to an existing record in another table." For example, a `posts` table might have a `user_id` column that must match an existing `users.id`—that's a foreign key. Indexes make lookups fast; foreign keys keep your data consistent.

---

### Key Characteristics

- **Database-Agnostic API**: A single set of PHP methods generates the correct index and foreign key SQL for MySQL, PostgreSQL, SQLite, and SQL Server.
- **Automatic Naming**: Laravel derives index and constraint names from the table, column(s), and type (e.g., `users_email_unique`, `posts_user_id_foreign`), but custom names can be specified.
- **Composite Support**: Indexes and foreign keys can span multiple columns by passing an array of column names.
- **Fluent Foreign Keys**: `foreignId()->constrained()` provides a terse, convention-based syntax that infers the related table and column automatically.
- **Referential Actions**: Foreign keys support `cascade`, `restrict`, `set null`, and `no action` actions for both `ON DELETE` and `ON UPDATE`, with dedicated helper methods (`cascadeOnDelete()`, etc.).
- **Database-Specific Availability**: `fullText` is supported on MariaDB, MySQL, and PostgreSQL; `spatialIndex` is supported on MySQL, PostgreSQL, and MariaDB, but not SQLite.
- **Reversible**: All index and foreign key creation methods have corresponding `drop*` methods for use in migration `down()` methods.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** with a configured database connection.
- **Basic knowledge of migrations and the `Blueprint` class**.
- **Understanding of SQL fundamentals** (indexes, foreign keys, referential integrity).
- **For `fullText`**: MySQL 5.6+, MariaDB, or PostgreSQL.
- **For `spatialIndex`**: MySQL 5.7+, MariaDB, or PostgreSQL with PostGIS (SQLite not supported).

---

### Related Programming Areas

- **Column Types**: Indexes are applied to columns defined by column type methods.
- **Eloquent Relationships**: Foreign keys are the database-level counterpart to Eloquent relationships (`belongsTo`, `hasMany`, etc.).
- **Query Optimization**: Indexes are the primary mechanism for accelerating `WHERE`, `JOIN`, and `ORDER BY` clauses.
- **Migrations**: Indexes and foreign keys are defined within migration files.
- **Database Seeding**: Foreign key constraints affect the order in which seeders can insert data.

---

## Core Concepts

---

### 1. Primary Index (`primary`)

#### Definitions

**Core Definition**

A primary index (primary key) is a unique, non-null index that uniquely identifies each row in a table and serves as the table's main identifier.

**Technical Definition**

`$table->primary(string|array $columns, string $name = null)` creates a primary key on the specified column(s). A table can have only one primary key, but it can be composite (spanning multiple columns). Primary keys automatically imply a unique index and a `NOT NULL` constraint. Laravel's `id()` and `increments()` methods create auto-incrementing primary keys automatically. The documentation notes that `primary()` is used for "adding a primary key" and accepts an array for composite keys.

**Beginner-Friendly Explanation**

A primary key is the unique ID for each row in a table. No two rows can have the same primary key, and it cannot be empty. Most tables have an `id` column as their primary key.

#### Purposes

- To uniquely identify every row in a table.
- To serve as the target for foreign key references.
- To enforce uniqueness and non-nullability on the identifying column(s).
- To enable efficient row lookups by primary key.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->primary('id');
$table->primary(['id', 'parent_id']); // Composite primary key
$table->primary('id', 'custom_primary_name');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$table->primary()` | Creates a primary key constraint. |
| `'id'` | A single column name. |
| `['id', 'parent_id']` | An array for a composite primary key. |
| `$name` (optional) | Custom constraint name. |

**Syntax Rules**

- A table can have only one primary key.
- Primary key columns are implicitly `NOT NULL` and `UNIQUE`.
- Composite primary keys are supported by passing an array.
- The `id()` and `increments()` column types create primary keys automatically; do not call `primary()` again for those.

**Constraints and Limitations**

- Only one primary key per table.
- Auto-incrementing columns are typically used as single-column primary keys; composite auto-incrementing keys are not supported.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Single-Column Primary Key**

**Code**

```php
Schema::create('flights', function (Blueprint $table) {
    $table->string('flight_number')->primary();
    $table->string('destination');
});
```

**Expected Output (MySQL)**

```sql
CREATE TABLE flights (
    flight_number VARCHAR(255) NOT NULL,
    destination VARCHAR(255) NOT NULL,
    PRIMARY KEY (flight_number)
);
```

**Why This Output Occurs**

`primary()` appends a `PRIMARY KEY (flight_number)` clause, making `flight_number` the unique, non-null identifier for the table.

---

**Example 2: Composite Primary Key**

**Code**

```php
Schema::create('role_user', function (Blueprint $table) {
    $table->unsignedBigInteger('role_id');
    $table->unsignedBigInteger('user_id');
    $table->primary(['role_id', 'user_id']);
});
```

**Expected Output**

```sql
CREATE TABLE role_user (
    role_id BIGINT UNSIGNED NOT NULL,
    user_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (role_id, user_id)
);
```

**Why This Output Occurs**

Passing an array to `primary()` creates a composite primary key across both columns, ensuring the combination is unique.

---

#### Real-World Cases with Explanation

**Case 1: Pivot Tables**

A many-to-many relationship uses a pivot table with a composite primary key on `(role_id, user_id)`, preventing duplicate associations.

**Explanation**: Composite primary keys are ideal for pivot tables where the combination of foreign keys must be unique.

---

### 2. Unique Index (`unique`)

#### Definitions

**Core Definition**

A unique index enforces that all values in the indexed column(s) are distinct, preventing duplicate entries while allowing fast lookups.

**Technical Definition**

`$table->unique(string|array $columns, string $name = null)` creates a unique index. Unlike a primary key, a table can have multiple unique indexes, and unique indexes generally allow multiple `NULL` values (database-dependent). The documentation confirms that `unique('email')` adds a unique index and that a custom name can be provided as the second argument.

**Beginner-Friendly Explanation**

A unique index means "no two rows can have the same value in this column." For example, email addresses in a users table must be unique.

#### Purposes

- To enforce uniqueness on columns other than the primary key (e.g., email, username, slug).
- To accelerate lookups on unique columns.
- To serve as a target for foreign key references (many databases require the referenced column to have a unique or primary index).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->unique('email');
$table->unique(['tenant_id', 'email']); // Composite unique index
$table->unique('email', 'unique_email'); // Custom name
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$table->unique()` | Creates a unique index. |
| `'email'` | A single column. |
| `['tenant_id', 'email']` | Composite unique index. |
| `'unique_email'` | Optional custom index name. |

**Syntax Rules**

- Multiple unique indexes per table are allowed.
- Composite unique indexes enforce uniqueness across the combination of columns.
- Custom names are useful for dropping indexes later.

**Constraints and Limitations**

- Unique indexes may allow multiple `NULL` values depending on the database (MySQL allows multiple `NULL`s; PostgreSQL treats `NULL`s as distinct).
- Unique indexes add overhead to `INSERT` and `UPDATE` operations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Unique Email**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('email')->unique();
});
```

**Expected Output (MySQL)**

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    UNIQUE (email)
);
```

**Why This Output Occurs**

`unique()` adds a `UNIQUE` constraint on the `email` column, preventing duplicate emails.

---

**Example 2: Composite Unique Index**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('tenant_id');
    $table->string('slug');
    $table->unique(['tenant_id', 'slug']);
});
```

**Expected Output**

The combination of `tenant_id` and `slug` must be unique, while individual values may repeat across tenants.

**Why This Output Occurs**

A composite unique index enforces uniqueness across the specified column combination.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant Slug Uniqueness**

A multi-tenant blog uses a composite unique index on `(tenant_id, slug)` so different tenants can use the same post slug without conflicts.

**Explanation**: Composite unique indexes enforce scoped uniqueness in multi-tenant applications.

---

### 3. Basic Index (`index`)

#### Definitions

**Core Definition**

A basic (non-unique) index is a data structure that accelerates data retrieval on a column or set of columns without enforcing uniqueness.

**Technical Definition**

`$table->index(string|array $columns, string $name = null)` creates a standard B-tree index. It does not enforce uniqueness and is used purely for performance optimization. The documentation confirms that `index('state')` adds a basic index and that composite indexes are created by passing an array.

**Beginner-Friendly Explanation**

A basic index is like a phone book sorted by last name—it helps you find rows quickly by that column, but it doesn't prevent duplicates.

#### Purposes

- To accelerate `WHERE`, `JOIN`, and `ORDER BY` queries on non-unique columns.
- To improve the performance of foreign key lookups (many databases do not automatically index foreign key columns).
- To support efficient range queries and sorting.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->index('state');
$table->index(['account_id', 'created_at']); // Composite index
$table->index('state', 'idx_state'); // Custom name
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$table->index()` | Creates a basic index. |
| `'state'` | A single column. |
| `['account_id', 'created_at']` | Composite index. |
| `'idx_state'` | Optional custom name. |

**Syntax Rules**

- Multiple basic indexes per table are allowed.
- Composite indexes are useful for queries that filter or sort by multiple columns.
- The order of columns in a composite index matters (leftmost prefix rule).

**Constraints and Limitations**

- Indexes add overhead to write operations.
- Not all columns benefit from indexing (e.g., low-cardinality columns like boolean flags).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Index on State**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('state');
    $table->index('state');
});
```

**Expected Output (MySQL)**

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    state VARCHAR(255) NOT NULL,
    INDEX users_state_index (state)
);
```

**Why This Output Occurs**

`index('state')` creates a non-unique B-tree index on the `state` column, accelerating queries that filter or sort by state.

---

**Example 2: Composite Index**

**Code**

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('account_id');
    $table->timestamp('created_at');
    $table->index(['account_id', 'created_at']);
});
```

**Expected Output**

The composite index on `(account_id, created_at)` accelerates queries that filter by `account_id` and sort by `created_at`.

**Why This Output Occurs**

Composite indexes serve queries that use the indexed columns in the same order as the index definition.

---

#### Real-World Cases with Explanation

**Case 1: Accelerating Foreign Key Joins**

A `posts` table has a `user_id` foreign key. Adding a basic index on `user_id` dramatically speeds up joins between `posts` and `users`.

**Explanation**: Many databases do not automatically index foreign key columns, making explicit indexes essential for join performance.

---

### 4. Full-Text Index (`fullText`)

#### Definitions

**Core Definition**

A full-text index is a specialized index that enables efficient keyword searches within text columns, supporting features like word stemming and relevance ranking.

**Technical Definition**

`$table->fullText(string|array $columns, string $name = null)` creates a full-text index. The documentation confirms that `fullText('body')` adds a full-text index and that a language can be specified with `->language('english')`. Full-text indexes are supported on MariaDB, MySQL, and PostgreSQL. Queries use `whereFullText()` (or `MATCH ... AGAINST` on MySQL/MariaDB).

**Beginner-Friendly Explanation**

A full-text index is like the search function in a document—it lets the database quickly find rows containing specific words or phrases in large text columns.

#### Purposes

- To enable fast keyword searches in `TEXT` and `VARCHAR` columns.
- To support relevance-ranked search results without an external search engine.
- To improve the performance of `MATCH ... AGAINST` queries.
- To provide a database-native alternative to LIKE queries for text search.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->fullText('body');
$table->fullText(['title', 'body']);
$table->fullText('body')->language('english'); // MySQL/MariaDB language
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$table->fullText()` | Creates a full-text index. |
| `'body'` | A single text column. |
| `['title', 'body']` | Multiple text columns. |
| `->language('english')` | Specifies the language for stemming (MySQL/MariaDB). |

**Syntax Rules**

- The indexed columns must be text types (`TEXT`, `VARCHAR`).
- Supported on MariaDB, MySQL, and PostgreSQL.
- On MySQL/MariaDB, the index language can be specified for stemming rules.

**Constraints and Limitations**

- Not supported on SQLite or SQL Server.
- Full-text indexes have specific storage and maintenance overhead.
- Query syntax differs between MySQL (`MATCH ... AGAINST`) and PostgreSQL (`to_tsvector`/`to_tsquery`).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Full-Text Index on a Single Column**

**Code**

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('body');
    $table->fullText('body');
});
```

**Expected Output (MySQL)**

```sql
CREATE TABLE articles (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    body TEXT NOT NULL,
    FULLTEXT articles_body_fulltext (body)
);
```

**Why This Output Occurs**

`fullText('body')` creates a `FULLTEXT` index on the `body` column, enabling `MATCH ... AGAINST` queries.

---

**Example 2: Full-Text Index with Language**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->text('content');
    $table->fullText('content')->language('english');
});
```

**Expected Output (MySQL)**

The full-text index uses the English stemming rules for word matching.

**Why This Output Occurs**

`->language('english')` configures the full-text index to use English stemming.

---

#### Real-World Cases with Explanation

**Case 1: Blog Search**

A blog uses a `fullText` index on `(title, body)` and `whereFullText()` to implement keyword search without an external search engine like Elasticsearch.

**Explanation**: Full-text indexes provide a lightweight, database-native search solution for moderate-scale applications.

---

### 5. Spatial Index (`spatialIndex`)

#### Definitions

**Core Definition**

A spatial index is a specialized index for geometric and geographic data types (points, lines, polygons), enabling efficient spatial queries such as distance and containment searches.

**Technical Definition**

`$table->spatialIndex(string|array $columns, string $name = null)` creates a spatial index. The documentation notes that it "adds a spatial index (except SQLite)". Spatial indexes are supported on MySQL (5.7+), MariaDB, and PostgreSQL (with PostGIS). They are used in conjunction with spatial column types (`geometry`, `geography`, `point`, etc.).

**Beginner-Friendly Explanation**

A spatial index is like a map index—it helps the database quickly find locations or shapes that are near each other or within a certain area.

#### Purposes

- To accelerate spatial queries (distance, containment, intersection).
- To support mapping, geolocation, and logistics features.
- To index geometric and geographic columns efficiently.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->spatialIndex('location');
$table->spatialIndex(['latitude', 'longitude']);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$table->spatialIndex()` | Creates a spatial index. |
| `'location'` | A spatial column. |
| `['latitude', 'longitude']` | Multiple spatial columns. |

**Syntax Rules**

- The indexed column must be a spatial type (`geometry`, `geography`, `point`, etc.).
- Supported on MySQL, MariaDB, and PostgreSQL; not supported on SQLite.

**Constraints and Limitations**

- SQLite does not support spatial indexes.
- The spatial column must use an appropriate spatial data type.
- Spatial queries require database-specific functions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Spatial Index on a Point Column**

**Code**

```php
Schema::create('places', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->point('location');
    $table->spatialIndex('location');
});
```

**Expected Output (MySQL)**

```sql
CREATE TABLE places (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    location POINT NOT NULL,
    SPATIAL INDEX places_location_spatialindex (location)
);
```

**Why This Output Occurs**

`spatialIndex('location')` creates a `SPATIAL INDEX` on the `location` point column, enabling spatial queries.

---

#### Real-World Cases with Explanation

**Case 1: Store Locator**

A retail application stores store locations as points and uses a spatial index to quickly find the nearest stores to a user's coordinates.

**Explanation**: Spatial indexes make geographic proximity queries feasible without scanning every row.

---

### 6. Foreign Key (`foreign`)

#### Definitions

**Core Definition**

A foreign key is a column (or set of columns) that references the primary or unique key of another table, enforcing referential integrity between the two tables.

**Technical Definition**

Foreign keys are created via `$table->foreign(string|array $columns)->references(string|array $columns)->on(string $table)` or the fluent `$table->foreignId('user_id')->constrained()`. The `constrained()` method uses naming conventions to infer the referenced table and column. Foreign keys support referential actions for `ON DELETE` and `ON UPDATE`: `cascade`, `restrict`, `set null`, and `no action`, accessible via helper methods like `cascadeOnDelete()`, `restrictOnDelete()`, `nullOnDelete()`, and `noActionOnDelete()`. Foreign keys are dropped with `dropForeign()`.

**Beginner-Friendly Explanation**

A foreign key is a link between two tables. It says "the value in this column must exist in that other table." For example, a `posts.user_id` must match an existing `users.id`. This prevents orphaned records.

#### Purposes

- To enforce referential integrity between related tables.
- To prevent orphaned or invalid records.
- To define cascading behaviour for deletes and updates.
- To serve as the database-level foundation for Eloquent relationships.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Verbose syntax
$table->unsignedBigInteger('user_id');
$table->foreign('user_id')->references('id')->on('users');

// Fluent syntax (recommended)
$table->foreignId('user_id')->constrained();

// With referential actions
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
$table->foreignId('user_id')->constrained()->restrictOnDelete()->cascadeOnUpdate();
$table->foreignId('user_id')->nullable()->constrained()->nullOnDelete();

// Custom table name
$table->foreignId('user_id')->constrained('users');

// Drop foreign key
$table->dropForeign(['user_id']);
$table->dropForeign('posts_user_id_foreign');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `foreignId()` | Creates an `UNSIGNED BIGINT` column (alias for `unsignedBigInteger`). |
| `constrained()` | Infers table/column from the column name and adds the foreign key. |
| `cascadeOnDelete()` | `ON DELETE CASCADE` — child rows are deleted with the parent. |
| `restrictOnDelete()` | `ON DELETE RESTRICT` — deletion is prevented if child rows exist. |
| `nullOnDelete()` | `ON DELETE SET NULL` — child foreign key is set to `null`. |
| `noActionOnDelete()` | `ON DELETE NO ACTION` — similar to `restrict`. |
| `dropForeign()` | Drops the foreign key constraint. |

**Syntax Rules**

- Additional column modifiers must be called **before** `constrained()`: `$table->foreignId('user_id')->nullable()->constrained()`.
- Foreign key constraints use the naming convention `{table}_{column}_foreign`.
- SQLite only supports foreign keys upon table creation, not when altering tables.
- `nullOnDelete()` requires the foreign key column to be `nullable()`.

**Constraints and Limitations**

- Foreign keys are not supported on SQLite when altering existing tables.
- Referential actions must be supported by the database engine (InnoDB on MySQL supports all actions).
- `SET NULL` requires the foreign key column to be nullable.
- Foreign keys add overhead to write operations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Foreign Key**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained();
    $table->string('title');
});
```

**Expected Output (MySQL)**

```sql
CREATE TABLE posts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    title VARCHAR(255) NOT NULL,
    CONSTRAINT posts_user_id_foreign FOREIGN KEY (user_id) REFERENCES users (id)
);
```

**Why This Output Occurs**

`foreignId('user_id')->constrained()` creates the `user_id` column and infers the `users(id)` reference from the column name, adding the foreign key constraint.

---

**Example 2: Cascading Deletes**

**Code**

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->foreignId('post_id')->constrained()->cascadeOnDelete();
    $table->text('body');
});
```

**Expected Output**

When a `posts` row is deleted, all associated `comments` are automatically deleted.

**Why This Output Occurs**

`cascadeOnDelete()` adds `ON DELETE CASCADE` to the foreign key constraint, propagating deletions to child rows.

---

**Example 3: Null on Delete**

**Code**

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('editor_id')->nullable()->constrained('users')->nullOnDelete();
});
```

**Expected Output**

When a `users` row is deleted, the `editor_id` of associated posts is set to `null` instead of deleting the posts.

**Why This Output Occurs**

`nullOnDelete()` adds `ON DELETE SET NULL`, which requires the `editor_id` column to be nullable.

---

**Example 4: Dropping a Foreign Key**

**Code**

```php
Schema::table('posts', function (Blueprint $table) {
    $table->dropForeign(['user_id']); // Uses convention: posts_user_id_foreign
});
```

**Expected Output**

The foreign key constraint on `posts.user_id` is removed.

**Why This Output Occurs**

Passing the column name as an array to `dropForeign()` automatically generates the conventional constraint name.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Items**

An `order_items` table has a foreign key to `orders` with `cascadeOnDelete()`, ensuring that deleting an order removes its items. It also has a foreign key to `products` with `restrictOnDelete()`, preventing a product from being deleted while it is referenced by an order.

**Explanation**: Different referential actions for different relationships balance data integrity with business rules.

---

## References

- Database: Migrations — Laravel Documentation (raw Markdown) — https://raw.githubusercontent.com/laravel/docs/bbb41c0afb1febf3cf82e7fdc3bed0ae7f787a07/migrations.md
- Laravel Migrations Documentation: Indexes & Foreign Keys (PDF excerpt) — https://github.com/DevStorm-Team/laravel-book/blob/f290014989ce7aee8657687973751b6685ad3f8a/laravel-docs-8.x.pdf
- Laravel Migrations Documentation: Foreign Keys (PDF excerpt) — https://github.com/DevStorm-Team/laravel-book/blob/db96be2bc446889263f2e15ec786e438174e5122/laravel-docs-master.pdf
- Laravel Full-Text Search Documentation — https://laravel.com/docs/master/search
- Laravel Scout Database Engine (Full-Text Indexes) — https://laravel.com/docs/13.x/scout#database-engine
- Harnessing Full-Text Search in Laravel — Laravel News — https://laravel-news.com/full-text-search
- Laravel Spatial Index Discussion — Laracasts — https://laracasts.com/discuss/channels/laravel/doest-table-spatialindex-location-support-postgresql
- Laravel API: ForeignKeyDefinition — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/ForeignKeyDefinition.html
- Laravel Migration Best Practices: Foreign Keys & Indexes — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/database-and-eloquent-orm/write-effective-migrations.md

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Index and foreign key methods have remained stable across these versions. `fullText` is supported on MariaDB, MySQL, and PostgreSQL. `spatialIndex` is supported on MySQL, MariaDB, and PostgreSQL, but not SQLite. `nullOnDelete()` and the other referential action helpers require Laravel 8+. Foreign keys on SQLite are only supported at table creation time. Developers should always consult the documentation for their specific Laravel version and database driver.