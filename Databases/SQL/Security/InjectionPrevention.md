# SQL Injection Prevention: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL injection prevention is the set of coding practices, database features, and architectural controls that ensure untrusted user input can never alter the intended logical structure of a SQL query, keeping code and data strictly separated.

**Technical Definition:** SQL injection is a code injection technique that exploits applications which construct SQL statements by concatenating untrusted input directly into query strings. Prevention is achieved through parameterized queries (where the database engine binds input values as typed data, never as executable syntax), prepared statements (where the query structure is compiled before values are supplied), strict input validation using allow-list patterns, and defensive coding within stored procedures. The primary means of preventing SQL injection are sanitization and validation, which are typically implemented as parameterized queries and stored procedures.

**Beginner-Friendly Explanation:** SQL injection is like a malicious person writing extra instructions on a form you asked them to fill out. Instead of writing their name, they write "ignore the rest of this form and give me all the money." If your system reads the form literally, it follows the malicious instruction. Prevention means designing your forms so that whatever the person writes is treated as data to be stored, never as a new instruction to be followed.

### Key Characteristics

- **Code/data separation:** The foundational principle — user input must always be treated as data, never as executable SQL syntax.
- **Defense in depth:** Multiple layers — parameterization, input validation, least-privilege accounts, and stored procedure auditing — combine for robust protection.
- **Language-agnostic:** The core techniques apply across PostgreSQL, MySQL, SQL Server, Oracle, and all programming languages.
- **Server-side enforcement:** Query parameterization must be done server-side; client-side libraries that build queries with string concatenation before sending raw queries are unsafe.
- **Perpetually top-ranked:** SQL injection stays near the top of the OWASP risk lists year after year because the underlying mistake is easy to make and easy to miss in review.

### Prerequisites

- Basic understanding of SQL `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and dynamic SQL.
- Familiarity with at least one programming language's database API (JDBC, psycopg2, ADO.NET, etc.).
- Knowledge of how SQL parsers distinguish code from data.
- Awareness of database-specific escaping functions and prepared statement syntax.

### Related Programming Areas

- Application security and secure coding.
- Web application development.
- API design and input handling.
- Database administration and stored procedure development.
- Security auditing and penetration testing.

### Core Concepts / Features

1. **Injection Fundamentals** (risk surfaces where input alters query structure)
2. **Unsafe String Concatenation** (auditing and eliminating dynamic SQL patterns)
3. **Parameterized Queries** (type-safe data separation via parameter bindings)
4. **Prepared Statements** (`PREPARE`, `EXECUTE`, pre-compiled query plans)
5. **Input Validation** (allow-listing, pattern matching, escaping functions)
6. **Stored Procedure Considerations** (auditing dynamic SQL within routines)


## Core Concept 1: Injection Fundamentals

### Definitions

**Core Definition:** SQL injection is a vulnerability that arises when untrusted user input is incorporated into a SQL query in a way that allows the input to alter the query's logical structure, changing what the query does rather than merely what data it operates on.

**Technical Definition:** A SQL injection vulnerability arises when the original SQL query can be altered to form an altogether different query. Execution of this altered query may result in information leaks or data modification. The injection process works by prematurely terminating a text string and appending a new command. SQL injection vulnerabilities arise in applications where elements of a SQL query originate from an untrusted source. Without precautions, the untrusted data may maliciously alter the query, resulting in information leaks or data modification.

**Beginner-Friendly Explanation:** Imagine a login form that asks for your username. The database query behind it is: `SELECT * FROM users WHERE username = 'your_input' AND password = 'your_password'`. If you type `' OR '1'='1` as the username, the query becomes: `SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '...'`. Because `'1'='1'` is always true, the query returns every row in the table — and the attacker is logged in without a valid password.

### Purposes

- **To understand the attack surface** where untrusted input can alter query structure.
- **To recognize the classic patterns** of SQL injection (login bypass, UNION extraction, blind injection).
- **To identify vulnerable code patterns** during security audits and code reviews.
- **To appreciate why string concatenation is dangerous** and why parameterization is essential.

### Syntax Rules and Structure

#### Complete General Syntax (Vulnerable Pattern)

```sql
-- VULNERABLE: Direct string concatenation
SELECT * FROM db_user
WHERE username='<USERNAME>' AND password='<PASSWORD>'
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<USERNAME>` | Untrusted user input inserted directly into the query string. |
| `<PASSWORD>` | Untrusted user input inserted directly into the query string. |
| `'` | The delimiter that the attacker can prematurely close. |

#### Syntax Rules

- **Input must never be treated as code:** Any input that originates from a user, API, file, or external system is untrusted.
- **String delimiters are the attack vector:** Attackers inject by closing a string literal and appending SQL syntax.
- **Comment characters (`--`, `/* */`) are used** to remove the remainder of the original query.
- **Boolean logic (`OR '1'='1`)** is used to make the WHERE clause always true.

#### Constraints and Limitations

- **Not just login forms:** Any query that concatenates input is vulnerable — search, sort, filter, report parameters.
- **Second-order injection:** Malicious input stored in the database may be used unsafely in a later query.
- **Blind injection:** Even when no output is returned, attackers can infer data using time delays or boolean conditions.

### Annotated Code Examples

#### Example 1: Java — Vulnerable Login Query

```java
// VULNERABLE: Direct string concatenation in Java
String username = request.getParameter("username");
String password = request.getParameter("password");

String query = "SELECT * FROM db_user WHERE username = '" + username
             + "' AND password = '" + password + "'";

Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);

if (rs.next()) {
    // Authentication succeeds
}
```

**Attack Input:** `username = "validuser' OR '1'='1"` and `password = "anything"`

**Resulting Query:**

```sql
SELECT * FROM db_user WHERE username = 'validuser' OR '1'='1' AND password = 'anything'
```

**Why This Works:** The `OR '1'='1'` condition makes the WHERE clause always true. If `validuser` is a valid username, the query yields the `validuser` record. The password is never checked because `username='validuser'` is true; consequently, the items after the OR are not tested.

#### Example 2: SQL — Login Bypass Without Valid Username

```sql
-- Attacker supplies: username = "anything" and password = "' OR '1'='1"
SELECT * FROM db_user
WHERE username='anything' AND password='' OR '1'='1'
```

**Expected Result:** The `'1'='1'` condition always evaluates to true, causing the query to yield every row in the database. The attacker is authenticated without needing a valid username or password.

**Why This Works:** The attacker prematurely terminated the password string and appended a boolean condition that is always true. The database engine cannot distinguish between the original query logic and the injected condition because they are part of the same string.

### Real-World Cases

- **Login bypass:** Attackers log in as administrators without valid credentials.
- **Data exfiltration:** Using `UNION SELECT` to extract data from other tables.
- **Data modification:** Injecting `UPDATE` or `DELETE` statements to corrupt data.
- **Command execution:** In some configurations, SQL injection can lead to operating system command execution.

### References

- OWASP: SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- SEI CERT: SQL Injection — https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88503449
- OWASP: SQL Injection — https://owasp.org/www-community/attacks/SQL_Injection


## Core Concept 2: Unsafe String Concatenation

### Definitions

**Core Definition:** Unsafe string concatenation is the coding pattern where untrusted input variables are stitched directly into static SQL command strings, creating an injection vulnerability because the input becomes part of the executable SQL syntax.

**Technical Definition:** Dynamic SQL (the construction of SQL queries by concatenation of strings) opens the door to SQL injection vulnerabilities. The advice is to avoid using string concatenation to create SQL query strings, even when using parameterized queries, especially if the concatenation involves building any items in the WHERE clause. In Oracle PL/SQL, `EXECUTE IMMEDIATE` with concatenated input is a common vulnerable pattern. In SQL Server, `EXEC` and `sp_executesql` with concatenated input are vulnerable.

**Beginner-Friendly Explanation:** String concatenation is like writing a sentence by gluing together pre-printed words and whatever the user hands you. If the user hands you a sentence fragment that changes the meaning of your sentence, you have a problem. The safe approach is to write the sentence with blanks, and have the database fill in the blanks with the user's data — separately from the sentence itself.

### Purposes

- **To audit existing code** for concatenation patterns that create injection vulnerabilities.
- **To eliminate dynamic SQL** where possible, replacing it with parameterized queries.
- **To identify the specific functions** (`EXECUTE IMMEDIATE`, `EXEC`, `sp_executesql`, `PREPARE` with concatenation) that are dangerous when misused.
- **To understand why even "sanitized" concatenation is fragile** compared to parameterization.

### Syntax Rules and Structure

#### Complete General Syntax (Vulnerable Oracle PL/SQL)

```sql
-- VULNERABLE: EXECUTE IMMEDIATE with concatenated input
CREATE OR REPLACE PROCEDURE get_recent_record(user_name IN VARCHAR2) IS
    query VARCHAR2(4000);
    rec   VARCHAR2(4000);
BEGIN
    query := 'SELECT value FROM records WHERE name = ''' || user_name || '''';
    EXECUTE IMMEDIATE query INTO rec;
    DBMS_OUTPUT.PUT_LINE('Record: ' || rec);
END;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `query := '...' \|\| user_name \|\| '...'` | Concatenates user input directly into the SQL string. |
| `EXECUTE IMMEDIATE query` | Executes the dynamically constructed query. |

#### Complete General Syntax (Vulnerable SQL Server)

```sql
-- VULNERABLE: EXEC with concatenated input
CREATE PROCEDURE GetProducts @orderby NVARCHAR(MAX) AS
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    SET @sql = 'SELECT * FROM Products ORDER BY ' + @orderby;
    EXEC sp_executesql @sql;
END;
```

#### Syntax Rules

- **Never concatenate values:** Table and column names are the only legitimate exceptions, and even then they must be validated against an allow-list.
- **Dynamic SQL with concatenation is the root cause:** If you must use dynamic SQL, use bind variables (parameters) rather than concatenation.
- **Even "safe" concatenation is risky:** Escaping functions like `QUOTE_LITERAL` help, but parameterization is always safer.

#### Constraints and Limitations

- **Escaping is fragile:** Escaping functions must be applied correctly to every input, in every query, every time. One missed call means a vulnerability.
- **Dynamic PL/SQL is more dangerous than dynamic SQL:** The impact of SQL injection vulnerabilities in dynamic PL/SQL is even more serious than in dynamic SQL because with dynamic PL/SQL, the attacker can execute arbitrary PL/SQL blocks.
- **`QUOTE_LITERAL` is a workaround:** It helps when dynamic SQL is unavoidable, but `USING` clauses (bind variables) are better and faster.

### Annotated Code Examples

#### Example 1: Oracle PL/SQL — Vulnerable Procedure

```sql
-- VULNERABLE: Procedure vulnerable to SQL injection
CREATE OR REPLACE PROCEDURE get_recent_record(user_name IN VARCHAR2) IS
    query VARCHAR2(4000);
    rec   VARCHAR2(4000);
BEGIN
    query := 'SELECT value FROM records WHERE name = ''' || user_name || '''';
    EXECUTE IMMEDIATE query INTO rec;
    DBMS_OUTPUT.PUT_LINE('Record: ' || rec);
END;
```

**Attack Input:** `user_name = "Andy' OR '1'='1"`

**Resulting Query:**

```sql
SELECT value FROM records WHERE name = 'Andy' OR '1'='1'
```

**Why This Works:** The concatenated input closes the string literal and appends `OR '1'='1'`, making the WHERE clause always true. The procedure returns every record in the table, not just Andy's.

#### Example 2: PostgreSQL PL/pgSQL — Vulnerable Dynamic SQL

```sql
-- VULNERABLE: PL/pgSQL function with EXECUTE and concatenation
CREATE OR REPLACE FUNCTION get_user_data(user_id TEXT)
RETURNS TABLE(name TEXT, email TEXT) AS $$
BEGIN
    RETURN QUERY EXECUTE
        'SELECT name, email FROM users WHERE id = ''' || user_id || '''';
END;
$$ LANGUAGE plpgsql;
```

**Attack Input:** `user_id = "1' OR '1'='1"`

**Resulting Query:**

```sql
SELECT name, email FROM users WHERE id = '1' OR '1'='1'
```

**Why This Works:** The `EXECUTE` statement executes the concatenated query. The `OR '1'='1'` condition makes the WHERE clause always true, returning all users' data. If you went over to using `EXECUTE`, you *would* need `quote_literal` to be safe, because then you're synthesizing the complete statement.

### Real-World Cases

- **Legacy codebases:** Older applications built before parameterized queries were standard.
- **Reporting tools:** Ad-hoc query builders that concatenate filter values.
- **Stored procedures:** Procedures that build dynamic SQL for flexible search criteria.
- **ORM-generated SQL:** Some ORMs generate concatenated SQL under the hood; audit their output.

### References

- Oracle: Avoiding SQL Injection in PL/SQL — https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/dynamic-sql.html
- PostgreSQL: Preventing SQL Injection in PL/pgSQL — https://www.postgresql.org/docs/current/plpgsql-statements.html
- OWASP: SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html


## Core Concept 3: Parameterized Queries

### Definitions

**Core Definition:** A parameterized query (also called a query with bind variables) is a SQL statement in which input values are supplied through dedicated parameter placeholders (`?`, `:name`, `@name`) rather than being embedded in the query string, ensuring the database engine always treats them as data, never as executable code.

**Technical Definition:** SQL injection is best prevented through the use of parameterized queries. Parameterized queries force the developer to define all SQL code first and pass in each parameter to the query later. If database queries use this coding style, the database will always distinguish between code and data, regardless of what user input is supplied. Also, prepared statements ensure that an attacker cannot change the intent of a query, even if SQL commands are inserted by an attacker. Dynamic SQL can be parameterized using bind variables, to ensure the dynamically constructed SQL is secure.

**Beginner-Friendly Explanation:** A parameterized query is like a form with blank fields. The form itself (the SQL structure) is printed in advance. You fill in the blanks with data, but the database knows that whatever you write in a blank is just data — not a new instruction. Even if you write "ignore the rest of the form," the database treats it as text to be stored, not as something to execute.

### Purposes

- **To enforce type-safe data separation** by passing inputs as dedicated parameter bindings rather than executable code segments.
- **To eliminate the root cause of SQL injection** — the database can never confuse code with data.
- **To improve performance** through query plan reuse (the database caches the execution plan).
- **To work consistently across all programming languages** and database systems.

### Syntax Rules and Structure

#### Complete General Syntax (JDBC — Java)

```java
String custname = request.getParameter("customerName");
String query = "SELECT account_balance FROM user_data WHERE user_name = ?";
PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, custname);
ResultSet results = pstmt.executeQuery();
```

#### Complete General Syntax (psycopg2 — Python)

```python
cursor.execute(
    "SELECT account_balance FROM user_data WHERE user_name = %s",
    (customer_name,)
)
```

#### Complete General Syntax (ADO.NET — C#)

```csharp
string query = "SELECT account_balance FROM user_data WHERE user_name = @CustomerName";
using (SqlCommand cmd = new SqlCommand(query, connection)) {
    cmd.Parameters.AddWithValue("@CustomerName", customerName);
    using (SqlDataReader reader = cmd.ExecuteReader()) {
        // Process results
    }
}
```

#### Complete General Syntax (Oracle PL/SQL — Bind Variables)

```sql
-- SAFE: Using bind variables in dynamic SQL
EXECUTE IMMEDIATE
    'SELECT value FROM records WHERE name = :1'
    INTO rec
    USING user_name;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `?`, `%s`, `@name`, `:1` | Parameter placeholders in the query string. |
| `setString(1, value)` / `(value,)` / `Parameters.AddWithValue` / `USING value` | The mechanism for binding the actual value to the placeholder. |
| The query string never contains the user input. | The input is supplied separately as a typed parameter. |

#### Syntax Rules

- **Placeholder syntax varies by database and driver:** `?` (JDBC, ADO.NET), `%s` (psycopg2), `:name` (Oracle, SQLite), `@name` (SQL Server).
- **The query string is static:** It does not change based on input. Only the parameter values change.
- **Parameterized queries work for all value types:** strings, numbers, dates, booleans, NULL.
- **Table and column names cannot be parameterized:** They must be validated against an allow-list if they come from user input.

#### Constraints and Limitations

- **Cannot parameterize identifiers:** Table names, column names, and sort directions cannot use parameter placeholders. Use allow-list validation for these.
- **Some ORMs and frameworks build queries with string concatenation:** Client-side query parameterization libraries may just build queries with string concatenation before sending raw queries to the server. Ensure that query parameterization is done server-side.
- **Dynamic SQL with `EXECUTE` requires bind variables:** When constructing dynamic SQL, use `USING` (Oracle/PostgreSQL) or `sp_executesql` (SQL Server) with parameters.

### Annotated Code Examples

#### Example 1: Java JDBC — Safe Prepared Statement

```java
// SAFE: Parameterized query with JDBC PreparedStatement
String custname = request.getParameter("customerName");
String query = "SELECT account_balance FROM user_data WHERE user_name = ?";

PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, custname);
ResultSet results = pstmt.executeQuery();
```

**Attack Input:** `customerName = "admin' OR '1'='1"`

**Result:** The parameter value is treated as a literal string. The query becomes `SELECT account_balance FROM user_data WHERE user_name = 'admin'' OR ''1''=''1'` — the injected SQL is part of the string value, not a new query condition. No rows are returned (unless a user is literally named `admin' OR '1'='1`).

**Why This Works:** The `?` placeholder is bound to the value of `custname`. The database engine knows that the parameter is data, not SQL code. The `PreparedStatement` compiles the query plan with the placeholder before the value is supplied.

#### Example 2: Oracle PL/SQL — Safe Bind Variables

```sql
-- SAFE: Bind variable with EXECUTE IMMEDIATE
CREATE OR REPLACE PROCEDURE get_recent_record(user_name IN VARCHAR2) IS
    query VARCHAR2(4000);
    rec   VARCHAR2(4000);
BEGIN
    query := 'SELECT value FROM records WHERE name = :1';
    EXECUTE IMMEDIATE query INTO rec USING user_name;
    DBMS_OUTPUT.PUT_LINE('Record: ' || rec);
END;
```

**Why This Works:** The `:1` placeholder is bound to `user_name` using the `USING` clause. Even if `user_name` contains SQL syntax, it is treated as a literal value. The query structure cannot be altered by the input.

### Real-World Cases

- **Web application login:** Authenticating users with parameterized username/password checks.
- **Search functionality:** Filtering results by user-supplied keywords.
- **API endpoints:** Accepting query parameters for data retrieval.
- **Batch processing:** Passing arrays of values as bind parameters for bulk operations.

### References

- OWASP: Query Parameterization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html
- OWASP: SQL Injection Prevention Cheat Sheet (Defense Option 1) — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Oracle: Using Bind Variables in Dynamic SQL — https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/dynamic-sql.html


## Core Concept 4: Prepared Statements

### Definitions

**Core Definition:** A prepared statement is a pre-compiled SQL statement whose query plan is generated by the database before any input values are supplied; values are bound to placeholders at execution time, ensuring the executable logic is strictly defined and cannot be altered by input.

**Technical Definition:** `PREPARE` creates a prepared statement. A prepared statement is a server-side object that can be used to optimize performance. When the `PREPARE` statement is executed, the specified statement is parsed, analyzed, and rewritten. When an `EXECUTE` command is subsequently issued, the prepared statement is planned and executed. This division of labor avoids repetitive parse analysis work, while allowing the execution plan to depend on the specific parameter values supplied. In PostgreSQL, prepared statements can also be created via the `PREPARE` SQL statement; a prepared statement takes parameters (like `$1`, `$2`) which are bound at execution time.

**Beginner-Friendly Explanation:** A prepared statement is like a recipe that the chef (the database) has already read and practiced. When you order, the chef just fills in the specific ingredients (your data) at the last moment. Because the chef already knows the recipe structure, no matter what ingredient name you say, the chef will not accidentally add a completely different dish to the menu. The recipe (query structure) is fixed before you speak.

### Purposes

- **To utilize pre-compiled query plans** (`PREPARE`, `EXECUTE`) that strictly define the executable logic before data variables are bound at runtime.
- **To improve performance** by avoiding repetitive parsing and planning for repeated queries.
- **To enforce the strongest form of code/data separation** at the database protocol level.
- **To support dynamic SQL safely** using bind parameters within `EXECUTE` statements.

### Syntax Rules and Structure

#### Complete General Syntax (PostgreSQL)

```sql
-- Step 1: Prepare a statement
PREPARE statement_name (data_type [, ...]) AS
    SELECT column_list FROM table_name WHERE column = $1;

-- Step 2: Execute with parameter values
EXECUTE statement_name (value1 [, ...]);

-- Step 3: Deallocate when no longer needed
DEALLOCATE statement_name;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `PREPARE statement_name` | Names the prepared statement. |
| `(data_type, ...)` | Declares the data types of the parameters. |
| `$1`, `$2`, ... | Parameter placeholders in the query. |
| `EXECUTE statement_name (value, ...)` | Binds values and executes. |
| `DEALLOCATE` | Releases the prepared statement and its resources. |

#### Complete General Syntax (MySQL)

```sql
-- Step 1: Prepare a statement from a string
PREPARE stmt FROM 'SELECT * FROM products WHERE id = ?';

-- Step 2: Set the parameter value
SET @id = 100;

-- Step 3: Execute
EXECUTE stmt USING @id;

-- Step 4: Deallocate
DEALLOCATE PREPARE stmt;
```

#### Complete General Syntax (SQL Server)

```sql
-- sp_executesql with parameters
EXEC sp_executesql
    N'SELECT * FROM Products WHERE CategoryID = @CatID',
    N'@CatID int',
    @CatID = @CategoryID;
```

#### Syntax Rules

- **PostgreSQL:** `PREPARE` parses, analyzes, and rewrites the statement. `EXECUTE` plans and executes. `$1`, `$2` placeholders are bound to values in `EXECUTE`.
- **MySQL:** `PREPARE stmt FROM 'query_string'` parses the query. `EXECUTE stmt USING @var` binds session variables. `?` placeholders are used.
- **SQL Server:** `sp_executesql` accepts a parameterized query string, a parameter definition string, and the parameter values. Always use `sp_executesql` with parameters rather than `EXEC` with concatenation.
- **PREPARE can be used for optimization:** Even if the statement is executed only once, `PREPARE` can be used to verify the syntax and determine parameter types.

#### Constraints and Limitations

- **Prepared statements are session-scoped:** In PostgreSQL, a prepared statement lasts for the current session only. In MySQL, the prepared statement is also session-scoped.
- **Not all statements can be prepared:** DDL statements and some utility statements cannot be prepared in some databases.
- **Plan caching:** In some databases, the query plan may be cached based on the parameter values; extreme parameter value skew can cause suboptimal plans.
- **SQL injection still possible with concatenation in EXECUTE:** If you use `EXECUTE` with a dynamically built string (not a `PREPARE`d statement), you must use bind parameters — `USING` in Oracle/PostgreSQL, `sp_executesql` with parameters in SQL Server.

### Annotated Code Examples

#### Example 1: PostgreSQL — PREPARE and EXECUTE

```sql
-- Prepare a statement for repeated execution
PREPARE get_user (TEXT) AS
    SELECT name, email FROM users WHERE username = $1;

-- Execute with a safe parameter value
EXECUTE get_user('alice');

-- Execute with a malicious parameter value
EXECUTE get_user('alice'' OR ''1''=''1');

-- Deallocate
DEALLOCATE get_user;
```

**Expected Output (first execution):**

```
 name  |       email
-------+-------------------
 Alice | alice@example.com
```

**Expected Output (second execution):**

```
 name | email
------+-------
(0 rows)
```

**Why This Works:** The `$1` placeholder is bound to the parameter value. Even when the input contains SQL syntax (`' OR '1'='1`), it is treated as a literal string value. The query returns no rows because no username literally matches `alice' OR '1'='1`.

#### Example 2: SQL Server — sp_executesql with Parameters

```sql
-- SAFE: Parameterized dynamic SQL with sp_executesql
DECLARE @sql NVARCHAR(MAX);
DECLARE @CatID INT = 1;

SET @sql = N'SELECT * FROM Products WHERE CategoryID = @CatID';

EXEC sp_executesql
    @sql,
    N'@CatID int',
    @CatID = @CatID;
```

**Why This Works:** The `@CatID` parameter is declared in the second argument and bound in the third. Even if `@CatID` were derived from user input, it would be treated as an integer value, not as SQL syntax. This is the recommended way to execute dynamic SQL safely in SQL Server.

### Real-World Cases

- **High-throughput OLTP:** Using prepared statements for frequently executed queries to benefit from plan caching.
- **Dynamic search:** Building search queries with variable filter values using `sp_executesql` or `EXECUTE ... USING`.
- **Stored procedure internals:** Using `PREPARE`/`EXECUTE` or `sp_executesql` within procedures that must construct dynamic SQL.
- **Batch operations:** Preparing an `INSERT` statement once and executing it many times with different parameter values.

### References

- PostgreSQL: PREPARE — https://www.postgresql.org/docs/current/sql-prepare.html
- PostgreSQL: EXECUTE — https://www.postgresql.org/docs/current/sql-execute.html
- MySQL: PREPARE Statement — https://dev.mysql.com/doc/refman/8.0/en/prepare.html
- SQL Server: sp_executesql — https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-executesql-transact-sql
- SQL Server: Writing Secure Dynamic SQL — https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-injection


## Core Concept 5: Input Validation

### Definitions

**Core Definition:** Input validation is the practice of inspecting and filtering untrusted input before it reaches the database, using allow-list patterns, data type enforcement, and character escaping functions to reject or neutralize malicious content.

**Technical Definition:** Input validation can be used to detect unauthorized input before it is passed to the SQL query. Developers frequently perform black list validation in order to try to detect attack characters and patterns like the `'` character or the string `1=1`, but this is a poor security strategy because black lists are fragile and incomplete. Always use an allowlist approach, which means you only accept "known good" input, instead of a blocklist (where you specifically look for bad input) because it's impossible to think of a complete list of potentially dangerous input. Do this work on the server side. Character escaping functions such as Oracle's `DBMS_ASSERT` package and PostgreSQL's `quote_literal` provide a last line of defense when dynamic SQL is unavoidable.

**Beginner-Friendly Explanation:** Input validation is like a bouncer at a club who checks IDs. Instead of trying to remember every possible fake ID (black-list), the bouncer only accepts IDs that match a known good format (allow-list). If your ID doesn't look exactly right, you don't get in. This is much more reliable than trying to spot every possible fake.

### Purposes

- **To enforce defensive server-side sanitization** of all input before it reaches the database.
- **To apply strict pattern matching** so that inputs conform to expected formats (e.g., a phone number must be exactly 10 digits).
- **To enforce data type validation** so that numeric fields receive only numbers.
- **To leverage character escaping functions** (`QUOTE_LITERAL`, `DBMS_ASSERT`) as a defense-in-depth measure when dynamic SQL is unavoidable.

### Syntax Rules and Structure

#### Complete General Syntax (Allow-List Validation)

```python
import re

def validate_sort_column(user_input):
    allowed = {'name', 'email', 'created_at', 'status'}
    if user_input not in allowed:
        raise ValueError("Invalid sort column")
    return user_input
```

#### Complete General Syntax (Oracle DBMS_ASSERT — ENQUOTE_LITERAL)

```sql
-- Enquote a string literal to prevent injection
EXECUTE IMMEDIATE
    'SELECT value FROM records WHERE name = '
    || DBMS_ASSERT.ENQUOTE_LITERAL(user_name)
    INTO rec;
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `ENQUOTE_LITERAL(str)` | Adds leading and trailing single quotes to a string literal; verifies that all single quotes except leading and trailing characters are paired with adjacent single quotes. |
| `ENQUOTE_NAME(str)` | Encloses the provided string in double quotes and checks that the result is a valid SQL identifier. |
| `SIMPLE_SQL_NAME(str)` | Verifies that the input string is a simple SQL name. |
| `QUALIFIED_SQL_NAME(str)` | Verifies that the input string is a qualified SQL name. |
| `SCHEMA_NAME(str)` | Verifies that the input string is an existing schema name. |
| `SQL_OBJECT_NAME(str)` | Verifies that the input parameter string is a qualified SQL identifier of an existing SQL object. |

#### Complete General Syntax (PostgreSQL QUOTE_LITERAL)

```sql
-- Safely quote a string literal for dynamic SQL
EXECUTE
    'INSERT INTO fx VALUES (' || quote_literal(my_var) || ')';
```

#### Syntax Rules

- **Allow-list, not block-list:** Only accept input that strictly conforms to a specification. Reject everything else.
- **Validate on the server side:** Client-side validation is for user experience; server-side validation is for security. Always do both.
- **Data type enforcement:** Use `CAST` or `TRY_CAST` to ensure numeric fields receive only numbers.
- **Escaping is a last resort:** `QUOTE_LITERAL` and `DBMS_ASSERT` should only be used when parameterization is impossible. `quote_literal` and `quote_ident` are sufficient, but the `USING` clause is better and faster.

#### Constraints and Limitations

- **Black-list validation is fragile:** Trying to detect attack characters is impossible to do completely. Attackers use encoding tricks, comments, and unusual syntax to bypass black-lists.
- **DBMS_ASSERT is Oracle-specific:** Other databases have different escaping functions (`quote_literal` in PostgreSQL, `QUOTENAME` in SQL Server).
- **Escaping functions are not a substitute for parameterization:** They are a defense-in-depth measure, not the primary defense.
- **Input validation does not protect against second-order injection:** Data that was validated on input may be used unsafely in a later query.

### Annotated Code Examples

#### Example 1: Oracle — DBMS_ASSERT.ENQUOTE_LITERAL

```sql
-- SAFE: Using DBMS_ASSERT.ENQUOTE_LITERAL for dynamic SQL
CREATE OR REPLACE PROCEDURE get_recent_record(user_name IN VARCHAR2) IS
    query VARCHAR2(4000);
    rec   VARCHAR2(4000);
BEGIN
    query := 'SELECT value FROM records WHERE name = '
          || DBMS_ASSERT.ENQUOTE_LITERAL(user_name);
    EXECUTE IMMEDIATE query INTO rec;
    DBMS_OUTPUT.PUT_LINE('Record: ' || rec);
END;
```

**Why This Works:** `DBMS_ASSERT.ENQUOTE_LITERAL` wraps the input in single quotes and verifies that all internal single quotes are properly paired. This prevents a malicious user from injecting text between an opening quotation mark and its corresponding closing quotation mark.

#### Example 2: Python — Allow-List Validation for Sort Column

```python
# SAFE: Allow-list validation for dynamic ORDER BY
import re

ALLOWED_SORT_COLUMNS = {'name', 'email', 'created_at', 'status'}

def get_sort_column(user_input):
    if user_input not in ALLOWED_SORT_COLUMNS:
        raise ValueError("Invalid sort column: " + user_input)
    return user_input

sort_col = get_sort_column(request.args.get('sort', 'created_at'))
query = f"SELECT * FROM users ORDER BY {sort_col} DESC"
# Now sort_col is guaranteed to be one of the known-safe column names
```

**Why This Works:** The user input is checked against a set of known-safe column names. If the input does not match exactly, an error is raised. Even though `sort_col` is concatenated into the query, it cannot contain SQL injection because it is one of four hard-coded values.

### Real-World Cases

- **Numeric IDs:** Validating that a user-supplied ID is a positive integer.
- **Sort parameters:** Allow-listing sort columns for report generation.
- **Date ranges:** Validating date format (YYYY-MM-DD) before parsing.
- **Email addresses:** Using regex patterns to validate email format.
- **Pagination:** Validating page number and page size as positive integers.

### References

- OWASP: SQL Injection Prevention Cheat Sheet (Defense Option 3: Allow-list Input Validation) — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Oracle: DBMS_ASSERT Package — https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_ASSERT.html
- Microsoft: Input Validation — https://learn.microsoft.com/en-us/previous-versions/msp-n-p/ff648339(v=pandp.10)
- CWE-20: Improper Input Validation — https://cwe.mitre.org/data/definitions/20.html


## Core Concept 6: Stored Procedure Considerations

### Definitions

**Core Definition:** Stored procedure considerations are the security audit and coding practices required to ensure that stored procedures — especially those that construct and execute dynamic SQL internally — do not reintroduce SQL injection vulnerabilities through escalated execution contexts.

**Technical Definition:** Using stored procedures can prevent SQL injection because the user input is no longer used to build the query dynamically. Since a stored procedure is a group of precompiled SQL statements and the procedure accepts input as parameters, a dynamic query is avoided. The exception to this is where the stored procedure takes a string as input and uses this string to build the query without validating it. While this is more difficult to exploit, this scenario still often leads to successful SQL injection. Stored procedures have the same effect as the use of prepared statements when implemented safely. Both techniques have the same effectiveness in preventing SQL injection. If dynamic queries in your stored procedures can't be avoided, validate or properly escape all user-supplied input to the dynamic query.

**Beginner-Friendly Explanation:** A stored procedure is like a recipe that the database has already learned. If the procedure just uses the ingredients you give it, it is safe. But if the procedure itself follows a sub-recipe that says "mix in whatever the user says," you are back to the same problem. You must audit the procedure's internal instructions to make sure it does not accidentally create a new injection point.

### Purposes

- **To audit wrapper routines** to ensure internal dynamic queries use parameters and do not accidentally reintroduce injection vulnerabilities.
- **To understand that stored procedures are not automatically safe** — they can be vulnerable if they construct dynamic SQL.
- **To ensure that stored procedures do not accidentally reintroduce injection vulnerabilities** through escalated runtime execution states (e.g., `SECURITY DEFINER`).
- **To apply the same parameterization discipline** inside stored procedures as in application code.

### Syntax Rules and Structure

#### Complete General Syntax (SQL Server — Vulnerable Stored Procedure)

```sql
-- VULNERABLE: Dynamic SQL inside a stored procedure
CREATE PROCEDURE GetProducts @orderby NVARCHAR(MAX) AS
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    SET @sql = 'SELECT * FROM Products ORDER BY ' + @orderby;
    EXEC sp_executesql @sql;
END;
```

**Attack Input:** `@orderby = 'ProductID DROP TABLE Products'`

**Resulting Query:**

```sql
SELECT * FROM Products ORDER BY ProductID DROP TABLE Products
```

**Why This Works:** Although lesser known, SQL injection is also possible if the stored procedure itself constructs dynamic SQL and executes it with the `exec` or `sp_executesql` statements. A malicious user entering `@orderby='ProductID DROP TABLE Products'` can execute the injected DDL.

#### Complete General Syntax (SQL Server — Safe Stored Procedure)

```sql
-- SAFE: Parameterized dynamic SQL with sp_executesql
CREATE PROCEDURE GetProducts @orderby NVARCHAR(50) AS
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    -- Validate @orderby against an allow-list
    IF @orderby NOT IN ('ProductID', 'ProductName', 'UnitPrice', 'CategoryID')
        RAISERROR('Invalid sort column', 16, 1);

    SET @sql = N'SELECT * FROM Products ORDER BY ' + QUOTENAME(@orderby);
    EXEC sp_executesql @sql;
END;
```

#### Syntax Rules

- **Stored procedures are not automatically safe:** They are safe only if they do not construct dynamic SQL with concatenated input.
- **`sp_executesql` with parameters is safe:** When you use `sp_executesql`, parameterize the queries. For more information, see SQL injection.
- **`QUOTENAME` protects object names:** When dealing with dynamic SQL, always use `QUOTENAME` to protect object names. `QUOTENAME` protects you from SQL injection via object name when just square brackets won't.
- **Avoid dynamic SQL entirely where possible:** The safest stored procedure is one that uses only static SQL with parameterized inputs.

#### Constraints and Limitations

- **`EXEC` with concatenation is vulnerable:** Even inside a stored procedure, `EXEC` with concatenated strings is vulnerable.
- **`sp_executesql` with concatenated parameters is vulnerable:** If you concatenate the parameters into the query string before passing to `sp_executesql`, you are still vulnerable.
- **`SECURITY DEFINER` escalates the risk:** A vulnerable stored procedure running with `SECURITY DEFINER` executes with the owner's privileges, potentially giving the attacker elevated access.
- **Oracle PL/SQL `EXECUTE IMMEDIATE` without `USING` is vulnerable:** Always use the `USING` clause for bind variables.

### Annotated Code Examples

#### Example 1: SQL Server — Vulnerable Stored Procedure

```sql
-- VULNERABLE: Stored procedure with dynamic SQL concatenation
CREATE PROCEDURE GetProducts @orderby NVARCHAR(MAX) AS
BEGIN
    DECLARE @sql NVARCHAR(MAX);
    SET @sql = 'SELECT * FROM Products ORDER BY ' + @orderby;
    EXEC sp_executesql @sql;
END;
```

**Attack Input:** `@orderby = 'ProductID; DROP TABLE Products; --'`

**Why This Works:** The concatenated `@orderby` value is inserted directly into the dynamic SQL string. The attacker can terminate the `ORDER BY` clause and append arbitrary SQL. The `--` comments out the remainder of the original query.

#### Example 2: Oracle PL/SQL — Vulnerable Dynamic SQL in Procedure

```sql
-- VULNERABLE: EXECUTE IMMEDIATE with concatenation inside a procedure
CREATE OR REPLACE PROCEDURE update_employee_salary(
    emp_id IN VARCHAR2,
    new_salary IN VARCHAR2
) IS
BEGIN
    EXECUTE IMMEDIATE
        'UPDATE employees SET salary = ' || new_salary
        || ' WHERE employee_id = ' || emp_id;
END;
```

**Attack Input:** `emp_id = "1 OR 1=1"` and `new_salary = "100000"`

**Resulting Query:**

```sql
UPDATE employees SET salary = 100000 WHERE employee_id = 1 OR 1=1
```

**Why This Works:** The `OR 1=1` condition makes the WHERE clause always true, updating every employee's salary. The procedure is vulnerable because it concatenates both `emp_id` and `new_salary` into the dynamic SQL string.

### Real-World Cases

- **Legacy stored procedures:** Procedures written before parameterized queries were standard practice.
- **Flexible search procedures:** Procedures that accept a sort column or filter expression as a parameter.
- **Batch processing procedures:** Procedures that build dynamic SQL for bulk operations.
- **SECURITY DEFINER procedures:** Procedures that execute with elevated privileges — a vulnerability here is especially dangerous.

### References

- OWASP: SQL Injection Prevention Cheat Sheet (Defense Option 2: Stored Procedures) — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP: Application Security FAQ — Are Stored Procedures Safe? — https://wiki.owasp.org/index.php/OWASP_Application_Security_FAQ
- SQL Server: Dynamic SQL in Stored Procedures — https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-injection
- Oracle: Avoiding SQL Injection in PL/SQL — https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/dynamic-sql.html


## Summary Table: SQL Injection Prevention Techniques

| Technique | Primary Mechanism | Effective Against | Key Limitation |
|-----------|------------------|-------------------|----------------|
| **Parameterized Queries** | Bind variables (`?`, `:name`, `$1`) | All value-based injection | Cannot parameterize identifiers |
| **Prepared Statements** | `PREPARE`/`EXECUTE`, `sp_executesql` | All value-based injection | Session-scoped; not all statements preparable |
| **Allow-List Validation** | Accept only known-good input | Identifiers, sort columns, enum values | Requires complete allow-list |
| **Escaping Functions** | `QUOTE_LITERAL`, `DBMS_ASSERT`, `QUOTENAME` | String literals in dynamic SQL | Last resort; fragile if misapplied |
| **Stored Procedure Auditing** | Static SQL, parameterized dynamic SQL | Dynamic SQL within routines | Requires code review; `SECURITY DEFINER` escalates risk |
| **Least-Privilege Accounts** | Minimal database permissions | Reduces blast radius | Does not prevent injection; limits damage |


## Final Notes on Deprecated and Unsafe Features

- **String concatenation in SQL:** The primary cause of SQL injection. Stop writing dynamic queries with string concatenation.
- **Escaping all user-supplied input:** STRONGLY DISCOURAGED by OWASP as a primary defense. It is fragile and error-prone compared to parameterized queries. Use it only as a defense-in-depth measure when parameterization is impossible.
- **Black-list input validation:** Do not try to detect attack characters (`'`, `--`, `1=1`) — it is impossible to think of a complete list of potentially dangerous input.
- **Client-side validation alone:** Client-side JavaScript validation is for user experience, not security. Always validate on the server side.
- **`EXEC` with concatenation in SQL Server:** Vulnerable to SQL injection. Use `sp_executesql` with parameters instead.
- **`EXECUTE IMMEDIATE` without `USING` in Oracle PL/SQL:** Vulnerable to SQL injection. Always use the `USING` clause for bind variables.
- **`SECURITY DEFINER` with dynamic SQL:** A vulnerable procedure running with `SECURITY DEFINER` executes with the owner's privileges, potentially giving the attacker elevated access.
- **`quote_literal` and `quote_ident` in PostgreSQL:** These are for PL/pgSQL functions that use `EXECUTE`. These days `quote_literal` is mostly obsoleted by `EXECUTE ... USING` (bind variables).
- **Version-specific:** Oracle `DBMS_ASSERT` was introduced in Oracle 10g Release 2 and backported to Release 1 in the October 2005 Critical Patch Update.