# PHP Data Objects (PDO) & Drivers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

PDO (PHP Data Objects) is a lightweight, consistent, object-oriented database access abstraction layer built into PHP that provides a uniform interface for interacting with multiple database systems through driver-specific implementations.

**Technical Definition**

PDO is a PHP extension that defines a core `PDO` class and a set of driver-specific extensions (`pdo_mysql`, `pdo_pgsql`, `pdo_sqlite`, etc.). It abstracts the "database driver" (the connection layer) but does **not** abstract the SQL dialect itself — that is, you still write database-specific SQL, but the PHP code to connect, prepare, execute, and fetch remains identical across databases.

**Beginner-Friendly Explanation**

Imagine you have several different power outlets in your house — one for a lamp, one for a TV, one for a phone charger. Each device has a different plug shape. PDO is like a universal power strip: you plug any device into it, and it handles the differences behind the scenes. You write one set of PHP commands, and PDO translates them for MySQL, PostgreSQL, or SQLite automatically.

### Key Characteristics

- **Database Driver Abstraction:** PDO abstracts the connection and data-access layer, not the SQL syntax. You still write `SELECT`, `INSERT`, etc., but the PHP API remains consistent.
- **Object-Oriented API:** All interactions use the `PDO` class and `PDOStatement` objects.
- **Prepared Statements:** Native support for parameterized queries, which prevents SQL injection.
- **Exception-Based Error Handling:** Can be configured via `PDO::ATTR_ERRMODE` to throw `PDOException` objects.
- **Multiple Driver Support:** Officially supports MySQL, PostgreSQL, SQLite, SQL Server, Oracle, and more via community drivers.
- **Persistent Connections:** Optional caching of connections across script executions to reduce connection overhead.
- **Flexible Fetch Modes:** Rows can be returned as associative arrays, objects, custom classes, or column values.

### Prerequisites

- **PHP 5.1.0+** (PDO became bundled by default in PHP 5.1; PHP 7.0+ recommended for production, PHP 8.x for modern features).
- **PDO Extension enabled** (`extension=pdo` in `php.ini`).
- **Driver-specific extensions installed** for the target database:
  - `pdo_mysql` for MySQL/MariaDB
  - `pdo_pgsql` for PostgreSQL
  - `pdo_sqlite` for SQLite (often enabled by default)
- **A running database server** (or SQLite file for SQLite).
- **Valid credentials and database name** for the target database.

### Related Programming Areas

- **Database Administration:** Understanding of MySQL, PostgreSQL, or SQLite server configuration, user privileges, and connection limits.
- **SQL (Structured Query Language):** PDO executes SQL; you must know how to write queries, joins, indexes, and transactions.
- **Web Application Security:** PDO's prepared statements are a primary defense against SQL injection.
- **Concurrency and Performance:** Connection pooling, persistent connections, and read/write splitting are performance-oriented topics.
- **Long-Running PHP Runtimes:** Swoole, RoadRunner, and FrankenPHP change how database connections behave across requests.

### Core Concepts / Features

1. **PDO Architecture** — The abstract layer and database drivers.
2. **Connection Lifecycle & DSN** — Configuring Data Source Names, persistent vs. non-persistent connections, and connection termination.
3. **Error Handlers & Attributes** — `PDO::ATTR_ERRMODE` and `PDO::ATTR_DEFAULT_FETCH_MODE`.
4. **Connection Pooling & Read/Write Splits** — Behavior in long-running CLI processes (Swoole, RoadRunner, FrankenPHP) vs. traditional FPM life cycles.

---

## Core Concept 1: PDO Architecture

### Definitions

**Core Definition**

PDO architecture consists of an abstract database access layer (the `PDO` class) and a set of database-specific drivers that implement the actual communication with each database engine.

**Technical Definition**

The PDO extension is composed of two layers: (1) the **PDO Core**, which provides the `PDO` class, `PDOStatement` class, and `PDOException` class, and (2) the **PDO Drivers**, which are separate PHP extensions (e.g., `pdo_mysql.so`, `pdo_pgsql.so`, `pdo_sqlite.so`) that translate PDO method calls into database-specific wire protocols.

**Beginner-Friendly Explanation**

Think of PDO as a universal remote control and the drivers as the specific codes for each TV brand. The remote has the same buttons (connect, query, fetch), but the driver knows how to talk to that particular TV. You don't need a different remote for each brand — you just load the right driver.

### Purposes

- To provide a single, consistent API for accessing different database systems without rewriting application code.
- To isolate database-specific implementation details from business logic.
- To enable database portability at the connection and query-execution level (though not at the SQL dialect level).
- To support prepared statements and parameter binding uniformly across databases.
- To allow developers to switch databases by changing only the DSN and driver, not the PHP code structure.

### Syntax Rules and Structure

**Complete General Syntax: Creating a PDO Instance**

```php
$pdo = new PDO(
    string $dsn,           // Data Source Name (driver:host=...;dbname=...)
    ?string $username = null,
    ?string $password = null,
    ?array $options = null  // Optional driver/connection options
);
```

**Component Breakdown:**

- `$dsn` (string, required): The Data Source Name, which specifies the driver (`mysql:`, `pgsql:`, `sqlite:`) followed by driver-specific connection parameters. This is the only required argument.
- `$username` (string|null, optional): The database username. Not required for SQLite.
- `$password` (string|null, optional): The database password. Not required for SQLite.
- `$options` (array|null, optional): An associative array of PDO attributes and driver-specific options, such as `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`.

**Syntax Rules:**

- The driver prefix (e.g., `mysql:`) must match an installed PDO driver extension.
- DSN parameters are separated by semicolons (`;`).
- The `charset` parameter is supported by MySQL and PostgreSQL drivers but not by all drivers.
- SQLite DSNs use a file path or `:memory:` for in-memory databases.

**Constraints and Limitations:**

- PDO does **not** abstract SQL syntax. A `LIMIT` clause in MySQL differs from PostgreSQL's `LIMIT ... OFFSET` in some contexts.
- Not all PDO drivers support the same attributes or methods. For example, `PDO::ATTR_PERSISTENT` behavior varies by driver.
- The `pdo_sqlite` driver is typically enabled by default, but `pdo_mysql` and `pdo_pgsql` must be explicitly enabled during PHP installation.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Connecting to MySQL**

```php
<?php
// Step 1: Define the DSN for a MySQL database
// The DSN format is: driver:host=HOST;dbname=DATABASE;charset=CHARSET
$dsn = 'mysql:host=localhost;dbname=testdb;charset=utf8mb4';

// Step 2: Define credentials
$username = 'webuser';
$password = 'secret123';

// Step 3: Create the PDO instance
try {
    $pdo = new PDO($dsn, $username, $password);
    echo "Connected to MySQL successfully.\n";
} catch (PDOException $e) {
    // Step 4: Handle connection errors
    echo "Connection failed: " . $e->getMessage() . "\n";
}
```

**Expected Output:**

```
Connected to MySQL successfully.
```

**Why:** The MySQL driver (`pdo_mysql`) is loaded, the DSN specifies `localhost` and `testdb`, and valid credentials are provided. If any part fails, a `PDOException` is thrown and caught.

**Example 2: Connecting to SQLite (File-Based)**

```php
<?php
// Step 1: SQLite DSN uses a file path or :memory:
$dsn = 'sqlite:/var/www/data/app.sqlite3';

// Step 2: SQLite requires no username or password
try {
    $pdo = new PDO($dsn);
    echo "Connected to SQLite database file.\n";
} catch (PDOException $e) {
    echo "Connection failed: " . $e->getMessage() . "\n";
}
```

**Expected Output:**

```
Connected to SQLite database file.
```

**Why:** SQLite is file-based, so the DSN points directly to the database file. No authentication is needed. The `pdo_sqlite` driver handles all file I/O.

**Example 3: Connecting to PostgreSQL**

```php
<?php
// Step 1: PostgreSQL DSN
$dsn = 'pgsql:host=127.0.0.1;port=5432;dbname=appdb';

// Step 2: Credentials
$username = 'pguser';
$password = 'pgpass';

// Step 3: Connect with options
try {
    $pdo = new PDO($dsn, $username, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    ]);
    echo "Connected to PostgreSQL successfully.\n";
} catch (PDOException $e) {
    echo "Connection failed: " . $e->getMessage() . "\n";
}
```

**Expected Output:**

```
Connected to PostgreSQL successfully.
```

**Why:** The `pgsql:` driver prefix selects the PostgreSQL PDO driver. The DSN includes host, port, and database name. The options array sets the error mode to throw exceptions.

### Real-World Cases

**Case 1: Multi-Database SaaS Application**

A SaaS platform offers both MySQL and PostgreSQL backends for different enterprise customers. By using PDO, the application's data access layer uses identical code for both databases:

```php
$pdo = new PDO($config['dsn'], $config['username'], $config['password']);
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute([':email' => $email]);
```

The only difference is the DSN string in the configuration. No code changes are needed in the application logic.

**Case 2: Drupal's Database Layer**

Drupal's database abstraction layer is built on top of PDO. Drupal ships with drivers for MySQL, PostgreSQL, and SQLite, each consisting of a series of files in a named module (e.g., `core/modules/mysql/src/Driver/Database/mysql`).

---

## Core Concept 2: Connection Lifecycle & DSN

### Definitions

**Core Definition**

The connection lifecycle encompasses how a PDO connection is established via a Data Source Name (DSN), maintained (persistent vs. non-persistent), and terminated.

**Technical Definition**

A PDO connection is established when a `PDO` object is instantiated with a DSN. The connection remains active for the lifetime of the `PDO` object. If `PDO::ATTR_PERSISTENT` is set to `true`, the connection is cached by the driver and reused across script executions when the same credentials are requested.

**Beginner-Friendly Explanation**

A connection is like a phone call to the database. A **non-persistent** connection is a phone call you hang up when you're done — it's closed at the end of your script. A **persistent** connection is like leaving the phone off the hook so the next person in your office can use the same line without dialing again.

### Purposes

- To configure database connection parameters via DSN strings.
- To reduce connection overhead by reusing persistent connections.
- To manage connection termination explicitly for long-running processes.
- To support multiple database drivers with a consistent connection API.
- To control connection behavior across different PHP execution models (FPM, CLI, Swoole).

### Syntax Rules and Structure

**Complete General Syntax: DSN Formats by Driver**

```php
// MySQL / MariaDB
$dsn = 'mysql:host=HOST;port=PORT;dbname=DATABASE;charset=CHARSET';

// PostgreSQL
$dsn = 'pgsql:host=HOST;port=PORT;dbname=DATABASE';

// SQLite (file-based)
$dsn = 'sqlite:/path/to/database.sqlite3';

// SQLite (in-memory)
$dsn = 'sqlite::memory:';

// SQL Server (via PDO_SQLSRV)
$dsn = 'sqlsrv:Server=HOST,PORT;Database=DATABASE';
```

**Component Breakdown:**

- `driver:` — The PDO driver prefix. Must match an installed PDO driver extension.
- `host=` — The database server hostname or IP address.
- `port=` — The database server port (default: 3306 for MySQL, 5432 for PostgreSQL, 1433 for SQL Server).
- `dbname=` — The name of the database to connect to.
- `charset=` — The character set for the connection (MySQL and PostgreSQL only).
- `sqlite:` — For SQLite, followed by the file path or `:memory:`.

**Complete General Syntax: Persistent Connection**

```php
$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_PERSISTENT => true
]);
```

**Component Breakdown:**

- `PDO::ATTR_PERSISTENT => true` — Enables persistent connection caching. The connection is not closed at script end but cached for reuse.
- The value is converted to `bool`. If a non-numeric string is used, multiple persistent connection pools can be created for incompatible settings.

**Complete General Syntax: Explicit Connection Termination**

```php
$pdo = null;  // Destroys the PDO object and closes the connection
```

**Syntax Rules:**

- A persistent connection is cached by the driver and reused when another script requests a connection with the same credentials (DSN, username, password, and options).
- To close a non-persistent connection, set all references to the `PDO` object to `null`. PHP will also close the connection automatically at script end.
- If there are other references (e.g., from `PDOStatement` instances), they must also be destroyed.
- Persistent connections cannot be closed by setting `$pdo = null`; they remain in the cache until the PHP process (or FPM worker) terminates.

**Constraints and Limitations:**

- **Persistent connections can carry over state:** Transactions, table locks, temporary tables, and session variables may persist across requests, causing unexpected behavior.
- **Connection limits:** Persistent connections occupy server sockets and threads. If many FPM workers hold persistent connections, the database's `max_connections` limit may be exhausted.
- **Deadlocks:** If a script dies mid-transaction, the lock may not be released, causing subsequent scripts to block indefinitely.
- **Persistent connections are not true connection pools:** They cache one connection per FPM worker, not a shared pool.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Non-Persistent Connection and Explicit Termination**

```php
<?php
// Step 1: Create a non-persistent connection (default behavior)
$dsn = 'mysql:host=localhost;dbname=testdb;charset=utf8mb4';
$pdo = new PDO($dsn, 'webuser', 'secret123');

// Step 2: Use the connection
$stmt = $pdo->query('SELECT COUNT(*) AS cnt FROM users');
$row = $stmt->fetch(PDO::FETCH_ASSOC);
echo "User count: " . $row['cnt'] . "\n";

// Step 3: Explicitly close the connection
$stmt = null;  // Destroy the statement object first
$pdo = null;   // Destroy the PDO object
echo "Connection closed.\n";
```

**Expected Output:**

```
User count: 42
Connection closed.
```

**Why:** The connection is non-persistent, so setting `$pdo = null` destroys the object and closes the database connection. The `$stmt` must be destroyed first because it holds a reference to the PDO object.

**Example 2: Persistent Connection**

```php
<?php
// Step 1: Create a persistent connection
$dsn = 'mysql:host=localhost;dbname=testdb;charset=utf8mb4';
$pdo = new PDO($dsn, 'webuser', 'secret123', [
    PDO::ATTR_PERSISTENT => true
]);

// Step 2: Use the connection
$stmt = $pdo->query('SELECT NOW() AS current_time');
$row = $stmt->fetch(PDO::FETCH_ASSOC);
echo "Current DB time: " . $row['current_time'] . "\n";

// Step 3: "Closing" the connection does not actually close it
$pdo = null;
echo "PDO object destroyed (connection cached for reuse).\n";
```

**Expected Output:**

```
Current DB time: 2025-01-15 10:30:45
PDO object destroyed (connection cached for reuse).
```

**Why:** Setting `$pdo = null` destroys the PHP object, but the underlying database connection remains in the driver's persistent connection cache. The next script using the same DSN and credentials will reuse this connection.

**Example 3: SQLite In-Memory Database**

```php
<?php
// Step 1: Connect to an in-memory SQLite database
$dsn = 'sqlite::memory:';
$pdo = new PDO($dsn);

// Step 2: Create a table and insert data
$pdo->exec('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)');
$pdo->exec("INSERT INTO users (name) VALUES ('Alice'), ('Bob')");

// Step 3: Query the data
$stmt = $pdo->query('SELECT * FROM users');
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
print_r($rows);
```

**Expected Output:**

```
Array
(
    [0] => Array
        (
            [id] => 1
            [name] => Alice
        )
    [1] => Array
        (
            [id] => 2
            [name] => Bob
        )
)
```

**Why:** The `:memory:` DSN creates a temporary database that exists only for the duration of the connection. It is ideal for testing and prototyping because no files are created and the database is automatically destroyed when the connection closes.

### Real-World Cases

**Case 1: High-Traffic Web Application with Persistent Connections**

A news website with 1 million daily visitors uses PHP-FPM with 50 worker processes. Each worker enables `PDO::ATTR_PERSISTENT => true` for its MySQL connection. Instead of opening and closing a connection for every request (which involves TCP handshake, authentication, and connection setup), each worker reuses its cached connection. This reduces connection overhead significantly. However, the site must ensure that `pm.max_children` (50) plus other connections does not exceed MySQL's `max_connections` setting.

**Case 2: Long-Running CLI Worker**

A queue worker processes jobs continuously. Using a non-persistent connection would open and close a connection for every job, which is extremely wasteful. Instead, the worker creates a persistent connection at startup and reuses it for all jobs. The connection is only closed when the worker shuts down.

---

## Core Concept 3: Error Handlers & Attributes

### Definitions

**Core Definition**

PDO attributes are configuration settings that control how the PDO object behaves, including how errors are reported and how rows are returned by default.

**Technical Definition**

PDO attributes are integer constants set via `PDO::setAttribute()` or passed in the options array of the `PDO` constructor. `PDO::ATTR_ERRMODE` controls error reporting behavior, and `PDO::ATTR_DEFAULT_FETCH_MODE` sets the default `fetch()` style for all statements created from the PDO object.

**Beginner-Friendly Explanation**

Attributes are like the settings on your TV remote. `ATTR_ERRMODE` is like choosing whether the TV beeps loudly (exceptions), shows a warning light (warnings), or stays silent (silent mode) when something goes wrong. `ATTR_DEFAULT_FETCH_MODE` is like choosing whether channels come in as numbers, names, or logos — it decides how your data arrives by default.

### Purposes

- To standardize error handling by throwing `PDOException` instead of silent failures or warnings.
- To set a consistent row-fetching style across all queries without specifying it every time.
- To reduce boilerplate code in query result handling.
- To enforce development best practices (exceptions in development, warnings in production logging).
- To improve application security by making errors visible during development.

### Syntax Rules and Structure

**Complete General Syntax: Setting Error Mode**

```php
// Via constructor options
$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
]);

// Via setAttribute()
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

**Component Breakdown:**

- `PDO::ATTR_ERRMODE` — The attribute constant for error reporting mode.
- `PDO::ERRMODE_EXCEPTION` — Throws a `PDOException` on error (recommended).
- `PDO::ERRMODE_WARNING` — Emits an `E_WARNING` message.
- `PDO::ERRMODE_SILENT` — Sets error codes silently (the default).

**Complete General Syntax: Setting Default Fetch Mode**

```php
// Via constructor options
$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
]);

// Via setAttribute()
$pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
```

**Component Breakdown:**

- `PDO::ATTR_DEFAULT_FETCH_MODE` — The attribute constant for default fetch mode.
- `PDO::FETCH_ASSOC` — Returns rows as associative arrays (column name => value).
- `PDO::FETCH_OBJ` — Returns rows as anonymous objects.
- `PDO::FETCH_NUM` — Returns rows as numerically indexed arrays.
- `PDO::FETCH_BOTH` — Returns both associative and numeric keys (default).

**Syntax Rules:**

- Attributes can be set at construction time (via the fourth argument) or later via `setAttribute()`.
- Attributes set at construction time take effect immediately.
- Some attributes are driver-specific and may not be supported by all drivers.
- `PDO::ATTR_ERRMODE` is a **PDO-level** attribute; it applies to all statements created from the PDO object.
- `PDO::ATTR_DEFAULT_FETCH_MODE` is also a **PDO-level** attribute but can be overridden per-statement via `PDOStatement::setFetchMode()`.

**Constraints and Limitations:**

- `PDO::ERRMODE_EXCEPTION` is recommended for development but may expose sensitive information in production if exception messages are displayed to users. Always set `display_errors = 0` in production.
- `PDO::ATTR_DEFAULT_FETCH_MODE` does not affect `PDOStatement::fetchAll()` in all drivers; some drivers may ignore the default for `fetchAll()`.
- Driver-specific attributes (e.g., `PDO::MYSQL_ATTR_USE_BUFFERED_QUERY`) are not portable across databases.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Setting Error Mode to Exception**

```php
<?php
// Step 1: Connect with ERRMODE_EXCEPTION
$dsn = 'mysql:host=localhost;dbname=testdb;charset=utf8mb4';
$pdo = new PDO($dsn, 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
]);

// Step 2: Intentionally trigger an error with an invalid table
try {
    $pdo->query('SELECT * FROM non_existent_table');
} catch (PDOException $e) {
    echo "Caught exception: " . $e->getMessage() . "\n";
    echo "Error code: " . $e->getCode() . "\n";
}
```

**Expected Output:**

```
Caught exception: SQLSTATE[42S02]: Base table or view not found: 1146 Table 'testdb.non_existent_table' doesn't exist
Error code: 42S02
```

**Why:** With `ERRMODE_EXCEPTION`, PDO throws a `PDOException` when the query fails. The exception contains the SQLSTATE code (42S02) and a descriptive message. This allows the application to handle errors gracefully instead of continuing silently.

**Example 2: Setting Default Fetch Mode to Associative**

```php
<?php
// Step 1: Connect with FETCH_ASSOC as default
$dsn = 'sqlite::memory:';
$pdo = new PDO($dsn, null, null, [
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
]);

// Step 2: Create and populate a table
$pdo->exec('CREATE TABLE products (id INTEGER, name TEXT, price REAL)');
$pdo->exec("INSERT INTO products VALUES (1, 'Widget', 9.99)");

// Step 3: Fetch using the default mode
$stmt = $pdo->query('SELECT * FROM products');
$row = $stmt->fetch();
print_r($row);
```

**Expected Output:**

```
Array
(
    [id] => 1
    [name] => Widget
    [price] => 9.99
)
```

**Why:** The `PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC` setting makes `fetch()` return an associative array by default. Without this setting, the default mode is `PDO::FETCH_BOTH`, which would return both numeric and string keys.

**Example 3: Combining Error Mode and Fetch Mode**

```php
<?php
// Step 1: Connect with both attributes set
$dsn = 'pgsql:host=127.0.0.1;port=5432;dbname=appdb';
$pdo = new PDO($dsn, 'pguser', 'pgpass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_OBJ,
]);

// Step 2: Query data
$stmt = $pdo->query('SELECT 1 AS id, \'Test\' AS name');
$row = $stmt->fetch();
echo $row->id . ' - ' . $row->name . "\n";

// Step 3: Trigger an error
try {
    $pdo->query('SELECT * FROM missing_table');
} catch (PDOException $e) {
    echo "Error: " . $e->getCode() . "\n";
}
```

**Expected Output:**

```
1 - Test
Error: 42P01
```

**Why:** The first query returns an object with properties `id` and `name`. The second query fails because the table does not exist, and the exception is caught and its SQLSTATE code (42P01) is printed.

### Real-World Cases

**Case 1: Development vs. Production Error Modes**

In development, `PDO::ERRMODE_EXCEPTION` is used so developers see full error details immediately. In production, the same mode is kept, but a global exception handler logs the error and displays a generic user-friendly message, while `display_errors` is disabled in `php.ini` to prevent leaking connection details.

**Case 2: API Data Fetching with Objects**

A REST API that returns JSON responses uses `PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_OBJ`. Each row is an object, which is then passed to `json_encode()` directly. This reduces array-to-object conversion code and keeps the response structure consistent.

---

## Core Concept 4: Connection Pooling & Read/Write Splits

### Definitions

**Core Definition**

Connection pooling reuses a set of database connections across multiple requests, while read/write splitting routes read queries to replica databases and write queries to the primary database.

**Technical Definition**

In traditional PHP-FPM, PDO connections are bound to the lifecycle of a single request, making true connection pooling impossible without persistent connections or external poolers. In long-running runtimes like Swoole, RoadRunner, and FrankenPHP, workers persist across requests, enabling in-memory connection pools. Read/write splitting routes `SELECT` queries to replicas and `INSERT`/`UPDATE`/`DELETE` queries to the primary.

**Beginner-Friendly Explanation**

Imagine a restaurant with a limited number of tables. **Connection pooling** is like keeping tables ready for the next customers instead of setting them up from scratch each time. **Read/write splitting** is like having one kitchen for cooking (writing) and several smaller stations for assembling salads (reading) — orders are routed to the right place.

### Purposes

- To eliminate the overhead of establishing a new database connection for every request.
- To distribute read load across multiple replica databases, improving scalability.
- To reduce the load on the primary database by offloading read queries.
- To enable automatic failover when a replica becomes unhealthy.
- To provide sticky-after-write behavior (reads route to primary after a write to avoid replication lag).
- To support high-concurrency environments with coroutine-safe connection management.

### Syntax Rules and Structure

**Complete General Syntax: Swoole PDO Connection Pool**

```php
use Swoole\Database\PDOConfig;
use Swoole\Database\PDOPool;

// Step 1: Configure the pool
$config = (new PDOConfig())
    ->withHost('127.0.0.1')
    ->withPort(3306)
    ->withDbName('testdb')
    ->withUsername('webuser')
    ->withPassword('secret123')
    ->withCharset('utf8mb4');

// Step 2: Create the pool
$pool = new PDOPool($config, 10);  // 10 connections

// Step 3: Borrow a connection
$pdo = $pool->get();

// Step 4: Use the connection
$stmt = $pdo->query('SELECT * FROM users');
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

// Step 5: Return the connection to the pool
$pool->put($pdo);
```

**Component Breakdown:**

- `PDOConfig` — Builder class for configuring host, port, database, username, password, and charset.
- `PDOPool` — The connection pool class. The second constructor argument is the maximum pool size.
- `$pool->get()` — Borrows a connection from the pool. If all connections are in use, it may block or create a new one up to the maximum.
- `$pool->put($pdo)` — Returns the connection to the pool for reuse.

**Complete General Syntax: Read/Write Splitting (Conceptual)**

```php
class ReadWriteConnectionManager
{
    private PDO $writer;
    private array $readers;
    private int $readerIndex = 0;

    public function __construct(
        PDO $writer,
        array $readers
    ) {
        $this->writer = $writer;
        $this->readers = $readers;
    }

    public function getWriter(): PDO
    {
        return $this->writer;
    }

    public function getReader(): PDO
    {
        // Round-robin load balancing
        $reader = $this->readers[$this->readerIndex];
        $this->readerIndex = ($this->readerIndex + 1) % count($this->readers);
        return $reader;
    }

    public function query(string $sql, array $params = []): PDOStatement
    {
        // Route SELECT to reader, everything else to writer
        $isRead = stripos(trim($sql), 'SELECT') === 0;
        $pdo = $isRead ? $this->getReader() : $this->getWriter();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt;
    }
}
```

**Component Breakdown:**

- `$writer` — A `PDO` instance connected to the primary (write) database.
- `$readers` — An array of `PDO` instances connected to replica (read) databases.
- `getReader()` — Returns a reader using round-robin load balancing.
- `query()` — Routes `SELECT` queries to a reader and all other queries to the writer.

**Complete General Syntax: RoadRunner / FrankenPHP Worker Mode**

```php
// In worker mode, the connection is created once and reused
// across all requests handled by that worker.

// Worker bootstrap:
$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_PERSISTENT => true,  // Optional; framework pool often preferred
]);

// Request handler (executed many times per worker):
function handleRequest($pdo): string
{
    $stmt = $pdo->query('SELECT * FROM products');
    $products = $stmt->fetchAll(PDO::FETCH_ASSOC);
    return json_encode($products);
}

// Worker shutdown (NOT per request):
// $pdo = null;  // Only on worker termination
```

**Component Breakdown:**

- The `PDO` instance is created once when the worker starts.
- It is reused across all HTTP requests handled by that worker.
- `disconnect()` should **not** be called between requests — only on worker shutdown.

**Syntax Rules:**

- In Swoole, each coroutine must have its own connection. A single PDO instance cannot be shared across concurrent coroutines because each query needs its own I/O state.
- In RoadRunner and FrankenPHP, one PDO instance per worker is reused across requests. Do not call `disconnect()` between requests.
- In traditional PHP-FPM, true connection pooling is not possible because PDO connections are bound to a single request. Persistent connections can reduce overhead but are not a true pool.

**Constraints and Limitations:**

- **Swoole:** Do not use `PDO::ATTR_PERSISTENT => true`. Persistent connections under Swoole do not release automatically and can accumulate dead connections.
- **RoadRunner/FrankenPHP:** Global state (including database connections) persists across requests. Any transaction left open by a request will affect subsequent requests.
- **Read/Write Splitting:** Replication lag means that reads immediately after a write may return stale data. Implement "sticky-after-write" behavior to route reads to the primary for a short time after a write.
- **Connection Pool Exhaustion:** If all connections in a pool are checked out, `borrow()` will block until one is returned. A pool that is exhausted is a queue, not an outage.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Swoole PDO Connection Pool**

```php
<?php
use Swoole\Database\PDOConfig;
use Swoole\Database\PDOPool;
use Swoole\Http\Server;

// Step 1: Create the pool before starting the server
$config = (new PDOConfig())
    ->withHost('127.0.0.1')
    ->withPort(3306)
    ->withDbName('testdb')
    ->withUsername('webuser')
    ->withPassword('secret123')
    ->withCharset('utf8mb4');

$pool = new PDOPool($config, 10);  // Pool of 10 connections

// Step 2: Create HTTP server
$server = new Server('0.0.0.0', 9501);

// Step 3: Handle requests
$server->on('request', function ($request, $response) use ($pool) {
    // Borrow a connection from the pool
    $pdo = $pool->get();

    try {
        // Step 4: Execute query
        $stmt = $pdo->query('SELECT COUNT(*) AS cnt FROM users');
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        $response->header('Content-Type', 'application/json');
        $response->end(json_encode(['user_count' => $row['cnt']]));
    } finally {
        // Step 5: Always return the connection to the pool
        $pool->put($pdo);
    }
});

// Step 6: Start the server
$server->start();
```

**Expected Output (when accessed via HTTP):**

```
{"user_count":42}
```

**Why:** The pool is created once before the server starts. Each request borrows a connection, executes the query, and returns the connection in a `finally` block. Even if the query fails, the connection is returned, preventing pool leaks.

**Example 2: RoadRunner Worker with Reused PDO**

```php
<?php
// RoadRunner worker script

// Step 1: Bootstrap — create the PDO connection once
$dsn = 'mysql:host=127.0.0.1;dbname=testdb;charset=utf8mb4';
$pdo = new PDO($dsn, 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

echo "Worker booted. PDO connection established.\n";

// Step 2: This function is called for every HTTP request
function handleRequest(PDO $pdo, array $request): array
{
    $stmt = $pdo->query('SELECT id, name FROM products LIMIT 5');
    return $stmt->fetchAll();
}

// Step 3: Simulate multiple requests (in production, RoadRunner calls this)
for ($i = 1; $i <= 3; $i++) {
    $products = handleRequest($pdo, ['uri' => '/products']);
    echo "Request $i: " . count($products) . " products fetched.\n";
}

// Step 4: On worker shutdown (NOT per request), close the connection
$pdo = null;
echo "Worker shutting down. PDO connection closed.\n";
```

**Expected Output:**

```
Worker booted. PDO connection established.
Request 1: 5 products fetched.
Request 2: 5 products fetched.
Request 3: 5 products fetched.
Worker shutting down. PDO connection closed.
```

**Why:** The PDO connection is created once at worker boot and reused across all three simulated requests. It is only destroyed at worker shutdown. This is the correct pattern for RoadRunner and FrankenPHP worker mode.

**Example 3: Read/Write Splitting with Sticky-After-Write**

```php
<?php
class ReadWriteManager
{
    private PDO $writer;
    private array $readers;
    private int $readerIndex = 0;
    private ?float $lastWriteTime = null;
    private float $stickyDuration = 2.0; // seconds

    public function __construct(PDO $writer, array $readers)
    {
        $this->writer = $writer;
        $this->readers = $readers;
    }

    private function getReader(): PDO
    {
        // Sticky-after-write: use writer if a write happened recently
        if ($this->lastWriteTime !== null
            && (microtime(true) - $this->lastWriteTime) < $this->stickyDuration) {
            return $this->writer;
        }
        $reader = $this->readers[$this->readerIndex];
        $this->readerIndex = ($this->readerIndex + 1) % count($this->readers);
        return $reader;
    }

    public function query(string $sql, array $params = []): PDOStatement
    {
        $isRead = stripos(trim($sql), 'SELECT') === 0;
        if (!$isRead) {
            $this->lastWriteTime = microtime(true);
        }
        $pdo = $isRead ? $this->getReader() : $this->writer;
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt;
    }
}

// Usage
$writer = new PDO('mysql:host=primary;dbname=shop', 'user', 'pass');
$readers = [
    new PDO('mysql:host=replica1;dbname=shop', 'user', 'pass'),
    new PDO('mysql:host=replica2;dbname=shop', 'user', 'pass'),
];

$manager = new ReadWriteManager($writer, $readers);

// Write operation — routes to primary
$manager->query("INSERT INTO orders (total) VALUES (99.99)");

// Read operation immediately after write — routes to primary (sticky)
$stmt = $manager->query("SELECT * FROM orders ORDER BY id DESC LIMIT 1");
print_r($stmt->fetch(PDO::FETCH_ASSOC));

// After 2 seconds, reads route to replicas again
```

**Expected Output:**

```
Array
(
    [id] => 101
    [total] => 99.99
)
```

**Why:** The `lastWriteTime` is set when a non-`SELECT` query is executed. For 2 seconds afterward, all reads are routed to the writer (sticky-after-write), ensuring the application sees its own writes despite replication lag. After the sticky period, reads are distributed across replicas.

### Real-World Cases

**Case 1: High-Concurrency API with Swoole**

A real-time bidding platform uses Swoole HTTP server with a PDO connection pool of 50 connections per worker. Each incoming bid request borrows a connection, executes a `SELECT` or `UPDATE`, and returns the connection. The pool ensures that no more than 50 database connections are open per worker, preventing connection exhaustion. The coroutine-safe pool suspends waiting coroutines instead of busy-waiting when all connections are checked out.

**Case 2: E-Commerce Platform with Read Replicas**

An e-commerce site uses MySQL with one primary and three read replicas. The application routes all `SELECT` queries (product listings, search) to replicas and all `INSERT`/`UPDATE`/`DELETE` queries (orders, inventory) to the primary. After a customer places an order, the application uses sticky-after-write for 2 seconds so the customer sees their order confirmation immediately, even if replica lag would otherwise show stale data.

**Case 3: RoadRunner Worker with Long-Lived Connections**

A microservices API built on RoadRunner uses one PDO connection per worker. The worker handles thousands of requests per minute without reconnecting. The connection is only closed when the worker is gracefully shut down. This pattern reduces database connection churn and improves throughput compared to PHP-FPM.

---

## References

- PHP: PDO — Introduction – https://www.php.net/manual/en/intro.pdo.php
- PHP: PDO Connections and Connection Management – https://www.php.net/manual/en/pdo.connections.php
- PHP: PDO::setAttribute – https://www.php.net/manual/en/pdo.setattribute.php
- PHP: PDO::ATTR_ERRMODE – https://www.php.net/manual/en/pdo.constants.php
- PHP: PDO::ATTR_DEFAULT_FETCH_MODE – https://www.php.net/manual/en/pdo.constants.php
- PHP: PDO Drivers – https://www.php.net/manual/en/pdo.drivers.php
- Microsoft Docs: PDO::setAttribute (SQL Server) – https://learn.microsoft.com/en-us/sql/connect/php/pdo-setattribute
- Swoole Documentation: PDO Connection Pool – https://openswoole.com/docs/modules/swoole-database-pdo-pool
- RoadRunner Documentation – https://roadrunner.dev/docs
- FrankenPHP Documentation – https://frankenphp.dev/docs
- GeeksforGeeks: Disadvantages of Persistent Connections in PDO – https://www.geeksforgeeks.org/what-are-the-disadvantages-of-using-persistent-connection-in-pdo/
- Drupal: Database API General Concepts – https://www.drupal.org/docs/develop/drupal-apis/database-api/general-concepts
- PHP: PDO::ATTR_PERSISTENT – https://www.php.net/manual/en/pdo.constants.php
- PHP: PDO::ERRMODE_EXCEPTION – https://www.php.net/manual/en/pdo.constants.php
- PHP: PDO::FETCH_ASSOC – https://www.php.net/manual/en/pdo.constants.php
- PHP: PDO::FETCH_OBJ – https://www.php.net/manual/en/pdo.constants.php