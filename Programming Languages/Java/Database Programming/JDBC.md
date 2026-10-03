# JDBC Fundamentals & Resource Management: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition
JDBC (Java Database Connectivity) is a Java API that defines how a client may access a database, providing methods for querying and updating data in a relational database.

### Technical Definition
JDBC is a Java-based data access technology that provides a standard interface for connecting Java applications to relational databases. It consists of a set of interfaces and classes in the `java.sql` and `javax.sql` packages, enabling Java programs to create sessions, execute SQL statements, and retrieve results from relational databases, providing vendor-independent access to relational data. The JDBC API consists of four major components: JDBC drivers, connections, statements, and result sets.

### Beginner-Friendly Explanation
JDBC is like a universal translator between your Java program and a database. Just as a translator helps two people who speak different languages communicate, JDBC lets your Java code "talk" to any database (MySQL, Oracle, PostgreSQL) without you needing to learn each database's unique language. You write standard Java code, and JDBC handles the translation.

### Key Characteristics
- **Vendor-Independent**: Write once, connect to any database
- **Standardized**: Defined by the `java.sql` package interfaces
- **Resource-Managed**: Connections, statements, and result sets must be explicitly closed
- **Driver-Based**: Database vendors provide JDBC driver implementations
- **SQL-Centric**: Executes SQL statements and processes tabular results

### Prerequisites
- Basic Java programming (interfaces, exceptions, try-with-resources)
- Understanding of SQL fundamentals (SELECT, INSERT, UPDATE, DELETE)
- Familiarity with relational database concepts (tables, rows, columns)
- A database instance and its JDBC driver JAR

### Related Programming Areas
- **Connection Pooling**: HikariCP, Apache DBCP for connection reuse
- **ORM Frameworks**: Hibernate, JPA for object-relational mapping
- **Transaction Management**: ACID properties, isolation levels
- **Data Source**: `javax.sql.DataSource` for connection abstraction

### Core Concepts Overview
1. **JDBC Architecture**: Core interfaces and SPI mechanics
2. **Modern Driver Loading**: Automatic SPI discovery (eliminating `Class.forName()`)
3. **Resource Safety**: Deterministic cleanup using Try-with-resources
4. **Statements vs. PreparedStatements**: Pre-compilation advantages
5. **ResultSet Mechanics**: Scrollability, concurrency, and column mapping

---

## Core Concept 1: JDBC Architecture

### Definitions
**Core Definition**: JDBC architecture defines the structural interfaces and driver SPI mechanics that enable Java applications to interact with relational databases in a vendor-independent manner.

**Technical Definition**: The JDBC API is expressed as a series of abstract Java interfaces that allow applications to connect to particular databases, execute SQL statements, and process the results. Each driver must provide implementations of `java.sql.Connection`, `java.sql.Statement`, `java.sql.PreparedStatement`, `java.sql.CallableStatement`, and `java.sql.ResultSet`. All JDBC drivers implement four core JDBC interfaces: Driver, Connection, Statement, and ResultSet.

**Beginner-Friendly Explanation**: JDBC is built like a standardized electrical socket. Your appliance (Java app) has a standard plug (JDBC API). Any country (database vendor) provides its own socket (JDBC driver) that matches your plug. You don't need to rewire your appliance when you move to a new country—you just need the right adapter (driver).

### Purposes
- To provide a standard API for database access
- To enable vendor-independent database connectivity
- To abstract database-specific implementation details
- To support SQL statement execution and result processing
- To facilitate connection management and transaction control

### Syntax Rules and Structure
#### Complete General Syntax: Core JDBC Interfaces
```
JDBC CORE INTERFACES (java.sql package)
│
├── Driver
│   └── The interface that every driver class must implement
│
├── Connection
│   └── A connection (session) with a specific database
│
├── Statement
│   └── The interface used to execute SQL statements
│   ├── PreparedStatement — precompiled SQL statement
│   └── CallableStatement — executes stored procedures
│
└── ResultSet
    └── A table of data representing a database result set
```

#### Component Breakdown
| Interface | Responsibility |
|-----------|---------------|
| `Driver` | Locates the driver for a database URL |
| `Connection` | Connects to a specific database |
| `Statement` | Executes SQL statements |
| `PreparedStatement` | Represents a precompiled SQL statement |
| `ResultSet` | Represents query results |

#### Syntax Rules
- All interfaces are in the `java.sql` package
- Driver implementations are provided by database vendors
- `DriverManager` manages driver registration and connection creation
- `Connection` is the entry point for all database operations

#### Constraints and Limitations
- JDBC drivers are database-specific; you need the correct driver JAR
- Connections are expensive resources; pooling is recommended for production
- SQL syntax varies across databases despite JDBC standardization

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs
#### Example 1: Basic JDBC Connection and Query
```java
// JdbcArchitectureDemo.java
import java.sql.*;

public class JdbcArchitectureDemo {
    public static void main(String[] args) {
        // Connection string (database URL)
        String url = "jdbc:mysql://localhost:3306/testdb";
        
        // Establish connection (Driver is auto-loaded via SPI)
        try (Connection conn = DriverManager.getConnection(url, "user", "pass");
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT id, name FROM users")) {
            
            // Process ResultSet
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                System.out.println("ID: " + id + ", Name: " + name);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
ID: 1, Name: Alice
ID: 2, Name: Bob
```
**Why This Output**: `DriverManager.getConnection()` uses the JDBC URL to locate and connect to the database. The `Statement` executes the query, and the `ResultSet` iterates over rows. The `try-with-resources` ensures all resources are closed.

### Real-World Cases
- **Web Applications**: Servlets query databases via JDBC
- **Batch Processing**: ETL tools use JDBC to read/write data
- **Reporting Tools**: Generate reports from database queries

### References
- Package java.sql - https://docs.oracle.com/javase/8/docs/api/java/sql/package-summary.html
- Lesson: JDBC Basics - https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html
- JDBC API - https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/

---

## Core Concept 2: Modern Driver Loading

### Definitions
**Core Definition**: Modern driver loading refers to the automatic discovery and registration of JDBC drivers via the Java Service Provider Interface (SPI), eliminating the need for explicit `Class.forName()` calls.

**Technical Definition**: Since JDBC 4.0 (Java 6), drivers are automatically loaded and registered via Java's Service Provider Interface (SPI). JDBC 4.0 Drivers must include the file `META-INF/services/java.sql.Driver` in their JAR, containing the name of the JDBC driver's implementation of `java.sql.Driver`. The `DriverManager.getConnection` method has been enhanced to support the Java Standard Edition Service Provider mechanism, using `ServiceLoader` to enumerate all `/META-INF/services/java.sql.Driver` files in the classpath and load all drivers so they get registered.

**Beginner-Friendly Explanation**: Before JDBC 4.0, you had to explicitly tell Java which database driver to use by calling `Class.forName("com.mysql.jdbc.Driver")`. Modern Java does this automatically—it scans the classpath, finds the driver's configuration file, and loads it for you.

### Purposes
- To simplify JDBC driver registration
- To eliminate boilerplate `Class.forName()` calls
- To enable automatic driver discovery from the classpath
- To support pluggable driver architectures

### Syntax Rules and Structure
#### Complete General Syntax: Automatic Driver Discovery
```
AUTOMATIC DRIVER DISCOVERY (JDBC 4.0+)
│
├── Driver JAR
│   └── META-INF/services/java.sql.Driver
│       └── Contains driver class name (e.g., com.mysql.cj.jdbc.Driver)
│
├── DriverManager Initialization
│   └── ServiceLoader scans classpath
│       └── Loads all drivers from META-INF/services
│
└── Application Code
    └── DriverManager.getConnection(url, user, pass)
        └── Driver is already registered — no Class.forName() needed
```

#### Syntax Rules
- Driver JAR must include `META-INF/services/java.sql.Driver`
- The file contains the fully qualified driver class name
- `DriverManager.getConnection()` triggers SPI discovery
- Legacy `Class.forName()` still works but is unnecessary

#### Constraints and Limitations
- Requires JDBC 4.0+ compliant driver
- Multiple drivers on classpath may cause conflicts
- `Class.forName()` is still needed for drivers that don't support SPI

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs
#### Example 1: Modern Driver Loading
```java
// ModernDriverLoadingDemo.java
import java.sql.*;

public class ModernDriverLoadingDemo {
    public static void main(String[] args) {
        // NO Class.forName() needed!
        // Driver is auto-loaded from META-INF/services/java.sql.Driver
        
        String url = "jdbc:mysql://localhost:3306/testdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            System.out.println("Connected successfully!");
            System.out.println("Driver: " + conn.getMetaData().getDriverName());
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
Connected successfully!
Driver: MySQL Connector/J
```
**Why This Output**: The MySQL JDBC driver JAR contains `META-INF/services/java.sql.Driver` with the driver class name. `DriverManager` automatically discovers and registers it via SPI.

### Real-World Cases
- **Spring Boot**: Auto-configures DataSource using SPI-discovered drivers
- **Cloud-Native Apps**: Drivers are bundled in the application JAR and auto-loaded
- **Multi-Database Apps**: Multiple drivers can coexist on the classpath

### References
- DriverManager - Java SE 8 - https://docs.oracle.com/javase/8/docs/api/java/sql/DriverManager.html
- JDBC 4.0 Specification - https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/

---

## Core Concept 3: Resource Safety (Try-with-Resources)

### Definitions
**Core Definition**: Resource safety refers to the deterministic cleanup of JDBC resources (Connection, Statement, ResultSet) using try-with-resources to prevent memory and cursor leaks.

**Technical Definition**: JDBC 4.1 (Java 7) introduced the ability to use a `try-with-resources` statement to automatically close resources of type `Connection`, `ResultSet`, and `Statement`. You can use a `try-with-resources` statement to automatically close `java.sql.Connection`, `java.sql.Statement`, and `java.sql.ResultSet` objects, regardless of whether a `SQLException` or any other exception has been thrown. Without proper cleanup, unclosed resources can cause memory leaks, cursor leaks, and connection pool exhaustion.

**Beginner-Friendly Explanation**: Think of database resources as library books. If you don't return them, the library runs out of books. `try-with-resources` is like an automatic return system—no matter what happens (even if your code throws an exception), the books get returned.

### Purposes
- To prevent connection leaks
- To prevent cursor leaks (unclosed ResultSets)
- To ensure deterministic resource cleanup
- To reduce boilerplate finally blocks
- To improve application stability and performance

### Syntax Rules and Structure
#### Complete General Syntax: Try-with-Resources
```
TRY-WITH-RESOURCES SYNTAX
│
├── try (Resource1 r1 = createResource1();
│        Resource2 r2 = createResource2()) {
│     // Use resources
│ } catch (Exception e) {
│     // Handle exception
│ }
│
└── Resources must implement AutoCloseable
    ├── Connection implements AutoCloseable
    ├── Statement implements AutoCloseable
    └── ResultSet implements AutoCloseable
```

#### Syntax Rules
- Declare resources in parentheses after `try`
- Multiple resources separated by semicolons
- Resources are closed in reverse order of declaration
- Exceptions in close() are suppressed if another exception occurs

#### Constraints and Limitations
- Resources must implement `AutoCloseable`
- Closing a ResultSet also closes the Statement (per JDBC spec)
- Closing a Statement does NOT close the Connection
- Must close Connection separately

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs
#### Example 1: Try-with-Resources Best Practice
```java
// ResourceSafetyDemo.java
import java.sql.*;

public class ResourceSafetyDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/testdb";
        String query = "SELECT id, name FROM users WHERE active = ?";
        
        // All three resources auto-closed
        try (Connection conn = DriverManager.getConnection(url, "user", "pass");
             PreparedStatement pstmt = conn.prepareStatement(query)) {
            
            pstmt.setBoolean(1, true);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    System.out.println(rs.getInt("id") + ": " + rs.getString("name"));
                }
            }
            // ResultSet closed here
            
        } catch (SQLException e) {
            System.err.println("Database error: " + e.getMessage());
        }
        // Connection and PreparedStatement closed here
    }
}
```
**Expected Output**:
```
1: Alice
2: Bob
```
**Why This Output**: The `try-with-resources` block automatically closes the `ResultSet`, `PreparedStatement`, and `Connection` in reverse order when the block exits, even if an exception occurs. The nested `try` ensures the `ResultSet` is closed before the `PreparedStatement`.

### Real-World Cases
- **Connection Pools**: Leaked connections exhaust the pool, causing application hangs
- **Batch Jobs**: Unclosed cursors cause database-side memory pressure
- **Web Applications**: Proper cleanup prevents "too many connections" errors

### References
- JDBC 4.1 - https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_41.html
- The try-with-resources Statement - https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
- Processing SQL Statements with JDBC - https://docs.oracle.com/javase/tutorial/jdbc/basics/processingsqlstatements.html

---

## Core Concept 4: Statements vs. PreparedStatements

### Definitions
**Core Definition**: `Statement` is used for executing static SQL, while `PreparedStatement` is a precompiled SQL statement that can be executed multiple times with different parameters, offering performance and security advantages.

**Technical Definition**: `PreparedStatement` is a special type of `Statement` that is given a SQL statement when it is created. The SQL statement is sent to the DBMS right away, where it is compiled. As a result, the `PreparedStatement` object contains not just a SQL statement, but a SQL statement that has been precompiled. When executed, the DBMS can just run the `PreparedStatement` SQL statement without having to compile it first. The most important advantage of prepared statements is that they help prevent SQL injection attacks.

**Beginner-Friendly Explanation**: Think of `Statement` as writing a new letter for each recipient, and `PreparedStatement` as creating a template with blanks that you fill in. The template is already processed (stamped, addressed) so you just fill in the blanks—much faster and less error-prone.

### Purposes
- To improve performance for repeated executions
- To prevent SQL injection attacks
- To enable parameterized queries
- To support batch updates efficiently
- To cache execution plans at the database level

### Syntax Rules and Structure
#### Complete General Syntax: Statement vs. PreparedStatement
```
STATEMENT vs. PREPAREDSTATEMENT
│
├── Statement (Static SQL)
│   └── stmt.executeQuery("SELECT * FROM users WHERE id = " + id)
│       └── SQL compiled each time — SQL injection risk
│
└── PreparedStatement (Parameterized SQL)
    └── pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?")
        ├── pstmt.setInt(1, id)
        └── pstmt.executeQuery()
            └── SQL precompiled once — no injection risk
```

#### Component Breakdown
| Aspect | Statement | PreparedStatement |
|--------|-----------|-------------------|
| Compilation | Every execution | Once (precompiled) |
| Parameters | String concatenation | Placeholders (`?`) |
| SQL Injection | Vulnerable | Safe |
| Batch Updates | No benefit | Significant benefit |
| Caching | No | Execution plan cached |

#### Syntax Rules
- `PreparedStatement` uses `?` as parameter placeholder
- Parameters set via `setInt()`, `setString()`, etc.
- Parameter indices start at 1 (not 0)
- `executeQuery()` for SELECT, `executeUpdate()` for INSERT/UPDATE/DELETE
- `addBatch()` / `executeBatch()` for batch processing

#### Constraints and Limitations
- PreparedStatement uses more memory per statement
- Not all SQL statements benefit from preparation (e.g., DDL)
- Some databases don't cache execution plans well

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs
#### Example 1: PreparedStatement Performance
```java
// PreparedStatementDemo.java
import java.sql.*;

public class PreparedStatementDemo {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:mysql://localhost:3306/testdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            
            // BAD: Statement with concatenation (injection risk, slow)
            String name = "Alice'; DROP TABLE users; --";
            try (Statement stmt = conn.createStatement()) {
                // This would be vulnerable to SQL injection
                // stmt.executeQuery("SELECT * FROM users WHERE name = '" + name + "'");
            }
            
            // GOOD: PreparedStatement with parameter
            String sql = "SELECT id, name, email FROM users WHERE name = ? AND active = ?";
            try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                
                // Set parameters (safe from injection)
                pstmt.setString(1, "Alice");
                pstmt.setBoolean(2, true);
                
                try (ResultSet rs = pstmt.executeQuery()) {
                    while (rs.next()) {
                        System.out.println(rs.getInt("id") + ": " + 
                                           rs.getString("name") + " (" + 
                                           rs.getString("email") + ")");
                    }
                }
            }
        }
    }
}
```
**Expected Output**:
```
1: Alice (alice@example.com)
```
**Why This Output**: The `PreparedStatement` uses `?` placeholders. `setString(1, "Alice")` safely binds the value, preventing SQL injection. The SQL is precompiled once and reused.

### Real-World Cases
- **Login Systems**: Must use PreparedStatement to prevent SQL injection
- **Batch Inserts**: PreparedStatement with batch updates processes thousands of rows efficiently
- **Reporting**: Repeated queries with different parameters benefit from precompilation

### References
- Using Prepared Statements - https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html
- PreparedStatement - https://docs.oracle.com/javase/8/docs/api/java/sql/PreparedStatement.html

---

## Core Concept 5: ResultSet Mechanics

### Definitions
**Core Definition**: `ResultSet` represents a table of data generated by executing a query, with configurable scrollability types (forward-only, scroll-insensitive, scroll-sensitive) and concurrency modes (read-only, updatable).

**Technical Definition**: A `ResultSet` object maintains a cursor pointing to its current row of data. Initially the cursor is positioned before the first row. The `next()` method moves the cursor to the next row, and because it returns `false` when there are no more rows in the `ResultSet` object, it can be used in a while loop to iterate through the result set. ResultSets are characterized by their type and concurrency. The type specifies whether the cursor can scroll. The concurrency specifies whether the ResultSet can be updated.

**Beginner-Friendly Explanation**: A `ResultSet` is like a spreadsheet of query results. You can move through the rows one by one (forward-only), or you can scroll back and forth (scrollable). You can also specify whether you want to just read the data (read-only) or modify it directly through the ResultSet (updatable).

### Purposes
- To represent tabular query results
- To navigate rows via cursor
- To retrieve column values by name or index
- To support scrollable navigation for complex processing
- To enable direct updates through the ResultSet

### Syntax Rules and Structure
#### Complete General Syntax: ResultSet Types and Concurrency
```
RESULTSET TYPES AND CONCURRENCY
│
├── Type (Scrollability)
│   ├── TYPE_FORWARD_ONLY         — cursor moves forward only
│   ├── TYPE_SCROLL_INSENSITIVE   — scrollable, not sensitive to changes
│   └── TYPE_SCROLL_SENSITIVE     — scrollable, sensitive to changes
│
├── Concurrency
│   ├── CONCUR_READ_ONLY          — cannot be updated
│   └── CONCUR_UPDATABLE          — can be updated via positioned updates
│
└── Creating ResultSet with Options
    └── stmt = conn.createStatement(type, concurrency)
        └── rs = stmt.executeQuery(sql)
```

#### Component Breakdown
| Type | Scrollable | Sensitive to Changes |
|------|-----------|---------------------|
| `TYPE_FORWARD_ONLY` | No | N/A |
| `TYPE_SCROLL_INSENSITIVE` | Yes | No |
| `TYPE_SCROLL_SENSITIVE` | Yes | Yes |

| Concurrency | Can Update |
|-------------|-----------|
| `CONCUR_READ_ONLY` | No |
| `CONCUR_UPDATABLE` | Yes (positioned updates) |

#### Syntax Rules
- Default `ResultSet` is `TYPE_FORWARD_ONLY`, `CONCUR_READ_ONLY`
- Use `createStatement(type, concurrency)` to specify options
- `rs.next()` moves cursor forward
- `rs.previous()`, `rs.first()`, `rs.last()` require scrollable type
- Column retrieval: `rs.getString("column")` or `rs.getString(1)`

#### Constraints and Limitations
- Not all databases support all ResultSet types
- `TYPE_SCROLL_SENSITIVE` is often not fully implemented
- Updatable ResultSets have database-specific limitations
- Column names may be case-sensitive depending on database

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs
#### Example 1: Scrollable ResultSet
```java
// ScrollableResultSetDemo.java
import java.sql.*;

public class ScrollableResultSetDemo {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:mysql://localhost:3306/testdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass");
             Statement stmt = conn.createStatement(
                 ResultSet.TYPE_SCROLL_INSENSITIVE,
                 ResultSet.CONCUR_READ_ONLY);
             ResultSet rs = stmt.executeQuery(
                 "SELECT id, name, salary FROM employees ORDER BY id")) {
            
            // Move to last row
            rs.last();
            System.out.println("Last row: " + rs.getInt("id") + ", " + 
                               rs.getString("name"));
            
            // Move to first row
            rs.first();
            System.out.println("First row: " + rs.getInt("id") + ", " + 
                               rs.getString("name"));
            
            // Move to row 3
            rs.absolute(3);
            System.out.println("Row 3: " + rs.getInt("id") + ", " + 
                               rs.getString("name"));
            
            // Iterate backward
            System.out.println("Backward iteration:");
            while (rs.previous()) {
                System.out.println("  " + rs.getInt("id") + ": " + 
                                   rs.getString("name"));
            }
        }
    }
}
```
**Expected Output**:
```
Last row: 5, Eve
First row: 1, Alice
Row 3: 3, Charlie
Backward iteration:
  3: Charlie
  2: Bob
  1: Alice
```
**Why This Output**: The `TYPE_SCROLL_INSENSITIVE` type allows the cursor to move backward and to absolute positions. `last()`, `first()`, and `absolute(3)` position the cursor. `previous()` iterates backward.

### Real-World Cases
- **Pagination**: Scrollable ResultSets enable jumping to specific pages
- **Reporting**: Backward iteration for summary calculations
- **Data Browsing**: UI components that allow users to navigate forward and backward

### References
- ResultSet - https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSet.html
- Retrieving and Modifying Values from Result Sets - https://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html

---

## References

### Official Specifications
- Package java.sql - https://docs.oracle.com/javase/8/docs/api/java/sql/package-summary.html
- DriverManager - https://docs.oracle.com/javase/8/docs/api/java/sql/DriverManager.html
- PreparedStatement - https://docs.oracle.com/javase/8/docs/api/java/sql/PreparedStatement.html
- ResultSet - https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSet.html

### Oracle Tutorials
- Lesson: JDBC Basics - https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html
- Establishing a Connection - https://docs.oracle.com/javase/tutorial/jdbc/basics/connecting.html
- Using Prepared Statements - https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html
- Retrieving and Modifying Values from Result Sets - https://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html

### JDBC Specifications
- JDBC 4.1 - https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_41.html
- JDBC API - https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/
- The try-with-resources Statement - https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html

### Additional Resources
- PostgreSQL JDBC Driver - https://jdbc.postgresql.org/
- MySQL Connector/J - https://dev.mysql.com/doc/connector-j/en/
- HikariCP (Connection Pool) - https://github.com/brettwooldridge/HikariCP