# Modern Abstractions (Beyond Raw JDBC): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition
Modern JDBC abstractions are higher-level libraries and frameworks that wrap the raw JDBC API to eliminate boilerplate, improve type safety, reduce error-proneness, and provide more maintainable database access code.

### Technical Definition
Modern JDBC abstractions are libraries that sit on top of the JDBC specification (java.sql) and provide simplified, fluent, or type-safe APIs for database operations. These include lightweight data mappers (JDBI, JOOQ), Spring's JDBC abstraction (JdbcTemplate, JdbcClient), and other non-JPA persistence frameworks. They address the fundamental friction of raw JDBC: connection lifecycle management, exception handling, result set mapping, and parameter binding, while preserving direct SQL control.

### Beginner-Friendly Explanation
Raw JDBC is like assembling furniture from scratch—you get all the raw materials, but you have to measure, cut, and assemble every piece yourself. Modern abstractions are like buying pre-assembled furniture—you get the same end result with far less effort and fewer mistakes. They don't hide SQL; they just handle the tedious plumbing so you can focus on the queries that matter.

### Key Characteristics
- **Boilerplate Elimination**: Connection management, exception handling, and resource cleanup are automated
- **Fluent APIs**: Method chaining creates readable, expressive database operations
- **Type Safety**: Compile-time checking of queries and mappings
- **SQL-Preserving**: Unlike ORMs, most abstractions keep SQL front and center
- **Exception Translation**: Checked `SQLException` is converted to unchecked, meaningful exceptions
- **Declarative Mapping**: Result sets are automatically mapped to Java objects

### Prerequisites
- Solid understanding of JDBC fundamentals (Connection, PreparedStatement, ResultSet)
- Familiarity with SQL and relational database concepts
- Experience with Java generics and lambda expressions
- Understanding of Spring Framework (for Spring JDBC ecosystem)
- Basic knowledge of Maven/Gradle dependency management

### Related Programming Areas
- **JDBC Fundamentals**: The underlying API that all abstractions wrap
- **Connection Pooling**: HikariCP integration with modern abstractions
- **Transaction Management**: Declarative transactions in Spring
- **ORM Frameworks**: JPA/Hibernate as an alternative approach
- **Testing**: Integration testing with Testcontainers and embedded databases

### Core Concepts Overview
1. **The Friction of Raw JDBC**: Identifying maintenance overhead, boilerplate bloat, and repetitive logic
2. **Fluent Database Libraries**: JDBI and JOOQ for type-safe query building
3. **Spring JDBC Ecosystem**: JdbcClient (Spring Framework 6.1) and JdbcTemplate

---

## Core Concept 1: The Friction of Raw JDBC

### Definitions

**Core Definition**: The friction of raw JDBC refers to the inherent complexity, verbosity, and maintenance burden of using the low-level JDBC API directly in production applications.

**Technical Definition**: Raw JDBC requires developers to manually manage database connections, create and configure statements, bind parameters, iterate result sets, handle checked exceptions, and close resources in a finally block. The API is procedural and imperative, offering no abstraction for common patterns like CRUD operations, query composition, or result mapping. SQL statements are hardcoded as strings, making refactoring and schema changes costly. Error handling is vendor-specific, with `SQLException` requiring parsing of `getSQLState()` and `getErrorCode()` to distinguish between transient and permanent failures. Result set processing involves repetitive code for column extraction and type conversion, often duplicated across every query.

**Beginner-Friendly Explanation**: Raw JDBC is like writing the same 20 lines of code every single time you want to talk to the database—open connection, create statement, set parameters, execute query, loop through results, close everything. If you forget one `close()` call, you leak connections. If you change a column name, you have to find every SQL string in your codebase. Modern abstractions exist to eliminate this repetitive, error-prone work.

### Purposes
- To identify the specific pain points of raw JDBC that motivate abstraction adoption
- To justify the investment in learning modern database libraries
- To understand the trade-offs between control (raw JDBC) and productivity (abstractions)
- To recognize anti-patterns in existing JDBC code
- To establish criteria for choosing the right abstraction level

### Syntax Rules and Structure

#### Complete General Syntax: Raw JDBC Boilerplate
```
RAW JDBC BOILERPLATE PATTERN
│
├── 1. Load Driver (legacy)
│   └── Class.forName("com.mysql.cj.jdbc.Driver");
│
├── 2. Establish Connection
│   └── Connection conn = DriverManager.getConnection(url, user, pass);
│
├── 3. Create Statement
│   └── PreparedStatement pstmt = conn.prepareStatement(sql);
│
├── 4. Bind Parameters
│   ├── pstmt.setString(1, value1);
│   └── pstmt.setInt(2, value2);
│
├── 5. Execute Query
│   └── ResultSet rs = pstmt.executeQuery();
│
├── 6. Process Result Set (repetitive)
│   └── while (rs.next()) { entity.setX(rs.getString("x")); ... }
│
├── 7. Handle Checked Exceptions
│   └── catch (SQLException e) { ... }
│
└── 8. Close Resources (finally block)
    ├── rs.close();
    ├── pstmt.close();
    └── conn.close();
```

#### Component Breakdown
| Friction Point | Problem | Impact |
|---------------|---------|--------|
| Connection Management | Manual open/close in finally | Resource leaks, pool exhaustion |
| SQL Hardcoding | Strings scattered across code | Maintenance burden, schema coupling |
| Parameter Binding | Positional indices, manual type mapping | Error-prone, verbose |
| Result Set Mapping | Repetitive column extraction | Duplication, low productivity |
| Exception Handling | Checked `SQLException` everywhere | Noisy code, poor classification |
| Transaction Management | Manual commit/rollback | Inconsistent transaction boundaries |

#### Syntax Rules
- Always use try-with-resources for JDBC resources
- Never concatenate user input into SQL strings
- Always use `PreparedStatement` for parameterized queries
- Close resources in reverse order of creation
- Handle `SQLException` explicitly; never swallow exceptions

#### Constraints and Limitations
- Raw JDBC is the lowest common denominator—it works with every database, but with maximum effort
- Abstractions add learning curve and dependency overhead
- Some abstractions (JOOQ) require code generation and commercial licensing for certain databases
- Spring JDBC requires Spring Framework as a dependency
- JDBI is lightweight but lacks compile-time type safety

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Raw JDBC vs. Abstracted Query
```java
// RawJdbcBoilerplateDemo.java
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class RawJdbcBoilerplateDemo {
    
    static class User {
        int id; String name; String email;
        User(int id, String name, String email) {
            this.id = id; this.name = name; this.email = email;
        }
        @Override public String toString() {
            return "User{" + id + ", " + name + ", " + email + "}";
        }
    }
    
    // ❌ RAW JDBC: ~25 lines of boilerplate
    static List<User> findUsersRaw(String namePattern) {
        List<User> users = new ArrayList<>();
        String sql = "SELECT id, name, email FROM users WHERE name LIKE ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/testdb", "user", "pass");
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, "%" + namePattern + "%");
            rs = pstmt.executeQuery();
            while (rs.next()) {
                users.add(new User(
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getString("email")
                ));
            }
        } catch (SQLException e) {
            throw new RuntimeException("Query failed", e);
        } finally {
            // Manual resource cleanup — easy to miss
            try { if (rs != null) rs.close(); } catch (SQLException ignored) {}
            try { if (pstmt != null) pstmt.close(); } catch (SQLException ignored) {}
            try { if (conn != null) conn.close(); } catch (SQLException ignored) {}
        }
        return users;
    }
    
    public static void main(String[] args) {
        System.out.println("Raw JDBC requires ~25 lines for one query");
        System.out.println("Every additional query repeats the same boilerplate");
        
        // With Spring JdbcClient (shown for comparison — see Core Concept 3)
        // List<User> users = jdbcClient.sql(sql)
        //     .param("pattern", "%" + namePattern + "%")
        //     .query(User.class)
        //     .list();
        // That's 4 lines instead of 25.
    }
}
```
**Expected Output**:
```
Raw JDBC requires ~25 lines for one query
Every additional query repeats the same boilerplate
```
**Why This Output**: The raw JDBC version requires manual connection management, statement creation, parameter binding, result iteration, exception handling, and resource cleanup. The commented Spring JdbcClient version achieves the same result in four lines. This demonstrates the boilerplate bloat that abstractions eliminate.

### Real-World Cases
- **Legacy Systems**: Millions of lines of raw JDBC code requiring maintenance and modernization
- **Migration Projects**: Teams migrating from raw JDBC to Spring JDBC or JDBI report significant productivity gains
- **Code Reviews**: Raw JDBC boilerplate obscures business logic, making reviews slower and less effective
- **Onboarding**: New developers spend days learning the JDBC boilerplate patterns instead of focusing on domain logic

### References
- Traditional JDBC Development Problems - Alibaba Cloud - https://developer.aliyun.com/ask/327885
- Raw JDBC and Spring JDBC Template - Stack Overflow - https://stackoverflow.com/revisions/0408968a-95c9-4540-b9e8-4fb65db35c9e/view-source
- Replace DBCP2 with HikariCP in RDBMS Plugin - Apache Drill - https://issues.apache.org/jira/browse/DRILL-7639

---

## Core Concept 2: Fluent Database Libraries (JDBI & JOOQ)

### Definitions

**Core Definition**: Fluent database libraries are lightweight data-mapping frameworks that provide chainable, expressive APIs for building and executing SQL queries, with JDBI offering convenience over JDBC and JOOQ providing compile-time type safety through generated code.

**Technical Definition**: **JDBI** is a SQL convenience library that attempts to expose relational database access in idiomatic Java, using collections, beans, and so on, while maintaining the same level of detail as JDBC. It exposes two style APIs: a fluent style (method chaining) and a SQL Object style (annotated interfaces). **JOOQ** (Java Object Oriented Querying) generates Java classes from database tables and lets developers create type-safe SQL queries through its fluent API. JOOQ implements SQL as an internal domain-specific language (DSL), wrapping strings, literals, and user-defined objects into an object-oriented, type-safe AST that models SQL statements.

**Beginner-Friendly Explanation**: Think of JDBI as a friendly assistant who takes your SQL and makes it easier to work with—you still write SQL, but the assistant handles the boilerplate. JOOQ is like having a Java compiler for your SQL—if you make a typo in a column name or use the wrong type, the compiler catches it before you even run the code. JDBI is simpler and more flexible; JOOQ is stricter but catches more errors at compile time.

### Purposes
- To reduce JDBC boilerplate while preserving direct SQL control
- To provide type-safe query building with JOOQ's generated classes
- To enable declarative result mapping with JDBI's SQL Object API
- To support fluent, chainable query construction
- To integrate seamlessly with connection pools and transaction managers
- To offer a middle ground between raw JDBC and full ORMs

### Syntax Rules and Structure

#### Complete General Syntax: JDBI Fluent API
```
JDBI FLUENT API
│
├── 1. Create DBI
│   └── DBI dbi = new DBI(dataSource);
│
├── 2. Open Handle
│   └── Handle handle = dbi.open();
│
├── 3. Execute Update
│   └── handle.execute("INSERT INTO something (id, name) VALUES (?, ?)", 1, "Brian");
│
├── 4. Create Query
│   └── Query<String> query = handle.createQuery("SELECT name FROM something WHERE id = :id");
│
├── 5. Bind Parameters
│   └── query.bind("id", 1);
│
├── 6. Map Results
│   └── query.map(StringMapper.FIRST);
│
├── 7. Execute and Retrieve
│   ├── .first()  — first result
│   ├── .list()   — all results
│   └── .iterator() — lazy iteration
│
└── 8. Close Handle
    └── handle.close();
```

#### Complete General Syntax: JOOQ DSL
```
JOOQ DSL
│
├── 1. Create DSLContext
│   └── DSLContext context = DSL.using(conn, SQLDialect.POSTGRES);
│
├── 2. Select
│   └── context.select(AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME, BOOK.TITLE)
│
├── 3. From
│   └── .from(AUTHOR)
│
├── 4. Join
│   └── .join(BOOK).on(AUTHOR.ID.eq(BOOK.AUTHOR_ID))
│
├── 5. Where
│   └── .where(BOOK.PUBLISHED.gt(LocalDate.of(2020, 1, 1)))
│
├── 6. Order By
│   └── .orderBy(BOOK.TITLE)
│
└── 7. Fetch
    ├── .fetch()          — list of records
    ├── .fetchInto(Class) — map to POJO
    └── .fetchOne()       — single record
```

#### Component Breakdown
| Library | API Style | Type Safety | Code Generation | License |
|---------|-----------|-------------|-----------------|---------|
| JDBI Fluent | Chainable | Runtime | No | Apache 2.0 |
| JDBI SQL Object | Annotated interfaces | Runtime | No | Apache 2.0 |
| JOOQ | Fluent DSL | Compile-time | Yes | Apache 2.0 / Commercial |

#### Syntax Rules
- **JDBI**: Use named parameters (`:id`) or positional (`?`); map results with `ResultSetMapper`
- **JDBI**: SQL Object API uses `@SqlQuery`, `@SqlUpdate`, `@Bind` annotations
- **JOOQ**: Always use generated table/field classes (e.g., `AUTHOR.FIRST_NAME`) for type safety
- **JOOQ**: Requires code generation from the database schema via Maven/Gradle plugin
- **JOOQ**: Supports multiple SQL dialects; the DSL generates dialect-appropriate SQL

#### Constraints and Limitations
- **JDBI**: No compile-time type checking of SQL; errors surface at runtime
- **JDBI**: SQL Object API requires interface definitions for every DAO
- **JOOQ**: Code generation adds build complexity; schema changes require regeneration
- **JOOQ**: Commercial license required for Oracle, SQL Server, and DB2
- **JOOQ**: Learning curve for the DSL API and code generation setup

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: JDBI Fluent API
```java
// JdbiFluentDemo.java
import org.jdbi.v3.core.Jdbi;
import org.jdbi.v3.core.Handle;
import org.jdbi.v3.core.result.ResultIterator;
import org.jdbi.v3.core.mapper.StringMapper;
import java.util.List;

public class JdbiFluentDemo {
    
    record User(int id, String name, String email) {}
    
    public static void main(String[] args) {
        // Step 1: Create Jdbi instance with H2 in-memory database
        Jdbi jdbi = Jdbi.create("jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1");
        
        // Step 2: Open handle and execute statements
        try (Handle handle = jdbi.open()) {
            
            // Create table
            handle.execute("""
                CREATE TABLE users (
                    id INT PRIMARY KEY,
                    name VARCHAR(100),
                    email VARCHAR(100)
                )
                """);
            
            // Insert with positional parameters
            handle.execute("INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
                1, "Alice", "alice@example.com");
            
            // Insert with named parameters
            handle.createUpdate("INSERT INTO users (id, name, email) VALUES (:id, :name, :email)")
                .bind("id", 2)
                .bind("name", "Bob")
                .bind("email", "bob@example.com")
                .execute();
            
            System.out.println("Data inserted via JDBI");
            
            // Step 3: Fluent query with mapping
            List<String> names = handle
                .createQuery("SELECT name FROM users ORDER BY id")
                .map(StringMapper.FIRST)
                .list();
            
            System.out.println("Names: " + names);
            
            // Step 4: Query with custom mapping to record
            List<User> users = handle
                .createQuery("SELECT id, name, email FROM users")
                .map((rs, ctx) -> new User(
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getString("email")
                ))
                .list();
            
            System.out.println("\nAll users:");
            users.forEach(System.out::println);
        }
    }
}
```
**Expected Output**:
```
Data inserted via JDBI
Names: [Alice, Bob]

All users:
User[id=1, name=Alice, email=alice@example.com]
User[id=2, name=Bob, email=bob@example.com]
```
**Why This Output**: JDBI's fluent API chains `createQuery()`, `map()`, and `list()` to retrieve results in a single expression. The `StringMapper.FIRST` extracts the first column as a `String`. Custom mapping with a lambda allows mapping to records or POJOs. The `try-with-resources` block ensures the handle (connection) is closed automatically.

---

#### Example 2: JOOQ Type-Safe Query
```java
// JooqTypeSafeDemo.java
import org.jooq.*;
import org.jooq.impl.DSL;
import static org.jooq.impl.DSL.*;
import static com.example.jooq.generated.Tables.*;

public class JooqTypeSafeDemo {
    
    public static void main(String[] args) {
        // Step 1: Create DSLContext with H2 in-memory database
        String url = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";
        try (Connection conn = DriverManager.getConnection(url)) {
            
            DSLContext context = DSL.using(conn, SQLDialect.H2);
            
            // Step 2: Create table using generated DSL (type-safe DDL)
            context.createTable(AUTHOR)
                .column(AUTHOR.ID, SQLDataType.INTEGER)
                .column(AUTHOR.FIRST_NAME, SQLDataType.VARCHAR(255))
                .column(AUTHOR.LAST_NAME, SQLDataType.VARCHAR(255))
                .execute();
            
            // Step 3: Insert using type-safe DSL
            context.insertInto(AUTHOR)
                .columns(AUTHOR.ID, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
                .values(1, "Ada", "Lovelace")
                .values(2, "Alan", "Turing")
                .execute();
            
            System.out.println("Data inserted via jOOQ DSL");
            
            // Step 4: Type-safe query
            // The compiler verifies AUTHOR.FIRST_NAME exists and is a String
            Result<Record2<String, String>> result = context
                .select(AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
                .from(AUTHOR)
                .orderBy(AUTHOR.ID)
                .fetch();
            
            System.out.println("\nAuthors:");
            result.forEach(r -> 
                System.out.println(r.get(AUTHOR.FIRST_NAME) + " " + 
                                   r.get(AUTHOR.LAST_NAME)));
            
            // Step 5: Type-safe query with condition
            String name = context
                .select(AUTHOR.FIRST_NAME)
                .from(AUTHOR)
                .where(AUTHOR.LAST_NAME.eq("Turing"))
                .fetchOne(AUTHOR.FIRST_NAME);
            
            System.out.println("\nFound: " + name);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
Data inserted via jOOQ DSL

Authors:
Ada Lovelace
Alan Turing

Found: Alan
```
**Why This Output**: The JOOQ DSL uses generated table/field classes (`AUTHOR.FIRST_NAME`) that provide compile-time type safety. The `select()`, `from()`, `where()`, and `orderBy()` methods chain fluently. The `fetch()` method returns a `Result` of typed records. Typos in column names or type mismatches are caught at compile time, not runtime.

### Real-World Cases
- **JDBI**: Used by companies like Airbnb and Netflix for lightweight, SQL-focused database access
- **JDBI SQL Object**: Ideal for DAO patterns where each method maps to a single SQL statement
- **JOOQ**: Used by companies requiring compile-time type safety and complex dynamic queries
- **JOOQ with PostgreSQL**: The most popular combination due to open-source licensing
- **Migration Path**: Teams migrating from raw JDBC often start with JDBI, then adopt JOOQ for critical queries

### References
- JDBI Five Minute Introduction - https://jdbi.org/jdbi2/five_minute_intro/
- JDBI Fluent Queries - https://jdbi.org/jdbi2/fluent_queries/
- jOOQ Manual - https://www.jooq.org/doc/3.17/manual-pdf/jOOQ-manual-3.17.pdf
- jOOQ Introduction - Baeldung - https://baeldung.cn/jooq-intro
- Java Persistence Frameworks Comparison - GitHub - https://github.com/bwajtr/java-persistence-frameworks-comparison

---

## Core Concept 3: Spring JDBC Ecosystem

### Definitions

**Core Definition**: The Spring JDBC ecosystem provides a comprehensive abstraction over raw JDBC, with `JdbcTemplate` as the mature, battle-tested core and `JdbcClient` (Spring Framework 6.1) as the modern, fluent alternative, both eliminating connection management, exception handling, and resource cleanup boilerplate.

**Technical Definition**: **JdbcTemplate** is a powerful mechanism provided by the Spring Framework that simplifies the use of JDBC and helps to avoid common errors. It handles resource acquisition, connection management, error handling, and statement cleanup, so developers can focus on executing SQL statements and processing results. **JdbcClient** is the latest addition to Spring Framework 6.1. It provides a fluent interface with a unified facade for `JdbcTemplate` and `NamedParameterJdbcTemplate`, supporting chaining of query definition, parameter setting, and execution. Both are located in the `spring-jdbc` module and integrate seamlessly with Spring's transaction management and exception translation (converting checked `SQLException` to unchecked `DataAccessException`).

**Beginner-Friendly Explanation**: Spring's JDBC abstractions are like a professional kitchen assistant. `JdbcTemplate` is the seasoned veteran—it's been in kitchens for decades, handles every situation, and never lets you forget to turn off the stove (close connections). `JdbcClient` is the modern, streamlined assistant—it does the same job but with a more natural, chainable style. Both free you from the repetitive, error-prone parts of database work so you can focus on the recipe (your SQL).

### Purposes
- To eliminate JDBC boilerplate (connection management, exception handling, resource cleanup)
- To provide a fluent, chainable API with `JdbcClient` for modern Spring applications
- To translate checked `SQLException` to Spring's unchecked `DataAccessException` hierarchy
- To integrate seamlessly with Spring's declarative transaction management (`@Transactional`)
- To support both positional (`?`) and named (`:param`) parameters
- To automatically map result sets to Java records or POJOs

### Syntax Rules and Structure

#### Complete General Syntax: JdbcClient (Spring 6.1+)
```
JDBCLIENT (SPRING 6.1+)
│
├── 1. Create JdbcClient
│   ├── JdbcClient jdbcClient = JdbcClient.create(dataSource);
│   └── Or inject via @Autowired (Spring Boot auto-configures)
│
├── 2. Define SQL with Named Parameters
│   └── String sql = "SELECT * FROM users WHERE name = :name";
│
├── 3. Set Parameters (fluent)
│   ├── .param("name", "Alice")
│   └── .param("age", 30)
│
├── 4. Execute
│   ├── .query(Class<T>)     — SELECT → List<T>
│   ├── .query()             — SELECT → List<Map>
│   ├── .update()            — INSERT/UPDATE/DELETE
│   └── .optional()          — SELECT → Optional<T>
│
└── 5. Retrieve Results
    ├── .list()              — all results
    ├── .single()            — exactly one
    └── .optional()          — zero or one
```

#### Complete General Syntax: JdbcTemplate
```
JDBCTEMPLATE
│
├── 1. Create JdbcTemplate
│   └── JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
│
├── 2. Query for List of Objects
│   └── jdbcTemplate.query(sql, rowMapper, args)
│
├── 3. Query for Single Value
│   └── jdbcTemplate.queryForObject(sql, Integer.class, args)
│
├── 4. Update (INSERT/UPDATE/DELETE)
│   └── jdbcTemplate.update(sql, args)
│
├── 5. Batch Update
│   └── jdbcTemplate.batchUpdate(sql, batchArgs)
│
└── 6. Named Parameters
    └── NamedParameterJdbcTemplate.update(sql, paramMap)
```

#### Component Breakdown
| Feature | JdbcTemplate | JdbcClient |
|---------|-------------|------------|
| Introduced | Spring 1.0 | Spring 6.1 |
| API Style | Method calls | Fluent chaining |
| Parameter Binding | Positional (`?`) or `Map` | Named (`:param`) or positional |
| Result Mapping | `RowMapper<T>` | `.query(Class<T>)` or `RowMapper<T>` |
| Exception Translation | ✅ Yes | ✅ Yes |
| Transaction Integration | ✅ Yes | ✅ Yes |
| Batch Operations | ✅ Yes | Limited (use JdbcTemplate) |
| Stored Procedures | ✅ Yes | Limited (use JdbcTemplate) |

#### Syntax Rules
- **JdbcClient**: Use `.sql()` to define the query, `.param()` to bind named parameters, then `.query()` or `.update()` to execute
- **JdbcClient**: Automatically maps column names to record components (case-insensitive)
- **JdbcClient**: Thread-safe; can be shared across the application
- **JdbcTemplate**: Use `queryForObject()` for single values, `query()` for lists, `update()` for DML
- **JdbcTemplate**: `RowMapper<T>` implementation is required for complex mappings
- **Both**: Never concatenate user input; always use parameter binding

#### Constraints and Limitations
- **JdbcClient**: Does not support batch operations or stored procedures directly (use `JdbcTemplate` for these)
- **JdbcTemplate**: More verbose than `JdbcClient` for simple queries
- **Both**: Require Spring Framework dependency (`spring-jdbc`)
- **JdbcClient**: Requires Spring Framework 6.1+ (Spring Boot 3.2+)
- **JdbcTemplate**: Older API but still fully supported and widely used

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: JdbcClient CRUD Operations
```java
// JdbcClientDemo.java
import org.springframework.jdbc.core.simple.JdbcClient;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import javax.sql.DataSource;
import java.util.List;
import java.util.Optional;

public class JdbcClientDemo {
    
    record User(int id, String name, String email) {}
    
    public static void main(String[] args) {
        // Step 1: Create DataSource (H2 in-memory)
        DataSource dataSource = new DriverManagerDataSource(
            "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1", "sa", "");
        
        // Step 2: Create JdbcClient (thread-safe)
        JdbcClient jdbcClient = JdbcClient.create(dataSource);
        
        // Step 3: Create table
        jdbcClient.sql("""
            CREATE TABLE users (
                id BIGINT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(255),
                email VARCHAR(255)
            )
            """).update();
        
        // Step 4: Insert with named parameters
        jdbcClient.sql("INSERT INTO users (name, email) VALUES (:name, :email)")
            .param("name", "Alice")
            .param("email", "alice@example.com")
            .update();
        
        jdbcClient.sql("INSERT INTO users (name, email) VALUES (:name, :email)")
            .param("name", "Bob")
            .param("email", "bob@example.com")
            .update();
        
        System.out.println("Inserted 2 users");
        
        // Step 5: Query all users (auto-mapped to record)
        List<User> users = jdbcClient
            .sql("SELECT id, name, email FROM users ORDER BY id")
            .query(User.class)
            .list();
        
        System.out.println("\nAll users:");
        users.forEach(System.out::println);
        
        // Step 6: Query single user (optional)
        Optional<User> alice = jdbcClient
            .sql("SELECT id, name, email FROM users WHERE name = :name")
            .param("name", "Alice")
            .query(User.class)
            .optional();
        
        alice.ifPresent(u -> System.out.println("\nFound: " + u));
        
        // Step 7: Update
        int updated = jdbcClient
            .sql("UPDATE users SET email = :email WHERE name = :name")
            .param("email", "alice.new@example.com")
            .param("name", "Alice")
            .update();
        
        System.out.println("\nRows updated: " + updated);
        
        // Step 8: Count
        int count = jdbcClient
            .sql("SELECT COUNT(*) FROM users")
            .query(Integer.class)
            .single();
        
        System.out.println("Total users: " + count);
    }
}
```
**Expected Output**:
```
Inserted 2 users

All users:
User[id=1, name=Alice, email=alice@example.com]
User[id=2, name=Bob, email=bob@example.com]

Found: User[id=1, name=Alice, email=alice@example.com]

Rows updated: 1
Total users: 2
```
**Why This Output**: `JdbcClient.create(dataSource)` creates a thread-safe client. The fluent API chains `.sql()`, `.param()`, and `.update()` or `.query()`. Named parameters (`:name`, `:email`) are bound safely. The `query(User.class)` method automatically maps columns to record components by name (case-insensitive). The `optional()` method returns an `Optional<User>` for queries that may return zero or one row.

---

#### Example 2: JdbcTemplate with RowMapper
```java
// JdbcTemplateDemo.java
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

public class JdbcTemplateDemo {
    
    record User(int id, String name, String email) {}
    
    // Custom RowMapper for complex mapping
    static class UserRowMapper implements RowMapper<User> {
        @Override
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new User(
                rs.getInt("id"),
                rs.getString("name"),
                rs.getString("email")
            );
        }
    }
    
    public static void main(String[] args) {
        DataSource dataSource = new DriverManagerDataSource(
            "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1", "sa", "");
        
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        
        // Create table
        jdbcTemplate.execute("""
            CREATE TABLE users (
                id INT PRIMARY KEY,
                name VARCHAR(255),
                email VARCHAR(255)
            )
            """);
        
        // Insert with positional parameters
        jdbcTemplate.update(
            "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
            1, "Alice", "alice@example.com");
        
        jdbcTemplate.update(
            "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
            2, "Bob", "bob@example.com");
        
        System.out.println("Inserted 2 users via JdbcTemplate");
        
        // Query with RowMapper
        List<User> users = jdbcTemplate.query(
            "SELECT id, name, email FROM users ORDER BY id",
            new UserRowMapper());
        
        System.out.println("\nAll users:");
        users.forEach(System.out::println);
        
        // Query for single value
        int count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM users", Integer.class);
        
        System.out.println("\nTotal users: " + count);
        
        // Batch update
        jdbcTemplate.batchUpdate(
            "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
            List.of(
                new Object[]{3, "Charlie", "charlie@example.com"},
                new Object[]{4, "Diana", "diana@example.com"}
            ));
        
        System.out.println("Batch insert completed");
        
        int newCount = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM users", Integer.class);
        System.out.println("Total users after batch: " + newCount);
    }
}
```
**Expected Output**:
```
Inserted 2 users via JdbcTemplate

All users:
User[id=1, name=Alice, email=alice@example.com]
User[id=2, name=Bob, email=bob@example.com]

Total users: 2
Batch insert completed
Total users after batch: 4
```
**Why This Output**: `JdbcTemplate` uses positional parameters (`?`) and explicit `RowMapper<T>` implementations for complex mappings. `queryForObject()` retrieves single values (like counts). `batchUpdate()` efficiently inserts multiple rows in a single database round-trip. `JdbcTemplate` automatically manages connections, statements, and result sets, and translates `SQLException` to Spring's `DataAccessException`.

---

### Real-World Cases
- **Spring Boot Applications**: `JdbcClient` is auto-configured and available for injection
- **Legacy Spring Applications**: `JdbcTemplate` remains the standard for existing codebases
- **Batch Processing**: `JdbcTemplate.batchUpdate()` for bulk inserts and updates
- **Stored Procedures**: `SimpleJdbcCall` for calling stored procedures declaratively
- **Testing**: `JdbcTemplate` with H2 or Testcontainers for integration testing
- **Migration**: Teams migrating from raw JDBC to Spring JDBC report significant productivity gains

### References
- A Guide to Spring JdbcClient API - Baeldung - https://www.baeldung.com/spring-6-jdbcclient-api
- Spring JdbcClient - ZetCode - https://zetcode.cn/java/jdbcclient/
- Spring JdbcTemplate - Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/spring/5-spring-data-access/1-spring-jdbctemplate.mdx
- Spring JDBC Example - Centron - https://www.centron.de
- Java Persistence Frameworks Comparison - GitHub - https://github.com/bwajtr/java-persistence-frameworks-comparison

---

## Deprecation and Safety Notes

| Feature | Status | Notes |
|---------|--------|-------|
| Raw JDBC Boilerplate | ❌ Discouraged | Use Spring JDBC, JDBI, or JOOQ |
| `JdbcTemplate` | ✅ Active (Spring 1.0+) | Mature, battle-tested, widely used |
| `JdbcClient` | ✅ Active (Spring 6.1+) | Recommended for new code |
| `NamedParameterJdbcTemplate` | ✅ Active | Still useful for named parameters |
| `SimpleJdbcInsert` | ✅ Active | For auto-generated key retrieval |
| `SimpleJdbcCall` | ✅ Active | For stored procedures |
| JDBI Fluent API | ✅ Active | Lightweight, SQL-focused |
| JDBI SQL Object API | ✅ Active | Declarative DAO pattern |
| JOOQ | ✅ Active | Type-safe SQL DSL; code generation required |
| JOOQ Commercial License | ⚠️ Required for Oracle, SQL Server, DB2 | Open-source for PostgreSQL, MySQL, H2 |
| Spring `@Transactional` | ✅ Active | Declarative transaction management |

---

## References

### Official Documentation
- JDBI Five Minute Introduction - https://jdbi.org/jdbi2/five_minute_intro/
- JDBI Fluent Queries - https://jdbi.org/jdbi2/fluent_queries/
- jOOQ Manual - https://www.jooq.org/doc/3.17/manual-pdf/jOOQ-manual-3.17.pdf
- Spring JdbcClient Javadoc - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/simple/JdbcClient.html

### Tutorials and Guides
- A Guide to Spring JdbcClient API - Baeldung - https://www.baeldung.com/spring-6-jdbcclient-api
- Spring JdbcClient - ZetCode - https://zetcode.cn/java/jdbcclient/
- jOOQ Introduction - Baeldung - https://baeldung.cn/jooq-intro
- Spring JdbcTemplate - Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/spring/5-spring-data-access/1-spring-jdbctemplate.mdx
- Spring JDBC Example - Centron - https://www.centron.de

### Comparisons and Analysis
- Java Persistence Frameworks Comparison - GitHub - https://github.com/bwajtr/java-persistence-frameworks-comparison
- Raw JDBC and Spring JDBC Template - Stack Overflow - https://stackoverflow.com/revisions/0408968a-95c9-4540-b9e8-4fb65db35c9e/view-source
- Traditional JDBC Development Problems - Alibaba Cloud - https://developer.aliyun.com/ask/327885

### Additional Resources
- HikariCP Best Practices for Oracle Database and Spring Boot - https://blogs.oracle.com/developers/hikaricp-best-practices-for-oracle-database-and-spring-boot
- Universal Connection Pool Developer's Guide - https://docs.oracle.com/en/database/oracle/oracle-database/26/jjucp/
- Replace DBCP2 with HikariCP in RDBMS Plugin - Apache Drill - https://issues.apache.org/jira/browse/DRILL-7639