# CRUD with PHP — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

CRUD is an acronym for Create, Read, Update, and Delete — the four fundamental operations for persistent data storage in database-driven applications. In PHP, these operations are implemented through PDO (PHP Data Objects), which provides a uniform, secure interface for executing SQL statements against MySQL, PostgreSQL, SQLite, and other database systems.

**Technical Definition**

CRUD operations map directly to SQL statements: Create corresponds to `INSERT`, Read to `SELECT`, Update to `UPDATE`, and Delete to `DELETE`. In PHP with PDO, each operation is executed via `PDO::prepare()` and `PDOStatement::execute()` (for parameterized queries) or `PDO::query()` / `PDO::exec()` (for queries without user input). Parameterized queries bind user-supplied values as data, never as SQL syntax, preventing SQL injection.

**Beginner-Friendly Explanation**

Think of a database as a filing cabinet. **Create** is adding a new file. **Read** is opening a file and reading it. **Update** is editing the contents of a file. **Delete** is shredding a file. CRUD is simply the vocabulary for these four everyday actions. PHP's PDO is the secure, standardized way to tell the filing cabinet what to do.

### Key Characteristics

- **SQL-to-PHP Mapping:** Each CRUD operation maps to a specific SQL statement type (`INSERT`, `SELECT`, `UPDATE`, `DELETE`).
- **Parameterized by Default:** All user-supplied values should be passed via prepared statement placeholders, never concatenated into SQL strings.
- **Return Value Semantics:** `INSERT` returns the last insert ID via `PDO::lastInsertId()`; `UPDATE` and `DELETE` return the number of affected rows via `PDOStatement::rowCount()`; `SELECT` returns a result set fetched via `fetch()` or `fetchAll()`.
- **Error Handling Dependency:** Reliable CRUD requires `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION` so that failures throw `PDOException` rather than returning `false` silently.
- **Idempotency Awareness:** `UPDATE` and `DELETE` are idempotent (running them twice produces the same state); `INSERT` is not.

### Prerequisites

- **PHP 8.1+** with the PDO extension and a driver extension (`pdo_mysql`, `pdo_pgsql`, `pdo_sqlite`) enabled.
- **A running database** (MySQL 8.0+, MariaDB 10.6+, PostgreSQL 14+, or SQLite 3) with a table prepared for the examples.
- **`PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`** set at connection time.
- **Understanding of SQL data types** (INTEGER, VARCHAR, TEXT, DECIMAL, DATETIME) and column constraints (PRIMARY KEY, NOT NULL, UNIQUE).
- **Composer** (optional) for installing testing or seeding libraries.

### Related Programming Areas

- **Secure Query Execution & Prepared Statements:** CRUD operations are the primary consumers of PDO's prepared statement API.
- **Data Integrity & Transactions:** Multi-step CRUD operations (e.g., order + line items) require transactions for atomicity.
- **Advanced Query Patterns:** Pagination, eager loading, and seeding all build on the CRUD primitives.
- **REST API Design:** CRUD maps to HTTP methods: `POST` (Create), `GET` (Read), `PUT`/`PATCH` (Update), `DELETE` (Delete).
- **Object-Relational Mapping (ORM):** ORMs like Doctrine and Eloquent generate CRUD SQL from object operations.

### Core Concepts / Features

1. **Create Records** — Inserting new rows with `INSERT` and retrieving the generated ID.
2. **Read Records** — Querying rows with `SELECT`, `fetch()`, and `fetchAll()`.
3. **Update Records** — Modifying existing rows with `UPDATE` and checking affected rows.
4. **Delete Records** — Removing rows with `DELETE` and checking affected rows.
5. **Parameterized Queries** — Binding user input safely via named and positional placeholders.

---

## Core Concept 1: Create Records

### Definitions

**Core Definition**

Create is the CRUD operation that inserts a new row into a database table using the SQL `INSERT` statement.

**Technical Definition**

`INSERT INTO table (column1, column2, ...) VALUES (:param1, :param2, ...)` is executed via `PDO::prepare()` and `PDOStatement::execute()`. The auto-generated primary key is retrieved via `PDO::lastInsertId()`. The number of rows inserted is always 1 for a single-row `INSERT` (or N for a multi-row `INSERT`).

**Beginner-Friendly Explanation**

Creating a record is like adding a new entry to a notebook. You decide which fields to fill in, write the values in the correct columns, and the database assigns a unique ID so you can find that entry later.

### Purposes

- To persist new data submitted by a user through a form, API, or CLI command.
- To capture the auto-generated primary key for use in subsequent related inserts.
- To enforce NOT NULL, UNIQUE, and FOREIGN KEY constraints at the moment of creation.
- To support single-row and multi-row insertion patterns.
- To generate audit trails by recording creation timestamps.

### Syntax Rules and Structure

**Complete General Syntax: Single-Row INSERT**

```php
$stmt = $pdo->prepare(
    'INSERT INTO table_name (column1, column2, column3)
     VALUES (:param1, :param2, :param3)'
);
$stmt->execute([
    ':param1' => $value1,
    ':param2' => $value2,
    ':param3' => $value3,
]);
$newId = $pdo->lastInsertId();
```

**Component Breakdown:**

- `INSERT INTO table_name (...)` — Specifies the target table and the columns to populate. Columns not listed receive their default values.
- `VALUES (:param1, :param2, ...)` — Named placeholders that correspond to the values passed to `execute()`. Positional `?` placeholders are also supported.
- `$stmt->execute([...])` — Binds the values and executes the statement. The array keys match the placeholder names.
- `$pdo->lastInsertId()` — Returns the auto-incremented ID generated by the `INSERT`. For PostgreSQL, use `lastInsertId('sequence_name')` or a `RETURNING` clause.

**Complete General Syntax: Multi-Row INSERT**

```php
$stmt = $pdo->prepare(
    'INSERT INTO table_name (column1, column2) VALUES (?, ?), (?, ?), (?, ?)'
);
$stmt->execute([$a1, $b1, $a2, $b2, $a3, $b3]);
```

**Syntax Rules:**

- The number of placeholders in `VALUES` must exactly match the number of values passed to `execute()`.
- `lastInsertId()` must be called **after** `execute()`, on the same PDO connection.
- For MySQL, `lastInsertId()` returns the ID of the **first** row in a multi-row insert. To get all IDs, query `LAST_INSERT_ID()` or use `INSERT ... RETURNING` (MySQL 8.0.19+).
- The `exec()` method can be used for `INSERT` statements without parameters, but it is not recommended when user input is involved.

**Constraints and Limitations:**

- **NOT NULL violations:** Attempting to insert `NULL` into a NOT NULL column throws a `PDOException` with SQLSTATE `23000`.
- **UNIQUE violations:** Inserting a duplicate value into a UNIQUE or PRIMARY KEY column throws a `PDOException`.
- **FOREIGN KEY violations:** Inserting a value that does not exist in a referenced table throws a `PDOException`.
- **`lastInsertId()` portability:** The behaviour of `lastInsertId()` varies by driver. For PostgreSQL, it requires a sequence name argument; for SQLite, it works without arguments.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Basic Single-Row Insert with Named Placeholders**

```php
<?php
// Step 1: Connect with exception error mode
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 2: Prepare the INSERT statement
$stmt = $pdo->prepare(
    'INSERT INTO products (name, price, stock, created_at)
     VALUES (:name, :price, :stock, NOW())'
);

// Step 3: Execute with bound values
$stmt->execute([
    ':name'  => 'Wireless Keyboard',
    ':price' => 49.99,
    ':stock' => 120,
]);

// Step 4: Retrieve the auto-generated ID
$newProductId = (int) $pdo->lastInsertId();

echo "Product created with ID: $newProductId\n";
```

**Expected Output:**

```
Product created with ID: 1
```

**Why:** The `INSERT` statement is prepared with named placeholders. The `execute()` call binds the values, and the database assigns an auto-incremented ID. `lastInsertId()` retrieves that ID immediately after execution.

**Example 2: Multi-Row Insert with Positional Placeholders**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Prepare a multi-row INSERT
$stmt = $pdo->prepare(
    'INSERT INTO categories (name, description) VALUES (?, ?), (?, ?), (?, ?)'
);

// Step 2: Execute with all six values in order
$stmt->execute([
    'Electronics', 'Devices and accessories',
    'Clothing', 'Apparel and fashion',
    'Books', 'Printed and digital media',
]);

echo "Inserted " . $stmt->rowCount() . " categories.\n";
```

**Expected Output:**

```
Inserted 3 categories.
```

**Why:** A single `INSERT` statement with three `VALUES` tuples inserts three rows. `rowCount()` returns the number of affected rows (3). Positional placeholders are supplied in order.

**Example 3: Inserting a Record with a Foreign Key**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Insert a parent record (category)
$pdo->prepare('INSERT INTO categories (name) VALUES (:name)')
    ->execute([':name' => 'Electronics']);
$categoryId = (int) $pdo->lastInsertId();

// Step 2: Insert a child record (product) referencing the parent
$stmt = $pdo->prepare(
    'INSERT INTO products (category_id, name, price)
     VALUES (:category_id, :name, :price)'
);
$stmt->execute([
    ':category_id' => $categoryId,
    ':name'        => 'USB-C Hub',
    ':price'       => 34.99,
]);

$productId = (int) $pdo->lastInsertId();

echo "Category #$categoryId created. Product #$productId created in that category.\n";
```

**Expected Output:**

```
Category #1 created. Product #1 created in that category.
```

**Why:** The category is inserted first, and its auto-generated ID is captured. That ID is then used as the `category_id` foreign key when inserting the product. This enforces referential integrity.

### Real-World Cases

**Case 1: User Registration Form**

A registration form collects username, email, and password. The `INSERT` statement stores the hashed password and returns the new user's ID, which is then used to create a session or send a welcome email.

**Case 2: E-Commerce Order Placement**

An order is created with `INSERT INTO orders`, and the returned `order_id` is used to insert the order's line items in a multi-row `INSERT`. Both operations are wrapped in a transaction.

**Case 3: Audit Logging**

Every significant action (login, edit, delete) is recorded via an `INSERT` into an `audit_log` table, capturing the user ID, action type, and timestamp.

---

## Core Concept 2: Read Records

### Definitions

**Core Definition**

Read is the CRUD operation that retrieves data from a database table using the SQL `SELECT` statement.

**Technical Definition**

`SELECT column_list FROM table WHERE condition ORDER BY column LIMIT n` is executed via `PDO::query()` (for parameterless queries) or `PDO::prepare()`/`execute()` (for parameterized queries). Results are retrieved via `PDOStatement::fetch()` (one row) or `PDOStatement::fetchAll()` (all remaining rows). The fetch mode controls the shape of the returned data.

**Beginner-Friendly Explanation**

Reading a record is like searching a catalog. You specify what you are looking for (the `WHERE` clause), and the database returns the matching entries. You can ask for one entry at a time or all matching entries at once.

### Purposes

- To display data to users in HTML tables, lists, or detail views.
- To retrieve a single record by its primary key for editing or deletion.
- To count, aggregate, or filter data for reports and dashboards.
- To populate dropdowns, checkboxes, and other form elements.
- To check for the existence of a record before performing a mutation.

### Syntax Rules and Structure

**Complete General Syntax: Parameterized SELECT**

```php
$stmt = $pdo->prepare(
    'SELECT column1, column2, column3
     FROM table_name
     WHERE condition_column = :param
     ORDER BY column1 ASC
     LIMIT :limit'
);
$stmt->bindValue(':param', $value);
$stmt->bindValue(':limit', $limit, PDO::PARAM_INT);
$stmt->execute();

// Fetch one row
$row = $stmt->fetch(PDO::FETCH_ASSOC);

// Or fetch all rows
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

**Component Breakdown:**

- `SELECT column1, column2, ...` — Lists the columns to retrieve. Use explicit column names instead of `*` to reduce memory and network overhead.
- `FROM table_name` — The source table.
- `WHERE condition_column = :param` — Filters rows. Use placeholders for all user-supplied values.
- `ORDER BY column1 ASC` — Sorts the result set. Sorting on an indexed column is faster.
- `LIMIT :limit` — Restricts the number of rows. When bound with `PDO::PARAM_INT`, the value is treated as an integer.
- `fetch(PDO::FETCH_ASSOC)` — Returns one row as an associative array, or `false` if no more rows.
- `fetchAll(PDO::FETCH_ASSOC)` — Returns all remaining rows as an array of associative arrays.

**Syntax Rules:**

- `LIMIT` and `OFFSET` parameters must be bound as `PDO::PARAM_INT` when using native prepared statements.
- When using `query()` without parameters, the result is a `PDOStatement` that must be iterated or fetched.
- `fetch()` advances the internal row pointer; subsequent calls return the next row.
- `fetchAll()` returns all remaining rows and leaves the pointer at the end.

**Constraints and Limitations:**

- **`SELECT *` performance:** Retrieving all columns when only a few are needed wastes memory and bandwidth, especially with large tables.
- **Unbounded `fetchAll()`:** Loading millions of rows into PHP memory causes exhaustion. Use `fetch()` in a loop or unbuffered queries for large result sets.
- **N+1 queries:** Fetching a list of parent records and then querying children inside a loop causes query explosion. Use JOINs or batched `IN` queries.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Fetching a Single Record by ID**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// Step 1: Prepare a SELECT by primary key
$stmt = $pdo->prepare('SELECT id, name, price, stock FROM products WHERE id = :id');

// Step 2: Bind the ID as an integer
$stmt->bindValue(':id', 42, PDO::PARAM_INT);

// Step 3: Execute and fetch one row
$stmt->execute();
$product = $stmt->fetch();

if ($product === false) {
    echo "Product not found.\n";
} else {
    echo "Product: {$product['name']} — \${$product['price']} ({$product['stock']} in stock)\n";
}
```

**Expected Output:**

```
Product: USB-C Hub — $34.99 (85 in stock)
```

**Why:** `fetch()` returns one row as an associative array (due to the default fetch mode set at connection time). If no row matches the `WHERE` clause, `fetch()` returns `false`, which is handled explicitly.

**Example 2: Fetching All Rows with a Filter**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// Step 1: Prepare a filtered SELECT
$stmt = $pdo->prepare(
    'SELECT id, name, price FROM products
     WHERE category_id = :category_id AND price <= :max_price
     ORDER BY price ASC
     LIMIT 20'
);

// Step 2: Bind values with appropriate types
$stmt->bindValue(':category_id', 3, PDO::PARAM_INT);
$stmt->bindValue(':max_price', 100.00);  // PDO::PARAM_STR is default

// Step 3: Execute and fetch all rows
$stmt->execute();
$products = $stmt->fetchAll();

echo "Found " . count($products) . " products:\n";
foreach ($products as $product) {
    echo "- {$product['name']}: \${$product['price']}\n";
}
```

**Expected Output:**

```
Found 3 products:
- USB-C Cable: $12.99
- USB-C Hub: $34.99
- Wireless Keyboard: $49.99
```

**Why:** The `WHERE` clause filters by category and maximum price. `ORDER BY price ASC` sorts the results. `fetchAll()` returns all matching rows in a single call. The loop then iterates over the array.

**Example 3: Fetching a Single Scalar Value with `fetchColumn()`**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Prepare a COUNT query
$stmt = $pdo->prepare('SELECT COUNT(*) FROM products WHERE stock > 0');

// Step 2: Execute and retrieve the count directly
$stmt->execute();
$inStockCount = (int) $stmt->fetchColumn();

echo "Products currently in stock: $inStockCount\n";
```

**Expected Output:**

```
Products currently in stock: 47
```

**Why:** `fetchColumn()` returns the first column of the next row. For a `COUNT(*)` query, this is the count value. This is more efficient than `fetch()` when only a single scalar value is needed.

### Real-World Cases

**Case 1: Product Listing Page**

An e-commerce product listing page uses a parameterized `SELECT` with `WHERE category_id = :id` and `LIMIT`/`OFFSET` for pagination. `fetchAll()` populates the product grid.

**Case 2: User Profile Edit Form**

An edit form loads a single user record by ID using `fetch()`. The values are pre-filled into the form fields. If the record does not exist, the user is redirected to a 404 page.

**Case 3: Dashboard Statistics**

A dashboard displays aggregate metrics: total orders (`COUNT(*)`), total revenue (`SUM(total)`), and average order value (`AVG(total)`). Each metric uses `fetchColumn()`.

---

## Core Concept 3: Update Records

### Definitions

**Core Definition**

Update is the CRUD operation that modifies existing rows in a database table using the SQL `UPDATE` statement.

**Technical Definition**

`UPDATE table SET column1 = :param1, column2 = :param2 WHERE condition = :condition` is executed via `PDO::prepare()` and `PDOStatement::execute()`. `PDOStatement::rowCount()` returns the number of rows affected. If the `WHERE` clause matches no rows, `rowCount()` returns 0; if it matches multiple rows, all are updated unless `LIMIT` is specified.

**Beginner-Friendly Explanation**

Updating a record is like editing an entry in a notebook. You find the entry (the `WHERE` clause), erase the old values, and write the new ones. The database tells you how many entries you changed.

### Purposes

- To save changes made by users in edit forms.
- To update inventory levels after a sale or restock.
- To change status flags (e.g., `active`, `pending`, `archived`).
- To record modification timestamps (`updated_at`).
- To perform bulk updates across multiple rows matching a condition.

### Syntax Rules and Structure

**Complete General Syntax: Parameterized UPDATE**

```php
$stmt = $pdo->prepare(
    'UPDATE table_name
     SET column1 = :param1, column2 = :param2, updated_at = NOW()
     WHERE id = :id'
);
$stmt->execute([
    ':param1' => $value1,
    ':param2' => $value2,
    ':id'     => $id,
]);
$affectedRows = $stmt->rowCount();
```

**Component Breakdown:**

- `UPDATE table_name` — The target table.
- `SET column1 = :param1, ...` — The columns to modify and their new values. Named placeholders are used for values.
- `WHERE id = :id` — The condition that identifies which rows to update. **Omitting the `WHERE` clause updates every row in the table.**
- `$stmt->execute([...])` — Binds the values and executes the statement.
- `$stmt->rowCount()` — Returns the number of rows affected by the `UPDATE`.

**Syntax Rules:**

- Always include a `WHERE` clause unless you intentionally want to update all rows.
- The number of placeholders in `SET` and `WHERE` must exactly match the values passed to `execute()`.
- `rowCount()` returns the number of rows that were **actually changed**, not the number that matched the `WHERE` clause. If a row's values are identical to the new values, MySQL may report 0 affected rows (depending on `PDO::MYSQL_ATTR_FOUND_ROWS`).
- `updated_at` columns are conventionally updated with `NOW()` or `CURRENT_TIMESTAMP` within the `SET` clause.

**Constraints and Limitations:**

- **Missing `WHERE` clause:** Updates every row. This is a common and catastrophic mistake. Always verify the `WHERE` clause before execution.
- **`rowCount()` inconsistencies:** Some drivers return the number of matched rows, not changed rows. Set `PDO::MYSQL_ATTR_FOUND_ROWS => true` to make `rowCount()` return matched rows instead of changed rows.
- **Concurrency:** If two users edit the same record simultaneously, the last write wins. Optimistic locking (using a `version` column) or pessimistic locking (`SELECT ... FOR UPDATE`) is needed to prevent lost updates.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Updating a Single Record by ID**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Prepare the UPDATE statement
$stmt = $pdo->prepare(
    'UPDATE products
     SET name = :name, price = :price, stock = :stock, updated_at = NOW()
     WHERE id = :id'
);

// Step 2: Execute with bound values
$stmt->execute([
    ':name'  => 'Wireless Keyboard (Revised)',
    ':price' => 44.99,
    ':stock' => 95,
    ':id'    => 1,
]);

// Step 3: Check affected rows
$affected = $stmt->rowCount();

if ($affected > 0) {
    echo "Product #1 updated successfully ($affected row(s) affected).\n";
} else {
    echo "No product found with ID 1, or no values were changed.\n";
}
```

**Expected Output:**

```
Product #1 updated successfully (1 row(s) affected).
```

**Why:** The `UPDATE` statement modifies the product with `id = 1`. `rowCount()` returns 1 because one row was changed. If the ID did not exist, `rowCount()` would return 0.

**Example 2: Bulk Update by Category**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Prepare a bulk UPDATE
$stmt = $pdo->prepare(
    'UPDATE products SET stock = stock + :increment WHERE category_id = :category_id'
);

// Step 2: Execute with bound values
$stmt->execute([
    ':increment'   => 50,
    ':category_id' => 3,
]);

echo "Restocked " . $stmt->rowCount() . " products in category 3.\n";
```

**Expected Output:**

```
Restocked 12 products in category 3.
```

**Why:** The `WHERE category_id = :category_id` clause matches all products in category 3. The `SET stock = stock + :increment` expression increments each matching row's stock by 50. `rowCount()` returns the number of affected rows (12).

**Example 3: Safe Update with Existence Check**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$productId = 42;
$newPrice = 29.99;

// Step 1: Check if the record exists
$check = $pdo->prepare('SELECT COUNT(*) FROM products WHERE id = :id');
$check->execute([':id' => $productId]);
$exists = (int) $check->fetchColumn() > 0;

if (!$exists) {
    echo "Product #$productId does not exist. Update aborted.\n";
    exit;
}

// Step 2: Perform the update
$stmt = $pdo->prepare('UPDATE products SET price = :price WHERE id = :id');
$stmt->execute([':price' => $newPrice, ':id' => $productId]);

echo "Product #$productId price updated to \$$newPrice.\n";
```

**Expected Output:**

```
Product #42 price updated to $29.99.
```

**Why:** The existence check prevents an unnecessary `UPDATE` on a non-existent row. This pattern is useful when the application needs to distinguish between "no such record" and "record exists but values unchanged."

### Real-World Cases

**Case 1: User Profile Update**

A user edits their profile (name, email, bio). The `UPDATE` statement modifies the `users` row matching the session's user ID. An `updated_at` timestamp is recorded for audit purposes.

**Case 2: Inventory Adjustment After a Sale**

When an order is placed, the `products` table is updated to decrement stock for each purchased item. The update is part of a transaction that also inserts the order and order items.

**Case 3: Bulk Status Change**

An admin selects multiple user accounts and changes their status from "pending" to "active." A single `UPDATE ... WHERE id IN (...)` statement updates all selected rows.

---

## Core Concept 4: Delete Records

### Definitions

**Core Definition**

Delete is the CRUD operation that removes existing rows from a database table using the SQL `DELETE` statement.

**Technical Definition**

`DELETE FROM table WHERE condition = :param` is executed via `PDO::prepare()` and `PDOStatement::execute()`. `PDOStatement::rowCount()` returns the number of rows deleted. If the `WHERE` clause matches no rows, `rowCount()` returns 0. Omitting the `WHERE` clause deletes every row in the table (equivalent to `TRUNCATE` but logged row-by-row).

**Beginner-Friendly Explanation**

Deleting a record is like shredding a page from a notebook. You find the page (the `WHERE` clause) and destroy it. The database tells you how many pages were shredded.

### Purposes

- To remove records that are no longer needed (e.g., cancelled orders, expired sessions).
- To implement soft deletes by setting a `deleted_at` flag instead of removing the row.
- To clean up temporary or test data.
- To enforce referential integrity through `ON DELETE CASCADE`.
- To allow users to delete their own content (with authorization checks).

### Syntax Rules and Structure

**Complete General Syntax: Parameterized DELETE**

```php
$stmt = $pdo->prepare('DELETE FROM table_name WHERE id = :id');
$stmt->execute([':id' => $id]);
$affectedRows = $stmt->rowCount();
```

**Component Breakdown:**

- `DELETE FROM table_name` — The target table.
- `WHERE id = :id` — The condition that identifies which rows to delete. **Omitting the `WHERE` clause deletes every row.**
- `$stmt->execute([...])` — Binds the value and executes the statement.
- `$stmt->rowCount()` — Returns the number of rows deleted.

**Complete General Syntax: Soft Delete**

```php
$stmt = $pdo->prepare('UPDATE table_name SET deleted_at = NOW() WHERE id = :id');
$stmt->execute([':id' => $id]);
```

**Syntax Rules:**

- Always include a `WHERE` clause unless you intentionally want to delete all rows.
- When deleting a parent record, foreign key constraints may prevent deletion if child records exist. Use `ON DELETE CASCADE` or delete children first.
- `rowCount()` returns the number of rows deleted. It does not return the deleted data — if you need the data, fetch it before deleting.

**Constraints and Limitations:**

- **Accidental full-table deletion:** Forgetting the `WHERE` clause deletes all rows. Some frameworks (e.g., Laravel's Eloquent) guard against this by throwing an exception if no `WHERE` clause is present.
- **Foreign key restrictions:** Deleting a parent row with existing child rows throws a `PDOException` unless `ON DELETE CASCADE` is defined.
- **Soft delete vs. hard delete:** Soft deletes preserve data for auditing but require every `SELECT` query to filter out deleted rows (`WHERE deleted_at IS NULL`). Forgetting this filter leaks deleted data.
- **`TRUNCATE` vs. `DELETE`:** `TRUNCATE` is faster but cannot be rolled back in MySQL and resets auto-increment counters. `DELETE` is transactional and can be rolled back.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Deleting a Single Record by ID**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Prepare the DELETE statement
$stmt = $pdo->prepare('DELETE FROM products WHERE id = :id');

// Step 2: Execute with the target ID
$stmt->execute([':id' => 42]);

// Step 3: Check affected rows
$deleted = $stmt->rowCount();

if ($deleted > 0) {
    echo "Product #42 deleted successfully.\n";
} else {
    echo "No product found with ID 42.\n";
}
```

**Expected Output:**

```
Product #42 deleted successfully.
```

**Why:** The `DELETE` statement removes the row where `id = 42`. `rowCount()` returns 1, confirming the deletion. If no row matched, `rowCount()` would return 0.

**Example 2: Conditional Deletion with Authorization**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=blog;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$postId = 15;
$currentUserId = 7; // From session

// Step 1: Prepare a DELETE that also checks ownership
$stmt = $pdo->prepare('DELETE FROM posts WHERE id = :id AND author_id = :author_id');
$stmt->execute([
    ':id'         => $postId,
    ':author_id'  => $currentUserId,
]);

$deleted = $stmt->rowCount();

if ($deleted > 0) {
    echo "Post #$postId deleted.\n";
} else {
    echo "Post not found or you are not the author. Deletion denied.\n";
}
```

**Expected Output:**

```
Post #15 deleted.
```

**Why:** The `WHERE id = :id AND author_id = :author_id` clause ensures that only the author can delete their own post. If the post belongs to another user, `rowCount()` returns 0 and the deletion is denied — without needing a separate authorization query.

**Example 3: Soft Delete (Logical Delete)**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=blog;charset=utf8mb4', 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 1: Soft delete — mark as deleted instead of removing
$stmt = $pdo->prepare('UPDATE posts SET deleted_at = NOW() WHERE id = :id');
$stmt->execute([':id' => 15]);

echo "Post #15 soft-deleted. " . $stmt->rowCount() . " row(s) affected.\n";

// Step 2: Read only non-deleted posts
$stmt = $pdo->query('SELECT id, title FROM posts WHERE deleted_at IS NULL');
$posts = $stmt->fetchAll(PDO::FETCH_ASSOC);
echo "Active posts: " . count($posts) . "\n";
```

**Expected Output:**

```
Post #15 soft-deleted. 1 row(s) affected.
Active posts: 23
```

**Why:** Instead of deleting the row, `deleted_at` is set to the current timestamp. Subsequent `SELECT` queries filter with `WHERE deleted_at IS NULL` to exclude soft-deleted records. This preserves the data for auditing or potential restoration.

### Real-World Cases

**Case 1: User Account Deletion**

A user requests account deletion. The application performs a soft delete (setting `deleted_at`) to preserve order history and audit logs, while marking the account as inactive.

**Case 2: Shopping Cart Cleanup**

Expired shopping carts are deleted nightly by a cron job: `DELETE FROM carts WHERE updated_at < NOW() - INTERVAL 30 DAY`.

**Case 3: Cascade Delete**

Deleting a category with `ON DELETE CASCADE` automatically deletes all products in that category. Without the cascade, the deletion would fail due to foreign key constraints.

---

## Core Concept 5: Parameterized Queries

### Definitions

**Core Definition**

Parameterized queries (also called prepared statements) are SQL statements in which user-supplied values are represented by placeholders (`:name` or `?`) and transmitted to the database separately from the SQL structure.

**Technical Definition**

`PDO::prepare()` submits an SQL template to the database (or PDO's emulation layer) for parsing and compilation. Placeholders mark where values will be inserted. `PDOStatement::execute()` then transmits the values, which the database treats strictly as data — never as SQL syntax. This separation prevents SQL injection by design.

**Beginner-Friendly Explanation**

Think of a fill-in-the-blank form. The questions are printed and fixed; only the blanks are filled with your answers. A malicious user cannot change the questions by writing clever answers, because the form's structure was locked before they arrived.

### Purposes

- To prevent SQL injection by ensuring user input is never interpreted as SQL code.
- To allow the database to cache query plans for repeated executions with different values.
- To provide a uniform binding API across all supported database drivers.
- To eliminate the need for manual escaping, which is error-prone and encoding-dependent.
- To support explicit type binding (`PDO::PARAM_INT`, `PDO::PARAM_STR`, etc.) for driver consistency.

### Syntax Rules and Structure

**Named Placeholders**

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id AND status = :status');
$stmt->execute([':id' => 42, ':status' => 'active']);
```

**Positional Placeholders**

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = ? AND status = ?');
$stmt->execute([42, 'active']);
```

**Explicit Binding with `bindValue()`**

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->bindValue(':id', 42, PDO::PARAM_INT);
$stmt->execute();
```

**Component Breakdown:**

- `:name` — A named placeholder. The colon prefix is optional in `bindParam()` and `bindValue()` but required in the `execute()` array keys.
- `?` — A positional placeholder. Values are supplied in order in the `execute()` array.
- `$data_type` — The PDO type constant: `PDO::PARAM_INT`, `PDO::PARAM_STR`, `PDO::PARAM_BOOL`, `PDO::PARAM_NULL`, `PDO::PARAM_LOB`.
- `bindValue()` — Binds a value by copy. Changes to the PHP variable after binding do not affect the query.
- `bindParam()` — Binds a variable by reference. Changes to the variable after binding are reflected in the executed query.

**Syntax Rules:**

- Named and positional placeholders cannot be mixed in the same statement.
- The same named placeholder cannot be used more than once (unless emulation mode is on).
- Placeholders cannot be used for identifiers (table names, column names, `ORDER BY` direction).
- The number of bound parameters must exactly match the number of placeholders.

**Constraints and Limitations:**

- **Identifiers cannot be parameterized:** `ORDER BY :column` does not work. Use an allow-list for dynamic column names.
- **`IN()` clauses require one placeholder per value:** `WHERE id IN (:ids)` does not work. Build the placeholder list dynamically.
- **`ATTR_EMULATE_PREPARES`:** When `true` (PDO's historical default), PDO interpolates values client-side. Set to `false` for true server-side preparation and stronger security guarantees.
- **Multibyte charset edge cases:** With emulated prepares, certain multibyte encodings (e.g., GBK) historically allowed injection. Native prepares eliminate this risk.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Named vs. Positional Placeholders**

```php
<?php
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, age INTEGER)');
$pdo->exec("INSERT INTO users (name, age) VALUES ('Alice', 30), ('Bob', 25), ('Carol', 35)");

// --- Named placeholders ---
$stmt = $pdo->prepare('SELECT * FROM users WHERE age > :min_age AND name != :exclude_name');
$stmt->execute([':min_age' => 26, ':exclude_name' => 'Alice']);
$named = $stmt->fetchAll(PDO::FETCH_ASSOC);
echo "Named: " . count($named) . " result(s)\n";

// --- Positional placeholders ---
$stmt2 = $pdo->prepare('SELECT * FROM users WHERE age > ? AND name != ?');
$stmt2->execute([26, 'Alice']);
$positional = $stmt2->fetchAll(PDO::FETCH_ASSOC);
echo "Positional: " . count($positional) . " result(s)\n";
```

**Expected Output:**

```
Named: 1 result(s)
Positional: 1 result(s)
```

**Why:** Both queries produce identical results. Named placeholders are more readable when a statement has many parameters; positional placeholders are more compact.

**Example 2: `bindValue()` vs. `bindParam()`**

```php
<?php
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE products (id INTEGER PRIMARY KEY, name TEXT)');
$pdo->exec("INSERT INTO products (name) VALUES ('Widget'), ('Gadget')");

// bindValue(): value captured at binding time
$stmt = $pdo->prepare('SELECT * FROM products WHERE id = :id');
$productId = 1;
$stmt->bindValue(':id', $productId, PDO::PARAM_INT);
$productId = 2;  // Ignored
$stmt->execute();
echo "bindValue: " . $stmt->fetchColumn(1) . "\n";  // Widget

// bindParam(): variable bound by reference
$stmt2 = $pdo->prepare('SELECT * FROM products WHERE id = :id');
$productId2 = 1;
$stmt2->bindParam(':id', $productId2, PDO::PARAM_INT);
$productId2 = 2;  // Reflected
$stmt2->execute();
echo "bindParam: " . $stmt2->fetchColumn(1) . "\n";  // Gadget
```

**Expected Output:**

```
bindValue: Widget
bindParam: Gadget
```

**Why:** `bindValue()` copies the value at binding time. `bindParam()` binds the variable itself, so changes made before `execute()` are reflected in the executed query.

**Example 3: Dynamic `IN()` Clause with Placeholders**

```php
<?php
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE items (id INTEGER PRIMARY KEY, name TEXT)');
$pdo->exec("INSERT INTO items (name) VALUES ('A'), ('B'), ('C'), ('D')");

// User-supplied array of IDs (e.g., from a multi-select form)
$selectedIds = [1, 3, 4];

// Step 1: Build the placeholder list
$placeholders = implode(',', array_fill(0, count($selectedIds), '?'));

// Step 2: Prepare the query with the dynamic IN clause
$stmt = $pdo->prepare("SELECT * FROM items WHERE id IN ($placeholders)");

// Step 3: Execute with the ID values
$stmt->execute($selectedIds);

$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
echo "Selected items: " . implode(', ', array_column($rows, 'name')) . "\n";
```

**Expected Output:**

```
Selected items: A, C, D
```

**Why:** The `IN()` clause cannot accept a single placeholder for an array. The solution builds one `?` placeholder per selected ID, then supplies the values in order. This keeps the query parameterized and safe.

### Real-World Cases

**Case 1: Login Authentication**

A login form uses `SELECT id FROM users WHERE email = :email AND password_hash = :hash` with bound parameters. An attacker supplying `' OR '1'='1` as the email cannot bypass authentication because the entire string is treated as a literal email value.

**Case 2: Search with Multiple Filters**

A product search form allows filtering by category, price range, and availability. Named placeholders make the dynamic query readable and safe, even as filters are added or removed based on form input.

**Case 3: REST API Resource Retrieval**

A REST API endpoint `GET /notes/42` uses `SELECT * FROM notes WHERE id = :id AND owner_id = :owner` with bound parameters. The `owner_id` check prevents broken object-level authorization (BOLA) attacks.

---

## References

- PHP: PDO::prepare — Manual – https://www.php.net/manual/en/pdo.prepare.php
- PHP: PDOStatement::execute — Manual – https://www.php.net/manual/en/pdostatement.execute.php
- PHP: PDOStatement::fetch — Manual – https://www.php.net/manual/en/pdostatement.fetch.php
- PHP: PDOStatement::fetchAll — Manual – https://www.php.net/manual/en/pdostatement.fetchall.php
- PHP: PDOStatement::fetchColumn — Manual – https://www.php.net/manual/en/pdostatement.fetchcolumn.php
- PHP: PDOStatement::bindValue — Manual – https://www.php.net/manual/en/pdostatement.bindvalue.php
- PHP: PDOStatement::bindParam — Manual – https://www.php.net/manual/en/pdostatement.bindparam.php
- PHP: PDO::lastInsertId — Manual – https://www.php.net/manual/en/pdo.lastinsertid.php
- PHP: PDO::exec — Manual – https://www.php.net/manual/en/pdo.exec.php
- PHP: PDO::query — Manual – https://www.php.net/manual/en/pdo.query.php
- Secure PHP Development: Building a CRUD API — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/REST-API/Building-a-CRUD-API.md
- Secure PHP Development: User Management CRUD — https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Mini-Projects/User-Management-CRUD.md
- OWASP: SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- PHP Delusions: PDO — https://phpdelusions.net/pdo
- Microsoft: How to Perform Transactions — PHP Drivers for SQL Server – https://learn.microsoft.com/en-us/sql/connect/php/how-to-perform-transactions