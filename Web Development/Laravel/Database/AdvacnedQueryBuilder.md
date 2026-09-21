# Laravel Fluent Query Builder: Core CRUD & Filtering — A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel's Fluent Query Builder is a database-agnostic, object-oriented interface for constructing and executing SQL queries using method chaining, providing a unified API that works identically across all supported database systems without writing raw SQL.

**Technical Definition:** The `Illuminate\Database\Query\Builder` class provides a fluent interface for building SQL statements programmatically. It uses PDO parameter binding throughout to protect against SQL injection attacks, and delegates grammar-specific compilation to driver-specific `Grammar` classes (MySqlGrammar, PostgresGrammar, SQLiteGrammar, SqlServerGrammar). The builder is accessed via the `DB` facade's `table()` method, which returns a new builder instance. All query components — selects, joins, wheres, groups, havings, orders, limits, and offsets — are accumulated on the builder and compiled into SQL only when a terminal method (get, first, insert, update, delete, etc.) is called.

**Beginner-Friendly Explanation:** The Query Builder lets you build database queries using PHP method chains instead of writing SQL by hand. Instead of writing `SELECT * FROM users WHERE age > 18 ORDER BY name`, you write `DB::table('users')->where('age', '>', 18)->orderBy('name')->get()`. Laravel translates this into the correct SQL for your database — MySQL, PostgreSQL, SQLite, or SQL Server — automatically. Because it uses parameter binding, your queries are protected against SQL injection without any manual sanitisation.

### Key Characteristics

- **Database Agnostic:** The same query builder code works across MySQL, PostgreSQL, SQLite, SQL Server, and MariaDB.
- **SQL Injection Protection:** All values are bound as PDO parameters — user input never becomes part of the raw SQL string.
- **Fluent Interface:** Every method returns the builder instance, enabling method chaining for complex queries.
- **Lazy Compilation:** The SQL statement is only compiled when a terminal method is called, allowing conditional query building.
- **Collection Results:** `get()` returns an `Illuminate\Support\Collection` of `stdClass` objects, providing access to all of Laravel's collection methods.
- **Chunking & Streaming:** Large result sets can be processed in chunks via `chunk()` or streamed lazily via `lazy()` and `cursor()`.
- **Conditional Clauses:** The `when()` method enables conditional query construction based on runtime values.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (the Query Builder API is stable across versions).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12) with the appropriate PDO extension.
- A configured database connection in `config/database.php` and `.env`.
- Basic understanding of SQL and database concepts.

### Related Programming Areas

- **Eloquent ORM:** Eloquent is built on top of the Query Builder; every Eloquent query is ultimately a Query Builder query.
- **Migrations & Schema:** The Schema Builder uses the same connection layer as the Query Builder.
- **Database Transactions:** Query Builder operations can be wrapped in transactions via `DB::transaction()`.
- **Raw Expressions:** `DB::raw()` and `selectRaw()`, `whereRaw()`, and `havingRaw()` allow embedding raw SQL fragments within the fluent interface.
- **Pagination:** The Query Builder's `paginate()` method integrates with Laravel's pagination system.

### Core Concepts / Features

1. Data Retrieval
2. Data Modification
3. Advanced Inserts
4. Filtering & Constraints
5. Sorting & Grouping
6. Aggregates

---

## 1. Data Retrieval

### Definitions

**Core Definition:** Data retrieval in the Query Builder encompasses the methods used to fetch rows, columns, and single values from database tables, including full result sets, single rows, single column values, and specific column selections.

**Technical Definition:** The `get()` method executes the compiled SELECT statement and returns an `Illuminate\Support\Collection` of `stdClass` objects, where each object represents a row and its properties correspond to column names. The `first()` method adds a `LIMIT 1` clause and returns a single `stdClass` object or `null`. The `find()` method is a shortcut for `where('id', $id)->first()`. The `pluck()` method retrieves a single column's values as a collection, optionally keyed by another column. The `value()` method retrieves a single column's value from the first result. The `select()` method constrains the columns returned in the result set.

**Beginner-Friendly Explanation:** When you need to read data from your database, the Query Builder gives you several options. `get()` fetches all matching rows. `first()` fetches just the first one. `find()` fetches a row by its primary key. `pluck()` grabs a single column from all rows as a simple list. `value()` grabs a single value from the first matching row. These methods cover every common data retrieval scenario.

### Purposes

- To retrieve all rows from a table using `get()` for full result sets.
- To retrieve a single row using `first()` when only one record is needed.
- To retrieve a record by its primary key using `find()` as a convenient shortcut.
- To extract a single column's values as a collection using `pluck()`.
- To extract a single value from the first matching row using `value()`.
- To limit the columns returned via `select()` for performance optimisation.

### Syntax Rules and Structure

**Complete General Syntax — `get()`:**

```php
$users = DB::table('users')->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `DB::table('users')` | Begins a query against the `users` table. |
| `->get()` | Executes the query and returns a Collection of stdClass objects. |

**Complete General Syntax — `first()`:**

```php
$user = DB::table('users')->where('name', 'John')->first();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->where('name', 'John')` | Adds a WHERE condition. |
| `->first()` | Adds LIMIT 1 and returns a single stdClass or null. |

**Complete General Syntax — `find()`:**

```php
$user = DB::table('users')->find(3);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->find(3)` | Shorthand for `where('id', 3)->first()`. |

**Complete General Syntax — `pluck()`:**

```php
$names = DB::table('users')->pluck('name');
$names = DB::table('users')->pluck('name', 'id');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->pluck('name')` | Returns a Collection of `name` values. |
| `->pluck('name', 'id')` | Returns a Collection keyed by `id` with `name` values. |

**Complete General Syntax — `value()`:**

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

**Complete General Syntax — `select()`:**

```php
$users = DB::table('users')->select('name', 'email')->get();
$users = DB::table('users')->select('name as user_name')->get();
```

**Syntax Rules:**

- `get()` returns a Collection of `stdClass` objects; access columns as `$user->name`.
- `first()` returns a single `stdClass` object or `null` if no rows match.
- `find()` accepts a single ID or an array of IDs; with an array, it returns a Collection.
- `pluck()` returns a Collection of values; when a second argument is provided, the collection is keyed by that column.
- `value()` returns the value of a single column from the first matching row.
- `select()` can be called with multiple column names or with column aliases (`'name as user_name'`).
- The `addSelect()` method adds columns to an existing select clause.

**Constraints and Limitations:**

- **`find()` assumes primary key is `id`:** `find()` is a shortcut for `where('id', $id)`; use `where()` explicitly if the primary key has a different name.
- **`pluck()` returns a Collection, not an array:** Use `->pluck('name')->toArray()` if an array is needed.
- **`value()` returns `null` if no rows match:** No exception is thrown for missing records.
- **`select()` limits columns:** Only the specified columns are retrieved; accessing undefined columns on the returned objects results in `null`.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Data Retrieval**

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;

class UserController extends Controller
{
    public function index()
    {
        // Retrieve all users
        $users = DB::table('users')->get();

        foreach ($users as $user) {
            echo $user->name . '<br>';
        }

        // Retrieve a single user by name
        $john = DB::table('users')->where('name', 'John')->first();
        echo $john->email;

        // Retrieve a user by ID
        $user = DB::table('users')->find(3);
        echo $user->name;
    }
}
```

**Expected Output:** The `get()` call returns all users; `first()` returns John's record; `find(3)` returns the user with ID 3.

**Why:** Each method retrieves data differently: `get()` for all rows, `first()` for one matching row, and `find()` for a row by primary key.

---

**Example 2: Retrieving Single Columns**

```php
<?php

use Illuminate\Support\Facades\DB;

// Get all user names as a Collection
$names = DB::table('users')->pluck('name');
// Collection: ['Alice', 'Bob', 'Charlie']

// Get user names keyed by their IDs
$names = DB::table('users')->pluck('name', 'id');
// Collection: [1 => 'Alice', 2 => 'Bob', 3 => 'Charlie']

// Get a single column value from the first matching row
$email = DB::table('users')->where('name', 'Alice')->value('email');
// 'alice@example.com'

// Select specific columns
$users = DB::table('users')->select('name', 'email')->get();
// Each object has only name and email properties
```

**Expected Output:** `pluck()` returns a flat collection of values; `value()` returns a single scalar; `select()` limits the columns returned.

**Why:** `pluck()` is ideal for dropdown lists or simple value extraction; `value()` for a single scalar; `select()` for reducing the amount of data transferred.

---

**Example 3: Using `find()` with Multiple IDs**

```php
<?php

use Illuminate\Support\Facades\DB;

// Find multiple users by their IDs
$users = DB::table('users')->find([1, 2, 3]);

foreach ($users as $user) {
    echo $user->name;
}
```

**Expected Output:** A Collection containing the users with IDs 1, 2, and 3.

**Why:** Passing an array to `find()` returns multiple records, equivalent to `whereIn('id', [1, 2, 3])->get()`.

### Real-World Cases

- **User Lists:** `DB::table('users')->get()` for admin user management pages.
- **Profile Pages:** `DB::table('users')->find($id)` for displaying a single user's profile.
- **Dropdown Options:** `DB::table('roles')->pluck('name', 'id')` for populating select dropdowns.
- **Single Settings:** `DB::table('settings')->where('key', 'site_name')->value('value')` for retrieving configuration values.
- **API Endpoints:** `DB::table('posts')->select('id', 'title', 'slug')->get()` for JSON API responses.

### References

- Laravel Query Builder: Retrieving All Rows — https://laravel.com/docs/12.x/queries#retrieving-all-rows-from-a-table
- Laravel Query Builder: Retrieving A Single Row — https://laravel.com/docs/12.x/queries#retrieving-a-single-row-column-from-a-table
- Laravel Query Builder: Retrieving A Single Column — https://laravel.com/docs/12.x/queries#retrieving-a-single-row-column-from-a-table
- Laravel Query Builder: Select Statements — https://laravel.com/docs/12.x/queries#select-statements

---

## 2. Data Modification

### Definitions

**Core Definition:** Data modification in the Query Builder encompasses the methods used to insert new records, update existing records, delete records, and perform atomic numeric operations (increment/decrement) on database tables.

**Technical Definition:** The `insert()` method accepts an array of column-value pairs (or an array of arrays for bulk inserts) and executes an INSERT statement. The `insertGetId()` method inserts a record and returns the auto-incrementing ID. The `update()` method accepts an array of column-value pairs and executes an UPDATE statement against all rows matching the current WHERE conditions. The `delete()` method executes a DELETE statement against all rows matching the current WHERE conditions. The `increment()` and `decrement()` methods atomically add or subtract a value from a numeric column, optionally applying additional WHERE conditions.

**Beginner-Friendly Explanation:** These methods let you change data in your database. `insert()` adds new records, `update()` changes existing ones, `delete()` removes them, and `increment()`/`decrement()` increase or decrease numeric values (like a vote count or stock level) in a single atomic operation — no race conditions.

### Purposes

- To insert new records into a table using `insert()`.
- To insert a record and retrieve its auto-generated ID using `insertGetId()`.
- To update existing records matching a WHERE condition using `update()`.
- To delete records matching a WHERE condition using `delete()`.
- To atomically increment a numeric column using `increment()`.
- To atomically decrement a numeric column using `decrement()`.
- To perform bulk inserts efficiently with a single query.

### Syntax Rules and Structure

**Complete General Syntax — `insert()`:**

```php
DB::table('users')->insert([
    'email' => 'john@example.com',
    'name' => 'John Doe',
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->insert([...])` | Inserts a single record with the given column-value pairs. |
| Returns | `true` on success. |

**Complete General Syntax — Bulk `insert()`:**

```php
DB::table('users')->insert([
    ['email' => 'a@example.com', 'name' => 'Alice'],
    ['email' => 'b@example.com', 'name' => 'Bob'],
]);
```

**Complete General Syntax — `insertGetId()`:**

```php
$id = DB::table('users')->insertGetId([
    'email' => 'john@example.com',
    'name' => 'John Doe',
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->insertGetId([...])` | Inserts a record and returns the auto-increment ID. |

**Complete General Syntax — `update()`:**

```php
DB::table('users')
    ->where('id', 1)
    ->update(['votes' => 100]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->update([...])` | Updates matching rows with the given column-value pairs. |
| Returns | Number of affected rows. |

**Complete General Syntax — `delete()`:**

```php
DB::table('users')->where('votes', '>', 100)->delete();
DB::table('users')->delete(); // Deletes all rows
DB::table('users')->truncate(); // Removes all rows and resets auto-increment
```

**Complete General Syntax — `increment()` / `decrement()`:**

```php
DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);
DB::table('users')->decrement('votes');
DB::table('users')->decrement('votes', 5);

// With additional columns to update
DB::table('users')->increment('votes', 1, ['updated_at' => now()]);
```

**Syntax Rules:**

- `insert()` accepts a single associative array or an array of associative arrays for bulk inserts.
- `insertGetId()` returns the auto-increment ID (works with MySQL, PostgreSQL, and SQL Server).
- `update()` requires a WHERE condition unless you intend to update all rows.
- `delete()` requires a WHERE condition unless you intend to delete all rows.
- `increment()` and `decrement()` are atomic — no race conditions.
- The third argument to `increment()`/`decrement()` allows updating additional columns in the same query.

**Constraints and Limitations:**

- **`update()` without WHERE:** Calling `update()` without a WHERE clause updates every row in the table. Always include a WHERE condition unless updating all rows intentionally.
- **`delete()` without WHERE:** Same as above — `delete()` without WHERE removes all rows.
- **`insertGetId()` and PostgreSQL:** PostgreSQL's `insertGetId()` uses `RETURNING id`, so the column name must be `id` or specified explicitly.
- **`truncate()`:** Truncating a table is not transactional in some databases and cannot be rolled back.

### Multiple Annotated Complete Code Examples

**Example 1: Inserting Records**

```php
<?php

use Illuminate\Support\Facades\DB;

// Insert a single user
DB::table('users')->insert([
    'email' => 'john@example.com',
    'name' => 'John Doe',
    'created_at' => now(),
    'updated_at' => now(),
]);

// Insert multiple users in one query
DB::table('users')->insert([
    ['email' => 'alice@example.com', 'name' => 'Alice', 'created_at' => now()],
    ['email' => 'bob@example.com', 'name' => 'Bob', 'created_at' => now()],
]);

// Insert and get the new ID
$id = DB::table('users')->insertGetId([
    'email' => 'carol@example.com',
    'name' => 'Carol',
]);
// $id = 4 (auto-increment value)
```

**Expected Output:** Records are inserted into the `users` table; `insertGetId()` returns the new row's ID.

**Why:** `insert()` is for adding records; `insertGetId()` is for when you need to know the generated primary key immediately.

---

**Example 2: Updating Records**

```php
<?php

use Illuminate\Support\Facades\DB;

// Update a specific user
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['votes' => 100, 'updated_at' => now()]);
// $affected = 1

// Update multiple users matching a condition
$affected = DB::table('users')
    ->where('status', 'inactive')
    ->update(['status' => 'archived']);
// $affected = 5 (number of rows updated)

// Increment a column atomically
DB::table('posts')->where('id', 5)->increment('views');
// views = views + 1

// Increment by a specific amount
DB::table('products')->where('id', 10)->decrement('stock', 3);
// stock = stock - 3
```

**Expected Output:** The `update()` method returns the number of affected rows. `increment()` and `decrement()` atomically modify numeric columns.

**Why:** `update()` modifies existing records; `increment()`/`decrement()` are atomic operations safe for concurrent access.

---

**Example 3: Deleting Records**

```php
<?php

use Illuminate\Support\Facades\DB;

// Delete a specific user
$deleted = DB::table('users')->where('id', 1)->delete();
// $deleted = 1

// Delete inactive users
$deleted = DB::table('users')->where('last_login', '<', now()->subYear())->delete();

// Delete all rows (use with caution)
DB::table('temp_data')->delete();

// Truncate (reset auto-increment and remove all rows)
DB::table('temp_data')->truncate();
```

**Expected Output:** `delete()` returns the number of deleted rows. `truncate()` removes all rows and resets the auto-increment counter.

**Why:** `delete()` is for selective removal; `truncate()` is for completely emptying a table.

### Real-World Cases

- **User Registration:** `insertGetId()` for creating user accounts and retrieving the new user ID.
- **Bulk Import:** `insert()` with an array of arrays for importing CSV data.
- **Profile Updates:** `update()` for saving changes to user profiles.
- **Soft Deletes:** `update(['deleted_at' => now()])` for implementing soft deletes.
- **Counter Updates:** `increment('views')` for tracking page views or post likes.
- **Inventory Management:** `decrement('stock', $quantity)` for reducing stock levels atomically.

### References

- Laravel Query Builder: Insert Statements — https://laravel.com/docs/12.x/queries#insert-statements
- Laravel Query Builder: Update Statements — https://laravel.com/docs/12.x/queries#update-statements
- Laravel Query Builder: Increment and Decrement — https://laravel.com/docs/12.x/queries#increment-and-decrement
- Laravel Query Builder: Delete Statements — https://laravel.com/docs/12.x/queries#delete-statements

---

## 3. Advanced Inserts

### Definitions

**Core Definition:** Advanced insert operations in the Query Builder include `insertOrIgnore()` for skipping duplicate records and `upsert()` for inserting new records or updating existing ones in a single atomic query.

**Technical Definition:** The `insertOrIgnore()` method generates an `INSERT IGNORE` (MySQL), `INSERT OR IGNORE` (SQLite/PostgreSQL), or equivalent statement that silently skips rows that would cause unique constraint violations or other errors. The `upsert()` method generates an `INSERT ... ON DUPLICATE KEY UPDATE` (MySQL), `INSERT ... ON CONFLICT ... DO UPDATE` (PostgreSQL/SQLite), or `MERGE` (SQL Server) statement that inserts new records and updates existing ones based on unique or primary key columns. Both methods accept an array of records (bulk operations) and are significantly more efficient than looping through individual inserts.

**Beginner-Friendly Explanation:** When you're importing data, you often encounter duplicates. `insertOrIgnore()` says "insert this, but if it already exists, skip it." `upsert()` says "insert this, but if it already exists, update it with the new values." Both are much faster than checking each record individually in a loop.

### Purposes

- To insert records while silently skipping duplicates using `insertOrIgnore()`.
- To insert new records or update existing ones in a single query using `upsert()`.
- To perform bulk upserts efficiently when syncing data from external sources.
- To handle idempotent data imports where re-running the import should not create duplicates.
- To implement "create or update" logic without multiple database round-trips.

### Syntax Rules and Structure

**Complete General Syntax — `insertOrIgnore()`:**

```php
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'taylor@example.com'],
    ['id' => 2, 'email' => 'dayle@example.com'],
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->insertOrIgnore([...])` | Inserts records, ignoring duplicate key errors. |
| Returns | Number of rows inserted (may be less than the input array if duplicates were skipped). |

**Complete General Syntax — `upsert()`:**

```php
DB::table('users')->upsert(
    [
        ['email' => 'taylor@example.com', 'votes' => 0, 'name' => 'Taylor'],
        ['email' => 'dayle@example.com', 'votes' => 0, 'name' => 'Dayle'],
    ],
    ['email'],           // Unique columns that identify existing records
    ['votes', 'name']    // Columns to update if a match is found
);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| First argument | Array of records to insert or update. |
| Second argument | Column(s) that uniquely identify existing records. |
| Third argument | Column(s) to update when a match is found. |

**Syntax Rules:**

- `insertOrIgnore()` accepts an array of records (single or bulk).
- `upsert()` accepts three arguments: the records, the unique identifier column(s), and the columns to update.
- The second argument to `upsert()` must correspond to a unique index or primary key in the database.
- `upsert()` works on MySQL, PostgreSQL, SQLite, and SQL Server (with version-specific SQL).
- Both methods return the number of affected rows.

**Constraints and Limitations:**

- **`insertOrIgnore()` ignores all errors:** Depending on the database engine, it may ignore errors other than duplicate key violations (e.g., data truncation). Use with caution.
- **`upsert()` requires a unique index:** The column(s) specified in the second argument must have a unique constraint, or the query will fail.
- **`upsert()` database support:** MySQL 5.7+, PostgreSQL 9.5+, SQLite 3.24+, and SQL Server 2008+ support upserts. Older versions may not support the syntax.
- **`upsert()` on SQL Server:** Uses `MERGE` which has known concurrency issues. Consider alternatives for high-concurrency scenarios.

### Multiple Annotated Complete Code Examples

**Example 1: Bulk `insertOrIgnore()`**

```php
<?php

use Illuminate\Support\Facades\DB;

// Attempt to insert multiple promo codes, ignoring duplicates
DB::table('promo_codes')->insertOrIgnore([
    ['code' => 'SAVE10', 'discount' => 10],
    ['code' => 'SAVE20', 'discount' => 20],
    ['code' => 'SAVE10', 'discount' => 10], // Duplicate — will be skipped
]);

// If 'SAVE10' already exists in the table, it will not be inserted again
```

**Expected Output:** Only non-duplicate records are inserted. If `SAVE10` already exists, only `SAVE20` is inserted.

**Why:** `insertOrIgnore()` generates database-specific SQL that silently ignores duplicate key errors, making it ideal for idempotent data seeding.

---

**Example 2: Bulk `upsert()` for Syncing Data**

```php
<?php

use Illuminate\Support\Facades\DB;

// Sync user data from an external API
$users = [
    ['email' => 'alice@example.com', 'name' => 'Alice Updated', 'votes' => 10],
    ['email' => 'bob@example.com', 'name' => 'Bob New', 'votes' => 5],
    ['email' => 'carol@example.com', 'name' => 'Carol', 'votes' => 0],
];

DB::table('users')->upsert(
    $users,
    ['email'],           // Match on email column
    ['name', 'votes']    // Update these columns if a match is found
);

// If alice@example.com exists, her name and votes are updated.
// If bob@example.com does not exist, a new record is inserted.
// If carol@example.com exists, her name and votes are updated.
```

**Expected Output:** Existing users are updated with new values; new users are inserted. The entire operation is performed in a single query.

**Why:** `upsert()` combines insert and update logic atomically, making it ideal for data synchronisation from external sources.

---

**Example 3: `upsert()` with Composite Unique Key**

```php
<?php

use Illuminate\Support\Facades\DB;

// Upsert inventory records with a composite unique key (warehouse_id + product_id)
DB::table('inventory')->upsert(
    [
        ['warehouse_id' => 1, 'product_id' => 100, 'quantity' => 50],
        ['warehouse_id' => 1, 'product_id' => 101, 'quantity' => 75],
        ['warehouse_id' => 2, 'product_id' => 100, 'quantity' => 30],
    ],
    ['warehouse_id', 'product_id'],  // Composite unique key
    ['quantity']                     // Update quantity on match
);
```

**Expected Output:** Inventory records are inserted or updated based on the combination of `warehouse_id` and `product_id`.

**Why:** `upsert()` supports composite unique keys, allowing precise matching in multi-tenant or multi-location scenarios.

### Real-World Cases

- **Data Import:** `upsert()` for syncing data from CRM, ERP, or external APIs without creating duplicates.
- **Promo Code Seeding:** `insertOrIgnore()` for seeding promo codes that may already exist.
- **Inventory Sync:** `upsert()` for updating stock levels across multiple warehouses.
- **User Synchronisation:** `upsert()` for keeping user records in sync with an identity provider.
- **Idempotent Migrations:** `insertOrIgnore()` for seeding reference data in migrations.

### References

- Laravel Query Builder: Insert or Ignore — https://laravel.com/docs/12.x/queries#insert-statements
- Laravel Query Builder: Upserts — https://laravel.com/docs/12.x/queries#upserts
- Laravel 8 Upsert Support (GitHub PR #34698) — https://github.com/laravel/framework/pull/34698
- Laravel Upsert Package (Packagist) — https://packagist.org/packages/iamirnet/laravel-upsert

---

## 4. Filtering & Constraints

### Definitions

**Core Definition:** Filtering and constraints in the Query Builder are the methods that restrict which rows are affected by a query, including equality and comparison checks, set membership, range checks, null checks, and logical grouping of conditions.

**Technical Definition:** The `where()` method adds a basic constraint with three arguments (column, operator, value) or two arguments (column, value for equality). The `orWhere()` method adds an OR constraint. The `whereIn()` and `whereNotIn()` methods check set membership against an array. The `whereBetween()` and `whereNotBetween()` methods check range inclusion. The `whereNull()` and `whereNotNull()` methods check for NULL values. The `whereDate()`, `whereMonth()`, `whereDay()`, `whereYear()`, and `whereTime()` methods compare date/time components. Logical grouping is achieved by passing a closure to `where()` or `orWhere()`, which wraps the grouped conditions in parentheses.

**Beginner-Friendly Explanation:** These methods are how you tell the database which rows you want. `where()` is the workhorse — it checks if a column equals, is greater than, or is less than a value. `whereIn()` checks if a value is in a list. `whereBetween()` checks if a value is in a range. `whereNull()` checks if a value is empty. And when you need to combine conditions with AND/OR logic, you use closures to group them, just like parentheses in SQL.

### Purposes

- To filter query results based on column values using `where()`.
- To check if a column's value is in a given set using `whereIn()`.
- To check if a column's value falls within a range using `whereBetween()`.
- To check for NULL values using `whereNull()`.
- To combine conditions with OR logic using `orWhere()`.
- To group conditions logically using closures, mirroring SQL parentheses.
- To filter by date components using `whereDate()`, `whereYear()`, etc.

### Syntax Rules and Structure

**Complete General Syntax — Basic `where()`:**

```php
// Three arguments: column, operator, value
$users = DB::table('users')->where('votes', '>', 100)->get();

// Two arguments: column, value (equality)
$users = DB::table('users')->where('votes', 100)->get();

// Operator-based
$users = DB::table('users')->where('votes', '>=', 100)->get();
$users = DB::table('users')->where('votes', '<>', 100)->get();
$users = DB::table('users')->where('name', 'like', 'T%')->get();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `where('col', 'op', 'val')` | Standard three-argument where clause. |
| `where('col', 'val')` | Two-argument shorthand for equality. |
| `orWhere(...)` | Adds an OR condition. |
| `whereNot(...)` | Negates a group of conditions. |

**Complete General Syntax — `whereIn()` / `whereNotIn()`:**

```php
$users = DB::table('users')->whereIn('id', [1, 2, 3])->get();
$users = DB::table('users')->whereNotIn('id', [1, 2, 3])->get();
```

**Complete General Syntax — `whereBetween()` / `whereNotBetween()`:**

```php
$users = DB::table('users')->whereBetween('votes', [1, 100])->get();
$users = DB::table('users')->whereNotBetween('votes', [1, 100])->get();
```

**Complete General Syntax — `whereNull()` / `whereNotNull()`:**

```php
$users = DB::table('users')->whereNull('updated_at')->get();
$users = DB::table('users')->whereNotNull('updated_at')->get();
```

**Complete General Syntax — Logical Grouping:**

```php
// WHERE (votes > 100 OR name = 'John') AND status = 'active'
$users = DB::table('users')
    ->where('status', 'active')
    ->where(function ($query) {
        $query->where('votes', '>', 100)
              ->orWhere('name', 'John');
    })
    ->get();
```

**Complete General Syntax — Date Filtering:**

```php
$users = DB::table('users')->whereDate('created_at', '2025-01-15')->get();
$users = DB::table('users')->whereMonth('created_at', 1)->get();
$users = DB::table('users')->whereYear('created_at', 2025)->get();
```

**Complete General Syntax — `whereColumn()`:**

```php
$users = DB::table('users')->whereColumn('first_name', 'last_name')->get();
$users = DB::table('users')->whereColumn('updated_at', '>', 'created_at')->get();
```

**Syntax Rules:**

- `where()` with two arguments checks for equality; with three arguments, the second is the operator.
- Supported operators: `=`, `<`, `>`, `<=`, `>=`, `<>`, `!=`, `like`, `not like`, `between`, `not between`, `in`, `not in`, `null`, `not null`.
- `orWhere()` accepts the same arguments as `where()`.
- Closures passed to `where()` or `orWhere()` create grouped conditions (parentheses in SQL).
- `whereIn()` accepts an array of values; `whereBetween()` accepts a two-element array.
- `whereColumn()` compares two columns rather than a column and a value.
- Multiple `where()` calls are joined with AND by default.

**Constraints and Limitations:**

- **`where()` without an operator:** When called with two arguments, the operator is assumed to be `=`.
- **`orWhere()` precedence:** `orWhere()` has lower precedence than `where()`. Use closures to control grouping explicitly.
- **`whereIn()` array size:** Very large arrays in `whereIn()` can exceed database parameter limits. Use `whereIntegerInRaw()` for integer arrays to reduce memory usage.
- **`whereDate()` performance:** `whereDate()` may prevent index usage on some databases because it applies a function to the column. Use range conditions with `where()` for better performance.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Where Clauses**

```php
<?php

use Illuminate\Support\Facades\DB;

// WHERE votes > 100
$users = DB::table('users')->where('votes', '>', 100)->get();

// WHERE name = 'John'
$users = DB::table('users')->where('name', 'John')->get();

// WHERE name LIKE 'T%'
$users = DB::table('users')->where('name', 'like', 'T%')->get();

// WHERE email IS NOT NULL AND votes > 50
$users = DB::table('users')
    ->whereNotNull('email')
    ->where('votes', '>', 50)
    ->get();
```

**Expected Output:** Each query returns users matching the specified conditions.

**Why:** `where()` is the fundamental filtering method, supporting all SQL comparison operators.

---

**Example 2: Set and Range Filtering**

```php
<?php

use Illuminate\Support\Facades\DB;

// WHERE id IN (1, 2, 3)
$users = DB::table('users')->whereIn('id', [1, 2, 3])->get();

// WHERE id NOT IN (1, 2, 3)
$users = DB::table('users')->whereNotIn('id', [1, 2, 3])->get();

// WHERE votes BETWEEN 1 AND 100
$users = DB::table('users')->whereBetween('votes', [1, 100])->get();

// WHERE votes NOT BETWEEN 1 AND 100
$users = DB::table('users')->whereNotBetween('votes', [1, 100])->get();

// WHERE created_at IS NULL
$users = DB::table('users')->whereNull('created_at')->get();
```

**Expected Output:** Each query filters based on set membership, range, or null checks.

**Why:** These methods provide expressive shortcuts for common SQL conditions that would otherwise require raw expressions.

---

**Example 3: Logical Grouping with Closures**

```php
<?php

use Illuminate\Support\Facades\DB;

// WHERE (votes > 100 OR name = 'John') AND status = 'active'
$users = DB::table('users')
    ->where('status', 'active')
    ->where(function ($query) {
        $query->where('votes', '>', 100)
              ->orWhere('name', 'John');
    })
    ->get();

// WHERE (age > 18 AND age < 65) OR (is_admin = 1)
$users = DB::table('users')
    ->where(function ($query) {
        $query->where('age', '>', 18)
              ->where('age', '<', 65);
    })
    ->orWhere('is_admin', 1)
    ->get();
```

**Expected Output:** The closure groups conditions in parentheses, ensuring the correct logical precedence.

**Why:** Without closures, `orWhere()` would have lower precedence than `where()`, potentially returning incorrect results. Closures mirror SQL parentheses for precise logic control.

---

**Example 4: Date-Based Filtering**

```php
<?php

use Illuminate\Support\Facades\DB;

// WHERE DATE(created_at) = '2025-01-15'
$users = DB::table('users')->whereDate('created_at', '2025-01-15')->get();

// WHERE MONTH(created_at) = 1
$users = DB::table('users')->whereMonth('created_at', 1)->get();

// WHERE YEAR(created_at) = 2025
$users = DB::table('users')->whereYear('created_at', 2025)->get();

// WHERE TIME(created_at) = '14:30:00'
$users = DB::table('users')->whereTime('created_at', '14:30:00')->get();
```

**Expected Output:** Each query filters based on the specified date/time component.

**Why:** These methods provide a clean interface for date/time component filtering, generating the appropriate database-specific function calls.

### Real-World Cases

- **E-commerce Filtering:** `whereBetween('price', [$min, $max])` for price range filters.
- **Admin User Search:** `whereIn('role', ['admin', 'moderator'])` for filtering by multiple roles.
- **Active Record Checks:** `whereNull('deleted_at')` for excluding soft-deleted records.
- **Date Reports:** `whereMonth('created_at', $month)` for monthly reporting.
- **Complex Permissions:** Logical grouping with closures for combining role and permission conditions.

### References

- Laravel Query Builder: Where Clauses — https://laravel.com/docs/12.x/queries#where-clauses
- Laravel Query Builder: Or Where Clauses — https://laravel.com/docs/12.x/queries#or-where-clauses
- Laravel Query Builder: Additional Where Clauses — https://laravel.com/docs/12.x/queries#additional-where-clauses
- Laravel Query Builder: Logical Grouping — https://laravel.com/docs/12.x/queries#logical-grouping
- Laravel Query Builder: Date Clauses — https://laravel.com/docs/12.x/queries#where-date-clauses

---

## 5. Sorting & Grouping

### Definitions

**Core Definition:** Sorting and grouping in the Query Builder are the methods that control the order of results (`orderBy`, `latest`, `inRandomOrder`), group rows by one or more columns (`groupBy`), and filter groups based on aggregate conditions (`having`).

**Technical Definition:** The `orderBy()` method adds an ORDER BY clause with a column and direction (`asc` or `desc`). The `latest()` and `oldest()` methods are shortcuts for `orderBy('created_at', 'desc')` and `orderBy('created_at', 'asc')` respectively. The `inRandomOrder()` method adds a database-specific random ordering function (`RAND()` in MySQL, `RANDOM()` in PostgreSQL). The `groupBy()` method adds a GROUP BY clause; multiple columns can be passed for composite grouping. The `having()` method adds a HAVING clause for filtering grouped results, accepting the same arguments as `where()`. The `reorder()` method removes existing ORDER BY clauses.

**Beginner-Friendly Explanation:** `orderBy()` sorts your results — alphabetically, by date, or by any column. `latest()` is a shortcut for "newest first," which is the most common sort order. `inRandomOrder()` shuffles the results. `groupBy()` collapses rows with the same values into groups, which is useful for counting or summarising. `having()` is like `where()` but for groups — it filters the grouped results after aggregation.

### Purposes

- To sort query results by one or more columns using `orderBy()`.
- To sort by the most recent records using `latest()` or `oldest()`.
- To randomise result order using `inRandomOrder()`.
- To group rows by one or more columns using `groupBy()`.
- To filter grouped results based on aggregate conditions using `having()`.
- To remove existing orderings using `reorder()`.

### Syntax Rules and Structure

**Complete General Syntax — `orderBy()`:**

```php
$users = DB::table('users')->orderBy('name', 'desc')->get();
$users = DB::table('users')->orderBy('name')->orderBy('email')->get();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `orderBy('col', 'asc\|desc')` | Sorts by the column in the given direction. |
| `latest('col')` | Sorts by `col` descending (defaults to `created_at`). |
| `oldest('col')` | Sorts by `col` ascending (defaults to `created_at`). |
| `inRandomOrder()` | Sorts randomly. |

**Complete General Syntax — `groupBy()` and `having()`:**

```php
$users = DB::table('users')
    ->select('account_id', DB::raw('COUNT(*) as total'))
    ->groupBy('account_id')
    ->having('total', '>', 100)
    ->get();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `groupBy('col')` | Groups rows by the column. |
| `having('col', 'op', 'val')` | Filters groups by the aggregate condition. |
| `havingBetween('col', [min, max])` | Filters groups within a range. |

**Complete General Syntax — `reorder()`:**

```php
$query = DB::table('users')->orderBy('name');
$unordered = $query->reorder()->get();
$orderedByEmail = $query->reorder('email', 'desc')->get();
```

**Syntax Rules:**

- `orderBy()` accepts a column name and optional direction (`asc` or `desc`); defaults to `asc`.
- `latest()` and `oldest()` default to `created_at` but accept a custom column.
- `inRandomOrder()` generates `ORDER BY RAND()` (MySQL) or `ORDER BY RANDOM()` (PostgreSQL).
- `groupBy()` accepts multiple column names for composite grouping.
- `having()` accepts the same arguments as `where()` but operates on grouped results.
- `havingBetween()` filters groups within a given range.
- `reorder()` removes all existing ORDER BY clauses; optionally applies a new ordering.

**Constraints and Limitations:**

- **`inRandomOrder()` performance:** Random ordering requires a full table scan and sort. For large tables, this can be slow. Consider alternative approaches for large datasets.
- **`having()` without `groupBy()`:** `having()` is typically used with `groupBy()`. Using it without grouping may produce unexpected results depending on the database.
- **Column aliases in `having()`:** Some databases allow column aliases in `having()` (e.g., `having('total', '>', 100)`), but this is not universally supported. Use `havingRaw()` for complex cases.
- **`reorder()` scope:** `reorder()` removes orderings from the current builder instance only; it does not affect previously executed queries.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Sorting**

```php
<?php

use Illuminate\Support\Facades\DB;

// Sort by name ascending
$users = DB::table('users')->orderBy('name')->get();

// Sort by name descending
$users = DB::table('users')->orderBy('name', 'desc')->get();

// Sort by multiple columns
$users = DB::table('users')
    ->orderBy('last_name', 'asc')
    ->orderBy('first_name', 'asc')
    ->get();

// Latest users first
$users = DB::table('users')->latest()->get();

// Oldest users first
$users = DB::table('users')->oldest()->get();

// Random order
$randomUser = DB::table('users')->inRandomOrder()->first();
```

**Expected Output:** Each query returns users in the specified order.

**Why:** `orderBy()` provides fine-grained control; `latest()`, `oldest()`, and `inRandomOrder()` are convenient shortcuts for common sorting patterns.

---

**Example 2: Grouping and Having**

```php
<?php

use Illuminate\Support\Facades\DB;

// Count users per account, filtering accounts with more than 100 users
$accounts = DB::table('users')
    ->select('account_id', DB::raw('COUNT(*) as user_count'))
    ->groupBy('account_id')
    ->having('user_count', '>', 100)
    ->get();

// Group by multiple columns
$orders = DB::table('orders')
    ->select('customer_id', 'status', DB::raw('SUM(total) as total_revenue'))
    ->groupBy('customer_id', 'status')
    ->having('total_revenue', '>', 1000)
    ->get();

// HavingBetween
$report = DB::table('orders')
    ->selectRaw('COUNT(id) as number_of_orders, customer_id')
    ->groupBy('customer_id')
    ->havingBetween('number_of_orders', [5, 15])
    ->get();
```

**Expected Output:** `groupBy()` collapses rows with matching column values; `having()` filters the grouped results.

**Why:** `groupBy()` and `having()` work together to produce aggregated summaries — group by one or more columns, then filter the groups based on aggregate conditions.

---

**Example 3: Using `reorder()`**

```php
<?php

use Illuminate\Support\Facades\DB;

$query = DB::table('users')->orderBy('name');

// Remove the default ordering
$unorderedUsers = $query->reorder()->get();

// Remove existing ordering and apply a new one
$usersByEmail = $query->reorder('email', 'desc')->get();
```

**Expected Output:** The `reorder()` method removes the `ORDER BY name` clause; `reorder('email', 'desc')` applies a new ordering by email.

**Why:** `reorder()` is useful when building complex queries where the ordering needs to be changed or removed after it has been initially set.

### Real-World Cases

- **Blog Post Listings:** `latest()` for showing newest posts first.
- **Product Catalogues:** `orderBy('price', 'asc')` for sorting by price.
- **Analytics Reports:** `groupBy('date')->having('total', '>', 100)` for daily aggregates exceeding a threshold.
- **Featured Content:** `inRandomOrder()` for rotating featured items.
- **Search Results:** `orderBy('relevance', 'desc')->orderBy('created_at', 'desc')` for multi-column sorting.

### References

- Laravel Query Builder: Ordering — https://laravel.com/docs/12.x/queries#ordering
- Laravel Query Builder: Grouping — https://laravel.com/docs/12.x/queries#grouping
- Laravel Query Builder: The Having Method — https://laravel.com/docs/12.x/queries#grouping
- Laravel Query Builder: In Random Order — https://laravel.com/docs/12.x/queries#ordering

---

## 6. Aggregates

### Definitions

**Core Definition:** Aggregate methods in the Query Builder perform calculations on sets of rows and return a single scalar result, including counting rows, finding maximum or minimum values, calculating averages, and summing numeric columns.

**Technical Definition:** The `count()` method executes a `SELECT COUNT(*)` query and returns the number of rows matching the current conditions. The `max()` and `min()` methods return the maximum and minimum values of a specified column. The `avg()` method returns the average value of a numeric column. The `sum()` method returns the total of a numeric column. The `exists()` and `doesntExist()` methods return boolean values indicating whether any rows match the conditions. All aggregate methods respect the current WHERE clauses and return a single scalar value (integer or float).

**Beginner-Friendly Explanation:** Aggregates are for answering questions like "how many?", "what's the highest?", "what's the total?" or "what's the average?" They collapse an entire result set into a single number. `count()` tells you how many rows match. `max()` and `min()` find the highest and lowest values. `sum()` adds everything up. `avg()` calculates the average. These are essential for dashboards, reports, and any feature that needs summary statistics.

### Purposes

- To count the number of rows matching a query using `count()`.
- To find the maximum value in a column using `max()`.
- To find the minimum value in a column using `min()`.
- To calculate the average of a numeric column using `avg()`.
- To calculate the total sum of a numeric column using `sum()`.
- To check if any rows exist matching a condition using `exists()` or `doesntExist()`.
- To build summary statistics for dashboards and reports.

### Syntax Rules and Structure

**Complete General Syntax — `count()`:**

```php
$count = DB::table('users')->count();
$count = DB::table('users')->where('active', 1)->count();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `->count()` | Returns the number of rows matching the query. |
| `->exists()` | Returns `true` if any rows match. |
| `->doesntExist()` | Returns `true` if no rows match. |

**Complete General Syntax — `max()` / `min()`:**

```php
$max = DB::table('orders')->max('total');
$min = DB::table('orders')->min('total');
```

**Complete General Syntax — `avg()` / `sum()`:**

```php
$avg = DB::table('orders')->avg('total');
$sum = DB::table('orders')->sum('total');
```

**Complete General Syntax — Aggregates with `groupBy()`:**

```php
$orders = DB::table('orders')
    ->select('customer_id', DB::raw('SUM(total) as total_spent'))
    ->groupBy('customer_id')
    ->get();
```

**Syntax Rules:**

- `count()`, `max()`, `min()`, `avg()`, and `sum()` are terminal methods — they execute the query and return a scalar value.
- Aggregates respect all previously applied WHERE conditions.
- `avg()` and `sum()` return `null` if no rows match (or `0` for `sum()` in some databases).
- `exists()` and `doesntExist()` return boolean values.
- When used with `groupBy()`, aggregates are applied per group rather than to the entire result set.

**Constraints and Limitations:**

- **`avg()` and `sum()` on non-numeric columns:** These methods expect numeric columns. Applying them to non-numeric columns may produce unexpected results or database errors.
- **`count()` performance:** `SELECT COUNT(*)` on large tables can be slow. Consider using `exists()` for boolean checks or maintaining a cached count.
- **Null handling:** `avg()` and `sum()` ignore NULL values. If all values are NULL, `avg()` returns `null` and `sum()` returns `0` in most databases.
- **Aggregates with `select()`:** When using `select()` with aggregates, you must include all non-aggregated columns in the `groupBy()` clause (SQL standard requirement).

### Multiple Annotated Complete Code Examples

**Example 1: Basic Aggregates**

```php
<?php

use Illuminate\Support\Facades\DB;

// Count all users
$totalUsers = DB::table('users')->count();
// 1,245

// Count active users only
$activeUsers = DB::table('users')->where('active', 1)->count();
// 987

// Find the highest order total
$maxOrder = DB::table('orders')->max('total');
// 4999.99

// Find the lowest order total
$minOrder = DB::table('orders')->min('total');
// 9.99

// Calculate average order value
$avgOrder = DB::table('orders')->avg('total');
// 149.50

// Calculate total revenue
$totalRevenue = DB::table('orders')->sum('total');
// 184,567.50
```

**Expected Output:** Each aggregate returns a single scalar value based on the query conditions.

**Why:** Aggregate methods execute the query and return a single value — the count, maximum, minimum, average, or sum of the matching rows.

---

**Example 2: Exists and DoesntExist**

```php
<?php

use Illuminate\Support\Facades\DB;

// Check if any user has the email 'alice@example.com'
if (DB::table('users')->where('email', 'alice@example.com')->exists()) {
    echo 'Email already registered.';
}

// Check if a table is empty
if (DB::table('users')->doesntExist()) {
    echo 'No users found.';
}
```

**Expected Output:** `exists()` returns `true` if at least one row matches; `doesntExist()` returns `true` if no rows match.

**Why:** These methods are more efficient than `count() > 0` for checking existence because they stop at the first matching row.

---

**Example 3: Aggregates with Grouping**

```php
<?php

use Illuminate\Support\Facades\DB;

// Total revenue per customer, sorted by highest revenue
$revenueByCustomer = DB::table('orders')
    ->select('customer_id', DB::raw('SUM(total) as total_spent'))
    ->groupBy('customer_id')
    ->orderBy('total_spent', 'desc')
    ->get();

// Average order value per month
$monthlyAverage = DB::table('orders')
    ->select(
        DB::raw('YEAR(created_at) as year'),
        DB::raw('MONTH(created_at) as month'),
        DB::raw('AVG(total) as avg_order_value')
    )
    ->groupBy('year', 'month')
    ->orderBy('year', 'desc')
    ->orderBy('month', 'desc')
    ->get();

// Count of orders per status
$ordersByStatus = DB::table('orders')
    ->select('status', DB::raw('COUNT(*) as count'))
    ->groupBy('status')
    ->get();
```

**Expected Output:** Each query returns grouped results with aggregate calculations for each group.

**Why:** Combining `groupBy()` with aggregate functions produces summary statistics per group — total revenue per customer, average order value per month, order count per status.

### Real-World Cases

- **Dashboard Metrics:** `count()` for total users, orders, or posts.
- **Revenue Reports:** `sum('total')` for total revenue calculations.
- **Price Ranges:** `max('price')` and `min('price')` for price filter bounds.
- **Average Ratings:** `avg('rating')` for calculating average product or service ratings.
- **Existence Checks:** `exists()` for checking if an email is already registered before inserting.
- **Grouped Analytics:** `groupBy('country')->count()` for geographic distribution reports.

### References

- Laravel Query Builder: Aggregates — https://laravel.com/docs/12.x/queries#aggregates
- Laravel Query Builder: Determining If Records Exist — https://laravel.com/docs/12.x/queries#determining-if-records-exist
- Laravel Query Builder: Select Statements with Aggregates — https://laravel.com/docs/12.x/queries#select-statements
- Laravel Daily: GroupBy Aggregation Examples — https://laraveldaily.com

---

## References

- Laravel Database: Query Builder (12.x) — https://laravel.com/docs/12.x/queries
- Laravel Database: Query Builder (11.x) — https://laravel.com/docs/11.x/queries
- Laravel Database: Query Builder Source (GitHub) — https://raw.githubusercontent.com/laravel/docs/11.x/queries.md
- Laravel Query Builder: Select Statements — https://laravel.com/docs/12.x/queries#select-statements
- Laravel Query Builder: Where Clauses — https://laravel.com/docs/12.x/queries#where-clauses
- Laravel Query Builder: Additional Where Clauses — https://laravel.com/docs/12.x/queries#additional-where-clauses
- Laravel Query Builder: Logical Grouping — https://laravel.com/docs/12.x/queries#logical-grouping
- Laravel Query Builder: Ordering, Grouping, Limit & Offset — https://laravel.com/docs/12.x/queries#ordering-grouping-limit-and-offset
- Laravel Query Builder: Insert Statements — https://laravel.com/docs/12.x/queries#insert-statements
- Laravel Query Builder: Upserts — https://laravel.com/docs/12.x/queries#upserts
- Laravel Query Builder: Update Statements — https://laravel.com/docs/12.x/queries#update-statements
- Laravel Query Builder: Increment and Decrement — https://laravel.com/docs/12.x/queries#increment-and-decrement
- Laravel Query Builder: Delete Statements — https://laravel.com/docs/12.x/queries#delete-statements
- Laravel Query Builder: Aggregates — https://laravel.com/docs/12.x/queries#aggregates
- Laravel Query Builder & Eloquent ORM (Webkul) — https://webkul.com/blog/laravel-query-builder-eloquent-orm/
- Laravel Upsert Package (Packagist) — https://packagist.org/packages/iamirnet/laravel-upsert
- Laravel 8 Upsert Support (GitHub PR #34698) — https://github.com/laravel/framework/pull/34698
- Laravel API: Query Builder — https://api.laravel.com/docs/12.x/Illuminate/Database/Query/Builder.html