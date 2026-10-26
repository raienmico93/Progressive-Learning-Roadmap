# SQL Injection Prevention — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **SQL injection (SQLi)** is a code-injection vulnerability in which untrusted data is interpreted as **SQL syntax** by a database engine, allowing an attacker to alter the structure and semantics of a query. It is classified under **CWE-89** and **OWASP Top 10 A03:2021-Injection** (A05:2025-Injection in the 2025 edition).

**Technical Definition.** SQL injection occurs when an application **concatenates or interpolates** untrusted input into a query string **before** the database parses it. The database cannot distinguish attacker-supplied syntax from application-supplied syntax, so the injected fragment becomes part of the query's grammar. The definitive mitigation is **query parameterization** — also called **prepared statements** — which separates the **SQL structure** from the **data values** at the protocol level, ensuring bound values are always treated as data and never as syntax.

**Beginner-Friendly Explanation.** SQL injection is **when someone types SQL code into a form field and your database runs it**. If your app builds a query by gluing strings together — `"SELECT * FROM users WHERE name = '" + input + "'"` — an attacker can type `' OR '1'='1` and change the meaning of the query. The fix is to **never glue input into SQL**: use **placeholders** (`?` or `$1`) and pass the values separately, so the database knows exactly what is code and what is data.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Injection through syntax confusion** | The attacker's input becomes part of the SQL grammar rather than remaining a literal value. |
| **Parameterization is the definitive fix** | Prepared statements with bound parameters eliminate injection at the protocol level. |
| **Escaping is discouraged** | Manual escaping depends on dialect and configuration and is **strongly discouraged** as a primary defense. |
| **Identifiers are a separate problem** | Table names, column names, and sort directions **cannot be parameterized**; they require **allow-list mapping**. |
| **ORMs are safe by default but bypassable** | ORMs parameterize query-builder methods, but raw-query escape hatches reintroduce risk. |
| **Least privilege limits blast radius** | It does not prevent injection, but it reduces the damage a successful attack can cause. |

---

### Prerequisites

- **SQL fundamentals**: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `WHERE`, and `JOIN`.
- **HTTP and web application basics**: request parameters, bodies, and headers.
- **A backend language and database driver**: Java/JDBC, Node.js/mysql2/pg, Python/psycopg2, or an ORM.
- **A database instance** (MySQL, PostgreSQL, or SQLite) to run the examples.

---

### Related Programming Areas

- **Input validation**: a secondary defense that reduces the attack surface but does not replace parameterization.
- **ORM and query builders**: provide automatic parameterization for their query-builder APIs.
- **Least privilege**: database account hardening that limits post-exploitation impact.
- **SAST and code review**: detecting string-concatenated SQL before it ships.
- **Stored procedures**: safe only when they bind arguments and avoid dynamic SQL concatenation.

---

## Core Concepts / Features

---

## 1. SQL Injection

### Definitions

**Core Definition.** **SQL injection** is a vulnerability in which attacker-controlled data is incorporated into a SQL query as **executable syntax**, allowing the attacker to read, modify, or delete data, bypass authentication, or execute database or operating-system commands.

**Technical Definition.** SQL injection arises when the application constructs a query by **concatenating, interpolating, or formatting** untrusted values into the query text before it reaches the database parser. The injected fragment can terminate the intended literal, add boolean conditions (`' OR '1'='1`), append stacked statements (`'; DROP TABLE ...; --`), or inject `UNION` clauses to exfiltrate data from other tables. The vulnerability class applies to **any** data source — HTTP parameters, headers, cookies, message queues, or previously stored data.

**Beginner-Friendly Explanation.** SQL injection is **the database being tricked into running code you did not write**. If your app trusts a user's input enough to paste it directly into a SQL string, the user can paste SQL commands instead of data. The result can be a full database breach: stolen passwords, deleted records, or even operating-system access.

### Purposes

- **To understand the attack surface** created by dynamic query construction.
- **To recognize vulnerable patterns** in code review and static analysis.
- **To motivate the adoption of parameterized queries** as the primary defense.
- **To prioritize remediation** for any query that concatenates untrusted input.
- **To design layered defenses** that limit impact even when a vulnerability exists.

### Syntax Rules and Structure

SQL injection is an **attack pattern**, not a syntax. The vulnerable construction patterns are:

| Vulnerable Pattern | Language Example | Why It Fails |
|---|---|---|
| **String concatenation** | `"SELECT * FROM users WHERE id = " + userId` | The value becomes part of the query text. |
| **Template interpolation** | `` `SELECT * FROM users WHERE name = '${name}'` `` | The value is spliced into the string before parsing. |
| **Format strings** | `"SELECT * FROM users WHERE id = %s" % userId` | The value is formatted into the query text. |
| **ORM raw-query interpolation** | `sequelize.query(\`... '${input}'\`)` | The raw-query API passes the string directly to the driver. |

**Constraints and limitations.**

- **Input validation does not fix SQL injection**; a value can pass every validation rule and still be dangerous when concatenated.
- **Escaping is dialect-dependent** and can be disabled by configuration (e.g., MySQL's `NO_BACKSLASH_ESCAPES`).
- **ORMs create a false sense of security**; their raw-query escape hatches are just thin wrappers around the driver.

### Annotated Complete Step by Step Code Examples

#### Example 1: Vulnerable vs. safe query (Node.js + `pg`)

```javascript
// ❌ VULNERABLE: template literal interpolation.
const result = await pool.query(
  `SELECT * FROM invoices WHERE customer_id = '${req.query.id}'`
);
// Input: ?id=1' OR '1'='1
// Resulting query: SELECT * FROM invoices WHERE customer_id = '1' OR '1'='1'
// → Returns EVERY invoice in the table.

// ✅ SAFE: parameterized query with $1 placeholder.
const result = await pool.query(
  'SELECT * FROM invoices WHERE customer_id = $1',
  [req.query.id]
);
// Input: ?id=1' OR '1'='1
// The driver binds the entire string as a single value.
// → Returns only rows where customer_id literally equals that string (none).
```

**Expected Output (conceptual):**
```
Vulnerable version: all invoices returned (authentication/authorization bypass).
Safe version: no rows returned; the payload is treated as a literal value.
```

**Why this output occurs.** In the vulnerable version, the input becomes part of the SQL grammar, so the `OR '1'='1` condition is always true. In the safe version, the `$1` placeholder tells PostgreSQL to expect a **value**; the driver sends the query template and the parameter separately, and the engine binds the parameter as data. The payload cannot alter the query structure.

---

#### Example 2: Authentication bypass via injection

```sql
-- Intended query (vulnerable concatenation):
SELECT * FROM users WHERE username = 'alice' AND password = 'secret';

-- Attacker input for password: ' OR '1'='1
-- Resulting query:
SELECT * FROM users WHERE username = 'alice' AND password = '' OR '1'='1';
-- Because OR '1'='1' is always true, the query returns alice's row.
-- The attacker is authenticated without knowing the password.
```

**Expected Output (conceptual):**
```
Vulnerable: login succeeds with any password when the payload is injected.
Safe (parameterized): login fails because the payload is treated as the literal password string.
```

**Why this output occurs.** The injected `OR '1'='1` changes the boolean logic of the `WHERE` clause. With parameterization, the password value — including the quote and `OR` — is bound as a single string, so the condition becomes `password = ''' OR ''1''=''1'` (a literal mismatch), and the login fails.

---

### Real-World Cases

- **CVE-2023-25813 (Sequelize)**: a SQL injection vulnerability in one of the most widely used Node.js ORMs, with a CVSS score of 9.8, arising from how replacements and bind parameters interacted.
- **Legacy authentication systems**: classic `' OR '1'='1` login bypasses remain common in codebases that concatenate credentials.
- **Reporting and export endpoints**: raw-query escape hatches in ORMs are a frequent injection point because reviewers assume the ORM handles escaping.

---

## 2. Parameterized Queries

### Definitions

**Core Definition.** A **parameterized query** is a SQL query in which **placeholders** (such as `?`, `$1`, or `:name`) stand in for data values, and the actual values are supplied **separately** to the database driver, which binds them as data rather than SQL syntax.

**Technical Definition.** Parameterization separates the **query template** from the **parameter values** at the API level. The driver sends the template to the database for parsing and planning, then transmits the values through a separate channel; the engine substitutes them into the planned statement as **bound data**. Because the values never pass through the SQL parser as text, they cannot alter the query's grammar. Placeholders represent **data values only** — not table names, column names, or keywords such as `ASC`/`DESC`.

**Beginner-Friendly Explanation.** A parameterized query is **a fill-in-the-blank query**. Instead of writing `"SELECT * FROM users WHERE name = 'Alice'"`, you write `"SELECT * FROM users WHERE name = ?"` and pass `"Alice"` separately. The database knows the `?` is a **slot for a value**, so even if the value is `' OR '1'='1`, it stays a literal string and never becomes SQL code.

### Purposes

- **To eliminate SQL injection at the protocol level** by ensuring values are never parsed as syntax.
- **To provide a uniform, language-agnostic defense** across drivers and frameworks.
- **To enable prepared-statement reuse** for repeated executions, improving performance.
- **To replace fragile escaping** with a mechanism that does not depend on dialect-specific rules.
- **To serve as the foundation for ORM safety** when query builders are used correctly.

### Syntax Rules and Structure

**General syntax pattern.**

```
queryTemplate = "SELECT ... WHERE column = placeholder"
values = [userInput]
execute(queryTemplate, values)
```

**Placeholder conventions by driver.**

| Driver / Language | Placeholder | Example |
|---|---|---|
| **`pg` (Node.js/PostgreSQL)** | `$1`, `$2`, … | `pool.query('SELECT * FROM users WHERE id = $1', [id])` |
| **`mysql2` (Node.js/MySQL)** | `?` | `connection.execute('SELECT * FROM users WHERE email = ?', [email])` |
| **`better-sqlite3` (Node.js/SQLite)** | `?` | `db.prepare('SELECT * FROM users WHERE id = ?').get(id)` |
| **`psycopg2` (Python/PostgreSQL)** | `%s` | `cursor.execute('SELECT * FROM users WHERE id = %s', (user_id,))` |
| **JDBC (Java)** | `?` | `PreparedStatement ps = conn.prepareStatement("... WHERE name = ?"); ps.setString(1, name);` |
| **PDO (PHP)** | `:name` or `?` | `$stmt = $dbh->prepare("... WHERE name = :name"); $stmt->bindParam(':name', $name);` |

**Parameterization rules.**

| Rule | Guidance |
|---|---|
| **Parameterize every data value** | Even values that pass validation must be bound, not concatenated. |
| **Never parameterize identifiers** | Table names, column names, and sort directions cannot be placeholders; use allow-list mapping. |
| **Prepare, then bind** | The template must be a **constant string**; concatenating input before preparing defeats the protection. |
| **Do not mix raw replacements with bind parameters** | This was the root cause of CVE-2023-25813 in Sequelize. |

**Constraints and limitations.**

- **Placeholders cannot represent SQL keywords or identifiers**; `ORDER BY ?` will not work as intended.
- **Some drivers require different placeholder syntax**; using the wrong one causes errors or, worse, silent fallback to concatenation.
- **Parameterization does not enforce authorization**; it prevents injection but does not decide whether the caller may access the matched row.

### Annotated Complete Step by Step Code Examples

#### Example 1: `mysql2` with `execute` (MySQL)

```javascript
// Step 1: Import the promise-based mysql2 driver.
const mysql = require('mysql2/promise');

async function getUserByEmail(email) {
  // Step 2: Create a connection.
  const connection = await mysql.createConnection({
    host: 'localhost', user: 'app', password: process.env.DB_PASS, database: 'myapp',
  });

  // Step 3: Use execute() with a ? placeholder and a values array.
  const [rows] = await connection.execute(
    'SELECT id, name, email FROM users WHERE email = ?',
    [email]
  );

  // Step 4: Close the connection and return the result.
  await connection.end();
  return rows[0] || null;
}

// Step 5: Call with a malicious payload.
getUserByEmail("victim@example.com' OR '1'='1")
  .then((user) => console.log(user));
// Expected: null (no user with that literal email string).
```

**Expected Output:**
```
null
```

**Why this output occurs.** `execute()` uses a **prepared statement** internally. The `?` placeholder is a data slot, and the driver sends the payload as a bound parameter. The database looks for a user whose email is literally `"victim@example.com' OR '1'='1"`, which does not exist, so no rows are returned.

---

#### Example 2: `pg` with `$1` (PostgreSQL)

```javascript
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function getUserById(userId) {
  // The $1 placeholder is replaced by the driver with the bound value.
  const result = await pool.query(
    'SELECT id, name, email FROM users WHERE id = $1',
    [userId]
  );
  return result.rows[0] || null;
}

// Malicious input: "1 OR 1=1"
getUserById("1 OR 1=1").then((user) => console.log(user));
// Expected: null (the entire string is treated as the id value).
```

**Expected Output:**
```
null
```

**Why this output occurs.** `pool.query()` with a parameter array performs **server-side parameter binding**. The `$1` token is a placeholder, not a textual substitution. PostgreSQL parses the query template once and binds the parameter as a value, so the injected `OR 1=1` never becomes SQL syntax.

---

#### Example 3: `psycopg2` with `%s` (Python)

```python
import psycopg2

def get_user(conn, user_id):
    with conn.cursor() as cursor:
        # ✅ Correct: %s placeholder, tuple of values.
        cursor.execute(
            "SELECT id, name, email FROM users WHERE id = %s",
            (user_id,)
        )
        return cursor.fetchone()

# Malicious input.
get_user(conn, "1 OR 1=1")
# Expected: None (the payload is bound as a literal value).
```

**Expected Output:**
```
None
```

**Why this output occurs.** `psycopg2` uses the `%s` placeholder (not Python string formatting). The driver sends the query and the parameter tuple separately; PostgreSQL binds the value as data. The payload is treated as an ID value, not as SQL syntax.

---

#### Example 4: JDBC `PreparedStatement` (Java)

```java
// Step 1: Define the query with a ? placeholder.
String query = "SELECT account_balance FROM user_data WHERE user_name = ?";

// Step 2: Prepare the statement.
PreparedStatement statement = connection.prepareStatement(query);

// Step 3: Bind the value.
statement.setString(1, customerName);

// Step 4: Execute.
ResultSet results = statement.executeQuery();
```

**Expected Output (conceptual):**
```
Input: "tom' or '1'='1"
→ The database searches for the literal username "tom' or '1'='1".
→ No account balance is returned (the payload is data, not syntax).
```

**Why this output occurs.** The `PreparedStatement` sends the SQL template to the database for compilation, then binds `customerName` as a string parameter. The bound value is treated as data and cannot change the query structure, even though it contains SQL metacharacters.

---

### Real-World Cases

- **Node.js/Express APIs**: `mysql2` and `pg` are the standard drivers; their `execute` and `query` methods with parameter arrays are the recommended pattern.
- **Python/Django and SQLAlchemy**: ORMs parameterize by default; raw SQL must use `%s` placeholders or `psycopg2.sql`.
- **Java enterprise applications**: JDBC `PreparedStatement` is the long-standing standard for parameterized access.

---

## 3. Prepared Statements

### Definitions

**Core Definition.** A **prepared statement** is a **precompiled SQL statement** stored on the database server, in which **placeholders** are bound to values at execution time. Prepared statements are the mechanism through which parameterized queries are executed.

**Technical Definition.** The database **parses, validates, and plans** the query template once, producing an executable plan with parameter slots. Each execution supplies values for those slots, and the engine executes the plan with the values bound as data. Prepared statements improve performance for repeated executions and provide the **strongest injection defense** because the query structure is fixed before any value is supplied. The terms “prepared statement” and “parameterized query” are often used interchangeably; OWASP uses the combined phrase **“Prepared Statements (with Parameterized Queries)”**.

**Beginner-Friendly Explanation.** A prepared statement is **a query template that the database compiles once and reuses**. You write the query with blanks, the database prepares it, and then you fill in the blanks with values. Because the database already knows the query's structure, the values cannot change it. This is both **faster** (for repeated queries) and **safer** (no injection).

### Purposes

- **To precompile the query structure** so values can never alter the grammar.
- **To improve performance** when the same query is executed repeatedly with different values.
- **To provide a consistent, driver-level mechanism** that works across languages and databases.
- **To serve as the implementation behind parameterized queries** in most drivers and ORMs.

### Syntax Rules and Structure

**General lifecycle.**

```
1. PREPARE: send the query template with placeholders to the database.
2. BIND:   supply values for the placeholders.
3. EXECUTE: run the prepared plan with the bound values.
4. DEALLOCATE (optional): release the prepared statement.
```

**Stored procedures vs. prepared statements.**

| Mechanism | Safety | Condition |
|---|---|---|
| **Prepared statement with parameterized query** | Safe by design | The template is constant; values are bound. |
| **Stored procedure** | **Not automatically safe** | Safe only when it **binds** arguments and does **not** concatenate dynamic SQL internally. |

**Prepared-statement rules.**

| Rule | Guidance |
|---|---|
| **Prepare a constant template** | Never concatenate input into the string before preparing. |
| **Bind values, not identifiers** | Placeholders represent data; identifiers require allow-list mapping. |
| **Use dynamic SQL carefully inside procedures** | If a procedure builds SQL dynamically, bind the values with `EXECUTE ... USING` or `sp_executesql`. |

**Constraints and limitations.**

- **Some drivers emulate prepared statements client-side** (e.g., older `mysql` without `execute`), which can weaken protection; prefer drivers that use server-side preparation.
- **Prepared statements are per-connection**; connection pools manage them, and some pools limit the number of cached statements.
- **DDL and some dynamic operations cannot be prepared**; identifiers must be handled through allow-lists.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit `PREPARE` / `EXECUTE` in PostgreSQL

```sql
-- Step 1: Prepare the statement with $1 and $2 placeholders.
PREPARE get_user (int) AS
  SELECT id, name, email FROM users WHERE id = $1;

-- Step 2: Execute with a value.
EXECUTE get_user(42);

-- Step 3: Execute again with a different value (reuses the plan).
EXECUTE get_user(99);

-- Step 4: Release the prepared statement.
DEALLOCATE get_user;
```

**Expected Output (conceptual):**
```
First execution: returns the user with id 42.
Second execution: returns the user with id 99.
The query plan is parsed and planned once, then reused.
```

**Why this output occurs.** `PREPARE` compiles the query template and stores it under the name `get_user`. `EXECUTE` supplies the parameter value; the engine runs the precompiled plan with the value bound. The template cannot be altered by the value, so injection is impossible.

---

#### Example 2: Stored procedure with safe binding (PostgreSQL PL/pgSQL)

```sql
-- ✅ SAFE: static SQL with parameters is bound automatically.
CREATE OR REPLACE PROCEDURE safe_get_balance(user_id INT, dept TEXT)
LANGUAGE plpgsql AS $$
BEGIN
  PERFORM balance FROM accounts
   WHERE user_id = user_id AND department = dept;
END;
$$;

-- ✅ SAFE: dynamic SQL with EXECUTE ... USING binds values.
CREATE OR REPLACE PROCEDURE safe_dynamic_get_balance(user_id INT, dept TEXT)
LANGUAGE plpgsql AS $$
DECLARE
  stmt TEXT := 'SELECT balance FROM accounts WHERE user_id = $1 AND department = $2';
  result NUMERIC;
BEGIN
  EXECUTE stmt INTO result USING user_id, dept;
  RAISE NOTICE 'Balance: %', result;
END;
$$;
```

**Expected Output (conceptual):**
```
Both procedures bind arguments as data.
The dynamic version uses EXECUTE ... USING, so $1 and $2 are bound values, not concatenated text.
```

**Why this output occurs.** The first procedure uses static SQL, and procedure arguments are naturally bound to their positions. The second uses dynamic SQL but binds the values with `EXECUTE ... USING`, which tells PostgreSQL that `user_id` and `dept` are data, not code. This is the safe pattern for dynamic SQL inside procedures.

---

### Real-World Cases

- **High-throughput OLTP systems**: prepared statements reduce parsing and planning overhead for repeated queries.
- **Connection pools**: drivers such as `pg` and `mysql2` cache prepared statements per connection.
- **Stored procedures in banking systems**: safe implementations bind arguments and avoid dynamic SQL concatenation.

---

## 4. ORM Protection

### Definitions

**Core Definition.** **ORM protection** refers to the safety properties that object-relational mapping frameworks provide **by default** — specifically, automatic parameterization of queries built through their **query-builder APIs** — and the **boundaries** beyond which that protection no longer applies.

**Technical Definition.** ORMs such as **Sequelize, Prisma, TypeORM, and Knex** (Node.js), **SQLAlchemy and Django ORM** (Python), and **Entity Framework Core** (.NET) translate high-level query expressions into parameterized SQL. Their **query-builder methods** (`where`, `filter`, `find`, `select`) bind values automatically. However, every major ORM exposes **raw-query escape hatches** — `sequelize.query()`, `prisma.$queryRawUnsafe()`, `typeorm.query()`, `knex.raw()` — which are thin wrappers around the database driver and **do not** parameterize interpolated strings.

**Beginner-Friendly Explanation.** ORMs are **safe by default, but only when you use their normal query methods**. If you ask the ORM to “find users where email equals this value,” it parameterizes the value for you. But if you drop down to a **raw SQL escape hatch** and glue strings together, the ORM steps aside and the injection risk returns. The rule is: **stay on the query-builder path; if you must write raw SQL, parameterize it yourself**.

### Purposes

- **To leverage automatic parameterization** for the majority of queries without writing placeholder syntax.
- **To recognize the escape hatches** where ORM protection stops and manual parameterization is required.
- **To prevent ORM-specific injection patterns** such as raw-query interpolation and unsafe identifier handling.
- **To combine ORM safety with least privilege** so that a bypass cannot cause maximum damage.

### Syntax Rules and Structure

**ORM safety matrix (Node.js ORMs).**

| ORM | Safe by Default | Unsafe Escape Hatches | Safe Raw-SQL Pattern |
|---|---|---|---|
| **Sequelize** | Query builder, `where` clauses | `sequelize.query(\`... ${input}\`)`, `sequelize.literal()` | `sequelize.query('... WHERE email = $email', { bind: { email } })` |
| **Prisma** | `findMany`, `where`, `create` | `$queryRawUnsafe`, `$executeRawUnsafe` | `` prisma.$queryRaw`... WHERE status = ${status}` `` (tagged template) |
| **TypeORM** | Repository methods, `createQueryBuilder` | `.query(\`... ${input}\`)` | `repository.query('... WHERE role = $1', [role])` |
| **Knex** | `where()`, `andWhere()`, `whereIn()` | `knex.raw(\`... ${input}\`)`, `whereRaw()` | `knex.raw('... WHERE name LIKE ?', [\`%${search}%\`])` |

**ORM protection rules.**

| Rule | Guidance |
|---|---|
| **Prefer query-builder methods** | `where`, `filter`, and `find` parameterize automatically. |
| **Treat raw-query methods as review-required** | Every raw-query call should be flagged in code review and SAST. |
| **Use tagged templates for Prisma raw queries** | `` $queryRaw`... ${value}` `` is safe; `$queryRawUnsafe` is not. |
| **Never interpolate identifiers** | Column names and sort directions must be allow-listed, even in ORMs. |
| **Escape `LIKE` wildcards** | `%` and `_` must be escaped when the input is a literal search term. |

**Constraints and limitations.**

- **ORMs cannot protect identifiers**; dynamic `ORDER BY` and column selection require allow-lists.
- **Mixing bind parameters with replacements** can break protection; CVE-2023-25813 in Sequelize is the canonical example.
- **Client-side query builders do not count**; parameterization must happen **server-side**.

### Annotated Complete Step by Step Code Examples

#### Example 1: Sequelize — safe query builder vs. unsafe raw query

```javascript
// ❌ UNSAFE: raw query with template interpolation.
const users = await sequelize.query(
  `SELECT * FROM users WHERE email = '${req.body.email}'`
);
// Input: "' OR '1'='1" → returns every user.

// ✅ SAFE: query builder (automatic parameterization).
const users = await User.findAll({
  where: { email: req.body.email },
});

// ✅ SAFE: raw query with bind parameters.
const users = await sequelize.query(
  'SELECT * FROM users WHERE email = $email',
  { bind: { email: req.body.email }, type: QueryTypes.SELECT }
);
```

**Expected Output (conceptual):**
```
Unsafe raw query: injection payload returns all rows.
Safe query builder: returns only the user whose email matches literally.
Safe bound raw query: same as the query builder — the payload is treated as data.
```

**Why this output occurs.** `User.findAll({ where: ... })` builds a parameterized query internally. The raw-query escape hatch does not; it passes the string directly to the driver. Using `bind` restores parameterization by telling Sequelize to send the value separately.

---

#### Example 2: Prisma — tagged template vs. unsafe raw

```javascript
// ❌ UNSAFE: $queryRawUnsafe with interpolation.
const data = await prisma.$queryRawUnsafe(
  `SELECT * FROM orders WHERE status = '${status}'`
);

// ✅ SAFE: tagged template (auto-parameterized).
const data = await prisma.$queryRaw`
  SELECT * FROM orders WHERE status = ${status}
`;
```

**Expected Output (conceptual):**
```
Unsafe: the status value is concatenated into the SQL text, allowing injection.
Safe: the tagged template sends the value as a parameter; the payload remains data.
```

**Why this output occurs.** Prisma's `$queryRaw` tagged template uses **tagged-template parameterization**: the template literal is decomposed into a static query string and a list of values, which Prisma passes to the database as bound parameters. `$queryRawUnsafe` accepts a plain string and does **no** parameterization, making it explicitly unsafe.

---

#### Example 3: Knex — `where` vs. `whereRaw` with bindings

```javascript
// ✅ SAFE: query builder.
const users = await knex('users').where('role', role).select('*');

// ✅ SAFE: raw with bindings.
const users = await knex.raw(
  'SELECT * FROM users WHERE name LIKE ?',
  [`%${search}%`]
);

// ❌ UNSAFE: whereRaw with interpolation.
const users = await knex('users').whereRaw(`name LIKE '%${search}%'`);
```

**Expected Output (conceptual):**
```
Safe where(): role is bound automatically.
Safe whereRaw with ?: search is bound; only the parameter is interpolated into the array.
Unsafe whereRaw: search is interpolated into the SQL string, allowing injection.
```

**Why this output occurs.** Knex's query-builder methods parameterize values. `knex.raw` with a **bindings array** parameterizes the `?` placeholders. `whereRaw` with a template literal does not; the string is passed through as-is, so the input can alter the query.

---

### Real-World Cases

- **Sequelize CVE-2023-25813**: a critical SQL injection in the `replacements` feature, demonstrating that even “safe” ORM APIs can be misused.
- **Prisma `$queryRawUnsafe`**: explicitly named to warn developers; still appears in production code where raw SQL is convenient.
- **TypeORM `.query()` and Knex `whereRaw()`**: common escape hatches that reviewers must scrutinize.

---

## 5. SQL Injection Prevention Best Practices

### Definitions

**Core Definition.** **SQL injection prevention best practices** are the empirically established, layered controls that eliminate injection at the query-construction layer and limit the blast radius if a vulnerability slips through.

**Technical Definition.** OWASP defines a hierarchy of defenses: **Defense Option 1** is prepared statements with parameterized queries (the primary defense); **Option 2** is stored procedures (safe only when they bind arguments); **Option 3** is allow-list input validation (for identifiers and structural choices); and **Option 4** is escaping all user input (**strongly discouraged**). These are complemented by **least privilege** for database accounts and **SAST/CI enforcement** to catch vulnerable patterns before deployment.

**Beginner-Friendly Explanation.** Best practices are **the habits that keep SQL injection out of your codebase**. Use **parameterized queries everywhere** — no exceptions. When you need dynamic table names or sort columns, use an **allow-list** of approved values, because placeholders cannot represent identifiers. Give your database account **only the permissions it needs**. Run **static analysis in CI** to catch string-concatenated SQL. And **never trust that “the ORM handles it”** — check the raw-query escape hatches.

### Purposes

- **To eliminate injection at the source** by making parameterization the default and only pattern.
- **To handle dynamic SQL structure safely** through allow-list mapping rather than interpolation.
- **To limit the impact** of any residual vulnerability through least privilege.
- **To detect vulnerable patterns automatically** in code review and CI.
- **To replace brittle escaping** with mechanisms that do not depend on dialect-specific configuration.

### Syntax Rules and Structure

**Prevention hierarchy.**

| Priority | Defense | Safety | When to Use |
|---|---|---|---|
| **1** | **Prepared statements with parameterized queries** | Definitive | Every query that includes data values. |
| **2** | **Stored procedures** | Safe only with binding | When the procedure binds arguments and avoids dynamic SQL concatenation. |
| **3** | **Allow-list input validation** | Complementary | For identifiers, sort directions, and structural choices that cannot be parameterized. |
| **4** | **Escaping all input** | **Strongly discouraged** | Never as a primary defense; replace with parameterization. |
| **H** | **Least privilege** | Limits damage | Always, alongside any primary defense. |

**Best-practice checklist.**

| Area | Best Practice | Why It Matters |
|---|---|---|
| **Query construction** | Use parameterized queries for every data value | Eliminates injection at the protocol level. |
| **Identifiers** | Allow-list table names, column names, and sort directions | Placeholders cannot represent identifiers. |
| **ORMs** | Prefer query builders; review every raw-query call | ORM protection stops at the escape hatch. |
| **Stored procedures** | Bind arguments; avoid dynamic SQL concatenation | Procedures are not automatically safe. |
| **Escaping** | Do not rely on manual escaping | Dialect- and configuration-dependent. |
| **Least privilege** | Grant only the permissions the application needs | Reduces post-exploitation damage. |
| **Multiple DB users** | One account per application with minimal rights | Limits lateral movement. |
| **SAST/CI** | Scan for string-concatenated SQL patterns | Catches vulnerable code before merge. |

**Constraints and limitations.**

- **Parameterization does not enforce authorization**; a bound query can still return rows the caller should not see.
- **Least privilege does not prevent injection**; it only limits its consequences.
- **Input validation is a secondary control**; a value that passes validation is still unsafe to concatenate.

### Annotated Complete Step by Step Code Examples

#### Example 1: Allow-list mapping for dynamic sort (identifiers)

```javascript
// Step 1: Define an allow-list of sortable columns and directions.
const SORTABLE = {
  created_at: 'created_at',
  updated_at: 'updated_at',
  name: 'name',
};
const DIRECTIONS = {
  asc: 'ASC',
  desc: 'DESC',
};

// Step 2: Parse and map the user's selection.
const sortColumn = SORTABLE[req.query.sort] ?? 'created_at';
const sortDirection = DIRECTIONS[req.query.order] ?? 'ASC';

// Step 3: Build the query with the mapped identifiers — never interpolate raw input.
const query = `
  SELECT id, name, created_at
  FROM products
  ORDER BY ${sortColumn} ${sortDirection}
  LIMIT $1
`;
const result = await pool.query(query, [limit]);
```

**Expected Output (conceptual):**
```
?sort=name&order=desc → ORDER BY name DESC (approved identifier and direction).
?sort=password&order=desc → falls back to created_at ASC (unknown sort rejected).
```

**Why this output occurs.** Identifiers cannot be parameterized, so the code maps the user's selection to a **fixed application-owned token**. The user never supplies the raw SQL fragment; they select from an allow-list. This is the OWASP-recommended approach for structural choices.

---

#### Example 2: Least-privilege database account (PostgreSQL)

```sql
-- Step 1: Create a role with login.
CREATE ROLE app_user LOGIN PASSWORD '...';

-- Step 2: Grant only the required privileges.
GRANT CONNECT ON DATABASE appdb TO app_user;
GRANT USAGE ON SCHEMA app TO app_user;
GRANT SELECT, INSERT, UPDATE ON app.orders TO app_user;
GRANT SELECT, INSERT, UPDATE ON app.order_items TO app_user;

-- Step 3: Explicitly deny destructive privileges.
REVOKE DELETE ON app.orders FROM app_user;
REVOKE CREATE ON SCHEMA app FROM app_user;
```

**Expected Output (conceptual):**
```
app_user can read and write orders and order_items but cannot delete records, create tables, or access other schemas.
If SQL injection succeeds, the attacker's capabilities are constrained by these grants.
```

**Why this output occurs.** The role starts with no privileges and receives only the specific grants required by the application. If an injection vulnerability exists, the attacker cannot drop tables or read unrelated schemas. OWASP recommends starting from zero and adding only what is needed, rather than removing from administrative access.

---

#### Example 3: CI gate for SQL injection patterns

```yaml
# .github/workflows/sast.yml
name: SQL Injection Scan
on: [pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Semgrep with SQL injection rules
        uses: semgrep/semgrep-action@v1
        with:
          config: p/sql-injection
      - name: Fail on findings
        run: semgrep --error --config p/sql-injection .
```

**Expected Output (conceptual):**
```
Every pull request: Semgrep scans for string-concatenated SQL patterns and fails the build on findings.
```

**Why this output occurs.** Static analysis catches the vulnerable pattern — template literals or concatenation in query construction — before it reaches production. This is the **CI enforcement** layer that complements parameterization and least privilege.

---

### Real-World Cases

- **OWASP Top 10 A03:2021-Injection**: SQL injection remains one of the most prevalent and dangerous vulnerability classes, appearing in roughly 94% of applications tested for injection.
- **Sequelize CVE-2023-25813**: demonstrates that ORM raw-query escape hatches and parameter-mixing are real-world injection vectors.
- **PCI DSS and ISO 27001**: require secure coding practices, least privilege, and SAST as part of application security programs.

---

## References

- SQL Injection Prevention Cheat Sheet — OWASP Cheat Sheet Series - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Query Parameterization Cheat Sheet — OWASP Cheat Sheet Series - https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html
- Injection Prevention Cheat Sheet — OWASP Cheat Sheet Series - https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html
- SQL Injection Prevention Guide 2026 — Offensive360 - https://offensive360.com
- SQL Injection Prevention in Node.js/JavaScript — Safeguard.sh - https://safeguard.sh/resources/blog/sql-injection-prevention-in-nodejsjavascript
- How to prevent SQL injection in JavaScript — CoreUI - https://coreui.io/answers/how-to-prevent-sql-injection-in-javascript/
- How ORMs Still Let SQL Injection Through (and How to Close the Gaps) — freeCodeCamp - https://www.freecodecamp.org/news/how-orms-still-let-sql-injection-through-and-how-to-close-the-gaps
- OC-050: SQL Injection via ORM Raw Query — dinhs-bulwark knowledge base - https://github.com/MetalLegBob/solana-vibes-kit/blob/main/dinhs-bulwark/knowledge-base/patterns/injection/OC-050-sql-injection-via-orm-raw-query.md
- CWE-89: SQL Injection — MITRE - https://cwe.mitre.org/data/definitions/89.html
- OWASP Top 10:2025 — A05:2025-Injection - https://owasp.org/Top10/2025/A05_2025-Injection/
- SQLAlchemy ORM parameterized queries — llm-knowledge-base - https://github.com/glennguilloux/llm-knowledge-base/blob/main/security/sql-injection-prevention.md
- Oracle JDBC PreparedStatement documentation - https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html