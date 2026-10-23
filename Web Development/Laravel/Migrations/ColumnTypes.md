# Laravel Schema Builder Column Types: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Schema Builder column types are the PHP methods on Laravel's `Blueprint` class that define the data type, constraints, and behaviour of individual columns within a database table.

**Technical Definition**

Each column type method on `Illuminate\Database\Schema\Blueprint` corresponds to a specific SQL data type, abstracted across database drivers (MySQL, PostgreSQL, SQLite, SQL Server). Methods like `string()`, `integer()`, and `timestamp()` generate the appropriate DDL column definition for the configured connection. Many types also carry implicit constraints (e.g., `id()` creates an auto-incrementing primary key, `foreignId()` creates an unsigned big integer for foreign keys). Column modifiers (`nullable()`, `default()`, `index()`, etc.) can be chained to refine behaviour.

**Beginner-Friendly Explanation**

When you create a database table, you need to tell the database what type of data each column will hold—numbers, text, dates, true/false values, and so on. Laravel gives you a simple PHP method for each of these types. Instead of remembering whether to write `VARCHAR(255)` or `INT UNSIGNED` or `TIMESTAMP`, you write `$table->string('name')`, `$table->integer('votes')`, or `$table->timestamp('created_at')`, and Laravel writes the correct SQL for your database.

---

### Key Characteristics

- **Database Agnostic**: A single PHP method generates the correct SQL type for MySQL, PostgreSQL, SQLite, and SQL Server.
- **Type-Specific Constraints**: Methods like `id()` and `foreignId()` bundle primary key and foreign key constraints, reducing boilerplate.
- **Companion "Increments" Methods**: `bigIncrements()`, `increments()`, `mediumIncrements()`, `smallIncrements()`, and `tinyIncrements()` create auto-incrementing primary key columns of varying integer sizes.
- **Unsigned Variants**: Unsigned integer types (`unsignedBigInteger()`, `unsignedInteger()`, etc.) are provided for databases that support them (primarily MySQL/MariaDB).
- **Convenience Methods**: `morphs()`, `nullableMorphs()`, `softDeletes()`, `rememberToken()`, and `timestamps()` bundle multiple column definitions into one method call.
- **Explicit Identifier Types**: `uuid()`, `ulid()`, `foreignUuid()`, and `foreignUlid()` support modern, non-integer primary and foreign keys.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** with a configured database connection.
- **Basic knowledge of migrations** and the `Schema::create()` / `Schema::table()` methods.
- **Familiarity with SQL data types** is helpful but not required.
- **`doctrine/dbal` package** for certain column modifications on some databases.

---

### Related Programming Areas

- **Migrations**: Column types are defined within migration files.
- **Eloquent Models**: Column types influence model casting and attribute behaviour.
- **Database Seeding**: Seeders populate the columns defined by these types.
- **Query Builder**: The data types influence how queries filter, sort, and compare values.

---

## Core Concepts

---

### 1. Numeric & Integer Types

#### Definitions

**Core Definition**

Numeric and integer column types define columns that store whole numbers, decimal numbers, floating-point numbers, and auto-incrementing primary keys.

**Technical Definition**

Laravel provides integer types of varying storage sizes (`tinyInteger`, `smallInteger`, `mediumInteger`, `integer`, `bigInteger`), their unsigned variants, and floating-point types (`float`, `double`, `decimal`). The "increments" family (`tinyIncrements` through `bigIncrements`) creates auto-incrementing unsigned primary key columns. The `decimal` type accepts precision and scale parameters for exact numeric storage, making it suitable for monetary values.

**Beginner-Friendly Explanation**

These are the column types for numbers. You choose how big the numbers can be (tiny, small, medium, regular, or big) and whether they can be negative (unsigned means only positive). For money, use `decimal` to avoid rounding errors.

#### Purposes

- To store integer values of appropriate size for the expected data range.
- To create auto-incrementing primary keys for tables.
- To store exact decimal values (e.g., monetary amounts) with `decimal`.
- To store approximate floating-point values with `float` and `double`.
- To enforce non-negative values with unsigned integer types (MySQL/MariaDB).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->tinyInteger('votes');
$table->smallInteger('votes');
$table->mediumInteger('votes');
$table->integer('votes');
$table->bigInteger('votes');
$table->unsignedTinyInteger('votes');
$table->unsignedSmallInteger('votes');
$table->unsignedMediumInteger('votes');
$table->unsignedInteger('votes');
$table->unsignedBigInteger('votes');
$table->tinyIncrements('id');
$table->smallIncrements('id');
$table->mediumIncrements('id');
$table->increments('id');
$table->bigIncrements('id');
$table->decimal('amount', $precision = 8, $scale = 2);
$table->unsignedDecimal('amount', $precision = 8, $scale = 2);
$table->float('amount', $precision = 53);
$table->double('amount', $total = null, $places = null);
```

**Component Breakdown**

| Method | SQL Equivalent (MySQL) | Description |
|--------|------------------------|-------------|
| `tinyInteger` | `TINYINT` | 1-byte integer. |
| `smallInteger` | `SMALLINT` | 2-byte integer. |
| `mediumInteger` | `MEDIUMINT` | 3-byte integer. |
| `integer` | `INT` | 4-byte integer. |
| `bigInteger` | `BIGINT` | 8-byte integer. |
| `unsignedTinyInteger` | `TINYINT UNSIGNED` | 1-byte, non-negative. |
| `unsignedSmallInteger` | `SMALLINT UNSIGNED` | 2-byte, non-negative. |
| `unsignedMediumInteger` | `MEDIUMINT UNSIGNED` | 3-byte, non-negative. |
| `unsignedInteger` | `INT UNSIGNED` | 4-byte, non-negative. |
| `unsignedBigInteger` | `BIGINT UNSIGNED` | 8-byte, non-negative. |
| `tinyIncrements` | `TINYINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Auto-incrementing PK. |
| `smallIncrements` | `SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Auto-incrementing PK. |
| `mediumIncrements` | `MEDIUMINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Auto-incrementing PK. |
| `increments` | `INT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Auto-incrementing PK. |
| `bigIncrements` | `BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Auto-incrementing PK. |
| `decimal` | `DECIMAL(precision, scale)` | Exact decimal number. |
| `unsignedDecimal` | `DECIMAL(precision, scale) UNSIGNED` | Exact, non-negative decimal. |
| `float` | `FLOAT` / `DOUBLE` (driver-dependent) | Approximate floating-point. |
| `double` | `DOUBLE` | Approximate double-precision. |

**Syntax Rules**

- `decimal` accepts optional `$precision` (total digits) and `$scale` (digits after decimal).
- `float` accepts an optional `$precision` parameter.
- `double` accepts optional `$total` and `$places` parameters.
- Unsigned types are primarily meaningful on MySQL/MariaDB; other databases may ignore the unsigned flag.

**Constraints and Limitations**

- Unsigned types are not supported on PostgreSQL; use check constraints or application-level validation instead.
- `float` and `double` are approximate; avoid them for monetary values.
- `increments` methods create primary keys; do not combine them with a separate `$table->primary()` call.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Integer Types and Auto-Incrementing Keys**

**Code**

```php
Schema::create('votes', function (Blueprint $table) {
    $table->tinyIncrements('id');             // Auto-incrementing PK
    $table->tinyInteger('score');             // TINYINT
    $table->smallInteger('rank');             // SMALLINT
    $table->mediumInteger('views');           // MEDIUMINT
    $table->integer('votes');                 // INT
    $table->bigInteger('total');              // BIGINT
});
```

**Expected Output**

The table `votes` is created with an auto-incrementing `id` and columns of the specified integer sizes.

**Why This Output Occurs**

Each method generates the corresponding integer column type for the database driver. The `tinyIncrements('id')` method creates a primary key that auto-increments.

---

**Example 2: Decimal for Monetary Values**

**Code**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->decimal('price', 10, 2);          // DECIMAL(10,2)
    $table->unsignedDecimal('discount', 5, 2); // DECIMAL(5,2) UNSIGNED
});
```

**Expected Output**

The `price` column can store values up to 99,999,999.99; the `discount` column stores non-negative values up to 999.99.

**Why This Output Occurs**

`decimal(10, 2)` means 10 total digits with 2 after the decimal point. `unsignedDecimal` adds the unsigned constraint on MySQL/MariaDB.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Pricing**

An e-commerce application uses `decimal(10, 2)` for product prices to avoid floating-point rounding errors that would occur with `float` or `double`.

**Explanation**: Exact decimal storage is essential for financial calculations.

---

### 2. String & Text Types

#### Definitions

**Core Definition**

String and text column types define columns that store character data, ranging from fixed-length short strings to variable-length long text.

**Technical Definition**

`char` creates a fixed-length string. `string` creates a variable-length `VARCHAR` with an optional length (default 255). `tinyText`, `text`, `mediumText`, and `longText` create `TEXT` columns of increasing maximum size. On MySQL/MariaDB, the text types can be converted to `BLOB` equivalents by applying a binary character set.

**Beginner-Friendly Explanation**

These columns hold text—names, emails, descriptions, articles. Use `string` for short text (like names), `text` for longer content (like descriptions), and `longText` for very long content (like entire articles).

#### Purposes

- To store short strings with `string` or fixed-length strings with `char`.
- To store longer text content with `text`, `mediumText`, or `longText`.
- To store binary data using the text types with a binary character set on MySQL/MariaDB.
- To provide appropriate storage sizes for different content lengths.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->char('code', 4);
$table->string('name', 100);
$table->tinyText('notes');
$table->text('description');
$table->mediumText('content');
$table->longText('body');
```

**Component Breakdown**

| Method | SQL Equivalent | Description |
|--------|----------------|-------------|
| `char` | `CHAR(length)` | Fixed-length string. |
| `string` | `VARCHAR(length)` | Variable-length string (default 255). |
| `tinyText` | `TINYTEXT` | Up to 255 bytes. |
| `text` | `TEXT` | Up to 65,535 bytes. |
| `mediumText` | `MEDIUMTEXT` | Up to 16,777,215 bytes. |
| `longText` | `LONGTEXT` | Up to 4,294,967,295 bytes. |

**Syntax Rules**

- `string` and `char` accept an optional length argument.
- Text types do not require a length; their maximum size is defined by the database engine.
- On MySQL/MariaDB, `->charset('binary')` converts text types to their `BLOB` equivalents.

**Constraints and Limitations**

- `char` pads values with spaces to the specified length.
- Text columns may have performance implications for indexing; index only prefixes if necessary.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: String and Text Columns**

**Code**

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title', 200);             // VARCHAR(200)
    $table->text('excerpt');                  // TEXT
    $table->longText('body');                 // LONGTEXT
    $table->char('language', 2);              // CHAR(2)
});
```

**Expected Output**

The `articles` table is created with the specified string and text columns.

**Why This Output Occurs**

Each method maps to the corresponding SQL string or text type for the database driver.

---

**Example 2: Binary Data with Text Types (MySQL/MariaDB)**

**Code**

```php
Schema::create('files', function (Blueprint $table) {
    $table->id();
    $table->longText('data')->charset('binary'); // LONGBLOB
});
```

**Expected Output**

The `data` column is created as a `LONGBLOB` on MySQL/MariaDB.

**Why This Output Occurs**

Applying `->charset('binary')` to a text column on MySQL/MariaDB creates the corresponding `BLOB` type.

---

#### Real-World Cases with Explanation

**Case 1: Blog Articles**

A blog uses `string('title')` for article titles and `longText('body')` for the full article content.

**Explanation**: Choosing the right text type balances storage efficiency with content capacity.

---

### 3. Date & Time Types

#### Definitions

**Core Definition**

Date and time column types define columns that store dates, times, timestamps, and combinations thereof, with optional timezone awareness.

**Technical Definition**

Laravel provides `date`, `dateTime`, `dateTimeTz`, `time`, `timeTz`, `timestamp`, `timestampTz`, `timestamps`, `timestampsTz`, `softDeletes`, `softDeletesTz`, `nullableTimestamps`, and `year`. The `Tz` variants store timezone-aware values. The `timestamps()` method creates `created_at` and `updated_at` columns, while `softDeletes()` creates a nullable `deleted_at` column for Eloquent's soft delete functionality.

**Beginner-Friendly Explanation**

These columns store dates and times. Use `date` for birthdays, `dateTime` for exact moments, `timestamp` for record creation times, and `softDeletes` to mark records as "deleted" without actually removing them.

#### Purposes

- To store dates, times, and datetimes with appropriate precision.
- To automatically maintain `created_at` and `updated_at` timestamps with `timestamps()`.
- To enable soft deletes with `softDeletes()`.
- To store timezone-aware values with the `Tz` variants.
- To store years with `year`.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->date('birthday');
$table->dateTime('appointment', $precision = 0);
$table->dateTimeTz('appointment', $precision = 0);
$table->time('sunrise', $precision = 0);
$table->timeTz('sunrise', $precision = 0);
$table->timestamp('added_at', $precision = 0);
$table->timestampTz('added_at', $precision = 0);
$table->timestamps($precision = 0);
$table->timestampsTz($precision = 0);
$table->softDeletes($column = 'deleted_at', $precision = 0);
$table->softDeletesTz($column = 'deleted_at', $precision = 0);
$table->year('birth_year');
```

**Component Breakdown**

| Method | SQL Equivalent | Description |
|--------|----------------|-------------|
| `date` | `DATE` | Date only. |
| `dateTime` | `DATETIME` | Date and time. |
| `dateTimeTz` | `DATETIME WITH TIME ZONE` (driver-dependent) | Timezone-aware datetime. |
| `time` | `TIME` | Time only. |
| `timeTz` | `TIME WITH TIME ZONE` | Timezone-aware time. |
| `timestamp` | `TIMESTAMP` | Unix-style timestamp. |
| `timestampTz` | `TIMESTAMP WITH TIME ZONE` | Timezone-aware timestamp. |
| `timestamps` | `created_at` and `updated_at` | Two timestamp columns. |
| `timestampsTz` | `created_at` and `updated_at` (TZ) | Timezone-aware timestamps. |
| `softDeletes` | `deleted_at` (nullable timestamp) | Soft delete support. |
| `softDeletesTz` | `deleted_at` (nullable, TZ) | Timezone-aware soft delete. |
| `year` | `YEAR` | Year only. |

**Syntax Rules**

- `dateTime`, `time`, `timestamp`, and their `Tz` variants accept an optional `$precision` argument for fractional seconds.
- `softDeletes` and `softDeletesTz` accept optional `$column` and `$precision` arguments.
- `timestamps` creates both `created_at` and `updated_at`.

**Constraints and Limitations**

- The `Tz` variants are only meaningful on databases that support timezone-aware types (e.g., PostgreSQL).
- `softDeletes` requires the model to use the `SoftDeletes` trait.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Standard Timestamps and Soft Deletes**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->timestamps();                       // created_at & updated_at
    $table->softDeletes();                      // nullable deleted_at
});
```

**Expected Output**

The `posts` table includes `created_at`, `updated_at`, and a nullable `deleted_at` column.

**Why This Output Occurs**

`timestamps()` creates the two timestamp columns, and `softDeletes()` adds the nullable `deleted_at` column used by Eloquent's soft delete system.

---

**Example 2: Timezone-Aware Timestamps**

**Code**

```php
Schema::create('events', function (Blueprint $table) {
    $table->id();
    $table->dateTimeTz('starts_at', precision: 6); // TZ-aware with microseconds
    $table->timestampsTz(precision: 6);
});
```

**Expected Output**

The `events` table uses timezone-aware datetime columns with microsecond precision.

**Why This Output Occurs**

`dateTimeTz` and `timestampsTz` create timezone-aware columns, and the `precision: 6` argument enables microsecond precision.

---

#### Real-World Cases with Explanation

**Case 1: Event Scheduling Across Timezones**

A global event platform uses `dateTimeTz` to store event start times in a timezone-aware format, ensuring correct display across different user timezones.

**Explanation**: Timezone-aware types prevent ambiguity when scheduling events across regions.

---

### 4. Keys, IDs & Identifiers

#### Definitions

**Core Definition**

Key, ID, and identifier column types define primary keys, foreign keys, UUIDs, ULIDs, and auto-incrementing IDs.

**Technical Definition**

`id()` creates an auto-incrementing `BIGINT UNSIGNED` primary key (alias for `bigIncrements()`). `foreignId()` creates an `UNSIGNED BIGINT` column for foreign keys and can be chained with `constrained()` for automatic foreign key constraint creation. `uuid()` and `ulid()` create string-based identifier columns (36 and 26 characters, respectively). `foreignUuid()` and `foreignUlid()` create foreign key columns of the corresponding type.

**Beginner-Friendly Explanation**

These are the columns that uniquely identify records and link tables together. `id()` gives you an automatic number for each row. `foreignId()` creates a link to another table. `uuid()` and `ulid()` give you long, unique strings instead of numbers.

#### Purposes

- To create primary keys automatically with `id()`.
- To create foreign key columns with `foreignId()`, `foreignUuid()`, or `foreignUlid()`.
- To use UUIDs or ULIDs as primary or foreign keys for distributed systems.
- To enforce referential integrity with `constrained()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->id();
$table->foreignId('user_id')->constrained();
$table->foreignUuid('user_id')->constrained();
$table->foreignUlid('user_id')->constrained();
$table->uuid('id')->primary();
$table->ulid('id')->primary();
```

**Component Breakdown**

| Method | SQL Equivalent | Description |
|--------|----------------|-------------|
| `id()` | `BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Auto-incrementing PK. |
| `foreignId()` | `BIGINT UNSIGNED` | Foreign key column. |
| `foreignUuid()` | `CHAR(36)` | UUID foreign key column. |
| `foreignUlid()` | `CHAR(26)` | ULID foreign key column. |
| `uuid()` | `CHAR(36)` | UUID column. |
| `ulid()` | `CHAR(26)` | ULID column. |

**Syntax Rules**

- `foreignId()->constrained()` automatically infers the related table and column from the column name (e.g., `user_id` → `users.id`).
- `uuid()` and `ulid()` can be chained with `->primary()` to create non-incrementing primary keys.
- `constrained()` can accept a custom table name, column, and index name.

**Constraints and Limitations**

- `foreignId()` creates an unsigned big integer on all databases, even those that do not support unsigned types.
- UUIDs and ULIDs are stored as strings, which can impact index performance compared to integers.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Auto-Incrementing ID and Foreign Key**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();                                 // BIGINT UNSIGNED PK
    $table->foreignId('user_id')->constrained();  // FK to users.id
    $table->string('title');
});
```

**Expected Output**

The `posts` table has an auto-incrementing `id` and a `user_id` foreign key referencing `users.id`.

**Why This Output Occurs**

`id()` creates the primary key. `foreignId('user_id')` creates the column, and `constrained()` adds the foreign key constraint.

---

**Example 2: UUID Primary Key**

**Code**

```php
Schema::create('orders', function (Blueprint $table) {
    $table->uuid('id')->primary();                // CHAR(36) PK
    $table->foreignUuid('user_id')->constrained(); // UUID FK
    $table->timestamps();
});
```

**Expected Output**

The `orders` table uses a UUID primary key and a UUID foreign key to `users`.

**Why This Output Occurs**

`uuid('id')->primary()` creates a `CHAR(36)` column as the primary key. `foreignUuid('user_id')` creates a matching UUID foreign key column.

---

#### Real-World Cases with Explanation

**Case 1: Distributed Systems with UUIDs**

A distributed application uses UUIDs as primary keys to avoid ID collisions across multiple database shards.

**Explanation**: UUIDs and ULIDs are suitable for distributed systems where auto-incrementing integers are impractical.

---

### 5. Polymorphic Relations

#### Definitions

**Core Definition**

Polymorphic relation column types create the pair of columns (`{name}_id` and `{name}_type`) needed for Eloquent polymorphic relationships.

**Technical Definition**

`morphs('taggable')` creates `taggable_id` (an unsigned big integer, UUID, or ULID depending on the default morph key type) and `taggable_type` (a `VARCHAR`). The `nullableMorphs`, `ulidMorphs`, `uuidMorphs`, `nullableUlidMorphs`, and `nullableUuidMorphs` variants create nullable or type-specific versions.

**Beginner-Friendly Explanation**

Polymorphic relations let one table belong to multiple other tables. For example, a `comments` table might have comments that belong to either `posts` or `videos`. The morph columns store which table the comment belongs to and the ID of the record.

#### Purposes

- To define the columns required for polymorphic Eloquent relationships.
- To support nullable polymorphic relationships.
- To use UUIDs or ULIDs for polymorphic identifiers.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->morphs('taggable');
$table->nullableMorphs('taggable');
$table->uuidMorphs('taggable');
$table->ulidMorphs('taggable');
$table->nullableUuidMorphs('taggable');
$table->nullableUlidMorphs('taggable');
```

**Component Breakdown**

| Method | Columns Created | Description |
|--------|-----------------|-------------|
| `morphs` | `{name}_id`, `{name}_type` | Standard polymorphic pair. |
| `nullableMorphs` | Nullable `{name}_id`, `{name}_type` | Nullable pair. |
| `uuidMorphs` | `{name}_id` (`CHAR(36)`), `{name}_type` | UUID-based pair. |
| `ulidMorphs` | `{name}_id` (`CHAR(26)`), `{name}_type` | ULID-based pair. |
| `nullableUuidMorphs` | Nullable UUID pair. | Nullable UUID pair. |
| `nullableUlidMorphs` | Nullable ULID pair. | Nullable ULID pair. |

**Syntax Rules**

- The name argument is the relationship name (e.g., `taggable`).
- The `_id` column type depends on the default morph key type (`int`, `uuid`, or `ulid`).
- The `_type` column is always a `VARCHAR`.

**Constraints and Limitations**

- The default morph key type can be configured via `Schema::defaultMorphKeyType()` or `morphUsingUuids()` / `morphUsingUlids()`.
- Morph columns should be indexed for performance; `morphs()` creates an index by default.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Standard Polymorphic Columns**

**Code**

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->morphs('commentable'); // commentable_id + commentable_type
    $table->text('body');
});
```

**Expected Output**

The `comments` table has `commentable_id` (unsigned big integer) and `commentable_type` (VARCHAR) columns, with an index on the pair.

**Why This Output Occurs**

`morphs('commentable')` creates the two columns required for a polymorphic relationship and adds a composite index.

---

**Example 2: UUID Polymorphic Columns**

**Code**

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->uuidMorphs('commentable'); // commentable_id CHAR(36) + commentable_type
    $table->text('body');
});
```

**Expected Output**

The `commentable_id` column is `CHAR(36)` instead of a big integer.

**Why This Output Occurs**

`uuidMorphs` creates the `_id` column as a UUID-compatible `CHAR(36)`.

---

#### Real-World Cases with Explanation

**Case 1: Comments on Multiple Content Types**

A CMS allows comments on both articles and videos. The `comments` table uses `morphs('commentable')` to associate comments with either content type.

**Explanation**: Polymorphic columns enable flexible relationships without separate comment tables.

---

### 6. Spatial / Geometry Types

#### Definitions

**Core Definition**

Spatial and geometry column types store geographic and geometric data such as points, lines, polygons, and geometry collections.

**Technical Definition**

Laravel provides `geometry`, `geography`, `geometryCollection`, `lineString`, `multiLineString`, `multiPoint`, `multiPolygon`, `point`, and `polygon` column types. These map to the spatial types supported by the database (e.g., PostGIS for PostgreSQL, MySQL spatial types). The availability and behaviour of these types depend heavily on the database driver and installed extensions.

**Beginner-Friendly Explanation**

These columns store shapes and locations—like a point on a map, a route, or a boundary. They require special database support (like PostGIS).

#### Purposes

- To store geographic coordinates (points, lines, polygons).
- To enable spatial queries (distance, containment, intersection).
- To support mapping and geolocation features.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->geometry('location');
$table->geography('location', $subtype = null, $srid = 4326);
$table->geometryCollection('shapes');
$table->lineString('path');
$table->point('location');
$table->polygon('area');
```

**Component Breakdown**

| Method | SQL Equivalent (PostGIS) | Description |
|--------|--------------------------|-------------|
| `geometry` | `GEOMETRY` | Generic geometry column. |
| `geography` | `GEOGRAPHY` | Geography column (SRID-aware). |
| `geometryCollection` | `GEOMETRYCOLLECTION` | Collection of geometries. |
| `lineString` | `LINESTRING` | Line string. |
| `point` | `POINT` | Single point. |
| `polygon` | `POLYGON` | Polygon. |

**Syntax Rules**

- `geography` accepts optional subtype and SRID parameters.
- Spatial types require the database to have spatial extensions enabled (e.g., PostGIS).
- Spatial indexes can be added with `$table->spatialIndex('column')`.

**Constraints and Limitations**

- Not all databases support spatial types natively; SQLite requires SpatiaLite.
- Spatial types may not be available in older database versions.
- Spatial queries require database-specific functions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Storing a Point Location (PostgreSQL/PostGIS)**

**Code**

```php
Schema::create('places', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->point('location');               // POINT
    $table->spatialIndex('location');        // Spatial index
});
```

**Expected Output**

The `places` table has a `location` column of type `POINT` with a spatial index.

**Why This Output Occurs**

`point('location')` creates a spatial point column, and `spatialIndex()` adds a GiST index for efficient spatial queries.

---

#### Real-World Cases with Explanation

**Case 1: Store Locator**

A retail application stores store locations as points and queries for the nearest stores using spatial functions.

**Explanation**: Spatial types enable efficient geographic queries that would be difficult with separate latitude/longitude columns.

---

### 7. Specialized and Miscellaneous Types

#### Definitions

**Core Definition**

Specialized column types cover binary data, booleans, enumerations, IP addresses, JSON, MAC addresses, and other niche data formats.

**Technical Definition**

Laravel provides `binary` for raw binary data, `boolean` for true/false values, `enum` for a fixed set of string values, `ipAddress` for IP addresses (with PostgreSQL using `INET`), `json` and `jsonb` for JSON data, `macAddress` for MAC addresses, `rememberToken` for authentication tokens, and `set` for multiple-choice values (MySQL). Each type maps to a database-specific equivalent or a sensible fallback.

**Beginner-Friendly Explanation**

These are special-purpose columns: true/false toggles (`boolean`), lists of allowed values (`enum`, `set`), IP addresses, JSON data, and authentication tokens.

#### Purposes

- To store boolean values with `boolean`.
- To restrict values to a predefined list with `enum`.
- To store JSON data with `json` or `jsonb`.
- To store IP and MAC addresses with dedicated types.
- To store binary data with `binary`.
- To support Laravel's authentication "remember me" feature with `rememberToken()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->binary('data');
$table->boolean('is_active');
$table->enum('status', ['pending', 'active', 'archived']);
$table->ipAddress('visitor');
$table->json('options');
$table->jsonb('options');
$table->macAddress('device');
$table->rememberToken();
$table->set('flavors', ['strawberry', 'vanilla']);
```

**Component Breakdown**

| Method | SQL Equivalent | Description |
|--------|----------------|-------------|
| `binary` | `BLOB` / `BYTEA` | Raw binary data. |
| `boolean` | `BOOLEAN` / `TINYINT(1)` | True/false value. |
| `enum` | `ENUM(...)` | Fixed set of string values. |
| `ipAddress` | `VARCHAR(45)` / `INET` | IP address. |
| `json` | `JSON` / `TEXT` | JSON data. |
| `jsonb` | `JSONB` / `TEXT` | Binary JSON (PostgreSQL). |
| `macAddress` | `VARCHAR(17)` / dedicated | MAC address. |
| `rememberToken` | `VARCHAR(100)` nullable | Remember-me token. |
| `set` | `SET(...)` | Multiple-choice set (MySQL). |

**Syntax Rules**

- `enum` and `set` require an array of allowed values.
- `jsonb` is only meaningfully different from `json` on PostgreSQL.
- `rememberToken()` creates a nullable `VARCHAR(100)` column.

**Constraints and Limitations**

- `enum` values cannot be easily changed without altering the column definition.
- `set` is MySQL-specific; other databases may not support it.
- `jsonb` is PostgreSQL-specific; on other databases, it falls back to `json` or `text`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Enum and Boolean**

**Code**

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->enum('status', ['pending', 'shipped', 'delivered']);
    $table->boolean('is_paid')->default(false);
});
```

**Expected Output**

The `status` column only accepts the three specified values; `is_paid` defaults to `false`.

**Why This Output Occurs**

`enum` creates a column with a `CHECK` constraint (or native `ENUM` on MySQL) restricting values to the specified list. `boolean` creates a boolean column with a default value.

---

**Example 2: JSON and IP Address**

**Code**

```php
Schema::create('sessions', function (Blueprint $table) {
    $table->id();
    $table->ipAddress('ip_address');         // VARCHAR(45) or INET
    $table->json('payload');                 // JSON
    $table->rememberToken();                 // VARCHAR(100) nullable
});
```

**Expected Output**

The `sessions` table has an IP address column, a JSON payload column, and a nullable remember token column.

**Why This Output Occurs**

`ipAddress` creates a string column sized for IPv6 addresses (or `INET` on PostgreSQL). `json` creates a JSON column. `rememberToken()` creates a nullable `VARCHAR(100)`.

---

#### Real-World Cases with Explanation

**Case 1: Feature Flags in JSON**

An application stores feature flags in a `json` column, allowing flexible schema-free configuration per user or team.

**Explanation**: JSON columns provide flexibility for semi-structured data without requiring schema changes.

---

## Column Types Best Practices

1. **Use `decimal` for Monetary Values**: Never use `float` or `double` for money; use `decimal` with appropriate precision and scale.

2. **Choose the Smallest Integer Type That Fits**: Use `tinyInteger` for small ranges (e.g., status codes), `integer` for standard IDs, and `bigInteger` only when necessary.

3. **Prefer `id()` for Primary Keys**: Use `id()` for auto-incrementing primary keys unless you have a specific reason to use UUIDs or ULIDs.

4. **Use `foreignId()->constrained()` for Foreign Keys**: This creates the column, index, and constraint in one expressive call.

5. **Use `string` with Explicit Lengths**: Specify a length for `string` columns instead of relying on the default 255, especially for indexed columns.

6. **Prefer `text` for Long Content**: Use `text`, `mediumText`, or `longText` for content that exceeds `VARCHAR` limits.

7. **Use `timestamps()` and `softDeletes()` Consistently**: These methods create the standard columns expected by Eloquent.

8. **Use `json` for Semi-Structured Data**: JSON columns are appropriate for configuration, metadata, and flexible payloads, but avoid using them for data that requires frequent querying or indexing.

9. **Be Aware of Database-Specific Types**: Types like `jsonb`, `set`, `unsigned*`, and spatial types behave differently or are unavailable on some databases.

10. **Index Strategically**: Index columns used in `WHERE`, `JOIN`, and `ORDER BY`. Use `spatialIndex()` for spatial columns.

11. **Use `nullable()` Judiciously**: Only make columns nullable when the application genuinely allows missing values; otherwise, enforce NOT NULL constraints.

12. **Document Type Choices**: For non-obvious choices (e.g., `uuid` vs. `id`), document the rationale in migration comments.

---

## References

- Database: Migrations — Laravel 12.x Documentation — https://laravel.com/framework/docs/12.x/migrations
- Database: Migrations — Laravel 11.x Documentation — https://laravel.com/framework/docs/11.x/migrations
- Schema Builder — Laravel 5.0 Documentation — https://laravel.com/framework/docs/5.0/schema
- Understanding numeric types in Laravel migration schemas: Integer, Float, Decimal, Double — DEV Community — https://dev.to/…/understanding-numeric-types-in-laravel-migration-schemas-integer-float-decimal-double
- Laravel Column Types — Laravel API — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/Blueprint.html
- How to Define a money Data Type in Laravel Migrations for SQL Server — Laracasts — https://laracasts.com/discuss/channels/laravel/how-to-define-a-money-data-type-in-laravel-migrations-for-sql-server

---

**Version Note**: This cheat sheet is based on Laravel 11.x and 12.x documentation. Column types have remained largely stable across recent versions. The `jsonb` type is PostgreSQL-specific. `set` is MySQL-specific. Spatial types require database-specific extensions (e.g., PostGIS). Unsigned integer types are primarily meaningful on MySQL/MariaDB. Developers should always consult the documentation for their specific Laravel version and database driver.