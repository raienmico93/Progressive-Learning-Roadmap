# Robust Data Integrity & PHP Transactions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Database transactions in PHP are a mechanism for grouping multiple SQL statements into a single atomic unit of work, ensuring that either all changes are persisted or none are, thereby maintaining data integrity even in the presence of errors, concurrent access, or system failures.

**Technical Definition**

A transaction is a sequence of database operations executed as a single logical unit, governed by the ACID properties (Atomicity, Consistency, Isolation, Durability). PDO provides three methods for transaction control: `beginTransaction()` to start a transaction (disabling autocommit mode), `commit()` to persist all changes, and `rollBack()` to discard all changes. These methods interact with the underlying database driver's native transaction support, which for MySQL requires the InnoDB storage engine.

**Beginner-Friendly Explanation**

Imagine you are transferring money between two bank accounts. If the system debits your account but crashes before crediting the recipient, the money vanishes. A transaction is like a protective envelope: either both the debit and the credit happen together, or neither happens at all. PDO's transaction methods let you create that envelope around your database operations.

### Key Characteristics

- **All-or-Nothing Execution:** All statements within a transaction are treated as a single unit; partial completion is not permitted.
- **Autocommit Disabled:** `beginTransaction()` turns off PDO's default autocommit mode, so no change is persisted until `commit()` is called.
- **Exception-Driven Safety:** When `PDO::ATTR_ERRMODE` is set to `ERRMODE_EXCEPTION`, any failing statement throws a `PDOException`, which can be caught to trigger a rollback.
- **Automatic Rollback on Script End:** If a script terminates with an open transaction, PDO automatically rolls it back as a safety measure.
- **DDL Implicit Commits:** Some databases (MySQL, Oracle) automatically commit any open transaction when a DDL statement (`CREATE TABLE`, `ALTER TABLE`, `DROP`) is executed, making prior changes impossible to roll back.
- **Storage Engine Dependency:** MySQL transactions require InnoDB; MyISAM silently ignores transactions and auto-commits every statement.

### Prerequisites

- **PHP 8.2+** with PDO and a transaction-capable driver (`pdo_mysql`, `pdo_pgsql`, `pdo_sqlite`) enabled.
- **MySQL 8.0+ / MariaDB 10.6+ with InnoDB** (or PostgreSQL / SQLite) — MyISAM does not support transactions.
- **`PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`** set at connection time to ensure failing statements throw exceptions rather than returning `false` silently.
- **Understanding of SQLSTATE error codes** for detecting deadlocks (40001) and lock-wait timeouts (1205).
- **Familiarity with PDO prepared statements** — all examples assume prepared statements are used for query execution.

### Related Programming Areas

- **Concurrency Control:** Row-level locking (`SELECT ... FOR UPDATE`), deadlock detection, and isolation levels.
- **Error Handling and Resilience:** Try/catch blocks, exponential backoff, and retry wrappers for transient failures.
- **ORM and Framework Architecture:** Doctrine DBAL's `Transaction Nesting`, Laravel's `DB::transaction()`, and Symfony's `TransactionManager` are built on PDO transaction primitives.
- **Database Administration:** InnoDB configuration, lock monitoring (`SHOW ENGINE INNODB STATUS`), and isolation level tuning.
- **Distributed Systems:** Two-phase commit, saga patterns, and transactional outbox patterns for cross-service consistency.

### Core Concepts / Features

1. **ACID Properties** — Atomicity, Consistency, Isolation, and Durability within a PHP execution context.
2. **Transaction Controls** — `beginTransaction()`, `commit()`, and `rollBack()`.
3. **Nested Transactions & Savepoints** — Simulating nested transactions with `SAVEPOINT` and `ROLLBACK TO SAVEPOINT`.
4. **Resilient Error Catching** — Try/catch wrapping, deadlock detection, and automated retry with exponential backoff.

---

## Core Concept 1: ACID Properties

### Definitions

**Core Definition**

ACID is an acronym for Atomicity, Consistency, Isolation, and Durability — the four guarantees that a database transaction must provide to ensure reliable data processing.

**Technical Definition**

**Atomicity** ensures that all statements within a transaction are executed as a single indivisible unit: either all succeed or all are rolled back. **Consistency** ensures that the database transitions from one valid state to another, preserving all defined constraints and rules. **Isolation** ensures that concurrent transactions do not interfere with each other's uncommitted work. **Durability** ensures that once a transaction is committed, its effects survive system crashes, power failures, or restarts. In MySQL, these guarantees require the InnoDB storage engine, which provides full ACID support.

**Beginner-Friendly Explanation**

Think of ACID as the four promises a reliable bank makes. **Atomicity** means a transfer either completes fully or not at all. **Consistency** means the bank never lets your account balance break the rules (e.g., go negative when overdrafts are disallowed). **Isolation** means two simultaneous transfers don't see each other's half-finished work. **Durability** means once the bank says "done," the change is permanent even if the building loses power.

### Purposes

- To guarantee that multi-statement database operations either succeed completely or leave the database untouched.
- To prevent data corruption caused by partial writes during script failures or server crashes.
- To ensure that concurrent transactions do not corrupt each other's data through uncommitted changes.
- To enforce database-level constraints (foreign keys, unique indexes, CHECK constraints) consistently across a group of statements.
- To provide a foundation for application-level business rules that depend on multiple database mutations succeeding together.

### Syntax Rules and Structure

ACID properties are not configured via a single PDO method or attribute. They are inherent to the database engine's transaction implementation and are activated when a transaction is started via `PDO::beginTransaction()`.

**Complete General Syntax: Activating ACID Guarantees**

```php
// Step 1: Configure the connection with exception mode
$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

// Step 2: Begin a transaction (activates ACID guarantees for the group)
$pdo->beginTransaction();

// Step 3: Execute statements...

// Step 4a: Commit (persists all changes — Durability)
$pdo->commit();

// Step 4b: Or rollback (discards all changes — Atomicity)
$pdo->rollBack();
```

**Component Breakdown:**

- `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION` — Required for reliable ACID behaviour. Without exception mode, a failed statement returns `false`, and the script may proceed to `commit()` a partial write.
- `beginTransaction()` — Disables autocommit mode. Nothing is persisted until `commit()`.
- `commit()` — Persists all changes to disk and restores autocommit mode.
- `rollBack()` — Discards all changes since `beginTransaction()` and restores autocommit mode.

**Syntax Rules:**

- The database storage engine must support transactions. MySQL requires InnoDB; MyISAM does not support transactions and silently auto-commits every statement.
- DDL statements (`CREATE TABLE`, `ALTER TABLE`, `DROP`, `TRUNCATE`) cause an **implicit commit** in MySQL and Oracle. Any changes made before the DDL statement are automatically committed and cannot be rolled back.
- PDO only performs automatic rollback on script termination if the transaction was started via `PDO::beginTransaction()`. Manually issuing `START TRANSACTION` bypasses PDO's tracking.

**Constraints and Limitations:**

- **Storage engine dependency:** Transactions are silently ignored on MyISAM tables. Always verify that the table engine is InnoDB.
- **DDL implicit commits:** Avoid mixing schema changes with data transactions. Perform DDL outside of transactional blocks.
- **Isolation level trade-offs:** Higher isolation levels (e.g., `SERIALIZABLE`) provide stronger consistency guarantees but reduce concurrency and increase deadlock risk.
- **Durability configuration:** MySQL's `innodb_flush_log_at_trx_commit` setting controls how strictly durability is enforced. Setting it to `0` improves performance at the cost of losing recent commits on crash.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Atomic Money Transfer (Demonstrating Atomicity)**

```php
<?php
declare(strict_types=1);

$pdo = new PDO('mysql:host=127.0.0.1;dbname=bank;charset=utf8mb4', 'root', '', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

function transferFunds(PDO $pdo, int $fromId, int $toId, int $amountInCents): void
{
    // Step 1: Begin transaction — Atomicity in action
    $pdo->beginTransaction();

    try {
        // Step 2: Debit the source account
        $stmt = $pdo->prepare('UPDATE accounts SET balance = balance - :amount WHERE id = :id');
        $stmt->execute([':amount' => $amountInCents, ':id' => $fromId]);

        // Step 3: Credit the destination account
        $stmt = $pdo->prepare('UPDATE accounts SET balance = balance + :amount WHERE id = :id');
        $stmt->execute([':amount' => $amountInCents, ':id' => $toId]);

        // Step 4: Commit — both changes are persisted together
        $pdo->commit();
        echo "Transfer of $amountInCents cents completed.\n";
    } catch (PDOException $e) {
        // Step 5: Rollback — no changes are persisted
        $pdo->rollBack();
        echo "Transfer failed: " . $e->getMessage() . "\n";
    }
}

// Setup
$pdo->exec('CREATE TABLE IF NOT EXISTS accounts (id INT PRIMARY KEY, owner VARCHAR(64), balance BIGINT NOT NULL DEFAULT 0)');
$pdo->exec("INSERT IGNORE INTO accounts (id, owner, balance) VALUES (1, 'alice', 100000), (2, 'bob', 50000)");

transferFunds($pdo, 1, 2, 25000);

// Verify
$rows = $pdo->query('SELECT id, owner, balance FROM accounts ORDER BY id')->fetchAll();
foreach ($rows as $row) {
    echo "Account {$row['id']} ({$row['owner']}): \${$row['balance']}\n";
}
```

**Expected Output:**

```
Transfer of 25000 cents completed.
Account 1 (alice): $75000
Account 2 (bob): $75000
```

**Why:** Both the debit and credit are executed inside the transaction. If either statement fails (e.g., due to a constraint violation or connection loss), the `catch` block calls `rollBack()`, undoing both changes. The database never ends up in a state where money has been debited but not credited.

**Example 2: Demonstrating DDL Implicit Commit**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=testdb', 'root', '', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$pdo->beginTransaction();

// Insert a row
$pdo->exec("INSERT INTO users (name) VALUES ('Rasmus')");
echo "Inserted 'Rasmus'.\n";

// DDL statement — triggers an implicit commit in MySQL
$pdo->exec("CREATE TABLE IF NOT EXISTS test_ddl (id INT PRIMARY KEY)");
echo "Created table 'test_ddl' (implicit commit triggered).\n";

// Attempt to rollback
$pdo->rollBack();
echo "Called rollBack().\n";

// Verify — the INSERT survives because the DDL committed it
$count = $pdo->query("SELECT COUNT(*) FROM users WHERE name = 'Rasmus'")->fetchColumn();
echo "Rows with name 'Rasmus': $count (the rollback did NOT undo the insert).\n";
```

**Expected Output:**

```
Inserted 'Rasmus'.
Created table 'test_ddl' (implicit commit triggered).
Called rollBack().
Rows with name 'Rasmus': 1 (the rollback did NOT undo the insert).
```

**Why:** MySQL automatically commits any open transaction when a DDL statement is executed. The `CREATE TABLE` statement caused an implicit commit of the `INSERT INTO users`, making it permanent. The subsequent `rollBack()` had nothing left to undo. This is why DDL statements must never be mixed with data transactions.

### Real-World Cases

**Case 1: E-Commerce Order Placement**

Placing an order involves inserting an order row, several order-item rows, and decrementing stock for each product. If the script fails after inserting the order but before decrementing stock, the inventory becomes inconsistent. Wrapping all operations in a transaction ensures that either the entire order is placed or none of it is.

**Case 2: Banking Funds Transfer**

A funds transfer debits one account and credits another. Without a transaction, a crash between the two updates would destroy money. The transaction guarantees atomicity: both updates succeed or neither does.

**Case 3: User Registration with Related Records**

Creating a user account may involve inserting into `users`, `user_profiles`, `user_roles`, and `user_settings`. If any insert fails, the transaction rolls back all of them, preventing orphaned records.

---

## Core Concept 2: Transaction Controls

### Definitions

**Core Definition**

Transaction controls are the PDO methods — `beginTransaction()`, `commit()`, and `rollBack()` — that manage the lifecycle of a database transaction.

**Technical Definition**

`PDO::beginTransaction()` disables autocommit mode and instructs the underlying database driver to start a transaction. `PDO::commit()` persists all changes made since `beginTransaction()` and restores autocommit mode. `PDO::rollBack()` discards all changes made since `beginTransaction()` and restores autocommit mode. PDO also provides `PDO::inTransaction()` to check whether a transaction is currently active.

**Beginner-Friendly Explanation**

Think of `beginTransaction()` as opening a protective envelope, `commit()` as sealing and sending the envelope (making the changes permanent), and `rollBack()` as shredding the envelope (discarding everything inside). `inTransaction()` is like checking whether the envelope is currently open.

### Purposes

- To start a transaction explicitly, disabling autocommit for the duration of the block.
- To persist all changes made during a transaction as a single atomic unit.
- To discard all changes made during a transaction when an error occurs.
- To check whether a transaction is currently active before calling commit or rollback.
- To ensure that multi-statement operations are executed safely without partial writes.

### Syntax Rules and Structure

**Complete General Syntax**

```php
// Start a transaction
$pdo->beginTransaction(): bool

// Commit the transaction
$pdo->commit(): bool

// Rollback the transaction
$pdo->rollBack(): bool

// Check if a transaction is active
$pdo->inTransaction(): bool
```

**Component Breakdown:**

- `beginTransaction()` — Disables autocommit mode. Returns `true` on success, `false` on failure (or throws `PDOException` if the driver does not support transactions).
- `commit()` — Persists all changes and restores autocommit. Returns `true` on success.
- `rollBack()` — Discards all changes and restores autocommit. Returns `true` on success.
- `inTransaction()` — Returns `true` if a transaction started by `beginTransaction()` is currently active. It does not detect transactions started by raw SQL (`START TRANSACTION`).

**Syntax Rules:**

- `beginTransaction()` must be called before `commit()` or `rollBack()`. Calling `commit()` or `rollBack()` without an active transaction throws a `PDOException`.
- `commit()` and `rollBack()` automatically restore autocommit mode after execution.
- If the script ends with an open transaction, PDO automatically rolls it back — but only if the transaction was started via `beginTransaction()`.
- `inTransaction()` is available in PHP 5.3.3+ and is particularly useful in `finally` blocks or destructors.

**Constraints and Limitations:**

- **DDL implicit commits:** As discussed in ACID, DDL statements cause implicit commits in MySQL and Oracle. After a DDL statement, `rollBack()` cannot undo prior changes.
- **Exception mode required:** Without `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`, a failing statement returns `false` rather than throwing, allowing the script to proceed to `commit()` a partial write.
- **`inTransaction()` limitations:** It only detects transactions started via `PDO::beginTransaction()`. Transactions started by `START TRANSACTION` or by the database's implicit transaction handling are not detected.
- **SQLite caveats:** In SQLite, `inTransaction()` may not reflect the actual transaction state after certain errors (e.g., `SQLITE_FULL`, `SQLITE_BUSY`), because SQLite may automatically roll back the transaction without PDO's knowledge.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Standard Transaction with Try/Catch/Finally**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=shop', 'root', '', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

try {
    // Step 1: Begin the transaction
    $pdo->beginTransaction();

    // Step 2: Execute multiple statements
    $pdo->exec("INSERT INTO orders (customer_id, total) VALUES (1, 99.99)");
    $orderId = (int) $pdo->lastInsertId();

    $pdo->exec("INSERT INTO order_items (order_id, product_id, quantity) VALUES ($orderId, 10, 2)");
    $pdo->exec("INSERT INTO order_items (order_id, product_id, quantity) VALUES ($orderId, 20, 1)");

    $pdo->exec("UPDATE products SET stock = stock - 2 WHERE id = 10");
    $pdo->exec("UPDATE products SET stock = stock - 1 WHERE id = 20");

    // Step 3: Commit all changes
    $pdo->commit();
    echo "Order #$orderId placed successfully.\n";
} catch (PDOException $e) {
    // Step 4: Rollback on any error
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }
    echo "Order failed: " . $e->getMessage() . "\n";
}
```

**Expected Output:**

```
Order #1 placed successfully.
```

**Why:** The script begins a transaction, inserts an order and two line items, and decrements stock. If any statement fails, the `catch` block checks `inTransaction()` and rolls back, ensuring no partial order is left in the database. The `inTransaction()` check prevents a secondary exception if the transaction was already rolled back automatically.

**Example 2: Using `inTransaction()` for Safe Cleanup**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=testdb', 'root', '', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$pdo->beginTransaction();

try {
    $pdo->exec("INSERT INTO logs (message) VALUES ('Processing started')");
    echo "Log entry inserted.\n";
    $pdo->commit();
    echo "Committed.\n";
} finally {
    // Safe cleanup: only rollback if a transaction is still active
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
        echo "Rolled back (transaction was still open).\n";
    } else {
        echo "No active transaction to rollback.\n";
    }
}
```

**Expected Output:**

```
Log entry inserted.
Committed.
No active transaction to rollback.
```

**Why:** After `commit()`, `inTransaction()` returns `false`, so the `finally` block does not attempt a redundant rollback. This pattern is safe for both success and failure paths.

### Real-World Cases

**Case 1: Batch Data Import**

A CSV import script processes thousands of rows. Wrapping the entire import in a single transaction (or batching several hundred rows per transaction) ensures that a failure midway through does not leave the database with a half-imported file.

**Case 2: Multi-Table User Registration**

Creating a user account involves inserting into `users`, `user_profiles`, and `user_roles`. The transaction ensures that if profile creation fails, the user row is also rolled back, preventing orphaned accounts.

**Case 3: Framework-Level Transaction Management**

Laravel's `DB::transaction(function () { ... })` wrapper internally calls `beginTransaction()`, executes the closure, and calls `commit()` on success or `rollBack()` on exception. Symfony's `TransactionManager` provides a similar abstraction with support for nested transactions via savepoints.

---

## Core Concept 3: Nested Transactions & Savepoints

### Definitions

**Core Definition**

Nested transactions allow a transaction to be started inside another transaction, with the ability to roll back only the inner transaction's changes without discarding the outer transaction's work.

**Technical Definition**

PDO does **not** natively support nested transactions. Calling `beginTransaction()` while a transaction is already active throws a `PDOException` ("There is already an active transaction"). To simulate nested transactions, developers use SQL `SAVEPOINT` statements: `SAVEPOINT name` creates a named savepoint within the current transaction, and `ROLLBACK TO SAVEPOINT name` rolls back to that savepoint without discarding the entire transaction. Savepoints are supported by MySQL (InnoDB) and PostgreSQL.

**Beginner-Friendly Explanation**

Imagine you are writing a document inside a protective envelope (the transaction). A savepoint is like a bookmark you place on page 10. If you make a mistake on page 15, you can roll back to the bookmark on page 10 without destroying the entire document. PDO does not give you a built-in "nested envelope," but you can use savepoints (bookmarks) to achieve the same effect.

### Purposes

- To simulate nested transactions in frameworks and ORMs that expect nesting support.
- To allow partial rollback of a transaction without discarding the entire transaction.
- To isolate the effects of sub-operations within a larger transaction.
- To enable transaction management in deep call stacks where multiple layers of code may independently start and commit transactions.
- To support testing scenarios where a test transaction wraps a business transaction and needs to roll back only the test's changes.

### Syntax Rules and Structure

**Complete General Syntax: Manual Savepoint Usage**

```php
$pdo->beginTransaction();           // Outer transaction

// ... outer statements ...

$pdo->exec('SAVEPOINT sp1');        // Create a savepoint

try {
    // ... inner statements ...
    $pdo->exec('RELEASE SAVEPOINT sp1');  // Success: release the savepoint
} catch (Throwable $e) {
    $pdo->exec('ROLLBACK TO SAVEPOINT sp1');  // Failure: rollback to savepoint
    // Outer transaction continues
}

// ... more outer statements ...
$pdo->commit();
```

**Component Breakdown:**

- `SAVEPOINT sp1` — Creates a named savepoint within the current transaction. The name must be a valid SQL identifier.
- `ROLLBACK TO SAVEPOINT sp1` — Rolls back all changes made after the savepoint was created. The savepoint itself remains active and can be reused.
- `RELEASE SAVEPOINT sp1` — Removes the savepoint. Changes made after it are not rolled back; they remain part of the transaction.
- `COMMIT` — Commits the entire transaction, including all changes made after the savepoint.

**Complete General Syntax: Nested Transaction Wrapper Class**

```php
class TransactionManager
{
    private PDO $pdo;
    private int $nestLevel = 0;
    private array $savepoints = [];

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function begin(): void
    {
        if ($this->nestLevel === 0) {
            $this->pdo->beginTransaction();
        } else {
            $name = 'sp_' . $this->nestLevel;
            $this->pdo->exec("SAVEPOINT $name");
            $this->savepoints[] = $name;
        }
        $this->nestLevel++;
    }

    public function commit(): void
    {
        $this->nestLevel--;
        if ($this->nestLevel === 0) {
            $this->pdo->commit();
        } else {
            $name = array_pop($this->savepoints);
            $this->pdo->exec("RELEASE SAVEPOINT $name");
        }
    }

    public function rollback(): void
    {
        $this->nestLevel--;
        if ($this->nestLevel === 0) {
            $this->pdo->rollBack();
            $this->savepoints = [];
        } else {
            $name = array_pop($this->savepoints);
            $this->pdo->exec("ROLLBACK TO SAVEPOINT $name");
        }
    }

    public function isNested(): bool
    {
        return $this->nestLevel > 1;
    }
}
```

**Component Breakdown:**

- `$nestLevel` — Tracks the current nesting depth. Level 0 means no transaction; level 1 means the outermost transaction; levels 2+ are nested (simulated via savepoints).
- `$savepoints` — A stack of savepoint names for the current nesting levels.
- `begin()` — At level 0, starts a real transaction. At deeper levels, creates a savepoint.
- `commit()` — At level 1, commits the real transaction. At deeper levels, releases the savepoint.
- `rollback()` — At level 1, rolls back the real transaction. At deeper levels, rolls back to the savepoint.

**Syntax Rules:**

- Savepoint names must be unique within a transaction. Reusing a name overwrites the previous savepoint.
- `ROLLBACK TO SAVEPOINT` does not remove the savepoint; `RELEASE SAVEPOINT` does.
- Savepoints are supported by MySQL (InnoDB) and PostgreSQL. SQLite does not support named savepoints in the same way.
- The nesting wrapper class must be used consistently — all transaction operations must go through the wrapper, not directly through `$pdo`.

**Constraints and Limitations:**

- **PDO does not natively support nested transactions.** The savepoint approach is a simulation, not a true nested transaction. If the outer transaction is rolled back, all inner changes are rolled back as well, regardless of savepoints.
- **Savepoints are not portable across all databases.** SQLite's savepoint implementation differs, and some drivers may not support them at all.
- **DDL implicit commits invalidate savepoints.** A DDL statement within a transaction implicitly commits everything, making savepoints useless.
- **Framework implementations vary.** Laravel uses savepoints for `DB::transaction()` nesting, but Doctrine ORM's transaction nesting is handled differently (via a nesting counter and nested transaction emulation).

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Manual Savepoint with Partial Rollback**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=testdb', 'root', '', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$pdo->beginTransaction();

// Step 1: Insert a valid record
$pdo->exec("INSERT INTO users (name) VALUES ('Alice')");
echo "Inserted Alice.\n";

// Step 2: Create a savepoint before risky operation
$pdo->exec('SAVEPOINT before_risky');

try {
    // Step 3: Attempt a risky operation
    $pdo->exec("INSERT INTO users (name) VALUES ('Bob')");
    $pdo->exec("INSERT INTO non_existent_table (x) VALUES (1)"); // Fails
    echo "This line will not be reached.\n";
} catch (PDOException $e) {
    // Step 4: Rollback only to the savepoint
    $pdo->exec('ROLLBACK TO SAVEPOINT before_risky');
    echo "Rolled back to savepoint. Error: " . $e->getMessage() . "\n";
}

// Step 5: Continue with the outer transaction
$pdo->exec("INSERT INTO users (name) VALUES ('Carol')");
echo "Inserted Carol.\n";

// Step 6: Commit the entire transaction
$pdo->commit();
echo "Committed.\n";

// Verify: Alice and Carol exist; Bob does not
$rows = $pdo->query('SELECT name FROM users ORDER BY name')->fetchAll(PDO::FETCH_COLUMN);
print_r($rows);
```

**Expected Output:**

```
Inserted Alice.
Rolled back to savepoint. Error: SQLSTATE[42S02]: Base table or view not found: 1146 Table 'testdb.non_existent_table' doesn't exist
Inserted Carol.
Committed.
Array
(
    [0] => Alice
    [1] => Carol
)
```

**Why:** Alice's insert is before the savepoint and survives. Bob's insert is after the savepoint and is rolled back when the risky operation fails. Carol's insert occurs after the rollback and is committed. The savepoint allows partial rollback without discarding the entire transaction.

**Example 2: Nested Transaction Wrapper in Action**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=testdb', 'root', '', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$tm = new TransactionManager($pdo);

// Outer transaction
$tm->begin();

$pdo->exec("INSERT INTO audit_log (action) VALUES ('outer_start')");

// Inner transaction (simulated via savepoint)
$tm->begin();
try {
    $pdo->exec("INSERT INTO audit_log (action) VALUES ('inner_operation')");
    $pdo->exec("INSERT INTO non_existent (x) VALUES (1)"); // Fails
    $tm->commit();
} catch (Throwable $e) {
    $tm->rollback(); // Rolls back to savepoint
    echo "Inner transaction rolled back.\n";
}

// Outer continues
$pdo->exec("INSERT INTO audit_log (action) VALUES ('outer_end')");
$tm->commit();

echo "Final transaction committed.\n";

// Verify: outer_start and outer_end exist; inner_operation does not
$rows = $pdo->query('SELECT action FROM audit_log ORDER BY id')->fetchAll(PDO::FETCH_COLUMN);
print_r($rows);
```

**Expected Output:**

```
Inner transaction rolled back.
Final transaction committed.
Array
(
    [0] => outer_start
    [1] => outer_end
)
```

**Why:** The `TransactionManager` tracks nesting levels. The inner `begin()` creates a savepoint; the inner `rollback()` rolls back to that savepoint without affecting the outer transaction. The outer `commit()` persists `outer_start` and `outer_end`, but not `inner_operation`.

### Real-World Cases

**Case 1: Laravel Nested Transactions**

Laravel's `DB::transaction()` method uses savepoints to support nested transactions. When a transaction is started inside another, Laravel issues a `SAVEPOINT` and increments a nesting counter. On rollback, it rolls back to the appropriate savepoint without discarding the outer transaction.

**Case 2: Doctrine DBAL Transaction Nesting**

Doctrine DBAL's `Connection` class implements nested transactions via a nesting level counter. When `beginTransaction()` is called at nesting level > 1, it does nothing (or creates a savepoint, depending on the driver). Only the outermost `commit()` actually commits the transaction to the database.

**Case 3: Integration Tests with Transaction Rollback**

A PHPUnit test suite wraps each test in a transaction. If the test itself uses transactions (e.g., testing a service that manages its own transactions), the test wrapper uses savepoints to allow the service's transactions to commit and rollback independently, while the outer test transaction is always rolled back at the end of the test.

---

## Core Concept 4: Resilient Error Catching

### Definitions

**Core Definition**

Resilient error catching is the practice of wrapping database mutations in try/catch blocks, detecting transient failures such as deadlocks, and automatically retrying the failed operation with backoff.

**Technical Definition**

A **deadlock** occurs when two or more transactions hold locks that the others need, creating a circular wait. MySQL's InnoDB engine detects deadlocks and rolls back one transaction (the "victim") with error 1213 (SQLSTATE 40001). A **lock-wait timeout** (error 1205) occurs when a transaction waits too long for a lock. Both conditions are transient and retryable: the correct response is to roll back the victim transaction and retry it from the beginning after a short pause. Exponential backoff with jitter is the standard retry strategy to avoid thundering-herd effects.

**Beginner-Friendly Explanation**

Imagine two people trying to walk through a narrow doorway from opposite sides at the same time. Neither can move until the other backs off. The database solves this by picking one person and telling them to step back and try again. A resilient application catches that "step back" signal (the deadlock error) and automatically retries the operation after a brief wait, rather than showing an error to the user.

### Purposes

- To detect and recover from transient database failures such as deadlocks and lock-wait timeouts.
- To ensure that a failed transaction is rolled back cleanly before retrying.
- To implement exponential backoff with jitter, reducing contention and avoiding synchronized retry storms.
- To limit the number of retry attempts, preventing infinite loops under persistent failure.
- To distinguish between retryable errors (deadlocks, timeouts) and non-retryable errors (constraint violations, syntax errors).

### Syntax Rules and Structure

**Complete General Syntax: Retry Wrapper with Exponential Backoff**

```php
function withRetry(PDO $pdo, callable $operation, int $maxAttempts = 3): mixed
{
    $attempt = 0;

    while (true) {
        $attempt++;

        try {
            $pdo->beginTransaction();
            $result = $operation($pdo);
            $pdo->commit();
            return $result;
        } catch (PDOException $e) {
            if ($pdo->inTransaction()) {
                $pdo->rollBack();
            }

            // Determine if the error is retryable
            $isRetryable = in_array($e->getCode(), ['40001', 'HY000'], true)
                || str_contains($e->getMessage(), 'Deadlock')
                || str_contains($e->getMessage(), 'Lock wait timeout');

            if (!$isRetryable || $attempt >= $maxAttempts) {
                throw $e;
            }

            // Exponential backoff with jitter
            $baseDelay = 100; // milliseconds
            $delay = $baseDelay * (2 ** ($attempt - 1));
            $jitter = random_int(0, $delay / 2);
            usleep(($delay + $jitter) * 1000);
        }
    }
}
```

**Component Breakdown:**

- `$maxAttempts` — The maximum number of retry attempts (default: 3). After this, the original exception is re-thrown.
- `$pdo->beginTransaction()` — Starts a fresh transaction for each attempt.
- `$operation($pdo)` — The business logic closure that performs the database mutations.
- `$pdo->commit()` — Commits if the operation succeeds.
- `$pdo->rollBack()` — Rolls back the failed attempt before retrying.
- `$e->getCode()` — Returns the SQLSTATE code. Deadlocks use `40001`; lock-wait timeouts use `HY000` (with error 1205 in the message).
- `$baseDelay` — The initial delay in milliseconds (100 ms). Each subsequent attempt doubles the delay.
- `$jitter` — A random additional delay (0 to half the base delay) to prevent synchronized retries.
- `usleep(($delay + $jitter) * 1000)` — Sleeps for the computed milliseconds.

**Syntax Rules:**

- The retry wrapper must roll back the failed transaction **before** retrying. Attempting to retry without rolling back leaves the connection in an inconsistent state.
- The retryable error codes must be checked carefully. Not all `PDOException` errors are retryable. Constraint violations (e.g., unique key violations, SQLSTATE 23000) should **not** be retried.
- The retry wrapper should be used only for transient failures. Persistent failures (e.g., missing table, syntax error) will never succeed on retry.
- The business logic closure must be **idempotent** or safe to re-execute. Re-running a transfer that partially succeeded before the deadlock must not double-apply changes.
- Use `random_int()` for cryptographically secure jitter, or `mt_rand()` for performance-critical paths.

**Constraints and Limitations:**

- **Deadlocks are normal.** Under InnoDB, deadlocks are a normal part of concurrent operation, not a bug. The application must expect them and retry.
- **Retrying forever is dangerous.** Always set a maximum attempt count. Under persistent contention, infinite retries can exhaust PHP-FPM workers and worsen the problem.
- **Stale data risk.** When a transaction is retried, the business logic must re-read any data it depends on. If the logic uses values read before the deadlock, the retry may operate on stale data.
- **Long transactions increase deadlock risk.** The longer a transaction holds locks, the wider the window for a deadlock cycle. Keep transactions as short as possible.
- **Index matters.** A `WHERE` clause without a usable index forces a broad scan that locks many rows, dramatically increasing collision odds. Ensure all filtered columns are indexed.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Basic Retry Wrapper for Deadlocks**

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=bank', 'root', '', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

function transferWithRetry(PDO $pdo, int $from, int $to, int $amount, int $maxAttempts = 3): void
{
    $attempt = 0;

    while (true) {
        $attempt++;
        try {
            $pdo->beginTransaction();

            // Lock rows in a deterministic order to reduce deadlock risk
            $stmt = $pdo->prepare('SELECT balance FROM accounts WHERE id = :id FOR UPDATE');
            $stmt->execute([':id' => $from]);
            $fromBalance = (int) $stmt->fetchColumn();

            if ($fromBalance < $amount) {
                throw new RuntimeException('Insufficient funds');
            }

            $pdo->prepare('UPDATE accounts SET balance = balance - :amt WHERE id = :id')
                ->execute([':amt' => $amount, ':id' => $from]);
            $pdo->prepare('UPDATE accounts SET balance = balance + :amt WHERE id = :id')
                ->execute([':amt' => $amount, ':id' => $to]);

            $pdo->commit();
            echo "Attempt $attempt: Transfer succeeded.\n";
            return;
        } catch (PDOException $e) {
            if ($pdo->inTransaction()) {
                $pdo->rollBack();
            }

            $isDeadlock = $e->getCode() === '40001'
                || str_contains($e->getMessage(), 'Deadlock');

            if (!$isDeadlock || $attempt >= $maxAttempts) {
                throw $e;
            }

            $delay = 100 * (2 ** ($attempt - 1)) + random_int(0, 50);
            echo "Attempt $attempt: Deadlock detected. Retrying in {$delay}ms...\n";
            usleep($delay * 1000);
        }
    }
}

transferWithRetry($pdo, 1, 2, 100);
```

**Expected Output (with a simulated deadlock on attempt 1):**

```
Attempt 1: Deadlock detected. Retrying in 100ms...
Attempt 2: Transfer succeeded.
```

**Why:** The wrapper catches the `PDOException`, checks whether the SQLSTATE code is `40001` (deadlock), rolls back the failed transaction, waits for an exponentially increasing delay, and retries the entire operation. The `SELECT ... FOR UPDATE` locks the rows in a consistent order, reducing the chance of a deadlock on subsequent attempts.

**Example 2: Retry with SQLSTATE and Error Code Detection**

```php
<?php
function isRetryableError(PDOException $e): bool
{
    // SQLSTATE 40001: Deadlock
    if ($e->getCode() === '40001') {
        return true;
    }

    // MySQL error 1213: Deadlock found when trying to get lock
    // MySQL error 1205: Lock wait timeout exceeded
    $errorInfo = $e->errorInfo;
    if (isset($errorInfo[1]) && in_array($errorInfo[1], [1213, 1205], true)) {
        return true;
    }

    return false;
}
```

**Expected Output:** No visible output (this is a helper function).

**Why:** The `PDOException` object provides `getCode()` (SQLSTATE) and `errorInfo` (driver-specific error code, error message, and SQLSTATE). Checking both ensures that deadlocks (1213) and lock-wait timeouts (1205) are correctly identified as retryable.

**Example 3: Complete Transaction Helper with Retry, Test Mode, and Nested Support**

```php
<?php
class Database
{
    private PDO $pdo;
    private int $nestLevel = 0;
    private array $savepoints = [];

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function transaction(callable $fn, int $attempts = 1, bool $testMode = false): mixed
    {
        if ($attempts < 1) {
            throw new InvalidArgumentException('Attempts must be at least 1.');
        }

        $lastException = null;

        for ($attempt = 1; $attempt <= $attempts; $attempt++) {
            try {
                $this->begin();

                $result = $fn($this);

                if ($testMode) {
                    $this->rollback();
                } else {
                    $this->commit();
                }

                return $result;
            } catch (Throwable $e) {
                if ($this->inTransaction()) {
                    $this->rollback();
                }

                $lastException = $e;

                if (!$this->isRetryable($e) || $attempt === $attempts) {
                    throw $e;
                }

                $delay = 100 * (2 ** ($attempt - 1)) + random_int(0, 50);
                usleep($delay * 1000);
            }
        }

        throw $lastException;
    }

    private function begin(): void
    {
        if ($this->nestLevel === 0) {
            $this->pdo->beginTransaction();
        } else {
            $name = 'sp_' . $this->nestLevel;
            $this->pdo->exec("SAVEPOINT $name");
            $this->savepoints[] = $name;
        }
        $this->nestLevel++;
    }

    private function commit(): void
    {
        $this->nestLevel--;
        if ($this->nestLevel === 0) {
            $this->pdo->commit();
        } else {
            $name = array_pop($this->savepoints);
            $this->pdo->exec("RELEASE SAVEPOINT $name");
        }
    }

    private function rollback(): void
    {
        $this->nestLevel--;
        if ($this->nestLevel === 0) {
            $this->pdo->rollBack();
            $this->savepoints = [];
        } else {
            $name = array_pop($this->savepoints);
            $this->pdo->exec("ROLLBACK TO SAVEPOINT $name");
        }
    }

    private function inTransaction(): bool
    {
        return $this->nestLevel > 0;
    }

    private function isRetryable(Throwable $e): bool
    {
        if (!$e instanceof PDOException) {
            return false;
        }

        if ($e->getCode() === '40001') {
            return true;
        }

        $errorInfo = $e->errorInfo;
        return isset($errorInfo[1]) && in_array($errorInfo[1], [1213, 1205], true);
    }
}
```

**Expected Output:** No visible output (this is a reusable helper class).

**Why:** This class combines nested transaction support (via savepoints), retry logic with exponential backoff, and test mode (rollback on success) into a single reusable component. It checks `isRetryable()` to avoid retrying non-transient errors, and it tracks nesting levels to simulate nested transactions.

### Real-World Cases

**Case 1: High-Concurrency Order Processing**

An e-commerce platform processes thousands of orders per minute. Under peak load, deadlocks occur when two orders try to update the same inventory row. A retry wrapper catches the deadlock, waits 100–500 ms with jitter, and retries the order. The customer sees no error; the order is processed successfully on the second attempt.

**Case 2: Banking System with Lock-Wait Timeouts**

A banking system uses `SELECT ... FOR UPDATE` to lock account rows during transfers. Under heavy load, a transaction may wait longer than `innodb_lock_wait_timeout` (default: 50 seconds), triggering error 1205. The retry wrapper catches the timeout, rolls back, and retries with a fresh transaction. If the timeout persists after three attempts, the user is shown a "please try again" message.

**Case 3: WordPress Deadlock Handling**

WordPress sites on high-traffic MySQL databases occasionally encounter InnoDB deadlocks (error 1213) during concurrent post updates or option writes. Resilient plugins wrap their database operations in retry loops that catch the deadlock error and retry after a short delay, preventing "database error" pages for end users.

---

## References

- PHP: Transactions and auto-commit — Manual – https://static.php.net/manual/en/pdo.transactions.php
- PHP: PDO::beginTransaction — Manual – https://www.php.net/manual/en/pdo.begintransaction.php
- PHP: PDO::commit — Manual – https://www.php.net/manual/en/pdo.commit.php
- PHP: PDO::rollBack — Manual – https://www.php.net/manual/en/pdo.rollback.php
- PHP: PDO::inTransaction — Manual – https://static.php.net/manual/en/pdo.intransaction.php
- PHP: API support for transactions (MySQLi) — Manual – https://static.php.net/manual/en/mysqli.quickstart.transactions.php
- PHP: PDO Transactions — Secure PHP Development (GitHub) – https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Using-PHP-to-Access-MySQL/PDO-Transactions.md
- Handling Deadlocks — Secure PHP Development (GitHub) – https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Using-PHP-to-Access-MySQL/Handling-Deadlocks.md
- Lab: PDO Transactions and Atomicity — Secure PHP Development (GitHub) – https://github.com/armourinfosec/Secure-PHP-Development/blob/586b887138a30c25fc459684c78dffe0a2296c17/Labs/Lab-PDO-Transactions.md
- Transactions — InitORM Database Wiki (GitHub) – https://github.com/InitORM/Database/wiki/Transactions
- How to use nested DB transactions (MySQL 5+, PostgreSQL) — Yii Framework Wiki – https://www.yiiframework.com/wiki/38/how-to-use-nested-db-transactions-mysql-5-postgresql
- Microsoft: How to Perform Transactions — PHP Drivers for SQL Server – https://learn.microsoft.com/en-us/sql/connect/php/how-to-perform-transactions
- PDO事务操作 — 传智播客 – http://book.itheima.net/course/1258677827423715330/1265953996862119937/1277478421638684678
- MyPDO — Packagist – https://packagist.org/packages/rlanvin/php-mypdo