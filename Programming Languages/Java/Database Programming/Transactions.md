# Transaction Management & Isolation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition
Transaction management in JDBC is the mechanism for grouping multiple SQL statements into a single atomic unit of work, ensuring that all operations either succeed together (commit) or fail together (rollback), while isolation levels control how concurrent transactions interact.

### Technical Definition
A database transaction is a unit of work that groups one or more SQL statements so they are treated as a single, all-or-nothing operation . JDBC supports transactions through the `Connection` interface, which provides `setAutoCommit()`, `commit()`, `rollback()`, and `setSavepoint()` methods. By default, JDBC operates in auto-commit mode, meaning each SQL statement is treated as a separate transaction and automatically committed after execution . To use transactions properly, auto-commit must be disabled, and the application must explicitly commit or roll back changes . Transaction isolation levels (defined by `Connection.TRANSACTION_*` constants) control the degree to which concurrent transactions are isolated from each other, preventing anomalies such as dirty reads, non-repeatable reads, and phantom reads .

### Beginner-Friendly Explanation
Imagine you're transferring money between two bank accounts. You need to subtract from one account and add to the other. If the system crashes after the subtraction but before the addition, the money disappears. A transaction is like a safety envelope: either both operations succeed, or neither happens. JDBC's auto-commit mode automatically wraps each statement in its own envelope—great for simple operations, but useless for multi-step workflows. Disabling auto-commit lets you put multiple statements in one envelope and decide when to seal it (commit) or tear it up (rollback).

### Key Characteristics
- **Atomic**: All operations in a transaction succeed or fail as a unit
- **Consistent**: Transactions take the database from one valid state to another
- **Isolated**: Concurrent transactions don't interfere with each other
- **Durable**: Once committed, changes persist even after a crash
- **Granular**: Savepoints allow partial rollback within a transaction
- **Distributed**: XA transactions coordinate across multiple databases

### Prerequisites
- Solid understanding of basic JDBC (Connection, Statement, ResultSet)
- Knowledge of SQL (INSERT, UPDATE, DELETE, SELECT)
- Familiarity with ACID properties
- Understanding of try-with-resources for resource cleanup

### Related Programming Areas
- **Connection Pooling**: Transaction scope and connection ownership
- **ORM Frameworks**: Hibernate/JPA manage transactions declaratively
- **Spring Transaction Management**: `@Transactional` annotation abstraction
- **Distributed Systems**: JTA and XA for multi-database transactions

### Core Concepts Overview
1. **Transaction Control**: Disabling auto-commit to define transaction boundaries
2. **Atomic Operations**: Reliable persistence via `commit()` and `rollback()`
3. **Granular Recovery**: Savepoints for partial rollback within transactions
4. **Transaction Isolation Levels**: Mitigating dirty reads, non-repeatable reads, and phantom reads
5. **Distributed Transactions**: Two-phase commits, XA transactions, and `XADataSource`

---

## Core Concept 1: Transaction Control

### Definitions
**Core Definition**: Transaction control is the process of disabling JDBC's default auto-commit mode to manually define the boundaries of a multi-statement transaction.

**Technical Definition**: When an application creates a connection, the connection has transaction auto-commit enabled by default . In this mode, each SQL statement is treated as a separate transaction and automatically committed after execution . To define a multi-statement transaction, the application calls `Connection.setAutoCommit(false)`, which begins a new transaction. All subsequent SQL statements are part of this transaction until `commit()` or `rollback()` is called. If auto-commit mode is on and you perform a `COMMIT` or `ROLLBACK` operation, you get an error: "Could not commit with auto-commit set on" . If auto-commit mode is disabled and you close the connection without explicitly committing or rolling back your last changes, an implicit `COMMIT` operation is run .

**Beginner-Friendly Explanation**: Auto-commit mode is like a vending machine—every button press immediately dispenses and charges you. Disabling auto-commit is like a shopping cart—you add items one by one, and only when you checkout (commit) does the purchase happen. If you change your mind, you can empty the cart (rollback) and nothing was bought.

### Purposes
- To group multiple SQL statements into a single atomic unit
- To ensure data consistency across related operations
- To provide control over when changes become permanent
- To enable rollback of all changes if any operation fails
- To support complex business workflows (e.g., fund transfers, order processing)

### Syntax Rules and Structure
#### Complete General Syntax: Transaction Boundary Control
```
TRANSACTION BOUNDARY CONTROL
│
├── 1. Disable Auto-Commit
│   └── connection.setAutoCommit(false)
│       └── Begins a new transaction
│
├── 2. Execute Multiple Statements
│   ├── Statement 1
│   ├── Statement 2
│   └── Statement N — all part of the same transaction
│
├── 3. End Transaction
│   ├── connection.commit()   — make changes permanent
│   └── connection.rollback() — undo all changes
│
└── 4. Restore Auto-Commit (optional)
    └── connection.setAutoCommit(true)
```

#### Component Breakdown
| Method | Purpose | Notes |
|--------|---------|-------|
| `setAutoCommit(false)` | Begin transaction | Disables per-statement commit |
| `commit()` | End transaction (success) | Makes changes permanent |
| `rollback()` | End transaction (failure) | Undoes all changes |
| `getAutoCommit()` | Check current mode | Returns boolean |
| `setAutoCommit(true)` | Restore default mode | Commits pending transaction |

#### Syntax Rules
- Auto-commit must be disabled before starting a multi-statement transaction
- `commit()` or `rollback()` ends the current transaction and begins a new one (if auto-commit remains off)
- Calling `commit()` or `rollback()` with auto-commit ON throws `SQLException` 
- Closing a connection with auto-commit OFF and uncommitted changes triggers an implicit commit 
- DDL operations (CREATE, ALTER, DROP) are always auto-committed regardless of auto-commit setting

#### Constraints and Limitations
- A transaction should be as short as possible to avoid lock contention
- Long-running transactions can cause deadlocks and block other users
- Not all databases support transactional DDL
- Auto-commit should be restored after transaction completes

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Transaction with Commit and Rollback
```java
// BasicTransactionDemo.java
import java.sql.*;

public class BasicTransactionDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/bankdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            // Step 1: Disable auto-commit to start a transaction
            conn.setAutoCommit(false);
            System.out.println("Auto-commit disabled — transaction started");
            
            try (Statement stmt = conn.createStatement()) {
                // Step 2: Execute multiple statements as one atomic unit
                stmt.executeUpdate(
                    "UPDATE accounts SET balance = balance - 500 WHERE id = 1");
                System.out.println("Debited $500 from account 1");
                
                stmt.executeUpdate(
                    "UPDATE accounts SET balance = balance + 500 WHERE id = 2");
                System.out.println("Credited $500 to account 2");
                
                // Step 3: Commit — both changes become permanent
                conn.commit();
                System.out.println("Transaction committed successfully");
                
            } catch (SQLException e) {
                // Step 4: Rollback on any error
                conn.rollback();
                System.err.println("Transaction rolled back: " + e.getMessage());
            }
            
            // Step 5: Restore auto-commit mode
            conn.setAutoCommit(true);
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
Auto-commit disabled — transaction started
Debited $500 from account 1
Credited $500 to account 2
Transaction committed successfully
```
**Why This Output**: `setAutoCommit(false)` begins the transaction. Both UPDATE statements execute within the same transaction. `commit()` makes both changes permanent simultaneously. If any statement fails, `rollback()` undoes all changes.

---

#### Example 2: Transaction with Rollback on Failure
```java
// RollbackTransactionDemo.java
import java.sql.*;

public class RollbackTransactionDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/bankdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            conn.setAutoCommit(false);
            System.out.println("Transaction started");
            
            try (Statement stmt = conn.createStatement()) {
                // Debit from account 1
                stmt.executeUpdate(
                    "UPDATE accounts SET balance = balance - 1000 WHERE id = 1");
                System.out.println("Debited $1000 from account 1");
                
                // This will fail — account 999 doesn't exist
                int rows = stmt.executeUpdate(
                    "UPDATE accounts SET balance = balance + 1000 WHERE id = 999");
                
                if (rows == 0) {
                    throw new SQLException("Target account not found");
                }
                
                conn.commit();
                
            } catch (SQLException e) {
                conn.rollback();
                System.err.println("Rolled back: " + e.getMessage());
                System.err.println("Account 1 balance restored");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
Transaction started
Debited $1000 from account 1
Rolled back: Target account not found
Account 1 balance restored
```
**Why This Output**: The first UPDATE succeeds, but the second UPDATE affects 0 rows (account 999 doesn't exist). The application throws an exception, triggering `rollback()`, which undoes the debit. The database returns to its original state.

### Real-World Cases
- **Banking Systems**: Fund transfers require atomic debit/credit operations
- **E-Commerce**: Order placement involves inventory decrement, payment processing, and order creation
- **Inventory Management**: Stock adjustments must be consistent across multiple warehouses

### References
- JDBC Developer's Guide: About Committing Changes - https://docs.oracle.com/en/database/oracle/oracle-database/26/jjdbc/JDBC-getting-started.html
- Using Transactions in Java JDBC - Cleverence - https://www.cleverence.com/articles/oracle-documentation/using-transactions-the-java-tutorials-jdbc-5831/

---

## Core Concept 2: Atomic Operations

### Definitions
**Core Definition**: Atomic operations are database operations that either complete entirely or have no effect at all, orchestrated through explicit `commit()` and `rollback()` calls to ensure reliable persistence workflows.

**Technical Definition**: Atomicity is one of the four ACID properties. In JDBC, atomicity is achieved by disabling auto-commit, executing multiple SQL statements, and then either committing all changes (`conn.commit()`) or rolling back all changes (`conn.rollback()`) . A `COMMIT` or `ROLLBACK` operation affects all DML statements run since the last `COMMIT` or `ROLLBACK` . If a transaction is rolled back, the database discards all changes made during the transaction. If committed, all changes are persisted to durable storage.

**Beginner-Friendly Explanation**: Atomicity is the "all-or-nothing" rule. Think of a wedding vow: "I do" (commit) means you're married—permanently. If you don't say "I do" (rollback), nothing happened. There's no middle ground—you can't be "half-married." JDBC transactions work the same way: either all statements in the transaction take effect, or none do.

### Purposes
- To ensure that related operations succeed or fail as a unit
- To prevent partial writes that leave data in an inconsistent state
- To provide a recovery mechanism (rollback) when failures occur
- To guarantee data integrity across multi-step business processes
- To support complex workflows with multiple interdependent operations

### Syntax Rules and Structure
#### Complete General Syntax: Atomic Operation Workflow
```
ATOMIC OPERATION WORKFLOW
│
├── 1. BEGIN
│   └── conn.setAutoCommit(false)
│
├── 2. EXECUTE (multiple statements)
│   ├── Statement 1
│   ├── Statement 2
│   └── Statement N
│
├── 3. DECISION POINT
│   ├── All succeeded? → conn.commit()
│   └── Any failed?     → conn.rollback()
│
└── 4. CLEANUP
    └── conn.setAutoCommit(true)  // optional
```

#### Component Breakdown
| Operation | Effect | Reversible? |
|-----------|--------|-------------|
| `commit()` | Persist all changes | No |
| `rollback()` | Discard all changes | No (changes undone) |
| Implicit commit | Auto-commit or connection close | Varies |

#### Syntax Rules
- `commit()` and `rollback()` are mutually exclusive endpoints
- Only one `commit()` or `rollback()` is allowed per transaction
- After `commit()` or `rollback()`, a new transaction begins automatically (if auto-commit remains off)
- Exceptions during `commit()` may require retry or manual recovery
- Use `try-catch` with `rollback()` in the catch block for deterministic error handling

#### Constraints and Limitations
- Committed transactions cannot be undone (no "un-commit")
- Rollback only works for DML operations; DDL is auto-committed
- Distributed transactions require XA (see Core Concept 5)
- Long transactions increase lock contention and rollback cost

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Complete Atomic Workflow
```java
// AtomicOperationsDemo.java
import java.sql.*;

public class AtomicOperationsDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/inventory";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            conn.setAutoCommit(false);
            
            try (Statement stmt = conn.createStatement()) {
                // Step 1: Check inventory
                ResultSet rs = stmt.executeQuery(
                    "SELECT quantity FROM products WHERE id = 101");
                rs.next();
                int stock = rs.getInt("quantity");
                System.out.println("Current stock: " + stock);
                
                if (stock < 5) {
                    throw new SQLException("Insufficient stock");
                }
                
                // Step 2: Decrement inventory
                stmt.executeUpdate(
                    "UPDATE products SET quantity = quantity - 5 WHERE id = 101");
                System.out.println("Decremented inventory by 5");
                
                // Step 3: Create order record
                stmt.executeUpdate(
                    "INSERT INTO orders (product_id, quantity, status) " +
                    "VALUES (101, 5, 'PENDING')");
                System.out.println("Order created");
                
                // Step 4: Commit — both operations take effect
                conn.commit();
                System.out.println("Transaction committed");
                
            } catch (SQLException e) {
                conn.rollback();
                System.err.println("Rolled back: " + e.getMessage());
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
Current stock: 50
Decremented inventory by 5
Order created
Transaction committed
```
**Why This Output**: The inventory check, decrement, and order creation are all part of one atomic transaction. Either all three succeed, or none do. The `commit()` ensures both the inventory update and order insertion are persisted together.

### Real-World Cases
- **Payment Processing**: Debit buyer, credit seller, record transaction—all atomically
- **Flight Booking**: Reserve seat, charge payment, issue ticket—atomic
- **Supply Chain**: Update inventory, create shipment, notify warehouse—atomic

### References
- ACID in Practice - Cleverence - https://www.cleverence.com/articles/oracle-documentation/using-transactions-the-java-tutorials-jdbc-5831/
- JDBC Developer's Guide - https://docs.oracle.com/en/database/oracle/oracle-database/26/jjdbc/

---

## Core Concept 3: Granular Recovery (Savepoints)

### Definitions
**Core Definition**: Savepoints are markers within a transaction that allow partial rollback to a specific point without discarding the entire transaction, enabling granular recovery in long-running transaction blocks.

**Technical Definition**: The JDBC 3.0 API adds the method `Connection.setSavepoint`, which sets a savepoint within the current transaction . The `Connection.rollback` method has been overloaded to take a savepoint argument . When a transaction is rolled back to a savepoint, changes made after the savepoint are undone, but changes made before the savepoint remain intact . Savepoints are only available when auto-commit is disabled . The `setSavepoint()` method returns a `Savepoint` object that can be used in `rollback(Savepoint)`.

**Beginner-Friendly Explanation**: A savepoint is like a "checkpoint" in a video game. If you die after reaching a checkpoint, you don't restart the entire level—you restart from the checkpoint. In JDBC, savepoints let you roll back part of a transaction without losing everything. This is useful when a long transaction has independent sub-tasks where one can fail without invalidating the others.

### Purposes
- To enable partial rollback within a transaction
- To recover from failures in specific sub-tasks without losing prior work
- To support complex workflows with independent steps
- To reduce the cost of rollback in long-running transactions
- To provide fine-grained error recovery

### Syntax Rules and Structure
#### Complete General Syntax: Savepoint Operations
```
SAVEPOINT OPERATIONS
│
├── 1. Set Savepoint
│   ├── Savepoint sp = conn.setSavepoint()
│   └── Savepoint sp = conn.setSavepoint("name")
│
├── 2. Execute Statements
│   ├── Statement after savepoint
│   └── Another statement
│
├── 3. Rollback to Savepoint (if needed)
│   └── conn.rollback(sp)  — undoes changes after sp
│
└── 4. Continue or Commit
    ├── conn.commit()  — commit remaining changes
    └── conn.rollback() — rollback entire transaction
```

#### Component Breakdown
| Method | Purpose | Returns |
|--------|---------|---------|
| `setSavepoint()` | Create unnamed savepoint | `Savepoint` |
| `setSavepoint(String)` | Create named savepoint | `Savepoint` |
| `rollback(Savepoint)` | Rollback to savepoint | void |
| `releaseSavepoint(Savepoint)` | Release savepoint | void |

#### Syntax Rules
- Savepoints require auto-commit to be disabled 
- `rollback(Savepoint)` undoes changes made after the savepoint but preserves earlier changes
- Savepoints can be named for easier debugging
- Multiple savepoints can exist in one transaction
- `releaseSavepoint()` removes a savepoint without rolling back

#### Constraints and Limitations
- Not all databases support savepoints (check `DatabaseMetaData.supportsSavepoints()`)
- Savepoints consume database resources
- Rolling back to a savepoint does not release locks acquired after the savepoint
- Savepoints cannot be used with XA distributed transactions 

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Savepoint Rollback
```java
// SavepointDemo.java
import java.sql.*;

public class SavepointDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/testdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            conn.setAutoCommit(false);
            System.out.println("Transaction started");
            
            try (Statement stmt = conn.createStatement()) {
                // Step 1: First operation
                stmt.executeUpdate(
                    "INSERT INTO orders (id, customer_id, status) VALUES (1, 100, 'NEW')");
                System.out.println("Inserted order 1");
                
                // Step 2: Set savepoint
                Savepoint sp1 = conn.setSavepoint("after_order_1");
                System.out.println("Savepoint set");
                
                // Step 3: Second operation
                stmt.executeUpdate(
                    "INSERT INTO orders (id, customer_id, status) VALUES (2, 100, 'NEW')");
                System.out.println("Inserted order 2");
                
                // Step 4: Third operation — this fails
                try {
                    stmt.executeUpdate(
                        "INSERT INTO orders (id, customer_id, status) VALUES (3, 999, 'NEW')");
                } catch (SQLException e) {
                    System.out.println("Order 3 failed: " + e.getMessage());
                    
                    // Step 5: Rollback to savepoint — undo order 2 only
                    conn.rollback(sp1);
                    System.out.println("Rolled back to savepoint — order 2 undone");
                }
                
                // Step 6: Commit — order 1 remains
                conn.commit();
                System.out.println("Committed — order 1 preserved");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
Transaction started
Inserted order 1
Savepoint set
Inserted order 2
Order 3 failed: Duplicate entry '3' for key 'PRIMARY'
Rolled back to savepoint — order 2 undone
Committed — order 1 preserved
```
**Why This Output**: Order 1 is inserted before the savepoint. Order 2 is inserted after the savepoint. When order 3 fails, `rollback(sp1)` undoes order 2 but preserves order 1. The final `commit()` persists order 1.

---

#### Example 2: Multiple Savepoints
```java
// MultipleSavepointsDemo.java
import java.sql.*;

public class MultipleSavepointsDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/testdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            conn.setAutoCommit(false);
            
            try (Statement stmt = conn.createStatement()) {
                stmt.executeUpdate("INSERT INTO logs (msg) VALUES ('step 1')");
                Savepoint sp1 = conn.setSavepoint("sp1");
                
                stmt.executeUpdate("INSERT INTO logs (msg) VALUES ('step 2')");
                Savepoint sp2 = conn.setSavepoint("sp2");
                
                stmt.executeUpdate("INSERT INTO logs (msg) VALUES ('step 3')");
                Savepoint sp3 = conn.setSavepoint("sp3");
                
                stmt.executeUpdate("INSERT INTO logs (msg) VALUES ('step 4')");
                
                // Rollback to sp2 — undo steps 3 and 4
                conn.rollback(sp2);
                System.out.println("Rolled back to sp2");
                
                // Commit — steps 1 and 2 remain
                conn.commit();
                System.out.println("Committed");
                
                // Verify
                ResultSet rs = stmt.executeQuery(
                    "SELECT msg FROM logs ORDER BY id");
                System.out.println("Remaining logs:");
                while (rs.next()) {
                    System.out.println("  " + rs.getString("msg"));
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
**Expected Output**:
```
Rolled back to sp2
Committed
Remaining logs:
  step 1
  step 2
```
**Why This Output**: Four steps are inserted. Rolling back to `sp2` undoes steps 3 and 4. Steps 1 and 2 remain and are committed.

### Real-World Cases
- **Batch Processing**: Process a batch of records; if one fails, roll back only that record
- **Multi-Step Forms**: Save user input at each step; allow correction without losing all data
- **ETL Pipelines**: Process data in chunks; roll back a failed chunk without reprocessing all data

### References
- Setting and Rolling Back to a Savepoint - Apache Derby - https://db.apache.org/derby/docs/10.3/ref/crefjavsavesetroll.html
- Using Savepoints - Microsoft JDBC Driver - https://learn.microsoft.com/en-us/sql/connect/jdbc/using-savepoints

---

## Core Concept 4: Transaction Isolation Levels

### Definitions
**Core Definition**: Transaction isolation levels define the degree to which one transaction must be isolated from resource or data modifications made by other concurrent transactions, controlling the occurrence of dirty reads, non-repeatable reads, and phantom reads.

**Technical Definition**: The JDBC API defines five transaction isolation levels via `Connection` constants: `TRANSACTION_NONE`, `TRANSACTION_READ_UNCOMMITTED`, `TRANSACTION_READ_COMMITTED`, `TRANSACTION_REPEATABLE_READ`, and `TRANSACTION_SERIALIZABLE` . A **dirty read** occurs when transaction B accesses a row that was updated by transaction A, but transaction A later rolls back the updates . A **non-repeatable read** occurs when transaction A retrieves a row, transaction B subsequently updates the row, and transaction A re-reads the row and sees different values . A **phantom read** occurs when transaction A retrieves a set of rows, transaction B inserts new rows, and transaction A re-executes the query and sees the new rows . Not all databases support all isolation levels; use `DatabaseMetaData.supportsTransactionIsolationLevel()` to verify .

**Beginner-Friendly Explanation**: Isolation levels are like different "shielding levels" for transactions. `READ_UNCOMMITTED` is like no shielding at all—you can see other transactions' uncommitted changes (dirty reads). `READ_COMMITTED` is like basic shielding—you only see committed data, but data can change between reads. `REPEATABLE_READ` is like stronger shielding—once you read a value, it won't change. `SERIALIZABLE` is like maximum shielding—transactions run as if they were alone, one after another.

### Purposes
- To control the trade-off between data consistency and concurrency performance
- To prevent specific anomalies (dirty reads, non-repeatable reads, phantom reads)
- To allow applications to choose the appropriate isolation for their needs
- To ensure data integrity in concurrent environments
- To balance correctness requirements against throughput

### Syntax Rules and Structure
#### Complete General Syntax: Isolation Level Configuration
```
ISOLATION LEVEL CONFIGURATION
│
├── 1. Set Isolation Level
│   └── conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED)
│
├── 2. Get Current Isolation Level
│   └── int level = conn.getTransactionIsolation()
│
├── 3. Check Support
│   └── DatabaseMetaData.supportsTransactionIsolationLevel(level)
│
└── 4. Constants
    ├── TRANSACTION_NONE             (0) — no transactions
    ├── TRANSACTION_READ_UNCOMMITTED (1) — dirty reads possible
    ├── TRANSACTION_READ_COMMITTED   (2) — dirty reads prevented
    ├── TRANSACTION_REPEATABLE_READ  (4) — + non-repeatable reads prevented
    └── TRANSACTION_SERIALIZABLE     (8) — all anomalies prevented
```

#### Component Breakdown
| Isolation Level | Constant | Dirty Read | Non-Repeatable Read | Phantom Read |
|----------------|----------|------------|---------------------|--------------|
| READ_UNCOMMITTED | 1 | Possible | Possible | Possible  |
| READ_COMMITTED | 2 | Prevented | Possible | Possible  |
| REPEATABLE_READ | 4 | Prevented | Prevented | Possible  |
| SERIALIZABLE | 8 | Prevented | Prevented | Prevented  |

#### Syntax Rules
- Set isolation level **before** beginning a transaction
- You cannot call `setTransactionIsolation` during an active transaction 
- The default isolation level is database-specific
- Use `DatabaseMetaData.supportsTransactionIsolationLevel()` to verify support 
- Higher isolation levels reduce concurrency and increase lock contention

#### Constraints and Limitations
- Not all databases support all isolation levels 
- Setting isolation level per `getConnection()` call can degrade performance 
- Changing isolation level on pooled connections can pollute the pool 
- `SERIALIZABLE` may cause deadlocks and timeouts in high-concurrency systems
- Isolation levels are not standardized across all databases (e.g., Oracle's "SERIALIZABLE" is snapshot isolation)

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Setting Isolation Level
```java
// IsolationLevelDemo.java
import java.sql.*;

public class IsolationLevelDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/testdb";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass")) {
            // Check current isolation level
            int current = conn.getTransactionIsolation();
            System.out.println("Default isolation: " + levelName(current));
            
            // Check if SERIALIZABLE is supported
            DatabaseMetaData meta = conn.getMetaData();
            boolean supportsSerializable = meta.supportsTransactionIsolationLevel(
                Connection.TRANSACTION_SERIALIZABLE);
            System.out.println("Supports SERIALIZABLE: " + supportsSerializable);
            
            // Set to REPEATABLE_READ
            conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
            System.out.println("Isolation set to: " + 
                levelName(conn.getTransactionIsolation()));
            
            // Now begin transaction
            conn.setAutoCommit(false);
            
            try (Statement stmt = conn.createStatement()) {
                stmt.executeQuery("SELECT * FROM users");
                conn.commit();
            }
            
            // Restore isolation level (good practice for pools)
            conn.setTransactionIsolation(current);
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    static String levelName(int level) {
        switch (level) {
            case Connection.TRANSACTION_NONE: return "NONE";
            case Connection.TRANSACTION_READ_UNCOMMITTED: return "READ_UNCOMMITTED";
            case Connection.TRANSACTION_READ_COMMITTED: return "READ_COMMITTED";
            case Connection.TRANSACTION_REPEATABLE_READ: return "REPEATABLE_READ";
            case Connection.TRANSACTION_SERIALIZABLE: return "SERIALIZABLE";
            default: return "UNKNOWN(" + level + ")";
        }
    }
}
```
**Expected Output**:
```
Default isolation: READ_COMMITTED
Supports SERIALIZABLE: true
Isolation set to: REPEATABLE_READ
```
**Why This Output**: The default isolation level is database-dependent (MySQL defaults to `READ_COMMITTED`). `supportsTransactionIsolationLevel()` verifies support. `setTransactionIsolation()` changes the level for the connection.

### Real-World Cases
- **Banking**: `SERIALIZABLE` for account balance queries to prevent phantom reads
- **E-Commerce**: `READ_COMMITTED` for product catalog browsing (balance consistency vs. performance)
- **Reporting**: `READ_UNCOMMITTED` for approximate analytics where performance matters more than precision
- **Inventory**: `REPEATABLE_READ` for stock level checks

### References
- Handling Transactions with Databases - Oracle GlassFish - https://docs.oracle.com/cd/E18930_01/html/821-2418/giybi.html
- SQLJ Developer's Guide: Isolation Levels - https://docs.oracle.com/en/database/oracle/oracle-database/26/jsqlj/

---

## Core Concept 5: Distributed Transactions

### Definitions
**Core Definition**: Distributed transactions are transactions that span multiple databases or resource managers, coordinated using the XA (eXtended Architecture) standard and two-phase commit (2PC) protocol via the `XADataSource` interface.

**Technical Definition**: X/Open XA is a standard interface for performing two-phase commit across different resource managers, like databases . XA support means the driver comes with an implementation of the `javax.sql.XADataSource` interface . You need two-phase commit if you want to update two databases in a single atomic transaction . The XA transaction uses two-phase commit (2PC) to ensure that the database participates correctly in a global transaction . In a JTA implementation, the transaction manager commits the distributed branches of a global transaction by using a two-phase commit protocol . The transaction manager coordinates the two-phase commit protocol across all servers, ensuring atomicity across the entire distributed transaction .

**Beginner-Friendly Explanation**: A distributed transaction is like coordinating a group project across multiple offices. One person (transaction manager) asks each office (database) "Are you ready to submit your work?" (prepare phase). If everyone says yes, the coordinator says "Submit!" (commit phase). If anyone says no, everyone discards their work (rollback). The XA standard defines how the coordinator and offices communicate, and `XADataSource` is the interface each database implements.

### Purposes
- To ensure atomicity across multiple databases or resource managers
- To coordinate transactions spanning heterogeneous systems
- To support enterprise integration scenarios
- To provide reliable data consistency in distributed architectures
- To enable XA-compliant resource enlistment

### Syntax Rules and Structure
#### Complete General Syntax: XA Transaction Workflow
```
XA TRANSACTION WORKFLOW
│
├── 1. Configure XADataSource
│   └── XADataSource xaDS = (XADataSource) ctx.lookup("jdbc/XADataSource")
│
├── 2. Obtain XAConnection
│   └── XAConnection xaConn = xaDS.getXAConnection()
│
├── 3. Obtain XAResource and Connection
│   ├── XAResource xaRes = xaConn.getXAResource()
│   └── Connection conn = xaConn.getConnection()
│
├── 4. Begin Transaction
│   └── xaRes.start(xid, XAResource.TMNOFLAGS)
│
├── 5. Execute Operations
│   └── Use conn for SQL operations
│
├── 6. End Branch
│   └── xaRes.end(xid, XAResource.TMSUCCESS)
│
├── 7. Prepare Phase
│   └── xaRes.prepare(xid)
│
├── 8. Commit or Rollback
│   ├── xaRes.commit(xid, false)  — two-phase commit
│   └── xaRes.rollback(xid)       — rollback
│
└── 9. Close
    ├── conn.close()
    └── xaConn.close()
```

#### Component Breakdown
| Interface | Purpose | Package |
|-----------|---------|---------|
| `XADataSource` | Creates XA connections | `javax.sql` |
| `XAConnection` | Provides XAResource and Connection | `javax.sql` |
| `XAResource` | Participates in 2PC | `javax.transaction.xa` |
| `Xid` | Transaction identifier | `javax.transaction.xa` |

#### Syntax Rules
- `XADataSource` is obtained from a JNDI lookup or configured directly
- `XAConnection` provides both a `Connection` (for SQL) and an `XAResource` (for transaction control)
- `xaRes.start()` begins a transaction branch
- `xaRes.end()` ends the branch (work is done)
- `xaRes.prepare()` asks the database if it's ready to commit
- `xaRes.commit(xid, false)` performs two-phase commit
- `xaRes.rollback(xid)` rolls back the transaction branch

#### Constraints and Limitations
- XA transactions are complex and require a transaction manager
- Not all databases support XA
- XA transactions have higher overhead than local transactions
- Do not execute DDL statements within an XA transaction 
- XA transactions may require `--add-opens` for JDK internals
- Two-phase commit introduces latency and potential for in-doubt transactions

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: XA Transaction with Two Databases
```java
// XATransactionDemo.java
import javax.sql.*;
import javax.transaction.xa.*;
import java.sql.*;

public class XATransactionDemo {
    public static void main(String[] args) throws Exception {
        // Assume XADataSource instances are configured via JNDI or directly
        // XADataSource xaDS1 = ...;
        // XADataSource xaDS2 = ...;
        
        System.out.println("=== XA Distributed Transaction Demo ===");
        
        // Step 1: Obtain XAConnections
        // XAConnection xaConn1 = xaDS1.getXAConnection();
        // XAConnection xaConn2 = xaDS2.getXAConnection();
        
        // Step 2: Get XAResource and Connection from each
        // XAResource xaRes1 = xaConn1.getXAResource();
        // XAResource xaRes2 = xaConn2.getXAResource();
        // Connection conn1 = xaConn1.getConnection();
        // Connection conn2 = xaConn2.getConnection();
        
        // Step 3: Create transaction ID
        // Xid xid1 = new MyXid(100, new byte[]{0x01}, new byte[]{0x02});
        // Xid xid2 = new MyXid(100, new byte[]{0x01}, new byte[]{0x03});
        
        // Step 4: Start branches
        // xaRes1.start(xid1, XAResource.TMNOFLAGS);
        // xaRes2.start(xid2, XAResource.TMNOFLAGS);
        
        // Step 5: Execute operations
        // conn1.createStatement().executeUpdate(
        //     "UPDATE accounts SET balance = balance - 100 WHERE id = 1");
        // conn2.createStatement().executeUpdate(
        //     "UPDATE accounts SET balance = balance + 100 WHERE id = 1");
        
        // Step 6: End branches
        // xaRes1.end(xid1, XAResource.TMSUCCESS);
        // xaRes2.end(xid2, XAResource.TMSUCCESS);
        
        // Step 7: Prepare
        // int prepare1 = xaRes1.prepare(xid1);
        // int prepare2 = xaRes2.prepare(xid2);
        
        // Step 8: Commit if both prepared
        // if (prepare1 == XAResource.XA_OK && prepare2 == XAResource.XA_OK) {
        //     xaRes1.commit(xid1, false);
        //     xaRes2.commit(xid2, false);
        //     System.out.println("Distributed transaction committed");
        // } else {
        //     xaRes1.rollback(xid1);
        //     xaRes2.rollback(xid2);
        //     System.out.println("Distributed transaction rolled back");
        // }
        
        System.out.println("XA transaction workflow:");
        System.out.println("1. Obtain XAConnection from XADataSource");
        System.out.println("2. Get XAResource and Connection");
        System.out.println("3. start() — begin transaction branch");
        System.out.println("4. Execute SQL operations");
        System.out.println("5. end() — end transaction branch");
        System.out.println("6. prepare() — ask database if ready");
        System.out.println("7. commit() or rollback() — finalize");
        System.out.println("\nRequires a JTA transaction manager");
        System.out.println("(e.g., Atomikos, Narayana, Bitronix)");
    }
}
```
**Expected Output**:
```
=== XA Distributed Transaction Demo ===
XA transaction workflow:
1. Obtain XAConnection from XADataSource
2. Get XAResource and Connection
3. start() — begin transaction branch
4. Execute SQL operations
5. end() — end transaction branch
6. prepare() — ask database if ready
7. commit() or rollback() — finalize

Requires a JTA transaction manager
(e.g., Atomikos, Narayana, Bitronix)
```
**Why This Output**: The XA workflow requires explicit management of transaction branches via `XAResource`. Each database is prepared independently, then committed together. In practice, a JTA transaction manager automates this process.

### Real-World Cases
- **Enterprise Integration**: Order system in one database, inventory in another
- **Microservices**: Distributed transactions across service boundaries
- **Legacy System Integration**: XA bridges old and new databases
- **Financial Systems**: Multi-bank transfers require distributed atomicity

### References
- Distributed Transactions - Oracle JDBC Developer's Guide - https://docs.oracle.com/en/database/oracle/oracle-database/26/jjdbc/
- Two-Phase Commit with JDBC - PostgreSQL Mailing List - https://www.postgresql.org/message-id/CAF-4BpOGX0fQpVJHvPOw=NZadp1mBL17OLHGOdePhr7XvU3_pQ@mail.gmail.com
- XA Transactions for JDBC - NuoDB Documentation - https://doc2.nuodb.com

---

## References

### Official Specifications
- JDBC Developer's Guide (Oracle) - https://docs.oracle.com/en/database/oracle/oracle-database/26/jjdbc/
- Handling Transactions with Databases (Oracle GlassFish) - https://docs.oracle.com/cd/E18930_01/html/821-2418/giybi.html
- JDBC 4.2 Specification - https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_42.html

### API Documentation
- Connection - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/sql/Connection.html
- Savepoint - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/sql/Savepoint.html
- XADataSource - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.sql/javax/sql/XADataSource.html
- XAResource - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.transaction.xa/javax/transaction/xa/XAResource.html

### Tutorials and Articles
- Using Transactions in Java JDBC: Commit, Rollback, Savepoints, Isolation - Cleverence - https://www.cleverence.com/articles/oracle-documentation/using-transactions-the-java-tutorials-jdbc-5831/
- Setting and Rolling Back to a Savepoint - Apache Derby - https://db.apache.org/derby/docs/10.3/ref/crefjavsavesetroll.html
- Two-Phase Commit with JDBC - PostgreSQL - https://www.postgresql.org/message-id/CAF-4BpOGX0fQpVJHvPOw=NZadp1mBL17OLHGOdePhr7XvU3_pQ@mail.gmail.com

### Additional Resources
- HikariCP (Connection Pool) - https://github.com/brettwooldridge/HikariCP
- Atomikos (JTA Transaction Manager) - https://www.atomikos.com/
- Narayana (JTA Transaction Manager) - https://narayana.io/