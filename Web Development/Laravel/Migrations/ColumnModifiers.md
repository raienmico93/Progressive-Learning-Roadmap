# Laravel Schema Builder Column Modifiers: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Column modifiers are chainable methods on Laravel's `ColumnDefinition` class that adjust a column's attributes—such as its default value, nullability, storage position, character set, and computed behaviour—after its base data type has been defined.

**Technical Definition**

In the Schema Builder, a column type method (e.g., `string()`, `integer()`, `timestamp()`) returns a `ColumnDefinition` (or `ForeignIdColumnDefinition`) instance. Column modifiers are fluent methods on this instance that append SQL clauses (e.g., `DEFAULT`, `NULL`, `AFTER`, `GENERATED ALWAYS AS`) to the compiled DDL statement. Each modifier is handled by a database-specific `Grammar` class, which determines whether the modifier is supported and how it is compiled. The Laravel 10.x documentation states that the column modifier list "does not include index modifiers."

**Beginner-Friendly Explanation**

Imagine you are describing a column to your database. You start by saying what type of data it holds—"this is a string called `email`." Column modifiers are the extra instructions you attach to that description: "make it allowed to be empty" (`nullable`), "give it a default value" (`default`), "put it after the `name` column" (`after`), or "calculate its value automatically" (`virtualAs`). They are the fine-tuning dials for your database columns.

---

### Key Characteristics

- **Chainable**: Multiple modifiers can be applied to a single column definition, e.g., `$table->string('email')->nullable()->default(null)->after('name');`.
- **Database-Aware**: Not every modifier is supported by every database. Laravel's grammars compile only the modifiers supported by the target driver, and some modifiers are silently ignored on unsupported databases.
- **Preservation on Change**: When modifying an existing column with `->change()`, you must explicitly re-declare every modifier you want to keep; otherwise, they are dropped. Laravel 11+ introduced native column modification, but the principle of re-declaring modifiers remains critical.
- **Version-Specific**: Some modifiers require minimum database versions (e.g., `invisible()` requires MySQL 8.0.23+; `useCurrentOnUpdate()` requires Laravel 8.36+).
- **Generated Column Support**: `virtualAs()`, `storedAs()`, and `generatedAs()` enable database-level computed columns, reducing the need for application-level mutators.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** with a configured database connection.
- **Basic knowledge of migrations and the `Blueprint` class**.
- **`doctrine/dbal`** is required for certain column modification operations in older Laravel versions (pre-11).

---

### Related Programming Areas

- **Column Types**: Modifiers are always applied to a column type (e.g., `string`, `integer`, `timestamp`).
- **Migrations**: Modifiers are defined inside the `up()` and `down()` methods of migration files.
- **Eloquent Casting**: Modifiers like `default()` and generated columns interact with how Eloquent hydrates and persists model attributes.
- **Database Grammars**: The `MySqlGrammar`, `PostgresGrammar`, `SQLiteGrammar`, and `SqlServerGrammar` classes determine modifier support.

---

## Core Concepts

---

### 1. `after`

#### Definitions

**Core Definition**

The `after` modifier places a newly added column immediately after a specified existing column in the table's physical column order.

**Technical Definition**

`after(string $column)` compiles to the `AFTER column` clause in MySQL's `ALTER TABLE` syntax. It is only meaningful when adding a column to an existing table via `Schema::table()`. When creating a new table with `Schema::create()`, column order is determined by the order of method calls, making `after` unnecessary. Laravel's documentation explicitly notes that `after` is MySQL-specific.

**Beginner-Friendly Explanation**

`after` lets you say "put this new column right after that one." It only works on MySQL when you're adding a column to an existing table.

#### Purposes

- To control the physical column order when adding a column to an existing MySQL table.
- To group related columns together for readability in database tools.
- To maintain a logical column sequence without recreating the table.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('new_column')->after('existing_column');
```

Laravel also provides a fluent grouping helper:

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->after('existing_column')` | Places the column after the named column. |
| `$table->after('column', $callback)` | Groups multiple columns after the named column. |

**Syntax Rules**

- The modifier is chainable onto any column type method.
- The referenced column must exist in the table.
- `after` is ignored on databases other than MySQL/MariaDB.

**Constraints and Limitations**

- MySQL/MariaDB only.
- Cannot be used to place a column before another; only after (`first` is the alternative).
- On large tables, adding a column with `after` may cause a table rebuild, impacting performance.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Adding a Column After an Existing Column**

**Code**

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->nullable()->after('email');
});
```

**Expected Output (MySQL)**

```sql
ALTER TABLE `users` ADD `phone` VARCHAR(255) NULL AFTER `email`;
```

**Why This Output Occurs**

`after('email')` appends the `AFTER email` clause to the `ALTER TABLE` statement, positioning the new `phone` column immediately after the `email` column.

---

**Example 2: Grouping Multiple Columns with `after`**

**Code**

```php
Schema::table('users', function (Blueprint $table) {
    $table->after('password', function (Blueprint $table) {
        $table->string('address_line1');
        $table->string('address_line2');
        $table->string('city');
    });
});
```

**Expected Output (MySQL)**

The three columns (`address_line1`, `address_line2`, `city`) are added immediately after the `password` column, in the order specified.

**Why This Output Occurs**

The `after` grouping helper compiles each column definition with the `AFTER password` clause, ensuring the group is positioned together.

---

#### Real-World Cases with Explanation

**Case 1: Keeping Related Columns Together**

A team adds billing address fields to the `users` table and wants them to appear immediately after the `password` column in database tools, grouping authentication and profile data logically. The `after` modifier achieves this without restructuring the table.

**Explanation**: `after` is a presentation-oriented modifier that improves schema readability without affecting data behaviour.

---

### 2. `always`

#### Definitions

**Core Definition**

The `always` modifier defines the precedence of sequence-generated values over user-supplied values for a PostgreSQL identity column.

**Technical Definition**

`always(bool $value = true)` is used in conjunction with `generatedAs()` on PostgreSQL to produce an identity column with either `GENERATED ALWAYS AS IDENTITY` (when `true`) or `GENERATED BY DEFAULT AS IDENTITY` (when `false`). With `GENERATED ALWAYS`, the database rejects explicit values for the column unless `OVERRIDING SYSTEM VALUE` is specified; with `GENERATED BY DEFAULT`, explicit values are accepted but the sequence is used when no value is provided. Laravel's documentation lists `always()` as a PostgreSQL-specific modifier for identity columns.

**Beginner-Friendly Explanation**

`always` controls whether PostgreSQL's identity column will always generate its own value or whether you're allowed to insert your own value instead.

#### Purposes

- To enforce strict database-generated identity values (`GENERATED ALWAYS AS IDENTITY`).
- To allow controlled overriding of identity values when necessary (`GENERATED BY DEFAULT AS IDENTITY`).
- To align identity column behaviour with data integrity requirements.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->bigInteger('id')->generatedAs()->always();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `generatedAs()` | Creates a PostgreSQL identity column. |
| `always()` or `always(true)` | Produces `GENERATED ALWAYS AS IDENTITY`. |
| `always(false)` | Produces `GENERATED BY DEFAULT AS IDENTITY`. |

**Syntax Rules**

- Must be chained after `generatedAs()`.
- Only meaningful on PostgreSQL identity columns.
- The column type must be an integer type compatible with sequences (`smallint`, `integer`, `bigint`).

**Constraints and Limitations**

- PostgreSQL only.
- `GENERATED ALWAYS` prevents application-level insertion of explicit IDs without `OVERRIDING SYSTEM VALUE`.
- Does not apply to MySQL's `AUTO_INCREMENT`, which has different semantics.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Strict Identity Column**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->bigInteger('id')->generatedAs()->always();
    $table->string('name');
});
```

**Expected Output (PostgreSQL)**

```sql
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY,
    name VARCHAR(255) NOT NULL
);
```

**Why This Output Occurs**

`generatedAs()->always()` compiles to `GENERATED ALWAYS AS IDENTITY`, instructing PostgreSQL to reject explicit values for `id` unless the override clause is used.

---

**Example 2: Overridable Identity Column**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->bigInteger('id')->generatedAs()->always(false);
    $table->string('name');
});
```

**Expected Output (PostgreSQL)**

```sql
CREATE TABLE users (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY,
    name VARCHAR(255) NOT NULL
);
```

**Why This Output Occurs**

`always(false)` compiles to `GENERATED BY DEFAULT AS IDENTITY`, allowing explicit values to be inserted while still auto-generating values when none are supplied.

---

#### Real-World Cases with Explanation

**Case 1: Data Import with Explicit IDs**

A team importing legacy data into a PostgreSQL table needs to insert rows with their original IDs. Using `generatedAs()->always(false)` allows the explicit IDs to be inserted without errors, while still supporting auto-generation for new rows.

**Explanation**: `always(false)` provides flexibility for migrations and imports without sacrificing identity column functionality.

---

### 3. `autoIncrement`

#### Definitions

**Core Definition**

The `autoIncrement` modifier marks an integer column as auto-incrementing, causing the database to generate a sequential value for each new row.

**Technical Definition**

`autoIncrement()` compiles to `AUTO_INCREMENT` on MySQL/MariaDB and to a `SERIAL` or `GENERATED ... AS IDENTITY` equivalent on PostgreSQL (Laravel's grammar translates it). On SQLite, it produces an `INTEGER PRIMARY KEY AUTOINCREMENT`. The modifier is typically used with the `increments()` family of column types but can be applied manually. The documentation notes that `autoIncrement()` sets "INTEGER columns as auto-incrementing (primary key)."

**Beginner-Friendly Explanation**

`autoIncrement` tells the database to automatically give each new row the next number in sequence—1, 2, 3, and so on.

#### Purposes

- To create auto-incrementing primary key columns.
- To ensure unique, sequential identifiers without application-level generation.
- To simplify model creation by letting the database handle ID assignment.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->integer('id')->autoIncrement();
$table->unsignedBigInteger('id')->autoIncrement();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->autoIncrement()` | Enables auto-incrementing behaviour. |

**Syntax Rules**

- Must be applied to an integer-type column.
- Typically combined with `primary()` or used via `increments()`/`bigIncrements()`.
- On PostgreSQL, Laravel's grammar compiles `autoIncrement` as a primary key and changes the column type to `serial`.

**Constraints and Limitations**

- Only one auto-incrementing column per table is allowed in most databases.
- The column should be indexed (usually as a primary key) for optimal performance.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Manual Auto-Incrementing Primary Key**

**Code**

```php
Schema::create('flights', function (Blueprint $table) {
    $table->unsignedBigInteger('id')->autoIncrement();
    $table->string('name');
});
```

**Expected Output (MySQL)**

```sql
CREATE TABLE flights (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

**Why This Output Occurs**

`autoIncrement()` appends `AUTO_INCREMENT PRIMARY KEY` (Laravel adds the primary key constraint when the column is auto-incrementing).

---

#### Real-World Cases with Explanation

**Case 1: Custom Primary Key Configuration**

A table uses an `unsignedBigInteger` primary key with `autoIncrement()` instead of the `id()` helper, allowing the developer to customize the column name or constraints while retaining auto-incrementing behaviour.

**Explanation**: `autoIncrement` provides granular control over the primary key definition.

---

### 4. `charset`

#### Definitions

**Core Definition**

The `charset` modifier specifies the character set used to store string data in a column, controlling which characters can be represented.

**Technical Definition**

`charset(string $charset)` compiles to the `CHARACTER SET` clause on MySQL/MariaDB. It is MySQL-specific according to Laravel's documentation. On MySQL, applying `->charset('binary')` to a text column creates the corresponding `BLOB` type instead of a text type.

**Beginner-Friendly Explanation**

`charset` tells the database which "alphabet" to use for storing text in a column—for example, `utf8mb4` supports emojis and a wide range of characters.

#### Purposes

- To override the table's default character set for a specific column.
- To support multibyte characters (e.g., emojis, CJK scripts) with `utf8mb4`.
- To store binary data by using the `binary` character set on text columns.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('name')->charset('utf8mb4');
$table->text('data')->charset('binary');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->charset('utf8mb4')` | Sets the column's character set. |

**Syntax Rules**

- The character set must be supported by the database server.
- `charset('binary')` on text columns produces `BLOB` types on MySQL.
- The column type must be a string or text type.

**Constraints and Limitations**

- MySQL/MariaDB only according to the official modifier table.
- The character set must be installed on the database server.
- Collation is a separate modifier (`collation`).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: UTF-8 Emoji Support**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title')->charset('utf8mb4');
    $table->text('body')->charset('utf8mb4');
});
```

**Expected Output (MySQL)**

The `title` and `body` columns use `utf8mb4` instead of the table's default character set.

**Why This Output Occurs**

`charset('utf8mb4')` appends `CHARACTER SET utf8mb4` to the column definition, overriding the table default.

---

#### Real-World Cases with Explanation

**Case 1: Emoji Support in User Content**

A social application stores user-generated content that includes emojis. Applying `charset('utf8mb4')` to text columns ensures emojis are stored correctly instead of being truncated or corrupted.

**Explanation**: `charset` is essential for applications that handle the full Unicode range.

---

### 5. `collation`

#### Definitions

**Core Definition**

The `collation` modifier specifies the collation (sorting and comparison rules) for a column's string data.

**Technical Definition**

`collation(string $collation)` compiles to the `COLLATE` clause. According to Laravel's documentation, it is supported on MySQL, PostgreSQL, and SQL Server. Collation determines how strings are compared and sorted, including case sensitivity and accent handling.

**Beginner-Friendly Explanation**

`collation` tells the database how to sort and compare text in a column—for example, whether "a" and "A" are considered equal.

#### Purposes

- To control case sensitivity and accent sensitivity in string comparisons.
- To ensure consistent sorting order for a column.
- To align column collation with application-level expectations.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('email')->collation('utf8mb4_unicode_ci');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->collation('utf8mb4_unicode_ci')` | Sets the column's collation. |

**Syntax Rules**

- The collation must be valid for the column's character set.
- Supported on MySQL, PostgreSQL, and SQL Server.

**Constraints and Limitations**

- Not supported on SQLite (collation is limited).
- The collation must exist on the database server.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Case-Insensitive Email Comparison**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('email')->collation('utf8mb4_unicode_ci')->unique();
});
```

**Expected Output (MySQL)**

The `email` column uses case-insensitive collation, so `User@Example.com` and `user@example.com` are considered equal in comparisons and uniqueness checks.

**Why This Output Occurs**

`collation('utf8mb4_unicode_ci')` appends `COLLATE utf8mb4_unicode_ci`, which defines case-insensitive, accent-insensitive comparison rules.

---

#### Real-World Cases with Explanation

**Case 1: Consistent Email Uniqueness**

A registration system enforces email uniqueness case-insensitively. Using a case-insensitive collation ensures that `John@Example.com` and `john@example.com` cannot both be registered.

**Explanation**: `collation` aligns database comparison behaviour with business rules.

---

### 6. `comment`

#### Definitions

**Core Definition**

The `comment` modifier attaches a descriptive comment to a column in the database schema.

**Technical Definition**

`comment(string $comment)` compiles to the `COMMENT` clause. Laravel's documentation states that it is supported on MySQL and PostgreSQL. The comment is stored as metadata and does not affect data or queries.

**Beginner-Friendly Explanation**

`comment` is a note you attach to a column to document its purpose. It is visible in database tools but does not affect how the column works.

#### Purposes

- To document the purpose or business logic of a column.
- To provide context for future developers inspecting the schema.
- To annotate deprecated or special-purpose columns.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('status')->comment('Order fulfilment status');
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->comment('...')` | Attaches a comment to the column. |

**Syntax Rules**

- The comment is a string.
- Supported on MySQL and PostgreSQL.

**Constraints and Limitations**

- Comments are metadata; they do not enforce constraints or change behaviour.
- Some database tools may not display comments prominently.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Documenting a Status Column**

**Code**

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->string('status')->comment('pending | shipped | delivered');
});
```

**Expected Output (MySQL)**

The `status` column has an associated comment listing the allowed values.

**Why This Output Occurs**

`comment()` appends a `COMMENT '...'` clause to the column definition, storing the metadata.

---

#### Real-World Cases with Explanation

**Case 1: Legacy Schema Documentation**

A legacy database lacks formal documentation. Developers add `comment()` modifiers to clarify ambiguous columns (e.g., `flag_a`, `flag_b`) without altering application code.

**Explanation**: `comment` provides low-cost, in-schema documentation.

---

### 7. `default`

#### Definitions

**Core Definition**

The `default` modifier specifies a default value for a column when no value is provided during an `INSERT`.

**Technical Definition**

`default(mixed $value)` compiles to the `DEFAULT` clause. The value can be a scalar (string, number, boolean, `null`) or an `Illuminate\Database\Query\Expression` instance. Using an `Expression` prevents Laravel from quoting the value, enabling database-specific functions (e.g., `JSON_ARRAY()`). Laravel's documentation notes that "support for default expressions depends on your database driver, database version, and the field type."

**Beginner-Friendly Explanation**

`default` tells the database "if nobody provides a value for this column, use this one instead."

#### Purposes

- To ensure columns have a meaningful value when omitted from inserts.
- To reduce application-level default-handling logic.
- To use database functions as defaults (via `Expression`).
- To mirror column defaults in Eloquent models for consistent in-memory behaviour.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('status')->default('pending');
$table->boolean('is_active')->default(true);
$table->timestamp('published_at')->nullable()->default(null);
$table->json('metadata')->default(new Expression('(JSON_ARRAY())'));
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->default($value)` | Sets a literal default. |
| `->default(new Expression('...'))` | Sets a raw database expression as default. |
| `->default(null)` | Makes `null` the default (often combined with `nullable`). |

**Syntax Rules**

- Scalar values are automatically quoted and escaped.
- `Expression` instances bypass quoting and are inserted verbatim.
- Default expressions are database-specific and may not be portable.

**Constraints and Limitations**

- Some default expressions are not supported on all databases or column types.
- Using database-specific functions couples the schema to that database.
- Defaults are not retroactively applied to existing rows when a column is added (unless the database supports it).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Simple and Expression Defaults**

**Code**

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->string('status')->default('pending');
    $table->json('metadata')->default(new Expression('(JSON_ARRAY())'));
});
```

**Expected Output (MySQL)**

The `status` column defaults to `'pending'`; the `metadata` column defaults to an empty JSON array `[]`.

**Why This Output Occurs**

`default('pending')` produces `DEFAULT 'pending'`. The `Expression` produces `DEFAULT (JSON_ARRAY())`, which MySQL evaluates as an empty JSON array.

---

#### Real-World Cases with Explanation

**Case 1: JSON Configuration Defaults**

An application stores per-user settings in a JSON column. Using `default(new Expression('(JSON_ARRAY())'))` ensures new users start with an empty array instead of `null`, simplifying application logic.

**Explanation**: Expression defaults enable database-level initialization of complex types.

---

### 8. `first`

#### Definitions

**Core Definition**

The `first` modifier places a newly added column at the beginning of the table's column order.

**Technical Definition**

`first()` compiles to the `FIRST` clause in MySQL's `ALTER TABLE` syntax. It is MySQL-specific, as confirmed by Laravel's documentation and community sources. Like `after`, it is only meaningful when adding a column to an existing table.

**Beginner-Friendly Explanation**

`first` tells MySQL to put the new column at the very front of the table.

#### Purposes

- To position a column at the start of a table for visibility or logical grouping.
- To control column order when adding columns to an existing MySQL table.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('priority')->first();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->first()` | Places the column first. |

**Syntax Rules**

- MySQL only.
- Used within `Schema::table()` when adding a column.
- Cannot be combined meaningfully with `after` on the same column.

**Constraints and Limitations**

- MySQL/MariaDB only.
- May trigger a table rebuild on large tables.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Placing a Column First**

**Code**

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('uuid')->first();
});
```

**Expected Output (MySQL)**

The `uuid` column is added at the beginning of the `users` table.

**Why This Output Occurs**

`first()` appends the `FIRST` clause to the `ALTER TABLE` statement.

---

#### Real-World Cases with Explanation

**Case 1: Adding an Identifier Column**

A team adds a `uuid` column to an existing table and wants it to appear first for quick visual identification in database tools.

**Explanation**: `first` provides a simple way to control physical column order on MySQL.

---

### 9. `generatedAs`

#### Definitions

**Core Definition**

The `generatedAs` modifier creates a PostgreSQL identity column with specified sequence options.

**Technical Definition**

`generatedAs(Expression $expression = 'null')` compiles to `GENERATED ... AS IDENTITY` on PostgreSQL. It is used for identity columns, which are the SQL-standard replacement for `SERIAL`. The modifier accepts sequence options via the `Expression` parameter. Laravel's documentation describes it as creating "an identity column with specified sequence options (PostgreSQL)." The `always()` modifier determines whether the column is `GENERATED ALWAYS` or `GENERATED BY DEFAULT`.

**Beginner-Friendly Explanation**

`generatedAs` tells PostgreSQL to create an identity column—a modern, standard way of auto-generating unique numbers.

#### Purposes

- To create SQL-standard identity columns on PostgreSQL.
- To specify sequence options for fine-grained control over identity generation.
- To replace legacy `SERIAL` columns with the modern `GENERATED AS IDENTITY` syntax.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->bigInteger('id')->generatedAs();
$table->bigInteger('id')->generatedAs()->always();
$table->bigInteger('id')->generatedAs()->always(false);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->generatedAs()` | Creates a default identity column. |
| `->generatedAs($expression)` | Creates an identity column with sequence options. |
| `->always()` | Produces `GENERATED ALWAYS AS IDENTITY`. |
| `->always(false)` | Produces `GENERATED BY DEFAULT AS IDENTITY`. |

**Syntax Rules**

- PostgreSQL only.
- The column type must be an integer type compatible with sequences (`smallint`, `integer`, `bigint`).
- Typically paired with `always()` to define precedence.

**Constraints and Limitations**

- PostgreSQL only.
- Not a synonym for `autoIncrement()`; it is the PostgreSQL-specific identity syntax.
- Misuse with non-integer columns causes errors.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Identity Column with Default Behaviour**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->bigInteger('id')->generatedAs();
    $table->string('name');
});
```

**Expected Output (PostgreSQL)**

```sql
CREATE TABLE users (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY,
    name VARCHAR(255) NOT NULL
);
```

**Why This Output Occurs**

`generatedAs()` without `always()` defaults to `GENERATED BY DEFAULT AS IDENTITY` on PostgreSQL.

---

#### Real-World Cases with Explanation

**Case 1: Modern PostgreSQL Schema**

A team building a new PostgreSQL application uses `generatedAs()->always()` instead of `SERIAL` for primary keys, aligning with SQL-standard identity columns and gaining finer control over sequence behaviour.

**Explanation**: `generatedAs` future-proofs the schema and avoids the limitations of legacy `SERIAL` columns.

---

### 10. `invisible`

#### Definitions

**Core Definition**

The `invisible` modifier marks a MySQL column as invisible, meaning it is excluded from `SELECT *` queries unless explicitly referenced.

**Technical Definition**

`invisible()` compiles to the `INVISIBLE` column attribute introduced in MySQL 8.0.23. Invisible columns are not returned by `SELECT *` and are therefore not hydrated into Eloquent models by default, but they can still be explicitly selected. Laravel added support for this modifier in version 8.76.

**Beginner-Friendly Explanation**

`invisible` hides a column from automatic `SELECT *` queries. It is still there, but the database pretends it does not exist unless you specifically ask for it.

#### Purposes

- To hide implementation-detail columns from default queries.
- To reduce data transfer for columns that are rarely needed.
- To prevent sensitive or internal columns from being accidentally exposed in API responses.
- To improve query performance by omitting unused columns from `SELECT *`.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('internal_note')->invisible();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->invisible()` | Marks the column as invisible. |

**Syntax Rules**

- MySQL 8.0.23+ only.
- The column remains accessible via explicit `SELECT` statements.
- Eloquent will not hydrate invisible columns unless explicitly selected.

**Constraints and Limitations**

- MySQL only.
- Requires MySQL 8.0.23 or later.
- Invisible columns still occupy storage and participate in indexes.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Hiding an Internal Column**

**Code**

```php
Schema::create('subscribers', function (Blueprint $table) {
    $table->id();
    $table->string('email');
    $table->string('email_suffix')->invisible(); // Internal column
});
```

**Expected Output (MySQL)**

`SELECT * FROM subscribers` returns only `id` and `email`. The `email_suffix` column must be explicitly selected.

**Why This Output Occurs**

`invisible()` marks the column with the `INVISIBLE` attribute, excluding it from wildcard projections.

---

#### Real-World Cases with Explanation

**Case 1: Internal Computed Suffix**

A team stores an internal `email_suffix` column for faster domain-based searches. They mark it `invisible()` so it does not clutter normal model hydration or API responses, while still allowing explicit queries to use it.

**Explanation**: `invisible` is ideal for columns that are implementation details rather than user-facing data.

---

### 11. `nullable`

#### Definitions

**Core Definition**

The `nullable` modifier allows `NULL` values to be stored in a column.

**Technical Definition**

`nullable(bool $value = true)` compiles to the `NULL` (or omits `NOT NULL`) clause. By default, Laravel columns are `NOT NULL`; `nullable()` removes this restriction. Passing `nullable(false)` explicitly sets `NOT NULL`. Laravel's documentation describes it as allowing "NULL values to be inserted into the column."

**Beginner-Friendly Explanation**

`nullable` means the column is allowed to be empty (contain `NULL`). Without it, the database requires a value.

#### Purposes

- To allow columns to store `NULL` when data is unknown or not applicable.
- To support optional fields in forms and APIs.
- To prevent insertion errors when a value is legitimately absent.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('middle_name')->nullable();
$table->string('required_field')->nullable(false); // Explicit NOT NULL
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->nullable()` | Allows `NULL` (default `true`). |
| `->nullable(false)` | Explicitly disallows `NULL`. |

**Syntax Rules**

- Chainable onto any column type.
- When modifying a column with `change()`, re-declare `nullable()` if you want to keep nullability.

**Constraints and Limitations**

- A `NULL` value is distinct from an empty string or `0`; application logic must handle both.
- Indexes on nullable columns may behave differently in uniqueness constraints (multiple `NULL`s are often allowed).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Optional and Required Columns**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');                       // NOT NULL by default
    $table->string('nickname')->nullable();       // NULL allowed
    $table->string('email')->nullable(false);     // Explicit NOT NULL
});
```

**Expected Output**

`name` and `email` reject `NULL`; `nickname` accepts `NULL`.

**Why This Output Occurs**

Laravel columns are `NOT NULL` by default. `nullable()` removes the constraint, while `nullable(false)` makes it explicit.

---

#### Real-World Cases with Explanation

**Case 1: Optional Profile Fields**

A user profile has optional fields like `bio` and `website`. Marking them `nullable()` allows users to leave them blank without database errors.

**Explanation**: `nullable` is the standard way to model optional data in relational schemas.

---

### 12. `storedAs`

#### Definitions

**Core Definition**

The `storedAs` modifier creates a stored generated column whose value is computed by the database when a row is inserted or updated, and then persisted to disk.

**Technical Definition**

`storedAs(string $expression)` compiles to `GENERATED ALWAYS AS (expression) STORED`. Stored generated columns are computed on write and stored physically, making them indexable and queryable like normal columns. Laravel's documentation lists support for MySQL, with later documentation indicating PostgreSQL and SQLite support as well. The expression is a raw SQL expression provided as a string.

**Beginner-Friendly Explanation**

`storedAs` tells the database to automatically calculate a column's value whenever a row is saved and store the result. For example, a `full_name` column could be automatically computed from `first_name` and `last_name`.

#### Purposes

- To compute and persist derived values at the database level.
- To create indexable computed columns (unlike virtual columns on some databases).
- To eliminate application-level mutators for simple calculations.
- To ensure consistency of derived data across all writes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('full_name')->storedAs("CONCAT(first_name, ' ', last_name)");
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->storedAs($expression)` | Creates a stored generated column with the given SQL expression. |

**Syntax Rules**

- The expression must be a valid SQL expression for the target database.
- The column type should match the expression's result type.
- Stored columns are computed on `INSERT` and `UPDATE`.

**Constraints and Limitations**

- Supported on MySQL, PostgreSQL (as `STORED` generated columns), and SQLite.
- Expressions are database-specific and may not be portable.
- Cannot reference other generated columns in some databases.
- Changing the expression requires dropping and recreating the column.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Stored Full Name**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('first_name');
    $table->string('last_name');
    $table->string('full_name')->storedAs("CONCAT(first_name, ' ', last_name)");
});
```

**Expected Output (MySQL/PostgreSQL)**

The `full_name` column is automatically populated with the concatenated first and last names on insert and update.

**Why This Output Occurs**

`storedAs` generates a `GENERATED ALWAYS AS (...) STORED` column, so the database computes and stores the value.

---

#### Real-World Cases with Explanation

**Case 1: Search-Optimized Full Name**

A user directory needs to search by full name. A stored generated `full_name` column allows efficient indexing and querying without recalculating the concatenation at query time.

**Explanation**: Stored generated columns are ideal for frequently queried derived values.

---

### 13. `unsigned`

#### Definitions

**Core Definition**

The `unsigned` modifier restricts an integer column to non-negative values only, effectively doubling the maximum positive range for a given storage size.

**Technical Definition**

`unsigned()` compiles to the `UNSIGNED` attribute on MySQL/MariaDB. Laravel's documentation confirms it is MySQL-specific. On other databases, the modifier is typically ignored or has no effect. Unsigned columns reject negative values and are commonly used for foreign keys and IDs.

**Beginner-Friendly Explanation**

`unsigned` means the column can only hold positive numbers (and zero). It cannot hold negative numbers.

#### Purposes

- To enforce non-negative values at the database level.
- To double the maximum positive range for integer types.
- To align foreign key columns with unsigned primary keys.
- To prevent accidental insertion of negative values.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->integer('votes')->unsigned();
$table->bigInteger('user_id')->unsigned();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->unsigned()` | Marks the integer column as unsigned. |

**Syntax Rules**

- Applies only to integer-type columns.
- On MySQL, `UNSIGNED` columns have no negative range.
- Laravel's `foreignId()` automatically creates unsigned big integers.

**Constraints and Limitations**

- MySQL/MariaDB only according to the documentation.
- PostgreSQL does not support unsigned integers; use `CHECK (column >= 0)` instead.
- Unsigned columns cannot store negative values, which may cause errors if application logic expects them.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Unsigned Foreign Key**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('user_id');
});
```

**Expected Output (MySQL)**

The `user_id` column rejects negative values.

**Why This Output Occurs**

`unsigned()` appends `UNSIGNED` to the column definition, restricting the range to 0 and above.

---

#### Real-World Cases with Explanation

**Case 1: Matching Foreign Key Types**

A posts table references an unsigned `bigIncrements` primary key. The `user_id` foreign key must also be unsigned to match, preventing referential integrity issues.

**Explanation**: `unsigned` ensures type compatibility between primary and foreign keys on MySQL.

---

### 14. `useCurrent`

#### Definitions

**Core Definition**

The `useCurrent` modifier sets a `TIMESTAMP` column's default value to the current timestamp (`CURRENT_TIMESTAMP`).

**Technical Definition**

`useCurrent()` compiles to `DEFAULT CURRENT_TIMESTAMP`. It is commonly applied to `created_at` columns but can be used on any `timestamp` column. Laravel's documentation describes it as setting "TIMESTAMP columns to use CURRENT_TIMESTAMP as default value."

**Beginner-Friendly Explanation**

`useCurrent` tells the database: "if no value is provided for this timestamp column, use the current date and time."

#### Purposes

- To automatically record the creation time of a row.
- To ensure timestamp columns have a meaningful default.
- To reduce application-level timestamp assignment.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->timestamp('created_at')->useCurrent();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->useCurrent()` | Sets `DEFAULT CURRENT_TIMESTAMP`. |

**Syntax Rules**

- Applies to `timestamp` columns.
- Typically combined with `useCurrentOnUpdate()` for `updated_at` columns.

**Constraints and Limitations**

- Only works on `timestamp` columns, not `dateTime` columns on some databases.
- The database must support `CURRENT_TIMESTAMP` as a default (most modern databases do).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Auto-Populated Creation Timestamp**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->timestamp('created_at')->useCurrent();
});
```

**Expected Output**

When a post is inserted without a `created_at` value, the database sets it to the current timestamp.

**Why This Output Occurs**

`useCurrent()` appends `DEFAULT CURRENT_TIMESTAMP` to the column definition.

---

#### Real-World Cases with Explanation

**Case 1: Audit Logging**

An audit table uses `timestamp('occurred_at')->useCurrent()` to automatically record the exact time of each logged event without application intervention.

**Explanation**: `useCurrent` simplifies timestamp management and ensures consistency.

---

### 15. `useCurrentOnUpdate`

#### Definitions

**Core Definition**

The `useCurrentOnUpdate` modifier sets a `TIMESTAMP` column to automatically update to the current timestamp whenever the row is updated.

**Technical Definition**

`useCurrentOnUpdate()` compiles to `ON UPDATE CURRENT_TIMESTAMP` on MySQL. It was added in Laravel 8.36.0 and is typically combined with `useCurrent()` on `updated_at` columns. Laravel's documentation describes it as setting "TIMESTAMP columns to use CURRENT_TIMESTAMP when a record is updated."

**Beginner-Friendly Explanation**

`useCurrentOnUpdate` tells MySQL: "whenever this row changes, automatically set this column to the current time."

#### Purposes

- To automatically maintain an `updated_at` timestamp without application logic.
- To track the last modification time of each row.
- To reduce Eloquent overhead for timestamp management.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->timestamp('updated_at')->useCurrent()->useCurrentOnUpdate();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->useCurrent()` | Sets the default to `CURRENT_TIMESTAMP`. |
| `->useCurrentOnUpdate()` | Adds `ON UPDATE CURRENT_TIMESTAMP`. |

**Syntax Rules**

- MySQL/MariaDB specific.
- Requires Laravel 8.36.0 or later.
- Best used on `timestamp` columns.

**Constraints and Limitations**

- MySQL only.
- Does not fire on `UPDATE` statements that do not change any values in some MySQL configurations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Auto-Updating Timestamp**

**Code**

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->timestamp('created_at')->useCurrent();
    $table->timestamp('updated_at')->useCurrent()->useCurrentOnUpdate();
});
```

**Expected Output (MySQL)**

`updated_at` is set to the current time on insert and automatically updated on every row modification.

**Why This Output Occurs**

The combined modifiers produce `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP`, implementing MySQL's automatic timestamp behaviour.

---

#### Real-World Cases with Explanation

**Case 1: Database-Level Audit Trail**

A team wants the database itself to track when rows are modified, without relying on Eloquent's `$timestamps`. They use `useCurrentOnUpdate()` on `updated_at` to ensure the timestamp is updated even for raw SQL updates.

**Explanation**: Database-level timestamp management is more resilient than application-level timestamping.

---

### 16. `virtualAs`

#### Definitions

**Core Definition**

The `virtualAs` modifier creates a virtual generated column whose value is computed by the database at query time but not stored on disk.

**Technical Definition**

`virtualAs(string $expression)` compiles to `GENERATED ALWAYS AS (expression) VIRTUAL`. Virtual generated columns are computed on the fly during `SELECT` queries and are not persisted. Laravel's documentation lists support for MySQL and SQLite; PostgreSQL support for virtual columns was added in PostgreSQL 18, with Laravel's modifier previously generating invalid SQL on older PostgreSQL versions.

**Beginner-Friendly Explanation**

`virtualAs` creates a column whose value is calculated every time you query it. It is not stored, so it saves disk space but may be slower to query than a stored column.

#### Purposes

- To compute derived values without storing them.
- To save disk space for columns that are infrequently queried.
- To create calculated columns that always reflect the latest data.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$table->string('full_name')->virtualAs("CONCAT(first_name, ' ', last_name)");
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `->virtualAs($expression)` | Creates a virtual generated column. |

**Syntax Rules**

- The expression must be a valid SQL expression.
- Virtual columns are computed at query time.
- Cannot be indexed on some databases (including MySQL before certain versions).

**Constraints and Limitations**

- Not all databases support virtual generated columns.
- Cannot be used as a primary key.
- Indexing virtual columns is restricted on MySQL (requires stored columns for indexing).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Virtual Full Name**

**Code**

```php
Schema::create('users', function (Blueprint $table) {
    $table->string('first_name');
    $table->string('last_name');
    $table->string('full_name')->virtualAs("CONCAT(first_name, ' ', last_name)");
});
```

**Expected Output**

Querying `full_name` returns the concatenated name computed at query time; it is not stored in the table.

**Why This Output Occurs**

`virtualAs` generates a `GENERATED ALWAYS AS (...) VIRTUAL` column, so the value is calculated per query.

---

#### Real-World Cases with Explanation

**Case 1: Infrequently Queried Derived Value**

A table has a `full_name` column that is only used in occasional reports. `virtualAs` avoids storing the value on every row, saving disk space while still providing the computed result when needed.

**Explanation**: Virtual columns are appropriate for derived values that are not queried frequently or indexed.

---

## Column Modifiers Best Practices

1. **Re-Declare Modifiers When Using `change()`**: When modifying an existing column, you must explicitly include every modifier you want to keep; otherwise, they are dropped. Laravel 11+ requires all attributes to be present during modification.

2. **Be Aware of Database Support**: Modifiers like `after`, `first`, `charset`, `unsigned`, `invisible`, and `useCurrentOnUpdate` are MySQL-specific. Check the documentation for your database driver before relying on them.

3. **Prefer `nullable()` with `default(null)` for Optional Fields**: This combination clearly expresses that a column is optional and defaults to `null`.

4. **Use `Expression` Defaults Sparingly**: While `default(new Expression('...'))` enables powerful database functions, it couples your schema to a specific database driver and may reduce portability.

5. **Prefer `storedAs` for Indexed Computed Columns**: Virtual columns cannot always be indexed; stored generated columns can. Choose `storedAs` when you need to index or frequently query the computed value.

6. **Use `useCurrent()` and `useCurrentOnUpdate()` for Audit Columns**: These modifiers delegate timestamp management to the database, reducing application code and ensuring consistency.

7. **Reserve `invisible()` for Implementation Details**: Invisible columns are useful for internal or rarely accessed data, but they can confuse developers who are unaware of their existence. Document them clearly.

8. **Use `generatedAs()->always()` for Strict PostgreSQL Identity Columns**: When you want to prevent explicit ID insertion, `always()` enforces database-generated values. Use `always(false)` for import-friendly behaviour.

9. **Keep Modifier Chains Readable**: While modifiers are chainable, long chains can become hard to read. Consider using the grouping helper for `after` or breaking complex definitions into multiple statements.

10. **Test Migrations on the Target Database**: Modifier behaviour varies across database engines and versions. Always test migrations on the database you will deploy to.

---

## References

- Laravel 10.x Migrations Documentation: Column Modifiers — https://github.com/DevStorm-Team/laravel-book/blob/7a980beccf7933f873e383c4dbc4aaa8e9319b6f/laravel-docs-10.x.pdf
- Laravel Migrations Documentation: Column Modifiers (raw Markdown) — https://raw.githubusercontent.com/laravel/docs/16e64aa8bb1c706ecd867f101dbd1c83c6acf1a6/migrations.md
- Laravel Migrations Documentation: Column Modifiers (blame view) — https://github.com/laravel/docs/blame/5cd91818357436e34b683c8750f1e72f73252cd3/migrations.md
- Laravel 10.x PR #45487: Native Column Modifying — https://github.com/laravel/framework/pull/45487
- Laravel News: Invisible Modifier for MySQL Columns — https://laravel-news.com/laravel-8-76-released
- Laravel Daily: Virtual DB Columns in Laravel Migrations and MySQL — https://laraveldaily.com/post/virtual-db-columns-laravel-migrations-mysql
- Laracasts: Postgres `generatedAs` and `storedAs` in Laravel Migration — https://laracasts.com/discuss/channels/laravel/postgres-generatedas-and-storedas-in-laravel-migaration
- Stack Overflow: Postgres `generatedAs` and `storedAs` in Laravel Migration — https://stackoverflow.com/questions/78890000/postgres-generatedas-and-storedas-in-laravel-migaration
- Stack Overflow: Laravel schema builder add column at start of table — https://stackoverflow.com/questions/21198125/laravel-schema-builder-add-column-at-start-of-table
- Stack Overflow: How to set the default value of a timestamp column to the current timestamp — https://stackoverflow.com/questions/18926900/how-can-i-set-the-default-value-of-a-timestamp-column-to-the-current-timestamp-wi
- Stack Overflow: Changing timestamp columns with `useCurrent` — https://laracasts.com/discuss/channels/laravel/changing-timestamp-columns-with-useCurrent
- Laravel API: ColumnDefinition — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/ColumnDefinition.html
- Laravel API: ForeignIdColumnDefinition — https://api.laravel.com/docs/12.x/Illuminate/Database/Schema/ForeignIdColumnDefinition.html

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Column modifier availability varies significantly by database driver. `after`, `first`, `charset`, `unsigned`, `invisible`, and `useCurrentOnUpdate` are MySQL/MariaDB-specific. `generatedAs` and `always` are PostgreSQL-specific. `storedAs` is supported on MySQL, PostgreSQL, and SQLite. `virtualAs` is supported on MySQL and SQLite, with PostgreSQL 18+ support emerging. Developers should always consult the documentation and test migrations against their target database version.