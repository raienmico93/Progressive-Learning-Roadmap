# Secure Query Execution & Prepared Statements — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Secure query execution with prepared statements is a database access methodology in which SQL query structure is defined separately from the data values supplied to it, preventing user input from being interpreted as executable SQL code.

**Technical Definition**

A prepared statement is a precompiled SQL template submitted to the database server via `PDO::prepare()`. It contains placeholders (named `:name` or positional `?`) instead of literal values. The database parses, compiles, and optimises the query plan once. Values are then transmitted separately via `execute()` or explicit binding methods (`bindParam()`, `bindValue()`), ensuring the server treats them strictly as data — never as SQL syntax.

**Beginner-Friendly Explanation**

Imagine filling in a form where the printed questions are fixed, and you only write your answers in the blank spaces. A malicious user cannot change the questions by writing clever answers, because the form's structure was already decided before they arrived. Prepared statements work the same way: the SQL query skeleton is fixed, and user input fills only the designated blanks.

### Key Characteristics

- **Structure–Data Separation:** SQL code and data travel to the database through separate channels.
- **Injection Immunity (by design):** Bound parameters cannot alter query structure, regardless of content.
- **Server-Side vs. Client-Side Preparation:** Controlled by `PDO::ATTR_EMULATE_PREPARES`. When `false`, the database server performs true preparation; when `true` (PDO's historical default), PDO interpolates values client-side.
- **Reusability:** A prepared statement can be executed multiple times with different parameter values without re-preparation.
- **Type Awareness:** Parameters can be bound with explicit types (`PDO::PARAM_INT`, `PDO::PARAM_STR`, `PDO::PARAM_LOB`, etc.).
- **Memory Efficiency Options:** Buffered and unbuffered query modes allow trade-offs between PHP memory usage and database server load.

### Prerequisites

- **PHP with PDO and a driver extension** (`pdo_mysql`, `pdo_pgsql`, `pdo_sqlite`, etc.) enabled.
- **A working database connection** established via a valid DSN.
- **Basic SQL knowledge** — prepared statements protect values, not identifiers (table names, column names, sort directions).
- **Understanding of SQLSTATE error codes** for effective error handling.

### Related Programming Areas

- **Web Application Security:** Prepared statements are the primary defence against SQL injection (SQLi), consistently ranked in the OWASP Top 10.
- **Database Performance Tuning:** Prepared statement reuse reduces parsing overhead; unbuffered queries reduce PHP memory pressure.
- **Object-Relational Mapping (ORM):** ORMs like Doctrine and Eloquent ultimately issue prepared statements via PDO or similar layers.
- **Long-Running PHP Runtimes:** Swoole, RoadRunner, and FrankenPHP require special handling of prepared statements and cursors across requests.

### Core Concepts / Features

1. **SQL Injection Mitigation** — Why raw concatenation is vulnerable and how prepared statements eliminate the attack surface.
2. **Parameter Binding** — Named vs. positional placeholders, `bindParam()` vs. `bindValue()`, and type specification.
3. **Execution & Fetching Strategies** — `execute()`, `fetch()`, `fetchAll()`, `fetchColumn()`, and `PDO::FETCH_CLASS`.
4. **Cursor & Stream Management** — Buffered vs. unbuffered queries and preventing PHP memory exhaustion.

---

## Core Concept 1: SQL Injection Mitigation

### Definitions

**Core Definition**

SQL injection is a code-injection vulnerability that occurs when untrusted data is concatenated into an SQL query string, allowing an attacker to alter the query's structure and execute arbitrary SQL commands.

**Technical Definition**

SQLi arises when the boundary between SQL code and data is not enforced. In a vulnerable query such as `SELECT * FROM users WHERE username = '$input'`, an attacker supplying `admin' --` changes the effective query to `SELECT * FROM users WHERE username = 'admin'`, bypassing the password check entirely. Escaping functions like `mysqli_real_escape_string()` reduce risk but do not eliminate it, particularly with multibyte encodings or misconfigured connection charsets.

**Beginner-Friendly Explanation**

Think of a SQL query as a sentence in a language the database understands. If you let a user insert their own words directly into the middle of that sentence, they can change its meaning entirely. Prepared statements are like a fill-in-the-blank exercise: the sentence structure is locked, and the user can only write in the blanks.

### Purposes

- To prevent attackers from altering SQL query structure through crafted input.
- To eliminate the need for manual escaping, which is error-prone and encoding-dependent.
- To enforce a strict separation between code and data at the database protocol level.
- To provide a uniform defence across all supported database drivers.
- To reduce the attack surface of applications that handle untrusted input from forms, APIs, or URL parameters.

### Syntax Rules and Structure

**Vulnerable Pattern (Never Use)**

```php
// DANGEROUS — DO NOT USE
$username = $_POST['username'];
$query = "SELECT * FROM users WHERE username = '$username'";
$result = $pdo->query($query);
```

**Secure Pattern with Prepared Statements**

```php
// SAFE — Prepared statement with named placeholder
$stmt = $pdo->prepare('SELECT * FROM users WHERE username = :username');
$stmt->execute([':username' => $_POST['username']]);
```

**Component Breakdown (Secure Pattern):**

- `$pdo->prepare(...)` — Submits the SQL template to the database (or PDO's emulation layer) for parsing and compilation. The placeholder `:username` marks where a value will later be supplied.
- `$stmt` — A `PDOStatement` object representing the prepared query.
- `$stmt->execute([...])` — Sends the parameter values. The array key (`:username`) matches the placeholder; the value (`$_POST['username']`) is transmitted as data, never concatenated into SQL text.

**Syntax Rules:**

- Placeholders cannot be used for identifiers (table names, column names, `ORDER BY` direction). These must be validated against an allow-list.
- `prepare()` does not sanitise input by itself — it is only safe when all variable values are passed as bound parameters, not concatenated.
- `PDO::ATTR_EMULATE_PREPARES` should be set to `false` to force real server-side preparation. With emulation on, PDO interpolates client-side, and multibyte charset edge cases become reachable.

**Constraints and Limitations:**

- **Prepared statements do not protect identifiers.** An application that builds `ORDER BY $column` from user input remains injectable even with prepared statements.
- **LIKE clauses** require careful handling: `WHERE name LIKE :pattern` with `$pattern = '%' . $search . '%'` is safe, but the wildcards must be part of the bound value, not the SQL string.
- **IN clauses** cannot use a single placeholder for multiple values. Each value needs its own placeholder, or the list must be constructed with repeated placeholders.
- **Emulated prepares** (the historical default) may still be vulnerable to edge-case charset attacks on older PHP versions. Setting `PDO::ATTR_EMULATE_PREPARES => false` is the modern best practice.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Vulnerable vs. Secure Login**

```php
<?php
// Assume $pdo is a valid PDO connection with ERRMODE_EXCEPTION

// --- VULNERABLE VERSION (do not use in production) ---
$username = $_POST['username'];   // Attacker supplies: admin' --
$password = $_POST['password'];   // Attacker supplies: anything
$vulnerableSql = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
// Effective query after injection:
// SELECT * FROM users WHERE username = 'admin' --' AND password = 'anything'
// The -- comments out the password check. Authentication bypassed.

// --- SECURE VERSION ---
$stmt = $pdo->prepare(
    'SELECT id, username FROM users WHERE username = :username AND password_hash = :password'
);
$stmt->execute([
    ':username' => $_POST['username'],
    ':password' => hash('sha256', $_POST['password']), // illustrative only; use password_hash()
]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);

if ($user) {
    echo "Login successful for user: " . $user['username'] . "\n";
} else {
    echo "Invalid credentials.\n";
}
```

**Expected Output (with valid credentials):**

```
Login successful for user: admin
```

**Expected Output (with injection attempt `admin' --`):**

```
Invalid credentials.
```

**Why:** In the vulnerable version, the `--` sequence terminates the SQL statement, removing the password condition. In the secure version, `:username` and `:password` are transmitted as discrete data values. Even if the attacker supplies `admin' --`, the database treats the entire string as a username literal — it never becomes SQL syntax.

**Example 2: Allow-List Validation for Identifiers**

```php
<?php
// Column name and sort direction CANNOT be parameterised.
// Use an allow-list instead.

$allowedColumns = ['id', 'name', 'created_at'];
$allowedDirections = ['ASC', 'DESC'];

$column = $_GET['sort'] ?? 'id';
$direction = strtoupper($_GET['dir'] ?? 'ASC');

// Validate against allow-lists
if (!in_array($column, $allowedColumns, true)) {
    $column = 'id'; // safe fallback
}
if (!in_array($direction, $allowedDirections, true)) {
    $direction = 'ASC';
}

// Now concatenate the VALIDATED identifier
$sql = "SELECT id, name, created_at FROM products ORDER BY {$column} {$direction}";
$stmt = $pdo->query($sql);
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

foreach ($rows as $row) {
    echo $row['name'] . "\n";
}
```

**Expected Output (with `?sort=name&dir=DESC`):**

```
Zebra Widget
Apple Widget
... (rows sorted by name descending)
```

**Why:** The identifier (`$column`) and direction (`$direction`) are resolved through strict allow-lists before concatenation. An attacker cannot inject arbitrary SQL through the sort parameter because only pre-approved values are accepted. The values in the `WHERE` clause (if any) would still be bound via placeholders.

**Example 3: Setting `ATTR_EMULATE_PREPARES => false`**

```php
<?php
$dsn = 'mysql:host=localhost;dbname=testdb;charset=utf8mb4';

$pdo = new PDO($dsn, 'webuser', 'secret123', [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_EMULATE_PREPARES   => false,  // Force real server-side preparation
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

// This prepared statement is now compiled by MySQL itself,
// not emulated by PDO on the client side.
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute([':email' => $email]);
```

**Expected Output:** No visible output, but the query is executed with true server-side preparation. If the MySQL server does not support native preparation for a particular statement type, an exception is thrown.

**Why:** With `ATTR_EMULATE_PREPARES => false`, PDO delegates preparation to the database driver. This provides stronger guarantees against charset-based bypasses and allows the database to optimise the query plan server-side. It is the recommended setting for production security.

### Real-World Cases

**Case 1: Content Management System Login Bypass**

A CMS used string concatenation in its authentication query. An attacker submitted `' OR '1'='1` as the username, causing the query to return all users and granting administrative access. After migrating to PDO prepared statements, the same input was treated as a literal username string, and authentication failed as expected.

**Case 2: E-Commerce Search with LIKE**

An e-commerce site allowed users to search products by name. The vulnerable implementation used `WHERE name LIKE '%$search%'`. An attacker injected `%' UNION SELECT credit_card FROM payments --` to exfiltrate payment data. The secure implementation binds the search term as a parameter: `WHERE name LIKE :pattern` with `$pattern = '%' . $search . '%'`. The wildcards are part of the bound value, not the SQL string.

---

## Core Concept 2: Parameter Binding

### Definitions

**Core Definition**

Parameter binding is the process of associating PHP variables or values with the placeholders in a prepared SQL statement.

**Technical Definition**

PDO supports two placeholder syntaxes: named placeholders (`:name`) and positional placeholders (`?`). Values can be bound either implicitly via the array passed to `PDOStatement::execute()`, or explicitly via `PDOStatement::bindParam()` (which binds a PHP variable by reference) or `PDOStatement::bindValue()` (which binds a value or constant).

**Beginner-Friendly Explanation**

Think of placeholders as empty cups in a vending machine. Named placeholders are cups with labels ("cola", "water"), while positional placeholders are unlabelled cups arranged in a row. You can either hand the machine the cups in the right order (`execute()`), or you can attach a reusable hose to a labelled cup (`bindParam()`) so that whatever liquid flows through the hose at dispensing time fills the cup.

### Purposes

- To supply values to a prepared statement without concatenating them into SQL text.
- To enforce data type consistency across different database drivers.
- To enable statement reuse with different parameter values without re-preparing.
- To support output parameters and stored procedures (driver-dependent).
- To provide flexibility in binding variables by reference (`bindParam`) or by value (`bindValue`).

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

**`bindValue()` — Bind a Value**

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->bindValue(':id', 42, PDO::PARAM_INT);
$stmt->execute();
```

**`bindParam()` — Bind a Variable by Reference**

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$id = 42;
$stmt->bindParam(':id', $id, PDO::PARAM_INT);
$id = 99;  // This change WILL affect the executed query
$stmt->execute();
```

**Component Breakdown:**

- `$parameter` (mixed) — For named placeholders, the placeholder name as a string (e.g., `':id'`). For positional placeholders, the 1-indexed position as an integer (e.g., `1`).
- `$variable` / `$value` — The PHP variable or value to bind. `bindParam()` takes it by reference (`&$variable`); `bindValue()` takes it by value.
- `$data_type` (optional) — The PDO type constant: `PDO::PARAM_INT`, `PDO::PARAM_STR`, `PDO::PARAM_LOB`, `PDO::PARAM_BOOL`, `PDO::PARAM_NULL`.
- `$length` (optional, `bindParam` only) — The maximum length for output parameters in stored procedures.
- `$driver_options` (optional, `bindParam` only) — Driver-specific options.

**Key Differences Between `bindParam()` and `bindValue()`:**

| Aspect | `bindParam()` | `bindValue()` |
|---|---|---|
| Binding mechanism | By reference (`&$variable`) | By value |
| Effect of later variable changes | Changes reflected in execution | No effect |
| When to use | Output parameters, variables that change between executions | Most common case; constants, expressions |
| Return value | `bool` | `bool` |

A common recommendation is to use `bindValue()` in place of `bindParam()` for most use cases, because `bindValue()` accepts both variables and literal values, and avoids subtle bugs caused by late variable mutation.

**Syntax Rules:**

- Named placeholders must be unique within a statement. The same name cannot be used twice.
- Positional placeholders are 1-indexed when used with `bindParam()` and `bindValue()`, but 0-indexed in the array passed to `execute()`.
- The number of bound parameters must exactly match the number of placeholders.
- `PDO::PARAM_STR` is the default type when no type is specified.

**Constraints and Limitations:**

- **Named placeholder limitation:** The same named placeholder cannot appear more than once in a statement. PDO internally replaces named placeholders with positional notation, causing a mismatch if a name is repeated. Workaround: use distinct names (e.g., `:gameid` and `:gameid2`).
- **`bindParam()` and `execute()` arrays are mutually exclusive for the same parameter.** If you bind a parameter explicitly, do not also supply it in the `execute()` array.
- **`IN()` clauses:** You cannot bind an array to a single placeholder. Each value in the `IN` list requires its own placeholder.
- **Type binding is driver-dependent.** Some drivers ignore the specified type and infer it from the PHP value.

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
$namedResults = $stmt->fetchAll(PDO::FETCH_ASSOC);
echo "Named placeholder results:\n";
print_r($namedResults);

// --- Positional placeholders ---
$stmt2 = $pdo->prepare('SELECT * FROM users WHERE age > ? AND name != ?');
$stmt2->execute([26, 'Alice']);
$positionalResults = $stmt2->fetchAll(PDO::FETCH_ASSOC);
echo "\nPositional placeholder results:\n";
print_r($positionalResults);
```

**Expected Output:**

```
Named placeholder results:
Array
(
    [0] => Array
        (
            [id] => 3
            [name] => Carol
            [age] => 35
        )
)

Positional placeholder results:
Array
(
    [0] => Array
        (
            [id] => 3
            [name] => Carol
            [age] => 35
        )
)
```

**Why:** Both queries produce identical results. The named-placeholder version uses `:min_age` and `:exclude_name`, while the positional version uses `?` placeholders supplied in order. Named placeholders are generally more readable when a statement has many parameters.

**Example 2: `bindValue()` vs. `bindParam()`**

```php
<?php
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE products (id INTEGER PRIMARY KEY, name TEXT)');
$pdo->exec("INSERT INTO products (name) VALUES ('Widget'), ('Gadget'), ('Gizmo')");

// --- bindValue(): value is captured at binding time ---
$stmt = $pdo->prepare('SELECT * FROM products WHERE id = :id');
$productId = 1;
$stmt->bindValue(':id', $productId, PDO::PARAM_INT);
$productId = 2;  // This change is IGNORED
$stmt->execute();
$result1 = $stmt->fetch(PDO::FETCH_ASSOC);
echo "bindValue result (id=1 expected): " . $result1['name'] . "\n";

// --- bindParam(): variable is bound by reference ---
$stmt2 = $pdo->prepare('SELECT * FROM products WHERE id = :id');
$productId2 = 1;
$stmt2->bindParam(':id', $productId2, PDO::PARAM_INT);
$productId2 = 2;  // This change IS reflected
$stmt2->execute();
$result2 = $stmt2->fetch(PDO::FETCH_ASSOC);
echo "bindParam result (id=2 expected): " . $result2['name'] . "\n";
```

**Expected Output:**

```
bindValue result (id=1 expected): Widget
bindParam result (id=2 expected): Gadget
```

**Why:** `bindValue()` captures the value of `$productId` at the moment of binding (1). Later reassigning `$productId = 2` has no effect. `bindParam()` binds the variable itself by reference. When `$productId2` is changed to 2 before `execute()`, the executed query uses the new value.

**Example 3: Binding with Explicit Types**

```php
<?php
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE metrics (id INTEGER PRIMARY KEY, value REAL, label TEXT, active INTEGER)');
$pdo->exec("INSERT INTO metrics (value, label, active) VALUES (3.14, 'pi', 1)");

$stmt = $pdo->prepare('INSERT INTO metrics (value, label, active) VALUES (:value, :label, :active)');

// Bind with explicit types
$stmt->bindValue(':value', 2.718, PDO::PARAM_STR);   // bound as string
$stmt->bindValue(':label', 'e', PDO::PARAM_STR);
$stmt->bindValue(':active', true, PDO::PARAM_BOOL);
$stmt->execute();

// Verify
$row = $pdo->query('SELECT * FROM metrics WHERE label = \'e\'')->fetch(PDO::FETCH_ASSOC);
print_r($row);
```

**Expected Output:**

```
Array
(
    [id] => 2
    [value] => 2.718
    [label] => e
    [active] => 1
)
```

**Why:** Each parameter is bound with an explicit PDO type constant. SQLite stores the boolean `true` as `1` and the float as a string representation. Explicit typing ensures consistent behaviour across drivers, particularly for numeric and boolean values.

### Real-World Cases

**Case 1: Batch Insert with `bindParam()`**

A data-import script inserts thousands of rows. Using `bindParam()` allows the statement to be prepared once and executed repeatedly with different variable values:

```php
$stmt = $pdo->prepare('INSERT INTO logs (level, message) VALUES (:level, :message)');
$stmt->bindParam(':level', $level, PDO::PARAM_STR);
$stmt->bindParam(':message', $message, PDO::PARAM_STR);

foreach ($logEntries as $entry) {
    $level = $entry['level'];
    $message = $entry['message'];
    $stmt->execute();
}
```

This avoids re-preparing the statement for every row and leverages the by-reference binding to update values between executions.

**Case 2: Search Form with Dynamic Filters**

A product search form allows filtering by category, price range, and availability. Named placeholders make the dynamic query readable:

```php
$sql = 'SELECT * FROM products WHERE 1=1';
$params = [];

if (!empty($category)) {
    $sql .= ' AND category = :category';
    $params[':category'] = $category;
}
if (!empty($minPrice)) {
    $sql .= ' AND price >= :min_price';
    $params[':min_price'] = $minPrice;
}
if (!empty($maxPrice)) {
    $sql .= ' AND price <= :max_price';
    $params[':max_price'] = $maxPrice;
}

$stmt = $pdo->prepare($sql);
$stmt->execute($params);
```

Named placeholders allow the query structure to grow dynamically while keeping parameter binding explicit and safe.

---

## Core Concept 3: Execution & Fetching Strategies

### Definitions

**Core Definition**

Execution and fetching strategies define how a prepared statement is run and how result rows are extracted into PHP data structures.

**Technical Definition**

`PDOStatement::execute()` runs a prepared statement with optional bound parameters. Result rows are retrieved via `fetch()` (one row at a time), `fetchAll()` (all remaining rows into an array), or `fetchColumn()` (a single column from the next row). Fetch modes control the shape of returned data: associative arrays (`PDO::FETCH_ASSOC`), objects (`PDO::FETCH_OBJ`), custom class instances (`PDO::FETCH_CLASS`), and others.

**Beginner-Friendly Explanation**

Executing a query is like placing an order at a restaurant. Fetching is how the food arrives: `fetch()` brings one plate at a time, `fetchAll()` brings everything on a tray at once, and `fetchColumn()` brings just one item from the next plate. `FETCH_CLASS` is like asking the waiter to serve the food as a specific type of dish (a custom object) rather than on a generic plate.

### Purposes

- To run a prepared statement with specific parameter values.
- To retrieve query results in a format suited to the application's data model.
- To map database rows directly into typed PHP objects, reducing manual array-to-object conversion.
- To extract a single scalar value efficiently without loading an entire row.
- To support iteration over large result sets one row at a time to conserve memory.

### Syntax Rules and Structure

**`execute()` — Run a Prepared Statement**

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE age > :min');
$stmt->execute([':min' => 18]);
```

**`fetch()` — Retrieve One Row**

```php
$row = $stmt->fetch(PDO::FETCH_ASSOC);
```

**`fetchAll()` — Retrieve All Remaining Rows**

```php
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

**`fetchColumn()` — Retrieve a Single Column**

```php
$value = $stmt->fetchColumn(0);  // 0-indexed column number
```

**`PDO::FETCH_CLASS` — Map Rows to Typed Objects**

```php
$users = $stmt->fetchAll(PDO::FETCH_CLASS, User::class);
```

**Component Breakdown:**

- `execute(array $params = [])` — An optional associative array of parameter values (for named placeholders) or a sequential array (for positional placeholders). Returns `bool`.
- `fetch(int $mode = PDO::FETCH_DEFAULT)` — Returns the next row in the specified mode, or `false` if no more rows.
- `fetchAll(int $mode = PDO::FETCH_DEFAULT)` — Returns an array of all remaining rows. Can also accept a column index or class name depending on the mode.
- `fetchColumn(int $column = 0)` — Returns a single value from the specified column (0-indexed) of the next row, or `false` if no more rows. **Do not use this method to retrieve boolean columns**, because `false` is indistinguishable from "no more rows".
- `PDO::FETCH_CLASS` — Returns instances of the specified class. Column values are mapped to properties with matching names. If a property does not exist on the class, it is created dynamically (unless `PDO::FETCH_PROPS_LATE` or `__set()` behaviour applies).

**Syntax Rules:**

- `fetchAll()` loads the entire remaining result set into memory. For large data sets, use `fetch()` in a loop instead.
- `fetchColumn()` advances the internal row pointer, so subsequent calls retrieve the next row's column.
- When using `PDO::FETCH_CLASS`, the class constructor is called **after** properties are set by default. Use `PDO::FETCH_PROPS_LATE` to call the constructor first.
- `fetchAll(PDO::FETCH_COLUMN, $index)` returns a flat array of all values from the specified column.

**Constraints and Limitations:**

- **`fetchColumn()` cannot retrieve multiple columns from the same row.** Once called, the row pointer advances. If you need another column from the same row, use `fetch()` instead.
- **`fetchAll()` memory usage:** For a result set of 1 million rows with 20 columns, `fetchAll(PDO::FETCH_ASSOC)` can consume hundreds of megabytes. Use unbuffered queries or row-by-row `fetch()` for large data sets.
- **`FETCH_CLASS` and private/protected properties:** PDO sets properties directly, bypassing visibility. If a property is private, PDO may set it as a public dynamic property instead, causing unexpected behaviour.
- **`FETCH_CLASS` and `__construct()`:** By default, PDO sets properties and then calls the constructor with no arguments. If your class requires constructor arguments, use `PDO::FETCH_PROPS_LATE` with `fetchAll(PDO::FETCH_CLASS, ClassName::class, $constructorArgs)`.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: `fetch()`, `fetchAll()`, and `fetchColumn()` Compared**

```php
<?php
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE employees (id INTEGER PRIMARY KEY, name TEXT, salary REAL)');
$pdo->exec("INSERT INTO employees (name, salary) VALUES ('Alice', 75000), ('Bob', 62000), ('Carol', 88000)");

$stmt = $pdo->query('SELECT * FROM employees ORDER BY salary DESC');

// --- fetch(): one row at a time ---
$row = $stmt->fetch(PDO::FETCH_ASSOC);
echo "Highest paid (fetch): " . $row['name'] . " - $" . $row['salary'] . "\n";

// --- fetchAll(): all remaining rows ---
$remaining = $stmt->fetchAll(PDO::FETCH_ASSOC);
echo "Remaining employees (fetchAll): " . count($remaining) . "\n";
foreach ($remaining as $r) {
    echo "  - " . $r['name'] . "\n";
}

// --- fetchColumn(): single scalar value ---
$stmt2 = $pdo->query('SELECT COUNT(*) FROM employees');
$count = $stmt2->fetchColumn();
echo "Total employees (fetchColumn): " . $count . "\n";

// --- fetchColumn() with column index ---
$stmt3 = $pdo->query('SELECT name, salary FROM employees ORDER BY salary DESC');
$highestSalary = $stmt3->fetchColumn(1);  // column index 1 = salary
echo "Highest salary (fetchColumn index 1): $" . $highestSalary . "\n";
```

**Expected Output:**

```
Highest paid (fetch): Carol - $88000
Remaining employees (fetchAll): 2
  - Alice
  - Bob
Total employees (fetchColumn): 3
Highest salary (fetchColumn index 1): $88000
```

**Why:** `fetch()` retrieves only the first row (Carol, highest salary). `fetchAll()` then retrieves the remaining two rows. `fetchColumn()` with no argument returns the first column (COUNT). `fetchColumn(1)` returns the second column (salary) of the next row, which is Carol's $88,000.

**Example 2: `PDO::FETCH_CLASS` into Typed Objects**

```php
<?php
class User
{
    public int $id;
    public string $name;
    public string $email;

    public function getDisplayName(): string
    {
        return strtoupper($this->name) . " <{$this->email}>";
    }
}

$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)');
$pdo->exec("INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com'), ('Bob', 'bob@example.com')");

$stmt = $pdo->query('SELECT id, name, email FROM users');

// Fetch all rows as User objects
$users = $stmt->fetchAll(PDO::FETCH_CLASS, User::class);

foreach ($users as $user) {
    echo $user->getDisplayName() . "\n";
}

// Verify type
echo "First result is a " . get_class($users[0]) . " object.\n";
```

**Expected Output:**

```
ALICE <alice@example.com>
BOB <bob@example.com>
First result is a User object.
```

**Why:** `PDO::FETCH_CLASS` creates a `User` instance for each row and maps the column values (`id`, `name`, `email`) to the corresponding public properties. The `getDisplayName()` method can then be called on each object, demonstrating that the result is a true typed object, not a generic `stdClass`.

**Example 3: `fetchAll()` with `PDO::FETCH_COLUMN` for Flat Arrays**

```php
<?php
$pdo = new PDO('sqlite::memory:', null, null, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->exec('CREATE TABLE tags (id INTEGER PRIMARY KEY, label TEXT)');
$pdo->exec("INSERT INTO tags (label) VALUES ('php'), ('pdo'), ('security'), ('database')");

$stmt = $pdo->query('SELECT label FROM tags ORDER BY label');

// fetchAll with FETCH_COLUMN returns a flat array of all values in column 0
$tags = $stmt->fetchAll(PDO::FETCH_COLUMN, 0);

print_r($tags);
echo "Tag count: " . count($tags) . "\n";
```

**Expected Output:**

```
Array
(
    [0] => database
    [1] => pdo
    [2] => php
    [3] => security
)
Tag count: 4
```

**Why:** `PDO::FETCH_COLUMN` with column index `0` returns a one-dimensional array containing every value from the first column of the result set. This is ideal for populating dropdown lists, checkboxes, or `IN()` clause value lists.

### Real-World Cases

**Case 1: REST API with Typed Response Objects**

A REST API uses `PDO::FETCH_CLASS` to map database rows directly to DTO (Data Transfer Object) classes. Each DTO implements `JsonSerializable`, so the API controller can pass the objects directly to `json_encode()` without manual array construction.

**Case 2: Admin Dashboard with Paginated Results**

An admin dashboard displays paginated user records. It uses `fetchAll(PDO::FETCH_ASSOC)` for the current page (typically 25–50 rows) and `fetchColumn()` to retrieve the total count for pagination controls:

```php
$countStmt = $pdo->query('SELECT COUNT(*) FROM users');
$totalUsers = (int) $countStmt->fetchColumn();

$pageStmt = $pdo->prepare('SELECT * FROM users LIMIT :limit OFFSET :offset');
$pageStmt->bindValue(':limit', 25, PDO::PARAM_INT);
$pageStmt->bindValue(':offset', ($page - 1) * 25, PDO::PARAM_INT);
$pageStmt->execute();
$users = $pageStmt->fetchAll(PDO::FETCH_ASSOC);
```

**Case 3: CLI Report Generator with Row-by-Row Processing**

A CLI script generates a CSV report from a table with millions of rows. It uses `fetch()` in a `while` loop and unbuffered queries to process rows one at a time, writing each to the CSV file immediately and discarding it from memory:

```php
$pdo->setAttribute(PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);
$stmt = $pdo->query('SELECT id, name, email FROM users ORDER BY id');

$fh = fopen('report.csv', 'w');
fputcsv($fh, ['ID', 'Name', 'Email']);

while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    fputcsv($fh, $row);
}

fclose($fh);
```

---

## Core Concept 4: Cursor & Stream Management

### Definitions

**Core Definition**

Cursor and stream management refers to the techniques used to control how database result sets are transferred from the database server to PHP, balancing memory consumption against database server load.

**Technical Definition**

By default, the MySQL PDO driver uses **buffered queries**: the entire result set is transferred from the MySQL server to PHP's process memory (via the `mysqlnd` driver) immediately upon execution. **Unbuffered queries** (`PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => false`) leave rows on the MySQL server and fetch them one at a time as `fetch()` is called, dramatically reducing PHP memory usage at the cost of increased server-side load and restrictions on concurrent queries.

**Beginner-Friendly Explanation**

Imagine you order 1,000 books from a warehouse. **Buffered mode** is like the warehouse shipping all 1,000 books to your house at once — your living room (PHP memory) fills up, but the warehouse can immediately handle your next order. **Unbuffered mode** is like the warehouse keeping the books and sending one at a time as you ask — your living room stays empty, but the warehouse cannot process another order until you finish receiving the current one.

### Purposes

- To prevent PHP memory exhaustion when processing very large result sets.
- To reduce the memory footprint of CLI scripts and background workers that process millions of rows.
- To stream query results directly to output (CSV, JSON, XML) without loading the entire result set into memory.
- To understand and mitigate the restrictions that unbuffered queries impose on concurrent database operations.
- To choose the appropriate cursor strategy for long-running versus short-lived PHP processes.

### Syntax Rules and Structure

**Enabling Unbuffered Queries (MySQL Driver)**

```php
$pdo = new PDO($dsn, $username, $password, [
    PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => false,
]);
```

**Or via `setAttribute()` After Connection**

```php
$pdo->setAttribute(PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);
```

**Component Breakdown:**

- `PDO::MYSQL_ATTR_USE_BUFFERED_QUERY` — A MySQL-driver-specific attribute. When set to `true` (the default), result sets are buffered in PHP memory. When set to `false`, result sets remain on the MySQL server and are fetched incrementally.

**Syntax Rules:**

- Unbuffered mode must be set **before** the query is executed. Setting it after execution has no effect on the current result set.
- While an unbuffered result set is open (i.e., not fully fetched or explicitly freed), no other queries can be executed on the same connection. Attempting to do so throws a `PDOException` with the message "Cannot execute queries while other unbuffered queries are active."
- To free an unbuffered result set before it is fully fetched, call `$stmt->closeCursor()`.

**Constraints and Limitations:**

- **Single active result set:** Unbuffered mode allows only one active query per connection. This makes it incompatible with patterns that interleave queries (e.g., fetching a row, then querying a related table, then fetching the next row).
- **Increased database server load:** Because rows are sent one at a time, the MySQL server must maintain the query state and connection until the result set is exhausted. With many concurrent unbuffered queries, server memory and connection limits can be strained.
- **Not supported by all drivers:** `PDO::MYSQL_ATTR_USE_BUFFERED_QUERY` is MySQL-specific. PostgreSQL's PDO driver does not have an equivalent attribute; it uses server-side cursors differently.
- **`rowCount()` unreliable:** In unbuffered mode, `rowCount()` may return an inaccurate value because the full result set has not been transferred yet.
- **Historical bugs:** PHP has had memory leaks in unbuffered queries when combined with native prepared statements (`ATTR_EMULATE_PREPARES => false`). Test thoroughly on your PHP version.
- **`fetchAll()` defeats the purpose:** Calling `fetchAll()` on an unbuffered result set loads all rows into PHP memory anyway, negating the benefit. Use `fetch()` in a loop instead.

### Multiple Annotated Step-by-Step Code Examples

**Example 1: Buffered vs. Unbuffered Memory Usage**

```php
<?php
// --- BUFFERED (default) ---
$pdo = new PDO('mysql:host=localhost;dbname=testdb', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);
$pdo->setAttribute(PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, true);

$memBefore = memory_get_usage();
$stmt = $pdo->query('SELECT * FROM large_table');  // e.g., 100,000 rows
$memAfterBuffered = memory_get_usage();
echo "Buffered memory increase: " . round(($memAfterBuffered - $memBefore) / 1024 / 1024, 2) . " MB\n";
$stmt->closeCursor();

// --- UNBUFFERED ---
$pdo->setAttribute(PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);

$memBefore = memory_get_usage();
$stmt = $pdo->query('SELECT * FROM large_table');
$memAfterUnbuffered = memory_get_usage();
echo "Unbuffered memory increase: " . round(($memAfterUnbuffered - $memBefore) / 1024 / 1024, 2) . " MB\n";

// Process rows one at a time
$rowCount = 0;
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    $rowCount++;
}
echo "Processed $rowCount rows without memory exhaustion.\n";
$stmt->closeCursor();
```

**Expected Output (illustrative):**

```
Buffered memory increase: 48.00 MB
Unbuffered memory increase: 0.01 MB
Processed 100000 rows without memory exhaustion.
```

**Why:** In buffered mode, the entire 100,000-row result set is transferred to PHP memory immediately, causing a 48 MB increase. In unbuffered mode, only the query metadata is stored in PHP memory; rows remain on the MySQL server and are fetched one at a time. The memory increase is negligible, and the script can process the entire result set without hitting `memory_limit`.

**Example 2: Streaming Results to CSV**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=analytics', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => false,
]);

$stmt = $pdo->query('SELECT id, event_name, event_time FROM events ORDER BY event_time');

// Open output stream
$fh = fopen('php://output', 'w');
header('Content-Type: text/csv');
header('Content-Disposition: attachment; filename="events.csv"');

// Write header row
fputcsv($fh, ['ID', 'Event', 'Timestamp']);

// Stream rows one at a time
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    fputcsv($fh, $row);
}

fclose($fh);
$stmt->closeCursor();
```

**Expected Output:** A downloadable CSV file containing all event rows, streamed directly to the browser without loading the full result set into PHP memory.

**Why:** The unbuffered query keeps rows on the MySQL server. Each `fetch()` call retrieves one row, which is immediately written to the output stream and discarded. PHP memory usage remains constant regardless of the total number of rows.

**Example 3: Handling the "Cannot Execute Queries" Error**

```php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=shop', 'user', 'pass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => false,
]);

$stmt = $pdo->query('SELECT id, name FROM products');

while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    // Attempting to execute another query while an unbuffered
    // result set is open will throw an exception.
    try {
        $related = $pdo->query('SELECT COUNT(*) FROM reviews WHERE product_id = ' . (int)$row['id']);
        echo $row['name'] . " has " . $related->fetchColumn() . " reviews.\n";
    } catch (PDOException $e) {
        echo "Error: " . $e->getMessage() . "\n";
        break;
    }
}

// Correct approach: fetch all product IDs first, then query reviews
$stmt->closeCursor();

$productIds = $pdo->query('SELECT id, name FROM products')->fetchAll(PDO::FETCH_ASSOC);
foreach ($productIds as $product) {
    $reviewCount = $pdo->query('SELECT COUNT(*) FROM reviews WHERE product_id = ' . (int)$product['id'])->fetchColumn();
    echo $product['name'] . " has $reviewCount reviews.\n";
}
```

**Expected Output (first loop):**

```
Error: SQLSTATE[HY000]: General error: 2014 Cannot execute queries while other unbuffered queries are active. Consider using PDOStatement::fetchAll(). Alternatively, if your code is only ever going to run against mysql, you may enable query buffering by setting the PDO::MYSQL_ATTR_USE_BUFFERED_QUERY attribute.
```

**Expected Output (second loop):**

```
Widget has 5 reviews.
Gadget has 3 reviews.
...
```

**Why:** In unbuffered mode, the connection is occupied by the open result set. Executing another query on the same connection throws an exception. The correct approach is to either `closeCursor()` before executing another query, fetch all primary results first, or use a separate connection for the nested queries.

### Real-World Cases

**Case 1: Data Export Tool for a Multi-Million-Row Table**

A reporting tool exports a 5-million-row table to a CSV file. With buffered queries, the script would exhaust PHP's `memory_limit` (typically 128 MB or 256 MB) and crash. By setting `PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => false` and using `fetch()` in a `while` loop, the script streams each row to the CSV file and maintains a constant memory footprint of a few megabytes.

**Case 2: Drupal's Default Buffered Query Behaviour**

Drupal's database layer uses buffered queries by default (`PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => true`). This forces the entire result set into PHP memory, which can cause issues with very large data sets. Drupal provides a `fetchAllAssoc()` method that buffers results, and advanced developers can override the connection attribute for specific queries that need unbuffered streaming.

**Case 3: Swoole Coroutine Worker with Unbuffered Queries**

In a Swoole HTTP server, each coroutine must use its own database connection. Unbuffered queries are particularly dangerous in this context because a coroutine that holds an open unbuffered result set while yielding control to another coroutine can cause the connection to be in an inconsistent state. Best practice is to either fetch all rows before yielding, or use a connection pool where each coroutine borrows a connection exclusively for its query and returns it only after the result set is fully consumed or explicitly freed with `closeCursor()`.

---

## References

- PHP: PDO::prepare — Manual – https://www.php.net/manual/en/pdo.prepare.php
- PHP: PDOStatement::execute — Manual – https://www.php.net/manual/en/pdostatement.execute.php
- PHP: PDOStatement::bindParam — Manual – https://www.php.net/manual/en/pdostatement.bindparam.php
- PHP: PDOStatement::bindValue — Manual – https://www.php.net/manual/en/pdostatement.bindvalue.php
- PHP: PDOStatement::fetch — Manual – https://www.php.net/manual/en/pdostatement.fetch.php
- PHP: PDOStatement::fetchAll — Manual – https://www.php.net/manual/en/pdostatement.fetchall.php
- PHP: PDOStatement::fetchColumn — Manual – https://www.php.net/manual/en/pdostatement.fetchcolumn.php
- PHP: PDO::setAttribute — Manual – https://www.php.net/manual/en/pdo.setattribute.php
- PHP: PDO Constants — Manual – https://www.php.net/manual/en/pdo.constants.php
- Invicti: How to Prevent SQL Injection Vulnerabilities in PHP Applications – https://www.invicti.com/blog/web-security/how-to-prevent-sql-injection-in-php-applications
- PHP Delusions: PDO — https://phpdelusions.net/pdo
- PHP: PDO::MYSQL_ATTR_USE_BUFFERED_QUERY — Manual – https://www.php.net/manual/en/ref.pdo-mysql.php
- Stack Overflow: PDO bindValue vs bindParam — https://stackoverflow.com/questions/1179874/pdo-bindvalue-versus-bindparam
- Stack Overflow: Unbuffered Queries Memory Benefits — https://stackoverflow.com/questions/16210421/pdo-unbuffered-queries-and-memory-usage
- PHP Bug #66558: MYSQL_ATTR_USE_BUFFERED_QUERY – https://bugs.php.net/bug.php?id=66558
- OWASP: SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Microsoft: PDO::ATTR_EMULATE_PREPARES – https://learn.microsoft.com/en-us/sql/connect/php/pdo-prepare